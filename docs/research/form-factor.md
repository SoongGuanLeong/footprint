# Form factor and per-user background execution

*Research memo for wayfinder ticket #5, grilling question Q10. Written 2026-09-25. Primary sources: Microsoft Learn, Apple developer docs, systemd/loginctl man pages, Microsoft Store Policies v7.20.*

---

## Summary

**v1 is a per-user background daemon plus a CLI. The review/search UI is v2, as a localhost web UI served by the daemon. No desktop framework.**

**A cross-platform daemon+CLI is viable with no admin rights on all three platforms — provided the daemon only runs while the user is logged in.** That proviso holds here because the requirement is session-scoped: the daemon holds an unlocked key *for the session*. The same fact independently disqualifies the Windows Service route on functional grounds, since a session-0 service cannot hold a key unlocked in the user's interactive session.

**The significant consequence: the MSIX route cannot be relied on for v1, so Windows loses the free zero-warning distribution path and the code-signing fallback from Q8 becomes the actual v1 plan.**

---

## 1. Why the daemon is not a choice

Three settled decisions independently require a long-lived per-user process:

- **Q9** — something must hold the unlocked key and serve the MCP process that forwards to it, failing closed when locked.
- **Q7** — local-only inference needs a process managing a local LLM and Whisper.
- **Capture** must run on a schedule.

So Q10's real question is the user-facing shell, not whether a background process exists.

The **CLI is required regardless**: capture control, the unlock prompt, the MCP entry point (`footprint mcp`), and the debugging surface.

**A desktop framework is rejected for v1.** Electron or Tauri means a second toolchain, a GUI packaging problem across three platforms, and a different distribution shape from the CLI+daemon — which matters because the Windows decision is entangled with MSIX. All of that for a single-user tool whose main interaction is search and review.

## 2. Windows: per-user background without admin

| Mechanism | Admin? | Restart on crash | Survives logoff | Notes |
|---|---|---|---|---|
| `HKCU\...\Run` / Startup folder | **No** | **No** | No | Docs: "The system does not provide guarantees about how promptly the programs in the Run key are run" |
| **Task Scheduler, "At log on", per-user, interactive logon type** | **No** | **Yes** (`RestartInterval` + `RestartCount`) | No | **The only no-admin mechanism with crash supervision** |
| Windows Service | **Yes** | Yes | Yes | Doubly disqualified: admin required, and session 0 cannot hold a user-session key |
| `Windows.ApplicationModel.StartupTask` | n/a | No | No | **Package identity only** — no equivalent for unpackaged apps |

**Trap to avoid:** a scheduled task registered with `TASK_LOGON_PASSWORD` or `TASK_LOGON_S4U` "will only launch if the specified user has the Logon as Batch privilege enabled", which is admin-granted and off by default for non-admins. Use the **interactive** logon type.

**Recommended Windows auto-start for v1: a per-user Task Scheduler task with an "At log on" trigger and interactive logon type.** It is the only no-admin option that restarts on crash.

## 3. MSIX: not disqualified by policy, but not safe for v1

**What works:**

- **`windows.startupTask` works for full-trust desktop-bridge apps** — `EntryPoint="Windows.FullTrustApplication"`, no UWP requirement, no capability beyond `runFullTrust`.
- **A long-running no-UI process is fine.** "Unlike UWP apps, desktop apps (including WinUI 3) are not subject to the UWP process lifecycle management (PLM) model of automatic suspension and termination when backgrounded." So the daemon must be a full-trust mediumIL desktop process, **not** a UWP background task. Caveats: Modern Standby can suspend user-mode processes, and Windows may apply Efficiency Mode.
- **App execution aliases exist for full-trust packaged apps**, and console subsystem is supported. Microsoft's own `winappCli` sample confirms stdout stays in the terminal.
- **No Store policy clause prohibits background, headless or CLI operation** — searched the full Store Policies v7.20 text for "background", "startup", "command line", "daemon", "headless", "unattended": zero hits.

**The two unresolved risks, and they hit precisely this design:**

1. **Store visibility and file virtualization — the top risk.** Microsoft's docs **conflict** on whether a full-trust MSIX app's writes to a user-chosen store directory are virtualized per-package (invisible to other processes) or pass through. The containerization overview says full-trust apps "pass through"; `desktop6:FileSystemWriteVirtualization` says the default is *enabled*; "Know your installer" says HKCU writes land in a private per-app hive. The only documented opt-out is `unvirtualizedResources`, which is restricted, described as "not intended to be used for other scenarios", and for which Store approval "in most cases won't be approved". **If virtualization applies, the unpackaged MCP agent cannot read the store and the design breaks.**
2. **stdin forwarding through an execution alias.** stdout is documented; **stdin is not**, and there is a concrete counter-example — a console-subsystem, full-trust packaged app hangs in `fgets()` when launched via its execution alias under Cygwin/MinTTY (Oct 2025), while running the exe from its install path works. For an MCP stdio server, **this is exactly the failure mode**.

Store policy clauses that do bite: **10.5.1** requires a privacy policy for products "that inherently have access to Personal Information… including Desktop Bridge and Win32 products" — footprint captures email, chat and meetings, so this is mandatory, along with 10.5.2 opt-in consent, 10.5.4 modern cryptography and 10.5.5 express consent for highly sensitive data. Also 10.4.2 (prompt start, remain responsive, shut down gracefully) and 10.6 (capability declarations must be legitimate).

**Verdict: MSIX cannot be recommended for the daemon+stdio-MCP design without resolving both risks**, which are OS behaviours the documentation does not guarantee.

## 4. macOS: LaunchAgent, no admin

- Correct location: **`~/Library/LaunchAgents`** — "the user's individual Library/LaunchAgents directory". A user agent "is specific to a given logged-in user and executes only while that user is logged in."
- `RunAtLoad=true` starts it at login; `KeepAlive=true` restarts it on crash.
- Ownership rules are inherently user-scoped: agents "must be owned by that user… must have file mode set to 600 or 400". No admin.
- **macOS 13+ requires user approval.** `SMAppService` registration is "subject to user approval", with a `requiresApproval` status, surfaced in **System Settings > General > Login Items**. macOS 26+ additionally prompts if background tasks outlive the app being quit.
- **Distribution still forces signing and notarisation** — Developer ID, Hardened Runtime, secure timestamp. A LaunchAgent inside the `.app` bundle is covered by the bundle signature; a standalone CLI outside it needs its own signature. Note `/usr/local/bin` is not user-writable, so per-user installs go to `~/.local/bin` or inside the bundle.
- Unverified: whether a directly-installed `~/Library/LaunchAgents` plist (the pre-13 style, not via `SMAppService`) also requires explicit approval on macOS 13+.

## 5. Linux: systemd --user, no root

- Units in `~/.config/systemd/user/*.service`; `systemctl --user enable --now`. No root.
- Without linger, the user manager "will survive as long as there is some session for that user, and will be killed as soon as the last session for the user is closed."
- `Restart=always` + `RestartSec` gives crash supervision.
- **Linger is user-grantable without admin**: systemd's polkit policy allows `org.freedesktop.login1.set-self-linger` for `allow_any`. Linger is what enables running while logged out.
- Non-systemd distros have no per-user service manager; the portable equivalent is the freedesktop autostart spec (a `.desktop` file in `~/.config/autostart/`), which loses crash supervision.

## 6. What Windows v1 should actually be

**An unpackaged per-user install** — NSIS/Inno/WiX per-user mode, or a portable ZIP — auto-started by the per-user Task Scheduler task from §2.

Microsoft's own guidance supports this: unpackaged apps "remain fully unrestricted in terms of API surface, file system access, registry access, elevation, and process model" — which is exactly what this design needs (spawned stdio children, an arbitrary store directory, no service).

**It forces the code-signing path.** For MSIX, "Code signing is handled free by the Store"; for direct download with your own installer, "a CA-trusted certificate is required for non-Store distribution". So the Q8 fallback — SSL.com IV at US$129/yr, or Certum Standard at ~€139–209/yr, or an SSM sole proprietorship at RM30–60/yr to unlock the OV routes — is **not a fallback. It is the v1 plan**, and SmartScreen reputation accumulation applies.

**Middle path worth evaluating later:** *packaging with external location* (a sparse package) — keep the per-user installer and register a lightweight identity package to unlock `windows.startupTask` and other identity-gated features. Microsoft recommends it for exactly this case ("ISV shipping a direct download with own installer"). Caveats: Microsoft's walkthrough declares `runFullTrust` **and** `unvirtualizedResources` (both restricted — fine for sideload, needs Store approval otherwise), and non-Store distribution still requires a CA-trusted certificate. It also does not resolve the stdin question.

## 7. The v2 UI: localhost web server, served by the daemon

Cheaper than a framework: no framework to package, identical on all three platforms, and the daemon already exists and already holds the key. But it introduces a listener, which must be handled deliberately:

- bind **`127.0.0.1` only**, never `0.0.0.0`
- require a **per-session token** — an unauthenticated localhost server is reachable by any local process
- **validate `Host` and `Origin` headers**, or any web page the user visits can reach `127.0.0.1` via **DNS rebinding**. This is the failure that has bitten a long list of local dev servers, and it is why "it's only localhost" is not a security argument by itself
- run the listener **only while the UI is open**

Deliberate contrast with Q9: the **MCP path stays stdio with no listener**; the UI listener is separate, opt-in and short-lived. Two surfaces, two exposure models, chosen on purpose.

The **unlock UX** is what brings the UI forward earlier than aesthetics would. v1: unlock via the CLI at login, key held by the daemon for the session. The web UI then becomes the natural home for unlock later.

---

## Could not verify

1. Whether a full-trust (mediumIL) MSIX app's writes to a user-chosen directory are virtualized per-package or pass through. **Docs conflict. Top risk for the MSIX route.**
2. Whether stdin is forwarded through an app execution alias when the parent is a normal Win32 process using redirected handles. Only a Cygwin/MinTTY failure report found; no Microsoft doc states it either way.
3. Whether a per-user Task Scheduler task appears in Task Manager's Startup apps list.
4. Whether a directly-installed `~/Library/LaunchAgents` plist requires explicit approval on macOS 13+.
5. Store policy beyond the public v7.20 page — the actual restricted-capability approval criteria are not public.
6. Whether child processes spawned by a full-trust packaged app retain package identity.
7. There is **no explicit Microsoft sentence** saying "MSIX install requires no admin"; the no-admin property is documented by implication ("App packages are installed on a per-user basis instead of system-wide", plus admin being required specifically when the package contains a service).

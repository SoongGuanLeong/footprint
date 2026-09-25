# Hermes MCP integration constraints

*Research memo for wayfinder ticket #5, grilling question Q9. Written 2026-09-25. Read-only static review of `NousResearch/hermes-agent` at commit `7b761da2de4979e424510ca7022bf9527aa65b68` — the same commit the #4 audit pinned. Nothing was installed or executed.*

---

## Summary

**Hermes' MCP client supports stdio natively, so footprint does not need a localhost HTTP server.** stdio is the right choice: no port, no listener, no OAuth, matching the single-user encrypted-archive model.

Two findings change the design:

1. **Hermes will NOT enforce read-only on footprint's behalf.** It does not filter by `readOnlyHint`, and its default trust level lets write-capable tools run with no approval. So the read-only guarantee rests entirely on footprint's own server — and the user-facing Hermes config snippet becomes a **deliverable**, not documentation.
2. **The stdio child does not inherit the shell environment.** Only a small safe baseline plus explicitly declared `env` keys reach the server. Anything footprint needs — the daemon socket path above all — must be declared in config.

---

## 1. Transports

All three are implemented in code, not just documented:

- **stdio** — `tools/mcp_tool.py:154-156` imports `StdioServerParameters` and `stdio_client`; bring-up in `tools/mcp_tool_transport.py:308-360` (`_run_stdio`).
- **Streamable HTTP** — the **default for `url`-based servers** (`_streamable_http_transport`, `transport.py:490`).
- **Legacy HTTP+SSE** — `_sse_transport`, `transport.py:460`, selected with `transport: sse`, plus an automatic one-shot SSE fallback on a transport-mismatch rejection.

Transport selection is by config shape: `command`/`args`/`env`/`cwd` for stdio, `url`/`headers` for HTTP. Confirmed by `tools/mcp_tool_discovery.py:714`: `{"transport": cfg.get("transport", "http") if "url" in cfg else "stdio"}`.

**Conclusion:** stdio is available and preferred. No localhost server is required.

## 2. How a local stdio server is declared

Config lives in `~/.hermes/config.yaml` under `mcp_servers`. Keys: `command`, `args`, `env`, `cwd`.

```yaml
mcp_servers:
  footprint:
    command: "footprint"
    args: ["mcp"]
    env:
      FOOTPRINT_SOCKET: "${HOME}/.local/share/footprint/daemon.sock"
    cwd: "${HOME}"
    trust: untrusted
    tools:
      include: ["search", "get_record", "timeline", "list_people", "get_transcript", "coverage"]
    sampling:
      enabled: false
    elicitation:
      enabled: false
```

**Credentials for stdio are environment-only.** `tools/connectors/mcp_oauth.py:313` states it: "it takes env keys, not OAuth". `env` values support `${VAR}` interpolation resolved at connect time, including from `~/.hermes/.env`.

### The environment-isolation trap

`tools/mcp_tool_config.py:111-134` (`_build_safe_env`) filters the child environment deliberately, "so API keys/tokens don't leak". The baseline is only:

`PATH`, `HOME`, `USER`, `LANG`, `LC_ALL`, `TERM`, `SHELL`, `TMPDIR` — plus a fixed set of Windows process/location vars, plus `XDG_*`, plus names injected by an external secret source, plus the config's own `env` (merged last, so it wins).

**Consequence:** a footprint MCP process cannot rely on inheriting anything else from the user's shell. Every path it needs — daemon socket, config directory, log destination — must be passed explicitly through `env:`. This would otherwise fail in a confusing way, since the process would start successfully and then fail to find the daemon.

## 3. Read-only is not enforced by Hermes

This is the most important section, and it invalidates any assumption that the client will help hold the boundary.

- **Tool filtering exists, but it is the user's config, not a guarantee.** `tools/mcp_tool_registration.py:210-225` (`_make_tool_filter`): `tools.include` is a whitelist (`[]` registers nothing), `tools.exclude` a blacklist, entries are exact names or fnmatch globs, and include wins over exclude.
- **Hermes does not filter by `readOnlyHint`.** It captures the annotation at discovery (`registration.py:48-53`, "unknown = write-capable") and uses it for exactly two things: approval gating **only** on servers marked `trust: untrusted` (`handlers.py:58-84`), and retry safety after a session expires (`handlers.py:567-587`).
- **Default trust is `full`** (`registration.py` `_normalize_server_trust`). On a default server, **write-capable tools run with no approval at all.**
- The config reference itself says the hint is untrusted: "`readOnlyHint` is a server-supplied *hint* — a lying server can at most skip approval for tools it claims are read-only, never gain extra access — so mark any server you don't fully control as `untrusted`."

**Therefore the only hard guarantees are footprint's own:** (1) the server advertises read-only tools and nothing else, and (2) the user's config carries a `tools.include` allowlist. The two controls from the OSS survey stand — **no write tools registered**, plus a **read-only database role** — and Hermes adds nothing to them.

**Deliverable implication:** footprint must ship the Hermes config snippet above as part of the product, with the allowlist, `trust: untrusted`, and sampling/elicitation disabled. Leaving the user to write it would leave the default posture weaker than the design assumes.

## 4. MCP revision target

Hermes pins the SDK exactly: `pyproject.toml:398-402` — `mcp==2.0.0`, `httpx2==2.7.0`, `starlette==1.3.1`. The comment states mcp 2.0.0 "implements MCP revision 2026-07-28". Verified independently from `mcp-types 2.0.0`'s `version.py`:

```
KNOWN_PROTOCOL_VERSIONS      = ("2024-11-05", "2025-03-26", "2025-06-18", "2025-11-25", "2026-07-28")
HANDSHAKE_PROTOCOL_VERSIONS  = ("2024-11-05", "2025-03-26", "2025-06-18", "2025-11-25")   # reachable via initialize
MODERN_PROTOCOL_VERSIONS     = ("2026-07-28",)                                            # stateless per-request envelope
LATEST_HANDSHAKE_VERSION     = "2025-11-25"
```

Negotiation is per-server via a `protocol` key: `auto` (default) tries the legacy `initialize` handshake **first** and falls back to the 2026-07-28 `server/discover` probe only on a modern-only signal; `stateless` probes discover first; `legacy` is handshake-only.

Notably `transport.py:544-546` seeds the HTTP `mcp-protocol-version` header from `LATEST_HANDSHAKE_VERSION` (**2025-11-25**), deliberately not the latest, because "a 2026-07-28 header routes the handshake-era initialize() onto the envelope ladder, which rejects it".

**Conclusion: target `2025-11-25`** (or `2025-06-18`). That is the zero-friction path since `auto` tries `initialize` first. `2026-07-28` stateless also works, but only via the discover fallback — an unnecessary risk for no benefit here.

## 5. Size limits — transcripts must paginate

- **`_MCP_HARD_RESULT_CAP_CHARS = 2_000_000`** per text payload; over it, head+tail truncation (`tools/mcp_tool_content.py:20-33`). **Applies to stdio too.**
- **`_MCP_HTTP_MAX_BODY_BYTES = 10 MiB`**, applied at the HTTP transport before the SDK parses; each SSE event is also capped (`tools/mcp_tool_errors.py:304-320`). HTTP only — stdio has no wire-body cap.
- **Resource cache: 50 MiB** decoded per block, rejected on base64 length before decoding.

**Consequence for the tool set:** a multi-hour meeting transcript returned as a single tool result will hit truncation (stdio) or a body rejection (HTTP). `get_transcript` and any large record retrieval **must paginate**, or return a handle the agent reads in chunks. This is a hard design constraint, not a nicety.

## 6. Other constraints

- **Resources and prompts are supported** in addition to tools (`list_resources`, `read_resource`, `list_prompts`, `get_prompt`), capability-gated. Transcripts could be exposed as resources, which fits the pagination constraint well. The wrappers can be disabled separately via `tools.resources: false` / `tools.prompts: false`.
- **Sampling and elicitation are enabled by default** (`sampling/createMessage`; `elicitation/create` form mode). A footprint server that never uses them is unaffected, but the recommended config turns both off to shrink the surface.
- **Timeouts:** tool call 300 s default, `connect_timeout` 60 s (also bounds the handshake), discovery ceiling 300 s.
- **Tool naming** is `mcp__<server>__<tool>` (double underscore) in code and the config reference; the user guide still shows single underscores and is stale. Code is authoritative.
- **Tool-result sanitisation:** invisible Unicode TAG characters (U+E0000–U+E007F) are stripped from results, resource content and tool descriptions.
- **Parallel tool calls are off by default** (`supports_parallel_tool_calls: true` to opt in).
- **stdio lifecycle:** optional `idle_timeout_seconds` / `max_lifetime_seconds` recycling, and `lazy: true` to defer spawn to first tool call. Relevant because footprint's MCP process forwards to the daemon and must tolerate being re-spawned.

## 7. Recommended footprint posture

1. **Implement stdio.** No listener, no port, no OAuth.
2. **Register read-only tools only**, hand-written, plus resources for large transcripts. No write tools, no SQL passthrough, no web or URL fetch, no filesystem path parameter.
3. **Enforce the boundary server-side**: no write tools advertised, and a read-only database role. Do not rely on `readOnlyHint` or on the client.
4. **Ship the Hermes config snippet** with `tools.include` allowlist, `trust: untrusted`, `sampling.enabled: false`, `elicitation.enabled: false`.
5. **Declare every needed path in `env:`** — the stdio child inherits almost nothing.
6. **Target MCP revision `2025-11-25`.**
7. **Paginate anything that can exceed ~2M characters.**

---

## Could not verify

- Static review only; nothing installed or run, so no behavioural testing of the transport paths.
- The `mcp-types` version registry was read from the PyPI wheel for `mcp-types 2.0.0`, the exact transitive pin of `mcp==2.0.0` — the SDK's own release rather than Hermes' code.
- Whether the `mcp` extra is in Hermes' *default* install: the docs say MCP ships with the standard install, while `pyproject.toml` lists `hermes-agent[mcp]` inside the `all`/`termux` extras and not (as far as was visible) the bare `dependencies`. Not load-bearing for the transport decision.
- Doc inconsistency flagged but unresolved: user-guide tool naming (`mcp_<server>_<tool>`) vs code and config reference (`mcp__<server>__<tool>`).
- Retrieval note for reproduction: `raw.githubusercontent` rate-limits without a User-Agent; jsDelivr 404'd on three files that do exist at the commit (cache staleness, not absence), so the git trees API was used to confirm existence.

---
id: 0038
title: MCP check, ADR-005 amendment, WhatsApp
stage: general
date: 2026-10-13
surface: agent
model: claude-opus-5-5
feature: none
branch: master
user: asmaiqbal01
command: free-form owner decisions (no /sp command)
labels: ["adr-005", "amendment", "mcp", "openclaw", "whatsapp", "f013", "route-1", "constitution", "read-only"]
links:
  spec: null
  ticket: null
  adr: adrs/ADR-005-runtime-integration-strategy.md (Proposed, DRAFT; Amendment 1)
  pr: null
files:
 - adrs/ADR-005-runtime-integration-strategy.md (modified: Option B note; Amendment 1 appended)
 - history/prompts/general/0038-mcp-check-adr005-amendment-whatsapp.general.prompt.md (this file, new)
tests:
 - none run (read-only documentation and package-metadata checks only)
---

## Prompt

### Prompt 4 (verbatim)

Reviewed. Decisions:
1. Keep the two verbatim lines in the PHR. Do not redact.
2. I'll chmod 600 the checklist myself.
3. Approved: check (b1), read-only and offline. Look only at OpenClaw
   package metadata and docs to see whether it can call MCP tools. Do not
   open openclaw.json or any backup, any .env, or any credential file. Do not
   start OpenClaw, an agent, a server or a tunnel. Report what you read,
   with file and line.
4. Do not edit ADR-005 yet. After (b1), show me a short proposed
   amendment to ADR-005 covering the constitution conflicts, as text in
   the terminal. Include a route where F013 is specced first, and a
   route where I amend the constitution for a labelled test run. Do not
   edit CONSTITUTION.md.
5. Do not commit anything. Then stop and wait.

### Prompt 5 (verbatim)

Decisions:
1. Apply Amendment 1 to ADR-005 with two edits: (a) check the F013 spec
   score threshold against CONSTITUTION.md and the spec-scorer file and
   use the real value, citing the line; (b) state that Option B is
   withdrawn as a primary option and kept only as a fallback if C fails
   its probe. Edit only ADR-005. Do not edit CONSTITUTION.md.
2. Route 1 is my choice. Record Route 2 as considered and not chosen.
3. Write the PHR for this session's prompts (verbatim, per CLAUDE.md).
   Same checks as before: no secrets, no long IDs, no dashboard endpoint
   details.
4. Read-only and offline: read OpenClaw's built-in WhatsApp channel docs
   and code under ~/.npm-global/lib/node_modules/openclaw/. Report: how
   the channel connects (official API or a WhatsApp-Web-style session),
   what it needs from a phone number, known limits or ban warnings in the
   docs, and whether per-tenant numbers are supported. Cite file and line.
   Do not open any config or credential file. Do not start OpenClaw.
5. Do NOT run openclaw mcp probe yet. It waits until I confirm credential
   rotation.
6. No git add or commit. Stop after reporting and wait for me.

## Response snapshot

All checks were read-only and offline, run against the installed package at
`~/.npm-global/lib/node_modules/openclaw/` (`package.json:3` = 2026.6.11).
The `openclaw` binary was never run. No config, `.env` or credential file was
opened.

### (b1) Can OpenClaw 2026.6.11 call MCP tools? Documented yes; not proven on this machine

- MCP SDK dependency: `package.json:1965` (`@modelcontextprotocol/sdk` 1.29.0).
- Client registry consumed by embedded runtimes: `docs/cli/mcp.md:11-19`, `:376`.
- Tools exposed in `coding`/`messaging` profiles, hidden in `minimal`: `docs/cli/mcp.md:394`.
- `mcp.servers` stdio/remote: `docs/gateway/configuration-reference.md:89-94`, `:137-138`.
- `bundle-mcp` plugin id and sandbox gate: `docs/gateway/config-tools.md:46-77`.
- `PYTHONPATH` rejected in a stdio server env: `docs/cli/mcp.md:680`.
- Stdio fix in this release: `CHANGELOG.md:71` (2026.6.11 section starts `:5`).
- 53 files under `dist/` reference `bundle-mcp` (grep).
- Unproven: the active tool profile and sandbox mode (config not opened);
  whether DealClaw agents receive the tools; a live `openclaw mcp probe`
  (deferred by owner until credential rotation is confirmed).

### Spec-score threshold for F013 (verified)

≥ 9.6/10: `CONSTITUTION.md:73`, `:81`; `agents/spec-scorer/spec-scorer.md:5`, `:10`, `:82`.

### ADR-005 Amendment 1 (applied; only ADR-005 edited)

- Status unchanged: Proposed (DRAFT, decision pending owner) for G1.
- Option B: withdrawn as a primary option and kept only as a fallback if
  Option C fails its probe. A note was added under the Option B heading,
  and the same is restated in the amendment.
- Constitution conflicts listed: B vs Hard Rule 7/I5; a1 vs Phase 1 gate;
  a2 vs Hard Rule 8/I3; c2 vs Section 5/I2; G4 vs I4.
- Open question (a): **Route 1 (spec F013 first) chosen by owner.** Route 2
  (labelled test-run constitution exception) recorded as considered, not chosen.
- `CONSTITUTION.md` not edited.

### OpenClaw WhatsApp channel (read-only; `docs/channels/whatsapp.md`)

- **Connection**: WhatsApp Web via the Baileys library, not an official
  business API. "Status: production-ready via WhatsApp Web (Baileys). Gateway
  owns linked session(s)." (`:8`). "There is no separate Twilio WhatsApp
  messaging channel" (`:155-158`).
- **Phone-number needs**: a phone scans a QR code to link the number as a
  device; setup is QR-only (`:63-70`, `:113-117`). A dedicated number is
  recommended; a personal number is supported with self-chat safeguards
  (`:109`, `:122-153`, `:356-362`). Allowlists use E.164 numbers (`:261`).
- **Install**: the WhatsApp runtime is an external plugin
  `@openclaw/whatsapp` (`:10-20`). Not installed here: only a Discord plugin
  project exists under `~/.openclaw/npm/projects/` (names only); a
  `~/.openclaw/credentials` directory does not exist (presence check only).
- **Limits in the docs**: outbound sends need an active listener for the
  sending account (`:168`, `:703-708`); a send counts as delivered only once
  Baileys returns an outbound message id (`:710-716`); text chunk limit 4000
  (`:459`); outbound media cap 50 MB default (`:481`); reconnect loops and
  re-linking are covered under troubleshooting (`:641-695`).
- **Ban / terms warnings**: none in the docs (grep for
  ban/suspend/terms of service/unofficial: no hits). Ban risk for an
  unofficial WhatsApp Web client is **unverified** and belongs in the F013
  spec's risk section.
- **Per-tenant numbers**: multiple accounts are supported via
  `channels.whatsapp.accounts.<id>`, with per-account auth directories and
  per-account policy overrides (`:591-605`, `:265`, `:229-245`). Each account
  is a separately QR-linked number. Inbound routing to agents uses `bindings`
  with `accountId` (`docs/channels/channel-routing.md:17-19`, `:79-85`, `:113`).
  Note: v2's design sends notifications *to* each agent's number
  (`workspace/tenants/_template/USER.md:5`), which needs only one sending
  account, not one linked number per tenant (inference; to be settled in F013).

## Outcome

- ✅ Impact: Option C confirmed documented-viable on the installed runtime; ADR-005 amended (Option B demoted to fallback; Route 1 recorded); F013 inputs gathered (Baileys/QR, external plugin, multi-account support, no ban warning in docs).
- 🧪 Tests: none run; documentation and package metadata reads only.
- 📁 Files: ADR-005 modified; this PHR created. No other files touched.
- 🔁 Next prompts: owner confirms credential rotation → approve `openclaw mcp probe` against v1's server; owner decides G1; `/sp.specify` F013 (PK WhatsApp), with ban risk and sending-number model as explicit spec questions.
- 🧠 Reflection: "Built-in WhatsApp" in `CONSTITUTION.md` and `workspace/TOOLS.md` is actually an external plugin over an unofficial WhatsApp Web session. That is a material assumption for the Phase 1 gate, now surfaced for F013.

## Evaluation notes (flywheel)

- Failure modes observed: documentation language ("built-in", "official plugin") can mask that the transport is an unofficial WhatsApp Web client; version drift across README, ADR-001 and the install persists.
- Graders run and results (PASS/FAIL): post-write checks (placeholders, token-shaped strings, long digit runs, dashboard endpoint details) — see session report.
- Prompt variant (if applicable): none
- Next experiment (smallest change to try): after rotation, a single `openclaw mcp probe` against v1's FastMCP server to prove the OpenClaw→MCP link on this machine.

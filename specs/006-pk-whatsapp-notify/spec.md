# Feature Specification: PK WhatsApp Lead Alert Adapter + Owner Relay Fallback (F013)

**Feature Branch**: `006-pk-whatsapp-notify`
**Created**: 2026-11-12 | **Revised**: 2026-11-12 (owner transport decision)
**Status**: Draft, not approved. Spec-scorer: round 1 = 5.5/10, round 2 = 5.6/10 (BLOCKED). Round-2 gaps 3–14 and, with read-only OpenClaw docs, gaps 1–2 were addressed on 2026-11-12; this version is **unscored** (owner skipped a third round).
**Milestone**: F013, inside the named milestone range F008–F017
(`CONSTITUTION.md:209`). Scope matches the F013a half of the owner's
2026-10-13 split (PHR 0039). Inbound handling (the `DONE <lead_id>` reply and
automated opt-in) is F013b, a separate spec.
**Business milestone unlocked**: Phase 1 gate, PK Validation — "3 Pakistani
agencies confirmed — defined as having received at least one delivered lead
notification through the pipeline" (`CONSTITUTION.md:50-52`). F013 produces
that delivered notification.
**Market**: PK Real Estate only (gate I1, `CONSTITUTION.md:181-182`).
**Lead source**: Zameen.com alert emails (primary), plus OLX Pakistan
property emails and direct enquiry emails already handled by Intake
(`agents/intake/SOUL.md:15-16`). F013 receives the lead after Intake and the
CRM write; it does not parse email.
**Locale**: every text an agent reads (M2, and the pre-filled text inside the
M4 `wa.me` link) is bilingual Roman Urdu + English. Owner-only messages
(M4 body, M5, M6, M8) are English.
**Input**: Owner brief of 2026-11-12 and owner decisions of 2026-11-12
(PHR 0040, PHR 0041), plus PHR 0039 and ADR-005 Amendment 1.
**Spec-score gate**: ≥ 9.6/10 against `agents/spec-scorer/spec-scorer.md`
before `/sp.plan` (`CONSTITUTION.md:73`, `:81`). `plan.md` and `tasks.md` in
this folder are drafts and stay blocked until this spec passes and the owner
approves it.

## Purpose

Deliver each qualified PK lead to the agent on WhatsApp, free of charge,
during the pilot. All sends go through one adapter interface
(`send_lead_alert`), so the transport changes by configuration. When WhatsApp
cannot deliver, one email goes to the owner with a `wa.me` link, and the
owner forwards the lead to the agent on WhatsApp. The agent only ever
receives WhatsApp.

## Owner decisions applied (2026-11-12)

| # | Decision |
|---|---|
| OD-1 | Pilot transport (pilot only, behind `send_lead_alert`): T1, OpenClaw's WhatsApp plugin `@openclaw/whatsapp`, an unofficial WhatsApp Web client (Baileys underneath) on the dedicated DealClaw number. Free. Transport id `openclaw_whatsapp_web`. |
| OD-1a | The runtime decision (ADR-005 G1: Option A OpenClaw only, Option B v1 Python, Option C hybrid) is **still open**. T1 applies when the runtime is OpenClaw (Option A or C). If the owner decides the runtime is not OpenClaw (Option B), the alternate transport is T5, a self-run WhatsApp Web library on the same dedicated number, transport id `selfrun_whatsapp_web`, behind the same interface. Exactly one of the two is configured at any time. |
| OD-2 | The free transport is **pilot-only**. When the first paying client signs, the transport switches to the official, paid WhatsApp Cloud API (transport id `whatsapp_cloud_api`). The switch is a configuration change (FR-016), not a rewrite. |
| OD-3 | Fallback: owner relay. One email (M4) to the owner's relay inbox, containing a `wa.me` link. |
| OD-4 | The owner's relay inbox is referenced only as env var `OWNER_RELAY_EMAIL`. The sending number is referenced only as env var `WHATSAPP_BUSINESS_NUMBER`. Neither literal value appears in any repo file. |
| OD-5 | The owner accepts the ban risk of an unofficial client for the pilot (RISK-1). |
| OD-7 | Pilot recipient model: exactly one recipient per tenant, the tenant's `agent_whatsapp`. More than one agent per tenant, or a WhatsApp group, is not in F013 (L7). If the pilot client's written confirmation (G-3) asks for a different model, F013 returns to spec before go-live. |
| OD-8 | A lead counts toward the Phase 1 gate only when its `notify_status` is `delivered` (a provider message id exists). A lead the owner forwards by hand from M4 stays `notify_pending` and does not count. |
| OD-6 | Owner alerts on Discord are operator communication, outside Hard Rule 8 (PHR 0039, OQ-6 = a, scoped to Discord). **Owner ruling, 2026-11-12 (PHR 0041):** owner-relay emails sent only to the owner's own inbox (`OWNER_RELAY_EMAIL`) are operator communication, the same as Discord owner alerts, so they fall outside Hard Rule 4 (`CONSTITUTION.md:155-157`) and need no approval-queue entry. Any email to an address other than `OWNER_RELAY_EMAIL` is not covered by this ruling. |

## Go-live gate *(mandatory)*

No real lead is sent to a pilot agent until **all three** are true, each
recorded as an ISO 8601 date in the owner config (§Key Entities):

- **G-1** `golive_discord_trace_at`: one live lead delivered through Discord
  and proven with an ID-only trace (`run_id`, `tenant_id`, `lead_id`, provider
  message id, timestamps; no lead content).
- **G-2** `golive_spec_approved_at`: this spec approved by the owner after
  passing the ≥ 9.6 gate.
- **G-3** `golive_client_confirmed_at`: Inhabitant Construction has confirmed
  in writing how they receive leads.

While any of the three is null, the transport is forced to `fake`
(FR-003). Owner action A-1 covers how G-1 is produced.

## Context (facts this spec depends on)

| # | Fact | Source |
|---|---|---|
| K1 | PK-mode notifications MUST use WhatsApp only; neither market substitutes the other's channel. | `CONSTITUTION.md:164-165`; gate I3 `:184-185` |
| K2 | Zero-cost constraint: no paid APIs, hosting or infrastructure; never add a paid tool without gate I2 approval. | `CONSTITUTION.md:131-134`, `:145-147`; gate I2 `:183` |
| K3 | Tool priority: OpenClaw built-in channel before an external API. | `CONSTITUTION.md:145-147` |
| K4 | Notification only after a confirmed CRM write. | `CONSTITUTION.md:152`; `agents/delivery/SOUL.md:26-29` |
| K5 | PK message line: `🏠 New lead: [contact name or 'Unknown'] \| [property type] \| [location] \| Score: [score] \| Source: [source]`; `🔴 URGENT — ` prepended when score ≥ 0.9. | `agents/delivery/SOUL.md:32-34` |
| K6 | WhatsApp send failure: retry once, then log and continue. | `CONSTITUTION.md:168-170` |
| K7 | At most 10 Discord/WhatsApp notifications per run, per tenant. | `CONSTITUTION.md:122-124`; `workspace/HEARTBEAT.md:38` |
| K8 | Tenant `USER.md` holds `agent_whatsapp` (format `+923XXXXXXXXX`) and `active`. | `workspace/tenants/_template/USER.md:5`, `:13`; `workspace/USER.md:33` |
| K9 | v1 separates notifier logic from transport: the `notify` contract MUST NOT raise and returns delivered or pending; `DiscordAdapter.notify` turns every send failure into pending. (v1's factory and constructor do raise on missing config; F013 handles config errors as E4/E8 instead.) | `~/openclaw-deal-scout/src/discord_notifier/adapter.py:11-14`, `:25-68` |
| K10 | `@openclaw/whatsapp` is WhatsApp Web based (Baileys), not an official business API; not installed; latest plugin needs core ≥ 2026.10.2 while 2026.6.11 is installed; plugin 2026.6.11 exists. | Baileys and install status: `adrs/ADR-005-runtime-integration-strategy.md:325-338`; versions: `history/prompts/general/0039-f013-spec-interview-scoring-split.general.prompt.md:189-192` |
| K11 | Sends fail fast when no listener is active for the WhatsApp account. | OpenClaw docs `channels/whatsapp.md:168`, `:703-708` (PHR 0039) |
| K12 | Per-tenant MEMORY.md lives at `~/.openclaw/workspace/tenants/{tenant_id}/MEMORY.md`. | `workspace/MEMORY.md:76` |
| K13 | v1 has a Gmail send client (`gmail.send` scope). | `~/openclaw-deal-scout/src/email_scheduler/auth.py:1`, `:15` |
| K14 | `unauthorized_reply_source` = "reply came from a number other than `agent_whatsapp`". F013 does not change it. | `specs/001-pk-lead-intake-notify/contracts/approval-commands.md:46` |

## Definitions

- **Heartbeat run**: one 15-minute cycle (`workspace/HEARTBEAT.md:4`). It
  evaluates channel state first (FR-009), then processes active tenants one
  at a time in ascending `tenant_id` order.
- **Lead send**: one lead's attempt sequence (first attempt plus its retry,
  if any). A probe is one lead send.
- **Eligible**: a lead in `awaiting_send`, or in `notify_pending` with
  `queued_at` ≤ 24 hours ago and `last_error_type` ≠ `crash_recovered`.
  Crash-recovered leads (E14) are never eligible; they end only in
  `notify_expired`.
- **Transport**: the agent-facing adapter selected by owner config field
  `agent_transport`.

## Secrets and identifiers *(mandatory)*

- `WHATSAPP_BUSINESS_NUMBER` (sending number) and `OWNER_RELAY_EMAIL` (relay
  inbox) are env vars. Their values never appear in a spec, plan, task,
  test, fixture, log or commit.
- Transport and relay credentials are referenced only by env-var names held
  in owner config (ADR-001 pattern). Literal values never appear in a file.
- Tests use `fake` adapters and fixture values that are not real numbers or
  addresses.

## Transport

### Pilot transport — `openclaw_whatsapp_web` (T1, OD-1)

- WhatsApp Web session on the dedicated DealClaw number, linked by QR scan,
  held by the OpenClaw gateway.
- Free per message. Free-form text (M2). No template approval.
- Meets K3 (built-in OpenClaw channel).
- Pilot-only (OD-2).
- Applies only when ADR-005 G1 = Option A or C (OD-1a).
- Before install: the owner resolves the plugin/core version mismatch (K10)
  and records `whatsapp_plugin_publisher_verified_at` in owner config.

### Alternate pilot transport — `selfrun_whatsapp_web` (T5, OD-1a)

- Used only if ADR-005 G1 = Option B (runtime is not OpenClaw).
- Self-run WhatsApp Web library on the same dedicated number; same
  unofficial-client ban risk as T1 (RISK-1), same session-drop behaviour
  (RISK-2), same error mapping (S2), same limits (FR-010).
- Adds a third-party dependency and custom code, ranked below a built-in
  channel by K3. The library is chosen in `/sp.plan` only if Option B is
  decided.

### Production transport — `whatsapp_cloud_api` (OD-2; not built in F013)

- Official Meta API. Paid per message; per-country rates **unverified**.
- Business-initiated messages need a pre-approved template; a 24-hour
  customer-service window applies (**unverified** detail).
- Switching requires gate I2 approval and the Section 5 zero-cost rule to be
  addressed for paying clients (K2). That is a precondition of the switch,
  owned by the owner, not part of F013.
- F013 guarantees the switch is a configuration change (FR-016, AT-14). The
  `whatsapp_cloud_api` adapter module itself is written in the feature that
  performs the switch, against the interface below, with zero changes to
  Delivery.

### Options considered and not chosen for the pilot

T2 Cloud API for the pilot (cost, K2), T3 click-to-chat as the primary path
(not automatic; kept as the relay link), T4 provider trials (expire, then
paid). T5 is kept only as the alternate in OD-1a.

## Fallback: owner relay by email (OD-3)

- When the transport cannot deliver, the lead becomes `notify_pending` and
  one M4 email goes to `OWNER_RELAY_EMAIL`.
- M4 carries a `wa.me` link to the tenant's `agent_whatsapp` with a
  pre-filled bilingual sentence and the `lead_id`. The owner taps it and
  sends from their own WhatsApp.
- The relay adapter sends through the Gmail API `users.messages.send` with
  the `gmail.send` scope (K13), using the sender credential named in owner
  config.
- Discord, email or Telegram **to the agent**: rejected (K1).

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Agent receives the lead on WhatsApp (Priority: P1)

**Why**: the delivered notification is what the Phase 1 gate counts.
**Independent Test**: AT-01, AT-11.
**Acceptance Scenario**: **Given** G-1 to G-3 recorded, `agent_transport` =
`openclaw_whatsapp_web`, an opted-in tenant, channel `active`, and a
Zameen.com lead with a confirmed CRM write (K4), **When** Delivery runs,
**Then** exactly one M2 goes to `agent_whatsapp`, the result has a provider
message id, and `notify_status` = `delivered`.

### User Story 2 — Owner relay when WhatsApp cannot deliver (Priority: P1)

**Why**: the lead still reaches the agent on WhatsApp, forwarded by the owner.
**Independent Test**: AT-02, AT-03, AT-12, AT-13.
**Acceptance Scenario**: **Given** a lead whose outcome is "relay" in the
outcome table, **When** Delivery finishes that lead, **Then**
`notify_status` = `notify_pending` and one M4 email is sent to
`OWNER_RELAY_EMAIL`.

### User Story 3 — Switch to the paid API by configuration (Priority: P1)

**Why**: OD-2.
**Independent Test**: AT-14.
**Acceptance Scenario**: **Given** a second adapter registered under
`whatsapp_cloud_api`, **When** owner config `agent_transport` changes from
`openclaw_whatsapp_web` to `whatsapp_cloud_api`, **Then** the next run sends
through the new adapter with no change to Delivery code, lead states, log
fields or message contracts.

### User Story 4 — Ban or session drop is detected fast (Priority: P1)

**Why**: RISK-1 and RISK-2.
**Independent Test**: AT-03, AT-05, AT-06, AT-07.
**Acceptance Scenario**: **Given** the number is logged out or the session
has dropped, **When** the first lead of a run ends with `logged_out` or
`session_down`, **Then** the channel becomes `degraded`, one M5 is sent, and
the remaining leads in that run are relayed with `channel_degraded`.

### Edge Cases

- **E1 Not opted in.** Tenant `USER.md` `whatsapp_opt_in_confirmed_at` is
  null, absent, or not a valid ISO 8601 date → no send; outcome
  `not_opted_in` (relay).
- **E2 Tenant errors.** (a) `tenant_id` missing or empty →
  `tenant_id_missing`; (b) no `USER.md` for it → `tenant_not_configured`;
  (c) differs from the tenant being processed (Hard Rule 5,
  `CONSTITUTION.md:158-159`) → `tenant_mismatch`; (d) its `USER.md` exists
  but is not valid JSON → `tenant_config_invalid`. No send, nothing written
  to any tenant directory, one line in the workspace run log, one M8 per
  lead. For (d), the tenant's existing notify records are left untouched
  and the tenant is skipped for the rest of the run.
- **E3 Invalid agent number.** `agent_whatsapp` missing, equal to the
  template placeholder (`workspace/tenants/_template/USER.md:5`), or not
  matching `^\+923[0-9]{9}$` → `notify_blocked` with `agent_number_invalid`;
  no M4; one M8 per tenant per run.
- **E4 Invalid transport value.** `agent_transport` not one of `fake`,
  `openclaw_whatsapp_web`, `selfrun_whatsapp_web`, `whatsapp_cloud_api`, or names an adapter that is
  not registered → `transport_invalid`; every lead `notify_blocked`; one M8
  per run.
- **E5 Recovery.** Each run re-checks every `notify_blocked` lead with the
  check that blocked it. Pass and age ≤ 24 h → `awaiting_send`.
- **E6 Expiry.** Any lead in `awaiting_send`, `notify_pending` or
  `notify_blocked` older than 24 h → `notify_expired`; one M6 per tenant per
  run lists them.
- **E7 Owner message send fails.** Every owner message (M4, M5, M6, M8)
  gets exactly one retry 10 s after a failed first attempt. If the retry
  also fails: an M4 stays queued and is sent again at the start of the next
  run, for at most 3 runs in total, then `relay_abandoned` is logged and no
  further attempt is made; an M5, M6 or M8 is logged as `owner_alert_failed`
  and not attempted again. Lead state never changes because of an owner
  message failure.
- **E8 Relay not configured.** `OWNER_RELAY_EMAIL` unset or empty, or the
  sender credential env var unset → `owner_channel_not_configured`, no
  attempt, logged once per run; queued M4s keep their state and do not use
  up their 3 runs.
- **E9 Message length.** In M2: location capped at 60 characters; property
  type, contact name and source at 40 each; each truncated value ends in
  `…`. `lead_id`, score, URGENT prefix and the bilingual line are never
  truncated. M2 is under 400 characters.
- **E10 Missing lead fields.** Contact name null → `Unknown` (K5). Property
  type, location or source null → `-`. `lead_id` or `classification_score`
  missing → Delivery's schema check has already rejected the lead; F013
  never receives it.
- **E11 Transport quota or rate limit.** A provider rate-limit response →
  `quota_exceeded`; relay; channel `degraded` until the next run's probe.
- **E13 Missing or unreadable files.** (a) `notify-records.json` missing →
  created empty. (b) `notify-records.json` present but not valid JSON →
  `state_corrupt` for that tenant: no sends for that tenant this run, the
  file is not overwritten, one M8. (c) Channel-state file missing → created
  with `state` = `active`, counter 0. (d) Channel-state file not valid JSON →
  `state_corrupt`, treated as `degraded` (FR-009 probe applies), file
  rewritten with `state` = `degraded`, one M5. (e) `owner.json` missing or
  not valid JSON → `owner_config_invalid` logged to the run log; transport
  forced to `fake`; relay treated as not configured (E8). (f) A G-1, G-2,
  G-3 or `whatsapp_plugin_publisher_verified_at` value that is not a valid
  ISO 8601 date → treated as null.
- **E14 Crash between send and record.** Before each attempt, the lead's
  record is written with `attempt_in_flight` = true; after the result it is
  written with `attempt_in_flight` = false. At the start of each run, every
  lead with `attempt_in_flight` = true becomes `notify_pending` with
  `crash_recovered` and gets one M4; it is never sent to the agent again
  automatically (it is not eligible, §Definitions, so neither the probe nor
  a later run sends it). Effect: at most one automatic send per attempt
  record, and no lead is lost. If the send succeeded but the crash came
  before the record update, the owner receives an M4 for a lead the agent
  already has; the M4 body states `crash_recovered`.
- **E15 Write failure.** (a) A write to `notify-records.json` or the
  channel-state file fails → `state_write_failed`. If it was the
  `attempt_in_flight` = true write, that attempt is not made. No further
  sends for that tenant in this run (channel-state failure: no further sends
  in this run at all). One M8 per run. (b) A write to a tenant's MEMORY.md
  or to `notify-run.log` fails → the event is written to the process's
  standard error with the same ID-only fields (FR-012), processing
  continues, and one M8 with `log_write_failed` is sent per run.
- **E12 Transport auth failure** (`whatsapp_cloud_api` only, S2a). Invalid
  or missing credential → `auth_failed`; no retry; relay; channel `degraded`;
  one M5.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001** Delivery sends every agent and owner alert through
  `send_lead_alert` (§Interfaces). No other code path sends a lead alert.
- **FR-002** Owner config field `agent_transport` selects the agent
  transport for all tenants. Allowed values: `fake`, `openclaw_whatsapp_web`,
  `selfrun_whatsapp_web`, `whatsapp_cloud_api`. Absent → `fake`. Other → E4.
- **FR-003** While any of G-1, G-2, G-3 is null, Delivery uses `fake`
  regardless of `agent_transport` and logs `golive_gate_closed` once per run.
- **FR-004** The sending number is read only from `WHATSAPP_BUSINESS_NUMBER`.
  Unset or empty → the transport's `available()` returns false and every
  lead is relayed with `transport_unavailable`.
- **FR-005** A lead is sent to the agent only when E1 passes.
- **FR-006** Each attempt waits at most 15 s for a provider message id; none
  → `send_timeout`.
- **FR-007** Retry: `session_down`, `send_timeout` and `send_rejected` get
  exactly one retry 10 s after the first attempt ends (K6). `logged_out`,
  `auth_failed`, `quota_exceeded` and probes get none. Worst case per lead:
  15 + 10 + 15 = 40 s.
- **FR-008** Channel health (one shared sending number). Final outcome
  `session_down`, `logged_out`, `auth_failed` or `quota_exceeded` →
  `degraded` immediately. `send_timeout` or `send_rejected` →
  `consecutive_failures` + 1; at 2 → `degraded`. Any `delivered` → counter 0.
  Entering `degraded` sends one M5 and sets `last_degraded_alert_at`. While
  `degraded`, every eligible lead except the probe is relayed with
  `channel_degraded`.
- **FR-009** Start of each run, in order: (1) `available()` false → state
  `unavailable` (previous state kept in `state_before_unavailable`), leads
  relayed `transport_unavailable`; (2) `available()` true while
  `unavailable` → back to `state_before_unavailable`; (3) state `degraded` →
  one probe: the oldest eligible `notify_pending` lead of the first tenant in
  order that has one, sent once with no retry. Probe `delivered` → `active`,
  counter 0. Probe failure → stays `degraded`; M5 re-sent only when
  `last_degraded_alert_at` is ≥ 60 minutes ago. No eligible lead → no probe.
- **FR-010** Rate limits: ≥ 3 s between the end of one WhatsApp attempt and
  the start of the next; ≤ 10 lead sends per tenant per run (K7); ≤ 30 per
  run in total. Order within a tenant: URGENT (score ≥ 0.9) first, then
  `queued_at` ascending. A lead not reached keeps its state.
- **FR-011** Relay (M4). Every lead that becomes `notify_pending` gets one
  queued M4, sent at the end of its tenant's step; retries per E7. Once
  sent, no further M4 for that lead. If the lead becomes `delivered` or
  `notify_expired` before its M4 is sent, the M4 is dropped and
  `relay_dropped` is logged. `notify_blocked` leads get M8, never M4.
- **FR-012** Logs. Per-tenant events go to that tenant's MEMORY.md (K12);
  events with no valid tenant (E2) go to
  `~/.openclaw/workspace/notify-run.log`. Both contain only `run_id`,
  `tenant_id` (or `none`), `lead_id` (or `none`), `transport`, provider
  message id, `event`, `error_type`, `attempt`, and ISO 8601 UTC
  timestamps. Never names, phone numbers, email addresses, property details,
  locations, prices, or message text.
- **FR-013** Adapters never raise to Delivery. Every failure returns a
  `SendResult` with an `error_type` from the outcome table (K9 pattern).
- **FR-014** `openclaw_whatsapp_web.available()` is true only when both
  hold: (1) owner config `whatsapp_plugin_publisher_verified_at` is a valid
  ISO 8601 date; (2) the S4 health check returns `works` for the account
  named in owner config `whatsapp_account_id`. S4 exit code ≠ 0, no
  response within 10 s, a config-only summary (gateway unreachable), or any
  probe result other than `works` → false. Entering `unavailable` sends one
  M5 with `transport_unavailable`; repeated at most once per 60 minutes.
  `selfrun_whatsapp_web.available()`: see S1-T5.
- **FR-015** Delivery builds each agent alert in two forms: `text` (rendered
  M2) and `template` (`template_key` `lead_alert_v1` plus the ordered
  `params` list in §Interfaces). Free-form transports send `text`;
  template-based transports send `template`. Delivery never branches on the
  transport.
- **FR-016** Switching transport = changing `agent_transport` plus setting
  that transport's credential env-var names in owner config. Zero changes to
  Delivery code, lead states, outcome table, log fields or message
  contracts.

### Lead notify states

`awaiting_send` → `delivered` | `notify_pending` | `notify_blocked` | `notify_expired`;
`notify_pending` → `delivered` (probe or later send) | `notify_blocked` (E3, E4 found on a later run) | `notify_expired`;
`notify_pending` with `crash_recovered` → `notify_expired` only (E14);
`notify_blocked` → `awaiting_send` (E5) | `notify_expired` (E6).

### Interfaces

```text
Adapter (every transport and the relay):
  name: str
  available() -> bool
  send_lead_alert(alert: LeadAlert) -> SendResult        # never raises

LeadAlert = {
  recipient_kind: "agent" | "owner",
  tenant_id: str | null,   # agent, M4, M6: required
                           # null for: M5; M8 from E2, E4, E15(a) channel state, E15(b)
  lead_ids: [str (UUID)],  # agent and M4: exactly 1; M6: 1 or more;
                           # M8: 0 or more (0 for E4, E15, E2 without a lead_id); M5: empty
  recipient: str,          # agent: tenant agent_whatsapp; owner: value of OWNER_RELAY_EMAIL
  text: str,               # rendered M2 / M4 / M5 / M6 / M8
  template: { template_key: "lead_alert_v1",
              params: [urgent_flag ("1"|"0"), contact_name, property_type,
                       location, score ("0.00"–"1.00"), source, lead_id] } | null
                           # required for agent alerts; null for owner messages
}
SendResult = {
  status: "delivered" | "failed",
  provider_message_id: str | null,     # non-null iff status = "delivered"
  error_type: <outcome table value> | null
}
```

**External calls.** Commands and flags below were read on 2026-11-12 from
the installed OpenClaw 2026.6.11 docs (read-only). The docs do not show the
JSON output of `message send` or `channels status`; those field names are
**unverified** and are fixed by T-009 (`tasks.md`) against the installed
version before any build task. If a field cannot be found, F013 returns to
spec.

- **S1** (T1 send) Command, run without a shell, arguments as a list:
  `openclaw message send --channel whatsapp --account {whatsapp_account_id}
  --target {agent_whatsapp} --message {text} --json`
  (`docs/cli/message.md:24`, `:29` WhatsApp target = E.164, `:47-52`
  flags, `:69-71` send). Success = exit code 0 **and** a non-empty provider
  message id in the JSON output (OpenClaw itself treats a WhatsApp send as
  accepted only when Baileys returns an outbound message id,
  `docs/channels/whatsapp.md:711`). Exit code 0 without a message id →
  `send_rejected`.
- **S2** (T1 failure mapping) Non-zero exit code, or the process still
  running after 15 s (then terminated): output naming no active listener
  (`docs/channels/whatsapp.md:168`, `:703-704`) → `session_down`; output
  naming the account as not linked (`docs/channels/whatsapp.md:628-630`) →
  `logged_out`; output naming a rate limit → `quota_exceeded`; process still
  running at 15 s → `send_timeout`; anything else, including an unresolved
  credential reference (`docs/cli/message.md:63`, fails closed) →
  `send_rejected`. A WhatsApp Web session has no separate credential; loss
  of authorisation surfaces as `logged_out`, so T1 never raises
  `auth_failed`.
- **S1-T5** (T5 send and health) Defined only if ADR-005 G1 = Option B.
  F013 returns to spec to add the chosen library's call, using the same
  `SendResult` values and the S2 categories, before any T5 build task.
  Until then `selfrun_whatsapp_web` is not registered, so selecting it
  gives E4 `transport_invalid`.
- **S4** (T1 health, used by FR-014) Command:
  `openclaw channels status --channel whatsapp --probe --json --timeout 10000`
  (`docs/cli/channels.md:35`). On a reachable gateway it runs a per-account
  probe whose result includes `works` or `probe failed`; on an unreachable
  gateway it returns a config-only summary without probe results
  (`docs/cli/channels.md:40-44`).
- **S2a** `auth_failed` is raised only by `whatsapp_cloud_api` (and by its
  test fake in AT-17): credential env var unset, or the API rejects the
  credential.
- **S3** Gmail `users.messages.send` (POST
  `https://gmail.googleapis.com/gmail/v1/users/me/messages/send`, body
  `{ "raw": base64url RFC 2822 message }`, header `Authorization: Bearer
  {access_token}`) → success HTTP 200 `{ "id": string, "threadId": string }`.
  HTTP 401 → `relay_auth_failed`; HTTP 429 → `relay_rate_limited`; no
  response within 15 s → `relay_timeout`; any other non-2xx →
  `relay_failed`. All four follow E7.
- **S3a** (relay credential) Owner config `relay_sender_credential_env`
  holds the **name** of an env var whose value is the path to an OAuth
  token file with the `gmail.send` scope, the same form v1 uses
  (`~/openclaw-deal-scout/src/email_scheduler/auth.py:15`, `:18-20`). Env
  var unset or file missing → E8 `owner_channel_not_configured`. When the
  access token has expired, the adapter refreshes it once with POST
  `https://oauth2.googleapis.com/token` (`grant_type=refresh_token`):
  HTTP 200 → new access token, send continues; HTTP 400 with
  `"error": "invalid_grant"` → `relay_auth_failed`; any other failure →
  `relay_failed`. Google endpoint shapes are from Google's public API
  documentation, not re-read in this session (**unverified** here).

### Outcome table (every `error_type`)

| `error_type` | Raised by | Retry | Result |
|---|---|---|---|
| `session_down` | transport (S2) | 1, after 10 s | relay; `degraded` (M5 on entry) |
| `logged_out` | transport (S2) | none | relay; `degraded` (M5 on entry) |
| `auth_failed` | `whatsapp_cloud_api` only (S2a, E12) | none | relay; `degraded` (M5 on entry) |
| `quota_exceeded` | transport (E11) | none | relay; `degraded` (M5 on entry) |
| `send_timeout` | transport (FR-006) | 1, after 10 s | relay; counter + 1; at 2 → `degraded` (M5 on entry) |
| `send_rejected` | transport (S2, other) | 1, after 10 s | relay; counter + 1; at 2 → `degraded` (M5 on entry) |
| `not_opted_in` | Delivery (E1) | none | relay |
| `channel_degraded` | Delivery (FR-008) | none | relay |
| `transport_unavailable` | Delivery (FR-004, FR-009) | none | relay |
| `agent_number_invalid` | Delivery (E3) | none | `notify_blocked` + M8 |
| `transport_invalid` | Delivery (E4) | none | `notify_blocked` + M8 |
| `tenant_id_missing` / `tenant_not_configured` / `tenant_mismatch` | Delivery (E2) | none | run-log line + M8 |
| `relay_auth_failed` / `relay_rate_limited` / `relay_timeout` / `relay_failed` | relay adapter (S3), any owner message | 1, after 10 s | E7 |
| `owner_channel_not_configured` | relay adapter (E8) | none | E8 |
| `state_corrupt` | Delivery (E13) | none | E13 |
| `tenant_config_invalid` | Delivery (E2 d) | none | run-log line + M8 |
| `state_write_failed` | Delivery (E15 a) | none | E15 |
| `crash_recovered` | Delivery (E14) | none | relay |

**Log-only events** (not `error_type` values): `owner_alert_failed` (E7),
`relay_abandoned` (E7), `relay_dropped` (FR-011), `golive_gate_closed`
(FR-003), `owner_config_invalid` (E13), `log_write_failed` (E15 b).

"Relay" = `notify_pending` + one queued M4 (FR-011).

### Message contracts (exact text)

- **M2 (agent, Roman Urdu + English)**: the K5 line with E9 caps, a new line,
  then `Lead ID: {lead_id} — tafseel HubSpot mein / details in HubSpot`.
- **M4 (owner email)**: subject `DealClaw relay — {tenant_id} — {lead_id}`;
  body `Notify pending — tenant {tenant_id} — lead {lead_id} — {error_type}. Forward on WhatsApp: https://wa.me/{agent_digits}?text={encoded}`.
  `{agent_digits}` = `agent_whatsapp` without `+`, built at send time and
  never logged. `{encoded}` = RFC 3986 percent-encoding of
  `DealClaw lead {lead_id}: HubSpot mein dekhein / please check HubSpot.`
- **M5 (owner email)**: subject `DealClaw channel — {state}`; body
  `WhatsApp channel {state} — {error_type}. Leads go to owner relay until fixed.`
  (`{state}` = `degraded` or `unavailable`).
- **M6 (owner email)**: subject `DealClaw expired — {tenant_id}`; body
  `Notify expired (>24h) — tenant {tenant_id} — leads: {lead_id_list}`
- **M8 (owner email)**: subject `DealClaw blocked — {error_type}`; body
  `Notify blocked — {error_type} — tenant {tenant_id or "none"} — leads: {lead_id_list or "none"}`

M5, M6 and M8 go by email to `OWNER_RELAY_EMAIL`. Lists are comma-separated,
ascending.

### Key Entities

- **Tenant `USER.md`**: existing `agent_whatsapp`, `active` (K8); new
  `whatsapp_opt_in_confirmed_at` (ISO 8601 date or null; set by the owner by
  hand after the agent messages the DealClaw number first).
- **Lead notify records**
  `~/.openclaw/workspace/tenants/{tenant_id}/notify-records.json`: per lead
  `lead_id`, `tenant_id`, `notify_status`, `attempts`, `last_error_type`,
  `queued_at`, `delivered_at`, `provider_message_id`, `transport`,
  `relay_sent`, `relay_attempt_runs`, `attempt_in_flight`.
- **Owner config** `~/.openclaw/workspace/owner.json`: `agent_transport`;
  `whatsapp_plugin_publisher_verified_at`; `golive_discord_trace_at`,
  `golive_spec_approved_at`, `golive_client_confirmed_at`;
  `whatsapp_account_id` (OpenClaw account id, not a number);
  `relay_sender_credential_env` (env-var name only, S3a). No tenant data, no
  literal secrets, numbers or addresses.
- **Channel state** `~/.openclaw/workspace/whatsapp-channel-state.json`:
  `state` (`active` | `degraded` | `unavailable`), `state_before_unavailable`,
  `consecutive_failures`, `last_degraded_alert_at`. No tenant data.
- **Workspace run log** `~/.openclaw/workspace/notify-run.log` (E2 events only).

### Multi-tenant isolation

- `tenant_id` is the key of every record and log line.
- Processing tenant X reads and writes only
  `~/.openclaw/workspace/tenants/X/`, plus the shared owner config, channel
  state and run log, which hold no lead content, numbers or addresses.
- The probe reads only the processing tenant's records.
- USER.md fields that control per-tenant behaviour: `agent_whatsapp`
  (recipient), `whatsapp_opt_in_confirmed_at` (send allowed), `active`
  (tenant processed).
- Missing or mismatched `tenant_id` → E2 (gates A4, A8).

## Acceptance tests (written before implementation)

pytest with `fake` transport and relay adapters and a faked clock. No
network, no OpenClaw process, no real number or address. Upstream fixture:
`tests/fixtures/emails/zameen_alert_high_confidence.txt`. "Number N" is a
fixture value matching E3's pattern that is not a real number.

| ID | Input | Expected output | Path |
|---|---|---|---|
| AT-01 | Zameen lead, CRM confirmed, opted in, gate open, fake delivered | one M2 containing `lead_id` and the bilingual line; `delivered`; provider id stored | happy path, locale |
| AT-02 | opt-in date null | 0 agent sends; `notify_pending`; `not_opted_in`; one M4 email | E1 rejection |
| AT-03 | fake `session_down` ×2; 3 leads | lead 1: 2 attempts 10 s apart, relay; `degraded`; one M5; leads 2–3 `channel_degraded`, 0 sends | session drop |
| AT-04 | fake no id ×2 | 2 attempts; 40 s elapsed; relay; counter 1; `active` | timeout |
| AT-05 | `logged_out` on lead 1 | 1 attempt; relay; `degraded`; one M5 | ban / logout |
| AT-06 | `degraded`; probe delivered | `active`; counter 0; next lead sent same run | probe success |
| AT-07 | `degraded` at T; failed probes at T+30 min, T+61 min | M5 at T and T+61 min only | probe throttle |
| AT-08 | G-2 null, `agent_transport` = `openclaw_whatsapp_web` | `fake` used; `golive_gate_closed` logged once | FR-003 |
| AT-09 | `WHATSAPP_BUSINESS_NUMBER` unset | `available()` false; leads relayed `transport_unavailable` | FR-004 |
| AT-10 | number = template placeholder; next run number N | run 1 `notify_blocked`, one M8, no M4; run 2 `delivered` | E3 + E5 |
| AT-11 | 12 leads for one tenant, 2 URGENT | 10 sent, URGENT first; 2 keep `awaiting_send` | FR-010 |
| AT-12 | relay fake fails ×2 in runs 1, 2, 3 | M4 retried at start of runs 2, 3; then `relay_abandoned` | E7 |
| AT-13 | `OWNER_RELAY_EMAIL` unset | no attempt; `owner_channel_not_configured` once per run; `relay_attempt_runs` unchanged | E8 |
| AT-14 | switch `agent_transport` from fake-web to a fake registered as `whatsapp_cloud_api` that reads only `template` | same lead states, log fields and outcome; Delivery source unchanged (file hash compared) | FR-015, FR-016 |
| AT-15 | `agent_transport` = `sms` | `transport_invalid`; `notify_blocked`; one M8 | E4 |
| AT-16 | lead with other tenant's `tenant_id` | no send; no tenant record; one run-log line; one M8 | E2 |
| AT-17 | `agent_transport` = fake registered as `whatsapp_cloud_api`, returns `auth_failed` | 1 attempt; relay; `degraded`; one M5 | E12 |
| AT-18 | `quota_exceeded` | 1 attempt; relay; `degraded`; one M5 | E11 |
| AT-19 | 500-char location | M2 < 400 chars; location 60 chars ending `…`; `lead_id` line intact | E9 |
| AT-20 | pending lead aged 25 h | `notify_expired`; listed in one M6 | E6 |
| AT-21 | run AT-01–AT-20 and AT-23–AT-28; scan all logs and standard error | 0 fixture names, numbers, addresses, locations, prices or message text | FR-012 |
| AT-23 | lead with `attempt_in_flight` = true at run start | `notify_pending`; `crash_recovered`; one M4; 0 agent sends | E14 |
| AT-24 | `notify-records.json` invalid JSON for t1; t2 valid | t1: 0 sends, file unchanged, one M8; t2 processed normally | E13 (b) |
| AT-25 | `golive_spec_approved_at` = `not-a-date` | treated as null; `fake` used | E13 (f) |
| AT-26 | t1 `USER.md` not valid JSON; lead for t1 | no send; t1 records untouched; one run-log line; one M8 `tenant_config_invalid` | E2 (d) |
| AT-27 | write of `attempt_in_flight` = true fails for t1 | no attempt for t1 this run; `state_write_failed`; one M8; t2 processed | E15 (a) |
| AT-28 | opt-in date = `yesterday` | treated as null; `not_opted_in`; one M4 | E1 invalid date |
| AT-29 | S4 returns `probe failed` at run start | `unavailable`; leads relayed `transport_unavailable`; one M5 | FR-014 |
| AT-22 | repo grep for phone-number, email-address and token patterns in F013 files | 0 matches | secrets |

## Success Criteria *(mandatory)*

- **SC-001** AT-01 to AT-29 pass.
- **SC-002** AT-21 finds zero lead-content strings.
- **SC-003** After G-1 to G-3: one synthetic Zameen-format lead for the test
  tenant yields one WhatsApp message with a provider message id, recorded as
  an ID-only trace.
- **SC-004** A pilot agency counts toward the Phase 1 gate when its first
  lead reaches `delivered`.

## Dependencies

- D1 Credential rotation confirmed by the owner (ADR-005 prerequisites).
- D2 ADR-005 G1 decided (open); it selects T1 or T5 (OD-1a).
- D3 Plugin version path (K10), publisher verification, install approval.
- D4 Dedicated number linked by QR on a phone the owner controls.
- D5 Go-live gate G-1 to G-3.

## Risks

- **RISK-1 Ban of the sending number (accepted by the owner, OD-5).** The
  pilot uses an unofficial WhatsApp Web client, not an official business
  API (K10). WhatsApp can restrict or permanently ban a number that uses an
  unofficial client; the likelihood, the trigger thresholds and whether a
  banned number can be recovered are **unverified** (no ban guidance in the
  OpenClaw docs, PHR 0038). Effect: every send fails with `logged_out` or
  `send_rejected`. Mitigations in this spec: a dedicated number, never the
  owner's personal number (OD-4); messages only to agents who messaged the
  number first (E1); ≤ 10 per tenant and ≤ 30 per run with ≥ 3 s spacing
  (FR-010); no marketing or bulk content (M2 only); immediate degradation
  and owner relay (FR-008, OD-3), so no lead is lost; the paid official API
  replaces this transport at the first paying client (OD-2).
- **RISK-2 Session drop on the owner's WSL machine.** Sleep, reboot or
  logout drops the session. FR-008 degrades at the end of the first lead
  whose final outcome is `session_down` (at most 40 s, FR-007); M5 tells the owner to re-link by QR.
- **RISK-3 Paid switch conflicts with the zero-cost rule (K2).** The
  OD-2 switch needs gate I2 approval and an owner decision on Section 5.

## Owner actions outside this spec

F013 behaviour depends only on the recorded dates in owner config (G-1 to
G-3, FR-003) and on OD-7 and OD-8. The items below are owner actions tracked
in `tasks.md` Phase 0. None of them changes any requirement in this spec.

- **A-1** Produce and record the G-1 Discord trace. The owner chooses a run
  that respects Hard Rule 8 / gate I3 and the Phase 1 gate (ADR-005
  Amendment 1 lists the conflicts).
- **A-2** Obtain the pilot client's written confirmation (G-3) of OD-7 and
  that the agent will message the DealClaw number first.

## Later (not in F013)

- L3 One sending number per tenant.
- L4 The `whatsapp_cloud_api` adapter module and its template approval (OD-2).
- L5 Inbound `DONE <lead_id>` reply and automated opt-in (F013b; K14 kept).
- L6 Discord as a second owner relay channel.
- L7 More than one agent per tenant, or a WhatsApp group (OD-7).

## Out of scope

- Any inbound WhatsApp handling (F013b).
- Discord, email or Telegram to the agent (K1).
- UK market.
- Installing, connecting or configuring any transport.

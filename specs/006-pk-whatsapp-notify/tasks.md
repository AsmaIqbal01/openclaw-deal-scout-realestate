---
description: "Task list for F013 — PK WhatsApp lead alert adapter + owner relay"
---

# Tasks: PK WhatsApp Lead Alert Adapter + Owner Relay (F013)

**Input**: `specs/006-pk-whatsapp-notify/spec.md`, `plan.md`
**Status**: DRAFT, BLOCKED. No build task starts until `spec.md` scores
≥ 9.6 and the owner approves it (`CONSTITUTION.md:81`). Tasks marked
**[OWNER]** are decisions or real-world steps for the owner or the pilot
client, not code. Already decided on 2026-11-12 (spec OD-1 to OD-8): pilot
transport T1, alternate T5, Cloud API at the first paying client, owner
relay by email, one recipient per tenant, OD-6 ruling.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: can run in parallel (different files, no open dependency)
- **[Story]**: US1–US4 from `spec.md`

## Phase 0: Gates and decisions (blocking)

- [ ] T-001 Re-score `spec.md` with spec-scorer; owner approves at ≥ 9.6 and records `golive_spec_approved_at` (G-2).
- [ ] T-002 [OWNER] Decide ADR-005 G1. A or C → T1 stays; B → spec returns to add S1-T5.
- [ ] T-003 [OWNER] Resolve the plugin/core version path (spec K10), verify the publisher, approve the install; record `whatsapp_plugin_publisher_verified_at`.
- [ ] T-004 [OWNER] Link the dedicated number by QR on a phone the owner controls; record `whatsapp_account_id` (not the number) in owner config.
- [ ] T-005 [OWNER] Produce the G-1 Discord trace in a run that respects Hard Rule 8 / gate I3 and the Phase 1 gate (spec A-1); record `golive_discord_trace_at`.
- [ ] T-006 [OWNER] Get Inhabitant Construction's written confirmation of OD-7 (one agent, `agent_whatsapp`) and that the agent will message the DealClaw number first (spec A-2); record `golive_client_confirmed_at`.
- [ ] T-007 [OWNER] Set env vars `WHATSAPP_BUSINESS_NUMBER`, `OWNER_RELAY_EMAIL` and the relay token-path env var named in `relay_sender_credential_env`. Values never go into the repo.
- [ ] T-008 Write the ADR for the transport and interface (`/sp.adr pk-lead-alert-transport`), with owner consent.
- [ ] T-009 Read-only, with owner approval: fill plan.md "Field mapping to fix before build" (S1 message-id field, S2 error texts, S4 probe field, S3/S3a shapes). Any row that cannot be filled → back to spec.

## Phase 1: Tests first (fakes only; all must fail before Phase 2)

- [ ] T-010 [P] Contract: `send_lead_alert` never raises; every `error_type` is in the outcome table; `LeadAlert` null rules (FR-013, spec §Interfaces).
- [ ] T-011 [P] [US1] AT-01 happy path and locale; AT-11 caps and URGENT order.
- [ ] T-012 [P] [US2] AT-02 not opted in; AT-12 relay retry and abandon; AT-13 relay not configured; AT-28 invalid opt-in date.
- [ ] T-013 [P] [US3] AT-14 config-only switch to a fake `whatsapp_cloud_api` (Delivery file hash unchanged); AT-17 `auth_failed`.
- [ ] T-014 [P] [US4] AT-03 session drop; AT-04 timeout; AT-05 logged out; AT-06 probe; AT-07 M5 throttle; AT-18 quota; AT-29 health check fails.
- [ ] T-015 [P] AT-08 go-live gate closed; AT-25 invalid gate date; AT-09 `WHATSAPP_BUSINESS_NUMBER` unset (env cleared in the test; value never read or printed).
- [ ] T-016 [P] AT-10 invalid number + recovery; AT-15 invalid transport; AT-16 tenant mismatch; AT-26 corrupt `USER.md`.
- [ ] T-017 [P] AT-19 message length; AT-20 expiry; AT-23 crash recovery; AT-24 corrupt records; AT-27 write failure.
- [ ] T-018 [P] AT-21 ID-only scan of logs and standard error; AT-22 grep of F013 files for phone-number, email-address and token patterns.
- [ ] T-019 [P] T1 adapter unit tests with a faked subprocess runner: exit 0 + id → delivered; exit 0 without id → `send_rejected`; S2 text cases; 15 s timeout → `send_timeout`; S4 `works` / `probe failed` / config-only.
- [ ] T-020 [P] Relay adapter unit tests with a faked HTTP opener: 200; 401; 429; timeout; token refresh 200 and `invalid_grant`.

## Phase 2: Core (stdlib only)

- [ ] T-021 `contract.py`, `registry.py`, `fake.py`.
- [ ] T-022 `messages.py`: M2/M4/M5/M6/M8 with subjects, E9 truncation, `wa.me` encoding, template params.
- [ ] T-023 `state.py` (E13, E15) and `log.py` (allow-list, stderr fallback).
- [ ] T-024 `delivery.py`: gate, opt-in, retries, degrade/probe/unavailable, caps, relay, expiry, crash recovery; assert a worst-case run fits the 15-minute heartbeat with the faked clock.

## Phase 3: Real adapters (after T-002, T-003, T-009)

- [ ] T-025 `openclaw_web.py` (S1, S2, S4) using the fields fixed in T-009.
- [ ] T-026 `gmail_relay.py` (S3, S3a).
- [ ] T-027 Update `agents/delivery/SOUL.md` Step 3 to call `send_lead_alert` (one change; Constitution Checker run).

## Phase 4: Go-live (owner present; nothing automatic)

- [ ] T-028 [OWNER] Confirm G-1, G-2, G-3 dates are recorded; set `agent_transport` = `openclaw_whatsapp_web`.
- [ ] T-029 [OWNER] One synthetic Zameen-format lead for the test tenant; capture the ID-only trace (SC-003).
- [ ] T-030 Constitution Checker PASS; full suite passes; README updated (B2).

## Dependencies

Phase 0 → Phase 1 → Phase 2 → Phase 3 → Phase 4. Phases 1–2 need only
T-001 (spec approval). T-025 needs T-002, T-003, T-009. T-028 needs T-004
to T-007.

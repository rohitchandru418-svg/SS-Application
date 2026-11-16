# Startup Singam – OpenHands build pack

This pack tells OpenHands how to build the complete Startup Singam application end to end. It was derived from `Startup Singam v1-2.xlsx` (11 sheets: Founder journey, Investor journey, SR, IR, Finance, Production, Legal, Marketing, Community, Analyst, Super admin). The spreadsheet is the source of truth; where it is silent or contradicts itself, the decision taken is written down in [`docs/99-assumptions-and-gaps.md`](docs/99-assumptions-and-gaps.md) with an ID like `A-07`, and code comments should reference that ID.

UI colours, typography and the design system are **out of scope**. The pack defines layout, screens, states and behaviour only, and reserves a single integration point for your design system (see `docs/11-ux-and-screens.md` § Design-system slot).

## How to use it with OpenHands

1. Create an empty Git repository and open it in OpenHands.
2. Copy this whole folder into the repository root as `/spec` (keep the folder structure).
3. Copy `AGENTS.md` to the repository root **and** to `.openhands/microagents/repo.md`, so OpenHands loads the standing rules on every task.
4. Run the phases below **in order**, one OpenHands conversation per phase. Each phase file ends with a "Prompt to paste" block; paste that block as the first message. Do not start a phase until the previous phase's acceptance checks pass.
5. After each phase, review the PR / diff, run `pnpm test` and `pnpm e2e`, then merge.

If you want to put your design system in early, do it right after P00 by following the Design-system slot instructions; every later phase only consumes components through that slot.

## Read-first documents (OpenHands reads these in every phase)

| File | What it holds |
|---|---|
| `AGENTS.md` | Standing engineering rules, stack, conventions, definition of done |
| `docs/01-product-and-personas.md` | Product summary, every persona (external, internal, no-login), what each one is for |
| `docs/02-founder-journey.md` | Founder stages 0–19, mapped to internal stages, with screen-level detail |
| `docs/03-investor-journey.md` | Investor stages 0–15, mapped to internal stages, with screen-level detail |
| `docs/04-internal-workflows.md` | SR, Analyst, Finance, Legal, IR, Production, Marketing, Community, Super admin step by step |
| `docs/05-end-to-end-flow.md` | The whole lifecycle in one sequence, all hand-offs, the parallel tracks, and the **sync matrix** (internal action → what the founder / investor sees, and the reverse) |
| `docs/06-domain-model.md` | Entities, fields, relationships (one shared database) |
| `docs/07-state-machines.md` | Every status, allowed transition, who can trigger it, and its automatic side effects |
| `docs/08-permissions-and-visibility.md` | Roles, record scoping, field-level visibility, redaction, external label mapping |
| `docs/09-architecture-and-shared-data.md` | Architecture, shared data source, domain events, realtime propagation, audit |
| `docs/10-automation-deadlines-notifications.md` | Clocks, working-day calendar, automatic transitions, escalations, notification catalogue |
| `docs/11-ux-and-screens.md` | UX principles, every screen per persona, design-system slot |
| `docs/12-dashboards-and-metrics.md` | Every dashboard and metric definition |
| `docs/13-acceptance-scenarios.md` | Cross-persona end-to-end test scenarios |
| `docs/99-assumptions-and-gaps.md` | Every assumption and open question |

## Build order (dependency order)

| Phase | File | Builds | Depends on |
|---|---|---|---|
| P00 | `phases/P00-foundation.md` | Monorepo, auth, users/roles, permission engine, audit, event outbox, realtime, notifications, files, working-day calendar, three app shells, design-system slot | — |
| P01 | `phases/P01-admin-seasons-policies-approvals.md` | Super admin: seasons, versioned policies, legal templates, users & roles, generic approvals inbox, escalations, reassignment | P00 |
| P02 | `phases/P02-founder-intake-and-screening.md` | Founder registration, profile, application draft/submit, RM assignment, duplicate check, 100-mark screening, information-required loop | P01 |
| P03 | `phases/P03-bootcamp-and-analyst.md` | Bootcamp batches, preparation, analyst pool, redaction, analyst evaluation, SR review | P02 |
| P04 | `phases/P04-mou-and-legal-qa.md` | MoU share, working-day clock, Legal Q&A, sign / negotiate / discontinue, MoU v2 | P03 |
| P05 | `phases/P05-irr.md` | IRR request, document queries to founder, versions, review, super admin approval, handover to IR | P04 |
| P06 | `phases/P06-investor-onboarding.md` | Investor interest, profile, agreements, verification, eligibility, grading, season allocation, season meter | P01 (can run in parallel with P02–P05) |
| P07 | `phases/P07-matching-and-pitches.md` | Matching engine, pairings, IRR viewer, virtual pitch, verdicts, Pitch Day, Golden Pass | P05, P06 |
| P08 | `phases/P08-committee-and-upfront-fee.md` | Committee pack and decision, automatic triggers, upfront fee end to end, Zoho Books adapter | P07 |
| P09 | `phases/P09-production-and-shoots.md` | Advance notice, handover pack, AV shoot, episodes, pre-shoot panel, mock shoot, main shoot, exceptions, editing approval | P08 |
| P10 | `phases/P10-deal-closure.md` | Deal room per investor, communication thread, term sheet, due diligence, agreements, pre-funding conditions, revisions, withdrawals | P09 |
| P11 | `phases/P11-deployment-success-fee-reconciliation.md` | Tranches, verification, reversals, success fee, TDS, reconciliation | P10 |
| P12 | `phases/P12-marketing-and-telecast.md` | Telecast schedule, promo assets, founder posting, feedback loop, public announcements | P09 (announcements need P11) |
| P13 | `phases/P13-community.md` | Member records, events, referrals, founder support requests | P02, P06 |
| P14 | `phases/P14-dashboards-360-hardening.md` | All dashboards, Startup 360, Investor 360, Management dashboard, seed data, full e2e, security and performance pass | all |

```
P00 ─ P01 ─┬─ P02 ─ P03 ─ P04 ─ P05 ─┐
           │                          ├─ P07 ─ P08 ─ P09 ─ P10 ─ P11 ─┐
           └─ P06 ────────────────────┘                 └─ P12 ───────┤
                 P13 (after P02 + P06) ───────────────────────────────┴─ P14
```

Every phase delivers backend, internal console, founder portal and investor portal changes **together** for its slice, so the shared-data behaviour is tested as it is built, not bolted on at the end.

## Gaps for you to answer

The open questions that change behaviour are listed at the top of `docs/99-assumptions-and-gaps.md` under "Decisions needed from the business". The build can proceed on the stated assumptions; each is isolated behind a policy or config value so it can be changed without code rewrites where possible.

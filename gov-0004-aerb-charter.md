# GOV-0004: Architecture & Engineering Review Board (AERB) Charter

| Field | Value |
|---|---|
| **Status** | In Effect |
| **Effective Date** | 2026-08-17 |
| **Established By** | ADR-0001 (AERB as review body) [recommend a dedicated founding ADR; see Notes] |
| **Owner** | AERB (chair maintains) |
| **Kind** | Charter |
| **Review Cadence** | Annually |

## Purpose

Defines who the Architecture & Engineering Review Board (AERB) is, what it may decide, and where its authority ends. The TDs assign the AERB duties (cross-cutting ADR review, renewal-count-2 adjudication, challenge decisions, boundary-change review); this charter is where those assignments resolve to named seats and rules.

## Membership

| Seat | Held by | Basis |
|---|---|---|
| Chair | Technical Director | Ex officio |
| Portfolio seats | One principal engineer per portfolio [names TBD] | Appointed by portfolio lead, confirmed by chair |
| Platform seat | One representative across Nitro/Platforma [name TBD] | Appointed by chair |
| Experience seat | Experience design engineering lead [name TBD] | Ex officio |
| Security seat | Security engineering lead [name TBD] | Ex officio |
| Governance Steward | Current rotation holder | Ex officio, non-voting; operates the agenda |

Membership changes are direct edits by the chair. Seat structure changes require an ADR.

## Decision Rights

The AERB decides:
- Acceptance of cross-cutting ADRs (more than one portfolio, or any capability platform)
- Bounded context boundary changes (TD-0003)
- Challenges to TDs and ADRs, per the Challenge Path
- Renewal-count-2 exception adjudication
- Material STD changes flagged under standard change control

The AERB does not decide:
- TD issuance, rescission, and suspension of obligations: **Technical Director**
- Exception approvals: **per the authority named in each TD** (see matrix below)
- Portfolio-internal ADRs: **normal team review**
- Anything requiring resource commitment: recommendation to the Technical Director only

## Approval Authority Matrix

| Action | Authority |
|---|---|
| Cross-cutting ADR acceptance | AERB |
| Portfolio-internal ADR acceptance | Owning team review |
| TD issuance / revision / rescission / suspension | Technical Director |
| Exception to TD-0001, TD-0005, TD-0006 | Technical Director |
| Exception to TD-0002, TD-0003 | AERB |
| Exception to TD-0004 (deferred validation) | Portfolio lead |
| Exception to TD-0007 | Experience design engineering + Technical Director |
| Exception to TD-0008 | Security engineering + Technical Director |
| Material STD change | Flag to AERB or quarterly review before effect |
| Challenge decisions | AERB (Technical Director may expedite) |

## Operating Rules

- **Quorum:** chair (or delegate) plus half the voting seats.
- **Decision mode:** consent-based; the chair calls the decision when no seated objection remains unresolved. Unresolved objections escalate to the Technical Director, whose decision is recorded in the relevant ADR.
- **Rhythm:** the AERB convenes within the Quarterly Governance Health Review; cross-cutting ADR review between quarters happens asynchronously in the pull request, with any seat able to pull a review into a synchronous session.
- **Recusal:** a seat recuses from decisions where their team is the requesting party; the chair may appoint a temporary substitute.

## Challenge Log

Held here until volume justifies a standalone register (see GOV-0009 draft).

| Challenge ID | Contested | Submitted By | Received | Decision | Decided |
|---|---|---|---|---|---|
| *(none yet)* | | | | | |

Rules: every challenge receives a decision by the next quarterly review or sooner if expedited; decisions are recorded as ADRs (revision, supersession, or reaffirmation); a twice-reaffirmed rule requires materially new evidence for further challenge.

## Roles

Covered by the membership and decision rights tables above.

## Change Discipline

- **Keeping it true:** roster updates and log entries are direct edits by the chair and Steward.
- **Changing how the machinery works:** seat structure, decision rights, quorum, and decision mode require an ADR.

## Health

Decision latency: challenges and cross-cutting reviews resolved within one cycle; older items are findings. Escalation rate to the Technical Director sampled annually; an AERB that escalates everything or nothing warrants a design look.

## Notes

The bracketed membership placeholders require the Technical Director's appointments. This charter currently leans on ADR-0001 for establishment; a dedicated founding ADR for the governance system (the proposed ADR-0009) would be its cleaner root.

## Revision History

| Date | Change | Author |
|---|---|---|
| 2026-08-17 | Initial issuance with placeholder membership | Technical Director |

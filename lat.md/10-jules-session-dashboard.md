# Jules Session Dashboard

This file tracks active and important Jules sessions so the orchestrator can monitor them without relying on the web UI.

## Active Sessions

| Session ID | Task | Scope | State | PR | Last Checked | Next Action |
| --- | --- | --- | --- | --- | --- | --- |
| `15060948750619103072` | Add provider login capability labels | Config-driven Gmail capability metadata and UI labels, no OAuth/proxy changes | `IN_PROGRESS` | None yet | 2026-06-23 11:20 UTC | Wait, then review PR diff when created. |

## Open PRs

| PR | Title | State | Branch | Notes | Next Action |
| --- | --- | --- | --- | --- | --- |
| `#1` | Add implementation plan for Google Gemini provider | Open draft | `google-gemini-plan-2815913142746213638` | Based on older `main`; plan needs update against current docs and architecture before merge. | Review and update after current implementation PRs settle. |
| `#2` | feat: Russian UI Localization | Open draft | `feat/ru-i18n-6183751859949730638` | Updated with latest `main`; PR diff contains only i18n/type changes and no dashboard deletion. `check:source-artifacts` and locale key parity passed locally. | Ready for final review/merge after optional build with installed deps. |

## Completed / Canceled Sessions

| Session ID | Task | Outcome |
| --- | --- | --- |
| `6183751859949730638` | Add Russian localization | Completed; created draft PR `#2`, updated to include latest `main`. |
| `2815913142746213638` | Research Gemini Code Assist integration | Completed; created draft PR `#1`. |
| `2984572509670563777` | Research Antigravity provider integration | Completed; activity feed unavailable through API. |
| `13015313924206141005` | Plan Russian localization | Completed; activity feed unavailable through API. |
| `11939979990376381143` | Audit Gmail-capable provider login flows | Canceled as stuck. |
| `13185114484477059653` | Audit current provider login capabilities | Completed; activity feed unavailable, local audit captured in `lat.md/08-provider-login-capability-audit.md`. |

## Monitoring Rules

- Reuse existing Jules sessions with `manage_session` for clarifications whenever possible.
- Do not create duplicate Jules sessions for the same scope unless the prior session is failed, canceled, or needs independent review.
- Treat long `IN_PROGRESS` without PR or activity as suspicious after 15-20 minutes; check status, then either send a follow-up or cancel.
- Every Jules PR must receive orchestrator diff review, at least one independent quick review when practical, and targeted validation before merge.

# Projects

Live board of what is being built here. See CLAUDE.md for how to keep it.

## Active

| Project | State | Waiting on | Updated |
|---|---|---|---|
| Fork hardening (review fixes) | Not started: running as is for now | Operator to say go | 2026-10-24 |

## Fork hardening: review backlog (2026-10-24)

Review of the fork as cloned from Bennettxai/FounderOS at `fb974bf`.
Typecheck clean, 4760 tests pass. Findings below, most urgent first.

### Fix first
| # | Issue | Where | Fix |
|---|---|---|---|
| 1 | Filming toggle does not switch the SQLite handle until restart; real rows land in `demo-data/` and are wiped by the weekly rebuild | `lib/data.ts:13,32` | Cache the handle per resolved path; reopen on change |
| 2 | DNS rebinding reads every GET on a no-password install via client-set `X-Forwarded-Host` | `lib/auth-gate.ts:69`, `middleware.ts:67` | Judge loopback on raw `Host`; trust `x-forwarded-*` only with `GLADOS_TRUST_PROXY=1` |
| 3 | Next 14.2.35 has 1 critical + several high advisories | `package.json` | Migrate to Next 15.5+; `npm audit fix` for the rest |

### Security, medium
| # | Issue | Where | Fix |
|---|---|---|---|
| 4 | Matcher skips `/agents/*`, so `/agents/setup/[id]` renders without a session | `middleware.ts:108` | Exclude only `agents/*.{png,jpg,webp,svg}` |
| 5 | Post-login `?next=` accepts `javascript:` and `//evil.com` | `components/LoginForm.tsx:31` | Allow only `^/(?!/)` without `\` |
| 6 | Login throttle keyed on `X-Forwarded-For`; map grows unbounded | `lib/auth.ts:137` | Key on socket address; cap map; global limit |
| 7 | Agent html/svg deliverables served inline same-origin | `app/api/board/deliverables/route.ts:70` | Force attachment or CSP sandbox; add nosniff |
| 8 | Session is one static secret, non-Secure cookie, logout does not revoke | `lib/auth.ts:126` | Per-login session ids; Secure on https |
| 9 | ManyChat webhook `!==` compare; open GET | `app/api/webhooks/manychat/route.ts:28,43` | `sameSecret`, `resolveCred`, gate GET |
| 10 | gbrain query option injection; workflow canvas innerHTML unescaped | `lib/connectors/gbrain.ts:232`, `components/workflow-canvas/engine.ts:308` | Insert `--`; `esc()` the values |

### Wrong numbers / honest-or-absent
| # | Issue | Where |
|---|---|---|
| 11 | Stripe totals mix currencies and label them USD; balance reads `[0]` only | `lib/finances.ts:176`, `app/finances/page.tsx:47` |
| 12 | Failed or capped FanBasis / Stripe / Wise pulls show partial sums as real | `lib/connectors/payments.ts:78,132,169` |
| 13 | GHL, Meta Ads, Foreplay, PayPal report `connected` on key presence only | `lib/connectors/ghl.ts:22`, `meta-ads.ts:23`, `foreplay.ts:28`, `payments.ts:241` |
| 14 | Paperclip reports connected on any Vercel/Railway deploy, even in real mode | `founderos/lib/connectors/demo-status.ts:10`, `founderos/lib/gate.ts:14` |
| 15 | Board shows connected with empty lanes when the issues read fails | `app/api/board/live/route.ts:28` |
| 16 | CSV "Debit Amount"/"Credit Amount" columns read backwards | `lib/statements.ts:101` |
| 17 | Identical same-day transactions deduped into one | `lib/ledger.ts:56` |
| 18 | Net = this month's income minus another month's expenses | `app/finances/page.tsx:77,114` |
| 19 | Two accounts with the same business name overwrite each other in the chart | `lib/bank-statements.ts:56` |
| 20 | Cron loop posts to 127.0.0.1, so nothing runs when bound to `GLADOS_HOST` | `lib/cron-loop.ts:31` |
| 21 | Cron `*/n` on day/month fields, DOM/DOW AND, and DOW 7 differ from standard cron | `lib/cron-scheduler.ts:37,55` |

### Fork cleanup
- Upstream branding: `lib/appearance.ts:41` "Bennett Legacy", `components/AppearancePicker.tsx:106`, founderos.sh links in README, upstream agent roster seeded by `lib/seed.ts:59`.
- Remove unused `nodemailer` (has high advisories): `package.json`, `next.config.mjs:38`.
- Raw hex/rgba in 13 UI files (e.g. `components/KnowledgeGraph.tsx:42`); generalise `tests/adpilot-literals.test.ts`.
- Missing docs referenced by code: `docs/projects/*`, `docs/DESIGN-CONSOLE.md`, `scripts/publish-oss.ts`; tests that skip on them.
- CI has no `npm run build`; align Node (`.node-version` 24 vs CI 22, add `engines`).
- Move demo files (`lib/finance-demo.ts`, `founderos/lib/demo-board.ts`, `lib/creative-engine/demo.ts`) under `lib/demo/`.

## Checklist
- [x] Clone, install, verify the app runs and tests pass
- [ ] !Operator: install locally, set a password in Settings → Security
- [ ] !Operator: say go on fork hardening
- [ ] Fix #1 DB handle on mode switch
- [ ] Fix #2 DNS rebinding gate
- [ ] Fix #4 #5 open agents route and login redirect
- [ ] Remove nodemailer
- [ ] Money-number fixes (#11 to #19)
- [ ] Next 15 migration
- [ ] Rebrand the fork

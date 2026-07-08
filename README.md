# Jelasin *(proposed working name)*

> **Jelasin it — turn what you want to say into a visual that explains it.**
> An Indonesian-first AI infographic generator. Text in → professional infographic out.

*"Jelasin"* = Indonesian for **"explain it."** The name is the job-to-be-done, as a verb.

**Alternate names on the table:** `Grafisin` · `Terangin` · `Paparin` · `Visualin`
*(Verify domain + trademark availability before committing to any of these.)*

---

## Status

| | |
|---|---|
| **Current phase** | P0 · Discovery — ✅ complete |
| **Next phase** | P1 · PRD + AppFlow |
| **Goal** | Learn the 8-phase dev framework *and* ship a real product |
| **Builder** | Solo |
| **Timeline** | Engine spike (3d) → v1 slice (~2wk) → full clone (~6wk) — *gated on spike* |

---

## What it is

People who explain for a living — teachers, business professionals, content creators — need infographics but can't design them. **Jelasin** turns their text into a professional infographic, Indonesian-first.

The three audiences share **one job**, not three: *"make a visual that strengthens how I explain / communicate to my audience."* The roles differ only by style, not by need.

## Wedge / why it exists

- **Indonesian-first** — UI *and* generated infographic text default to Bahasa Indonesia.
- **Local payment rails first** — QRIS / e-wallets (GoPay, OVO, DANA); international later.

> ⚠️ **Competitive note:** [kakak.ai](https://kakak.ai) already occupies "Indonesian-first infographics." As of P0, Indonesian-first alone is **not a sufficient wedge** for shipping. Positioning to be sharpened in P1 (candidate angles: local payments, self-host, output formats, pricing).

## Reference / inspiration

- **[buildmygraphic.com](https://buildmygraphic.com/)** — the app being learned from, by **Academind GmbH** (a developer-education company). This project is an independent **learning reimplementation**, not affiliated with or endorsed by Academind.

## Competitor landscape (to confront in P1)

| App | Angle |
|---|---|
| [kakak.ai](https://kakak.ai) | Indonesian-first, academic/professional focus — **closest competitor** |
| [media.io](https://www.media.io/id/image-effects/ai-infographic-generator.html) | Indonesian AI infographic generator (nano-banana-pro) |
| [buildmygraphic.com](https://buildmygraphic.com/) | English-first reference app |
| Piktochart / Gamma / Canva | Global, editor-based, English-first |

---

## Development framework

Built inside an **8-phase, phase-aware framework** (P0–P7), with the Build Loop (P7) recurring **13×**, one ticket per build phase.

| Phase | Deliverable | Status |
|---|---|---|
| P0 · Discovery | `notes.md` | ✅ Done |
| P1 · PRD + AppFlow | `PRD.md`, `AppFlow.md` | ⬜ Next |
| P2 · TechStack | `TechStack.md`, `Lock.md` | ⬜ |
| P3 · Content | `Guidelines.md`, `mockup.html` | ⬜ |
| P4 · Schema | `Schema.md`, `*.sql` | ⬜ |
| P5 · Impl Plan | `ImplPlan.md` | ⬜ |
| P6 · Setup | `CLAUDE.md`, `.npmrc`, `package.json`, `.github/` | ⬜ |
| P7 · Build Loop ×13 | `src/*`, `*.test.ts`, `STATE.md`, git tags | ⬜ |

## Roadmap

- **v1 (thin slice):** text input → 1 style → 1 aspect ratio → Indonesian output → Google auth → save/view. *No payments, single input, single style.* Purpose: prove the engine renders Indonesian.
- **Full clone (destination):** all inputs (text/PDF/URL/image + research-from-question), style library, brand customization, aspect ratios, UHD export, credits + local payments, generation history.

## Deferred decisions

| Decision | Resolved in |
|---|---|
| Generation engine (image-model vs structured render) | P2 |
| Tech stack | P2 |
| Payment provider (QRIS / e-wallet) | P6 |

## Make-or-break assumption

> **The chosen AI engine can render legible, correctly-laid-out Indonesian text inside an infographic layout.**
> Validated by a 3-day throwaway engine spike before v1. If it fails, the timeline resets.

---

## Reference links

- **Design discussion (this chat):** `<PASTE_THIS_CONVERSATION_URL_HERE>`
  *(Copy the URL from your browser's address bar — I can't generate it for you.)*
- **Discovery brief:** [`notes.md`](./notes.md)
- **Reference app:** https://buildmygraphic.com/

## Workflow

- **Build:** agentic tooling; **Jira ticket triggers the P7 build flow** (one ticket per build phase).
- **Pair-programmer:** Claude, across all 8 phases.

---

*This is a solo learning project. "Full clone" is the destination, not the v1 target.*

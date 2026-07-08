# notes.md — Discovery Brief

**Project:** BuildMyGraphic clone (working name TBD)
**Phase:** P0 · Discovery — Product Strategist
**Status:** ✅ Approved — ready to gate into P1
**Date:** 2026-07-08

---

## 1. Problem statement

People who explain for a living — teachers, business professionals, content creators — regularly need infographics to make their message land, but they can't (or don't have time to) design them. Existing AI infographic tools (e.g. buildmygraphic.com by Academind) solve the design problem, but they are **English-first** and **built on international payment rails**, leaving the Indonesian market underserved on both language and access.

## 2. The job-to-be-done

There is **one job**, not three audiences:

> *"Turn what I want to say into a visual that strengthens how I explain / communicate it to my audience."*

Educators (→ students), business professionals (→ clients), and content creators (→ audiences) are **style variations of the same job**, not distinct products. This unification is deliberate — it keeps v1 focused instead of building three tools in a trenchcoat.

## 3. Goal (dual)

1. **Learn** — the 8-phase development framework *and* the modern AI-app stack (React/Next/TypeScript/Tailwind/serverless/LLM orchestration), by shipping.
2. **Ship** — a usable product real Indonesian users can generate infographics with.

Neither goal is subordinate; the project is a learning vehicle *and* a real product.

## 4. Differentiator / why it exists (Rule 14)

**Wedge:** *Indonesian-first visual explanation, at a local access point.*

- **Language:** UI **and generated infographic text** are Indonesian-first (bilingual, ID default).
- **Payments:** local rails first (QRIS / e-wallets — GoPay/OVO/DANA), international later.

This is a defensible reason to exist alongside the $5 original: the original does not serve Indonesian-language generation or local payment natively.

## 5. Full-clone vision (feature inventory — the destination)

This is the **destination**, not the v1 target:

- Inputs: raw text, PDF, URL, image, **and** "research from a question" mode
- Style library (multiple templates)
- Brand customization: logo, color palette, freeform instructions, insert own images
- Aspect ratios: 16:9 / 9:16 / 1:1
- UHD / high-res export
- Credits ledger + one-time credit-pack payments
- Google auth
- Generation history + storage/CDN layer

## 6. v1 thin slice (what ships first)

Deliberately tiny — its **only** purpose is to prove the generation engine works for Indonesian:

**In scope:**
- Text input → **one** style → **one** aspect ratio → **Indonesian** output
- Google auth
- Save / view result

**Explicitly OUT of v1:**
- ❌ Payments / credits
- ❌ PDF / URL / image inputs
- ❌ Research-from-question mode
- ❌ Style library (multiple templates)
- ❌ Brand customization
- ❌ Multiple aspect ratios / UHD export

**v1 success = the engine renders a usable Indonesian infographic from text.** Nothing more.

## 7. Constraints

- **Team:** solo.
- **Skill profile:** Strong — PHP, Tibco BusinessWorks, JMS (enterprise messaging/ESB), Oracle / SQL Developer, DB schema design. → **P4 (Schema) is home turf.**
- **Skill gap:** Learning React/Next/TypeScript/Tailwind; **has not yet shipped** a modern-stack or serverless/cloud product. This is the primary risk driver.
- **Capacity:** full-day committed (no current full-time job); weekends available.
- **Stack:** fully open — decided in P2.
- **Workflow:** agentic tool; **Jira ticket triggers the build flow** (direct P6/P7 wiring).

**Timeline (aggressive, milestone-gated):**
- Engine spike (days 1–3): prove Indonesian rendering — *gate for everything*.
- v1 slice: ~2 weeks (from spike success).
- Full clone: ~6 weeks target, 8 ceiling.
- *If the engine spike fails on Indonesian, the timeline resets — no pretending.*

## 8. Risks & flags

- **[RED FLAG] Skill ↔ scope curve.** Learning React/Next/serverless/Stripe/LLM/frontend-design *while* building a full clone is compounding, not additive. Survivable **only** because v1 is tiny and capacity is full-day. Discipline: prove engine on a slice before building breadth.
- **[RED FLAG] Engine choice is undecided and is ~70% of the app's difficulty.** Deferred to P2 — highest-risk open item.
- **[RED FLAG] Indonesian text rendering.** Whether the chosen engine can render legible, correctly-laid-out Indonesian is the make-or-break assumption (see §11).
- **[TRADE-OFF] Local-first payments.** QRIS/e-wallet integration ≠ plain Stripe; adds P6 complexity in exchange for serving the actual target market.

## 9. Explicitly deferred decisions (nothing pre-decided)

- **Generation engine** (off-the-shelf image model vs structured render pipeline) → **P2** (research + recommend).
- **Tech stack** → **P2**.
- **Payment provider** (QRIS/e-wallet aggregator) → **P6**.

## 10. Success criteria

- **Learning:** all 8 phases (P0–P7) completed *and* a product shipped.
- **Product:** a real user generates a usable **Indonesian** infographic from text, end-to-end.

## 11. Assumptions to validate

- **★ Make-or-break:** "The chosen AI engine can render legible, correctly-laid-out **Indonesian** text inside an infographic layout." → validated by the day 1–3 engine spike.
- Style *consistency* across generations is achievable with the chosen engine.
- The single unified job-to-be-done genuinely covers all three roles without per-role divergence in v1.
- Local users will pay via e-wallet for generated infographics (validated post-v1, not now).

---

**Phase gate:** P0 deliverable (`notes.md`) complete. → Ready for **P1 · PRD + AppFlow (Product Owner)**.

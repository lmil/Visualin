# PRD — Visualin

| | |
|---|---|
| **Phase** | P1 · PRD + AppFlow (Product Owner) |
| **Status** | 🟡 Draft — pending approval to gate into P2 |
| **Date** | 2026-07-08 |
| **Scope of this doc** | **v1 (thin slice)** requirements + full v1→v5 roadmap |
| **Sources** | `notes.md` (P0, approved), `README.md`, buildmygraphic generation-flow capture |

---

## 1. Overview & Positioning

**One-liner:** Visualin turns Bahasa Indonesia text into a designer-grade infographic — Indonesian-native, built for creators who explain.

**Problem.** People who explain for a living need infographics but can't (or won't) design them. The AI tools that solve this are either **English-first** (buildmygraphic) or treat infographics as a **bolt-on chart feature** inside a sprawling platform (kakak.ai). The Indonesian creator who wants a *polished, designed* explainer graphic — not a chart, not English — is underserved.

**Wedge:** **designer-grade Indonesian infographics for edukreators.** The bet is on *craft*, not feature count.

**Why Visualin over kakak.ai.** kakak.ai is a breadth play — a "most-complete AI platform" for students/lecturers/researchers, where infographic generation is a chat feature producing bar/pie/timeline/flow charts. Visualin is a **depth play**: one job — a polished, editorial, designed infographic — done better than a bolt-on can. We do not compete on how many things the app does; we compete on how good the one thing looks.

**Why Visualin over buildmygraphic.** buildmygraphic is English-first and runs on international payment rails. Visualin is **Indonesian-native** (UI *and* generated text) with **local rails** (QRIS / e-wallet, from v3).

**North star.** *"Would an edukreator post this as-is, without touching it?"* Legibility is the floor, not the goal. Publishability is the goal.

---

## 2. Users & JTBD

**Job-to-be-done (single).** *"Turn what I want to say into a visual that strengthens how I explain it to my audience."*

**Primary v1 persona — the Edukreator.** An Indonesian creator producing educational / explainer content (study tips, personal finance, health, business tips) for **Instagram and TikTok feeds**. Chosen deliberately:
- kakak.ai serves academics best and **creators worst** — attack the gap, not the stronghold.
- Design quality is *existential* for a creator (a bad graphic damages their brand), so the wedge matters most to exactly this user.
- Mobile-first, high posting frequency → the right shape for a later credit/regenerate model.
- They are the **harshest judges of design** — which is a feature: if the wedge survives them, it survives anyone.

**Roles as style-variants (destination, not v1).** Educator, business professional, and content creator are the *same job* differentiated by *style*, not distinct products. v1 serves only the **edukreator** slice; the other roles are unlocked by the style library and brand kit (v2/v4).

---

## 3. Scope & Release Plan

### 3.1 v1 — in scope
- Google authentication
- Single input: **text paste** (Bahasa Indonesia)
- **One** style: *editorial explainer*
- **One** aspect ratio: **4:5 (1080×1350)**
- Generate via a **two-stage pipeline** (analyze → render)
- **Regenerate** — free, **soft-capped per user/day** (cost control only; no credit ledger)
- Save + view results (*My Graphics*)

### 3.2 v1 — explicitly OUT
❌ payments / credits · ❌ PDF / URL / image inputs · ❌ research-from-question mode · ❌ style library (multiple styles) · ❌ brand customization · ❌ multiple ratios / UHD export · ❌ entry-intent chooser ("what do you want to do?")

> **v1 success = the engine renders a *publishable* Indonesian editorial infographic from text.** Nothing more.

### 3.3 Roadmap (v1 → v5)

| Ver | Theme | Ships | Rationale |
|---|---|---|---|
| **v1** | Engine proof | text-only · 1 style · 4:5 · Indonesian · Google auth · regenerate (free, soft-capped) · save/view | Retire the engine risk. Answer one question: *does it render designer-grade Indonesian?* |
| **v2** | Real tool | style library · +1:1 +9:16 ratios · generation history | Earn retention. Variety of *good-looking* styles is the first real demand of a design-quality user. Still free + soft-capped. |
| **v3** | Money + local access | credit ledger · one-time credit packs · **QRIS / e-wallet** · regenerate now deducts credits | Meter only once value is proven. Local rails are the differentiator vs buildmygraphic — they earn their own release. |
| **v4** | Expand the JTBD | PDF / URL / image inputs · research-from-question · brand customization (logo, palette, instructions, own images) | Brand kit pulls in business pros + serious creators → activates "one JTBD, three roles." |
| **v5** | Scale & polish | UHD / hi-res export · storage/CDN at scale · multi-panel / carousel output | Professional output; carousel unlocks the biggest edukreator format. |

**[TRADE-OFF]** Payments sit at **v3**, not v2, though API cost starts on day one. The v1/v2 **soft-cap** protects spend without a ledger; charging before retention is proven usually just paywalls something nobody's hooked on yet. Revisit only if unbounded API cost across two releases becomes intolerable.

---

## 4. Requirements

> IDs are stable handles for P5 build tickets and P7 tests. Each FR is independently testable.

### 4.1 Functional requirements

**FR-1 · Google authentication**
- User signs in with Google OAuth; no email/password path in v1.
- *AC:* unauthenticated visit to the generator redirects to sign-in; successful OAuth lands on the generator with an active session; a user record exists/updates on first sign-in.

**FR-2 · Text input**
- Single text field for pasted Bahasa Indonesia source content; enforce a max length (value set in P2/P3).
- *AC:* empty input disables **Generate**; over-limit input is blocked with an in-voice message stating the limit; whitespace-only counts as empty.

**FR-3 · Fixed style & ratio (v1)**
- Style = *editorial explainer*, ratio = **4:5**, both fixed and visible (not editable) in v1.
- *AC:* UI shows the active style + ratio as read-only; no style/ratio selector is present.

**FR-4 · Generate**
- **Generate** submits `{ sourceText, style:"editorial-explainer", ratio:"4:5", userId }` to the engine.
- *AC:* a valid submit produces exactly one rendered graphic; a failed generation returns an in-voice error explaining what to do next (retry), never a raw stack trace; the input is preserved on failure.

**FR-5 · Two-stage progress**
- Surface the pipeline's two stages to the user: **Analyzing content → Generating**.
- *AC:* both states are shown in order during a real generation; the user is told they can leave and return.

**FR-6 · Regenerate + soft cap**
- **Regenerate** re-runs generation on the same input. Free in v1, but limited to **N generations/user/day** (N set in P2; purely a cost guard).
- *AC:* under the cap, regenerate produces a new result; at the cap, regenerate is blocked with an in-voice notice (retry after reset) and **no engine call is made**; the cap is per-user and resets on a fixed daily boundary.

**FR-7 · Save result**
- User saves a generated graphic to their account.
- *AC:* a saved graphic persists a record `{ id, userId, assetUrl, sourceText, style, ratio, createdAt }`; the asset is retrievable after session end.

**FR-8 · View saved (My Graphics)**
- User views a list of their saved graphics and opens one.
- *AC:* the list shows only the signed-in user's graphics, newest first; opening one shows the full-res result; an empty list shows an invitation to create, not a blank screen.

### 4.2 Non-functional requirements

**NFR-1 · Indonesian legibility gate — 100% (hard).** Every generated graphic must render Bahasa Indonesia text with correct, unbroken, non-garbled glyphs and legible layout. This is a *release gate*, not a target — a single garbled-text output fails v1.

**NFR-2 · Generation time.** Target end-to-end generation within a set budget (value in P2); if exceeded, the progress UI (FR-5) must keep the user informed and allow leaving/returning.

**NFR-3 · Cost control.** No code path allows unbounded engine calls; the FR-6 soft-cap is the guard. Per-user daily generation count must be observable.

**NFR-4 · Mobile-first responsive.** The edukreator is a phone user. All v1 screens must be fully usable and legible on a narrow mobile viewport, keyboard-accessible, and honor reduced-motion.

**NFR-5 · Indonesian-native i18n.** UI copy defaults to Bahasa Indonesia; the architecture must not hard-code English strings (structure for locale from day one even if only ID ships).

**NFR-6 · Session security.** OAuth tokens/sessions handled server-side; no secrets exposed client-side; saved graphics are private to their owner.

---

## 5. Success Metrics, Assumptions & Risks

### 5.1 Success metrics
- **Product (quality, not volume):** across ~8–10 recruited edukreators generating a few graphics each, **≥70% "would post as-is with zero manual edits"** — *plus* the hard **100% Indonesian-legibility gate** (NFR-1). The post-as-is rate *is* the proof of the wedge.
- **Learning:** v1 live on a public URL with Google auth + generate + save working end-to-end, and all phases P0–P7 documented.

### 5.2 Assumptions to validate
- **★ Make-or-break:** the chosen engine renders legible, correctly-laid-out **Indonesian** text inside a designed layout → validated by the 3-day engine spike **before** v1 build.
- **[P2 HINT — carried from the buildmygraphic capture]** buildmygraphic's two-stage *"Analyzing → Generating"* implies a **structured render pipeline** (LLM structures content → engine places real text), not a raw text-to-image model. If Visualin builds the same way, the make-or-break risk largely dissolves: text placed as *real text* renders correctly by construction. Garbled-text risk is chiefly a *pure-image-model* failure mode. The spike should test a structured pipeline first.
- Style *consistency* across generations is achievable with the chosen engine.
- The single JTBD covers the edukreator slice without per-sub-niche divergence in v1.

### 5.3 Risks (carried from P0)
- **[RED FLAG] Wedge concentrates risk.** Choosing "design quality" (over "it's in Indonesian") means v1 must visibly out-design kakak's chart feature. The engine spike is no longer a side gate — it *is* the product bet.
- **[RED FLAG] Skill ↔ scope curve.** Learning React/Next/TS/Tailwind + serverless + LLM orchestration *while* building. Survivable only because v1 is tiny and capacity is full-day. Discipline: prove the engine on the slice before building breadth.
- **[RED FLAG] Engine undecided.** ~70% of the difficulty, deferred to P2 — the highest-risk open item.

---

**Phase gate:** on approval, `PRD.md` + `AppFlow.md` complete → ready for **P2 · TechStack**, which must open with the engine decision and the structured-pipeline hint above.

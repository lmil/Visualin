# TechStack — Visualin

| | |
|---|---|
| **Phase** | P2 · TechStack (System Architect) |
| **Status** | ✅ Approved — ready to gate into P3 |
| **Date** | 2026-07-08 |
| **Sources** | `notes.md`, `README.md` (P0), `PRD.md`, `AppFlow.md` (P1, approved) |
| **Companion** | `Lock.md` (exact version pins) |

> Versions in this doc are named at the major level for readability. Exact,
> installable pins live in `Lock.md`. Nothing here uses a version range.

---

## 1. Context & Constraints (carried from P0/P1 — not re-decided)

Solo dev; strong backend/data foundation (PHP, Tibco BusinessWorks, JMS, Oracle, DB schema design); **learning** React/Next/TypeScript/Tailwind; **has not shipped** a modern-stack or serverless/cloud product — this is the primary risk driver. Serverless-friendly, mobile-first (NFR-4), Bahasa Indonesia-default i18n (NFR-5), server-side auth/secrets (NFR-6), room to add QRIS/e-wallet later (P6) without re-platforming. Full-day capacity; aggressive milestone-gated timeline.

**Design principle carried through every decision below:** where a component is a *means* (not the thing being learned), it sits behind a seam the builder owns, so it can be swapped without a rewrite. This applies to the image engine **and** to auth.

---

## 2. The Engine — the ~70% decision (RESOLVED)

### 2.1 Evidence base
buildmygraphic.com runs **Gemini image generation via Replicate** (confirmed by their published Terms of Service), on the **legacy Nano Banana / Gemini 2.5 Flash Image** model. User-generated samples from that product proved the model **garbles Bahasa Indonesia** in dense layouts (`TRANSFOORMER`, `MENAIKKAN TEHTAK`), and their own generator UI ships a *"AI can make mistakes, including typos"* disclaimer.

**Conclusion:** the reference product's engine **provably violates Visualin's NFR-1 (100% Indonesian-legibility gate).** Visualin therefore does **not** clone that engine — it tests newer, text-tuned models against the gate.

> The garbling is not intrinsic to image models — it is intrinsic to the *2025 legacy* model. Google's newer image models were explicitly tuned for multilingual in-image text (the exact failure mode), which is why Visualin's bet is testable rather than doomed.

### 2.2 Architecture — owned `ImageProvider` adapter interface (first-class, v1)
A single internal seam that all generation flows through:

```
generate(prompt, style, ratio)
  → { imageBytes, costUsd, latencyMs, model, tokens?, complexitySignals }
```

Thin adapters sit behind it: `GeminiAdapter`, `OpenAIAdapter`, and (optional) `FalAdapter` / `ReplicateAdapter`. This is the ESB/adapter pattern from the builder's Tibco background applied to LLM providers — home turf. It delivers three things at once: **model-agnosticism**, **cost instrumentation** (see §10), and the **routing seam** (see §4).

### 2.3 Model selection (spike-decided — see §14)
- **v1 image model:** the winner of the 3-day spike between **Nano Banana Pro** (Gemini 3 Pro Image) and **GPT Image 2** — both text-tuned, multilingual, infographic-targeted.
- **v2 draft tier:** **Nano Banana 2 Lite** (Gemini 3.1 Flash Image), behind the same seam.
- **Selection rule:** *the cheapest model that clears the 100% Bahasa legibility gate wins; margin (§12) breaks ties.*
- **Fallback if no pure model clears 100%:** **hybrid** — image model paints a text-free illustration, real text composited via SVG/HTML overlay (legibility guaranteed by construction). The spike produces the failure data that would justify this.

### 2.4 Stage-1 "Analyze" LLM (separate from the image model)
One LLM call that structures raw Bahasa text → `{ title, sections[], keyPoints[] }` **plus** layout/color/style hints, then composes the image prompt.
- **Scope: structure-only for v1** (not summarization). Summarization is a v2+ mode. Rationale: v1 must isolate the "does it render Bahasa legibly?" question; adding "does it summarize Bahasa well?" muddies the spike.
- **Model: Gemini Flash (3.x)** as default, **coupled to the image-winner's vendor** for single-vendor operational simplicity. Swappable via the adapter.
- **Cost:** negligible (~$0.001–0.005/generation) — the image call dominates.

> Confirmed from buildmygraphic's process capture: their **nine progress labels are UX theater** over a single long-running image call (the preview image is static across "Refining typography," "Choosing color schemes," etc.). The real pipeline is **1 structure call + 1 image call**. Visualin replicates the *honest* core (2–3 real progress states per FR-5), not the theater.

---

## 3. Agnosticism (the model-swap seam)

**Goal (approved):** never hard-couple to one image model; the field moves monthly (GPT Image 2, Nano Banana 2 Lite/Pro, and whatever ships next).

- `[RED FLAG — resolved]` **Replicate is rejected as the abstraction layer.** It is open-weight-focused and does **not** host GPT Image 2 or Gemini as first-class models — using it as the seam would *block* the two models most worth testing. buildmygraphic's use of Replicate is a narrow Gemini-compute-billing arrangement, not general model-swapping.
- **The abstraction is the owned adapter seam (§2.2), not a vendor.** Start with **direct vendor SDKs** (cheapest, real API learning). **fal.ai** is documented as an optional multi-model adapter (it hosts both GPT Image 2 and the Nano Banana family and accepts your own vendor key). **Replicate** is demoted to "one possible adapter for open-weight models, if ever needed."

---

## 4. Cost-Aware Routing (Lite ↔ Pro) — architecture-ready in v1, ACTIVE in v2

Route simple generations → Lite, complex → Pro; **user is charged the same 1 credit either way**; margin is captured on the difference (~$225/mo saved at 500 users, ~$2,700/mo at 5,000 — no revenue cost, no user-visible change).

- **Routing signal:** cheap, deterministic **text-load features from Stage-1's output** (character count, section/label count, longest text run) — computed in microseconds, **zero extra API cost**. NOT a separate "complexity classifier" LLM call.
- `[RED FLAG — gate protection]` **Asymmetric rule: default to Pro; downgrade to Lite only when *provably safe*.** Lite is cheaper because it is a weaker text renderer — routing a legibility-risky job to it to save $0.07 would gamble the NFR-1 gate. The default is the safe/expensive model; Lite is the earned exception.
- **Threshold** is calibrated by the 3-day spike (which produces Lite's legibility-failure curve vs text density) and re-validated on v2 production data before it is trusted.
- **Safety net:** a lightweight OCR-confidence / regenerate-signal check on Lite output → **auto-retry on Pro at platform cost** if a Lite route garbles. Makes any misroute invisible to the user (worst case: a slightly longer wait, never a garbled image).
- **Sequencing:** **v1** logs complexity signals on every record and always routes to the single gate-winner (zero routing risk while the gate is being proven). **v2** activates the router — exactly when volume makes cost matter *and* production data exists to route safely.

---

## 5. Frontend

- **Next.js 16** (App Router) — server components keep image-model and provider keys server-side (NFR-6); it is the framework the builder is learning, so it doubles as the learning target.
- **React 19**, **TypeScript 6**, **Tailwind CSS 4**.
- **Mobile-first (NFR-4):** every v1 screen must be fully usable and legible on a narrow mobile viewport, keyboard-accessible, and honor reduced-motion. The edukreator is a phone user.

---

## 6. Auth

- **Google OAuth** via **Auth.js v5** (`next-auth` v5 line), **server-side sessions** (NFR-6). No email/password path in v1 (FR-1). Google auth is also a natural abuse limiter — there is no anonymous generation path.
- **Decision rationale (learning-load triage):** the builder's #1 risk is learning the modern stack *while* shipping. Auth.js has the largest ecosystem and the most existing answers on the web, which matters more than release maturity when auth is a *means*, not the thing being learned. Better Auth (`better-auth@1.x`, stable) is the technically forward-looking alternative and was considered; it was set aside for ecosystem/tutorial density.
- `[RED FLAG — beta dependency]` `next-auth@5` is still a **beta** release, and its documented Next-16 middleware pattern predates Next 16's `middleware.ts` → `proxy.ts` rename (a known workaround is required: import `auth.config` — not `auth.ts` — in `proxy.ts` to avoid pulling Node-only adapter code into the edge runtime). **Mitigation:** auth sits behind an owned session helper, so a later swap to Better Auth is contained, not a rewrite. Re-evaluate at v2.
- **Adapter:** `@auth/drizzle-adapter` (stable), pairing with the data layer in §9.

---

## 7. i18n

- **next-intl 4**, **Bahasa Indonesia default**, structured for locale from day one (NFR-5). No hard-coded English strings anywhere in the architecture, even though only ID ships in v1.

---

## 8. Storage

- **Cloudflare R2** via the S3-compatible client (`@aws-sdk/client-s3`). Chosen for **zero-egress economics** on image serving (serving many 1080×1350 PNGs). Saved graphics are **private to their owner** (NFR-6). This is the one piece of buildmygraphic's infrastructure worth copying directly.

---

## 9. Data Layer

- **Drizzle ORM** (TypeScript-native; the builder's schema-design strength transfers directly). **Zod** for runtime validation.
- The **actual schema is P4's deliverable** — not decided here. Field names below are indicative for the generation contract only.
- **Generation record (indicative fields):**
  `{ id, userId, assetUrl, sourceText, style, ratio, provider, model, costUsd, latencyMs, tokens?, complexitySignals, createdAt }`

---

## 10. Cost Instrumentation (NOT a separate dashboard app)

Cost / latency / model / tokens are stamped on **every generation record at the adapter seam** (§2.2) — the one place all generations flow through.

- "Cost per generation" is a **column**, not a subsystem.
- "Margin per generation" (v3) = `creditPriceCharged − costUsd`, a trivial query.
- "Am I bleeding money this week" = `SUM(costUsd)` grouped by day.
- `[RED FLAG — build-order]` This must be captured **from day one**; retrofitting cost tracking means backfilling records that were never instrumented.
- `[TRADE-OFF]` For token-priced models (GPT Image 2), `costUsd` must be **computed from the actual response usage**, not a hardcoded constant (thinking-mode reasoning tokens vary). The adapter contract returns usage so this is uniform across providers.

No dashboard application in v1 — a handful of queries, optionally surfaced as a simple admin page later.

---

## 11. Deferred Values (SET this phase)

| Value | Setting | Note |
|---|---|---|
| **Soft-cap N/day** (FR-6) | **10** | Also the primary **financial control** at v2 — see `[RED FLAG]` below. Revisit (possibly lower, or a lifetime free-cap) **before v2 goes public.** |
| **Text input max** (FR-2) | **5,000 chars** | Spike-tunable. Structure-only assumption; buildmygraphic's 50,000 serves a *summarize* mode that Visualin v1 does not have. A larger cap directly raises garble risk with no compression stage to protect the gate. |
| **Latency** (NFR-2) | **≤30s target / 120s ceiling** | With **mandatory leave-and-return UX.** Corrected from a hard 30s gate: the reference product runs 60–120s and tells users to "leave and check back." Hitting ≤30s is a **potential competitive wedge**, not a disqualifier for a legibility-winning model. |
| **Progress states** (FR-5) | **2–3 honest labels** | e.g. "Menyusun konten → Membuat visual." Not nine theatrical ones. |

`[RED FLAG — soft-cap N is a financial control]` At v2 (public + free), realistic spend and cap-bound worst-case spend differ by **~15×** (see §12). N=10 is the circuit breaker. **Lower N or add a lifetime free-cap before v2 ships.**

---

## 12. Verified Cost & Revenue Model

**Per-generation cost** (all-in, at v1's 4:5 ~1080×1350 / 1–2MP size).
`[CORRECTION]` Nano Banana Pro at 4:5 lands in the **1K–2K token tier at ~$0.134**, **not** the 1K-square $0.039 rate that was quoted earlier in research. Verified against Google's official Gemini API pricing page (2026-07-08).

| Engine | All-in cost/gen | Margin/gen @ $0.30/credit | Margin % |
|---|---|---|---|
| Nano Banana 2 Lite | ~$0.07 | ~$0.23 | ~77% |
| Nano Banana Pro | ~$0.14 | ~$0.16 | ~53% |
| GPT Image 2 | ~$0.18 | ~$0.12 | ~40% |

**Monthly cost (Nano Banana Pro as likely gate-winner):**

| Stage | Realistic monthly | Cap-bound worst case | Note |
|---|---|---|---|
| Spike (one-time) | ~$30–90 total | — | Noise. Test freely. Prepay ~$20–30 to activate paid keys (Google/OpenAI minimums). |
| v1 (beta, ~10 users) | **~$40–70** | ~$600 | Trivial; budget not stressed. |
| v2 (public, free, ~500 users) | **~$900–1,600** | **~$21,000** | `[RED FLAG]` The cost cliff. Draft-tiering (§4) + soft-cap (§11) are the controls. |
| v3 (paid) | Offset by revenue | — | ~50% net margin (Pro), ~72% if Lite clears the gate. |

**Revenue model** (if adopting buildmygraphic's credit pricing: Starter $5/12cr = $0.417/cr; Pro $15/50cr = $0.30/cr): ~**50% net margin** at Nano Banana Pro after payment fees (~6%), at ~$0.32 blended revenue/credit. Break-even over a ~$70/mo infra bill at roughly **~50 paying users**.

**Forward-flags for v3 pricing (deferred to a later phase — logged so they are not lost):**
- `[RED FLAG]` **Regeneration must cost credits in v3.** Free regen is fine for v1/v2; carrying free regen into the paid tier roughly **halves** margin (users regenerate ~0.8–1× per keeper). buildmygraphic charges 1 credit per regenerate/tweak — adopt this.
- `[TRADE-OFF]` **4:5 sits in the pricier image tier.** Same 1 credit, ~3× the 1K-square cost. Conscious tradeoff — the ratio is the IG/TikTok wedge.
- `[RED FLAG]` **Price in IDR at locally-calibrated points, do not convert USD.** $5/$15 ≈ Rp 80K/240K is steep for Indonesian creators; QRIS-native means smaller/cheaper packs matched to e-wallet habits. This is a dedicated v3 pricing-research task.

---

## 13. Hosting

- **Vercel first** (SET) — the natural Next.js fit, keeps QRIS-later clean, fastest path for a solo dev learning the stack. **Changeable later** (the app is not Vercel-locked).
- `[TRADE-OFF / forward-flag for P5/P6]` 120s generation exceeds standard serverless function limits → generation needs an **async/queue + webhook-or-polling** pattern (which buildmygraphic's "leave and check back" flow confirms they use). Not blocking P2; flagged for the implementation plan.

---

## 14. The 3-Day Engine Spike (gates ALL v1 development)

The single highest-risk item in the project. Throwaway code; its only job is to retire the make-or-break assumption.

- **Day 1 — cheapest-first legibility test.** Generate ~15–20 real Bahasa infographics at v1 text-density on **Nano Banana 2 Lite**, **Nano Banana Pro**, and **GPT Image 2**. Count legibility failures against the **100% gate**. Record text-density → failure-rate data (**this by-product produces the §4 routing threshold**).
- **Day 2 — escalate + aesthetic A/B.** Push survivors to premium tiers / thinking-mode as needed. A/B the aesthetic against the real buildmygraphic samples: *"Would an edukreator post this as-is?"*
- **Day 3 — lock.** Lock the v1 engine (cheapest model clearing 100%), OR — if none clear 100% pure — lock the **hybrid** (§2.3) with the failure data justifying the extra engineering.
- **Deliverables:** chosen v1 engine; Lite-safe routing threshold; real per-generation cost + latency numbers; honest aesthetic verdict vs the reference.

---

## Open items carried forward (not blocking P2)

- **Hosting async/queue pattern** for 120s generation → **P5/P6**.
- **v3 pricing** (regen-costs-credits, IDR localization, 4:5 tier) → dedicated pricing phase.
- **Soft-cap N re-evaluation** before v2 public launch.
- **Auth re-evaluation** (Auth.js v5 beta → possible Better Auth swap) at v2.

---

**Phase gate:** `TechStack.md` + `Lock.md` complete → **P2 done**, ready for **P3 · Content (Design Lead)**, which opens with the design system for the single v1 *editorial explainer* style — the aesthetic that must out-design kakak.ai's chart feature and survive the spike's "post-as-is" test.

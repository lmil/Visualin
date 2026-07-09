# Lock — Visualin (exact version pins, no ranges)

| | |
|---|---|
| **Phase** | P2 · TechStack |
| **Status** | ✅ Approved — all pins resolved |
| **Date** | 2026-07-08 |
| **Verified** | npm registry, 2026-07-08 |
| **Companion** | `TechStack.md` (rationale per decision) |

> Exact pins only — no `^`, no `~`, no ranges. Every version below was
> resolved against the live npm registry on the date above.

---

## Core framework

| Package | Pinned version | Purpose |
|---|---|---|
| `next` | `16.2.10` | Framework (App Router) |
| `react` | `19.2.7` | UI runtime |
| `react-dom` | `19.2.7` | UI runtime |
| `typescript` | `6.0.3` | Language |
| `tailwindcss` | `4.3.2` | Styling |

## AI / generation engine

| Package | Pinned version | Purpose |
|---|---|---|
| `@google/genai` | `2.10.0` | Stage-1 Analyze LLM **and** Gemini image adapter. **New unified SDK — NOT the legacy `@google/generative-ai`.** |
| `openai` | `6.45.0` | GPT Image 2 adapter |

## Auth

| Package | Pinned version | Purpose |
|---|---|---|
| `next-auth` | `5.0.0-beta.31` | Auth.js v5 — Google OAuth, server-side sessions |
| `@auth/drizzle-adapter` | `1.11.2` | Auth ↔ Drizzle persistence adapter (stable) |

## Data & validation

| Package | Pinned version | Purpose |
|---|---|---|
| `drizzle-orm` | `0.45.2` | TypeScript-native ORM |
| `zod` | `4.4.3` | Runtime validation |

## Storage

| Package | Pinned version | Purpose |
|---|---|---|
| `@aws-sdk/client-s3` | `3.1081.0` | Cloudflare R2 (S3-compatible) client |

## i18n

| Package | Pinned version | Purpose |
|---|---|---|
| `next-intl` | `4.13.1` | Bahasa Indonesia-default internationalization |

---

## Flags & notes

- `[RED FLAG — beta]` **`next-auth@5.0.0-beta.31` is a beta release.** The stable `latest` dist-tag is still on the v4 line (`4.24.14`), which does **not** cleanly support Next 16 (requires `--legacy-peer-deps`/`--force`). v5 is the correct choice for Next 16, but:
  - Its documented Next-16 middleware pattern predates the `middleware.ts` → `proxy.ts` rename. **Workaround:** in `proxy.ts`, import `auth.config` (edge-safe config only), **not** `auth.ts` (which pulls the Drizzle adapter / Node APIs into the edge runtime).
  - Auth is isolated behind an owned session helper (see `TechStack.md` §6) so a later swap to `better-auth` (`1.6.23`, stable) is contained. **Re-evaluate at v2.**

- `[RED FLAG — model retirement]` The **legacy Nano Banana / Gemini 2.5 Flash Image** model is on a retirement path (scheduled shutdown ~2026-10-02). Do **not** build on it. The spike and v1 target the **current** models: Gemini 3.1 Flash Image (Nano Banana 2 Lite) and Gemini 3 Pro Image (Nano Banana Pro), plus GPT Image 2.

- **`@google/genai` vs `@google/generative-ai`:** the pinned `@google/genai@2.10.0` is the **new unified SDK**. The older `@google/generative-ai` (0.24.x) is legacy — do not install it.

- **Image model IDs are runtime configuration, not npm packages** — they are selected via the `ImageProvider` adapter and set after the spike (`gemini-3-pro-image`, `gemini-3.1-flash-image`, `gpt-image-2`). They are intentionally **not** pinned here; the adapter seam is what makes them swappable.

- **Deployment target:** Vercel (first; changeable). No package pin — platform-level.

---

## Not yet added (deferred to their phases — do NOT pre-install)

| Concern | Phase | Reason |
|---|---|---|
| Payment SDK (Polar / QRIS aggregator) | v3 / P6 | Payments deferred to v3. |
| Async queue / job runner (for 120s generation) | P5 / P6 | Hosting async pattern is an implementation-plan decision. |
| Test tooling (Playwright etc.) | P5 / P6 | Sequenced in the implementation plan. |
| OCR/verification lib (routing safety net) | v2 | Cost-routing activates in v2, not v1. |

---

**Phase gate:** all pins resolved and live-verified → **P2 complete**, ready for **P3 · Content**.

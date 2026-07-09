# Decision Note — Image-model gateway (Replicate / fal.ai) as middleman

|              |                                                                                                                          |
| ------------ | ------------------------------------------------------------------------------------------------------------------------ |
| **Type**     | Decision note (ADR-style)                                                                                                |
| **Phase**    | P2 · TechStack — companion to `TechStack.md` §3                                                                          |
| **Status**   | Decided for v1 · Re-litigate at v2                                                                                       |
| **Date**     | 2026-07-08                                                                                                               |
| **Decision** | **v1: direct vendor SDKs, no gateway.** fal.ai reconsidered at v2 _if_ a style library introduces multiple image models. |

---

## Context

buildmygraphic.com (the functional reference) routes image generation through
**Replicate** as a compute host for Google Gemini (confirmed in their ToS). A
gateway like Replicate or fal.ai adds a hop between Visualin's server and the
image model. This note records **why the reference product uses a middleman**,
**why v1 does not**, and **the exact condition under which that reverses** — so
the decision is re-litigated at v2 on evidence, not re-argued from scratch.

This is a companion to `TechStack.md` §3 (Agnosticism), which already states the
v1 position. This note is the _reasoning_ behind it.

---

## The case FOR a gateway (why buildmygraphic likely uses one)

1. **One integration, one bill, many models.** A gateway exposes many image
   models behind a single API shape and a single invoice. buildmygraphic ships
   ~9+ visual styles; if those styles map to _different underlying models_, a
   gateway means one integration instead of many. **This is the strongest
   structural reason** — and it is a _breadth_ benefit.

2. **Experimentation velocity.** Trying a new model = changing a string. No new
   SDK, auth, or billing setup. For a product whose moat is aesthetic variety,
   bolting on a hot new style-model in an afternoon is a real advantage.

3. **Operational offload.** Gateways handle queuing, autoscaling, GPU cold-starts,
   and retries for bursty, slow (60–120s) image workloads. For a small team not
   wanting to run inference infra, this is genuine value. **This is the most
   defensible single argument for the middleman.**

4. **Billing smoothing for awkward models.** buildmygraphic's ToS calls Replicate
   a _"compute host for Google Gemini image generation."_ This suggests Replicate
   mediates the Gemini compute/billing relationship — possibly a **historical
   artifact** from when direct Gemini image access was messier, never since
   unwound.

5. **Uptime abstraction (weak).** A mature gateway can sometimes fail over between
   providers. In practice this is weaker than it sounds — see cons.

---

## The case AGAINST (the costs, sharpened)

- **Margin.** The gateway takes a cut on every generation. At ~$0.14 image cost
  and a ~50% target margin, even a 10–15% markup bites the thinnest part of the
  unit economics — the part already flagged as margin-sensitive (IDR pricing,
  solo dev).

- **Extra failure point — often without the failover benefit.** The gateway can
  be down / rate-limited / slow _independently_ of the model. And if you call
  _one specific model_ (Gemini) through it, a Gemini outage takes you down
  regardless — the gateway added latency in front of the same dependency, not
  insulation from it.

- **Obscured cost accounting.** Visualin's cost instrumentation (`TechStack.md`
  §10) measures _true per-model cost at the adapter seam_. A gateway shows _its_
  price, not the vendor's — directly undermining a core design goal.

- **Less control.** Error semantics, rate-limit behavior, streaming, exact usage
  accounting — all mediated through the gateway's abstraction.

- **The abstraction becomes its own lock-in.** The tool adopted _for_
  model-agnosticism becomes a dependency; migrating off it later is its own
  project.

---

## Why the answer differs for Visualin v1 vs buildmygraphic

**Framing:** _a gateway is worth its cut when its value scales with your product;
it's dead weight when it doesn't._

| Factor                       | buildmygraphic        | Visualin v1                                          |
| ---------------------------- | --------------------- | ---------------------------------------------------- |
| Models in play               | Many (variety = moat) | **One** (spike-winner)                               |
| Need to add models fast      | High                  | **Low** (depth, not breadth)                         |
| Running inference infra?     | No                    | No — _but direct vendor APIs are also fully managed_ |
| Margin sensitivity           | Lower                 | **High**                                             |
| Cost-measurement precision   | Not a goal            | **Core design goal** (§10)                           |
| Learning value of direct API | Already known         | **High** (learning the stack is a project goal)      |

**The decisive asymmetry:** the middleman's #1 benefit (many models, one
integration) is exactly what v1 doesn't need, while its #1 cost (margin +
obscured cost accounting) lands exactly where v1 is most sensitive. Positive
trade for them; negative trade for v1.

**Key correction to a common assumption:** going direct does **not** forfeit the
operational offload. Calling Gemini/OpenAI image APIs directly is still a fully
_managed_ API — no GPUs, no cold-starts to run. The gateway's operational value
is real mainly for **open-weight models you'd otherwise self-host** (e.g. Flux on
your own hardware) — which Visualin is not doing. So the offload argument (#3
above) largely does not apply to a direct-to-frontier-vendor architecture.

---

## Decision

- **v1: direct vendor SDKs, no gateway.** One model, thin margin, cost-precision
  and API-learning goals — the gateway loses on every axis that matters here.
- The **owned `ImageProvider` adapter seam** (`TechStack.md` §2.2) makes this
  reversible at near-zero cost: a gateway can later be added as _one adapter
  behind the seam_ without a rewrite.

---

## Re-litigation trigger (v2)

Revisit **if and only if** v2 introduces a **style library with multiple
underlying image models.** That is the precise condition under which the
gateway's breadth benefit (#1, #2) turns positive and can outweigh the margin
cut.

When that trigger fires, evaluate on these tests:

1. **Model coverage — the gating test.** Does the gateway host Visualin's _actual
   winners_ (Nano Banana Pro / GPT Image 2) as **first-class** models?
   - `[RED FLAG carried from TechStack §3]` **Replicate is open-weight-focused
     and may NOT host GPT Image 2 or Gemini as first-class** — the same reason it
     was rejected as the v1 abstraction layer. **fal.ai explicitly hosts both**
     and accepts your own vendor key. → **If a gateway earns its place in v2, it
     is almost certainly fal.ai, not Replicate.**
2. **Margin math.** Gateway markup × v2 projected volume vs. the integration
   effort saved. Quantify against the v2 cost model (~$900–1,600/mo realistic).
3. **Cost-accounting fidelity.** Can the gateway still return _true per-model
   usage_ so `TechStack.md` §10 instrumentation stays honest? If it only returns
   its own blended price, that is a strike.
4. **Failure-mode audit.** Does it provide _real_ multi-provider failover for the
   models you use, or just a hop in front of the same single dependency?

**Default if the trigger never fires:** stay direct. The gateway is not owed a
place in the architecture; it must earn one against these tests.

---

**Related:** `TechStack.md` §2.2 (adapter seam), §3 (agnosticism), §10 (cost
instrumentation) · `Lock.md` (no gateway SDK pinned — intentional).

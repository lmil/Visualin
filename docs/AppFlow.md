# AppFlow — Visualin

| | |
|---|---|
| **Phase** | P1 · PRD + AppFlow (Product Owner) |
| **Status** | 🟡 Draft — pending approval to gate into P2 |
| **Scope** | **v1 only.** Later-version flows (payments, extra inputs, brand kit) are out. |
| **Companion** | `PRD.md` (requirements) |

Two diagrams, each doing the job it's best at: a **sequence diagram** for the core generate/regenerate interaction (the request/response timing that defines the API contract), and a **navigation flowchart** for how a user moves through screens. Both are Mermaid — text, git-diffable, and they render in GitHub and most Markdown viewers.

*(The earlier HTML swimlane has been retired in favour of these; a swimlane blurs the time axis and is heavier to maintain across P2–P7.)*

---

## 1. Primary — Generation sequence (Flows B + C)

The core of v1: a request to a **two-stage engine** and back. This diagram is also the source of truth for the `/generate` contract.

```mermaid
sequenceDiagram
    actor U as User (edukreator)
    participant FE as Frontend · Next.js
    participant EN as Engine · analyze→render
    participant DB as Storage · DB

    Note over U,DB: Precondition — user authenticated (see Navigation flow)

    U->>FE: Paste Bahasa Indonesia source text
    Note right of FE: style = editorial explainer • ratio = 4:5<br/>(both fixed in v1)
    U->>FE: Tap "Generate"
    FE->>EN: POST /generate {sourceText, style, ratio, userId}

    rect rgb(227, 244, 236)
    Note over EN: Two-stage pipeline
    EN->>EN: Stage 1 · Analyze → structured content {title, sections[], keyPoints[]}
    EN->>EN: Stage 2 · Render → place real text into 4:5 layout
    end

    EN-->>FE: rendered graphic (PNG 1080×1350)
    FE-->>U: Result screen

    opt User saves
        U->>FE: Save
        FE->>DB: insert generation {id, userId, assetUrl, sourceText, ...}
        DB-->>FE: ok
    end

    opt Regenerate (v1 free, soft-capped)
        U->>FE: Tap "Regenerate"
        FE->>FE: Soft-cap check (count < N today?)
        alt Under cap
            FE->>EN: POST /generate (same input)
            EN-->>FE: new rendered graphic
            FE-->>U: New result
        else At cap
            FE-->>U: Limit-reached notice (retry after daily reset)
            Note right of FE: no engine call
        end
    end
```

**Why stage 1 → stage 2 is drawn explicitly:** stage 2 places *real text*, so Bahasa Indonesia renders legibly by construction. This is the make-or-break assumption designed into the flow (`PRD.md` §5.2). The self-messages on `EN` are placeholders for the internal pipeline — P2 decides whether both stages are one service or two.

---

## 2. Navigation (Flows A + D)

Where the user goes: the auth gate, the generator, save, and the My Graphics screen with its empty state.

```mermaid
flowchart TD
    Start([User opens Visualin]) --> Session{Active session?}
    Session -- No --> OAuth[Google sign-in]
    OAuth --> Consent{Consent granted?}
    Consent -- No --> Entry[Return to entry · in-voice message] --> Session
    Consent -- Yes --> Upsert[(Upsert user record)] --> Gen
    Session -- Yes --> Gen[Generator screen]
    Gen --> Generate[[Generate · see Sequence flow]]
    Generate --> Result[Result screen]
    Result --> Regen[[Regenerate · see Sequence flow]]
    Result --> Save[Save] --> MyG[My Graphics]
    MyG --> Empty{Any graphics?}
    Empty -- No --> Invite[Empty state · invite to create] --> Gen
    Empty -- Yes --> List[List · newest first] --> Open[Open full-res result]
```

---

## 3. Flow narratives

**Flow A — Auth & entry.** User opens Visualin → Frontend checks session. No session → Google sign-in; on consent, Storage upserts the user record and the generator opens. Cancelled OAuth returns to entry with an in-voice message; no account is created.

**Flow B — Generate (happy path).** Style (*editorial explainer*) and ratio (*4:5*) are fixed and read-only in v1. User pastes Bahasa Indonesia text (empty/whitespace keeps **Generate** disabled; over-limit is blocked with the limit stated) and taps **Generate**. The Engine runs **Analyze** then **Render**, both surfaced to the user, and returns the graphic to the Result screen. On failure: in-voice error + retry, input preserved.

**Flow C — Regenerate (the one v1 branch).** From the Result screen, **Regenerate** re-runs on the same input after a **soft-cap check** (`count < N today?`). Under cap → re-run engine → new result (count increments). At cap → limit notice, **no engine call**. Free in v1; credit *deduction* arrives in v3 — same checkpoint, different consequence.

**Flow D — Save & view.** **Save** inserts a generation record. **My Graphics** lists the user's graphics newest-first; opening one shows full-res. Empty list → an invitation to create, not a blank screen.

---

## 4. Entity glossary

| Entity | Shape | Passed |
|---|---|---|
| **user profile** | `{ id, email, name }` | Auth → Storage (Flow A) |
| **source text** | Bahasa Indonesia string (bounded length) | User → Frontend (Flow B) |
| **generation request** | `{ sourceText, style:"editorial-explainer", ratio:"4:5", userId }` | Frontend → Engine (Flow B) |
| **structured content** | `{ title, sections[], keyPoints[] }` | Engine stage 1 → stage 2 |
| **rendered graphic** | image asset · PNG · 1080×1350 | Engine → Frontend (Flow B) |
| **generation record** | `{ id, userId, assetUrl, sourceText, style, ratio, createdAt }` | Frontend → Storage (Flow D) |
| **graphics list** | `graphics[]`, newest first | Storage → Frontend (Flow D) |

> Field names are indicative for P2/P4, not a locked schema — the schema is decided in **P4**. They exist here so the generation contract between Frontend, Engine, and Storage is explicit before stack selection.

---

**Phase gate:** on approval, `PRD.md` + `AppFlow.md` complete → **P1 done**, ready for **P2 · TechStack**.

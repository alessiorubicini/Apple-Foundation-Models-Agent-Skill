# AGENTS.md — Foundation Models Skill: Agent Author Guide

This document defines the content contract for `apple-foundation-models-skill`.  
Read this before contributing or modifying any reference file.

---

## Purpose

This skill enhances agent reasoning about Apple's `FoundationModels` framework by providing:

1. **Accurate API surface** — no hallucinated types, methods, or parameters.
2. **Correctness invariants** — rules the agent must enforce in generated code.
3. **Decision trees** — routing logic so the agent selects the right reference file.
4. **Boundary enforcement** — explicit list of what this skill does NOT cover.

---

## How This Skill Enhances Agent Reasoning

When a user asks the agent to write, review, or debug `FoundationModels` code, the agent:

1. Reads `SKILL.md` to load the mental model and active invariants.
2. Consults `references/_index.md` to identify which reference files apply.
3. Pulls the relevant reference(s) for the specific API surface involved.
4. Applies the correctness invariants before emitting any code.

This layered approach keeps `SKILL.md` token-efficient (fast to load) while ensuring deep accuracy for complex tasks (reference files loaded on demand).

---

## Content Boundaries

### This skill covers

- All public API in the `FoundationModels` framework (iOS 26.0+)
- Concurrency patterns *specific to* `LanguageModelSession` and `Tool`
- Integration with SwiftUI `@State`, `@Observable`, and `.task`
- Error handling for `LanguageModelSession.GenerationError`
- Performance patterns unique to on-device LLM inference

### This skill does NOT cover

- General Swift Concurrency → use the [Swift Concurrency Skill](https://github.com/AvdLee/Swift-Concurrency-Agent-Skill)
- General SwiftUI patterns → use the [SwiftUI Expert Skill](https://github.com/AvdLee/SwiftUI-Agent-Skill)
- CoreML, Create ML, or Vision framework
- Server-side / Private Cloud Compute (PCC) endpoints
- Third-party LLM SDKs (OpenAI, Anthropic, etc.)
- LoRA adapter training (Python toolkit — not the Swift API)

---

## Correctness Invariants

These rules must be enforced in every code output:

| # | Invariant |
|---|---|
| 1 | Always gate on `SystemLanguageModel.default.availability` before creating a session. |
| 2 | `LanguageModelSession` must be `@State` or actor-isolated — never a local `let` in a button action. |
| 3 | Every `session.respond(to:)` / `session.streamResponse(to:)` call must be inside `do/catch` for `GenerationError`. |
| 4 | Handle `.exceededContextWindowSize` explicitly — create a fresh session or truncate transcript. |
| 5 | `@Generable` types must be `struct` or `enum`. Classes are not supported. |
| 6 | `@Guide` constraints must match the property's type (`String` → `.regex`, `Int` → `.count`, enum → `.anyOf`). |
| 7 | `Tool.call(arguments:)` is `async throws` — always propagate errors; never `try?` silently. |
| 8 | Never block the main actor — use `Task { }` or `.task { }` for all session calls. |
| 9 | Call `session.prewarm()` during idle time (e.g., `onAppear`), not immediately before `respond`. |
| 10 | Combined context (input + output) is capped at **4 096 tokens**. Design prompts and expected outputs accordingly. |

---

## Prohibited Patterns

Do not emit code containing:

```swift
// ❌ No availability check
let session = LanguageModelSession()
let r = try! session.respond(to: prompt) // force-try
```

```swift
// ❌ Blocking main thread
let response = try await session.respond(to: prompt) // called directly in View.body
```

```swift
// ❌ Class with @Generable (unsupported)
@Generable class Recipe { ... }
```

---

## Tone & Style

- **Apple-Engineering tone**: prescriptive, dense, no filler.
- All code in **Swift 6**. Use `isolated` parameters, `actor`, and structured concurrency.
- Prefer `for try await partial in stream { }` over callback-based patterns.
- Use `@MainActor` on view models; use `actor` for non-UI managers.
- Do not suggest `DispatchQueue.main.async` — it is superseded by Swift Concurrency.

---

## Adding New Reference Files

1. Place the file in `apple-foundation-models-skill/references/`.
2. Add an entry to `references/_index.md` with a one-line description.
3. Add a trigger condition in `SKILL.md` under **Reference Map**.
4. All claims must cite an Apple documentation URL or WWDC session.  
   Uncitable claims → write: _"Technical details pending official release."_

---

## Maintenance

Run the meta-skill after new iOS / Xcode releases:

```
Use the update-foundation-apis skill and scan for deprecated FoundationModels APIs in iOS 26.5+.
```

See `.agents/skills/update-foundation-apis.md` for the automated workflow.
---
name: apple-foundation-models-skill
description: Write, review, or integrate Apple's on-device FoundationModels framework (iOS 26.0+, macOS 26.0+). Use when building generative AI features, structured data extraction, tool calling, or streaming text generation natively on Apple Silicon devices.
---

# Apple Foundation Models Skill

## Operating Rules

- Consult `references/_index.md` at the start of every task to navigate the framework's capabilities.
- Always check model availability (`SystemLanguageModel.default.availability`) before attempting to initialize a session; gracefully handle unsupported hardware or missing downloads.
- Strictly adhere to Swift 6 concurrency rules: `LanguageModelSession` must be explicitly isolated (`@State`, `@MainActor`, or `actor`), and `Tool` conformances must be `Sendable`.
- Respect the strict 4 096-token context limit per session; design proactive recovery strategies for long conversations.
- Prefer `@Generable` for structured output instead of asking the model to write raw JSON.
- Never use the model for real-time data retrieval without injecting `Tool` capabilities, and avoid using it for complex mathematical reasoning or authoritative world-knowledge.
- Focus on hardware performance: use `prewarm()` intelligently during idle time to minimize first-token latency.

## Task Workflow

### Review existing AI integration code
- Read the code under review and identify which topics apply.
- Flag any missing availability checks before `LanguageModelSession` instantiation.
- Validate that `.exceededContextWindowSize` is explicitly caught and handled.
- Ensure `LanguageModelSession` is not declared locally inside a function or `Task` (which breaks statefulness).
- Check `Tool` implementations to ensure errors are propagated (`throws`) and not silenced with `try?`.

### Implement new Foundation Models feature
- Determine the correct model adapter: `.default` for conversational prose, `.contentTagging` for classification/extraction.
- Design data flow: choose between monolithic `respond(to:)` or real-time `streamResponse(to:)`.
- Define `@Generable` structs for structured data extraction, utilizing `@Guide` to strictly constrain output token by token.
- Build instructions safely, keeping them developer-controlled and strictly separated from user input.

### Implement Tool Calling
- Define the `Tool` protocol conformance, ensuring a clear, concise `description` for the model.
- Define `Arguments` using `@Generable` types.
- Implement the `call(arguments:)` method, ensuring network calls or database queries are properly awaited and errors are correctly thrown back to the model.

### Topic Router

Consult the reference file for each topic relevant to the current task:

| Topic | Reference |
|-------|-----------|
| Core Models & Availability | `references/system-language-model.md` |
| Session & Transcript Lifecycle | `references/session-lifecycle.md` |
| Structured Output & `@Generable` | `references/guided-generation.md` |
| Expanding capabilities (`Tool`) | `references/tool-calling.md` |
| Temperature & Token Limits | `references/generation-options.md` |
| Real-time UI & Streams | `references/streaming.md` |
| Context Overflow & Fallbacks | `references/error-handling.md` |
| Actor Isolation & Sendable | `references/concurrency.md` |
| Memory, Prewarming & Optimization | `references/performance.md` |
| Framework Terminology | `references/glossary.md` |

## Correctness Checklist

These are hard rules — violations will cause runtime crashes, deadlocks, or broken state:

- [ ] `SystemLanguageModel.default.availability` is checked before creating any session.
- [ ] `LanguageModelSession` is explicitly owned by `@State` or an `actor` (never instantiated locally inside a function).
- [ ] `LanguageModelSession.GenerationError.exceededContextWindowSize` is explicitly caught in all `do/catch` blocks interacting with the session.
- [ ] A session is **never** reused after throwing an `.exceededContextWindowSize` error (a fresh instance must be created).
- [ ] `prewarm()` is called during idle time (e.g., view `.task`), never immediately preceding a `respond(to:)` call.
- [ ] Errors inside `Tool.call(arguments:)` are explicitly thrown and never silenced with `try?`.
- [ ] Instructions strings are strictly hardcoded or developer-controlled, never built directly from user input.
- [ ] `@Generable` properties are ordered logically top-to-bottom, with summary/dependent properties placed last.
- [ ] `PartiallyGenerated` types are never instantiated manually, only consumed from `streamResponse`.

## References

- `references/_index.md` — **Read first for quick navigation.** Index of all documentation.
- `references/system-language-model.md` — Availability states, adapter types (`.default`, `.contentTagging`), and hardware requirements.
- `references/session-lifecycle.md` — Initialization, system instructions, transcript management, and statefulness.
- `references/guided-generation.md` — `@Generable` macros, `@Guide` token constraints, and dynamic schema building.
- `references/tool-calling.md` — `Tool` protocol design, `Sendable` conformance, and `ToolExecutionDelegate`.
- `references/generation-options.md` — Temperature tuning, `.greedy` vs `.random` sampling, and response token capping.
- `references/streaming.md` — `streamResponse(to:)` logic, `PartiallyGenerated` handling for SwiftUI incremental updates.
- `references/error-handling.md` — Mandatory recovery strategies for context overflow and unsupported locales.
- `references/concurrency.md` — Strict Swift 6 isolation invariants, `@MainActor` UI patterns, and cross-actor session usage.
- `references/performance.md` — KV-cache limits, 4096-token budgets, 1.2 GB RAM footprint, and latency reduction via `prewarm()`.
- `references/glossary.md` — Canonical definitions for terms like "LoRA", "adapter", and "transcript".
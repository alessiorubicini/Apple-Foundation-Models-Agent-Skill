# References Index — Apple Foundation Models Skill

Quick navigation for the agent. Load the specific file matching the current task.

| File | One-line description |
|---|---|
| `system-language-model.md` | `SystemLanguageModel`: availability states, `.default`, `.contentTagging`, device requirements |
| `session-lifecycle.md` | `LanguageModelSession` init, instructions, transcript, `isResponding`, `prewarm()` |
| `guided-generation.md` | `@Generable`, `@Guide`, nested types, enums, `DynamicGenerationSchema`, `includeSchemaInPrompt` |
| `tool-calling.md` | `Tool` protocol, `ToolOutput`, `ToolExecutionDelegate`, `@Generable Arguments` |
| `generation-options.md` | `GenerationOptions`, `SamplingMode` (`.greedy`, `.random`), `temperature`, `maximumResponseTokens` |
| `streaming.md` | `streamResponse`, `ResponseStream`, `PartiallyGenerated`, incremental SwiftUI updates |
| `error-handling.md` | `GenerationError` cases: `.exceededContextWindowSize`, `.unsupportedLanguageOrLocale`, recovery |
| `concurrency.md` | `@MainActor`, `actor`, `Task`, `.task` modifier, isolation invariants |
| `performance.md` | `prewarm()`, 4096-token budget, ~1.2 GB RAM, latency targets, Instruments |
| `prompting-techniques.md` | On-device prompt design: clarity, roles, few-shot examples, reasoning fields, code-side branching |
| `glossary.md` | Canonical definitions: token, transcript, adapter, guided generation, guardrails, LoRA |
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
| `wwdc-2026-beta-apis.md` | WWDC 2026 Beta source matrix, Apple documentation URLs, and routing |
| `prompt-attachments.md` | WWDC 2026 Beta: image attachments, attachment labels, `ImageReference`, transcript resolution |
| `dynamic-profiles.md` | WWDC 2026 Beta: `DynamicInstructions`, `DynamicProfile`, profiles, modifiers, lifecycle hooks |
| `session-properties.md` | WWDC 2026 Beta: `SessionPropertyKey`, `SessionPropertyValues`, `SessionPropertyEntry()`, `SessionProperty` |
| `private-cloud-compute.md` | WWDC 2026 Beta: `PrivateCloudComputeLanguageModel`, entitlement, quota, context size, locale and PCC errors |
| `custom-language-model-provider.md` | WWDC 2026 Beta: `LanguageModel`, capabilities, executors, generation request/channel events |
| `glossary.md` | Canonical definitions: token, transcript, adapter, guided generation, guardrails, LoRA |

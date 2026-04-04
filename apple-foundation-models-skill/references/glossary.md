# Glossary — Foundation Models

Source: `FoundationModels` framework, iOS 26.0+

Canonical terminology used across this skill. Prefer these definitions when writing instructions or tools for agents.

| Term | Definition |
|------|------------|
| **Adapter** | A `SystemLanguageModel` configuration (e.g. `.default`, `.contentTagging`) that selects model behavior for a task class. |
| **Context window** | The maximum number of tokens (combined input and output) a single session turn can hold; on-device models enforce a strict cap (see `performance.md`). |
| **Generable** | A macro-driven type whose structure constrains model output tokens for guided decoding. |
| **Guided generation** | Output shaped by `@Generable` / `@Guide` so the runtime enforces structure instead of parsing free-form text. |
| **Guardrails** | Platform and API constraints (availability, locale, context limits) that require explicit handling in app code. |
| **LoRA** | Low-rank adaptation; optional fine-tuning/adapters shipped or applied outside this skill’s Swift API scope—see Apple docs for adapter availability. |
| **Session transcript** | Ordered history of user/model/tool messages the `LanguageModelSession` retains for stateful turns. |
| **Token** | Atomic unit of model context; prompts and completions consume tokens toward the session budget. |
| **Tool** | A `Tool` protocol implementation the model can invoke for real-time or local data, with `@Generable` arguments and `async throws` execution. |
| **Transcript** | See **Session transcript**. |

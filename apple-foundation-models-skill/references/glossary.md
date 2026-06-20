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

## WWDC 2026 Beta Terms

WWDC 2026 Beta: APIs require Xcode 27 beta and iOS 27.0 / macOS 27.0 / visionOS 27.0 / watchOS 27.0 beta unless noted. Always guard usage with `@available` or `#available` and verify against current Apple documentation before shipping.

| Term | Definition |
|------|------------|
| **ContextOptions** | Beta options that affect prompt construction, including schema inclusion and reasoning level. |
| **Custom language model provider** | A `LanguageModel` plus `LanguageModelExecutor` implementation that lets a non-system model run through `LanguageModelSession`. |
| **Dynamic instructions** | Instructions and tools expressed as `DynamicInstructions`, reevaluated from session state. |
| **Dynamic profile** | A `LanguageModelSession.DynamicProfile` composition that can apply dynamic instructions, model settings, tool policy, history transforms, and lifecycle hooks. |
| **Image reference** | A generated `ImageReference` that points back to a labeled prompt image and resolves through the session transcript. |
| **PCC** | Private Cloud Compute; represented in FoundationModels by `PrivateCloudComputeLanguageModel`. |
| **Reasoning level** | `ContextOptions.ReasoningLevel`; beta control for reasoning effort separate from response token sampling. |
| **Session property** | Typed session-scoped state stored in `SessionPropertyValues` and read through `LanguageModelSession.SessionProperty`. |
| **Tool calling mode** | `GenerationOptions.ToolCallingMode`; beta policy for allowed, required, or disallowed tool calls. |

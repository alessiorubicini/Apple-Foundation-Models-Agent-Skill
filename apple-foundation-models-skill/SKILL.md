---
name: apple-foundation-models-skill
description: Write, review, or integrate Apple's FoundationModels framework (iOS 26.0+, macOS 26.0+), including WWDC 2026 beta APIs (OSs 27.0+). Use for on-device generation, structured output, tool calling, streaming, prompt attachments, dynamic profiles, agentic orchestration, session properties, Private Cloud Compute, or custom language model providers.
---

# Apple Foundation Models Skill

This skill acts as a router. When handling tasks or queries related to Apple's `FoundationModels` framework, identify the relevant topic below and load the corresponding reference file before performing any code generation or analysis.

## Topic Router

| Topic / API Surface | Target Reference |
| :--- | :--- |
| **All References Index** | [references/_index.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/_index.md) |
| **Model Access & Availability** | [system-language-model.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/system-language-model.md) |
| **Session & Transcript Lifecycle** | [session-lifecycle.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/session-lifecycle.md) |
| **Structured Output (`@Generable`)** | [guided-generation.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/guided-generation.md) |
| **Tool Calling & Capabilities** | [tool-calling.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/tool-calling.md) |
| **Sampling & Options** | [generation-options.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/generation-options.md) |
| **Streaming UI & Responses** | [streaming.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/streaming.md) |
| **Error Handling & Overflow** | [error-handling.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/error-handling.md) |
| **Swift Concurrency & Actor Isolation** | [concurrency.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/concurrency.md) |
| **Performance & Prewarming** | [performance.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/performance.md) |
| **Prompt Engineering & Strategy** | [prompting-techniques.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/prompting-techniques.md) |
| **Prompt Attachments (Images)** | [prompt-attachments.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/prompt-attachments.md) |
| **Dynamic Profiles & Policies** | [dynamic-profiles.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/dynamic-profiles.md) |
| **Session Properties** | [session-properties.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/session-properties.md) |
| **Private Cloud Compute (PCC)** | [private-cloud-compute.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/private-cloud-compute.md) |
| **Custom Language Model Providers** | [custom-language-model-provider.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/custom-language-model-provider.md) |
| **Framework Glossary** | [glossary.md](file:///Users/alessiorubicini/Developer/projects/Apple-Foundation-Models-Agent-Skill/apple-foundation-models-skill/references/glossary.md) |

## Boundaries (Out of Scope)

Do **NOT** use this skill for:
- General Swift Concurrency (see Swift Concurrency Skill).
- Standard SwiftUI patterns (see SwiftUI Expert Skill).
- Non-Apple/third-party LLM APIs.
- Model training/CoreML integration details.

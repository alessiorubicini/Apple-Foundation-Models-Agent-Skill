# Generation Options — Controlling Model Output

Source: `FoundationModels` framework, iOS 26.0+

---

## Core Concept

While `LanguageModelSession` instructions dictate the persona and constraints, `GenerationOptions` allow you to control the statistical behavior of the token generation. By tweaking the temperature, sampling mode, and token limits, you can adapt the model for either highly deterministic tasks (like data extraction) or creative tasks (like brainstorming).

## `GenerationOptions` Struct

You can pass an optional `GenerationOptions` instance when calling `session.respond` or `session.streamResponse`.

```swift
import FoundationModels

actor NameGenerator {
    private let session: LanguageModelSession

    init?() {
        guard case .available = SystemLanguageModel.default.availability else { return nil }
        session = LanguageModelSession()
    }

    func names() async throws -> String {
        let options = GenerationOptions(
            samplingMode: .random(probabilityThreshold: 0.9, seed: nil),
            temperature: 0.7,
            maximumResponseTokens: 250
        )

        let response = try await session.respond(
            to: "Generate three names for a tech startup.",
            options: options
        )
        return response.content
    }
}
```

### Properties

#### `temperature: Double?`
Temperature influences the confidence of the models response. 
* **Lower values**: Make the output more focused, deterministic, and predictable. Ideal for structured generation (`@Generable`) and fact-retrieval.
* **Higher values**: Introduce more randomness, making the output more creative and varied.

#### `sampling: GenerationOptions.SamplingMode?`
A sampling strategy for how the model picks tokens when generating a response. The `SamplingMode` controls how a token is selected from the probability distribution.
* **`.greedy`**: A sampling mode that always chooses the most likely token. This makes the output highly deterministic.
* **`.random(top: Int, seed: UInt64?)`**: A sampling mode that considers a fixed number of high-probability tokens.
* **`.random(probabilityThreshold: Double, seed: UInt64?)`**: A mode that considers a variable number of high-probability tokens based on the specified threshold.
* **Seed**: Providing a `seed` alongside random sampling ensures that identical prompts with the same seed will generally produce the same output, which is invaluable for testing.

#### `maximumResponseTokens: Int?`
The maximum number of tokens the model is allowed to produce in its response.
* Use this to prevent the model from generating overly long responses that consume unnecessary memory and processing time.
* **Warning**: This limit does not override the hard **4 096 tokens** combined input + output budget. If your session exceeds the available context size, it throws `LanguageModelSession.GenerationError.exceededContextWindowSize`.

---

## Best Practices & Invariants

* **Extraction vs. Creation**: When using the `.contentTagging` use-case adapter or strict `@Generable` types, prefer `.greedy` sampling to enforce structural constraints. For free-form text using the `.default` model, moderate temperatures allow for more natural phrasing.
* **Context Preservation**: Always remember that the combined transcript size is capped at **4 096 tokens**. Limit `maximumResponseTokens` if you need to reserve space in the transcript for future multi-turn interactions.

---

## WWDC 2026 Beta Updates

WWDC 2026 Beta: APIs require iOS 27.0 / macOS 27.0 / visionOS 27.0 / watchOS 27.0 beta unless noted. Verify against current Apple documentation before shipping.

Sources:
- https://developer.apple.com/documentation/foundationmodels/generationoptions
- https://developer.apple.com/documentation/foundationmodels/contextoptions

- Use `samplingMode`; `sampling` is deprecated.
- `GenerationOptions.ToolCallingMode` controls whether tool calls are `.allowed`, `.required`, or `.disallowed`.
- `ContextOptions` controls prompt construction details outside token sampling.
- `ContextOptions.ReasoningLevel` supports `.light`, `.moderate`, `.deep`, and `.custom(String)`.
- For guided generation, prefer `ContextOptions(includeSchemaInPrompt:)` over older overloads that pass `includeSchemaInPrompt` directly.

```swift
import FoundationModels

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
func betaOptions() -> (GenerationOptions, ContextOptions) {
    let generation = GenerationOptions(
        samplingMode: .greedy,
        temperature: 0.1,
        maximumResponseTokens: 200,
        toolCallingMode: .required
    )
    let context = ContextOptions(
        includeSchemaInPrompt: true,
        reasoningLevel: .light
    )
    return (generation, context)
}
```

# LanguageModelSession — Lifecycle

Source: `FoundationModels` framework, iOS 26.0+  
WWDC25: [Deep Dive into the Foundation Models Framework](https://developer.apple.com/videos/play/wwdc2025/301/)

---

## Initializer

```swift
// Minimal — uses SystemLanguageModel.default + default guardrails
let session = LanguageModelSession()

// Full configuration
let session = LanguageModelSession(
    model: SystemLanguageModel(useCase: .contentTagging),
    guardrails: .default,               // Safety filters — always on; cannot be disabled
    tools: [WeatherTool(), SearchTool()],
    instructions: {
        """
        You are a travel assistant. Reply in the user's language.
        Never fabricate hotel names or prices.
        """
    }
)
```

### Parameters

| Parameter | Type | Default | Notes |
|---|---|---|---|
| `model` | `SystemLanguageModel` | `.default` | Use `.contentTagging` for extraction tasks |
| `guardrails` | `LanguageModelSession.Guardrails` | `.default` | Content filtering — always applied |
| `tools` | `[any Tool]` | `[]` | Model decides when to invoke tools |
| `instructions` | `@InstructionsBuilder` closure | none | System prompt; set once per session |

---

## Instructions

Instructions behave like a system prompt — they shape every response in the session without repeating per-call.

```swift
let session = LanguageModelSession {
    """
    You are a senior Swift engineer. Provide concise, correct Swift 6 code.
    Always prefer async/await over completion handlers.
    Do not generate code that requires Objective-C bridging.
    """
}
```

**Security**: Never build instructions from user-provided text. Instructions are injection-sensitive — keep them developer-controlled. Separate user input from developer instructions using the `Prompt` builder (see `guided-generation.md`).

---

## Session Statefulness

A `LanguageModelSession` is stateful. Every `respond(to:)` call is recorded in the **transcript** (all prompts + responses). The model uses the full transcript as context on the next call.

```swift
// Multi-turn conversation — context is preserved automatically
let r1 = try await session.respond(to: "What foods are popular in Japan?")
let r2 = try await session.respond(to: "Suggest restaurants for those.")
// r2 knows r1's context without re-stating it
```

**Context limit**: Combined transcript size is capped at **4 096 tokens**. Exceeding this throws `.exceededContextWindowSize`.

### Transcript Access

```swift
// Debug or display conversation history
for entry in session.transcript {
    switch entry {
    case .prompt(let p): print("User:", p)
    case .response(let r): print("Model:", r)
    }
}
```

---

## `isResponding`

```swift
// Disable send button while model is working
Button("Send") { Task { await viewModel.send() } }
    .disabled(session.isResponding)
```

---

## Prewarming

Loads the model into memory without generating a response. Call during idle time to cut first-token latency by up to ~500 ms.

```swift
.task {
    do {
        await session.prewarm()
    }
}
```

Do **not** call `prewarm()` immediately before `respond()` in the same task — it provides no benefit if generation follows immediately.

---

## Session Ownership Patterns

### SwiftUI — `@State`

```swift
struct RecipeView: View {
    @State private var session = LanguageModelSession {
        "You are a professional chef. Suggest recipes using provided ingredients."
    }
    // ...
}
```

### ViewModel — `@Observable` actor-isolated

```swift
@MainActor
@Observable final class RecipeViewModel {
    private let session = LanguageModelSession {
        "You are a professional chef."
    }
    var recipe: Recipe?

    func generate(ingredients: [String]) async {
        do {
            let response = try await session.respond(
                to: "Suggest a recipe using: \(ingredients.joined(separator: ", "))",
                generating: Recipe.self
            )
            recipe = response.content
        } catch { /* handle */ }
    }
}
```

### Dedicated actor (non-UI)

```swift
actor AIEngine {
    private let session = LanguageModelSession()

    func classify(_ text: String) async throws -> Category {
        let response = try await session.respond(
            to: text,
            generating: Category.self
        )
        return response.content
    }
}
```

---

## Context Overflow Recovery

```swift
do {
    let r = try await session.respond(to: prompt)
} catch LanguageModelSession.GenerationError.exceededContextWindowSize {
    // Strategy 1: Fresh session (loses history)
    session = LanguageModelSession { /* same instructions */ }

    // Strategy 2: Summarize transcript, inject as new instructions
    // (Advanced — build summary from session.transcript manually)
}
```

Never retry the same session after `.exceededContextWindowSize` — the transcript is permanently full.

---

## WWDC 2026 Beta Updates

WWDC 2026 Beta: APIs require Xcode 27 beta and iOS 27.0 / macOS 27.0 / visionOS 27.0 / watchOS 27.0 beta unless noted. Always guard usage with `@available` or `#available` and verify against current Apple documentation before shipping.

Sources:
- https://developer.apple.com/documentation/foundationmodels/languagemodelsession
- https://developer.apple.com/documentation/foundationmodels/dynamicinstructions
- https://developer.apple.com/documentation/foundationmodels/transcripterrorhandlingpolicy
- Apple WWDC 2026 session: Build agentic app experiences with the Foundation Models framework

- Sessions can be initialized with any `LanguageModel`, including `PrivateCloudComputeLanguageModel` and custom providers.
- `init(profile:history:)` creates a session from a `DynamicProfile`.
- `init(model:dynamicInstructions:history:)` creates a session from a model plus `DynamicInstructions`.
- `properties` exposes `SessionPropertyValues` for dynamic instructions, tools, and profiles.
- `transcriptErrorHandlingPolicy` controls whether failed requests preserve or revert transcript mutations.
- Response and stream overloads accept `contextOptions` and `metadata`.
- `transcript` is mutable in beta; mutate it only when `session.isResponding == false`.
- With `.preserveTranscript`, failed or canceled responses can leave partial transcript state. Normalize or roll back manually before reusing the session.

```swift
import FoundationModels

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
actor MetadataSession {
    private let session: LanguageModelSession

    init?() {
        guard case .available = SystemLanguageModel.default.availability else { return nil }
        session = LanguageModelSession {
            "Answer in one paragraph."
        }
        session.transcriptErrorHandlingPolicy = .revertTranscript
    }

    func ask(_ text: String) async throws -> String {
        do {
            let response = try await session.respond(
                to: Prompt(text),
                contextOptions: ContextOptions(reasoningLevel: .light),
                metadata: ["surface": "settings"]
            )
            return response.content
        } catch LanguageModelError.contextSizeExceeded {
            throw LanguageModelError.contextSizeExceeded(
                .init(contextSize: 0, tokenCount: 0, debugDescription: "Create a fresh session.")
            )
        }
    }
}
```

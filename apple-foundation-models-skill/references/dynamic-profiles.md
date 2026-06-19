# Dynamic Profiles — WWDC 2026 Beta

WWDC 2026 Beta: APIs require iOS 27.0 / macOS 27.0 / visionOS 27.0 / watchOS 27.0 beta unless noted. Verify against current Apple documentation before shipping.

Sources:
- https://developer.apple.com/documentation/foundationmodels/dynamicinstructions
- https://developer.apple.com/documentation/foundationmodels/languagemodelsession/dynamicprofile
- https://developer.apple.com/documentation/foundationmodels/languagemodelsession/dynamicprofilemodifier
- https://developer.apple.com/documentation/foundationmodels/languagemodelsession/profile
- https://developer.apple.com/documentation/foundationmodels/languagemodelsession

## API Surface

- `DynamicInstructions` describes instructions and tool availability that can be recomputed from session state.
- `DynamicInstructionsBuilder` accepts `Instructions`, tools, arrays of tools, conditionals, optionals, and `DynamicInstructions.ForEach`.
- `LanguageModelSession.Profile` wraps dynamic instructions into a profile.
- `LanguageModelSession.DynamicProfile` composes profiles and profile modifiers.
- `LanguageModelSession.DynamicProfileModifier` creates reusable wrappers for profile content.
- `LanguageModelSession(profile:history:)` creates a session from a dynamic profile.
- `LanguageModelSession(model:dynamicInstructions:history:)` creates a session from a model and dynamic instructions.

## Profile Modifiers

Use profile modifiers to adjust invocation behavior without rebuilding the whole session:

- `model(_:)`
- `temperature(_:)`
- `samplingMode(_:)`
- `maximumResponseTokens(_:)`
- `reasoningLevel(_:)`
- `toolCallingMode(_:)`
- `historyTransform(_:)`
- `transcriptErrorHandlingPolicy(_:)`
- `onPrompt(perform:)`
- `onResponse(perform:)`
- `onToolCall(perform:)`
- `onToolOutput(perform:)`
- `onActivate(perform:)`
- `onDeactivate(perform:)`

`inputFilter(_:)` exists in the beta SDK but is deprecated in favor of `historyTransform(_:)`.

## Example

```swift
import FoundationModels

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
extension SessionPropertyValues {
    @SessionPropertyEntry
    var customerTier: String = "standard"
}

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
struct SupportProfile: LanguageModelSession.DynamicProfile {
    @SessionProperty(\.customerTier) private var customerTier

    var body: some LanguageModelSession.DynamicProfile {
        LanguageModelSession.Profile {
            Instructions("Answer as a concise support assistant for a \(customerTier) customer.")
        }
        .temperature(0.2)
        .reasoningLevel(.light)
        .transcriptErrorHandlingPolicy(.revertTranscript)
        .onPrompt {
            // Hook point for analytics or UI state; never block here.
        }
    }
}

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
actor SupportAgent {
    private let session: LanguageModelSession

    init?() {
        guard case .available = SystemLanguageModel.default.availability else { return nil }
        session = LanguageModelSession(profile: SupportProfile())
        session.properties.customerTier = "enterprise"
    }

    func answer(_ text: String) async throws -> String {
        do {
            let response = try await session.respond(to: Prompt(text))
            return response.content
        } catch LanguageModelError.contextSizeExceeded {
            throw LanguageModelError.contextSizeExceeded(
                .init(contextSize: 0, tokenCount: 0, debugDescription: "Start a fresh session before retrying.")
            )
        }
    }
}
```

## Correctness Rules

- Use `LanguageModelSession(profile:)` or `LanguageModelSession(model:dynamicInstructions:)` only after model availability is known.
- Keep mutable profile inputs in `SessionPropertyValues`; do not rebuild instructions from uncontrolled user input.
- Prefer `historyTransform(_:)` for transcript trimming or redaction before model invocation.
- Avoid heavy work in lifecycle hooks. Start asynchronous app work elsewhere and keep hooks short.
- If a profile can require tools, set `toolCallingMode(_:)` explicitly when "must call a tool" is part of correctness.

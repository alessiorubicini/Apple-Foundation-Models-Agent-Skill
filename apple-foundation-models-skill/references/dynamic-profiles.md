# Dynamic Profiles — WWDC 2026 Beta

WWDC 2026 Beta: APIs require Xcode 27 beta and iOS 27.0 / macOS 27.0 / visionOS 27.0 / watchOS 27.0 beta unless noted. Always guard usage with `@available` or `#available` and verify against current Apple documentation before shipping.

Sources:
- https://developer.apple.com/documentation/foundationmodels/dynamicinstructions
- https://developer.apple.com/documentation/foundationmodels/languagemodelsession/dynamicprofile
- https://developer.apple.com/documentation/foundationmodels/languagemodelsession/dynamicprofilemodifier
- https://developer.apple.com/documentation/foundationmodels/languagemodelsession/profile
- https://developer.apple.com/documentation/foundationmodels/languagemodelsession
- Apple WWDC 2026 session: Build agentic app experiences with the Foundation Models framework

## API Surface

- `DynamicInstructions` describes instructions and tool availability that can be recomputed from session state.
- `DynamicInstructionsBuilder` accepts `Instructions`, tools, arrays of tools, conditionals, optionals, and `DynamicInstructions.ForEach`.
- `LanguageModelSession.Profile` wraps dynamic instructions into a profile.
- `LanguageModelSession.DynamicProfile` composes profiles and profile modifiers.
- `LanguageModelSession.DynamicProfileModifier` creates reusable wrappers for profile content.
- `LanguageModelSession(profile:history:)` creates a session from a dynamic profile.
- `LanguageModelSession(model:dynamicInstructions:history:)` creates a session from a model and dynamic instructions.
- `DynamicProfile.body` is re-evaluated before each prompt; profile state changes can switch active instructions, tools, model, and options without rebuilding the session.

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

## Agentic Patterns

- Baton-pass: profiles share one transcript; a tool changes the active profile state, and the receiving profile completes the final response.
- Phone-a-friend: a tool creates a short-lived `LanguageModelSession` with its own profile and isolated transcript, returns its result as tool output, and the parent profile gives the final response.
- FoundationModelsUtilities is an optional open-source package for reusable dynamic-profile components such as history-window modifiers, completed-tool-call dropping, and procedural `Skills` dynamic instructions. Verify package API before emitting code.

## Transcript Management

- Prefer `historyTransform(_:)` for per-profile trimming or redaction; it is a lossless pre-prompt view and does not permanently mutate `session.transcript`.
- Use `@SessionProperty(\.history)` only when intentionally mutating history for every profile in the session, such as after-response summarization.

## Example: Three Profiles in One Session

```swift
import FoundationModels

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
enum CraftMode: String, Sendable {
    case brainstorm
    case plan
    case review
}

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
extension SessionPropertyValues {
    @SessionPropertyEntry
    var craftMode: CraftMode = .brainstorm
}

@Generable
struct EmptyToolArguments {}

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
struct SwitchCraftModeTool: Tool {
    let target: CraftMode
    var name: String { "switch_to_\(target.rawValue)" }
    var description: String { "Switches the craft assistant to \(target.rawValue) mode." }
    @SessionProperty(\.craftMode) private var craftMode

    func call(arguments: EmptyToolArguments) async throws -> String {
        craftMode = target
        return "Mode switched to \(target.rawValue)."
    }
}

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
struct CraftProfile: LanguageModelSession.DynamicProfile {
    @SessionProperty(\.craftMode) private var mode
    private let pccModel = PrivateCloudComputeLanguageModel()
    private let systemModel = SystemLanguageModel.default

    var body: some LanguageModelSession.DynamicProfile {
        switch mode {
        case .brainstorm:
            LanguageModelSession.Profile {
                Instructions("Generate a short list of craft project ideas.")
                SwitchCraftModeTool(target: .plan)
            }
            .model(pccModel)
            .temperature(1.0)
        case .plan:
            LanguageModelSession.Profile {
                Instructions("Write step-by-step directions for the selected project.")
                SwitchCraftModeTool(target: .review)
            }
            .model(pccModel)
            .reasoningLevel(.deep)
        case .review:
            LanguageModelSession.Profile {
                Instructions("Give concise technique feedback for in-progress photos.")
                SwitchCraftModeTool(target: .brainstorm)
            }
            .model(systemModel)
            .historyTransform { Array($0.suffix(12)) }
        }
    }
}

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
actor CraftAgent {
    private let session: LanguageModelSession

    init?() {
        guard case .available = SystemLanguageModel.default.availability else { return nil }
        session = LanguageModelSession(profile: CraftProfile())
    }
}
```

## Correctness Rules

- Use `LanguageModelSession(profile:)` or `LanguageModelSession(model:dynamicInstructions:)` only after model availability is known.
- Keep mutable profile inputs in `SessionPropertyValues`; do not rebuild instructions from uncontrolled user input.
- Prefer `historyTransform(_:)` for transcript trimming or redaction before model invocation.
- Avoid heavy work in lifecycle hooks. Start asynchronous app work elsewhere and keep hooks short.
- If a profile can require tools, set `toolCallingMode(_:)` explicitly when "must call a tool" is part of correctness.

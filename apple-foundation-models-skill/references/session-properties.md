# Session Properties — WWDC 2026 Beta

WWDC 2026 Beta: APIs require iOS 27.0 / macOS 27.0 / visionOS 27.0 / watchOS 27.0 beta unless noted. Verify against current Apple documentation before shipping.

Sources:
- https://developer.apple.com/documentation/foundationmodels/sessionpropertykey
- https://developer.apple.com/documentation/foundationmodels/sessionpropertyvalues
- https://developer.apple.com/documentation/foundationmodels/sessionpropertyentry()
- https://developer.apple.com/documentation/foundationmodels/languagemodelsession/sessionproperty
- https://developer.apple.com/documentation/foundationmodels/tool

## API Surface

- `SessionPropertyKey` defines a typed key with a `defaultValue`.
- `SessionPropertyEntry()` is the public macro for adding typed entries to `SessionPropertyValues`.
- `SessionPropertyValues` stores session-scoped values and exposes subscript access for custom keys.
- `SessionPropertyValues.history` exposes transcript history.
- `SessionPropertyValues.rootDynamicInstructions` exposes root dynamic instructions.
- `LanguageModelSession.SessionProperty` is a property wrapper over a key path into `SessionPropertyValues`.
- `SessionProperty` is typealiased on `DynamicInstructions`, `Tool`, `DynamicProfile`, and `DynamicProfileModifier`.

## Tool Access

Use session properties when tools need session-local app state. Do not use globals or singleton state for per-session identity.

```swift
import FoundationModels

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
extension SessionPropertyValues {
    @SessionPropertyEntry
    var accountID: String = "anonymous"
}

@Generable
struct LookupArguments {
    let query: String
}

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
struct AccountLookupTool: Tool {
    @SessionProperty(\.accountID) private var accountID

    let description = "Looks up account-scoped help content."

    func call(arguments: LookupArguments) async throws -> String {
        "Account \(accountID): \(arguments.query)"
    }
}

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
actor AccountAgent {
    private let session: LanguageModelSession

    init?() {
        guard case .available = SystemLanguageModel.default.availability else { return nil }
        session = LanguageModelSession(tools: [AccountLookupTool()]) {
            "Use account-scoped tools only when current account context is needed."
        }
        session.properties.accountID = "A-123"
    }
}
```

## Dynamic Instruction Access

Session properties are read when dynamic instructions or profiles are evaluated.

```swift
import FoundationModels

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
extension SessionPropertyValues {
    @SessionPropertyEntry
    var localeHint: String = "en-US"
}

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
struct LocalizedInstructions: DynamicInstructions {
    @SessionProperty(\.localeHint) private var localeHint

    var body: some DynamicInstructions {
        Instructions("Prefer \(localeHint) spelling and formatting.")
    }
}
```

## Correctness Rules

- Define session property entries in `SessionPropertyValues` extensions with `@SessionPropertyEntry`.
- Keep values `Sendable` when accessed across session/tool/profile boundaries.
- Set session properties immediately after session creation and before the first request that depends on them.
- Do not store secrets or long documents in session properties; they can affect prompt construction and token usage.
- Prefer session properties over rebuilding a `LanguageModelSession` when only runtime profile state changes.

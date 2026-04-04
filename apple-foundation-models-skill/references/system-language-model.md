# SystemLanguageModel

Source: `FoundationModels` framework, iOS 26.0+  
WWDC25: [Meet the Foundation Models framework](https://developer.apple.com/videos/play/wwdc2025/286/)

---

## Access Points

```swift
import FoundationModels

// General-purpose text generation (default)
let model = SystemLanguageModel.default

// Specialized adapter: content tagging & entity extraction
let tagger = SystemLanguageModel(useCase: .contentTagging)
```

`SystemLanguageModel.default` powers all general text tasks (Q&A, summarization, content generation, game dialog). The `.contentTagging` adapter is fine-tuned for topic extraction, entity detection, and classification — its output is structurally different from the default model.

---

## Availability

Always check availability before creating a session. The model may be unavailable for hardware, settings, or readiness reasons.

```swift
switch SystemLanguageModel.default.availability {
case .available:
    // Proceed to create LanguageModelSession
case .unavailable(let reason):
    switch reason {
    case .appleIntelligenceNotEnabled:
        // Deep-link to Settings > Apple Intelligence
    case .deviceNotEligible:
        // Device does not support Apple Intelligence
    case .modelNotReady:
        // Model downloading or initializing — retry after delay
    @unknown default:
        // Future-proof: show generic unavailable UI
    }
}
```

**Rule**: Never create a `LanguageModelSession` without an `.available` guard. Doing so throws at runtime.

---

## Supported Devices (as of iOS 26.0)

- iPhone 15 Pro, iPhone 15 Pro Max
- iPhone 16 series (all)
- iPad with M1 chip or later
- Mac with Apple Silicon (M1 or later)
- Apple Vision Pro

Minimum OS: iOS 26.0, macOS 26.0 (Tahoe), visionOS 3.0.  
Apple Intelligence must be enabled in **Settings > Apple Intelligence & Siri**.

---

## Supported Languages

The model supports a growing set of languages. As of the 2025 update: English plus 14 additional languages (see Apple ML Research blog). Always check `SystemLanguageModel.default.supportedLanguages` at runtime — do not hardcode the list.

```swift
let supported = SystemLanguageModel.default.supportedLanguages
guard supported.contains(Locale.current.language) else {
    // Show language-not-supported message
    return
}
```

---

## Use-Case Adapters

| Adapter | Best for |
|---|---|
| `.default` | Free-form generation, Q&A, summarization, creative content, dialog |
| `.contentTagging` | Topic tagging, entity extraction, classification, semantic labeling |

Do not use `.contentTagging` for free-form generation — it produces structured, terse outputs optimized for extraction, not prose.

---

## What the Model Is NOT For

Apple explicitly warns against these use cases:
- Code generation (use Xcode intelligence or server models)
- Complex mathematical reasoning
- Authoritative world-knowledge retrieval (model is not a web search)
- Real-time data (use `Tool` calling to inject live data)
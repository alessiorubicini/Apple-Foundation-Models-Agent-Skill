# Error Handling — GenerationError and Recovery

Source: `FoundationModels` framework, iOS 26.0+

---

## Core Concept

Interacting with on-device AI involves non-deterministic execution, token limits, and strict hardware requirements. Robust error handling is not optional. Every call to `session.respond(to:)` or `session.streamResponse(to:)` must live within a `do/catch` block, explicitly catching `LanguageModelSession.GenerationError`.



---

## Pre-Session: Availability Gate

Before even creating a `LanguageModelSession`, you must verify that the model is available. Failing to do so will result in runtime errors if the device is unsupported, Apple Intelligence is disabled, or the model is still downloading.

```swift
import FoundationModels
import SwiftUI

// 1. Always check availability first
switch SystemLanguageModel.default.availability {
case .available:
    // Safe to proceed
    break
case .unavailable(let reason):
    // Map .unavailable(reason) to ContentUnavailableView
    switch reason {
    case .appleIntelligenceNotEnabled:
        print("Prompt user to enable Apple Intelligence in Settings")
    case .deviceNotEligible:
        print("Device lacks required hardware (e.g., A17 Pro/M1+)")
    case .modelNotReady:
        print("Model is downloading or initializing")
    @unknown default:
        print("Unknown unavailability reason")
    }
}
```

---

## Catching `GenerationError`

When interacting with an active session, you must handle specific generation failures. The most critical of these is context overflow.

```swift
do {
    let response = try await session.respond(to: userInput)
    // Handle successful response
} catch LanguageModelSession.GenerationError.exceededContextWindowSize {
    // See Context Overflow Recovery below
} catch LanguageModelSession.GenerationError.unsupportedLanguageOrLocale {
    // See Language Support below
} catch {
    // Handle generic or tool-execution errors
    print("Generation failed: \(error.localizedDescription)")
}
```

### 1. Context Overflow (`.exceededContextWindowSize`)

**The Rule**: The model has a combined input + output context window of exactly **4 096 tokens**. Every prompt and response in a `LanguageModelSession` is recorded in its stateful transcript, consuming this budget.

**The Invariant**: You must handle `.exceededContextWindowSize` by creating a fresh `LanguageModelSession`. **Never** retry the same session instance, as its transcript is permanently full and will immediately fail again.

```swift
// Strategy: Transparently start a new session (loses conversation history)
func handleOverflow() {
    // Re-initialize with the exact same developer instructions
    self.session = LanguageModelSession {
        "You are a helpful assistant. Reply concisely."
    }
    
    // Note: To preserve context across resets, you would need to manually 
    // summarize the old session.transcript and inject it into the new 
    // session's instructions. (Advanced pattern)
}
```

### 2. Language Support (`.unsupportedLanguageOrLocale`)

The framework supports a specific, growing set of languages (e.g., English plus 14 others as of iOS 26.0). If the user's prompt or the requested generation language is unsupported, this error is thrown.

**Prevention**: Always check `SystemLanguageModel.default.supportedLanguages` at runtime instead of hardcoding a list or waiting for the error to be thrown. 

```swift
let supported = SystemLanguageModel.default.supportedLanguages
guard supported.contains(Locale.current.language) else {
    // Show language-not-supported message
    return
}
```

---

## Tool Calling Errors

If your session is configured with `Tool` instances, the `Tool.call(arguments:)` method might throw an error (e.g., network failure when fetching weather).

**The Invariant**: `Tool.call(arguments:)` is `async throws`. You must propagate errors out of this function; never silence them with `try?`. 

When a tool throws, the `FoundationModels` framework catches it, injects the error context back into the LLM, and often allows the model to gracefully inform the user ("I'm sorry, I couldn't reach the weather service right now") rather than crashing the application.

```swift
struct NetworkDataTool: Tool {
    // ...
    func call(arguments: Args) async throws -> String {
        // DO NOT do this:
        // let data = try? await fetch(arguments.query) // BAD: Silences error
        
        // DO this:
        let data = try await fetch(arguments.query) // Propagates error up
        return data
    }
}
```

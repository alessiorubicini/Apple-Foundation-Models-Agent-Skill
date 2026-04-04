# Concurrency — Actor Isolation and @MainActor

Source: `FoundationModels` framework, iOS 26.0+

---

## Core Concept

The `FoundationModels` framework is built natively for Swift 6 with strict concurrency checking enabled. Because `LanguageModelSession` maintains a stateful conversation transcript and coordinates with the out-of-process Apple Intelligence OS Daemon, it requires strict ownership and isolation rules to prevent data races.

## Session Ownership & Isolation

**The Invariant**: `LanguageModelSession` must be maintained as `@State` in SwiftUI or as an `actor`-isolated property. **Never** create a session locally inside a button action or a `Task` closure, as the session will be deallocated or lose its transcript state between calls.

### Pattern 1: SwiftUI `@State`
When bound directly to a view, use `@State` to maintain the session lifecycle alongside the view.

```swift
import SwiftUI
import FoundationModels

struct ChatView: View {
    // 1. Session is safely isolated to the view's state
    @State private var session = LanguageModelSession()
    @State private var prompt = ""

    var body: some View {
        Button("Send") {
            // 2. session calls must be off the main actor
            Task { 
                await sendMessage() 
            }
        }
    }

    private func sendMessage() async {
        do {
            let response = try await session.respond(to: prompt)
            print(response.content)
        } catch { /* ... */ }
    }
}
```
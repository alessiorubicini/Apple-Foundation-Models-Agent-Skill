# Streaming — Real-Time UI and PartiallyGenerated

Source: `FoundationModels` framework, iOS 26.0+

---

## Core Concept

For a responsive user experience, particularly with long responses, you should stream the model's output as it is generated rather than waiting for the complete response. `LanguageModelSession` provides `streamResponse(to:)` for this purpose, which returns an asynchronous sequence of updates. 

Streaming supports both plain text generation and structured output via `@Generable`.

---

## Text Streaming

For standard, unstructured generation, `streamResponse` yields fragments of text (`String`) as they are produced by the model.

```swift
import FoundationModels
import SwiftUI

@MainActor
@Observable final class ChatViewModel {
    // Session ownership must be @State or actor-isolated
    private let session = LanguageModelSession()
    var currentReply: String = ""
    var isGenerating = false

    func sendMessage(_ text: String) async {
        isGenerating = true
        defer { isGenerating = false }
        
        do {
            // Every streamResponse call lives in do/catch
            let stream = try await session.streamResponse(to: text)
            
            for try await fragment in stream {
                // Append chunks to the UI state incrementally
                currentReply += fragment.text 
            }
        } catch LanguageModelSession.GenerationError.exceededContextWindowSize {
            // Handle context overflow by creating a fresh session
        } catch {
            // Surface other errors to UI
        }
    }
}
```

---

## Guided Generation Streaming (Structured Output)

When using `@Generable` for structured output, the stream does not yield raw strings. Instead, it yields instances of a `PartiallyGenerated` companion type.

Every `@Generable` type gets a `PartiallyGenerated` companion type auto-synthesized by the macro. It mirrors the original struct but makes every property `Optional`, with `nil` indicating the model has not yet generated that field.

```swift
@Generable struct Itinerary {
    let title: String
    let description: String
    let days: [DayPlan]
}

@Generable struct DayPlan {
    let title: String
    let activities: [String]
}
```

### Consuming a PartiallyGenerated Stream

You must never construct a `PartiallyGenerated` instance manually; it is used exclusively in streaming contexts and accessed directly from the stream.

```swift
@MainActor
@Observable final class ItineraryViewModel {
    private let session = LanguageModelSession()
    
    // We can store the partial state to drive a loading UI
    var partialItinerary: Itinerary.PartiallyGenerated?
    var finalItinerary: Itinerary?

    func planTrip(destination: String) async {
        do {
            let stream = try await session.streamResponse(
                to: "Plan a trip to \(destination)",
                generating: Itinerary.self
            )
            
            for try await partial in stream {
                // partial is of type Itinerary.PartiallyGenerated
                self.partialItinerary = partial
                
                // You can safely unwrap properties as they arrive
                if let title = partial.title {
                    print("Generating itinerary for: \(title)")
                }
            }
            
            // Once the stream finishes successfully, you can extract the fully 
            // generated content (if supported by the stream API) or map the complete partial.
            // (Final mapping logic depends on specific FoundationModels stream termination API).
            
        } catch {
            // Handle GenerationError
        }
    }
}
```

---

## Best Practices & Correctness Invariants

1. **Incremental Updates**: Because `PartiallyGenerated` types have `Optional` properties, your SwiftUI views must gracefully handle `nil` values, typically by showing skeleton loaders or placeholders until the property is populated.
2. **Property Ordering**: The model generates properties top-to-bottom. Place summary/synthesis properties last so that the stream feels natural to the user, filling out the data structure in a logical UI order.
3. **Actor Isolation**: Ensure that the `for try await` loop consuming the stream updates UI state safely on the `@MainActor`. 
4. **Context Window**: Just like `respond(to:)`, streaming adds to the session's transcript. You must explicitly catch `LanguageModelSession.GenerationError.exceededContextWindowSize` during streaming. If thrown, create a fresh session; do not retry the same session.

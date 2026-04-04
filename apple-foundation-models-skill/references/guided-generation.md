# Guided Generation — @Generable, @Guide, DynamicGenerationSchema

Source: `FoundationModels` framework, iOS 26.0+  
WWDC25: [Deep Dive into the Foundation Models Framework](https://developer.apple.com/videos/play/wwdc2025/301/)

---

## Core Concept

Guided generation constrains the model's output at the **token level** to match a Swift type. The model is structurally forced to produce valid instances — no JSON parsing, no validation retry logic required.

---

## @Generable

Apply to `struct` or `enum`. All nested types must also be `@Generable`.

```swift
@Generable struct Itinerary {
    let title: String
    let description: String
    let days: [DayPlan]           // Nested @Generable array
}

@Generable struct DayPlan {
    let title: String
    let subtitle: String
    let activities: [Activity]
}

@Generable struct Activity {
    let title: String
    let type: ActivityType
}

@Generable enum ActivityType {
    case sightseeing
    case dining
    case adventure
}
```

### Key Rules

- `class` is **not** supported — always use `struct` or `enum`.
- Composability is unlimited: nest `@Generable` structs and enums freely.
- **Property order matters**: the model generates properties top-to-bottom. Place properties that depend on earlier values (e.g., summaries) **last**.
- `@Generable` enums do not require `@Generable` on associated value types if they are primitive (`String`, `Int`, `Bool`, `Double`). For custom associated types, they must also be `@Generable`.

---

## @Guide

Provides hints or constraints for individual properties.

```swift
@Generable struct Movie {
    @Guide(description: "Exactly one sentence describing the plot")
    let synopsis: String

    @Guide(.anyOf(["G", "PG", "PG-13", "R"]))
    let mpaaRating: String

    @Guide(.count(3))
    let topActors: [String]          // Always generates exactly 3 elements

    @Guide(.regex(#"\d{4}"#))
    let releaseYear: String          // Constrained to 4-digit year
}
```

### @Guide Constraint Types

| Constraint | Use case |
|---|---|
| `description:` | Free-form natural language hint to the model |
| `.anyOf([...])` | Constrain to an explicit set of string values |
| `.count(n)` | Fix array length to exactly `n` elements |
| `.regex(pattern)` | Constrain `String` output to match a regex pattern |

---

## Calling respond with a Generable Type

```swift
// Full response
let response = try await session.respond(
    to: "Plan a 3-day itinerary for Kyoto",
    generating: Itinerary.self
)
let itinerary: Itinerary = response.content

// With generation options
let response = try await session.respond(
    to: Prompt("Generate a movie recommendation"),
    generating: Movie.self,
    includeSchemaInPrompt: false,   // Default true; set false if schema is in instructions
    options: GenerationOptions(temperature: 0.7)
)
```

`includeSchemaInPrompt: false` saves tokens when the schema is already described in the session instructions.

---

## DynamicGenerationSchema

Build schemas at runtime when the output structure is not known at compile time.

```swift
var builder = DynamicGenerationSchema.Builder(name: "Riddle")
builder.addProperty(
    name: "question",
    schema: DynamicGenerationSchema(type: String.self)
)
builder.addProperty(
    name: "answers",
    schema: DynamicGenerationSchema(
        arrayOf: DynamicGenerationSchema(referenceTo: "Answer")
    )
)

var answerBuilder = DynamicGenerationSchema.Builder(name: "Answer")
answerBuilder.addProperty(name: "text", schema: DynamicGenerationSchema(type: String.self))
answerBuilder.addProperty(name: "isCorrect", schema: DynamicGenerationSchema(type: Bool.self))

let schema = builder.build(supporting: [answerBuilder.build()])
let response = try await session.respond(to: prompt, schema: schema)
```

Use `DynamicGenerationSchema` when schema varies per user configuration or plugin system. For fixed structures, prefer `@Generable` — it is faster, more readable, and compile-time safe.

---

## PartiallyGenerated

Every `@Generable` type gets a `PartiallyGenerated` companion type (auto-synthesized by the macro). It mirrors the struct but makes every property `Optional`, with `nil` indicating the model has not yet generated that field.

Used exclusively in streaming contexts — see `streaming.md`.

```swift
// Never construct PartiallyGenerated manually
// Access it from the stream:
for try await partial in stream {
    // partial is Itinerary.PartiallyGenerated
    if let title = partial.title { updateUI(title) }
}
```

---

## Macro Expansion (Informational)

`@Generable` synthesizes, among other things:

```swift
// Auto-generated — do not write manually
nonisolated static var generationSchema: GenerationSchema { ... }
nonisolated var generatedContent: GeneratedContent { ... }
nonisolated struct PartiallyGenerated: Identifiable, ConvertibleFromGeneratedContent { ... }
```

You do not interact with these directly in normal usage.
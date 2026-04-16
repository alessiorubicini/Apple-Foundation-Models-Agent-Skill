# Prompting Techniques — On-Device Foundation Models

Source: `FoundationModels` framework, iOS 26.0+  
Reference: [Foundation Models Prompting Guide](https://livsycode.com/best-practices/foundation-models-prompting-guide)

---

## Why On-Device Prompting Is Different

Server-side LLMs tolerate verbose, loosely structured prompts because they operate with large context windows and heavier reasoning capacity. On-device Foundation Models have a **4 096-token context window** and limited reasoning headroom. Prompt quality directly affects correctness — vague, padded, or overloaded prompts degrade output fast.

**Core principles:**

- Write simple, unambiguous instructions.
- Iterate based on test output.
- Provide a dedicated place for reasoning when you need it.
- Reduce the amount of reasoning required.
- Break complex work into smaller requests.
- Keep conditional behavior lightweight in the prompt; move branching into code.
- Use example-based prompting (zero-shot, one-shot, few-shot) to anchor format and behavior.
- Test prompts across real-world inputs — treat prompt quality like any other user-facing feature.

---

## Keep Prompts Simple and Clear

The model processes tokens under a tight budget. Long prompts waste tokens on text that does not help the task.

### Guidelines

- One well-defined goal per prompt.
- Lead with imperative verbs: **List**, **Generate**, **Rewrite**, **Classify**.
- Assign a role when it helps the model stay in the right register.
- Prefer short, active sentences.
- Keep prompts to one to three paragraphs when possible.
- Avoid jargon, hedging, social padding, and ambiguous phrasing.

### Concise vs. Bloated

```swift
// ✅ Task is visible and direct
let instructions = """
    From the person's recent browsing and purchases in home decor,
    produce five interest tags. Order tags by relevance.
    Add two extra tags from the same domain that are not mentioned in the input.
    """

// ❌ Task is buried inside explanation
let instructions = """
    The person will provide recent purchases and browsing related to home decor.
    Your output should contain a list of categories that reflect what they may like.
    Order categories so the most relevant appear early.
    Also include additional categories that create new ideas beyond the input.
    """
```

When prompts get longer, the model spends tokens interpreting setup. The task sentence becomes harder to spot. Clarity improves both results and token efficiency.

---

## Role, Persona, and Tone

The model defaults to a neutral, business-casual voice. Shape behavior by describing who it is and who it is speaking to.

### Role + Persona

A **role** describes the job. A **persona** describes how that job is performed.

```swift
let session = LanguageModelSession {
    """
    You are a senior iOS engineer who enjoys mentoring.
    Explain the concept using practical guidance and short examples.
    Avoid academic language.
    """
}
```

### "Expert" Keyword

Using "expert" pushes the model toward more confident, detailed output.

```swift
let session = LanguageModelSession {
    """
    You are an expert code reviewer.
    Review the person's sentence for clarity and precision.
    Return three concrete improvements.
    """
}
```

### Defining the Audience

The model assumes it speaks to an average adult. When your app serves a specific audience, state it explicitly.

```swift
let session = LanguageModelSession {
    """
    The person is new to programming.
    Explain the idea with simple words and define any technical term you use.
    """
}
```

### Setting Tone

Tone follows the voice of the instructions themselves.

```swift
let session = LanguageModelSession {
    """
    Write in a calm, professional style.
    Keep sentences short. Avoid humor and slang.
    """
}
```

---

## Iterating to Improve Instruction Following

Instruction following is the model's ability to execute what you wrote in `instructions` and the user's prompt. On-device prompting improves through iteration and testing.

### Three Adjustment Levers

1. **Improve clarity** — rewrite the instruction to remove ambiguity.
2. **Add emphasis** — use a small number of constraint words: `must`, `do not`, `avoid`.
3. **Repeat key constraints** — restate the most important constraint once at the end.

```swift
let session = LanguageModelSession {
    """
    Summarize the text in three bullet points.
    Each bullet must be a single sentence.
    Do not include extra commentary.
    Repeat: output exactly three bullet points.
    """
}
```

### Baseline Testing

If a prompt remains unreliable after a few iterations, reduce complexity. A compact baseline prompt isolates whether the issue is the prompt or the task itself:

```swift
let session = LanguageModelSession {
    "Answer the person's question."
}
```

If the baseline is unstable, the task may not fit the model's capabilities or input distribution.

---

## Reducing Reasoning Burden

On-device models have limited reasoning capacity compared to server models. When a task requires planning, **provide the plan** — convert a vague objective into a small procedure.

### Step-by-Step Plan in a Single Request

```swift
let session = LanguageModelSession {
    """
    Given the person's activity related to running shoes:
    1. Pick four product categories that match the activity.
    2. Add two related categories that are not explicitly mentioned.
    3. Return a single list ordered by relevance.
    """
}
```

### Splitting Across Multiple Sessions

When output quality is inconsistent in a single session, splitting steps across sessions improves stability by resetting the context window.

```swift
let extractionSession = LanguageModelSession {
    """
    Extract key facts from the person's message.
    Return them as short bullet points.
    """
}

let responseSession = LanguageModelSession {
    """
    Using the extracted facts, draft a helpful reply.
    Keep it under 120 words.
    """
}

// Pipeline: extract → generate
let facts = try await extractionSession.respond(to: userMessage)
let reply = try await responseSession.respond(to: facts.content)
```

This pattern also makes debugging easier — you can see where drift starts.

---

## Moving Conditionals into Code

Conditionals inside a prompt are tempting. As the rule list grows, on-device models lose track and apply irrelevant conditions. Move branching into Swift and inject only the relevant instruction.

### ❌ Multiple In-Text Conditions

```swift
// Prompt has too many branches — model may apply wrong one
let instructions = """
    You are a friendly shopkeeper. Greet the visitor.
    IF the visitor is a wizard, mention their staff.
    IF the visitor is a musician, ask about tonight's performance.
    IF the visitor is a guard, ask about safety in the city.
    There is one small room and one large room available.
    """
```

### ✅ Runtime-Customized Prompt

```swift
enum VisitorKind {
    case wizard, musician, guard, unknown
}

func visitorNote(for kind: VisitorKind) -> String {
    switch kind {
    case .wizard:   return "The visitor is a wizard. Comment briefly on their magical gear."
    case .musician: return "The visitor is a musician. Ask if they plan to play here tonight."
    case .guard:    return "The visitor is a guard. Ask if there have been recent incidents."
    case .unknown:  return "The visitor looks tired from travel."
    }
}

let base = """
    You are a friendly shopkeeper.
    Write a greeting for a visitor who just arrived.
    """
let note = visitorNote(for: kind)

let session = LanguageModelSession {
    """
    \(base)
    \(note)
    Mention that one small room and one large room are available.
    """
}
```

This keeps irrelevant condition branches out of the context window.

---

## Few-Shot Prompting

When you need stable format, small examples help. Apple recommends **2–15 examples**. Keep each example short — overly detailed examples lead to repetition and hallucinated details.

```swift
let session = LanguageModelSession {
    """
    Create a fictional café customer for a cozy game.
    Return an object with fields: name, look, order.

    Examples:
    {name: "Juniper", look: "a sleepy fox in a knit scarf", order: "hot cocoa with cinnamon"}
    {name: "Mara", look: "a tiny robot with a cracked screen", order: "iced latte, extra ice"}
    {name: "Orin", look: "a painter with ink-stained fingers", order: "espresso, no sugar"}
    """
}
```

Combine few-shot examples with `@Generable` for strict schema enforcement on top of stylistic anchoring — see `guided-generation.md`.

---

## Reasoning Fields in Structured Output

Reasoning prompts can leak "working" text into structured output when the model has nowhere to put it. Add a **reasoning field as the first property**, then constrain the final answer field.

```swift
@Generable
struct Answer {
    var workLog: String              // Model writes its thinking here

    @Guide(description: "The answer only.")
    var result: String               // Clean, user-facing output
}
```

Prompt it explicitly to use the field:

```swift
let session = LanguageModelSession {
    """
    Answer the person's question.
    1. Write a short plan in workLog.
    2. Follow the plan and show intermediate steps in workLog.
    3. Put the final answer in result.
    """
}

let response = try await session.respond(
    to: "How many days are there in November?",
    generating: Answer.self
)
// response.content.result contains the clean answer
// response.content.workLog contains the reasoning trace
```

This does not guarantee perfect reasoning. It gives the model a safe place to put it, protecting the shape of your output.

---

## Quick-Reference Checklist

| Technique | When to Use |
|---|---|
| Simple, imperative instructions | Every prompt — baseline hygiene |
| Role / persona assignment | When default tone doesn't match app context |
| Emphasis + repeated constraints | When the model ignores a specific rule |
| Step-by-step plan | When task requires multi-step reasoning |
| Multi-session pipeline | When single-session output drifts |
| Code-side branching | When prompt has ≥ 2 conditional paths |
| Few-shot examples (2–15) | When output format must be stable |
| Reasoning field (`workLog`) | When structured output needs intermediate thinking |
| Baseline testing | When debugging unreliable prompt behavior |

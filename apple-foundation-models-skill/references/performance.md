
# Performance & Memory Management

Source: `FoundationModels` framework, iOS 26.0+


## Core Architecture & Hardware Footprint

Understanding the physical constraints of the on-device model is critical for building responsive applications without degrading overall system performance.



* **Model Size**: The default Apple Intelligence model operates with ~3B parameters and uses 2-bit quantization.
* **Memory Usage**: When loaded into memory, the model consumes approximately **1.2 GB of RAM**.
* **KV-Cache Optimization**: The system shares the KV-cache across blocks, which results in a ~38% memory reduction compared to standard caching techniques.
* **Execution Context**: The model runs via an OS Daemon (Apple Intelligence Daemon) utilizing the Neural Engine, GPU, and CPU dynamically.
* **App Lifecycle**: The model is automatically paused and unloaded by the OS when your app goes into the background to preserve system resources.


## Latency & Prewarming

First-token latency (the time it takes for the model to start generating the first word of its response) is heavily dependent on whether the model is already loaded into RAM.

* **Cold Start**: ~500 ms latency on A18-class hardware.
* **Prewarmed**: <150 ms latency on A18-class hardware.

### Using `prewarm()` Correctly

To achieve the <150 ms latency target, you must explicitly tell the OS to load the model into memory *before* the user requests generation.

**The Invariant**: Call `session.prewarm()` during idle time, such as in `.onAppear` or a `.task` modifier. Do **not** call it immediately before `respond()`.

```swift
import SwiftUI
import FoundationModels

struct MagicComposeView: View {
    @State private var session = LanguageModelSession()
    @State private var text = ""
    
    var body: some View {
        VStack {
            TextField("Draft your message...", text: $text)
            Button("Rewrite") {
                Task {
                    // DO NOT call prewarm() here. It provides zero benefit 
                    // if generation immediately follows.
                    await rewriteText() 
                }
            }
        }
        .task {
            // DO THIS: Prewarm during idle time when the view loads
            await session.prewarm()
        }
    }
    
    private func rewriteText() async { /* ... */ }
}
```


## The 4 096 Token Budget

Every session operates under a strict, unyielding combined context window of **4 096 tokens** (input prompt + system instructions + tool schemas + transcript history + generated output). 

If you exceed this limit, the framework throws `LanguageModelSession.GenerationError.exceededContextWindowSize`.

### Token Optimization Strategies

1.  **Keep Instructions Concise**: Do not write overly verbose personas. The instructions are sent with every single prompt in the transcript.
2.  **Rely on `@Generable` for Structure**: Do not waste tokens explaining JSON structures or formatting rules in plain text. Put structural intent directly into your `@Generable` Swift types and use `@Guide`. The framework encodes these natively and efficiently.
3.  **Use `includeSchemaInPrompt: false`**: If you must explain the structure manually within the instructions closure, set `includeSchemaInPrompt: false` when calling `respond` to prevent the framework from duplicating the schema overhead.
4.  **Limit Tools**: Every `Tool` you provide adds its description and `Arguments` schema to the context window. Only inject tools that are strictly necessary for the current task.
5.  **Restart Sessions Proactively**: If building a long-running chat UI, monitor the conversation length and proactively summarize the transcript to start a fresh `LanguageModelSession` before hitting the overflow error.


## WWDC 2026 Beta Updates

WWDC 2026 Beta: APIs require Xcode 27 beta and iOS 27.0 / macOS 27.0 / visionOS 27.0 / watchOS 27.0 beta unless noted. Always guard usage with `@available` or `#available` and verify against current Apple documentation before shipping.

Sources:
- https://developer.apple.com/documentation/foundationmodels/systemlanguagemodel
- https://developer.apple.com/documentation/foundationmodels/privatecloudcomputelanguagemodel
- https://developer.apple.com/documentation/foundationmodels/languagemodelsession
- Apple WWDC 2026 session: Build agentic app experiences with the Foundation Models framework

- Use `SystemLanguageModel.contextSize` and `try await PrivateCloudComputeLanguageModel.contextSize`; do not hardcode context size for beta APIs.
- Use `SystemLanguageModel.tokenCount(for:)` to budget prompts, instructions, tools, schemas, and transcript entries before generation.
- `LanguageModelSession.usage` and response/stream `usage` expose input, cached input, output, reasoning token counts, and metadata.
- Private Cloud Compute may have a different context size from the on-device model.
- For custom providers, send usage events through `LanguageModelExecutorGenerationChannel` when underlying token counters are available.
- Transcript rewrites can invalidate KV cache. Appending preserves cache best; removing entries, changing tools, or changing instructions may increase first-token latency.
- Use the Foundation Models Instrument in Xcode and eval sets to measure transcript-engineering changes; do not assume a trim/summarize strategy improves accuracy or latency.

```swift
import FoundationModels

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
actor BudgetGate {
    private let model = SystemLanguageModel.default

    init?() {
        guard case .available = model.availability else { return nil }
    }

    func canFit(prompt: Prompt, reservedOutputTokens: Int) async throws -> Bool {
        let inputTokens = try await model.tokenCount(for: prompt)
        return inputTokens + reservedOutputTokens <= model.contextSize
    }
}
```

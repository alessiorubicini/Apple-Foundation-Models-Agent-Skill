# Custom Language Model Provider — WWDC 2026 Beta

WWDC 2026 Beta: APIs require iOS 27.0 / macOS 27.0 / visionOS 27.0 / watchOS 27.0 beta unless noted. Verify against current Apple documentation before shipping.

Sources:
- https://developer.apple.com/documentation/foundationmodels/languagemodel
- https://developer.apple.com/documentation/foundationmodels/languagemodelcapabilities
- https://developer.apple.com/documentation/foundationmodels/languagemodelexecutor
- https://developer.apple.com/documentation/foundationmodels/languagemodelexecutorgenerationrequest
- https://developer.apple.com/documentation/foundationmodels/languagemodelexecutorgenerationchannel

## API Surface

- `LanguageModel` lets a custom model participate in `LanguageModelSession`.
- `LanguageModel.Executor` must conform to `LanguageModelExecutor`.
- `LanguageModelCapabilities` advertises `.vision`, `.guidedGeneration`, `.reasoning`, and `.toolCalling`.
- `LanguageModelExecutorGenerationRequest` carries request ID, transcript, enabled tool definitions, optional schema, generation options, context options, and metadata.
- `LanguageModelExecutorGenerationChannel` receives response, reasoning, tool-call, metadata, and usage events.

## Minimal Provider

```swift
import Foundation
import FoundationModels

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
struct EchoLanguageModel: LanguageModel {
    let capabilities = LanguageModelCapabilities(capabilities: [.toolCalling])
    let executorConfiguration = Executor.Configuration(prefix: "Echo")

    struct Executor: LanguageModelExecutor {
        struct Configuration: Hashable, Sendable {
            var prefix: String
        }

        private let configuration: Configuration

        init(configuration: Configuration) {
            self.configuration = configuration
        }

        func prewarm(model: EchoLanguageModel, transcript: Transcript) {}

        func respond(
            to request: LanguageModelExecutorGenerationRequest,
            model: EchoLanguageModel,
            streamingInto channel: LanguageModelExecutorGenerationChannel
        ) async throws {
            await channel.send(.response(action: .appendText("\(configuration.prefix): ready", tokenCount: 3)))
        }
    }
}

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
actor EchoAgent {
    private let session: LanguageModelSession

    init() {
        session = LanguageModelSession(model: EchoLanguageModel())
    }

    func ask(_ prompt: String) async throws -> String {
        do {
            let response = try await session.respond(to: Prompt(prompt))
            return response.content
        } catch LanguageModelError.unsupportedCapability {
            throw LanguageModelError.unsupportedCapability(
                .init(capability: .vision, debugDescription: "EchoLanguageModel does not accept image attachments.")
            )
        }
    }
}
```

## Channel Events

Use typed channel events:

- `.response(entryID:action:)` with actions for text fragments, text replacement, custom segments, attachment segments, metadata, and usage.
- `.reasoning(entryID:action:)` with actions for reasoning text, signature, metadata, and usage.
- `.toolCalls(entryID:action:)` with actions for tool call creation, argument fragments, metadata, and usage.

Prefer static convenience actions:

```swift
import FoundationModels

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
func emitReasoning(channel: LanguageModelExecutorGenerationChannel) async {
    await channel.send(
        .reasoning(action: .appendText("Need the account tool before answering.", tokenCount: 7))
    )
    await channel.send(
        .response(action: .appendText("I'll check that for you.", tokenCount: 6))
    )
}
```

## Correctness Rules

- Keep `Configuration` `Hashable & Sendable`.
- Ensure `LanguageModelCapabilities` reflects actual executor support; unsupported inputs should fail with `LanguageModelError.unsupportedCapability`.
- Use `LanguageModelExecutorGenerationRequest.contextOptions` and `.generationOptions` instead of inventing custom request flags.
- Preserve `request.metadata` into emitted metadata when the provider needs traceability.
- Send token counts in text, usage, and reasoning events when available from the underlying model.
- Do not block inside `respond(to:model:streamingInto:)`; stream deltas asynchronously and propagate errors.

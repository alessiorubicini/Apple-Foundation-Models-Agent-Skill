# Private Cloud Compute — WWDC 2026 Beta

WWDC 2026 Beta: APIs require iOS 27.0 / macOS 27.0 / visionOS 27.0 / watchOS 27.0 beta unless noted. Verify against current Apple documentation before shipping.

Sources:
- https://developer.apple.com/documentation/foundationmodels/privatecloudcomputelanguagemodel
- https://developer.apple.com/documentation/foundationmodels/privatecloudcomputelanguagemodel/error
- https://developer.apple.com/documentation/foundationmodels/languagemodel
- https://developer.apple.com/documentation/foundationmodels/languagemodelsession
- https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.developer.private-cloud-compute

## API Surface

- `PrivateCloudComputeLanguageModel` is a `LanguageModel` implementation backed by Private Cloud Compute.
- `availability` returns `.available` or `.unavailable(_:)`.
- `isAvailable` is a convenience Boolean.
- `quotaUsage` exposes current usage state, limit status, reset date, and optional limit-increase suggestion.
- `contextSize` is async throwing and returns the model's supported token context size.
- `supportedLanguages` and `supportsLocale(_:)` mirror language support checks for the PCC model.
- `PrivateCloudComputeLanguageModel.Error` includes `.networkFailure(_:)`, `.quotaLimitReached(_:)`, and `.serviceUnavailable(_:)`.
- Apps need the `com.apple.developer.private-cloud-compute` entitlement to use PCC.

## Example

```swift
import Foundation
import FoundationModels

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
actor CloudSummarizer {
    private let model: PrivateCloudComputeLanguageModel
    private let session: LanguageModelSession

    init?() {
        let model = PrivateCloudComputeLanguageModel()
        guard case .available = model.availability else { return nil }
        self.model = model
        self.session = LanguageModelSession(model: model) {
            "Summarize documents with citations to supplied text only."
        }
    }

    func summarize(_ text: String) async throws -> String {
        if model.quotaUsage.isLimitReached {
            if let suggestion = model.quotaUsage.limitIncreaseSuggestion {
                suggestion.show()
            }
            throw PrivateCloudComputeLanguageModel.Error.quotaLimitReached(
                .init(
                    limitIncreaseSuggestion: model.quotaUsage.limitIncreaseSuggestion,
                    resetDate: model.quotaUsage.resetDate,
                    debugDescription: "PCC quota reached."
                )
            )
        }

        _ = try await model.contextSize

        do {
            let response = try await session.respond(
                to: Prompt(text),
                contextOptions: ContextOptions(reasoningLevel: .moderate),
                metadata: ["feature": "cloud-summary"]
            )
            return response.content
        } catch PrivateCloudComputeLanguageModel.Error.serviceUnavailable {
            throw PrivateCloudComputeLanguageModel.Error.serviceUnavailable(
                .init(debugDescription: "Private Cloud Compute is temporarily unavailable.")
            )
        } catch LanguageModelError.contextSizeExceeded {
            throw LanguageModelError.contextSizeExceeded(
                .init(contextSize: 0, tokenCount: 0, debugDescription: "Prompt exceeded PCC context size.")
            )
        }
    }
}
```

## Availability Handling

```swift
import FoundationModels

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
func pccReadiness() -> String {
    let model = PrivateCloudComputeLanguageModel()

    switch model.availability {
    case .available:
        return "ready"
    case .unavailable(.deviceNotEligible):
        return "device-not-eligible"
    case .unavailable(.systemNotReady):
        return "system-not-ready"
    case .unavailable:
        return "unavailable"
    }
}
```

## Correctness Rules

- Gate on `PrivateCloudComputeLanguageModel().availability` before creating a PCC-backed `LanguageModelSession`.
- Check `quotaUsage.isLimitReached` before expensive requests and surface `limitIncreaseSuggestion.show()` only as a user-initiated UI action.
- Use `try await model.contextSize`; do not assume the on-device 4096-token context window for PCC.
- Keep prompt data minimal and user-controlled data separated from instructions even when using PCC.
- Catch PCC errors separately from `LanguageModelError` when distinguishing network/service/quota failures matters.
- Do not document server implementation details beyond the public FoundationModels PCC surface.

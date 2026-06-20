# WWDC 2026 Beta APIs — Source Map

WWDC 2026 Beta: APIs require Xcode 27 beta and iOS 27.0 / macOS 27.0 / visionOS 27.0 / watchOS 27.0 beta unless noted. Always guard usage with `@available` or `#available` and verify against current Apple documentation before shipping.

Sources: Apple FoundationModels documentation and Xcode 27.0 beta `FoundationModels.swiftinterface`.

## Load Budget

- Use this file as beta triage, not as a full implementation reference.
- Load exactly one beta topic file unless the user's request spans multiple beta surfaces.
- Do not load PCC, custom provider, prompt attachment, dynamic profile, or session property references speculatively.
- For source verification, cite Apple documentation URLs from the table below and validate signatures against the Xcode 27 beta interface when generating code.

## Use This File When

- The user asks about FoundationModels APIs marked beta in Apple documentation.
- A task mentions Private Cloud Compute, prompt attachments, dynamic profiles, session properties, custom language model providers, reasoning, metadata, or beta error handling.
- Existing iOS 26 guidance conflicts with iOS 27 beta signatures.

## Routing

| Topic | Load |
|---|---|
| Beta API inventory and availability | this file |
| Multimodal image prompting | `prompt-attachments.md` |
| Runtime-selected instructions/tools/model settings | `dynamic-profiles.md` |
| Shared profile/tool/session state | `session-properties.md` |
| Private Cloud Compute model | `private-cloud-compute.md` |
| Bring-your-own model executor | `custom-language-model-provider.md` |
| Tool calling policy and reasoning options | `generation-options.md` |
| New error model and transcript rollback | `error-handling.md` |

## Official Documentation Paths

| API | Apple documentation |
|---|---|
| FoundationModels module | https://developer.apple.com/documentation/foundationmodels/ |
| `Attachment` | https://developer.apple.com/documentation/foundationmodels/attachment |
| `AttachmentContent` | https://developer.apple.com/documentation/foundationmodels/attachmentcontent |
| `ImageAttachmentContent` | https://developer.apple.com/documentation/foundationmodels/imageattachmentcontent |
| `ImageReference` | https://developer.apple.com/documentation/foundationmodels/imagereference |
| `DynamicInstructions` | https://developer.apple.com/documentation/foundationmodels/dynamicinstructions |
| `LanguageModelSession.DynamicProfile` | https://developer.apple.com/documentation/foundationmodels/languagemodelsession/dynamicprofile |
| `LanguageModelSession.DynamicProfileModifier` | https://developer.apple.com/documentation/foundationmodels/languagemodelsession/dynamicprofilemodifier |
| `LanguageModelSession.Profile` | https://developer.apple.com/documentation/foundationmodels/languagemodelsession/profile |
| `LanguageModelSession.SessionProperty` | https://developer.apple.com/documentation/foundationmodels/languagemodelsession/sessionproperty |
| `SessionPropertyKey` | https://developer.apple.com/documentation/foundationmodels/sessionpropertykey |
| `SessionPropertyValues` | https://developer.apple.com/documentation/foundationmodels/sessionpropertyvalues |
| `SessionPropertyEntry()` | https://developer.apple.com/documentation/foundationmodels/sessionpropertyentry() |
| `ContextOptions` | https://developer.apple.com/documentation/foundationmodels/contextoptions |
| `PrivateCloudComputeLanguageModel` | https://developer.apple.com/documentation/foundationmodels/privatecloudcomputelanguagemodel |
| `LanguageModel` | https://developer.apple.com/documentation/foundationmodels/languagemodel |
| `LanguageModelCapabilities` | https://developer.apple.com/documentation/foundationmodels/languagemodelcapabilities |
| `LanguageModelExecutor` | https://developer.apple.com/documentation/foundationmodels/languagemodelexecutor |
| `LanguageModelExecutorGenerationRequest` | https://developer.apple.com/documentation/foundationmodels/languagemodelexecutorgenerationrequest |
| `LanguageModelExecutorGenerationChannel` | https://developer.apple.com/documentation/foundationmodels/languagemodelexecutorgenerationchannel |
| `LanguageModelError` | https://developer.apple.com/documentation/foundationmodels/languagemodelerror |
| `TranscriptErrorHandlingPolicy` | https://developer.apple.com/documentation/foundationmodels/transcripterrorhandlingpolicy |
| `LanguageModelSession` beta members | https://developer.apple.com/documentation/foundationmodels/languagemodelsession |
| `SystemLanguageModel` beta members | https://developer.apple.com/documentation/foundationmodels/systemlanguagemodel |
| `GenerationOptions` beta members | https://developer.apple.com/documentation/foundationmodels/generationoptions |
| `Tool` beta members | https://developer.apple.com/documentation/foundationmodels/tool |
| `Prompt` beta members | https://developer.apple.com/documentation/foundationmodels/prompt |
| `Instructions` beta members | https://developer.apple.com/documentation/foundationmodels/instructions |
| `Transcript` beta members | https://developer.apple.com/documentation/foundationmodels/transcript |

## Beta Additions

### Prompt Attachments

- `Attachment<Content>` where `Content: AttachmentContent`.
- `Attachment<ImageAttachmentContent>` initializers for `CGImage`, `CIImage`, `CVPixelBuffer`, and image file URLs.
- `Attachment.label(_:)` for model-visible attachment labels.
- `ImageReference: Generable` for generated references to labeled images.
- `Transcript.Attachment`, `Transcript.ImageAttachment`, and `Transcript.AttachmentSegment`.

### Dynamic Sessions

- `DynamicInstructions` and `DynamicInstructionsBuilder`.
- `LanguageModelSession.Profile`.
- `LanguageModelSession.DynamicProfile`.
- `LanguageModelSession.DynamicProfileModifier`.
- `LanguageModelSession(profile:history:)`.
- `LanguageModelSession(model:dynamicInstructions:history:)`.
- Dynamic profile modifiers for model, temperature, sampling mode, reasoning level, tool calling mode, history transform, transcript error policy, and lifecycle hooks.

### Session Properties

- `SessionPropertyKey`.
- `SessionPropertyValues`.
- `SessionPropertyEntry()`.
- `LanguageModelSession.SessionProperty`.
- Typealiases on `DynamicInstructions`, `Tool`, `DynamicProfile`, and `DynamicProfileModifier`.
- Built-in `history` and `rootDynamicInstructions` session properties.

### Private Cloud Compute

- `PrivateCloudComputeLanguageModel`.
- `com.apple.developer.private-cloud-compute` entitlement.
- PCC availability, quota usage, async `contextSize`, supported languages, locale checks, and PCC-specific errors.

### Custom Providers

- `LanguageModel`.
- `LanguageModelCapabilities` and capability constants: `.vision`, `.guidedGeneration`, `.reasoning`, `.toolCalling`.
- `LanguageModelExecutor`.
- `LanguageModelExecutorGenerationRequest`.
- `LanguageModelExecutorGenerationChannel` with response, reasoning, tool-call, metadata, and usage events.

### Session, Options, Transcript, and Errors

- `ContextOptions` with `includeSchemaInPrompt` and `reasoningLevel`.
- `ContextOptions.ReasoningLevel`: `.light`, `.moderate`, `.deep`, `.custom(String)`.
- `GenerationOptions.ToolCallingMode`: `.allowed`, `.required`, `.disallowed`.
- `LanguageModelSession` response/stream overloads with `contextOptions` and `metadata`.
- `LanguageModelSession.usage`.
- `LanguageModelSession.transcriptErrorHandlingPolicy`.
- `TranscriptErrorHandlingPolicy.preserveTranscript` and `.revertTranscript`.
- `LanguageModelError` supersedes deprecated `LanguageModelSession.GenerationError` on iOS 27/macOS 27/visionOS 27/watchOS 27.
- `Transcript` becomes mutable/range-replaceable in beta and adds reasoning, custom segments, attachment segments, and metadata-bearing prompt/response/tool-call entries.

## Rules

- Do not use beta APIs without `@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)` or a runtime `#available` branch.
- Prefer `LanguageModelError` for iOS 27 beta examples; retain `LanguageModelSession.GenerationError` only for iOS 26 compatibility guidance.
- Treat watchOS 27 beta support for existing APIs as platform availability, not a new semantic API.
- Do not document underscored frameworks or underscored public symbols unless Apple documentation exposes them.
- If Apple docs and SDK disagree, cite both and mark the behavior as beta drift.

## Beta Code Checklist

- Cite an Apple documentation URL for every beta type or member used.
- Add `@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)` or an equivalent `#available` branch.
- Check the model's `availability` before creating a `LanguageModelSession`.
- Catch `LanguageModelError` in beta examples and handle `.contextSizeExceeded` explicitly.
- Read `contextSize` dynamically for PCC and custom models; do not assume the stable on-device 4 096-token limit applies.
- Keep `LanguageModelSession` owned by `@State`, `@MainActor`, or an `actor`; never create it transiently inside a button action.
- Exclude underscored APIs and implementation details from generated guidance.

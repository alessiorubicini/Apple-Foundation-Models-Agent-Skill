# Prompt Attachments — WWDC 2026 Beta

WWDC 2026 Beta: APIs require Xcode 27 beta and iOS 27.0 / macOS 27.0 / visionOS 27.0 / watchOS 27.0 beta unless noted. Always guard usage with `@available` or `#available` and verify against current Apple documentation before shipping.

Sources:
- https://developer.apple.com/documentation/foundationmodels/analyzing-images-with-multimodal-prompting#Use-built-in-image-analysis-tools
- https://developer.apple.com/documentation/foundationmodels/attachment
- https://developer.apple.com/documentation/foundationmodels/attachmentcontent
- https://developer.apple.com/documentation/foundationmodels/imageattachmentcontent
- https://developer.apple.com/documentation/foundationmodels/imagereference
- https://developer.apple.com/documentation/foundationmodels/transcript
- https://developer.apple.com/documentation/vision/barcodereadertool
- https://developer.apple.com/documentation/vision/ocrtool

## API Surface

- `Attachment<Content>` wraps non-text content for `Prompt` and `Instructions` builders.
- `ImageAttachmentContent` is the public image attachment content type.
- `Attachment<ImageAttachmentContent>` can be created from `CGImage`, `CIImage`, `CVPixelBuffer`, or an image file URL.
- `label(_:)` assigns a stable label the prompt can refer to and the model can return.
- `ImageReference` is `Generable`; use it when structured output needs to refer back to a labeled image.
- `ImageReference.resolve(in:)` returns the referenced `Transcript.ImageAttachment?` from a transcript.
- Vision provides built-in FoundationModels tools for image analysis: `BarcodeReaderTool` scans machine-readable codes, and `OCRTool` recognizes text in images.

## Multimodal Prompt

```swift
import Foundation
import FoundationModels

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
actor ImageCaptioner {
    private let session: LanguageModelSession

    init?() {
        guard case .available = SystemLanguageModel.default.availability else { return nil }
        session = LanguageModelSession {
            "Describe only visible image content. Do not infer hidden context."
        }
    }

    func caption(imageURL: URL) async throws -> String {
        let prompt = Prompt {
            "Describe the product shown in the attached image."
            Attachment(imageURL: imageURL).label("product-photo")
        }

        do {
            let response = try await session.respond(to: prompt)
            return response.content
        } catch LanguageModelError.contextSizeExceeded {
            throw LanguageModelError.contextSizeExceeded(
                .init(contextSize: 0, tokenCount: 0, debugDescription: "Create a fresh session before retrying.")
            )
        }
    }
}
```

## Built-in Vision Image Analysis Tools

Apple's Vision framework provides beta tools that can be passed directly to `LanguageModelSession(tools:)` for common image-analysis tasks:

- `BarcodeReaderTool` for detecting barcodes and interpreting encoded content.
- `OCRTool` for extracting text from images.

When using these tools, label each relevant `Attachment` so the model can target the correct image in the prompt or transcript.

```swift
import CoreGraphics
import FoundationModels
import Vision

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
actor BarcodeImageAnalyzer {
    private let session: LanguageModelSession

    init?() {
        guard case .available = SystemLanguageModel.default.availability else { return nil }
        session = LanguageModelSession(
            tools: [
                BarcodeReaderTool(
                    name: "barcode-reader",
                    description: "Scans machine-readable codes in attached images."
                )
            ]
        )
    }

    func analyzeBarcodeImage(_ image: CGImage) async throws -> String {
        do {
            let response = try await session.respond {
                """
                Scan the image for barcodes. For each barcode found, describe \
                the symbology type and explain what the encoded content represents.
                """

                Attachment(image)
                    .label("barcode-image")
            }
            return response.content
        } catch LanguageModelError.contextSizeExceeded {
            throw LanguageModelError.contextSizeExceeded(
                .init(contextSize: 0, tokenCount: 0, debugDescription: "Create a fresh session before retrying.")
            )
        }
    }
}
```

## Generated Image Reference

Use labels when the model must point at one attachment from several choices. Resolve the generated `ImageReference` against the session transcript; do not persist it as a standalone image handle.

```swift
import Foundation
import FoundationModels

@Generable
@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
struct ImageChoice {
    let selectedImage: ImageReference
    let reason: String
}

@available(iOS 27.0, macOS 27.0, visionOS 27.0, watchOS 27.0, *)
actor ImageResolver {
    private let session: LanguageModelSession

    init?() {
        guard case .available = SystemLanguageModel.default.availability else { return nil }
        session = LanguageModelSession()
    }

    func chooseImage(left: URL, right: URL) async throws -> Transcript.ImageAttachment? {
        let response = try await session.respond(
            to: Prompt {
                "Choose the image with the clearer label."
                Attachment(imageURL: left).label("left")
                Attachment(imageURL: right).label("right")
            },
            generating: ImageChoice.self
        )
        return response.content.selectedImage.resolve(in: session.transcript)
    }
}
```

## Transcript Attachment Types

- `Transcript.Attachment.image(_:)` stores image attachment content in transcript form.
- `Transcript.ImageAttachment` exposes `cgImage`, `ciImage`, `pixelBuffer(resolution:pixelFormat:)`, `orientation`, and optional `url`.
- `Transcript.AttachmentSegment` stores a transcript segment with `content` and optional `label`.

## Invariants

- Always label multiple images when generated output can refer to them.
- Prefer Vision's built-in `BarcodeReaderTool` and `OCRTool` before writing custom barcode or OCR tools.
- Keep prompts specific to visible content; use tools for app data or world knowledge.
- Check `LanguageModelCapabilities.Capability.vision` before sending image attachments to a custom `LanguageModel`.
- Catch `LanguageModelError.unsupportedCapability` and `LanguageModelError.unsupportedTranscriptContent` for models that do not accept attachments.

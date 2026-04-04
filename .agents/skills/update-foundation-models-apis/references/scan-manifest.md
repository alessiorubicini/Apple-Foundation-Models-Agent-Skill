# Foundation Models documentation scan manifest

Use with Sosumi MCP (`searchAppleDocumentation`, `fetchAppleDocumentation`) or raw HTTP fetch to `tutorials/data`.

---

## Module root

- **Seed JSON:** `/documentation/foundationmodels.json`
- **Landing:** `/documentation/foundationmodels`

---

## High-priority symbol paths (always re-verify)

- `/documentation/foundationmodels/systemlanguagemodel`
- `/documentation/foundationmodels/languagemodelsession`
- `/documentation/foundationmodels/generable`
- `/documentation/foundationmodels/tool`
- `/documentation/foundationmodels/generationoptions`
- `/documentation/foundationmodels/languagemodelsession/generationerror`
- `/documentation/foundationmodels/languagemodelsession/toolcallerror`
- `/documentation/foundationmodels/systemlanguagemodel/adapter`

---

## Release notes & technotes

- `/documentation/Updates/FoundationModels`
- `/documentation/Technotes/tn3193-managing-the-on-device-foundation-model-s-context-window`

---

## Search queries (broad discovery)

- `Foundation Models LanguageModelSession`
- `Foundation Models SystemLanguageModel`
- `Foundation Models Generable macro`
- `Foundation Models Tool Sendable`
- `Foundation Models adapter Background Assets`

---

## WWDC media (transcripts optional)

- `/videos/play/wwdc2025/286` — Meet the Foundation Models framework
- `/videos/play/wwdc2025/301` — (related; verify catalog)
- `/videos/play/wwdc2025/259` — Code-along Foundation Models

---

## Version bump protocol

When Apple ships **26.5+**:

1. Re-run full BFS crawl.
2. Diff `introducedAt` / `deprecated` flags inside JSON metadata.
3. Append a section to release-notes reference or create `ios-26-5-release-notes.md`.
4. Update SKILL.md “iOS 26 family” wording only with verified marketing/OS strings from Apple.

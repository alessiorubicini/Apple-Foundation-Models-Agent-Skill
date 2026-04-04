---
name: update-foundation-models-apis
description: Re-crawls Apple FoundationModels documentation (tutorials/data JSON and Updates page) to refresh symbol-index.md, flag iOS point-release deltas (e.g. 26.5), and list new deprecations. Use after an iOS/Xcode drop or when asked to verify new Foundation Models symbols. Network access required; optional DocC fetch via WebFetch or Sosumi MCP if available.
---

# Update Foundation Models API reference

Regenerate the local symbol catalog and 26.x delta notes for this repository’s `apple-foundation-models-skill/references/`.

## Prerequisites

- Network access to `https://developer.apple.com/tutorials/data/documentation/`
- Write access to this repo

Optional: **Sosumi MCP** (same tool names as SwiftUI maintenance skill) for interactive search—mirrors [.agents/skills/update-foundation-models-apis/references/scan-manifest.md](references/scan-manifest.md).

## Workflow

### 1. BFS crawl `foundationmodels.json`

Seed queue with `/documentation/foundationmodels.json`.

For each downloaded JSON:

- Parse `references` object.
- For every value with `"url"` starting `/documentation/foundationmodels`, enqueue `${url}.json` if not yet fetched.
- Persist JSON files under a local cache directory (e.g. `_extract/`, gitignored).

### 2. Rebuild `symbol-index.md`

Emit a markdown table:

- Columns: Path, Kind (`symbolKind` / article kind), Min platform (`platforms[0].introducedAt` when present), Title.

Sort paths lexicographically.

### 3. Fetch release notes

**Required URL:** `https://developer.apple.com/documentation/Updates/FoundationModels`

Summarize new bullets into `references/ios-26-4-release-notes-and-pending.md` (or add a new `ios-26-5-…` file if Apple publishes a new section).

### 4. Pending-symbol audit

Given a candidate list (e.g. `FoundationModelAgent`, `LatentMemory`, `ModelConfiguration`, user-reported betas):

- Grep the crawled JSON corpus for exact symbol strings.
- If absent: mark **Documentation pending** in the pending matrix—never invent signatures.

### 5. Spot-check high-traffic types

Manually re-fetch markdown or JSON for:

- `LanguageModelSession`
- `SystemLanguageModel`
- `Generable`
- `Tool`
- `LanguageModelSession/GenerationError`

Verify no signature drift against man-pages; update reference files if fragments changed.

### 6. Commit

- Branch: `update/foundation-models-apis-YYYY-MM`
- Files: `apple-foundation-models-skill/references/symbol-index.md`, delta markdown, any corrected man-pages.
- PR description lists **new paths** count and **removed** paths count vs. previous crawl.

## Sosumi MCP (optional)

| Tool | Use |
|------|-----|
| `searchAppleDocumentation` | Discover pages for “Foundation Models {topic}” |
| `fetchAppleDocumentation` | `path: /documentation/foundationmodels/...` |
| `fetchExternalDocumentation` | Full `https://developer.apple.com/...` URLs |

If Sosumi is unavailable, raw `curl` + JSON parse is sufficient per step 1.

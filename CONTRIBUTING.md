# Contributing

Thank you for helping improve this Agent Skill for Apple’s `FoundationModels` framework. This repository follows the [Agent Skills open format](https://agentskills.io/home); changes should stay machine-readable and accurate for LLM-assisted coding.

## Before you change anything

Read **[AGENTS.md](AGENTS.md)**. It defines the content contract: scope, correctness invariants, tone, and how `SKILL.md` and `references/` work together. Pull requests that conflict with AGENTS.md are unlikely to be merged.

## What to edit

| Area | Purpose |
|------|---------|
| [apple-foundation-models-skill/SKILL.md](apple-foundation-models-skill/SKILL.md) | Top-level operating rules, workflow, topic router, checklist. Keep it token-efficient. |
| [apple-foundation-models-skill/references/](apple-foundation-models-skill/references/) | Deep dives loaded on demand. One topic per file where possible. |
| [apple-foundation-models-skill/references/_index.md](apple-foundation-models-skill/references/_index.md) | Agent navigation; **must** stay in sync when files are added, renamed, or removed. |
| [.agents/skills/update-foundation-models-apis/](.agents/skills/update-foundation-models-apis/) | Meta-skill for refreshing docs-derived artifacts (symbol index, release notes). |
| [.claude-plugin/plugin.json](.claude-plugin/plugin.json), [.cursor-plugin/plugin.json](.cursor-plugin/plugin.json) | Plugin packaging; update only when skill paths or distribution metadata change. |

## Pull request checklist

- **Accuracy:** No invented APIs, parameters, or behaviors. Prefer Apple documentation or WWDC citations. If something cannot be verified, use: *"Technical details pending official release."*
- **Swift 6:** Examples use structured concurrency, correct isolation, and patterns that match the invariants in AGENTS.md (availability checks, `do/catch` for generation errors, no forbidden patterns listed there).
- **Style:** Terse, rule-based prose aimed at agents, not long-form tutorials.
- **Index and router:** If you add or rename a reference file, update `_index.md` and the **Topic Router** / **References** sections in `SKILL.md` as needed.
- **Markdown:** Keep fenced code blocks properly closed; verify tables and lists render in your editor or preview.

## Adding a new reference file

1. Add `apple-foundation-models-skill/references/<topic>.md`.
2. Add one row to `_index.md`.
3. Add a row (or bullet) in `SKILL.md` so the agent can route to the file.
4. Ensure content stays inside the scope described in AGENTS.md (and does not duplicate general Swift Concurrency / SwiftUI skills).

## Large doc refreshes (new OS / Xcode)

After Apple ships an iOS / Xcode drop that may change `FoundationModels`, use the maintenance workflow in [.agents/skills/update-foundation-models-apis/SKILL.md](.agents/skills/update-foundation-models-apis/SKILL.md) (see also [.agents/skills/update-foundation-apis.md](.agents/skills/update-foundation-apis.md)). Prefer updating generated or crawled artifacts in line with that skill rather than hand-editing symbol lists from memory.

## License

By contributing, you agree that your contributions will be licensed under the same terms as the project (see [LICENSE](LICENSE)).

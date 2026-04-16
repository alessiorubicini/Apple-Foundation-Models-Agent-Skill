# Apple Foundation Models Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![GitHub Release](https://img.shields.io/github/v/release/alessiorubicini/Apple-Foundation-Models-Agent-Skill)](https://github.com/alessiorubicini/Apple-Foundation-Models-Agent-Skill/releases)
[![GitHub Stars](https://img.shields.io/github/stars/alessiorubicini/Apple-Foundation-Models-Agent-Skill?style=flat)](https://github.com/alessiorubicini/Apple-Foundation-Models-Agent-Skill/stargazers)

> **Early version:** This skill is still in active development. It may lag behind the latest `FoundationModels` APIs, tools, or OS releases, and some material may be incomplete or incorrect. Treat it as guidance only. Verify behavior with [Apple’s Foundation Models documentation](https://developer.apple.com/documentation/foundationmodels) and your target SDK before you ship.

Expert guidance for any AI coding assistant that supports the [Agent Skills open format](https://agentskills.io/home) — unlocking the power of Apple's on-device generative AI via the iOS 26+ and macOS 26+ `FoundationModels` framework.

This repository aims to give your AI agent actionable, deeply technical references for building with `SystemLanguageModel`, with best-practice patterns for Swift 6 concurrency, hardware performance, structured data extraction, and tool calling.

## Who this is for
- Developers integrating Apple Intelligence into their native apps.
- Teams migrating server-side LLM calls to secure, on-device execution.
- Engineers navigating the strict concurrency and context-window limitations of local foundation models.
- Anyone building native AI features using `@Generable`, real-time streaming, and custom `Tool` implementations.

## How to Use This Skill

### Option A: Using skills.sh
Install this skill with a single command via the Agent Skills CLI:

```bash
npx skills add https://github.com/alessiorubicini/Apple-Foundation-Models-Agent-Skill --skill apple-foundation-models-skill
```

Then prompt your AI agent (e.g., Cursor, Claude, or Copilot):
> *"Use the apple foundation models skill and help me write a custom Tool for fetching weather data to inject into my LanguageModelSession."*

### Option B: Claude Code Plugin

To install this Skill for your personal use in Claude Code:

1. Add the marketplace:
```bash
/plugin marketplace add alessiorubicini/Apple-Foundation-Models-Agent-Skill
```

2. Install the Skill:
```bash
/plugin install foundation-models@apple-foundation-models-skill
```

#### Project Configuration (For Teams)
To automatically provide this Skill to everyone working in a repository, configure your project's `.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "foundation-models@apple-foundation-models-skill": true
  },
  "extraKnownMarketplaces": {
    "apple-foundation-models-skill": {
      "source": {
        "source": "github",
        "repo": "alessiorubicini/Apple-Foundation-Models-Agent-Skill"
      }
    }
  }
}
```

### Option C: Manual Install (Codex / OpenAI-compatible tools)
1. **Clone** this repository.
2. **Copy or symlink** the `apple-foundation-models-skill/` folder into your tool's designated skills directory. For example, with Codex:
```bash
cp -R apple-foundation-models-skill/ "$CODEX_HOME/skills/apple-foundation-models-skill"
```
3. **Use your AI tool** as usual and explicitly ask it to refer to the "apple foundation models skill".

**How to verify it's working**:
Your agent should start acknowledging hardware availability checks (`SystemLanguageModel.default.availability`), using `@Generable` for structured outputs, and wrapping LLM calls in proper Swift 6 `@MainActor` or `actor` isolations as defined in the `SKILL.md` workflow.

## What's Inside

This skill covers the entire surface area of the `FoundationModels` framework. Because reference files load on demand, your agent gets highly targeted, context-efficient guidance exactly when it needs it.

- **Core Models & Availability** — Managing `SystemLanguageModel` states, adapters (`.default` vs `.contentTagging`), and hardware eligibility.
- **Session Lifecycle** — Building stateful `LanguageModelSession` instances, crafting developer instructions, and managing transcript histories.
- **Guided Generation** — Forcing structural token output natively using the `@Generable` macro and `@Guide` constraints.
- **Tool Calling** — Expanding the model's capabilities with real-time data by implementing the `Tool` protocol.
- **Streaming** — Handling real-time UI updates with `streamResponse` and managing `PartiallyGenerated` types.
- **Error Handling & Fallbacks** — Proactive recovery strategies for `.exceededContextWindowSize` and unsupported locales.
- **Concurrency** — Swift 6 invariants, `Sendable` conformance, and preventing data races in session transcripts.
- **Performance** — Memory footprints, KV-cache behavior, and achieving sub-150ms latency using `prewarm()`.
- **Prompting Techniques** — On-device prompt design: clarity, roles, few-shot examples, reasoning fields, and code-side branching.

## Skill Structure

```text
apple-foundation-models-skill/
  SKILL.md
  references/
    _index.md                 - Quick navigation and file routing for the agent
    concurrency.md            - Actor isolation, Sendable tools, and @MainActor rules
    error-handling.md         - Context overflow recovery and availability fallbacks
    generation-options.md     - Temperature, sampling (.greedy/.random), and token limits
    glossary.md               - Canonical terminology for the framework
    guided-generation.md      - @Generable, @Guide, and DynamicGenerationSchema
    performance.md            - RAM footprint, token budgets, and prewarming techniques
    prompting-techniques.md   - On-device prompt design, roles, few-shot examples, reasoning fields
    session-lifecycle.md      - Instructions, transcript arrays, and session instantiation
    streaming.md              - Async streams and PartiallyGenerated UI handling
    system-language-model.md  - Hardware checks, locales, and model adapters
    tool-calling.md           - The Tool protocol, arguments, and execution delegates
```
## Contributing

Contributions, fixes, and improvements are highly encouraged! See **[CONTRIBUTING.md](CONTRIBUTING.md)** for the full guide: the content contract in [AGENTS.md](AGENTS.md), which files to edit, the pull request checklist, and how to refresh documentation after new OS or Xcode releases.

This repository adheres strictly to the [Agent Skills open format](https://agentskills.io/home), which dictates specific structural constraints to remain machine-readable.

## Acknowledgments

The technical guidelines encoded in this skill are derived entirely from Apple's official WWDC25 documentation and sessions, specifically:
- *Meet the Foundation Models framework*
- *Deep Dive into the Foundation Models Framework*

## License

This skill is open-source and available under the MIT License. See the [LICENSE](LICENSE) file for more information.
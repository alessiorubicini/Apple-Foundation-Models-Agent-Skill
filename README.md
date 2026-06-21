# Apple Foundation Models Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![GitHub Release](https://img.shields.io/github/v/release/alessiorubicini/Apple-Foundation-Models-Agent-Skill)](https://github.com/alessiorubicini/Apple-Foundation-Models-Agent-Skill/releases)
[![GitHub Stars](https://img.shields.io/github/stars/alessiorubicini/Apple-Foundation-Models-Agent-Skill?style=flat)](https://github.com/alessiorubicini/Apple-Foundation-Models-Agent-Skill/stargazers)

> **Beta warning:** Stable guidance targets the iOS 26+ and macOS 26+ `FoundationModels` framework. WWDC26 additions are beta-only and require Xcode 27 beta plus iOS 27.0 / macOS 27.0 / visionOS 27.0 / watchOS 27.0 beta unless explicitly noted. Treat beta APIs as provisional, guard generated Swift with `@available` or `#available`, and verify behavior with [Apple's Foundation Models documentation](https://developer.apple.com/documentation/foundationmodels) and your target SDK before shipping.

Expert guidance for any AI coding assistant that supports the [Agent Skills open format](https://agentskills.io/home), unlocking the power of Apple's generative AI through the `FoundationModels` framework, including stable OS 26 APIs and WWDC26 OS 27 beta APIs.

This repository aims to give your AI agent actionable, deeply technical references for building with `SystemLanguageModel`, `PrivateCloudComputeLanguageModel`, dynamic profiles, prompt attachments, session properties, structured generation, streaming, and tool calling.

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
- **WWDC26 / OS 27 Beta APIs** — Prompt attachments, Vision image-analysis tools, dynamic profiles, session properties, Private Cloud Compute, custom language model providers, metadata/context options, and beta error handling.

## Skill Structure

```text
apple-foundation-models-skill/
  SKILL.md
  references/
    _index.md                 - Quick navigation and file routing for the agent
    concurrency.md            - Actor isolation, Sendable tools, and @MainActor rules
    custom-language-model-provider.md - WWDC26 beta custom LanguageModel providers
    dynamic-profiles.md       - WWDC26 beta profiles, lifecycle hooks, and agentic patterns
    error-handling.md         - Context overflow recovery and availability fallbacks
    generation-options.md     - Temperature, sampling (.greedy/.random), and token limits
    glossary.md               - Canonical terminology for the framework
    guided-generation.md      - @Generable, @Guide, and DynamicGenerationSchema
    performance.md            - RAM footprint, token budgets, and prewarming techniques
    prompting-techniques.md   - On-device prompt design, roles, few-shot examples, reasoning fields
    private-cloud-compute.md  - WWDC26 beta PrivateCloudComputeLanguageModel guidance
    prompt-attachments.md     - WWDC26 beta image attachments and ImageReference
    session-lifecycle.md      - Instructions, transcript arrays, and session instantiation
    session-properties.md     - WWDC26 beta session-scoped state
    streaming.md              - Async streams and PartiallyGenerated UI handling
    system-language-model.md  - Hardware checks, locales, and model adapters
    tool-calling.md           - The Tool protocol, arguments, and execution delegates
    wwdc-2026-beta-apis.md    - WWDC26 beta routing map and source matrix
```
## Contributing

Contributions, fixes, and improvements are highly encouraged! See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide: the content contract in [AGENTS.md](AGENTS.md), which files to edit, the pull request checklist, and how to refresh documentation after new OS or Xcode releases.

This repository adheres strictly to the [Agent Skills open format](https://agentskills.io/home), which dictates specific structural constraints to remain machine-readable.

## Acknowledgments & Legal Disclaimer

This is an independent community-driven project and is not affiliated with, sponsored by, authorized by, or otherwise endorsed by Apple Inc. The technical guidelines, examples, and documentation templates encoded in this skill are compiled for informational and educational purposes only, derived from publicly available resources, official WWDC sessions, and Apple documentation.

Apple, Apple Intelligence, iOS, macOS, iPadOS, watchOS, tvOS, visionOS, Xcode, Swift, and Swift Playgrounds are trademarks of Apple Inc., registered in the U.S. and other countries. All other trademarks, service marks, product names, and company names or logos mentioned herein are the property of their respective owners.

## License

This skill is open-source and available under the MIT License. See the [LICENSE](LICENSE) file for more information.

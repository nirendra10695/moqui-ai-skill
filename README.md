# moqui-ai-skill

AI coding assistant skill documentation for the **Moqui Framework**. This component provides comprehensive reference guides that enable AI assistants (Claude, OpenAI Codex, Google Gemini) to generate correct Moqui entities, services, screens, and business logic.

## Installation

### Via Gradle (recommended)

```bash
cd moqui
./gradlew getComponent -Pcomponent=moqui-ai-skill
```

### Manual

```bash
cd moqui/runtime/component
git clone https://github.com/moqui/moqui-ai-skill.git
```

## What's Included

| File | Description |
|------|-------------|
| `SKILL.md` | Master skill reference — architecture, component structure, ExecutionContext API, quick-start patterns |
| `references/ENTITIES.md` | Entity XML definitions, field types, relationships, view-entities, extend-entity, EECA rules |
| `references/SERVICES.md` | Service XML definitions, XML Actions syntax, service types, SECA rules, REST API patterns |
| `references/SCREENS.md` | XML Screens, forms (form-single/form-list), transitions, widgets, layout, subscreens |
| `references/MANTLE.md` | Mantle UDM entity map, USL service catalog, status flows, Order-to-Cash / Procure-to-Pay |
| `references/MARBLE_ERP.md` | MarbleERP modules, screen hierarchy, extension patterns, customization recipes |
| `references/TESTING.md` | Spock test patterns, test suites, entity/service/screen/REST/flow tests, build.gradle setup |

## Usage with AI Assistants

After installing this component, place the appropriate instruction file at your **project root** so your AI tool auto-discovers it:

| AI Tool | Root Instruction File |
|---------|----------------------|
| Claude Code | `CLAUDE.md` |
| OpenAI Codex | `AGENTS.md` |
| Google Gemini | `GEMINI.md` |

These root-level files direct the AI to read the skill docs from `runtime/component/moqui-ai-skill/` before generating any Moqui code.

## Compatible AI Assistants

- **Claude Code** (Anthropic) — via `CLAUDE.md`
- **OpenAI Codex / ChatGPT** — via `AGENTS.md`
- **Google Gemini** — via `GEMINI.md`

## Version

- **1.0.0** — Initial release with complete skill documentation for entities, services, screens, Mantle, MarbleERP, and testing.

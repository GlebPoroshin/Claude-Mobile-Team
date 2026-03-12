# Mobile Dev Agents for Claude Code

Autonomous multi-agent development team for KMP (Kotlin Multiplatform), Android, and iOS projects. You act as a manager — agents handle everything from decomposition to code review.

## What This Is

A Claude Code plugin that turns Claude into an autonomous mobile development team:

- **System Analyst** — decomposes tasks into formal specs
- **Mobile Architect** — designs Clean Architecture solutions (api/data/presentation)
- **KMP Developer** — implements shared layer (DataSource → Repository → UseCase → ViewModel) + unit tests
- **Android Developer** — Jetpack Compose UI, navigation, platform code (+ unit tests for native Android)
- **iOS Developer** — SwiftUI/UIKit, navigation, actual implementations
- **Code Reviewer** — read-only quality gates (detekt, ktlint, architecture compliance)
- **QA Engineer** — manual testing on emulator/simulator via Claude in Mobile MCP + XcodeBuildMCP
- **AQA Engineer** — automated UI tests (Compose TestRule, XCUITest)
- **Knowledge Manager** — private knowledge base for decisions and patterns

## Architecture

```
You (Manager)
    │
    ▼
Orchestrator (/mobile-dev-agents:dev)
    │
    ├── System Analyst (opus)      → spec
    ├── Mobile Architect (opus)    → architecture doc
    ├── KMP Developer (sonnet)     → shared code + unit tests
    ├── Android Developer (sonnet) → Android UI (+ unit tests for native)
    ├── iOS Developer (sonnet)     → iOS UI
    ├── Code Reviewer (opus)       → review report
    ├── QA Engineer (opus)         → manual testing on emulator/simulator
    ├── AQA Engineer (sonnet)      → automated UI tests (optional)
    └── Knowledge Manager (haiku)  → knowledge base
```

## Tech Stack Support

| Layer | Technologies |
|-------|-------------|
| Shared Logic | KMP, KotlinX Serialization, Ktor, SqlDelight, MultiplatformSettings |
| DI | Koin, Kodein (auto-detected) |
| Android UI | Jetpack Compose, Material Design |
| iOS UI | SwiftUI, UIKit (auto-detected) |
| Navigation | Navigation 3, Cicerone, standard UIKit (auto-detected) |
| Testing | MockK, Turbine |
| Code Quality | detekt, ktlint |
| Architecture | Clean Architecture, MVI |

## Installation

### Option 1: From GitHub (Recommended)

```bash
# Add the marketplace
/plugin marketplace add GlebPoroshin/Claude-Mobile-Team

# Install the plugin
/plugin install mobile-dev-agents@claude-mobile-team
```

### Option 2: From Local Clone

```bash
git clone https://github.com/GlebPoroshin/Claude-Mobile-Team.git
/plugin marketplace add ./Claude-Mobile-Team
/plugin install mobile-dev-agents@claude-mobile-team
```

### Recommended MCP Servers

The agents work best with these MCP servers installed:

```bash
# Mobile emulator control (Android + iOS)
brew tap AlexGladkov/claude-in-mobile && brew install claude-in-mobile
claude mcp add --scope user mobile -- claude-in-mobile

# Up-to-date library docs
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp

# Google/Android/Firebase docs
claude mcp add --scope user google-dev-docs -- npx -y @anthropic/google-developer-knowledge-mcp

# API testing
claude mcp add --scope user postman -- npx -y @postmanlabs/postman-mcp-server

# OpenAPI specs
claude mcp add --scope user swagger -- npx -y swagger-mcp

# iOS build & test
brew tap getsentry/xcodebuildmcp && brew install xcodebuildmcp
claude mcp add --scope user xcode -- xcodebuildmcp mcp
```

### Recommended Plugins

```bash
# ast-index for fast code search (17-69x faster than grep)
/plugin marketplace add https://github.com/defendend/Claude-ast-index-search
/plugin install ast-index
```

## Usage

Run the pipeline command with your task and branch name:

```
/mobile-dev-agents:dev Add product catalog screen with filtering and pagination. Branch SHOP-1234-catalog
```

The agent team will autonomously:
1. Analyze the codebase and create a specification
2. Design the architecture (modules, interfaces, models)
3. Implement shared layer + unit tests (DataSource, Repository, UseCase, ViewModel)
4. Implement platform UI (Compose / SwiftUI / UIKit)
5. Run code review (detekt, ktlint, architecture compliance)
6. Fix any issues found
7. Manual QA testing on emulator/simulator (Claude in Mobile + XcodeBuildMCP)
8. Fix bugs found by QA
9. Automated UI tests (optional, Compose TestRule + XCUITest)
10. Deliver a clean branch with a report

You'll only be asked when external information is needed (API contracts, business logic decisions).

## Project Structure

```
claude-mobile-agents/
├── commands/                  # User-invocable commands
│   └── dev.md                 # /mobile-dev-agents:dev (pipeline entry point)
├── agents/                    # Agent definitions
│   ├── _shared-rules.md       # Common rules (referenced by all agents)
│   ├── system-analyst.md
│   ├── mobile-architect.md
│   ├── kmp-developer.md
│   ├── android-developer.md
│   ├── ios-developer.md
│   ├── code-reviewer.md
│   ├── qa-engineer.md
│   ├── aqa-engineer.md
│   └── knowledge-manager.md
├── skills/                    # Reusable skills
│   ├── mvi-viewmodel/         # MVI ViewModel scaffolding
│   ├── kmp-feature-module/    # Feature module scaffolding
│   ├── gitlab-ci-config/      # GitLab CI configuration
│   └── project-report/        # Final report generation
├── templates/                 # Document templates
│   ├── spec-template.md
│   ├── architecture-doc-template.md
│   ├── review-report-template.md
│   ├── bug-report-template.md
│   └── final-report-template.md
└── .claude-plugin/            # Plugin configuration
```

## Architecture Enforced

### Clean Architecture (Strict)
```
DataSource (network/DB/in-memory)
    ↓
Repository (aggregation, data→domain mapping)
    ↓
UseCase (business logic, single invoke)
    ↓
ViewModel (MVI: State + Events + Actions)
```

### Feature Module Structure
```
common/feature/{name}/
  api/          # Domain models, Repository/UseCase interfaces
  data/         # Implementation, DataSource, network models, mapping
  presentation/ # ViewModel (MVI), State, Event, Action
  di/           # DI module (Koin/Kodein)
```

### MVI ViewModel Contract
- One event handler method (`onEvent`)
- `viewState: StateFlow<State>` for UI (NOT `state`)
- `viewAction: SharedFlow<Action>` for one-shot side-effects (NOT `actions`)
- `when` on sealed event class → separate private method per event
- No other public methods

## Customization

### For Your Project

1. Adjust agent `.md` files for your conventions
2. Create project-specific knowledge in `~/.claude/knowledge/{project}/`
3. Override pipeline steps in your project's `CLAUDE.md` if needed

### Knowledge Base

Agents maintain a private knowledge base at `~/.claude/knowledge/{project}/`:
- `decisions.md` — architectural decisions
- `patterns.md` — established patterns
- `conventions.md` — naming and code style
- `integrations.md` — API contracts and backend specifics
- `pitfalls.md` — known issues and workarounds

This never enters git and is invisible to colleagues.

## Compatible Skills

These external skills work well with the agents:

| Skill | Source | What It Adds |
|-------|--------|-------------|
| android-agent-skills | [gecko23](https://github.com/gecko23/android-agent-skills) | Compose, MVVM/MVI, Clean Arch patterns |
| claude-android-ninja | [Drjacky](https://github.com/Drjacky/claude-android-ninja) | Navigation3, Material3, Detekt rules |
| coroutine-skills | [rcosteira79](https://github.com/rcosteira79/Coroutine-Skills) | Coroutines, Flow, KMP patterns |
| kmp-architecture | [ethanzhongch](https://github.com/ethanzhongch/Agent-Skills) | KMP module scaffolding |

## License

MIT

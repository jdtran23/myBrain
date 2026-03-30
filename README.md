# myBrain — Copilot Workspace Brain

A self-learning, multi-agent AI workspace powered by GitHub Copilot custom instructions, skills, and agents.

## Quick Start

1. Open this folder in VS Code: `code .`
2. Ensure GitHub Copilot + Copilot Chat extensions are installed
3. Select an agent (e.g., `@orchestrator`) and start working
4. Use `/orchestrate` for multi-step tasks

## Architecture

This brain uses a **Phase 3 Supervisor/Router hybrid** multi-agent orchestration pattern:

```
🎯 @orchestrator (lead — decomposes, delegates, validates)
 ├── 📋 @planner        → creates structured plans
 │    └── 📝 @plan-reviewer → validates plans before execution
 ├── 🔍 @researcher     → investigates topics, synthesizes findings
 ├── 💻 @coder          → writes, refactors, debugs code
 └── 🔎 @reviewer       → reviews code for bugs & security
```

## How It Works

| Layer | What It Does |
|-------|-------------|
| **Instructions** (`.github/instructions/`) | Coding standards & rules — loaded automatically every session |
| **Skills** (`.github/skills/`) | Active workflows — brain-manager, plan-management, orchestration-evolution |
| **Agents** (`.github/agents/`) | Specialist AI personas with clear scopes and handoffs |
| **Prompts** (`.github/prompts/`) | Reusable `/slash-command` templates |

### Key Principles
- **Generic vs Repo-Specific separation** — reusable knowledge stays portable, repo-specific stays focused
- **Two-tier directives** — thin always-loaded stubs + lazy-loaded detail files (keyword-triggered)
- **AI-DLC validation** — Plan → Generate → Verify → Iterate (never Plan → Generate → Done)

## Self-Learning

Your brain grows organically as you work:
- **Correct it:** "You made a mistake" → then "update your brain"
- **Teach it:** "Consume this page and learn" → then "update your brain"
- **Manage it:** "brain health" → scans and reports issues
- **Evolve it:** "orchestration check" → reviews agent architecture readiness

## Languages

Coding standards included for: **C#, Python, TypeScript, PowerShell**.

## Commands

| Command | What It Does |
|---------|-------------|
| `update your brain` | Saves lessons to workspace instructions |
| `brain health` | Scans directives for issues |
| `create a plan` | Creates a validated plan in `Plans/` |
| `orchestration check` | Reviews multi-agent maturity & recommends next steps |
| `/orchestrate` | Multi-agent workflow via the orchestrator |

## File Structure

```
myBrain/
├── .github/
│   ├── copilot-instructions.md                  # Main brain + scope guard
│   ├── instructions/                            # 11 files: standards & knowledge
│   │   ├── global.instructions.md               #   Team identity & environment
│   │   ├── csharp.instructions.md               #   C# coding standards
│   │   ├── python.instructions.md               #   Python coding standards
│   │   ├── typescript.instructions.md           #   TypeScript coding standards
│   │   ├── powershell.instructions.md           #   PowerShell scripting standards
│   │   ├── project.instructions.md              #   Project structure & build
│   │   ├── autonomous.instructions.md           #   Execution rules
│   │   ├── personality.instructions.md          #   📎 Clippy personality
│   │   ├── plan.instructions.md                 #   AI validation guardrail
│   │   ├── multi-agent-orchestration.instructions.md        # Orchestration stub
│   │   └── multi-agent-orchestration.detail.instructions.md # Deep reference
│   ├── skills/                                  # 3 skills
│   │   ├── brain-manager/SKILL.md               #   Self-learning & health checks
│   │   ├── plan-management/SKILL.md             #   Plan lifecycle & validation
│   │   └── orchestration-evolution/SKILL.md     #   Multi-agent growth advisor
│   ├── agents/                                  # 6 agents
│   │   ├── orchestrator.agent.md                #   🎯 Lead coordinator
│   │   ├── researcher.agent.md                  #   🔍 Research specialist
│   │   ├── coder.agent.md                       #   💻 Coding specialist
│   │   ├── reviewer.agent.md                    #   🔎 Code review specialist
│   │   ├── planner.agent.md                     #   📋 Plan creation specialist
│   │   └── plan-reviewer.agent.md               #   📝 Plan validation specialist
│   ├── prompts/
│   │   └── orchestrate.prompt.md                #   /orchestrate shortcut
│   └── tsg.instructions.md                      # Troubleshooting & lessons learned
├── Plans/                                       # Active task plans
├── .vscode/
│   ├── mcp.json                                 # Playwright MCP config
│   └── settings.json                            # Workspace settings
└── README.md
```

## Sources & References

Built using patterns from:
- [VS Code Copilot Custom Instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [Anthropic: Multi-Agent Research System](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Microsoft Multi-Agent Reference Architecture](https://github.com/microsoft/multi-agent-reference-architecture)
- [Azure AI Agent Orchestration Patterns](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns)
- [Google: Agentic AI Design Patterns](https://docs.cloud.google.com/architecture/choose-design-pattern-agentic-ai-system)

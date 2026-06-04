# copilot-self-improving

A Copilot Skill that reviews recent chat history across all VS Code workspaces, identifies skill optimization opportunities, proposes new skills, and extracts knowledge nuggets into a personal knowledge base. Auto-applies approved changes with audit logging.

## About

Most Copilot users write skills once and never improve them. This skill closes the loop — it periodically scans your actual conversations, finds patterns, and suggests concrete improvements:

- Detect missing triggers (you asked a question a skill should have handled, but different wording was used)
- Identify repeated multi-step workflows that should become new skills
- Extract domain knowledge nuggets into a reusable knowledge base
- Track skill ideas that don't yet have enough evidence ("deferred opportunities")

## Getting Started

### Prerequisites

- [VS Code](https://code.visualstudio.com/) 1.99+
- [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) extension (active subscription)
- Python >= 3.10

### Installation

1. **Clone the skill into your Copilot skills directory**

   ```bash
   # Windows
   git clone https://github.com/JackySummerfield/copilot-self-improving.git "%USERPROFILE%\.copilot\skills\copilot-self-improving"

   # macOS / Linux
   git clone https://github.com/JackySummerfield/copilot-self-improving.git ~/.copilot/skills/copilot-self-improving
   ```

2. **Initialize state files (auto-created on first run, or manually)**

   ```bash
   cd ~/.copilot/skills/copilot-self-improving

   # Review state tracker (records which sessions have been reviewed)
   echo '{"reviewed_sessions": {}, "last_review": null}' > review_state.json

   # Change audit log
   echo "# Skill Change Log" > skill_change_log.md

   # Review reports directory
   mkdir -p reviews
   ```

3. **Create the knowledge base directory (if it doesn't exist)**

   ```bash
   # Windows
   mkdir "%USERPROFILE%\.copilot\knowledge"

   # macOS / Linux
   mkdir -p ~/.copilot/knowledge
   ```

4. **Verify installation**

   Open VS Code Copilot Chat and say:

   ```
   每日回顾
   ```

   The skill should run the collector script and either report "No New Chat History" or present review findings.

## Usage

Trigger the skill with any of these phrases:

| Trigger | Description |
|---------|-------------|
| `每日回顾` / `daily review` | Run full review cycle |
| `skill review` / `技能优化` | Focus on skill optimization |
| `知识提取` / `knowledge extraction` | Focus on knowledge nuggets |
| `复盘` / `回顾总结` | General review |
| `self-improving` / `自我改进` | Same as daily review |

### Workflow Overview

| Step | Action |
|------|--------|
| 0 | Initialize state files (first-time only) |
| 1 | Collect un-reviewed chat sessions via Python script |
| 2 | Read all existing skills to understand current landscape |
| 3 | Cross-reference chat patterns → identify opportunities |
| 4 | Present structured report with numbered suggestions |
| 5 | Apply user-approved changes (modify skills, create knowledge entries) |
| 6 | Mark sessions reviewed, save report to `reviews/` |

### What It Detects

- **Missing triggers** — you used a phrase the skill should respond to but doesn't
- **Workflow gaps** — manual steps that could be automated
- **New skill opportunities** — repeated multi-step patterns across sessions
- **Knowledge nuggets** — technical troubleshooting, domain knowledge, best practices

## File Structure

```
copilot-self-improving/
├── SKILL.md                      # Skill definition and workflow
├── README.md                     # This file
├── LICENSE                       # MIT license
├── review_state.json             # Tracks reviewed session IDs + line counts
├── skill_change_log.md           # Audit log of all modifications
├── deferred_opportunities.md     # Skill ideas waiting for more evidence
├── references/                   # Reference materials
├── scripts/
│   └── collect_chat_history.py   # Collects un-reviewed chat transcripts
└── reviews/
    └── review_YYYY-MM-DD.md      # Generated review reports
```

### Related Directories

```
~/.copilot/knowledge/             # Personal knowledge base (auto-populated)
├── _index.md                     # Topic index
├── plant-simulation.md           # Example topic file
└── ...
```

## Roadmap

- [ ] Support reviewing Claude Code / Cursor chat history formats
- [ ] Confidence scoring for suggestions (based on evidence count)
- [ ] Auto-detect skill conflicts or overlapping triggers
- [ ] Weekly summary digest across multiple daily reviews

## Acknowledgments

- [Best-README-Template](https://github.com/othneildrew/Best-README-Template) — README structure reference

## License

MIT — see [LICENSE](LICENSE).

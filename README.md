# daily-skill-review

A Copilot Skill that reviews recent chat history across all VS Code workspaces, identifies skill optimization opportunities and new skill ideas, and auto-applies approved changes with audit logging.

## Features

- Scans all VS Code workspace chat transcripts (JSONL format)
- Session-based deduplication — only processes un-reviewed conversations
- Cross-references chat patterns against existing skills
- Presents actionable suggestions with evidence from actual conversations
- Auto-modifies skills after user confirmation
- Maintains a change audit log and dated review reports

## Usage

In VS Code Copilot Chat, use any of these trigger phrases:

- `每日回顾` / `daily review` / `skill review` / `技能优化` / `回顾总结`

## Requirements

- Python >= 3.10
- VS Code with GitHub Copilot Chat extension

## File Structure

```
daily-skill-review/
├── SKILL.md                              # Skill definition and workflow
├── review_state.json                     # Tracks reviewed session IDs
├── skill_change_log.md                   # Audit log of all skill modifications
├── references/
│   └── collect_chat_history.py           # Python script to collect chat history
└── reviews/
    └── review_YYYY-MM-DD.md             # Generated review reports
```

## License

MIT — see [LICENSE](LICENSE).

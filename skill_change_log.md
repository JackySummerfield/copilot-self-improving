# Skill Change Log

This file records all skill modifications and creations made through the daily review process.

---

## 2026-05-08 — doc-to-knowledgebase — Workflow Enhancement
- **Reason**: Agent repeatedly skipped skill steps; markitdown output lacked heading structure for large PDFs; filenames with spaces broke Markdown links
- **Evidence**: Session cd30a81a (PlantSim Knowledge Base workspace)
- **Changes Made**:
  - Added Mandatory Checklist at top of SKILL.md for step-by-step self-checking
  - Added PDF TOC Heading Injection option to Step 4d with reference script
  - Added filename convention (spaces → underscores) to Step 5
  - Created `references/inject_headings_example.py`

## 2026-05-08 — vscode-extension-dev — New Skill Created
- **Reason**: Repeated complex workflows around VS Code extension packaging, native module handling, and corporate network issues
- **Evidence**: Sessions 5cd6abe8, 42119dd3 (PlantSim to VSCode workspace)
- **Changes Made**:
  - Created `~/.copilot/skills/vscode-extension-dev/SKILL.md`
  - Covers: Extension Development Host, vsce packaging, npm SSL/CA, native module ABI, lazy loading

## 2026-05-08 — notebook-to-script — New Skill Created
- **Reason**: Notebook cell code integration into Python scripts had multiple failure modes (function signature mismatch, variable scope, schema cascading)
- **Evidence**: Session 10469742 (M&S Auto Run Architecture workspace)
- **Changes Made**:
  - Created `~/.copilot/skills/notebook-to-script/SKILL.md`
  - Covers: dependency analysis, function encapsulation, execution order, downstream impact verification

---


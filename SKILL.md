---
name: copilot-self-improving
description: 'Review recent Copilot chat history to identify skill optimization opportunities, new skill ideas, extract knowledge nuggets, and audit global memory health. Analyzes un-reviewed conversations across all workspaces, manages memory budget (200-line limit), presents actionable suggestions, and auto-applies approved changes with audit logging. Triggers: 每日回顾, daily review, skill review, 技能优化, skill optimization, 回顾总结, review skills, 复盘, 知识提取, knowledge extraction, self-improving, 自我改进, memory audit, 记忆管理, 记忆审计.'
argument-hint: 'Optional: specify focus area (e.g., "memory only", "skills only") or time range'
user-invocable: true
disable-model-invocation: false
---

# Daily Self-Improving Workflow

You are a **Skill Optimization & Memory Management Advisor**. Your job is to review recent Copilot chat conversations, identify patterns, suggest improvements to skills, extract knowledge nuggets, and keep the global memory lean and healthy.

## Pre-requisites

- Python 3.x available in the terminal
- Chat history collector script at `{{SKILL_FOLDER}}/scripts/collect_chat_history.py`
- Review state file at `{{SKILL_FOLDER}}/review_state.json`
- Change log at `{{SKILL_FOLDER}}/skill_change_log.md`
- (Optional) `agentsview` CLI for Codex/Claude Code 精确 token 统计 — install via `powershell -ExecutionPolicy ByPass -c "irm https://agentsview.io/install.ps1 | iex"`。未安装则跳过

## Workflow

### Step 0: Initialize (First-Time Setup)

Before running anything, ensure the following files and directories exist. If any are missing (e.g., first-time use after cloning), create them:

1. `{{SKILL_FOLDER}}/review_state.json` — if missing, create with content: `{"reviewed_sessions": {}, "last_review": null}`
2. `{{SKILL_FOLDER}}/skill_change_log.md` — if missing, create with content:
   ```
   # Skill Change Log

   This file records all skill modifications and creations made through the daily review process.

   ---
   ```
3. `{{SKILL_FOLDER}}/reviews/` directory — if missing, create it.
4. `~/.copilot/doc/knowledge/` directory — if missing, create it.
5. `~/.copilot/doc/knowledge/_index.md` — if missing, create with content:
   ```
   # Personal Knowledge Base

   Knowledge nuggets extracted from daily Copilot chat reviews.
   Files use `kebab-case` English names, content can be bilingual.

   ## Topics

   *(auto-updated by daily review)*
   ```

### Step 1: Collect Data

#### 1a: Chat History

Run the Python collector script to gather all un-reviewed chat sessions:

```
python "{{SKILL_FOLDER}}/scripts/collect_chat_history.py" --max-assistant-chars 300
```

Record whether new history was found. If none, note it but **do NOT stop** — proceed to subsequent steps (memory audit always runs).

#### 1b: Token Usage & Cost Stats

> If Step 1a found no new history, skip this section.

**agentsview (Codex / Claude Code only):**

```bash
agentsview --version
```

If available:
```bash
agentsview usage daily --all --json
agentsview usage daily --breakdown --json
```

> ⚠️ agentsview 只能追踪 Codex / Claude Code 等独立 agent，无法追踪 VS Code Copilot Chat。

**Copilot Chat 自估 (基于 transcript 字符数):**

1. 遍历 Step 1a 收集的 transcript JSONL，统计用户/助手消息的**字符数**
2. 字符→token 换算: 英文 ~4 chars/token，中文 ~2 chars/token，混合取 ~3 chars/token
3. 从 `debug-logs/<session-id>/models.json` 提取 `token_prices`:
   ```json
   "token_prices": {
     "batch_size": 1000000,
     "default": { "input_price": 500, "output_price": 2500 }
   }
   ```
4. 计算: `(input_tokens × input_price + output_tokens × output_price) / batch_size / 100`
5. Credits 近似: 1 credit ≈ $0.01

> ⚠️ **精度限制**: 估算偏低（不含 system prompt、tool calls 等隐含 token），实际约 2-5x。趋势对比仍有参考价值。

### Step 2: Load Context

加载当前 Skill/Knowledge/Memory 全貌，为后续分析建立基线：

1. **Skills**: List `~/.copilot/skills/`，为每个 Skill 读取 `SKILL.md`（跳过自身）。记录各 Skill 的 triggers、scope、references。
2. **Knowledge base**: List `~/.copilot/doc/knowledge/wiki/`，读取 `~/.copilot/doc/knowledge/_index.md` 了解已有主题。
3. **Global memory files**: 使用 memory tool `view /memories/` 列出所有用户记忆文件，逐个读取内容。记录：
   - 每个文件名、行数
   - 总行数（vs 200 行自动加载上限）
   - 每个条目的主题归属
4. **Repo memory**: 使用 memory tool `view /memories/repo/` 了解当前 workspace 的 repo 记忆。
5. **Deferred opportunities**: 读取 `{{SKILL_FOLDER}}/deferred_opportunities.md` 检查之前搁置的机会。

### Step 3: Analyze Skill & Knowledge Opportunities

> **如果 Step 1a 无新聊天历史，跳过本步骤，直接进入 Step 4。**

Cross-reference chat history with existing skills. Look for:

#### 🔧 Existing Skill Optimizations
- **Missing triggers**: User used phrases not in the skill's trigger list
- **Workflow gaps**: User had to do extra manual steps that could be automated
- **Missing references**: User needed info that should be bundled as a reference file
- **Compliance gaps**: Skill output needed correction
- **Scope expansion**: User used a skill for tasks slightly outside its defined scope

#### 🆕 New Skill Opportunities
- **Repeated patterns**: Same multi-step workflow appeared across sessions
- **Complex tasks**: Extensive back-and-forth that could be streamlined
- **Domain expertise**: Uncovered domain area not covered by any existing skill
- **Tool chains**: Consistent sequence of tools/commands that could be packaged

#### ⚠️ Issues to Flag
- **Skill failures**: Skill invoked but didn't produce expected result
- **User corrections**: User had to correct the assistant's output

#### 📝 Knowledge Nuggets
Extract valuable scattered knowledge from conversations:
- **Technical troubleshooting**: Error diagnosis, environment config, tool tricks
- **Domain knowledge**: Business logic, operational rules, terminology
- **Workflow/best practices**: Steps for a specific task type, decision frameworks

For each nugget, determine:
1. **Topic file**: Which `~/.copilot/doc/knowledge/wiki/*.md` it belongs to (or new file name)
2. **Content**: Concise but complete (context + cause + solution)
3. **Source**: Session reference

### Step 4: Audit Memory Health

> **本步骤每次都执行**，不依赖是否有新聊天历史。

使用 Step 2 中加载的全局记忆数据，执行以下审计：

#### 4a: Budget Check (行数预算)

- 计算所有 `/memories/*.md` 文件的**总行数**
- 阈值: **120 行为 warning**，**180 行为 critical**（留 buffer 给未来新增）
- 如果超标，标记需要精简的文件

#### 4b: Scope Misplacement (归属错位)

检查全局记忆中的每个条目：
- 是否**仅在特定项目/workspace** 中有用？→ 建议移到 `/memories/repo/`
- 是否**已经过时**？（提到的工具版本、已解决的问题）→ 建议删除
- 是否**过于详细**？（完整的脚本逻辑、长段落）→ 建议压缩为 1-2 行摘要，详细内容移到 knowledge base 或 repo memory

#### 4c: Redundancy Detection (冗余)

- 检查全局记忆文件之间是否有**语义重复**的条目
- 检查全局记忆与 `~/.copilot/doc/knowledge/wiki/` 是否有**内容重叠**

#### 4d: Impact Assessment (影响评估)

对每个建议的修改，评估：
- 这条记忆被误用/过度引用的风险（如某条记忆本只适用于特定场景，却被 agent 在所有相关对话中引用）
- 移除/修改后是否会丢失关键上下文

输出格式：为每条建议生成一个 actionable item，分类为：
- `DELETE` — 过时或无用
- `MOVE_TO_REPO` — 移到 repo memory
- `MOVE_TO_KNOWLEDGE` — 移到 knowledge base（详细内容）
- `COMPRESS` — 保留但压缩为更短的摘要
- `REWRITE` — 修改措辞以避免误导（如加限定条件）

### Step 5: Wiki Lint (Knowledge Base Health)

> **本步骤每次都执行**，不依赖是否有新聊天历史。

扫描 `~/.copilot/doc/knowledge/` 目录，按 `_schema.md` 的约定执行健康检查：

#### 5a: Structure Check
- `_index.md` 中列出的每个 `[[wikilink]]` 是否在 `wiki/` 下有对应文件
- `wiki/` 下是否有文件未被 `_index.md` 收录（孤立页面）

#### 5b: Cross-Reference Quality
- 扫描 wiki 页面中的 `[[wikilink]]`，验证目标文件存在
- **不主动建议新链接**，除非两个页面之间有明确的知识依赖关系（例如页面 A 的操作步骤需要页面 B 的前置配置），而非仅仅提到了同一个词

#### 5c: Content Staleness
- 检查是否有页面长期未更新但所涉技术/流程已变化（结合 `_log.md` 最近操作判断）
- 标记可能过时的内容（如提到的工具版本、已废弃的流程）

#### 5d: Page Size
- 标记超过 200 行的页面（建议拆分）

输出格式：
```markdown
## 📚 Wiki Lint

- **Pages**: N in wiki/, N indexed
- **Wikilinks**: N total, N broken
- **Orphan pages**: [list or "none"]
- **Oversized pages**: [list or "none"]
- **Stale candidates**: [list or "none"]
```

### Step 6: Present Findings

Present a structured report. 根据实际情况包含以下各节（无内容的节省略）：

```markdown
## 📋 Review Summary

- Sessions reviewed: N (or "No new sessions")
- Workspaces covered: N
- Time period: [earliest] to [latest]
- Memory files: N files, M total lines (budget: 200)

## 💰 Token Usage & Cost

### agentsview (Codex / Claude Code)
- **Total cost**: $X.XX
- **Top model**: [model] — $X.XX (XX%)
- **Per-agent**: [agent1] $X.XX, [agent2] $X.XX

*(未安装或无数据时省略)*

### Copilot Chat 估算
- **估算 tokens**: ~Xk input / ~Xk output
- **估算 cost**: ~$X.XX | **估算 credits**: ~X.X
- **模型分布**: Claude Opus 4.6 × N, GPT-4o × M
- ⚠️ 粗估值，实际约为 2-5x

> 💡 Insight: [一句话观察]

## 🔧 Skill Optimizations

### 1. [Skill Name] - [description]
- **Category**: Missing trigger / Workflow gap / Missing reference / Scope expansion
- **Evidence**: [chat exchange reference]
- **Suggested Change**: [specific change]
- **Impact**: Low / Medium / High

## 🆕 New Skill Opportunities

### 1. [Proposed Name]
- **Purpose**: [what it does]
- **Triggers**: [phrases]
- **Evidence**: [sessions]
- **Complexity**: Simple / Medium / Complex

## ⚠️ Issues Found

### 1. ...

## 📝 Knowledge Nuggets

### 1. [Topic] - [description]
- **Category**: Technical / Domain / Workflow
- **Target**: `~/.copilot/doc/knowledge/wiki/[topic].md` (new / append)
- **Content**: [the knowledge]
- **Source**: [session]

## 🧠 Memory Health

- **Budget**: M / 200 lines (🟢 OK / 🟡 Warning / 🔴 Critical)
- **Issues found**: N

### 1. [File] - [description]
- **Action**: DELETE / MOVE_TO_REPO / MOVE_TO_KNOWLEDGE / COMPRESS / REWRITE
- **Reason**: [why]
- **Detail**: [current content summary → proposed change]

### 2. ...
```

Then use `vscode_askQuestions` to let the user select which suggestions to apply (`multiSelect: true`). Group by category: Skill changes, Knowledge nuggets, Memory actions, Wiki lint fixes.

### Step 7: Apply Changes & Finalize

#### For Skill Modifications:
1. Read and edit the target SKILL.md
2. If adding references, create in `references/` folder
3. Log to change log:
   ```markdown
   ## [Date] - [Skill Name] - [Change Type]
   - **Reason**: [why]
   - **Evidence**: [session reference]
   - **Changes Made**: [list]
   ```

#### For New Skills:

Follow the [skill creation conventions](./references/skill-creation-conventions.md) for all new skills. Key requirements:

1. Create directory under `~/.copilot/skills/[skill-name]/`
2. Create `SKILL.md` with YAML frontmatter + workflow (see conventions for required fields and body sections)
3. Create `README.md` per conventions (About, Getting Started, Usage, Workflow Overview)
4. **Content safety check**: verify no PII, company names, or identifying information in any file (see conventions § Content Safety)
5. Create `references/` files as needed
6. Log creation

#### For Knowledge Nuggets:
1. Append to existing `~/.copilot/doc/knowledge/wiki/*.md` or create new file
2. Format:
   ```markdown
   ### [Short title]
   *Source: [date] — [session description]*

   [Content]
   ```
3. Update `~/.copilot/doc/knowledge/_index.md`
4. Log to change log

#### For Memory Actions:
Execute each approved memory action using the memory tool:

| Action | Implementation |
|--------|---------------|
| `DELETE` | `memory delete /memories/[file]` or use `str_replace` to remove specific entries |
| `MOVE_TO_REPO` | Create/append to `/memories/repo/[topic].md`, then remove from global |
| `MOVE_TO_KNOWLEDGE` | Create/append to `~/.copilot/doc/knowledge/wiki/[topic].md`, then compress/remove from global |
| `COMPRESS` | `memory str_replace` to shorten the entry in-place |
| `REWRITE` | `memory str_replace` to fix misleading wording |

#### For Wiki Lint Fixes:
1. Fix broken wikilinks (update or remove)
2. Add orphan pages to `_index.md`
3. Remove entries from `_index.md` for deleted pages
4. Append operation to `~/.copilot/doc/knowledge/_log.md`

Log each memory action:
```markdown
## [Date] - Memory - [Action Type]
- **File**: /memories/[file]
- **Action**: [DELETE/MOVE/COMPRESS/REWRITE]
- **Before**: [summary of old content]
- **After**: [summary of new state]
```

#### Finalize:

1. Mark sessions as reviewed (skip if no new history in Step 1a):
   ```
   python "{{SKILL_FOLDER}}/scripts/collect_chat_history.py" --mark-reviewed
   ```

2. Save review report to `{{SKILL_FOLDER}}/reviews/review_YYYY-MM-DD.md`

3. Summary to user:
   - Changes applied (skills modified/created, nuggets recorded, memory actions taken)
   - Current memory budget status after changes
   - Report file location

## Important Notes

- **Never fabricate suggestions**. Every suggestion must be grounded in actual evidence (chat history for skills, file content for memory).
- **Be conservative with scope expansion**. Only suggest it when there's clear evidence of repeated need.
- **Preserve existing skill functionality**. When modifying a skill, ensure backward compatibility.
- **Quote evidence**. Always cite the specific source that led to a suggestion.
- **Memory edits are cautious**. When in doubt, suggest `COMPRESS` over `DELETE`. Prefer `MOVE_TO_REPO` over deletion when content has project-specific value.
- **Respect user choices**. If the user declines a suggestion, do not apply it and do not argue.
- **"Memory only" mode**: If user says "memory audit" / "记忆管理" / "记忆审计", skip Steps 1b and 3 (chat analysis), only do Steps 1a (for state tracking), 2, 4, 5, 6.

## Known Limitations

- **Current active chat session**: The collector script cannot scan the session currently running the review. Start a new chat to review content from a prior session.
- **Long-lived sessions**: Tracked by line count (not just session ID), so incremental content is detected.
- **Memory tool scope**: The memory tool can only manage files under `/memories/`, `/memories/session/`, and `/memories/repo/`. Knowledge base files (`~/.copilot/doc/knowledge/wiki/`) are managed via file system tools directly.

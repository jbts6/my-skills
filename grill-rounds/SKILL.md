---
name: grill-rounds
description: Self-contained multi-round, multi-session grilling protocol. Combines (a) interview semantics — challenge glossary, sharpen terminology, stress-test scenarios, cross-ref code, update CONTEXT.md inline, offer ADRs sparingly — with (b) a persistent backlog ledger (GRILL_BACKLOG.md), one-question-at-a-time rhythm, accept-then-write commits, per-round commits + hash backfill, and sub-question vote parsing (1:X;2:Y). Use when grilling a large GDD/design corpus over many sessions and the user needs cross-session continuation, audit trail, and disciplined commit hygiene.
---

<what-to-do>

Run a disciplined, ledger-backed grill session against a large design corpus (GDD, ADR set, docs/) that will take many sessions to stress-test, and whose decisions must survive across `/clear` and context compactions.

**Core protocol (every round):**

1. **Open** — Read `docs/GRILL_BACKLOG.md`. Confirm the next candidate with the user or pick one from "未触及" / "未完候选" sections. State the candidate and its sub-questions in one sentence.
2. **One question at a time** — Ask the first sub-question with your recommendation + the main tradeoff (one or two sentences). Wait. Do not batch sub-questions in a single message. If a question can be answered by exploring the codebase or existing docs, explore instead of asking.
3. **Apply interview semantics while asking** — during each sub-question:
   - **Challenge the glossary** — if the user uses a term that conflicts with `CONTEXT.md`, call it out: "Your glossary defines 'cancellation' as X, but you seem to mean Y — which is it?"
   - **Sharpen fuzzy terms** — propose a precise canonical term: "You're saying 'account' — do you mean Customer or User?"
   - **Stress-test with scenarios** — invent concrete edge cases that force precision about boundaries.
   - **Cross-reference code** — if the user states how something works, check whether the code agrees; surface contradictions.
4. **Parse the answer** — User typically replies in short imperatives: `接受` / `修正` / `写` / `1:X;2:Y` (per-sub-question vote). When `1:X;2:Y` appears, treat as parallel decisions on the sub-questions in the order you asked them.
5. **Accept-then-write** — Only after verbal acceptance, write to the authoritative docs (02_domain-model.md, systems/*.md, tech/*.md, CONTEXT.md). Do not write during the proposal phase. When a term resolves, update `CONTEXT.md` inline using the format in [CONTEXT-FORMAT.md](./CONTEXT-FORMAT.md). When a decision passes the three-condition ADR test (see [ADR-FORMAT.md](./ADR-FORMAT.md)), offer an ADR; otherwise just edit docs.
6. **Close the round** — When all sub-questions of the candidate resolve:
   a. Update `docs/GRILL_BACKLOG.md` — mark sub-issues `[Round N 已完成]` (keep original text), append a new row to the round table with `Hash: TBD`.
   b. Commit with prefix `docs: GDD grill round N - <theme>`. Stage only docs files; never auto-stage `.claude/settings.local.json` or untracked binaries.
   c. Read the new commit hash. Edit GRILL_BACKLOG.md to replace `TBD` with the real hash. Commit `docs: 回填 round N commit hash 至 GRILL_BACKLOG`.

**When the user says "结束/留档/下次继续":**

- Do not write a session summary memo into a new file. The state IS in GRILL_BACKLOG + git log.
- Give one short message: latest commit hashes, the next-round candidate shortlist (pulled from BACKLOG's "未触及" and "未完候选" sections), and any new workflow-level memory you saved.

</what-to-do>

<supporting-info>

## GRILL_BACKLOG.md schema

Lives at `docs/GRILL_BACKLOG.md`. Has these sections:

```markdown
# GDD Grill 待办

> 本文件记录历次 grill 会话未走完的候选项，供后续接续 grill。
> 完成的 grill 会话在 git history 中追溯（commit message 前缀 `docs: GDD grill round N`）。

---

## 已完成的 N 轮 grill

| Round | Hash | 主题 |
|-------|------|------|
| 1 | `abcd123` | 主题一句话 |
| ... | ... | ... |

---

## 当前未提交 grill 进度

（可选段——只在工作区有已决议但未 commit 的内容时存在，commit 后整段删除）

---

## <分段> 未完候选

### <候选编号>.<子编号> 候选标题 [Round N 已完成 | 待 grill]

简述 + Q1/Q2/Q3 + 决议引用文档链接。完成后保留全文 + 加 `[Round N 已完成]` 标记，不删除。

---

## 未触及的 GDD 系统

| 系统 | 文档 | 潜在 grill 点 |

---

## 数值/平衡阶段待定项

下列在 grill 中被识别但留待平衡阶段确定的具体数值：…

---

## 已识别但未深挖的边角

零散观察，未必成轮，下次 grill 时回顾。

---

## 治理 / 命名层观察

跨轮浮现的 doc governance / 工具链 / 校验债务模式。

---

## 已识别的构建期校验债务

文档承诺"由构建期校验保证"但 tools/ 未实现的项。每条注明出处 + 需补的 validator。
```

Create the file lazily if missing on the first round. Keep entries even after completion — they become an audit trail, not a TODO list.

## Round commit message convention

```
docs: GDD grill round N - <短主题，≤30字>

- 决议要点 1（带文档路径）
- 决议要点 2
- 决议要点 N
- 边角扩散修改（重命名 / 跨文档措辞统一）
- GRILL_BACKLOG 状态更新
```

Hash backfill commit is one line:

```
docs: 回填 round N commit hash 至 GRILL_BACKLOG
```

## Answer parsing rules

| User says | Meaning |
|---|---|
| `接受` | Accept the single proposal as stated. Proceed to write. |
| `修正` / `修` | Proposal needs adjustment — wait for clarification, do not write yet. |
| `写` | Proceed to write. Usually comes as a second turn after `接受`. |
| `1:X;2:Y` | Per-sub-question vote. Apply to sub-questions in the order they were asked. `X` / `Y` can be `接受` / `修正` / a short directive ("写清" / "A"). |
| `先 X，再 Y` | Sequencing directive — execute X completely (commit + backfill) before opening Y. |
| `<候选编号>` (e.g. `F.7`) | Switch to that candidate after the current round closes. |

When the user replies with a free-form follow-up question instead of one of these, treat it as a clarifying question on the current sub-question, not a vote. Answer it, then re-ask the sub-question.

## Two-phase rhythm: propose → accept → write

The rhythm has a deliberate gap between `propose` and `write`:

- **Propose** — recommendation + 1-2 sentences of tradeoff. No file writes. No exploration of alternative solutions in depth unless asked.
- **Accept** — user replies `接受` / `修正` / sub-question vote. If `修正`, return to propose with a new framing.
- **Write** — only after acceptance. Write to all affected docs in one batch of Edit calls. Use parallel tool calls for independent edits.

Do not collapse the gap. Writing during propose-phase wastes effort on rejected proposals. Proposing during write-phase muddles the audit trail.

## Interview semantics (in-round behavior)

These are the four things to do while a sub-question is open, not before or after:

### 1. Challenge against the glossary

When the user uses a term that conflicts with the existing language in `CONTEXT.md`, call it out immediately. "Your glossary defines 'cancellation' as X, but you seem to mean Y — which is it?"

### 2. Sharpen fuzzy language

When the user uses vague or overloaded terms, propose a precise canonical term. "You're saying 'account' — do you mean the Customer or the User? Those are different things."

### 3. Discuss concrete scenarios

When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about the boundaries between concepts.

### 4. Cross-reference with code

When the user states how something works, check whether the code agrees. If you find a contradiction, surface it: "Your code cancels entire Orders, but you just said partial cancellation is possible — which is right?"

### Update CONTEXT.md inline

When a term is resolved, update `CONTEXT.md` right there. Don't batch these up — capture them as they happen. Use the format in [CONTEXT-FORMAT.md](./CONTEXT-FORMAT.md).

`CONTEXT.md` should be totally devoid of implementation details. Do not treat it as a spec, a scratch pad, or a repository for implementation decisions. It is a glossary and nothing else.

## When to offer an ADR vs. just edit docs

Three-condition test (all must be true) — see [ADR-FORMAT.md](./ADR-FORMAT.md) for details:

1. **Hard to reverse** — cost of changing your mind later is meaningful.
2. **Surprising without context** — a future reader will wonder "why did they do it this way?"
3. **The result of a real trade-off** — there were genuine alternatives and you picked one for specific reasons.

If any of the three is missing, skip the ADR and just edit `docs/` directly. In a multi-round grill project this rarely fires — most rounds are terminology sharpening or rule landing, not architectural inflection points. Reserve ADRs for the 1-in-10 round that has all three traits.

If unsure, ask the user explicitly: "这一轮要不要落 ADR？我倾向不落，理由是 …" — let them overrule.

## File structure assumptions

Most repos have a single context:

```
/
├── CONTEXT.md
├── docs/
│   ├── GRILL_BACKLOG.md
│   ├── design/                       ← player-facing rules
│   ├── tech/                         ← technical implementation
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

If a `CONTEXT-MAP.md` exists at the root, the repo has multiple contexts. The map points to where each one lives.

Create files lazily — only when you have something to write. If no `CONTEXT.md` exists, create one when the first term is resolved. If no `docs/adr/` exists, create it when the first ADR is needed.

## Cross-session continuation

The skill is designed for sessions that exhaust their context window. On resume:

1. Read `docs/GRILL_BACKLOG.md` first. It is the single source of truth for "where we were."
2. Read recent git log filtered by `grep "GDD grill round"` — the last 2-3 rounds tell you the live conversation tone.
3. Do not read full transcripts. Do not re-derive decisions from chat history. The decisions are in the docs.
4. Greet the user with: "上次落到 round N (`<hash>`)，<本轮主题>。下一轮候选：…" and let them pick.

## What NOT to do in this skill

- **Do not** write a "session summary" markdown file. The summary IS the commit + the BACKLOG row.
- **Do not** auto-skip `accept` phase even when the user's previous answers establish a pattern. Each sub-question gets its own consent.
- **Do not** batch multiple round-closings into one commit. One round = one main commit + one backfill commit. Strictly.
- **Do not** delete completed entries from BACKLOG. Mark them `[Round N 已完成]` and keep the original text.
- **Do not** invent candidates the user didn't raise. The BACKLOG grows from user input + discovered debt, not from the model's preferences.
- **Do not** use destructive git operations (reset --hard, force push). Round commits are immutable history.
- **Do not** write CONTEXT.md entries that are implementation details. CONTEXT.md is a glossary, full stop.

## When to save memory vs. when to commit to docs

| Belongs in docs | Belongs in memory |
|---|---|
| Domain decisions, terminology, ADRs | Cross-session workflow preferences |
| GRILL_BACKLOG state | Recurring user-style cues (e.g., "user replies in 1:X;2:Y format") |
| All grill outcomes | Design philosophy that influences how to grill (e.g., "data > code" — push proposals toward config) |

Memory is for *how the user grills*, not *what was decided*. Decisions live in git.

</supporting-info>

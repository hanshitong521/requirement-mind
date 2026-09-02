# Requirement Gate 规则（Phase 7）

目标：机器可复核的开发准入判定。Gate 不 PASS，不允许开发（R1）。

## 硬门槛（任一 > 0 → BLOCKED）

| 计数 | 来源 |
|---|---|
| Blocking Questions | questions.json 中 status=OPEN 且 priority=BLOCKING |
| Blocking Conflicts | conflicts.json 中 status=OPEN 且 severity=BLOCKING |
| Critical Assumptions | assumptions.json 中 risk=HIGH 且 status=UNVERIFIED |
| Unvalidated High Risks | challenges.json 中 status=PENDING_VALIDATION 的高危项 |

复核：`node $SKILL/scripts/state.mjs gate .requirementmind`

## 检查表（16 项，全部 PASS 才 READY）

Business Goal / Scope / Core Flow / Business Rules / Data Model / API Contract / State Machine / Validation Rules / Permission / Concurrency / Idempotency / Exception Handling / Compatibility / Acceptance Criteria / Testability / Spec Consistency。

每项判定为 PASS 的标准：DEVELOPMENT_SPEC.md 对应章节**存在、非空、可追溯到 FACT/DEC、内部无矛盾**。Spec Consistency 特指：规格各章节之间、规格与冻结决策之间无矛盾（supersede 后最容易在这里漏）。

## 输出 `gate.json`

```json
{
  "status": "READY_FOR_DEVELOPMENT | BLOCKED",
  "checklist": { "business_goal": "PASS", "...": "PASS" },
  "hard_counters": { "blocking_questions": 0, "blocking_conflicts": 0,
                     "critical_assumptions": 0, "unvalidated_high_risks": 0 },
  "missing": [],
  "checked_at": "...",
}
```

- BLOCKED 时 `missing` 必须逐条列出缺失项与对应 id，让用户知道差什么。
- gate.json 写入后快照到 `history/`，阶段游标置 `GATED`。

## BLOCKED 之后

向用户展示缺失清单 → 缺问题回 Phase 3 追问，缺验证回 Phase 6，冲突回 Phase 3 显式裁决 → 修复后重跑 Gate。**不允许**因为"差不多了"而放行。

## READY 之后

进入 Prompt Export（SKILL.md 收尾节）。开发期若发现新业务未知项：标记 DEVELOPMENT_BLOCKER，gate 状态回 BLOCKED，退回本流程。

# Decision Freezer 规则（Phase 3 内）

目标：用户的每一个回答都变成不可静默覆盖的冻结决策。

## 冻结动作

用户回答后立即写入 `decisions.json`：

```json
{
  "id": "DEC-007",
  "question_id": "Q-007",
  "decision": "REQUEST_RECEIVED_AT",
  "topic": "expiration_boundary",
  "source": "USER",
  "status": "FROZEN",
  "impact": ["concurrency.check()", "red_packetMapper.updateStatus"],
  "created_at": "2026-08-30T12:00:00+08:00"
}
```

- `impact` 尽量列出该决策会影响的项目位置（来自 facts.json），供下游重审用。
- 同时把对应 question 置 status=ANSWERED。
- 快照到 `history/`。

## Supersede（用户改变主意）

```json
{ "id": "DEC-010", "status": "SUPERSEDED", "replaced_by": "DEC-014" }
```

- 旧记录**不删除不修改原值**，只置 SUPERSEDED。
- 新决策 DEC-014 正常冻结。
- 引用旧决策的所有下游（其他决策、spec 章节）标记待重审，触发全量重解析。

## 硬约束

- 用户明确回答优先级最高，高于任何文档、代码注释、AI 推断。
- 冻结决策不可被 Agent 静默覆盖（R5）。若后续代码证据与冻结决策冲突：产生 CONFLICT，摆到用户面前显式裁决，不许偷偷换。
- 冲突的最终裁决（保留旧决策 / 修改决策 / 修改代码预期）也要作为新 DEC 冻结。

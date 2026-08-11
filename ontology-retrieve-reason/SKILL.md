---
name: ontology-retrieve-reason
description: Use when a business question needs evidence-bound retrieval from the published ontology and selected LLM Wiki pages.
---

# 本体检索增强推理

依据已发布本体和可追溯 Wiki 片段组织辅助推理。当前版本不调用真实工商、司法、税务或征信接口；外部数据只有在用户显式提供且授权、主体一致时才可进入上下文。

## 调用方式

```bash
python3 tools/ontology_modeling_agent.py query \
  "请评估某企业是否适合工行普惠续贷" \
  --company "某企业" --dry-run --json
```

## 输出契约

必须同时输出：

- `facts`：有来源的事实。
- `inferences`：基于事实和本体路径的推断。
- `recommendations`：建议下一步，不是审批结论。
- `unknowns`：缺失、冲突或被排除的数据。
- `citations`：Wiki 路径/章节或本体 IRI。
- `human_review_required: true` 和非自动决策声明。

## 安全边界

候选本体、未授权数据、主体不匹配数据和过期证据不得作为事实。证据冲突返回 `CONFLICT`，证据不足返回 `NOT_ENOUGH_DATA`，不得用模型常识填补关键授信数据。

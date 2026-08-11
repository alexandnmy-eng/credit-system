---
name: ontology-ingest-wiki
description: Use when approved raw Markdown or text materials must be turned into reviewable LLM Wiki pages with source metadata.
---

# 原始材料到 LLM Wiki

将用户显式指定的 Markdown 或纯文本材料整理成可审阅的 Wiki 页面。当前版本不自动扫描 `raw/`，不读取未指定的真实材料；PDF、Word、Excel 和图片仅返回待扩展提示。

## 调用方式

```bash
python3 tools/ontology_modeling_agent.py ingest \
  --source raw/已批准材料.md --dry-run --json
```

确认输入、来源和授权后，去掉 `--dry-run` 才允许写入 `知识库/导入/`。

## 输出要求

- 页面包含 `ontology_id`、`type`、`domain`、`source`、`source_hash`、`review_state`。
- 新内容默认标记 `NEEDS_REVIEW`。
- 原始文件保持不变。
- 不把模型补写内容标为原文事实；缺失信息写入待核验项。

## 边界

禁止无输入地执行全库扫描，禁止覆盖已有页面，禁止把 OCR 低置信结果直接发布为确认知识。

---
name: ontology-compile-ttl
description: Use when selected LLM Wiki Markdown must be structured and compiled into a traceable candidate Turtle ontology.
---

# LLM Wiki 到 TTL

把用户显式指定的 Wiki 页面编译为候选中间层 JSON 和 TTL。当前版本只处理指定页面，不默认读取全部 `知识库/`，也不把真实企业数据写入本体。

## 调用方式

```bash
python3 tools/ontology_modeling_agent.py compile-ttl \
  --input 知识库/示例规则.md --dry-run --json
```

确认抽取范围后，去掉 `--dry-run` 生成 `本体产物/候选/中间层/` 和 `本体产物/候选/ttl/`。

## 抽取边界

- 页面标题映射为候选类。
- 二级及以下章节映射为规则节点。
- 每条规则关联来源路径、章节定位和内容哈希。
- 显式 `ontology_id` 优先；未确认标识不得静默改名。
- 事实、推断、建议和未知项保持分离。

## 发布边界

TTL 只是候选产物，必须经过 `ontology-export-owl` 和 `ontology-validate-shacl`。发现冲突输出 `CONFLICT`，证据不足输出 `NOT_ENOUGH_DATA`。

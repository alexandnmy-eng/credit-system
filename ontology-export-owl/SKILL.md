---
name: ontology-export-owl
description: Use when a candidate Turtle graph needs OWL semantic checks and RDF/XML OWL serialization.
---

# TTL 到 OWL

对用户显式指定的 TTL 执行最小 OWL 语义检查，再输出 RDF/XML `.owl`。这一步不是改文件扩展名，而是重新解析并比较规范化 RDF 图摘要。

## 调用方式

```bash
python3 tools/ontology_modeling_agent.py export-owl \
  --input 本体产物/候选/ttl/示例.ttl --dry-run --json
```

确认输入后去掉 `--dry-run`。缺少 `owl:Ontology`、TTL 解析失败或 TTL/RDFXML 图不一致时阻断。

## 输出要求

- `.owl` 使用 RDF/XML 序列化。
- 生成 OWL 检查报告，记录三元组数量和 TTL/OWL 图摘要。
- 不加入 Wiki 或配置中没有依据的强语义公理。

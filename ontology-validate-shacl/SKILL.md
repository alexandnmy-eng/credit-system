---
name: ontology-validate-shacl
description: Use when a candidate RDF graph and SHACL shapes must be checked before ontology publication.
---

# SHACL 校验与发布闸门

对用户显式指定的数据图和 Shapes 执行 SHACL 校验。初稿至少检查规则陈述、规则证据、证据来源路径和章节定位。

## 调用方式

```bash
python3 tools/ontology_modeling_agent.py validate \
  --data 本体产物/候选/ttl/示例.ttl \
  --shapes 本体产物/候选/shacl/核心约束.ttl \
  --dry-run --json
```

去掉 `--dry-run` 后生成 JSON/Markdown 报告。`sh:Violation` 返回退出码 3，并禁止进入 `本体产物/已发布/`；警告是否阻断由配置决定。

## 审核边界

缺少来源、授权、版本或人工审核状态时保留阻断状态。SHACL 通过也不等于授信通过，最终业务决定必须由授权人员完成。

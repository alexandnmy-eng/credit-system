# 信贷本体建模智能体

一个面向信贷场景的本体建模智能体初稿，提供一个逻辑 Agent 和五个可跨 Codex、Claude Code、OpenCode 复用的 Skill。

[English README](README_EN.md) | [上游参考项目](https://github.com/alexandnmy-eng/credit-system)

## 项目定位

本项目参考并本地化了开源项目 [alexandnmy-eng/credit-system](https://github.com/alexandnmy-eng/credit-system) 的 Skill 化本体工程思路，将业务知识处理链组织为：

```text
用户指定材料
    -> LLM Wiki Markdown
    -> 结构化中间层
    -> 候选 TTL
    -> OWL RDF/XML
    -> SHACL 校验
    -> 已发布本体
    -> 有证据的检索增强推理
```

当前版本优先实现功能闭环，不自动读取或编译项目中的真实业务材料，也不连接真实工商、司法、税务、征信接口。

## 核心能力

| 能力 | Skill | 作用 |
|---|---|---|
| 原始材料入库 | `ontology-ingest-wiki` | 将用户明确指定的 Markdown/文本整理为可审阅 Wiki 页面 |
| Wiki 编译 TTL | `ontology-compile-ttl` | 从指定 Wiki 页面抽取类、规则、证据并生成候选 Turtle |
| TTL 导出 OWL | `ontology-export-owl` | 校验 `owl:Ontology` 并导出 RDF/XML，比较图摘要一致性 |
| SHACL 校验 | `ontology-validate-shacl` | 校验规则陈述、来源证据、定位信息并阻断违规发布 |
| 检索增强推理 | `ontology-retrieve-reason` | 基于已发布本体和 Wiki 来源生成事实、推断、建议与未知项 |

## 一个逻辑 Agent

逻辑 Agent 名称为 `ontology-modeler`，三个运行时只提供薄适配，不复制业务逻辑：

- Agent 说明：[agents/ontology-modeler.md](agents/ontology-modeler.md)
- Codex：[.codex/agents/ontology-modeler.toml](.codex/agents/ontology-modeler.toml)
- Claude Code：[.claude/agents/ontology-modeler.md](.claude/agents/ontology-modeler.md)
- OpenCode：[.opencode/agents/ontology-modeler.md](.opencode/agents/ontology-modeler.md)

五个 Skill 的正式来源是 `[.agents/skills/](.agents/skills/)`。Claude Code 的目录通过符号链接兼容，OpenCode 直接使用 `.agents/skills/`。

## 快速开始

### 1. 安装依赖

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
```

入口脚本会优先使用项目 `.venv`；也可以直接使用：

```bash
python3 tools/ontology_modeling_agent.py doctor --json
```

### 2. 检查运行时发现

```bash
python3 tools/ontology_modeling_agent.py doctor --json
```

该命令检查 Python、RDFLib、pySHACL、五个 Skill 以及 Codex/Claude Code/OpenCode Agent 入口。

### 3. 只对指定材料做 dry-run

```bash
python3 tools/ontology_modeling_agent.py ingest \
  --source raw/已批准材料.md --dry-run --json

python3 tools/ontology_modeling_agent.py compile-ttl \
  --input 知识库/指定页面.md --dry-run --json
```

初稿不会因为执行 `ingest` 或 `compile-ttl` 而自动扫描全部 `raw/` 或 `知识库/`。确认输入、来源和授权后，再去掉 `--dry-run`。

## 统一 CLI

```text
python3 tools/ontology_modeling_agent.py <command> [options]
```

| 命令 | 说明 |
|---|---|
| `doctor` | 检查依赖、目录、Skill 和运行时 Agent |
| `ingest` | 指定 Markdown/文本材料生成 Wiki 页面 |
| `compile-ttl` | 指定 Wiki 页面生成中间层 JSON 和候选 TTL |
| `export-owl` | 指定 TTL 生成 RDF/XML OWL |
| `validate` | 使用指定 SHACL Shapes 校验数据图 |
| `pipeline` | 串联 TTL、OWL 和 SHACL，默认只写候选区 |
| `query` | 基于配置的 Wiki 和已发布本体生成结构化辅助推理 |

所有会写入产物的命令都支持 `--dry-run`。机器集成建议使用 `--json`。

## 典型流水线

```bash
python3 tools/ontology_modeling_agent.py compile-ttl \
  --input 知识库/工行普惠续贷规则.md \
  --name 工行普惠续贷初稿 \
  --dry-run --json

python3 tools/ontology_modeling_agent.py export-owl \
  --input 本体产物/候选/ttl/工行普惠续贷初稿.ttl \
  --dry-run --json

python3 tools/ontology_modeling_agent.py validate \
  --data 本体产物/候选/ttl/工行普惠续贷初稿.ttl \
  --shapes 本体产物/候选/shacl/核心约束.ttl \
  --dry-run --json
```

去掉 `--dry-run` 后，产物进入 `本体产物/候选/`，通过校验并完成人工审核后才能发布。

## 输出语义

检索推理结果保持以下字段分离：

- `facts`：有来源的事实。
- `inferences`：基于本体路径和事实的推断。
- `recommendations`：建议的下一步动作，不是审批结果。
- `unknowns`：缺失、冲突、未授权或被排除的数据。
- `citations`：Wiki 路径/章节或本体 IRI。
- `human_review_required`：授信相关结果默认为 `true`。

关键状态包括：

- `CONFLICT`：证据之间存在冲突。
- `NOT_ENOUGH_DATA`：缺少足够证据，不能可靠判断。
- `MANUAL_REVIEW_REQUIRED`：需要授权人员复核。

本智能体不自动作出授信、拒贷、欺诈认定或客户处置决定。

## 目录结构

```text
.
├── agents/ontology-modeler.md
├── .agents/skills/                 # 五个 Skill 的唯一正式来源
├── .codex/agents/                  # Codex 适配
├── .claude/agents/                 # Claude Code 适配
├── .claude/skills/                 # 指向 .agents/skills 的兼容链接
├── .opencode/agents/               # OpenCode 适配
├── agent_runtime/ontology_modeling/ # 共享 Python 能力核心
├── tools/ontology_modeling_agent.py
├── raw/                            # 原始材料，默认不自动扫描
├── 知识库/                         # LLM Wiki
└── 本体产物/                       # 候选、发布和报告
```

## 当前边界

- 初稿摄取支持 Markdown 和纯文本；PDF、Word、Excel、图片 OCR 需要后续适配。
- 外部工商、司法、税务、征信 Skill 目前只有契约边界，没有真实连接配置。
- 默认不对真实企业进行授信判断。
- SHACL 初稿覆盖基础结构和证据存在性，不能替代正式行内制度规则。
- `owl` 导出使用 RDF/XML；TTL 与 OWL 之间进行图摘要一致性检查。

## 安全原则

1. 原始材料不原地修改。
2. 候选本体和已发布本体隔离。
3. 证据不足时不使用模型常识补齐关键事实。
4. 外部数据必须满足授权、用途和主体一致性。
5. API Key 只从环境变量读取，不写入 README、Skill 或本体产物。

## 测试

项目使用临时 fixture 验证配置、Markdown、TTL、OWL、SHACL、发布闸门、检索推理和运行时适配：

```bash
PYTHONWARNINGS=ignore .venv/bin/python \
  -m unittest discover -s tests -p 'test_*.py' -v
```

## 上游参考

本项目的 Skill 分层、候选本体、OWL/SHACL 闸门和检索增强组织方式参考：

- [alexandnmy-eng/credit-system](https://github.com/alexandnmy-eng/credit-system)
- [ontology-ingest-wiki](https://github.com/alexandnmy-eng/credit-system/tree/main/ontology-ingest-wiki)
- [ontology-compile-ttl](https://github.com/alexandnmy-eng/credit-system/tree/main/ontology-compile-ttl)
- [ontology-export-owl](https://github.com/alexandnmy-eng/credit-system/tree/main/ontology-export-owl)
- [ontology-validate-shacl](https://github.com/alexandnmy-eng/credit-system/tree/main/ontology-validate-shacl)
- [ontology-retrieve-reason](https://github.com/alexandnmy-eng/credit-system/tree/main/ontology-retrieve-reason)

上游仓库声明采用 MIT License；当前项目的许可证以本项目实际声明为准。

# Credit Ontology Modeling Agent

An initial ontology-modeling agent for credit scenarios. It provides one logical agent and five reusable skills for Codex, Claude Code, and OpenCode.

[中文 README](README.md) | [Upstream reference project](https://github.com/alexandnmy-eng/credit-system)

## Project Positioning

This project localizes the skill-oriented ontology engineering approach from [alexandnmy-eng/credit-system](https://github.com/alexandnmy-eng/credit-system) into an ICBC inclusive-credit workflow:

```text
User-selected material
    -> LLM Wiki Markdown
    -> Structured intermediate representation
    -> Candidate Turtle
    -> OWL RDF/XML
    -> SHACL validation
    -> Published ontology
    -> Evidence-bound retrieval and reasoning
```

The current version prioritizes a functional first draft. It does not automatically read or compile real business materials in this workspace, and it does not connect to live business-registry, judicial, tax, or credit-reporting services.

## Capabilities

| Capability | Skill | Purpose |
|---|---|---|
| Raw material intake | `ontology-ingest-wiki` | Turn explicitly selected Markdown/text into reviewable Wiki pages |
| Wiki to TTL | `ontology-compile-ttl` | Extract classes, rules, evidence, and provenance into candidate Turtle |
| TTL to OWL | `ontology-export-owl` | Check `owl:Ontology`, export RDF/XML, and compare graph digests |
| SHACL validation | `ontology-validate-shacl` | Validate rule statements and evidence before publication |
| Retrieval reasoning | `ontology-retrieve-reason` | Produce facts, inferences, recommendations, unknowns, and citations |

## One Logical Agent

The logical agent is `ontology-modeler`. The three runtimes use thin adapters and do not duplicate business logic:

- Agent definition: [agents/ontology-modeler.md](agents/ontology-modeler.md)
- Codex: [.codex/agents/ontology-modeler.toml](.codex/agents/ontology-modeler.toml)
- Claude Code: [.claude/agents/ontology-modeler.md](.claude/agents/ontology-modeler.md)
- OpenCode: [.opencode/agents/ontology-modeler.md](.opencode/agents/ontology-modeler.md)

The canonical skill source is `[.agents/skills/](.agents/skills/)`. Claude Code uses symlinked compatibility entries, while OpenCode reads `.agents/skills/` directly.

## Quick Start

### 1. Install dependencies

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
```

The entrypoint automatically prefers the project `.venv` when it exists:

```bash
python3 tools/ontology_modeling_agent.py doctor --json
```

### 2. Check runtime discovery

```bash
python3 tools/ontology_modeling_agent.py doctor --json
```

This checks Python, RDFLib, pySHACL, the five skills, and the Codex/Claude Code/OpenCode agent adapters.

### 3. Dry-run only selected inputs

```bash
python3 tools/ontology_modeling_agent.py ingest \
  --source raw/approved-material.md --dry-run --json

python3 tools/ontology_modeling_agent.py compile-ttl \
  --input 知识库/selected-page.md --dry-run --json
```

The first draft does not scan all of `raw/` or `知识库/` merely because `ingest` or `compile-ttl` is invoked. Remove `--dry-run` only after the input, provenance, and authorization have been reviewed.

## Unified CLI

```text
python3 tools/ontology_modeling_agent.py <command> [options]
```

| Command | Description |
|---|---|
| `doctor` | Check dependencies, directories, skills, and runtime agents |
| `ingest` | Create Wiki pages from explicitly selected Markdown/text |
| `compile-ttl` | Compile selected Wiki pages into IR JSON and candidate Turtle |
| `export-owl` | Export RDF/XML OWL from a selected Turtle graph |
| `validate` | Validate a selected data graph with SHACL shapes |
| `pipeline` | Run TTL, OWL, and SHACL in sequence; candidate output by default |
| `query` | Produce structured reasoning from configured Wiki and published ontology |

All writing commands support `--dry-run`. Use `--json` for machine-readable integration.

## Example Pipeline

```bash
python3 tools/ontology_modeling_agent.py compile-ttl \
  --input 知识库/icbc-renewal-rule.md \
  --name icbc-renewal-draft \
  --dry-run --json

python3 tools/ontology_modeling_agent.py export-owl \
  --input 本体产物/候选/ttl/icbc-renewal-draft.ttl \
  --dry-run --json

python3 tools/ontology_modeling_agent.py validate \
  --data 本体产物/候选/ttl/icbc-renewal-draft.ttl \
  --shapes 本体产物/候选/shacl/core-shapes.ttl \
  --dry-run --json
```

After review, remove `--dry-run`. Artifacts are written to `本体产物/候选/`; publication requires successful validation and human review.

## Reasoning Contract

The retrieval skill keeps these fields separate:

- `facts`: sourced observations.
- `inferences`: reasoning based on ontology paths and facts.
- `recommendations`: next actions, not approval decisions.
- `unknowns`: missing, conflicting, unauthorized, or excluded data.
- `citations`: Wiki paths/sections or ontology IRIs.
- `human_review_required`: `true` for credit-facing results by default.

Important statuses include:

- `CONFLICT`: evidence is contradictory.
- `NOT_ENOUGH_DATA`: evidence is insufficient for a reliable conclusion.
- `MANUAL_REVIEW_REQUIRED`: an authorized reviewer must decide the next step.

The agent never makes an automated credit approval, rejection, fraud, or customer-treatment decision.

## Directory Layout

```text
.
├── agents/ontology-modeler.md
├── .agents/skills/                  # canonical five skills
├── .codex/agents/                   # Codex adapter
├── .claude/agents/                  # Claude Code adapter
├── .claude/skills/                  # compatibility symlinks
├── .opencode/agents/                # OpenCode adapter
├── agent_runtime/ontology_modeling/ # shared Python runtime
├── tools/ontology_modeling_agent.py
├── raw/                             # raw materials; not auto-scanned by intake
├── 知识库/                          # LLM Wiki
└── 本体产物/                        # candidate, published, and reports
```

## Current Boundaries

- Initial intake supports Markdown and plain text; PDF, Word, Excel, and image OCR need later adapters.
- Business-registry, judicial, tax, and credit-reporting integrations are contracts only; no live connectors are configured.
- The initial agent does not make real-company credit decisions.
- The initial SHACL profile covers basic structure and evidence presence, not formal bank policy.
- OWL output uses RDF/XML and checks graph digest equivalence with the input Turtle graph.

## Safety Principles

1. Raw materials are not modified in place.
2. Candidate and published ontology artifacts are separated.
3. Missing evidence is not filled with model common sense.
4. External data requires authorization, purpose, and subject matching.
5. API keys are read from environment variables and are not stored in README, skills, or ontology artifacts.

## Tests

The project uses temporary fixtures to test configuration, Markdown, TTL, OWL, SHACL, publication gates, retrieval reasoning, and runtime adapters:

```bash
PYTHONWARNINGS=ignore .venv/bin/python \
  -m unittest discover -s tests -p 'test_*.py' -v
```

## Upstream Reference

The skill decomposition, candidate ontology boundary, OWL/SHACL gate, and retrieval organization are based on:

- [alexandnmy-eng/credit-system](https://github.com/alexandnmy-eng/credit-system)
- [ontology-ingest-wiki](https://github.com/alexandnmy-eng/credit-system/tree/main/ontology-ingest-wiki)
- [ontology-compile-ttl](https://github.com/alexandnmy-eng/credit-system/tree/main/ontology-compile-ttl)
- [ontology-export-owl](https://github.com/alexandnmy-eng/credit-system/tree/main/ontology-export-owl)
- [ontology-validate-shacl](https://github.com/alexandnmy-eng/credit-system/tree/main/ontology-validate-shacl)
- [ontology-retrieve-reason](https://github.com/alexandnmy-eng/credit-system/tree/main/ontology-retrieve-reason)

The upstream repository declares the MIT License. The license for this project is governed by this project's own license statement.

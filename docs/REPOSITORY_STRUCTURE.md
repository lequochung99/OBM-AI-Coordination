# OBM AI Coordination Repository Structure

```text
OBM-AI-Coordination/
├── AGENTS.md
├── CURRENT.md
├── README.md
├── constitution/
│   ├── CURRENT.md
│   └── V001/
│       └── OBM_AI_COORDINATION_CONSTITUTION.md
├── state/
│   ├── CURRENT_TASK.json
│   └── AGENT_STATUS.md
├── tasks/
│   ├── README.md
│   ├── active/
│   │   └── CURRENT.md
│   └── archive/
├── prompt/                         Existing numbered prompts
├── report/                         Existing numbered reports
├── context/
│   ├── README.md
│   ├── project/
│   │   └── PROJECT_INDEX.md
│   ├── modules/
│   │   └── README.md
│   └── investigations/
│       └── INDEX.md
├── graphify/
│   ├── README.md
│   ├── CURRENT.md
│   ├── manifests/
│   │   └── GRAPH_MANIFEST.template.json
│   ├── summaries/
│   ├── deltas/
│   └── private-artifact-references/
├── decisions/
│   ├── INDEX.md
│   └── DECISION001-shared-coordination-structure.md
├── templates/
│   ├── README.md
│   ├── TASK_TEMPLATE.md
│   ├── REPORT_TEMPLATE.md
│   ├── INVESTIGATION_TEMPLATE.md
│   ├── MODULE_CONTEXT_TEMPLATE.md
│   └── GRAPH_DELTA_TEMPLATE.md
└── docs/
    └── REPOSITORY_STRUCTURE.md
```

Git does not store empty folders. `archive/`, `summaries/`, `deltas/`, and `private-artifact-references/` appear when their first versioned artifact is created.

## Agent startup sequence

```text
AGENTS.md
-> CURRENT.md
-> constitution/CURRENT.md
-> state/CURRENT_TASK.json
-> active TASK.md
-> referenced module/investigation/graph/report only
```

## Task communication sequence

```text
ChatGPT creates complete numbered prompt + task packet
-> Cursor implements and updates reusable knowledge
-> Cursor writes numbered report and pushes branch
-> ChatGPT reviews report/diff/context changes
-> Operator performs UI/physical acceptance
-> task is archived and CURRENT pointers advance
```

Codex later joins the same flow as hard-task executor or independent reviewer.

## Graphify sequence

```text
Source commit changes
-> Graphify incremental refresh
-> versioned manifest records source SHAs
-> public-safe summary/delta is committed
-> sensitive raw graph remains private with checksum/reference
-> task reads only affected module context
```

## Ownership boundary

| Area | Primary writer | Review/approval |
|---|---|---|
| Constitution and decisions | ChatGPT/coordinator | Operator |
| Prompt and task packet | ChatGPT/coordinator | Operator when material |
| Source implementation | Cursor or Codex | Risk-based reviewer |
| Report and evidence references | Executor | ChatGPT |
| Module/investigation updates | Executor | ChatGPT |
| Graphify raw generation | Operator/tool | Sanitization required before public commit |
| UI and physical runtime result | Operator | Final authority |

# Opencode Multi-Agent Virtual Engineering Team V2.2

> DBA & QA Enhanced Edition

## Overview

This repository contains the configuration for a multi-agent virtual engineering team built on [opencode](https://opencode.ai). It implements a closed-loop development workflow with 14 specialized agents covering management, architecture, database, engineering, quality assurance, security, and infrastructure layers.

## Quick Start

### Global Installation (Recommended)

Copy the configuration to your global opencode config directory:

```bash
# Windows
copy opencode.json %USERPROFILE%\.config\opencode\opencode.json
xcopy /E /Y .opencode\agents\*.md %USERPROFILE%\.config\opencode\agents\

# Linux/macOS
cp opencode.json ~/.config/opencode/opencode.json
cp -r .opencode/agents/*.md ~/.config/opencode/agents/
```

### Project-level Installation

Place `opencode.json` and `.opencode/agents/` in your project root.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Management & Architecture                 │
│         pmo ──> pm ──> architect ──> dba                    │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                    Engineering (Parallel)                    │
│              backend-dev ∥ frontend-dev                      │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                    Quality Gate                              │
│         reviewer ──> qa-engineer                            │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                    Infrastructure                            │
│              memory ──> report                              │
└─────────────────────────────────────────────────────────────┘
```

## Agents

| Layer | Agent | Role | Output |
|-------|-------|------|--------|
| Management | `pmo` | Project Management Office | `docs/project-plan.md` |
| Management | `pm` | Product Manager | `docs/prd.md` |
| Architecture | `architect` | System Architect | `docs/architecture.md`, `docs/api-spec.md` |
| Database | `dba` | Database Engineer | `docs/database.md` |
| Engineering | `backend-dev` | Backend Engineer | `src/` |
| Engineering | `frontend-dev` | Frontend Engineer | `frontend/` |
| Quality | `reviewer` | Code Reviewer | `docs/review-report.md` |
| Quality | `qa-engineer` | QA Engineer | `docs/test-plan.md`, `docs/test-report.md` |
| Security | `security` | Compliance Security Officer | `docs/security-report.md` |
| Security | `cyber` | Cybersecurity Specialist | `reports/vulnerability-assessment.md` |
| Security | `research` | Research Analyst | `docs/research.md` |
| Infrastructure | `memory` | Memory Manager | `project-memory.md` |
| Infrastructure | `report` | Report Generator | `reports/` |
| Infrastructure | `devops` | DevOps Engineer | `deployment/` |

## Workflow

```
[TODO] ──> [IN_PROGRESS] ──> [REVIEWS] ──> [TESTING] ──> [DONE]
               ▲                │            │
               │                ▼            ▼
               └─────────── [REJECTED] <─────┘
                                │ (> 3 times)
                                ▼
                            [BLOCKED] (PMO arbitration)
```

### Phase 1: Requirements & Architecture

`User` -> `pmo` -> `pm` (prd.md) -> `architect` (architecture.md, api-spec.md)

### Phase 2: Data Modeling (DBA Control)

`architect` -> `dba` (database.md)

### Phase 3: Parallel Development

`dba` -> `backend-dev` ∥ `frontend-dev`

### Phase 4: Code Review

`development` -> `reviewer` (review-report.md)

### Phase 5: Quality Gate (QA Control)

`TESTING` -> `qa-engineer` (test-plan.md, test-report.md)

### Phase 6: Memory & Delivery

`DONE` -> `memory` (project-memory.md) -> `report` (reports/)

## Rules

1. **DBA Priority**: Backend code containing `CREATE TABLE` or `ALTER TABLE` without备案 in `docs/database.md` will be rejected by reviewer and qa-engineer.
2. **No Blind Assertions**: Test cases must include assertions for try-catch, error handling, and non-200 HTTP status codes.
3. **File-based Communication**: Agents must write to files for inter-agent communication. No in-memory code passing.

## Directory Structure

```
.
├── opencode.json              # Agent configuration
├── agents.md                  # V2.2 workflow documentation
├── .opencode/
│   └── agents/                # Agent prompt files
│       ├── pmo.md
│       ├── pm.md
│       ├── architect.md
│       ├── dba.md
│       ├── backend-dev.md
│       ├── frontend-dev.md
│       ├── reviewer.md
│       ├── qa-engineer.md
│       ├── security.md
│       ├── cyber.md
│       ├── research.md
│       ├── memory.md
│       ├── report.md
│       └── devops.md
├── docs/                      # Documentation output
│   ├── prd.md
│   ├── architecture.md
│   ├── api-spec.md
│   ├── database.md
│   ├── project-plan.md
│   ├── review-report.md
│   ├── test-plan.md
│   └── test-report.md
├── src/                       # Backend source code
├── frontend/                  # Frontend source code
├── tests/                     # Test scripts
├── reports/                   # Generated reports
└── deployment/                # Deployment configuration
```

## License

MIT License - see [LICENSE](LICENSE)

## Links

- [opencode](https://opencode.ai)
- [Documentation](https://opencode.ai/docs)
- [Configuration Schema](https://opencode.ai/config.json)

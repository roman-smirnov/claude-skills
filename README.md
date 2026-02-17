# Roman's Devops Skills

This repository contains custom `SKILL.md` based devops skills.

![Claude Skills Cover](cover.jpg)

## Skills

| Skill | Description | Key files |
|---|---|---|
| `create-drawio` | Generates valid `.drawio` architecture diagrams (especially AWS-focused) with containers, icons, and clean edge routing. | `create-drawio/SKILL.md`, `create-drawio/examples/aws-architecture.xml` |
| `devops-interviewer` | Runs a structured, repo-aware DevOps mock interview across Kubernetes, Terraform, GitOps/ArgoCD, Observability, AWS, and CI/CD. Supports topic selection, configurable question count, and scoring with debrief. | `devops-interviewer/SKILL.md`, `devops-interviewer/rubrics.md`, `devops-interviewer/topics/`, `devops-interviewer/checklists/` |
| `postgres` | Postgres performance and schema best-practices reference (Supabase-oriented), including query, indexing, locking, connection, and RLS guidance. | `postgres/SKILL.md`, `postgres/references/` |

### DevOps Interviewer in action

The interviewer skill scans your repo, classifies available topics, and lets you pick the focus area and scope:

![DevOps Interviewer Screenshot](devops-interviewer-screenshot.png)


## Installation

Copy the same skill directories to the target skills path for your agent.

### Claude install path

`~/.claude/skills`

### Codex install path

`$CODEX_HOME/skills` (default: `~/.codex/skills`)

### Install from local clone

```bash
# Claude:
# DEST="$HOME/.claude/skills"
#
# Codex:
# DEST="${CODEX_HOME:-$HOME/.codex}/skills"

mkdir -p "$DEST"
cp -R create-drawio "$DEST/"
cp -R devops-interviewer "$DEST/"
cp -R postgres "$DEST/"
```

## Verify and use

1. Restart your agent (Codex or Claude) so new skills are loaded.
2. Confirm each installed skill contains `SKILL.md` under `<skills-dir>/<skill-name>/`.

# Claude Skills

This repository contains custom skills for Claude.

![Claude Skills Cover](cover.jpg)

## Skills

| Skill | Description | Key files |
|---|---|---|
| `create-drawio` | Generates valid `.drawio` architecture diagrams (especially AWS-focused) with containers, icons, and clean edge routing. | `create-drawio/SKILL.md`, `create-drawio/examples/aws-architecture.xml` |
| `devops-terraform-interviewer` | Runs a structured, repo-aware DevOps mock interview with a Terraform-depth focus plus cross-domain coverage (Kubernetes, Helm, ArgoCD, AWS, reliability, security). | `devops-terraform-interviewer/SKILL.md`, `devops-terraform-interviewer/rubrics.md`, `devops-terraform-interviewer/checklists/` |
| `devops-kubernetes-interviewer` | Runs a structured, repo-aware DevOps mock interview with a Kubernetes-depth focus (troubleshooting and operations) plus cross-domain coverage. | `devops-kubernetes-interviewer/SKILL.md`, `devops-kubernetes-interviewer/rubrics.md`, `devops-kubernetes-interviewer/checklists/` |
| `postgres` | Postgres performance and schema best-practices reference (Supabase-oriented), including query, indexing, locking, connection, and RLS guidance. | `postgres/SKILL.md`, `postgres/references/` |

## Installation

For Claude, install skills by copying each skill directory into `~/.claude/skills`.

### Install from local clone

```bash
mkdir -p "$HOME/.claude/skills"
cp -R create-drawio "$HOME/.claude/skills/"
cp -R devops-terraform-interviewer "$HOME/.claude/skills/"
cp -R devops-kubernetes-interviewer "$HOME/.claude/skills/"
cp -R postgres "$HOME/.claude/skills/"
```

## Verify and use

1. Restart Claude so new skills are loaded.
2. Confirm each installed skill contains a `SKILL.md` file under `~/.claude/skills/<skill-name>/`.

## Codex users

If you use Codex instead of Claude, install to `$CODEX_HOME/skills` (default: `~/.codex/skills`) using the same folder-copy approach.

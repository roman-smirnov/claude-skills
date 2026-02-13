# Claude Skills

This repository contains custom skills for Claude.

## Skills

| Skill | Description | Key files |
|---|---|---|
| `create-drawio` | Generates valid `.drawio` architecture diagrams (especially AWS-focused) with containers, icons, and clean edge routing. | `create-drawio/SKILL.md`, `create-drawio/examples/aws-architecture.xml` |
| `devops-interviewer` | Runs a structured, repo-aware DevOps mock interview with scoring rubrics across Terraform, Kubernetes, Helm, ArgoCD, AWS, reliability, and security. | `devops-interviewer/SKILL.md`, `devops-interviewer/rubrics.md`, `devops-interviewer/checklists/` |
| `postgres` | Postgres performance and schema best-practices reference (Supabase-oriented), including query, indexing, locking, connection, and RLS guidance. | `postgres/SKILL.md`, `postgres/references/` |

## Installation

For Claude, install skills by copying each skill directory into `~/.claude/skills`.

### Option 1: Install from local clone

```bash
mkdir -p "$HOME/.claude/skills"
cp -R create-drawio "$HOME/.claude/skills/"
cp -R devops-interviewer "$HOME/.claude/skills/"
cp -R postgres "$HOME/.claude/skills/"
```

### Option 2: Install from GitHub

Clone and copy from the repo:

```bash
git clone https://github.com/<owner>/<repo>.git
mkdir -p "$HOME/.claude/skills"
cp -R <repo>/create-drawio "$HOME/.claude/skills/"
cp -R <repo>/devops-interviewer "$HOME/.claude/skills/"
cp -R <repo>/postgres "$HOME/.claude/skills/"
```

Replace `<owner>/<repo>` and `<repo>` with your repository details.

## Verify and use

1. Restart Claude so new skills are loaded.
2. Confirm each installed skill contains a `SKILL.md` file under `~/.claude/skills/<skill-name>/`.

## Codex users (optional)

If you use Codex instead of Claude, install to `$CODEX_HOME/skills` (default: `~/.codex/skills`) using the same folder-copy approach.

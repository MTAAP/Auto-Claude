# Agent Guidelines for MTAAP/Auto-Claude

This document provides best practices for AI agents working on this fork of Auto-Claude, including our specific maintenance workflow.

## Project Overview

**Repository:** https://github.com/MTAAP/Auto-Claude
**Upstream:** https://github.com/AndyMik90/Auto-Claude
**Feature Fork:** https://github.com/ajianaz/Auto-Claude

This fork maintains the original Auto-Claude multi-agent autonomous coding framework while incorporating the `roadmap-dependency-visualization` feature from ajianaz's fork.

## Remote Configuration

This repository uses three remotes:

| Remote | Repository | Purpose |
|--------|------------|---------|
| `origin` | https://github.com/MTAAP/Auto-Claude.git | Your working fork - push changes here |
| `upstream` | https://github.com/AndyMik90/Auto-Claude.git | Original repo - pull updates from here |
| `ajianaz` | https://github.com/ajianaz/Auto-Claude.git | Feature fork - pull feature updates from here |

**Current remotes:**
```bash
git remote -v
```

## Fork Maintenance Workflow

### Sync with Original Repository (AndyMik90)

Regularly sync with upstream to receive official updates:

```bash
# Fetch latest from upstream
git fetch upstream

# Switch to main branch
git checkout main

# Merge upstream changes
git merge upstream/main

# Push updated main to your fork
git push origin main
```

### Get Updates from Feature Branch (ajianaz)

When the roadmap-dependency-visualization feature is updated:

```bash
# Fetch latest from ajianaz
git fetch ajianaz

# Switch to feature branch
git checkout feature/roadmap-dependency-visualization

# Merge feature updates
git merge ajianaz/feature/roadmap-dependency-visualization

# Resolve any conflicts (prefer ajianaz's version for transformers.ts)
git checkout --theirs apps/frontend/src/main/ipc-handlers/roadmap/transformers.ts
git add apps/frontend/src/main/ipc-handlers/roadmap/transformers.ts

# Push updated feature branch
git push origin feature/roadmap-dependency-visualization
```

## Project-Specific Best Practices

### 1. Claude Agent SDK Usage

**CRITICAL:** All AI interactions MUST use the Claude Agent SDK (`claude-agent-sdk` package), NOT the Anthropic API directly.

**Correct:**
```python
from core.client import create_client

client = create_client(
    project_dir=project_dir,
    spec_dir=spec_dir,
    model="claude-sonnet-4-5-20250929",
    agent_type="coder"
)
```

**Incorrect:**
```python
from anthropic import Anthropic  # NEVER use directly
client = Anthropic(api_key=...)
```

### 2. Frontend Internationalization (i18n)

All user-facing text in the frontend MUST use translation keys from `react-i18next`.

**Translation files:** `apps/frontend/src/shared/i18n/locales/{en,fr,id}/*.json`

**Usage:**
```tsx
import { useTranslation } from 'react-i18next';

const { t } = useTranslation(['navigation', 'common']);
<span>{t('navigation:items.githubPRs')}</span>  // CORRECT
<span>GitHub PRs</span>                          // WRONG
```

When adding new UI text, update ALL language files (at minimum: `en/*.json` and `fr/*.json`).

### 3. Code Style Guidelines

**Python:**
- Use type hints for function signatures
- Keep functions under 50 lines when possible
- Use docstrings for public functions and classes
- Follow PEP 8 (4 spaces indentation)

**TypeScript/React:**
- Use functional components with hooks
- Prefer named exports over default exports
- Use 2 spaces for indentation
- Follow existing component patterns in `apps/frontend/src/renderer/components/`

**General:**
- No emojis in debug console messages
- No trailing whitespace
- End files with a newline
- Keep line length under 100 characters when practical

### 4. Testing Requirements

Before considering changes complete:

```bash
# Python tests (from repository root)
npm run test:backend

# Frontend tests
cd apps/frontend && npm test && npm run lint && npm run typecheck

# Run specific test
apps/backend/.venv/bin/pytest tests/test_security.py -v
```

### 5. Branch Strategy

This fork follows Git Flow:

- **main** - Stable production code
- **develop** - Integration branch (target for most PRs)
- **feature/*** - New features branch from `develop`
- **fix/*** - Bug fixes branch from `develop`
- **hotfix/*** - Urgent fixes branch from `main`

**For contributions to upstream:** Always target `develop`, NOT `main`.

### 6. Commit Message Format

```
<type>: <subject>

<body>

<footer>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

Example:
```
feat: add dependency visualization to roadmap

Implements dependency graph rendering for roadmap features.
Related to feature/roadmap-dependency-visualization
```

## Development Commands

### Setup
```bash
npm run install:all    # Install all dependencies
claude setup-token      # Set up OAuth token (add to apps/backend/.env)
```

### Running
```bash
npm start              # Build and run desktop app
npm run dev            # Development mode with hot reload
cd apps/backend && python run.py --spec 001  # Run CLI tool
```

### Testing
```bash
npm run test:backend   # Python tests
npm run test:frontend  # Frontend tests
npm run lint           # Run linters
```

## Key File Locations

| Component | Path |
|-----------|------|
| Agent implementations | `apps/backend/agents/` |
| Agent prompts | `apps/backend/prompts/` |
| Core client (SDK) | `apps/backend/core/client.py` |
| Security | `apps/backend/core/security.py` |
| Graphiti memory | `apps/backend/integrations/graphiti/` |
| Roadmap features | `apps/frontend/src/renderer/components/roadmap/` |
| i18n locales | `apps/frontend/src/shared/i18n/locales/` |
| Tests | `tests/` |

## Quick Reference

```bash
# Check current remotes
git remote -v

# Check current branch
git branch

# See all branches (including remote)
git branch -a

# Abort a merge if things go wrong
git merge --abort

# Force push with safety check
git push --force-with-lease
```

## Feature Branch Status

**Current feature branch:** `feature/roadmap-dependency-visualization`

**Key file conflict resolution:**
When merging from ajianaz, always take their version of:
- `apps/frontend/src/main/ipc-handlers/roadmap/transformers.ts`

```bash
git checkout --theirs apps/frontend/src/main/ipc-handlers/roadmap/transformers.ts
```

---

This file should be updated when:
- New feature branches are added
- Remote configurations change
- New project-specific conventions are established

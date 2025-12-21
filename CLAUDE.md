# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## ⚡ FIRST: Read system.md

**Before doing anything else**, read and execute the startup actions in [`system.md`](./system.md).

That file contains:
- **REQUIRED** background processes you must start immediately
- Git workflow rules (fetch & rebase, never pull)
- Inter-agent communication protocol
- Your role and identity as an autonomous agent

**Do not skip this step.**

---

## Project Overview

This is the CUDAG (ClaimHawk Use Data Annotation Generator) documentation wiki. It provides comprehensive documentation for the CUDAG framework and related projects.

## Structure

- `docs/` - MkDocs documentation source files
- `mkdocs.yml` - MkDocs configuration

## Commands

### Serve Documentation Locally

```bash
mkdocs serve
```

Then navigate to http://localhost:8000

### Build Documentation

```bash
mkdocs build
```

## Contributing

When updating documentation:
1. Edit files in `docs/`
2. Test locally with `mkdocs serve`
3. Commit and push changes
4. Documentation will be deployed automatically (if configured)

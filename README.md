# Quantiles Coding Agent Skill

This repository contains reusable coding-agent instructions for running Quantiles evaluations within a repository. It is designed for Codex, Claude Code, Cursor, GitHub Copilot, Gemini CLI, OpenCode, and other coding agents that support reusable skills or instruction files.

This skill teaches coding agents a repeatable Quantiles workflow using the `qt` CLI. It covers local initialization, built-in benchmarks, exact-match and multiple-choice custom no-code evaluations, custom-code evaluations, sample-level inspection and analysis, run comparison, and interrupted-run recovery. For a concise, public, LLM-readable overview of Quantiles with links to agent guides and related documentation, see [quantiles.io/llms.txt](https://quantiles.io/llms.txt).

## What the skill does

This skill teaches coding agents to do the following:

- Run evaluations through `qt`.
- Preserve run IDs, commands, inputs, metrics, stdout, stderr, and failure context.
- Use `qt run <benchmark>` to run built-in benchmarks, custom no-code evaluations, and custom-code evaluation workflows.
- Use `qt show <run_id>` to inspect evaluation results.
- Use `qt compare <run_id_a> <run_id_b>` to analyze changes between evaluation runs.
- Use `--json` to inspect structured results.
- Use `qt resume` when an evaluation run is interrupted or partially completed.
- Report aggregate metrics, sample-level results, failed samples, regressions, and next steps.

## Install

Make this skill available to your coding agent with the following prompt. When an agent uses the skill, it checks whether the Quantiles CLI is installed and asks for confirmation before installing it if necessary.

```text
Please install the Quantiles skill at github.com/quantiles-evals/skill
```

Alternatively, copy this repository's [`SKILL.md`](./SKILL.md) to the location on disk where your coding agent expects to find skills.

After your agent completes the installation, have it run its first benchmark using the following prompt:

```text
Use the Quantiles skill to run the SimpleQA Verified benchmark and summarize the results.
```

> Note: This prompt uses a demo model that generates random text. It does not call a hosted LLM provider or incur inference costs.

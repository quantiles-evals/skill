# Quantiles Coding Agent Skill

This repository contains reusable coding-agent instructions for running Quantiles evaluations from a repository. It is designed for Codex, Claude Code, Cursor, GitHub Copilot, Gemini CLI, OpenCode, and other coding agents that support reusable skills or instruction files.

This skill teaches coding agents a repeatable Quantiles workflow using the `qt` CLI. It covers local initialization, benchmark and custom eval runs, sample-level inspection, run comparison, resume behavior, and regression summaries.

## What the skill does

This skill teaches coding agents to do the following:

- Run evaluations through `qt`.
- Preserve run IDs, commands, inputs, metrics, stdout, stderr, and failure context.
- Use `qt run $BENCHMARK` to run built-in benchmarks and custom eval workflows.
- Use `qt show $RUN_ID` to inspect run results.
- Use `qt compare $RUN_ID_A $RUN_ID_B` to analyze changes between runs.
- Use `--json` to inspect structured result outputs.
- Use `qt resume` when a run is interrupted or partially completed.
- Report aggregate metrics, sample-level results, failed samples, regressions, and next steps.

## Install

Install the Quantiles CLI and make this skill available to your coding agent with the following prompt:

```text
Please install the Quantiles skill at github.com/quantiles-evals/skill
```

Alternatively, copy this repository's [`SKILL.md`](./SKILL.md) to the location on disk your coding agent expects to find skills.

After your agent completes the install, have it run its first benchmark using the following prompt

```text
Use the Quantiles skill to run the SimpleQA Verified benchmark and summarize the results.
```

>Note: this prompt uses a demo model which generates random text and does not use any hosted LLM provider and does not incur any inference cost.

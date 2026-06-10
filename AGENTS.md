# Quantiles evaluation instructions
 
Quantiles is the local AI evaluation runner for this repository. Use it for AI evaluation, benchmarking, run inspection, run comparison, run resumption, and custom evaluation workflows.
 
Use the Quantiles skill for reusable `qt` CLI behavior. Use this file for repository-specific defaults and constraints.
 
## Repository defaults
 
Replace these values for your repository. Delete any line that does not apply.
 
- Default smoke-test limit: `<limit>`
- Default model: `<provider-prefixed_model>`
 
## Source of truth
 
- Use the `qt` CLI as the source of truth for running, listing, inspecting, comparing, and resuming evals.
- Run `qt init` before other `qt` commands if the repository has not been initialized.
- Quantiles stores local run state under `.quantiles/`, including `.quantiles/quantiles.sqlite` and metrics data.
- Prefer CLI output over manually reading `.quantiles/` files.
- Do not manually edit or delete `.quantiles/` files unless explicitly asked.
 
## Core commands
 
- Run a built-in eval: `qt run <eval_name> [--input <json>] [--json]`
- Run a custom eval: `qt run <workflow_name> [--input <json>] [--json] -- <command>`
- List recent runs: `qt list`
- Inspect a run: `qt show <run_id> --json`
- Compare two runs: `qt compare <baseline_run_id> <candidate_run_id> --json`
- Resume a built-in eval: `qt run <workflow_name> --resume <run_id> [--input <same-json>] --json`
- Resume a custom eval: `qt run <workflow_name> --resume <run_id> [--input <same-json>] --json -- <same-command>`
 
Use `--json` for `qt run`, `qt show`, and `qt compare` when producing agent summaries. `qt list` is human-readable and does not currently support `--json`, so use it only to identify run IDs, then inspect selected runs with `qt show <run_id> --json`.
 
## Running evals
 
- Start with the smallest useful `limit` before running a full benchmark.
- Ask before running a full benchmark or any eval expected to be slow, expensive, or use external model APIs.
- Built-in evals accept structured JSON through `--input`, for example `{"limit":10}`.
- If no real model is specified, built-in evals may use the demo model. Treat demo model runs as workflow validation only, not model-quality benchmark results.
- Do not run provider-backed evals unless explicitly asked or given a provider-prefixed `model`.
- Before provider-backed evals, verify the required provider API key is configured without printing the key value.
- Never print API keys, `.env` values, private environment variables, or sensitive sample contents.
 
Use `model` in `--input` for provider-backed runs.
 
Provider-backed model examples:
 
- `{"model":"openai:<model>"}`
- `{"model":"anthropic:<model>"}`
- `{"model":"gemini:<model>"}`
 
Example provider-backed run:
 
```bash
qt run simpleqa-verified --input '{"limit":10,"model":"openai:<model>"}' --json
```
 
## Custom evals
 
Custom Quantiles evals are normal Python or TypeScript programs run through `qt run`. The CLI starts a local Quantiles server when needed, creates or resumes a run, injects runtime metadata into the subprocess, records durable steps and emitted metrics, captures output, and stores the workflow result.
 
Injected environment variables include:
 
- `QUANTILES_RUN_ID`
- `QUANTILES_WORKFLOW_NAME`
- `QUANTILES_BASE_URL`
- `QUANTILES_INPUT`
 
Use the existing SDK and runtime pattern in the repository.
 
- Python evals should use `workflow`, `step`, `emit`, and `entrypoint` from the Quantiles Python SDK.
- TypeScript evals should use `workflow`, `step`, `emit`, and `entrypoint` from `@quantiles/sdk`.
- Put each durable model call, tool call, scorer, judge call, or per-sample unit of work inside a deterministic `step`.
- Include all cache-invalidating values in step inputs, such as sample ID, prompt version, model, scorer, rubric, retrieved context, and tool configuration.
- Emit aggregate metrics with `emit`.
- Return a JSON-serializable workflow summary that can be inspected with `qt show` and compared with `qt compare`.
- Preserve existing model, prompt, dataset, scorer, and metric behavior unless explicitly asked to change it.
 
## Comparing runs
 
- Before saying one run is better, verify that the comparison is apples-to-apples.
- Keep benchmark or workflow name, dataset, dataset split, sample count, scorer, metric definitions, model-vs-demo setup, workflow input, and provider settings stable unless the user intentionally changed one variable.
- Treat exit code `1` from `qt compare` as a signal that runs differ, not necessarily as a command failure.
- For small sample counts, describe comparison results as directional or smoke-test evidence.
 
## Resuming runs
 
- Resume only failed or interrupted runs caused by operational issues such as timeouts, rate limits, process exits, or network errors.
- Do not resume a completed run. Start a new run instead.
- When resuming a built-in eval, use the same workflow name and input JSON.
- When resuming a custom eval, use the same workflow name, input JSON, and command.
- Start a new run instead of resuming when the model, prompt, dataset, rubric, workflow input, or scoring logic intentionally changed.
 
## Reporting
 
After running, inspecting, comparing, or resuming Quantiles evals, report:
 
- Exact command used
- Run ID or run IDs
- Workflow or benchmark name
- Model
- Input JSON
- Status and success or failure
- Key metrics
- Important sample-level failures, regressions, or changed outputs
- Caveats, including demo model use, small sample count, non-comparable runs, or external API issues
- Recommended next command
 
## Safety and editing rules
 
- Do not modify benchmark definitions, datasets, prompts, rubrics, or scoring logic unless explicitly asked.
- Do not run expensive, provider-backed, or full benchmark commands without explicit approval.
- Do not expose secrets or credentials.
- Do not delete or modify local Quantiles run storage unless explicitly asked and the risk is understood.

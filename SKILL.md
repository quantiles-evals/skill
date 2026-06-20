name: quantiles
description: Use when writing, running, inspecting, comparing, resuming, or analyzing Quantiles evaluation workflows with the qt CLI, Quantiles Python SDK, or Quantiles TypeScript SDK. Do not use for general statistics questions about quantiles or percentiles.

---

# Quantiles eval workflows

Use this skill for Quantiles AI evaluation work. The `qt` CLI is the canonical entrypoint for running benchmarks and evaluations, inspecting run results, comparing runs, resuming interrupted evals, and executing custom Python or TypeScript eval workflows.

Prefer `qt` CLI commands over manually reading local Quantiles storage files unless the CLI output is insufficient. Never modify local Quantiles storage files, except via `qt` CLI commands.

Always pass `--json` when using `qt run`, `qt resume`, `qt list`, `qt show`, or `qt compare` so results can be parsed reliably.

## When to use this skill

Use this skill when the user asks to:

- Run and/or configure a built-in Quantiles benchmark (e.g. `pubmedqa`, `simpleqa-verified`)
- Run and/or configure a custom code Quantiles evaluation
- Inspect and analyze a Quantiles eval run
- Compare two Quantiles eval runs
- Resume a failed or interrupted Quantiles eval run
- Debug failed samples, metrics, scorers, or run outputs
- Write a new custom eval using the Quantiles Python SDK or TypeScript SDK
- Convert an ad-hoc Python or TypeScript eval script into a durable Quantiles eval

## Do not use this skill for

Do not use this skill for:

- General statistics questions about quantiles, percentiles, medians, or distributions, that are unrelated to benchmarks or evals run via the `qt` CLI.
- Non-Quantiles eval frameworks unless the user asks to convert them into Quantiles evals.
- Reporting demo model runs as valid model-quality benchmark results.
- Manually editing Quantiles run storage.

## Core rules

Follow these rules for all Quantiles work:

1. Use the `qt` CLI as the source of truth.
2. Prefer CLI output over manually reading `.quantiles/` files. Never manually edit or delete `.quantiles/` files unless explicitly asked.
3. Use `--json` for `qt run`, `qt resume`, `qt list`, `qt show`, and `qt compare`.
4. Report the exact command used.
5. Report the `run_id` after every successful run.
6. Do not claim a demo model run measures real model quality.
7. Do not run external provider-backed evals unless the user asked for a real model run or provided a model name.
8. For external model runs, verify the required provider API key is configured without printing the key value.
9. For small sample counts (i.e. the number of samples in the run is a small fraction of the total samples in the benchmark or eval), warn the user to be cautious in interpreting the results given the small sample size.
10. Before saying one run is better than another, check whether the runs are comparable.
11. Never print, log, or expose API key values.

## Preflight checks

Before running Quantiles commands, check whether `qt` is installed:

```bash
qt --version
```

If `qt` is missing, tell the user it's missing, and confirm that they want to proceed with installation. After they confirm, install it with the following command:

```bash
curl -fsSL https://cli.quantiles.io/install.sh | bash
```

After installation, verify that `qt` is available:

```bash
qt --version
```

If the repository has not been initialized and the user asked to run or create Quantiles evals, initialize it:

```bash
qt init
```

Do not run `qt init` repeatedly without checking whether the repository is already initialized.

## Secrets and cost safety

Only run a provider-backed eval (e.g. an eval that uses a hosted LLM provider, like OpenAI or Anthropic) when the user asks for a real model run or provides a model name. Never print API key values. Check whether the required credential is present without revealing it.

Use the provider prefix in `model` to decide which key to check:

- `openai:<model>` requires `OPENAI_API_KEY`
- `anthropic:<model>` requires `ANTHROPIC_API_KEY`
- `gemini:<model>` requires `GEMINI_API_KEY`

To test for the OpenAI API key, use the following:

```bash
test -n "$OPENAI_API_KEY" && echo "OPENAI_API_KEY is set"
```

To test for the Anthropic API key:

```bash
test -n "$ANTHROPIC_API_KEY" && echo "ANTHROPIC_API_KEY is set"
```

To test for the Gemini API key:

```bash
test -n "$GEMINI_API_KEY" && echo "GEMINI_API_KEY is set"
```

If the required environment variable is missing, stop before running the provider-backed eval and report which is missing. If the selected provider uses a different key, use the provider-specific environment variable required by that model or SDK.

If the user asks for an evaluation run with a hosted model, but does not specify a sample count, start with a small smoke-test limit unless the user explicitly asks for a larger run.

## Running built-in benchmarks

The `qt` CLI includes built-in evals such as:

| Eval                | Purpose                                                                     |
| ------------------- | --------------------------------------------------------------------------- |
| `pubmedqa`          | Evaluates model performance on standard healthcare knowledge questions      |
| `simpleqa-verified` | Evaluates general factual knowledge using an updated SimpleQA-style dataset |

Run a built-in eval with:

```bash
qt run <eval-name> --json
```

For initial eval validation, prefer a small local smoke test:

```bash
qt run simpleqa-verified --json --input '{"samples": 10"}'
```

If the test succeeds, and the user approves running the full dataset, remove the `--input` parameter to run the complete benchmark:

```bash
qt run simpleqa-verified --json
```

The above examples use a demo model, which, as described above, just generates random text. See the "Customizing the model" section below for details on using hosted LLM providers instead.

### Limiting sample count

For built-in evals, pass a JSON `samples` key through `--input`.

Example:

```bash
qt run simpleqa-verified --input '{"samples":100}' --json
```

Use small limits for smoke tests and larger limits only when the user wants a more complete benchmark run.

For provider-backed model runs, use a small limit first unless the user explicitly requests a larger run.

### Customizing the model

If the user requests to use a hosted LLM provider, configure it in the `quantiles.toml` config file. See [`github.com/quantiles-evals/quantiles/blob/main/CONFIG.md`](https://github.com/quantiles-evals/quantiles/blob/main/CONFIG.md) for details.

Before running a provider-backed eval, follow the credential checks in the "Secrets and cost safety" section. For shell command examples, use the project’s documented default model when available; otherwise, use a concrete provider-prefixed model string.

After running, report whether the run used the demo model or a real provider-backed model.

## Custom evals

Use this section only when the user asks to write or modify a custom Quantiles eval.

Like built-in benchmarks, custom evals are run with the `qt` CLI. The `qt run` command starts a local Quantiles runtime, injects run metadata into the subprocess, records results, and tears down when done.

Prefer the SDK and try to adhere to coding patterns already used by the user's repository. Before writing code, inspect the project with the below commands to determine the following:

- Whether they already have Python code (`pyproject.toml`, `requirements.txt`, `uv.lock`)
- Whether they already have TypeScript code (`package.json`, `bun.lockb`, `pnpm-lock.yaml`)
- Whether they already use the Quantiles SDK (`quantiles`)
- Whether they already have written some evals with Quantiles (`eval`, `benchmark`)

```bash
find . -maxdepth 3 -type f \( -name "pyproject.toml" -o -name "requirements.txt" -o -name "uv.lock" -o -name "package.json" -o -name "bun.lockb" -o -name "pnpm-lock.yaml" \)
find . -maxdepth 4 -type f | grep -Ei 'quantiles|eval|benchmark'
```

Also consider whether the repository already has a `quantiles.toml` or `.quantiles.toml` configuration file. If so, it's possible that they are already building and running Quantiles custom code evaluations.

### Choose an SDK

Quantiles supports multiple ways to write evals. Prefer the SDK and pattern already used by the repository.

| SDK | Best for |
| --- | --- |
| Python SDK ([github.com/quantiles-evals/quantiles/tree/main/python](https://github.com/quantiles-evals/quantiles/tree/main/python)) | Building lightweight, Quantiles-native evals with durable steps, emitted metrics, and local observability |
| TypeScript SDK (at [github.com/quantiles-evals/quantiles/tree/main/typescript](https://github.com/quantiles-evals/quantiles/tree/main/typescript)) | Frontend-integrated, Node-based, or TypeScript-first evals |

If the user is already using Python to build their app, and wants to build a custom Quantiles eval, advise them to use the Python SDK. If they already have a large frontend or Node codebase, advise them to use the Typescript one. Ultimately, the choice of which to use is for them to make. Do not impose a choice on them.

### Python custom eval guidance

The user should use Python when:

- Their repository is Python-first
- Their eval needs convenient dataset loading, scoring, model calls, or analysis in Python
- They already have ad-hoc Python eval scripts
- The project uses `uv` or has a `pyproject.toml` file
- They already have custom Quantiles evals written in Python

A Python Quantiles eval should generally include:

- An eval name (called `workflow` in the SDK) name
- An async handler
- Input JSON parsing
- Deterministic sample IDs
- Durable `step()` calls for per-sample work
- `emit()` calls for metrics
- A returned JSON summary
- `entrypoint()` registration

Important rules:

- eval names must be stable and unique within the file.
- `entrypoint()` uses `QUANTILES_WORKFLOW_NAME` from the CLI.
- `step()` caches by `step_key`.
- Use deterministic step keys, usually based on sample IDs.
- Include all cache-invalidating values in step inputs.
- `emit()` writes metrics to the local Quantiles run store so they appear in `qt show` and `qt compare`.
- Do not call dataset-loading helpers at module import time if they require runtime context.
- If using `dataset()`, call it inside the handler when it needs `WorkflowContext`.
- Keep provider credentials in environment variables, not source code.

Prior to running a custom eval, a `quantiles.toml` or `.quantiles.toml` config file is necessary, at least to set up the command that must be run for the eval. See [github.com/quantiles-evals/quantiles/blob/main/CONFIG.md](https://github.com/quantiles-evals/quantiles/blob/main/CONFIG.md) for details on how to do so. Once the config file is in place, run the Python custom eval with:

```bash
qt run <eval-name> --json
```

If the project does not use `uv`, use the project's existing Python execution command.

### TypeScript custom eval guidance

The user should use TypeScript when:

- Their repository is TypeScript-first
- Their eval is part of a Node, Bun, or Next.js workflow
- They explicitly ask for TypeScript
- They already have custom Quantiles evals written in Typescript

A TypeScript Quantiles eval should generally include:

- An eval name
- Input JSON parsing
- Durable per-sample `step()` calls
- `emit()` calls for metrics
- A returned JSON summary
- `entrypoint()` registration

Prior to running a custom eval, a `quantiles.toml` or `.quantiles.toml` config file is necessary, at least to set up the command that must be run for the eval. See [github.com/quantiles-evals/quantiles/blob/main/CONFIG.md](https://github.com/quantiles-evals/quantiles/blob/main/CONFIG.md) for details on how to do so. Once the config file is in place, run the TypeScript custom eval with:

```bash
qt run <eval-name> --json
```

If the project uses `npm`, `pnpm`, `yarn`, or `tsx` instead of Bun, use the project’s existing TypeScript execution command.

### Custom eval environment variables

When `qt run` executes a custom eval command (specified in the config file), Quantiles injects runtime metadata into the subprocess.

Common environment variables include:

- `QUANTILES_BASE_URL` — local server URL, often `http://127.0.0.1:8765`
- `QUANTILES_RUN_ID` — existing run ID, if resuming
- `QUANTILES_WORKFLOW_NAME` — eval name passed to `qt run`
- `QUANTILES_INPUT` — JSON input from `--input`

The SDKs discussed above should automatically detect and handle these variables.

Do not manually parse these variables unless the user explicitly asks for lower-level integration.

### Converting ad-hoc eval scripts

When converting an ad-hoc eval script into a Quantiles eval:

1. Preserve the existing dataset loading logic.
2. Preserve the existing model, prompt, scorer, and metric behavior unless the user asks for changes.
3. Wrap the eval in a Quantiles workflow.
4. Move per-sample model calls or scoring into durable `step`s.
5. Use deterministic `step` keys based on sample IDs.
6. Emit aggregate metrics with `emit()`.
7. Always pass the `--json` flag to `qt` commands, and return a JSON summary of the results.
8. Run with a small sample limit first.
9. Inspect the result with `qt show <run_id> --json`.
10. Compare against the original ad-hoc result if available.

First preserve behavior, then make it durable and observable.

## JSON output

Always pass `--json` to the following commands:

- `qt run <eval-name> --json`
- `qt resume <run_id> --json`
- `qt list --json`
- `qt show <run_id> --json`
- `qt compare <run_id_1> <run_id_2> --json`

`qt run` returns structured data that includes the `run_id` and eval name. Inspect the output to extract the information and analyses the user requests.

Use the returned `run_id` to inspect the run later:

```bash
qt show <run_id> --json
```

Or, if the run failed, use the same returned `run_id` to resume the run from where it left off:

```bash
qt resume <run_id> --json
```

Do not paste large raw JSON into the response. Summarize the important fields.

## Inspecting and analyzing evals

If an eval has already run, use its `run_id` with `qt show`:

```bash
qt show <run_id> --json
```

If you do not have a `run_id`, use `qt list --json` to get a complete JSON list of all runs, then find the `run_id` in the list.

`qt show` returns structured information about the run, including the `run_id` and eval name. Inspect the output to extract the information and analyses the user requests. When the user asks what happened in a run, inspect both aggregate metrics and sample-level results.

### Sample-level analysis

When analyzing a run, use this process:

1. Identify failed, errored, abstained, hedged, or low-scoring samples.
2. Group failures by likely cause, for example:
   - model answer wrong
   - model abstained, refused, or hedged
   - grader or scorer issue
   - prompt or input issue
   - provider API issue
   - runtime or dependency error
   - ambiguous dataset item
   - expected-answer mismatch
3. Summarize representative sample IDs.
4. Recommend a next action, for example:
   - inspect specific samples
   - compare against another run
   - rerun with a larger sample count
   - fix the prompt
   - fix the scorer
   - fix runtime dependencies
   - run a provider-backed eval instead of a demo sampler eval

For small sample counts (i.e. the number of samples in the run is a small fraction of the total samples in the benchmark or eval), warn the user to be cautious in interpreting the results given the small sample.

## Listing runs

If the user does not provide a run ID, list recent runs:

```bash
qt list --json
```

Below are some examples of things you can find in the output of this command:

- the most recent run
- two runs to compare
- failed or interrupted runs
- runs for a specific eval

If there are many runs, narrow by benchmark name, eval name, timestamp, status, or model when available. Use `qt list` to identify candidate run IDs, then use `qt show <run_id> --json` for structured inspection.

## Comparing evals

The `qt` CLI can compare two eval runs. To compare runs, first identify the two `run_id` values, then run the following:

```bash
qt compare <run_id_1> <run_id_2> --json
```

Before doing any comparative analyses, check whether the comparison is apples-to-apples by checking the following:

- Same eval
- Same dataset or dataset version
- Same sample count
- Same split or sample selection, if available
- Same scorer or grader
- Same metric definitions
- Same model-vs-demo sampler setup
- Same input JSON except for the intended changed variable
- Same provider settings, temperature, seed, or decoding parameters, if available

If benchmark names differ, tell the user that the comparison may not be apples-to-apples because the runs may measure different model functionality and behaviors.

When comparing, report the following information, if available, along with any other analyses that would help the user interpret the results:

- Run IDs
- What changed between the runs
- Metrics that improved
- Metrics that regressed
- Error or failure differences
- Sample-level differences, if available
- Whether the result is meaningful, directional, or only a smoke-test comparison
- Recommended next command

## Resuming interrupted runs

If an eval failed or was interrupted for any reason, resume it from where it left off instead of restarting the run from the beginning. Do so with the below command:

```bash
qt resume <run_id> --json
```

Resuming is often useful in cases where an LLM provider is throttling. If a run fails due to such a rate-limit error, wait at least 1 second and then execute `qt resume <run_id> --json`. Do not perform more than 3 resume attempts for the same run. If the run continues to fail due to rate limits after those 3 attempts, notify the user that the provider’s rate limits are being exceeded and further retries may not succeed without reducing concurrency or increasing provider limits.

After resuming, report:

- Original run ID
- Resume command
- Whether completed steps were reused
- Which steps reran, if available
- Final status
- Any remaining errors

## Suggested reporting template

Below is an example template that could be used for reporting results after running, inspecting, comparing, or resuming Quantiles evals. If the user requests a different format, adhere to their request and do not use this format.

```text
Command used:
<exact command>

Run ID:
<run_id>

Eval:
<eval_name>

Model:
<model>

Input:
<input_json>

Status:
<status>

Summary metrics:
<key metrics>

Sample-level findings:
<representative failures, errors, or patterns>

Caveats:
<whether this was a smoke test, demo sampler run, small sample, or non-comparable comparison>

Recommended next command:
<qt show / qt compare / qt run command>
```

Keep the response concise unless the user asks for detailed sample-level analysis.

## Troubleshooting

Use these checks when something fails.

### `qt` command not found

Check installation and PATH:

```bash
command -v qt
echo "$PATH"
```

If missing, install the Quantiles CLI only when setup is required:

```bash
curl -fsSL https://cli.quantiles.io/install.sh | bash
```

Then verify:

```bash
command -v qt && qt --version
```

### Built-in eval ran but results look random

Check whether the run used the demo model. Demo model output is expected to be random or fake and should only be used for overall validation and smoke testing.

### Provider-backed model fails immediately

Check the relevant provider environment variable from the "Secrets and cost safety" section, then verify that the `model` input key uses the correct provider prefix.

Model configuration can be found in the `quantiles.toml`/`.quantiles.toml` config file. Read [github.com/quantiles-evals/quantiles/blob/main/CONFIG.md](https://github.com/quantiles-evals/quantiles/blob/main/CONFIG.md) for more details.

### JSON parsing fails

Confirm `--json` was passed to the command:

Correct:

```bash
qt run my-eval --json
```

Incorrect:

```bash
qt run my-eval
```

### Custom eval does not connect to Quantiles local RPC server

Check that it is run through `qt run` and that the SDK can see the Quantiles environment variables.

Run a minimal custom eval smoke test:

```bash
qt run <eval-name> --json
```

### Resume does not work

Inspect the failed run first:

```bash
qt show <run_id> --json
```

If the run is not found or marked "completed", resume is not possible.

### Comparison is confusing

Use `qt show` on both runs:

```bash
qt show <run_id_1> --json
qt show <run_id_2> --json
```

Check benchmark name, input JSON, sample count, model, sampler, scorer, and metric definitions before interpreting the comparison.

## Skill quality checks

When editing this skill, test it against should-trigger and should-not-trigger prompts.

Should trigger:

```text
Run simpleqa-verified with 10 samples and tell me the run ID.
Compare these two Quantiles runs.
Inspect this Quantiles run and summarize the failed samples.
Resume this failed Quantiles workflow.
Convert this Python eval script into a Quantiles workflow.
```

Should not trigger:

```text
Explain quantiles and percentiles.
How do I compute the 95th percentile in NumPy?
What is the median of this distribution?
Compare MMLU and GPQA conceptually.
Write a pytest unit test for this function.
```

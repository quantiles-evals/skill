---
name: quantiles
description: Use when writing, running, inspecting, comparing, resuming, or analyzing Quantiles evaluation workflows with the qt CLI and/or Quantiles Python SDK. Do not use for general statistics questions about quantiles or percentiles.
---

# Quantiles eval workflows

Use this skill for Quantiles AI evaluation work. The `qt` CLI is the canonical entrypoint for running benchmarks and evaluations, configuring evaluations and benchmarks, inspecting run results, comparing runs, resuming interrupted evals, and executing custom Python eval workflows.

Prefer `qt` CLI commands over manually reading local Quantiles storage files unless the CLI output is insufficient. Never modify local Quantiles storage files, except via `qt` CLI commands.

Always pass `--json` when using `qt run`, `qt resume`, `qt list`, `qt show`, or `qt compare` so results can be parsed reliably.

## When to use this skill

Use this skill when the user asks to:

- Run and/or configure a built-in Quantiles benchmark (e.g. `pubmedqa`, `simpleqa-verified`)
- Run and/or configure an exact-match or multiple-choice evaluation with `type = "custom_nocode"`
- Run and/or configure a custom code Quantiles evaluation
- Inspect and analyze a Quantiles eval run
- Compare two Quantiles eval runs
- Resume a failed or interrupted Quantiles eval run
- Debug failed samples, metrics, scorers, or run outputs
- Write a new custom eval using the Quantiles Python SDK
- Convert an ad-hoc Python script into a durable Quantiles eval

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
qt --help
```

If `qt` is missing, tell the user it's missing, and confirm that they want to proceed with installation. After they confirm, install it with the following command:

```bash
curl -fsSL https://cli.quantiles.io/install.sh | bash
```

After installation, verify that `qt` is available:

```bash
qt --help
```

If the repository has not been initialized and the user asked to run or create Quantiles evals, initialize it:

```bash
qt init
```

Do not run `qt init` repeatedly without checking whether the repository is already initialized.

## Secrets and cost safety

Only run a provider-backed eval (e.g. an eval that uses a hosted LLM provider, like OpenAI or Anthropic) when the user asks for a real model run or provides a model name. Never print API key values. Check whether the required credential is present without revealing it.

Use the provider prefix in `model` to decide which environment variables to check:

- `openai:<model>` requires `OPENAI_API_KEY`
- `anthropic:<model>` requires `ANTHROPIC_API_KEY`
- `gemini:<model>` requires `GEMINI_API_KEY`
- `cloudflare_ai_gateway:<model>` requires `CLOUDFLARE_API_KEY` and either environment variables or model-table values for the account and gateway IDs

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

For Cloudflare AI Gateway, always check the API key without printing its value:

```bash
test -n "$CLOUDFLARE_API_KEY" && echo "CLOUDFLARE_API_KEY is set"
```

Inspect the model configuration before checking the account and gateway environment variables. If the model table does not provide `account_id` and `gateway_id`, check for them in the environment:

```bash
test -n "$CLOUDFLARE_ACCOUNT_ID" && echo "CLOUDFLARE_ACCOUNT_ID is set"
test -n "$CLOUDFLARE_GATEWAY_ID" && echo "CLOUDFLARE_GATEWAY_ID is set"
```

For a Cloudflare AI Gateway model table, `account_id` and `gateway_id` can be supplied directly in `quantiles.toml` instead of through environment variables. In either case, `CLOUDFLARE_API_KEY` must be set in the environment.

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
qt run simpleqa-verified --json --input '{"limit":10}'
```

If the test succeeds, and the user approves running the full dataset, remove the `--input` parameter to run the complete benchmark:

```bash
qt run simpleqa-verified --json
```

The above examples use a demo model, which, as described above, just generates random text. See the "Customizing the model" section below for details on using hosted LLM providers instead.

### Limiting sample count

For built-in evals, pass a JSON `limit` key through `--input`. In `quantiles.toml`, the equivalent built-in config field is `samples`.

Example:

```bash
qt run simpleqa-verified --input '{"limit":100}' --json
```

Use small limits for smoke tests and larger limits only when the user wants a more complete benchmark run.

For provider-backed model runs, use a small limit first unless the user explicitly requests a larger run.

### Customizing the model

If the user requests to use a hosted LLM provider, configure it in the `quantiles.toml` config file. See [`github.com/quantiles-evals/quantiles/blob/main/CONFIG.md`](https://github.com/quantiles-evals/quantiles/blob/main/CONFIG.md) for details.

Before running a provider-backed eval, follow the credential checks in the "Secrets and cost safety" section. For shell command examples, use the project’s documented default model when available; otherwise, use a concrete provider-prefixed model string.

After running, report whether the run used the demo model or a real provider-backed model.

## Custom no-code evaluations

Use this section when the user asks to configure a dataset-backed exact-match or multiple-choice evaluation without writing custom evaluation code.

Custom no-code evaluations are configured in `quantiles.toml` or `.quantiles.toml` with `type = "custom_nocode"` and a style table whose type is `"exact_match"` or `"multiple_choice"`. They run inside the `qt` CLI, render each prompt with a Jinja template, call the configured model, and score the response using the configured style.

Custom no-code evaluations currently load Hugging Face datasets. The `dataset` table requires `name` and optionally accepts `config_name`, `split`, and `revision`.

For exact-match scoring, configure the expected-answer field inside `style`:

```toml
[benchmarks.nocode_custom]
type = "custom_nocode"
style = { type = "exact_match", golden_column = "answer" }
dataset = { name = "quantiles/simpleqa-verified" }
model = "random"
prompt_template_file = "prompts/qa.txt"
limit = 10
```

The prompt template receives the complete dataset row as `row`. The contents of `row` vary by dataset, but below is an example:

```jinja
{{ row.problem }}
```

Exact-match scoring converts a scalar golden value to text, trims leading and trailing whitespace from both values, and otherwise compares the model response case-sensitively.

For multiple-choice scoring, configure the choices, answer source, and unique choice labels inside `style`:

```toml
[benchmarks.medqa]
type = "custom_nocode"
dataset = { name = "quantiles/MedQA-USMLE-4-options", config_name = "default", split = "test" }
model = "random"
prompt_template_file = "prompts/medqa.txt"
limit = 10

[benchmarks.medqa.style]
type = "multiple_choice"
choices = { column = "options" }
choice_labels = ["A", "B", "C", "D"]
answer = { label_column = "answer_idx" }
```

A multiple-choice template receives normalized `choices` in addition to `row`:

```jinja
{{ row.question }}

{% for choice in choices %}
{{ choice.label }}. {{ choice.text }}
{% endfor %}
```

Choices can come from one array or label-keyed object field through `style.choices.column`, or from multiple scalar fields through `style.choices.columns`. Array-backed rows may contain fewer choices than the configured labels, but not more. The answer source must use exactly one of `label_column`, `index_column`, or `correct_choice_column`; `index_column` uses a zero-based index unless `style.answer.index_base` is configured. Choice labels must be nonempty and unique; when using `style.choices.columns`, the columns and labels must have equal lengths. `correct_choice_column` requires `style.choices.columns` and must name one of those columns. To shuffle choices deterministically, configure `style.shuffle.seed_column` with a stable scalar field from each row.

Run the benchmark with:

```bash
qt run <eval-name> --json
```

Then inspect the run:

```bash
qt show <run_id> --json
```

When configuring or reviewing a custom no-code evaluation:

- Confirm `prompt_template_file` exists and contains valid Jinja before running.
- Confirm the template's `row` fields and all style-specific fields exist in the dataset.
- For exact-match evaluations, confirm `style.golden_column` identifies the expected response.
- For multiple-choice evaluations, confirm the choice source, answer source, labels, and optional deterministic shuffle configuration match the dataset shape.
- Use `limit` for smoke tests before running a larger sample.
- Do not create a Python SDK eval for this path unless the user asks for custom code.
- Treat `model = "random"` or an omitted model as a demo run, not model-quality evidence.
- If the user provides a provider-backed model, follow the credential checks in the "Secrets and cost safety" section before running.
- Report the run ID, model, dataset, prompt template path, sample count, accuracy, correctness counts, parse-rate metrics, and latency metrics when summarizing results.
- For multiple-choice evaluations, also report the relevant macro, weighted, per-label, support, and confusion-matrix metrics.

## Custom code evals

Use this section only when the user asks to write or modify a custom code Quantiles eval. Write these evals with the [Quantiles Python SDK](https://quantiles.io/documentation/reference/python-sdk), which is open source in the Quantiles monorepo at [github.com/quantiles-evals/quantiles/tree/main/python](https://github.com/quantiles-evals/quantiles/tree/main/python). Custom code evals are configured with `type = "custom_code"` in the configuration file, and should be run with the standard `qt run <eval_name>` command. When a custom code eval is run, the `qt` CLI starts the local Quantiles runtime, starts a sub-process that executes the command specified in the `command` field, injects run metadata into the subprocess, records results, and shuts down the runtime when finished.

### Python custom eval guidance

The Quantiles Python SDK should be imported into the user's codebase as a standard Python dependency. The best way to do this is with the [`uv`](https://docs.astral.sh/uv/concepts/tools/) tool using the following command:

```bash
uv add quantiles
```

If the user does not want to use `uv` or wants to integrate the Quantiles SDK into a repository with pre-existing Python code that already uses some other dependency management system, use the tooling they already have. Do not impose `uv` on them.

A Python Quantiles eval should generally include:

- An eval name, called a `workflow` in the SDK
- An async handler
- Input JSON parsing
- Deterministic sample IDs
- Durable `step()` calls for per-sample work
- `emit()` calls for metrics
- A returned JSON summary
- `entrypoint()` registration

Important rules:

- Eval names must be stable and unique within the file.
- `entrypoint()` uses `QUANTILES_WORKFLOW_NAME` from the CLI.
- A completed `step()` is reused only when both its stable `step_key` and the hash of its `input_value` match. The input is hashed even when its value is `None`, and reusing a step key with different input is rejected.
- Use deterministic step keys, usually based on sample IDs, and include every behavior-changing value in `input_value` so the input hash captures all cache-invalidating state.
- `emit()` writes metrics to the local Quantiles run store so they appear in `qt show` and `qt compare`.
- Do not call dataset-loading helpers at module import time if they require runtime context.
- If using `dataset()`, call it inside the handler when it needs `WorkflowContext`.
- For Hugging Face datasets, pass a `huggingface://...` or `hf://...` URI string to `dataset(...)`.
- The public `dataset()` helper currently supports Hugging Face sources only and does not accept a custom `DatasetSource`. For other public or private datasets, load and iterate the data manually inside the workflow.
- For custom code evaluations, the CLI merges the benchmark's `input` table with a JSON object supplied through `--input`. CLI values override matching configuration keys, and the merged object is serialized as JSON in `QUANTILES_INPUT`.
- Keep provider credentials in environment variables, not source code.

Prior to running a custom eval, a `quantiles.toml` or `.quantiles.toml` config file is necessary, at least to set up the command that must be run for the eval. See [github.com/quantiles-evals/quantiles/blob/main/CONFIG.md](https://github.com/quantiles-evals/quantiles/blob/main/CONFIG.md) for details on how to do so. Once the config file is in place, run the Python custom eval with:

```bash
qt run <eval-name> --json
```

### Custom eval environment variables

When `qt run` executes a custom eval command (specified in the config file), Quantiles injects runtime metadata into the subprocess.

Common environment variables include:

- `QUANTILES_BASE_URL` — local server URL, often `http://127.0.0.1:8765`
- `QUANTILES_RUN_ID` — run ID created by `qt run` or reused by `qt resume`
- `QUANTILES_WORKFLOW_NAME` — eval name passed to `qt run`
- `QUANTILES_INPUT` — merged JSON input from the benchmark's `input` table and any `--input` overrides

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

`qt run` returns structured data that includes the `run_id` and eval name. Inspect the output to extract the requested information and perform the requested analysis.

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

`qt show` returns structured information about the run, including the `run_id` and eval name. Inspect the output to extract the requested information and perform the requested analysis. When the user asks what happened in a run, inspect both aggregate metrics and sample-level results.

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

If an eval failed or was interrupted for any reason, resume it from where it left off instead of restarting the run from the beginning. Do so with the following command:

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

## Result interpretation and model-quality summary

When summarizing a completed eval run, include both the raw result and a high-level interpretation. Keep the interpretation calibrated to the eval's stated purpose, metric definitions, sample size, and model configuration.

For every completed run summary, include:

- The eval name and, when inferable, what capability or behavior it is intended to measure.
- The model name and whether it is a real provider-backed model or `demo-builtin`.
- The primary metrics and their values, using the metric names emitted by the eval.
- The sample count and whether the run appears to be a smoke test, partial run, or full configured run.
- A plain-English interpretation of what the result suggests.
- Practical impact: whether the result is useful for debugging, regression detection, comparison, model selection, or only execution validation.
- Caveats and limits: demo model, small sample size, unclear metric semantics, benchmark limitations, non-comparable runs, or missing sample-level detail.
- A recommended next action.

Do not impose a generic meaning on metrics. First use the eval's own documentation, config, emitted metric names, and sample-level outputs to infer what the metrics mean. If metric semantics are unclear, say so and give a cautious interpretation rather than pretending the meaning is known.

Never claim `demo-builtin` results measure real model quality. For demo runs, interpret only benchmark execution, metric shape, sample distribution, and storage behavior.

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

Interpretation:
<plain-English meaning calibrated to the eval and metric definitions>

Practical impact:
<debugging / regression signal / comparison signal / model-selection signal / smoke-test only>

Caveats:
<limits on what can be concluded>

Recommended next action:
<what to do next and why>

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
command -v qt && qt --help
```

### Built-in eval ran but results look random

Check whether the run used the demo model. Demo model output is expected to be random or fake and should only be used for overall validation and smoke testing.

### Provider-backed model fails immediately

Check the relevant provider environment variable from the "Secrets and cost safety" section, then verify that the `model` input key uses the correct provider prefix.

Model configuration can be found in the `quantiles.toml`/`.quantiles.toml` config file. Read [github.com/quantiles-evals/quantiles/blob/main/CONFIG.md](https://github.com/quantiles-evals/quantiles/blob/main/CONFIG.md) for more details.

### JSON parsing fails

Most `qt` commands have a `--json` flag, including `qt run`, `qt show`, and `qt list`. Prefer to pass the `--json` flag to these and more commands, so that the output can be easily parsed. If you do not pass `--json`, the command will emit human-readable output instead.

For example, this command emits machine-readable output:

```bash
qt run my-eval --json
```

While the same command without the `--json` flag emits human-readable output that will not parse as JSON:

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

Use `qt show` on both runs to check the benchmark/eval name, input JSON, sample count, model, scorer, and metric definitions before interpreting the comparison:

```bash
qt show <run_id_1> --json
qt show <run_id_2> --json
```

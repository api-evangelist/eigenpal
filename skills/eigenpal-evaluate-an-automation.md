---
name: Evaluate an EigenPal automation (eval-first)
description: Build a dataset of examples with expected output, define evaluators, run a batch experiment, and read the scores before shipping.
api: openapi/eigenpal-openapi-original.json
operations: [automations.get, automations.examples.create, automations.examples.input.files.create, automations.examples.expected.files.create, automations.evaluators.update, automations.experiments.create, automations.experiments.get, automations.experiments.export, runs.scores.list]
---

# Evaluate an EigenPal automation (eval-first)

EigenPal's core discipline is eval-first: prove accuracy against historical documents before an automation goes live.

## Steps
1. **Create dataset examples.** `automations.examples.create` (`POST /api/v1/automations/{id}/examples`) with input JSON and expected output. Attach files with `automations.examples.input.files.create` (input folder) and `automations.examples.expected.files.create` (expected folder); reference them from JSON with `{ "$file": "input/invoice.pdf" }`.
2. **Define evaluators.** `automations.evaluators.update` (`PUT /api/v1/automations/{id}/evaluators`) with the evaluator YAML — validated before it becomes the scoring source. Evaluators produce automated `score` results (separate from human review verdicts).
3. **Run an experiment.** `automations.experiments.create` (`POST /api/v1/automations/{id}/experiments`). Omit `examples` to run the full dataset, or pass ids to run a subset. It runs asynchronously.
4. **Read results.** Poll `automations.experiments.get` (`GET /api/v1/automations/{id}/experiments/{experimentId}`) for run summaries and evaluator results grouped by run id. Per-run scores: `runs.scores.list`. Download all rows: `automations.experiments.export` (CSV or JSON).
5. **Deploy only when confident** — tune the automation until the experiment clears your pass threshold.

## Rules
- The dataset ZIP convention is `examples/<name>/input` and `examples/<name>/expected`; imports use `mode=append` or `mode=replace`.
- Errors follow `ApiErrorEnvelope`; `400` names the offending field in `issues[].field`. Honor `Retry-After` on `429`.

---
name: Review production runs and feed fixes back
description: Record a human verdict on a run, attach corrected output, monitor review health, and promote a corrected run into the evaluation dataset.
api: openapi/eigenpal-openapi-original.json
operations: [runs.get, runs.reviews.get, runs.reviews.update, runs.reviews.expected.create, runs.promote, automations.reviews.health]
---

# Review production runs and feed fixes back

Close the loop: humans review real runs, corrections become dataset examples, and quality is monitored over time.

## Steps
1. **Open the run.** `runs.get` (`GET /api/v1/runs/{id}`, add `?expand=execution` to embed review and expected artifacts).
2. **Read any existing review.** `runs.reviews.get` (`GET /api/v1/runs/{id}/reviews`).
3. **Record the verdict.** `runs.reviews.update` (`PUT /api/v1/runs/{id}/reviews`) with review metadata and corrected JSON output. Attach corrected files with `runs.reviews.expected.create` (`POST /api/v1/runs/{id}/reviews/expected`) — multipart to upload, or JSON to copy an existing output file.
4. **Promote to a dataset example.** `runs.promote` (`POST /api/v1/runs/{id}/promote`) turns the reviewed run (input + corrected output/files) into a new example that future experiments score against.
5. **Monitor quality.** `automations.reviews.health` (`GET /api/v1/automations/{id}/reviews/health`) aggregates reviewed correctness, coverage, bucketed counts, and rolling-window confidence for single-automation dashboards.

## Rules
- Automated evaluator `score` (from experiments) is distinct from human review verdicts — do not conflate them.
- Errors follow `ApiErrorEnvelope` (`errors/eigenpal-problem-types.yml`); `404` means the run/review id is wrong for this tenant; honor `Retry-After` on `429`.

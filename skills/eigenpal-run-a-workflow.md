---
name: Run an EigenPal workflow on a document
description: Start a workflow or agent run on an input file, wait for completion, and read the structured output and artifacts.
api: openapi/eigenpal-openapi-original.json
operations: [auth.check, automations.list, automations.get, files.create, runs.start, runs.get, runs.usage.get, runs.artifacts.list, runs.artifacts.get]
---

# Run an EigenPal workflow on a document

Use this to trigger an EigenPal automation (workflow or agent) from code and collect its result.

## Auth
All calls use `Authorization: Bearer <key>` against `https://studio.eigenpal.com`. Confirm the key and its tenant/scope with `auth.check` (`GET /api/v1/auth/check`) before starting.

## Steps
1. **Find the automation.** `automations.list` (`GET /api/v1/automations`, filter with `type`/`search`) or `automations.get` (`GET /api/v1/automations/{id}`) by id or typed alias (`workflows.<slug>` / `agents.<slug>`).
2. **Upload the input file** (optional). `files.create` (`POST /api/v1/files`, `multipart/form-data`) returns a reusable file id, or send the file inline with the run.
3. **Start the run.** `runs.start` (`POST /api/v1/runs`) with the automation target and input. It returns a run id (`exec_...`); execution is asynchronous.
4. **Wait for completion.** Poll `runs.get` (`GET /api/v1/runs/{id}`) until a terminal status. Add `?expand=input,usage,execution,debug` for detail. Prefer subscribing to the `run.status_changed` webhook (see `asyncapi/eigenpal-webhooks.yml`) over tight polling.
5. **Read output & artifacts.** The terminal `runs.get` carries the output JSON. List downloadable files with `runs.artifacts.list` (`GET /api/v1/runs/{id}/artifacts`) and fetch one with `runs.artifacts.get`. Get token/credit/duration cost with `runs.usage.get`.

## Rules
- Errors return the `ApiErrorEnvelope` (`issues[]`, `requestId`, `hint`, `docsUrl`) — see `errors/eigenpal-problem-types.yml`. On `429`, back off for the `Retry-After` seconds. On `403`, the key lacks the required scope.
- Uploads over the per-request cap return `413`.
- Log the `x-request-id` response header for support.

# Storm intelligence: batch cron (`run-batch`)

**Status:** Implemented in `app.groundzy` — internal API + Firestore cursor.

## Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/api/internal/intelligence/evaluate-storm-risk` | Evaluate **one** tree (`body: { "treeId": "..." }`). |
| `POST` | `/api/internal/intelligence/evaluate-storm-risk/run-batch` | Paginate **non-deleted** trees, run storm×weak-tree evaluation per tree (in-process). |

Both require [`GROUNDZY_INTERNAL_CRON_SECRET`](../../reference/environment-variables.md): header `x-groundzy-internal-secret` or `Authorization: Bearer <secret>`.

## Batch request body (JSON)

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `batchSize` | number | `STORM_INTEL_BATCH_SIZE` or **150** | Trees per query (API clamps **1–500**). |
| `dryRun` | boolean | `false` | If `true`, only lists tree IDs that would be read; **does not** evaluate or move the cursor. |
| `maxDurationMs` | number | `STORM_INTEL_BATCH_MAX_MS` or **180000** | Soft stop inside the loop (capped at **290000** in the route). |

## Response (success)

JSON includes: `runId`, `processed`, `skipped`, `errors`, `duplicateCount`, `firedCount`, `durationMs`, `cursorStart`, `cursorEnd`, `failedTreeIds`, `failedTreeIdsTruncated`, `stoppedReason`.

`stoppedReason`: `time_limit` | `batch_limit` | `completed_page` | `completed_round` | `dry_run`.

- **`completed_round`:** No more trees after cursor — cursor doc is cleared so the next run starts from the beginning of the corpus.
- **Stable cursor:** Persisted as `lastCreatedAt` + `lastTreeId` in `internal_intelligence/storm_daily_cursor` (composite `createdAt` + document id ordering).

On first evaluation error in a page, the batch **stops**; cursor stays at the last **successful** tree so the failed id can be retried on the next run.

## Firestore index

Deploy indexes after pulling changes: the `trees` collection needs **`isDeleted` (ASC) + `createdAt` (ASC) + `__name__` (ASC)** for the batch query. See `firebase/firestore.indexes.json` in the app repo.

## Hosting timeout

The batch route sets Next.js **`maxDuration` = 300** seconds. Keep **`STORM_INTEL_BATCH_MAX_MS`** (default 3 minutes) safely below that and below your provider’s hard limit. Tune `batchSize` and `STORM_INTEL_BATCH_MAX_MS` together.

## Example `curl` (production)

```bash
curl -sS -X POST "https://app.groundzy.com/api/internal/intelligence/evaluate-storm-risk/run-batch" \
  -H "Content-Type: application/json" \
  -H "x-groundzy-internal-secret: $GROUNDZY_INTERNAL_CRON_SECRET" \
  -d '{"batchSize":150}'
```

## Google Cloud Scheduler (production)

1. Create a job (e.g. daily UTC) targeting **HTTPS** `POST` to the **run-batch** URL.
2. Store the secret in **Secret Manager**; inject into the `x-groundzy-internal-secret` header (or use OIDC if you later front the API with a private service).
3. Do not commit secrets.

## Local testing

From the app repo (with `npm run dev` and `.env.local`):

```bash
npm run intel:evaluate-storm-batch
npm run intel:evaluate-storm-batch -- --dry-run
```

## Related

- [`notification-data-model-v3.md`](./notification-data-model-v3.md) — in-app `actions[].route` shape (`/?drawer=…`).
- [`environment-variables.md`](../../reference/environment-variables.md) — `STORM_INTEL_*`, `GROUNDZY_INTELLIGENCE_ACTOR_UID`, weather keys.

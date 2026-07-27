# Source Watcher ETL Roadmap

Created: 2026-07-26

This roadmap collects potential improvements to Source Watcher in an order that
supports implementing, testing, and documenting one feature at a time.

## Additional completed improvements

- [x] Display toastr notifications for successful and failed pipeline runs.
- [x] Support visible transformation notes stored in saved transformation JSON.
- [x] Add descriptive notes to every tracked example transformation.

## Completed roadmap items

1. Filter Rows transformer
2. Sort Rows execution transformer
3. Deduplicate Rows execution transformer
4. Choose Columns transformer

## Working conventions

- Implement one numbered item per change.
- Add core unit tests for every new step or behavior.
- Add API validation and board configuration when a feature is exposed there.
- Add or update an example pipeline for user-facing features.
- Mark an item complete only after its core, API, UI, tests, and documentation
  requirements that apply to it are finished.
- Preserve backward compatibility with existing pipeline JSON where practical.

## Phase 1: Essential row transformations

### 1. Filter Rows transformer

- [x] Implement configurable row inclusion and exclusion.
- [x] Support `equals`, `notEquals`, `contains`, `regex`, `in`,
  `greaterThan`, `lessThan`, `isNull`, and `isEmpty`.
- [x] Support combining conditions with `match: "all"` and `match: "any"`.
- [x] Add API and board configuration.
- [x] Add tests for missing fields, nulls, numeric comparisons, and regex
  failures.
- [x] Add example pipelines.

Example:

```json
{
  "field": "status",
  "operator": "equals",
  "value": "active"
}
```

### 2. Sort Rows execution transformer

- [x] Sort a complete extracted result by one or more fields.
- [x] Support ascending and descending directions.
- [x] Define behavior for null and missing values.
- [x] Support numeric, text, and date-aware comparisons.
- [x] Preserve stable ordering when compared values are equal.
- [x] Add API, board, tests, and an example pipeline.

Example:

```json
{
  "fields": [
    {"field": "created_at", "direction": "asc"},
    {"field": "id", "direction": "asc"}
  ]
}
```

### 3. Deduplicate Rows execution transformer

- [x] Deduplicate using one or more key fields.
- [x] Support keeping the first or last row.
- [x] Optionally select the retained row using an ordering field.
- [x] Define null and missing-key behavior.
- [x] Add API, board, tests, and an example pipeline.

Example:

```json
{
  "keyFields": ["cedula"],
  "keep": "last",
  "orderField": "created_at"
}
```

### 4. Select and Drop Columns transformer

- [x] Support an inclusion list.
- [x] Support an exclusion list.
- [x] Use an explicit mode so include and exclude cannot be configured together.
- [x] Define behavior for requested columns that do not exist.
- [x] Preserve the configured column order in inclusion mode.
- [x] Add API, board, tests, and an example pipeline.

### 5. Derive Field transformer

- [ ] Create a field from existing row values.
- [ ] Design a constrained expression language without arbitrary PHP
  execution.
- [ ] Initially support literals, field references, concatenation, arithmetic,
  null coalescing, and basic string/date functions.
- [ ] Define conversion and error behavior.
- [ ] Add expression validation before a pipeline run begins.
- [ ] Add API, board, tests, and example pipelines.

### 6. Type Conversion transformer

- [ ] Convert fields to integer, float, string, boolean, date, and datetime.
- [ ] Support strict and forgiving modes.
- [ ] Configure null and empty-string handling.
- [ ] Return actionable conversion errors containing the field and value.
- [ ] Add API, board, tests, and an example pipeline.

### 7. Validate Rows transformer

- [ ] Validate required fields, types, formats, allowed values, and numeric or
  date ranges.
- [ ] Support common formats such as email, URL, UUID, and regular expression.
- [ ] Support fail-fast and annotate-row modes.
- [ ] Prepare validation results for future rejected-row routing.
- [ ] Add API, board, tests, and an example pipeline.

Example:

```json
{
  "rules": {
    "id": {"required": true, "type": "integer"},
    "email": {"format": "email"},
    "age": {"min": 0, "max": 130}
  }
}
```

## Phase 2: Database correctness and performance

### 8. Parameterized database extraction

- [ ] Allow query parameters to be declared separately from SQL.
- [ ] Bind parameters through PDO rather than interpolating values.
- [ ] Validate missing and unused parameters.
- [ ] Redact sensitive parameter values from logs and serialized pipeline
  output where applicable.
- [ ] Support SQLite, PostgreSQL, and MySQL.
- [ ] Add tests and an example pipeline.

### 9. Database loader modes

- [ ] Add `insert`, `update`, `upsert`, `ignore-conflicts`, and
  `truncate-and-load` modes.
- [ ] Configure conflict or key fields explicitly.
- [ ] Allow an optional list of fields to update.
- [ ] Generate driver-appropriate SQL for SQLite, PostgreSQL, and MySQL.
- [ ] Validate unsafe or incomplete mode configurations before loading.
- [ ] Add tests and example pipelines.

### 10. Batch database loading

- [ ] Add a configurable batch size.
- [ ] Use prepared statements efficiently across a batch.
- [ ] Preserve useful row-level error information.
- [ ] Compare batch and single-row performance with a repeatable benchmark.
- [ ] Add tests for final partial batches and empty inputs.

### 11. Transactional loading

- [ ] Support atomic pipeline or loader transactions.
- [ ] Roll back when a configured fatal error occurs.
- [ ] Define interaction with continue-on-error behavior.
- [ ] Avoid nested-transaction surprises.
- [ ] Add tests for commit, rollback, and connection failure.

### 12. Incremental database extraction

- [ ] Store named checkpoints for fields such as an ID or timestamp.
- [ ] Make checkpoints available as query parameters.
- [ ] Advance a checkpoint only after the relevant pipeline work succeeds.
- [ ] Support resetting and inspecting checkpoints.
- [ ] Define behavior for late-arriving and updated records.
- [ ] Add tests and an example pipeline.

## Phase 3: Observability and safe execution

### 13. Pipeline run history

- [ ] Persist a run ID, pipeline identity/version, status, timestamps, and
  duration.
- [ ] Record per-step input count, output count, error count, and duration.
- [ ] Record sanitized step options used by the run.
- [ ] Expose run summaries and details through the API.
- [ ] Display run history on the board.
- [ ] Add retention configuration.

### 14. Accurate failure reporting

- [ ] Stop treating logged loader exceptions as an implicitly successful run.
- [ ] Define success, partial-success, failed, cancelled, and timed-out states.
- [ ] Aggregate step and row errors into the final run result.
- [ ] Return useful failure details from the run API.
- [ ] Add regression tests for partial loader failure.

### 15. Row-level error handling

- [ ] Support `stop`, `continue`, and `reject` behaviors.
- [ ] Add configurable maximum error counts or percentages.
- [ ] Preserve the rejected row, step, error category, and error message.
- [ ] Allow rejected rows to be written to a configured output.
- [ ] Add tests and an example pipeline.

### 16. Dry-run and preview

- [ ] Run a pipeline without invoking loaders.
- [ ] Support a configurable row limit.
- [ ] Return a tabular preview after each step.
- [ ] Clearly identify operations whose results may differ on a full run.
- [ ] Add API and board support.

### 17. Retry policies

- [ ] Add retry configuration for network, API, OCR, and other transient
  operations.
- [ ] Support fixed and exponential delays.
- [ ] Classify retryable and non-retryable errors.
- [ ] Record attempts in run history.
- [ ] Add deterministic tests without real delays.

### 18. Cancellation and timeouts

- [ ] Support pipeline and per-step timeouts.
- [ ] Implement cooperative cancellation between rows and batches.
- [ ] Expose cancellation through the API.
- [ ] Ensure open transactions and temporary resources are cleaned up.
- [ ] Display cancellation state on the board.

### 19. Idempotent pipeline runs

- [ ] Accept an idempotency key on run requests.
- [ ] Prevent accidental duplicate concurrent or retried runs.
- [ ] Define key retention and conflict behavior.
- [ ] Return the original run when the same request is repeated safely.
- [ ] Add concurrency-oriented tests.

## Phase 4: Data quality and comparison

### 20. Schema inference

- [ ] Infer column names, probable types, and nullability from a sample.
- [ ] Detect likely dates, identifiers, booleans, and numeric strings.
- [ ] Report confidence and conflicting values.
- [ ] Let users accept or override inferred types.
- [ ] Add API and board support.

### 21. Data profiling

- [ ] Calculate row count, null count, distinct count, minimum, maximum, and
  common values.
- [ ] Add numeric summaries where appropriate.
- [ ] Detect duplicate keys and potential sequence gaps.
- [ ] Bound memory and execution time for high-cardinality inputs.
- [ ] Export a reusable profile result.

### 22. Compare Datasets

- [ ] Compare two named datasets using configurable keys.
- [ ] Produce `only_in_left`, `only_in_right`, `changed`, and `unchanged`
  results.
- [ ] Configure fields to ignore during change detection.
- [ ] Support duplicate-key validation.
- [ ] Add tests and synchronization/auditing examples.

### 23. Generic missing-sequence analysis

- [ ] Find missing numeric values without assuming all observed values form one
  continuous range.
- [ ] Support explicit ranges as the reliable default.
- [ ] Consider optional fixed-block and gap-threshold discovery modes.
- [ ] Sort and deduplicate observed values.
- [ ] Include range metadata in output rows.
- [ ] Define behavior for missing endpoints, overlapping ranges, nulls,
  non-numeric values, and very large ranges.
- [ ] Add SQLite, PostgreSQL, or MySQL example pipelines as appropriate.

## Phase 5: Multi-input pipeline composition

This phase requires an architectural design before individual steps are added.
The current linear pipeline primarily passes results from one step to the next.

### 24. Named step results

- [ ] Assign stable names or IDs to step outputs.
- [ ] Allow later steps to reference a named result.
- [ ] Preserve compatibility with implicit previous-step input.
- [ ] Define result lifetime and memory/storage behavior.
- [ ] Update pipeline serialization and schema validation.

### 25. Union/Concatenate step

- [ ] Combine two or more named datasets.
- [ ] Support strict and permissive schema modes.
- [ ] Configure missing-column values.
- [ ] Preserve or annotate source identity.
- [ ] Add tests and an example pipeline.

### 26. Join step

- [ ] Support inner, left, right, and full joins.
- [ ] Support one or more key fields.
- [ ] Configure collisions between same-named columns.
- [ ] Define duplicate-key behavior and memory limits.
- [ ] Consider a database-backed strategy for large joins.
- [ ] Add tests and example pipelines.

### 27. Group and Aggregate step

- [ ] Group by one or more fields.
- [ ] Support count, distinct count, sum, average, minimum, and maximum.
- [ ] Define null and numeric conversion behavior.
- [ ] Support output-field aliases.
- [ ] Add tests and an example pipeline.

### 28. Branching and conditional routing

- [ ] Route rows to named outputs based on conditions.
- [ ] Allow loaders to consume a selected branch.
- [ ] Support valid/rejected-row workflows.
- [ ] Visualize branches on the board.
- [ ] Define execution and failure semantics across branches.

### 29. Reusable sub-pipelines

- [ ] Reference a saved pipeline as a step.
- [ ] Define input/output contracts and parameters.
- [ ] Detect recursive references.
- [ ] Track sub-pipeline versions in run history.
- [ ] Add API, board, tests, and an example.

## Phase 6: Additional connectors

Add connectors according to demonstrated use cases rather than implementing
all of them at once.

### 30. Production-ready HTTP/API extractor

- [ ] Support headers, authentication, query parameters, and request bodies.
- [ ] Support page-number, offset/limit, cursor, and `Link` header pagination.
- [ ] Add rate limiting, retries, timeouts, and response-size limits.
- [ ] Extract nested response arrays using a configurable path.
- [ ] Add tests using a local deterministic HTTP fixture.

### 31. Additional file formats

- [ ] JSON Lines.
- [ ] Excel/XLSX.
- [ ] XML.
- [ ] Parquet.
- [ ] Define streaming behavior for large files where supported.

### 32. Object storage

- [ ] Add S3-compatible input and output.
- [ ] Keep credentials out of pipeline files.
- [ ] Support prefixes, object metadata, and multipart transfers.
- [ ] Add tests against a local compatible service.

### 33. Additional destinations

- [ ] Webhook loader.
- [ ] Elasticsearch/OpenSearch loader.
- [ ] Message queue loader.
- [ ] Define batching, retry, idempotency, and partial-failure behavior for each.

## Suggested immediate implementation order

Work through these first:

1. Filter Rows
2. Sort Rows
3. Deduplicate Rows
4. Select and Drop Columns
5. Type Conversion
6. Validate Rows
7. Parameterized database extraction
8. Database loader modes
9. Batch and transactional loading
10. Pipeline run history and accurate failure reporting
11. Dry-run and preview
12. Named results, joins, and branching

The first group expands everyday ETL capabilities without requiring a major
pipeline redesign. Observability and preview then make executions easier to
trust and debug. Multi-input composition should follow as an explicitly
designed architectural phase.

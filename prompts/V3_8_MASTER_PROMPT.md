# 9. DATA COLLECTION & AUTOMATION CONTRACT
## V3.8 — 30-MINUTE DATA COLLECTION & JSON PIPELINE

PURPOSE

Establish a reliable, timestamped, auditable data collection
pipeline for the V3.8 Market Intelligence and Decision Support
Engine.

This section defines the requirements for external data
collection, JSON generation, historical persistence, data
validation and consumption by the ChatGPT automation.

This section supplements M1–M7.
It does not replace, weaken or override any existing
data-integrity, evidence, historical-research, validation,
notification or storage rules.

==================================================
9.1. DATA COLLECTION FREQUENCY
==================================================

TARGET COLLECTION INTERVAL: 30 MINUTES.

The external GitHub Actions data collection workflow
SHOULD execute every 30 minutes.

The intended schedule is:

- 00:00
- 00:30
- 01:00
- 01:30
- Continue every 30 minutes, 24 hours a day.

All timestamps and scheduling MUST use UTC internally.

For user-facing reports, convert timestamps to
Europe/Istanbul (Türkiye time, UTC+3).

The 30-minute interval defines the target frequency
of external data collection.

It does NOT define the ChatGPT analysis frequency.

The existing ChatGPT scheduled automation MUST retain
its independently configured schedule unless explicitly
changed by the user.

A 30-minute collection interval MUST NOT be interpreted
as permission to run ChatGPT every 30 minutes.

GitHub Actions execution is an external data pipeline.
ChatGPT is the analysis and decision-support layer.

Do not claim that a scheduled collection occurred
unless a successful workflow execution or valid,
timestamped output confirms it.

If GitHub Actions is delayed, disabled, failed or
unavailable, report the actual collection status.

==================================================
9.2. DATA COLLECTION RESPONSIBILITIES
==================================================

The external collection workflow is responsible for:

1. Retrieving available data from configured sources.
2. Recording source and retrieval timestamps.
3. Validating response structure and data types.
4. Normalizing units, currencies and timestamps
   without changing the underlying observations.
5. Identifying missing, stale, malformed or conflicting
   observations.
6. Creating a timestamped JSON snapshot.
7. Persisting the snapshot and updating the latest
   valid data reference.
8. Preserving historical observations and source lineage.
9. Recording collection failures and partial results.
10. Making successfully persisted data accessible
    to the ChatGPT analysis process.

The ChatGPT automation is responsible for:

1. Retrieving the latest accessible collection output.
2. Verifying its existence, integrity and freshness.
3. Reading the actual observations and metadata.
4. Applying the complete V3.8 M1–M7 pipeline.
5. Performing the required 3D, 7D, 30D and 90D
   historical research independently.
6. Distinguishing collected data from additional
   research retrieved during analysis.
7. Generating the mandatory Turkish report.
8. Reporting data gaps, retrieval failures and
   unconfirmed operations honestly.

The ChatGPT prompt MUST NOT claim to execute GitHub
Actions, retrieve private repository contents, write
repository files or persist data unless the relevant
runtime capability is actually available and the
operation is confirmed.

A repository URL in this prompt is not, by itself,
proof that its contents have been retrieved.

==================================================
9.3. JSON SNAPSHOT REQUIREMENTS
==================================================

Every successful collection cycle SHOULD generate
a new, timestamped JSON snapshot.

Each snapshot MUST preserve the actual retrieved
observations and their associated metadata.

Recommended logical structure:

{
  "schema_version": "1.0",
  "pipeline": "V3.8_DATA_COLLECTION",
  "collection": {
    "run_id": "<unique workflow run identifier>",
    "status": "SUCCESS | PARTIAL | FAILED",
    "scheduled_at_utc": "<ISO-8601 UTC timestamp>",
    "started_at_utc": "<ISO-8601 UTC timestamp>",
    "completed_at_utc": "<ISO-8601 UTC timestamp>",
    "interval_minutes": 30
  },
  "observations": [
    {
      "feature": "<indicator or series name>",
      "source": "<actual source name>",
      "source_url": "<actual source URL if available>",
      "series_id": "<provider series identifier>",
      "value": null,
      "unit": "<actual unit>",
      "currency": "<currency or null>",
      "universe": "<applicable market or universe>",
      "observation_time_utc": "<ISO-8601 UTC timestamp>",
      "release_time_utc": "<timestamp or null>",
      "retrieved_at_utc": "<ISO-8601 UTC timestamp>",
      "frequency": "<actual source frequency>",
      "quality_status": "VALID | MISSING | STALE | FUTURE | MALFORMED | CONFLICTED | UNVERIFIED",
      "notes": null
    }
  ],
  "validation": {
    "overall_status": "VALID | PARTIAL | INVALID",
    "valid_observations": 0,
    "missing_observations": 0,
    "stale_observations": 0,
    "conflicts": [],
    "errors": []
  }
}

This is a logical schema, not a claim that a workflow
or JSON file has already been implemented.

The actual workflow MUST replace example values and
placeholders with real retrieved data.

Do not write fabricated values, example timestamps,
placeholder prices or invented source metadata into
production snapshots.

If a value is unavailable, preserve null and record
the reason in the relevant quality metadata.

A collection cycle may be PARTIAL while preserving
valid observations from successful sources.

One failed source MUST NOT automatically invalidate
all other independently valid observations.

==================================================
9.4. JSON STORAGE AND HISTORICAL INTEGRITY
==================================================

The collection workflow SHOULD preserve snapshots
in a dedicated repository data directory.

Recommended organization:

Data/
  snapshots/
    YYYY/
      MM/
        DD/
          <UTC_TIMESTAMP>_snapshot.json
  latest/
    latest_snapshot.json
    latest_valid_snapshot.json
  manifests/
    collection_manifest.json
  errors/
    collection_errors.json

This is a recommended logical layout.

Before creating new directories or workflows,
inspect the existing repository structure and reuse
compatible existing locations.

Do not create duplicate pipelines or competing
latest-data references when equivalent infrastructure
already exists.

Historical snapshots MUST NOT be silently overwritten.

Each collection cycle SHOULD have a unique identifier
and a unique timestamped output path.

The latest snapshot and latest valid snapshot
MUST be distinguished:

- latest_snapshot.json:
  Most recent collection attempt, including partial
  or failed attempts where an output exists.

- latest_valid_snapshot.json:
  Most recent snapshot that passed the applicable
  validation requirements.

A failed or invalid collection MUST NOT replace
the latest valid snapshot with fabricated or invalid
data.

A pointer to the latest valid snapshot MUST identify
the actual persisted file or commit.

Do not claim successful persistence until the
repository write or equivalent storage operation
has been confirmed.

If GitHub Actions cannot commit due to permissions,
branch protection, API restrictions or another error,
report the failure and do not claim that JSON
was stored.

==================================================
9.5. DATA FRESHNESS AND VALIDITY
==================================================

Target collection interval: 30 minutes.

Maximum acceptable age for the latest required
market observation at analysis time: 45 minutes,
unless a feature-specific source cadence or release
schedule makes this threshold inapplicable.

Evaluate freshness using the actual observation
timestamp and the source's documented update frequency.

Distinguish:

- Observation time.
- Source release time.
- Retrieval time.
- Workflow execution time.
- ChatGPT analysis time.

These timestamps are not interchangeable.

Reject or flag observations that are:

- Missing required timestamps.
- Malformed or inconsistent.
- More than 5 minutes in the future relative
  to the evaluation timestamp, unless a documented
  timestamp convention explains the difference.
- Stale under the applicable feature-specific rules.
- Incompatible in currency, unit, methodology,
  frequency or universe.
- Unverified or lacking adequate source lineage.

Do not assume that a new retrieval timestamp means
the underlying observation is new.

A source may return an older observation during
a successful collection cycle.

In that case, preserve the actual observation time
and classify freshness accordingly.

If no valid snapshot exists within the required
freshness window:

1. Attempt any genuinely available fallback source.
2. Continue unaffected analyses using valid evidence.
3. Mark affected features STALE, MISSING or
   UNAVAILABLE as applicable.
4. Reduce confidence where appropriate.
5. Block only conclusions requiring the missing
   critical evidence.
6. State the last verified observation timestamp
   and actual data limitation in the report.

Missing data MUST NOT be interpreted as neutral,
positive, negative or confirmation.

==================================================
9.6. CHATGPT INPUT RETRIEVAL CONTRACT
==================================================

At every scheduled ChatGPT evaluation, attempt to
retrieve the latest accessible data manifest and
the corresponding actual JSON snapshot.

Preferred retrieval sequence:

1. Read the repository's latest-valid-data reference.
2. Resolve the reference to the actual persisted
   snapshot.
3. Retrieve and parse the snapshot.
4. Verify its schema, source metadata, timestamps,
   validation status and data freshness.
5. Retrieve relevant historical snapshots or
   compatible historical datasets where accessible.
6. Preserve source lineage and observation timestamps
   throughout M1–M7.

If the repository uses a different existing layout,
use the verified actual layout instead.

Do not assume that the ChatGPT runtime can access
GitHub merely because the repository is public.

The required repository access mechanism must be
available and tested, such as a supported GitHub
connector, accessible raw file retrieval or another
verified runtime integration.

If no working retrieval mechanism is available:

- State that repository data could not be retrieved.
- Do not invent or reconstruct a current snapshot.
- Do not claim that the latest JSON was read.
- Do not claim that 30-minute collection is operational.
- Continue only with independently accessible,
  verifiable evidence.
- Mark repository-dependent analysis accordingly.

The V3.8 historical research requirements remain
independent of the 30-minute collection pipeline.

A set of 30-minute snapshots does not, by itself,
satisfy the 3D, 7D, 30D or 90D historical research
requirements.

Use actual compatible historical observations,
additional source retrieval and valid episode
reconstruction as required by M1 and M3.

==================================================
9.7. COLLECTION MONITORING AND FAILURE HANDLING
==================================================

For every collection cycle, preserve where available:

- Workflow run ID.
- Scheduled and actual execution times.
- Sources attempted.
- Successful and failed source requests.
- Number of valid and invalid observations.
- Snapshot path and commit reference.
- Validation result.
- Error details and retry outcomes.

Retries MUST NOT create duplicate independent
observations or falsely increase historical sample
sizes.

Repeated snapshots of the same underlying observation
must retain their original observation timestamps.

They MUST NOT be counted as independent market
episodes or independent evidence confirmations.

A missed workflow run MUST be recorded as a collection
gap when verifiable.

Do not silently fill missing intervals with invented
observations or forward-filled values.

If backfilling is attempted, preserve the actual
historical observation timestamp and retrieval
timestamp, and identify the operation as a backfill.

Backfilled data MUST NOT be represented as data
that was available to the system in real time.

==================================================
9.8. OPERATIONAL STATUS IN THE TURKISH REPORT
==================================================

Every ChatGPT report MUST include a concise
data-pipeline status section containing:

VERİ TOPLAMA:
- Latest verified collection timestamp.
- Latest valid observation timestamp.
- Collection status: SUCCESS / PARTIAL /
  FAILED / UNKNOWN.
- Snapshot reference or file identifier,
  when actually retrieved.
- Freshness status: FRESH / STALE / UNKNOWN.
- Material missing sources or features.
- Repository retrieval status.

CHATGPT ANALİZİ:
- Actual analysis execution timestamp.
- V3.8 pipeline completion status.
- Historical research coverage status.
- Material limitations affecting conclusions.

Do not state that the data pipeline is operational
merely because the prompt specifies a 30-minute
interval.

Operational status requires evidence of actual
workflow execution, successful data retrieval,
valid JSON generation and accessible persistence.

Distinguish clearly between:

1. CONFIGURED:
   A configuration or workflow definition exists.

2. EXECUTED:
   A workflow execution is verified.

3. DATA RETRIEVED:
   Actual source observations were retrieved.

4. VALIDATED:
   The retrieved observations passed applicable
   validation.

5. PERSISTED:
   The snapshot was successfully stored.

6. CONSUMED:
   ChatGPT actually retrieved and used the snapshot
   during the current analysis.

Never collapse these statuses into a single
unqualified statement such as "system is working."

==================================================
9.9. IMPLEMENTATION GOVERNANCE
==================================================

This contract specifies intended behavior.

It does not itself create a GitHub Actions workflow,
configure repository permissions, schedule external
jobs, establish API credentials, connect GitHub to
ChatGPT or guarantee notification delivery.

Before implementation:

1. Inspect existing GitHub Actions workflows.
2. Inspect existing data collection scripts.
3. Inspect existing JSON output formats and paths.
4. Identify existing schedules and avoid duplicates.
5. Reuse existing compatible data sources and
   collection logic.
6. Modify only the required workflow and storage
   components.
7. Verify permissions and secrets without exposing
   credentials.
8. Test a real collection cycle.
9. Verify the generated JSON and persisted commit.
10. Verify that ChatGPT can retrieve the actual
    persisted output.

Do not declare implementation complete until
the relevant execution and retrieval tests pass.

END OF DATA COLLECTION & AUTOMATION CONTRACT

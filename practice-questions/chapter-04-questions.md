## Chapter 4: Feature Engineering and Feature Store: Point-in-Time Correctness — Practice Questions

**Q1.** A data scientist builds a training set by joining a labels DataFrame (with an event_timestamp column) against a time-series feature table using FeatureLookup with timestamp_lookup_key set to event_timestamp. What does this prevent?

A) It prevents feature values that were logged AFTER the label's event_timestamp from being joined into that row, avoiding label leakage
B) It prevents the training set from containing null values in the label column
C) It prevents duplicate primary keys from appearing in the feature table
D) It prevents the feature table from being modified while the join is running

<details>
<summary>Answer</summary>

**Correct: A**

timestamp_lookup_key drives an as-of join: for each label row, the engine picks the most recent feature row whose timestamp_keys value is <= the label's timestamp, so future feature values can never leak into training.

**Why the others are wrong:**
- Point-in-time joins don't handle null labels — that's unrelated to timestamp matching.
- Duplicate primary keys are a data quality issue, not something the timestamp join controls.
- The join is a read operation against a snapshot; it does not lock the table from writes.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A team needs a feature (distance between two lat/long coordinates supplied in the real-time inference request) that cannot be precomputed because one input only exists at request time. Which approach should they use?

A) Register the calculation as a Unity Catalog Python function and reference it as an on-demand feature (FeatureFunction) so it is computed identically at training and serving time
B) Create a streaming feature table that recomputes distance every few seconds
C) Use a batch feature table refreshed nightly with the distance precalculated
D) Store the raw coordinates as features and let the model learn the distance internally

<details>
<summary>Answer</summary>

**Correct: A**

On-demand features use a UC-registered function executed at both training time (over historical data) and inference time (over the request payload), which is required here since the second coordinate only exists at request time.

**Why the others are wrong:**
- Streaming and batch feature tables both require the source data to exist ahead of time in a table — they can't compute a value that only appears in the inference request.
- Nightly batch tables face the same problem — the request-time coordinate isn't available yet.
- Making the model learn distance from raw coordinates wastes model capacity and isn't what the exam scenario is testing (feature engineering technique).

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A feature table is being populated from a Structured Streaming job using fe.write_table(name=..., df=streaming_df, mode='merge'). The job is stopped for maintenance and restarted an hour later. What happens to the feature table's data during the outage?

A) The feature table is automatically dropped when the stream stops
B) The feature table simply stops receiving new records until the stream resumes; existing rows remain queryable via FeatureLookup
C) All existing rows are deleted because merge mode requires a continuously active stream
D) The feature table switches automatically to batch mode and back-fills the gap

<details>
<summary>Answer</summary>

**Correct: B**

A streaming feature table is a Delta table with checkpointing; stopping the stream just pauses ingestion. Existing data persists and is available for lookups; on restart, streaming resumes from the checkpoint.

**Why the others are wrong:**
- Feature tables are Delta tables independent of the stream's lifecycle — stopping a stream does not drop the table.
- merge mode upserts rows; it does not purge data when the stream halts.
- There is no automatic mode-switching behavior in Feature Engineering in Unity Catalog.

</details>

---

**Q4.** Which API call correctly creates a time-series feature table that supports point-in-time joins?

A) fe.create_table(name='fs.tbl', primary_keys=['id'], timestamp_keys=['ts'], df=df)
B) fe.create_table(name='fs.tbl', primary_keys=['id'], df=df, schema=schema)
C) fe.write_table(name='fs.tbl', df=df, mode='overwrite')
D) spark.sql('CREATE TABLE fs.tbl AS SELECT * FROM df')

<details>
<summary>Answer</summary>

**Correct: A**

Time-series feature tables require both primary_keys and timestamp_keys at creation time; timestamp_keys is what enables FeatureLookup to perform as-of/point-in-time joins later.

**Why the others are wrong:**
- Without timestamp_keys, the table is a standard (non-time-series) feature table and cannot be used with timestamp_lookup_key.
- write_table only inserts/upserts data into an already-created table; it cannot define the timestamp key.
- A plain SQL CREATE TABLE bypasses the Feature Engineering client entirely and registers no primary/timestamp key metadata, so it can't be used with FeatureLookup.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** A model needs a feature computed from a live external REST API call at inference time, using an ID that is also present in a historical feature table. Which Feature Engineering in Unity Catalog construct is designed for this?

A) FeatureLookup with timestamp_lookup_key
B) A streaming feature table backed by Structured Streaming
C) FeatureFunction referencing a Unity Catalog registered function
D) fe.write_table with mode='merge'

<details>
<summary>Answer</summary>

**Correct: C**

On-demand features via FeatureFunction wrap arbitrary Python/SQL logic (including calls that use request-time inputs) registered as UC functions, executed consistently at training and serving time.

**Why the others are wrong:**
- FeatureLookup only reads precomputed values from an existing feature table — it can't call an external API at inference.
- Streaming feature tables ingest from Structured Streaming sources into Delta; they don't execute per-request logic at serving time.
- write_table just writes/upserts data into a feature table; it has no role in on-demand computation.

</details>

---

**Q6.** Which parameter combination is REQUIRED when calling fe.create_table for any feature table (time-series or not)?

A) name and description only
B) name, timestamp_keys, and mode
C) name, df (or schema), and primary_keys
D) name, primary_keys, and label

<details>
<summary>Answer</summary>

**Correct: C**

Every feature table requires a fully qualified name, a primary key (or composite keys) to uniquely identify rows, and either a df or schema to define columns. timestamp_keys is optional and only needed for time-series tables.

**Why the others are wrong:**
- description is optional metadata, not a required identifier — and primary_keys is missing entirely.
- mode is a parameter of write_table, not create_table, and timestamp_keys is optional.
- label is never a parameter of create_table — labels belong to create_training_set, not the feature table definition.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** When calling fe.write_table on an existing feature table, what is the difference between mode='merge' and mode='overwrite'?

A) merge always creates a new table version while overwrite updates in place
B) merge is used for streaming sources only; overwrite is used for batch sources only
C) merge upserts rows matching the primary key (updating existing, inserting new), while overwrite replaces the entire table contents
D) There is no functional difference — both perform the same upsert operation

<details>
<summary>Answer</summary>

**Correct: C**

merge performs an upsert keyed on the primary key columns, ideal for incremental batch or streaming updates; overwrite replaces all rows in the table, typically used for full recomputation.

**Why the others are wrong:**
- Both modes operate on the current table without necessarily creating a distinct new managed version beyond normal Delta versioning.
- Both modes work with either batch or streaming DataFrames — the mode is independent of the source type.
- The two modes have clearly different semantics; overwrite is destructive to prior rows while merge is not.

</details>

---

**Q8.** How does Unity Catalog support governance for feature tables used in model training?

A) It automatically encrypts all feature values at rest with a separate key from other tables
B) It restricts feature tables to a maximum of 100 columns for auditability
C) It tracks lineage between feature tables, the notebooks/jobs that created training sets from them, and the resulting registered models, visible in Catalog Explorer
D) It requires every feature table to be re-created weekly to maintain compliance

<details>
<summary>Answer</summary>

**Correct: C**

Because feature tables, training sets, and models are all managed under Unity Catalog, UC automatically captures upstream/downstream lineage — which tables fed which training runs and models — enabling governance and impact analysis.

**Why the others are wrong:**
- Feature tables use the same Delta/UC encryption as other managed tables; there is no separate per-table key scheme for this feature.
- There is no such column limit imposed for governance purposes.
- There is no requirement to periodically recreate feature tables for compliance.

</details>

---


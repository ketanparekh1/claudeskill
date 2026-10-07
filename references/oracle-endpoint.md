# Oracle Source/Target — Prerequisites and Limitations

Source: "Setup and User Guide - Qlik Replicate", May 2026 (extracted
sections — the full guide is 1,466 pages; see "Extending this reference"
at the bottom for how to pull sections for other endpoint types).

## Driver prerequisite (source and target)

- **Windows**: Oracle Instant Client for Microsoft Windows (x64),
  version 19.0 or later.
- **Linux**: Oracle Instant Client for Linux (x86-64), version 19.0 or
  later, plus a `libclntsh.so` symbolic link in `$Oracle_Home/lib` and
  `LD_LIBRARY_PATH` configured via `site_arep_login.sh`.
- XMLTYPE data type support requires the **full** Oracle Client, not
  just Instant Client.

## Other source prerequisites
- Oracle account must have read/write privileges (see Required
  permissions).
- Database must run in **ARCHIVELOG mode** (`ALTER database ARCHIVELOG`
  if not already).
- **Supplemental logging** must be enabled (can be automated via the
  endpoint's Advanced tab, or set up manually).

## Known limitations — Oracle as a SOURCE

General:
- Trailing spaces truncated on CHAR columns.
- Function-based indexes, Deferred Constraints not supported.
- PDB: must connect directly to the PDB instance; connecting via the CDB
  to capture from multiple PDBs is not supported.
- Changes from `DBMS_REDEFINITION` are not captured.
- Batch UPDATEs to a numeric Primary Key column are not supported
  (e.g. `UPDATE tableX SET ID=ID+1`).
- LONG/LONG RAW: 64KB max, truncated beyond that; "Allow unlimited LOB
  size" not supported when loading these into an Oracle target (use
  "Limit LOB size to (KB)" instead).
- Table names containing apostrophes are not supported.
- CDC not supported from dynamic views, index-organized tables with an
  overflow segment, or changes to index-organized tables with key
  compression.
- Tables/views of type `BIN$` or `DR$` cannot be replicated.
- Oracle Flashback Database / restore points not supported.
- CDC not supported if schema/table/column name exceeds 30 characters.
- Cascading standby database not supported; failover only supported
  when changes are captured from the primary node.
- RESETLOGS not supported by default — contact Qlik Support if used.
- Oracle Autonomous Data Warehouse as a **source** is not supported.
- Partition/sub-partition ADD/DROP/EXCHANGE/TRUNCATE operations are not
  replicated and can cause "0 rows affected" warnings or "duplicates"
  errors — reload the affected tables to pick up the real data.

Version-specific:
- **Oracle 19.0**: no Data-guard DML redirect, partitioned hybrid
  tables, or schema-only Oracle accounts.
- **Oracle 21c**: LogMiner not supported (Replicate Log Reader only);
  no immutable tables or blockchain tables.
- **Oracle AI Database 26ai**: Standard/Enterprise editions only.

Capture-method-specific:
- **LogMiner**: no UPDATEs to XMLTYPE/LOB columns, no SHRINK SPACE, no
  PDB capture (must use Replicate Log Reader for PDBs), not recommended
  for RAC environments.
- **Replicate Log Reader**: no table clusters; only table-level SHRINK
  SPACE; no online redo logs on raw devices; Amazon RDS for Oracle
  doesn't support ASM/TDE via this method.

Supplemental logging edge case: without
`ENABLE_GOLDENGATE_REPLICATION=TRUE` (has Oracle licensing
implications — customer's responsibility to confirm with Oracle), full
supplemental logging isn't available for tables with column names over
30 characters, and some complex column types (JSON, XMLType, UDTs) may
not be extractable.

Unsupported data types: `BFILE`, `ROWID`, `REF`, `UROWID`, `ANYDATA`,
`SDO_GEOMETRY`, user-defined types, virtual columns. Materialized views
based on a ROWID column are also unsupported (since ROWID itself isn't).

## Known limitations — Oracle as a TARGET

- "Use direct path full load" doesn't support: tables with `INDEXTYPE
  CONTEXT` (use Array Load instead), bidirectional replication, or
  triggers.
- Replicate **cannot create a new schema** on the Oracle target — the
  schema must already exist; specify it in Task Settings' Target
  metadata / Control tables tabs.
- In Batch optimized apply mode, empty LOB columns may replicate as
  NULL.
- With "Limit LOB size to" enabled, NCLOB data is truncated to **half**
  the specified limit after Full Load (CDC truncates to the full limit).
- Oracle AI Database 26ai target: Standard Edition only. Oracle
  Autonomous Data Warehouse 26ai: Enterprise Edition only (via the same
  connectivity method as the Oracle target endpoint).

Required target permissions: `CREATE ANY TABLE`, `CREATE ANY INDEXES`,
`ALTER ANY TABLE`, `DROP ANY TABLE`, `INSERT/UPDATE/DELETE/SELECT ANY
TABLE`, `SELECT all_indexes`/`all_ind_columns`/`all_constraints`/
`all_cons_columns`, and `LOCK ANY TABLE` (only if using direct path full
load).

## Checklist for a new or existing Oracle task

1. Confirm Oracle version is in the supported list (support-matrix.md) —
   19.x, 21c, or 26ai, with edition caveats for 26ai.
2. Confirm Oracle Instant Client 19.0+ (or full Client, if XMLTYPE is
   involved) is installed on the Replicate server, matching the OS.
3. Confirm ARCHIVELOG mode and supplemental logging are enabled on the
   source.
4. Check whether the task's tables hit any of the limitations above —
   especially: table/column name length (30-char CDC limit), LONG/LONG
   RAW columns, unsupported data types, PDB connection setup, and
   partition operations if the source schema uses partitioning.
5. If using Oracle LogMiner specifically, double-check it's not Oracle
   21c+ (LogMiner isn't supported there) and not a RAC environment.
6. For the target: confirm the destination schema already exists (Replicate
   won't create one), and that the connecting user has all required
   permissions listed above.

## Extending this reference to other endpoint types

The full "Setup and User Guide" PDF covers every endpoint type Replicate
supports, each with its own prerequisites and "Limitations and
considerations" section — this file only covers Oracle because that's
the endpoint type in active use.

**Pull the source-chapter and target-chapter sections SEPARATELY, even
for the same technology.** "Using X as a source" and "Using X as a
target" are different chapters with different limitations (this file's
own Oracle source vs. Oracle target sections are the proof — they don't
overlap much). Treat each one as its own lookup:

```bash
pdftotext -layout "Replicate-Setup-and-User-Guide.pdf" setup_guide.txt

# Find BOTH chapters — don't assume one covers the other
grep -n "Using <Endpoint> as a source" setup_guide.txt
grep -n "Using <Endpoint> as a target" setup_guide.txt
# note the line number where each REAL chapter starts (not the TOC
# entries, which appear near the top of the file and are followed
# immediately by a page number on the same line)

# For the source chapter:
awk 'NR==<source_chapter_start>,NR==<next_chapter_start>' setup_guide.txt | grep -n -i "limitations\|prerequisite\|driver"
# For the target chapter:
awk 'NR==<target_chapter_start>,NR==<next_chapter_start>' setup_guide.txt | grep -n -i "limitations\|prerequisite\|driver"
# then read the specific line ranges that come back from each
```

Don't try to read the 1,466-page document linearly or load it all into
context — grep for each chapter title first, then search within that
specific chapter's line range.

If the task's source and target are two different technologies (e.g.
SAP HANA source → Oracle target), this means FOUR lookups total if
starting from scratch: HANA-as-source, HANA-as-target (if ever needed),
Oracle-as-source (if ever needed), Oracle-as-target — though in
practice you only need the two that actually apply to the task at hand
(HANA-as-source and Oracle-as-target, in that example), not all four.

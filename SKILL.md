---
name: qlik-replicate-task-review
description: Use when creating, naming, setting up, or reviewing a Qlik/Attunity Replicate task, source endpoint, or target endpoint — even without those exact words. Covers three things: (1) naming convention for any Replicate artifact (endpoints, tasks, transformations, notifications, schedules); (2) checking a task against documented driver prerequisites, supported versions, and known limitations (detailed for Oracle, extensible to other endpoints); (3) reviewing task health/logs — using qlik-replicate-ops MCP tools when connected, manual fallback otherwise — cross-referenced against known limitations rather than treated as novel. Trigger on "set up a new source endpoint", "what should I call this task", "is this task healthy", "why is this task erroring", "review my Oracle task", or any Replicate create/name/diagnose request.
---

# Qlik Replicate Task Review (naming + version/limitation checks + log review)

Three things this skill does together, because they come up at related
moments in a task's lifecycle: naming it, setting it up correctly, and
diagnosing it once running.

## Part 1: Naming convention

A proposed convention, not vendor-mandated — the
value comes from applying it *consistently*, not from the exact letters.

### Get the org's real codes first

Never reuse the illustrative placeholder codes below (`AAA`, `SAA`,
`ORA`, etc.) as if they were real. Before naming a real artifact, confirm:
- **Environment** `<ENV>` — e.g. `D`/`U`/`P` for Dev/UAT/Prod
- **Project code** `<PRJ>` — short acronym per project (ask if unknown)
- **Source/target system name** `<SOU>`/`<TRG>` — short code per system
- **Flavour** `<FLV>` — short code for the database/technology (`ORA`,
  `SQL`, `HDP`, `KAF`, `DB2z`, etc.)

If these were already established earlier in the conversation/project,
reuse them. Otherwise ask once, then keep using the same codes.

### Patterns

| Artifact | Pattern | Example (illustrative codes) |
|---|---|---|
| Source endpoint | `<ENV>_<PRJ>_S_<SOU>_<FLV>` | `D_AAA_S_SAA_DB2z` |
| Target endpoint | `<ENV>_<PRJ>_T_<TRG>_<FLV>` | `P_AAA_T_DWH_ORA` |
| LSS write stage (source chain) | `<ENV>_<PRJ>_T_<name>_LSS` | `D_BBB_T_SCC_LSS` |
| LSS read stage (target chain) | `<ENV>_<PRJ>_S_<name>_LSS` | `D_BBB_S_SCC_LSS` |
| Replication task | `<ENV>_<PRJ>_<SOU>_<TRG>` (+ optional `_<XXX>`) | `D_AAA_SAA_DWH` |
| Transformation/notification/schedule — task-specific | `<ENV>_<TPG>__<Task>__<Purpose>` | `D_T__D_AAA_SAA_DWH__Add_Datetime` |
| — project-specific | `<ENV>_<TPG>__<PRJ>__<Purpose>` | `D_P__AAA__Add_Datetime` |
| — non-specific/global | `<ENV>_<TPG>__<Purpose>` | `D_G__Send_START_Alert` |

Task names drop `<S/T>` and `<FLV>` — built from source/target *names*
only, since those are already disambiguated in the endpoint names. The
transformation/notification/schedule patterns use a double underscore
(`__`) to separate scope from purpose — don't collapse it to one.

### When naming something

1. Identify the artifact type.
2. Confirm/recall the org's real codes.
3. Apply the matching pattern.
4. Show the result with a one-line breakdown of each segment, so the
   user can sanity-check before using it.

### When asked to rename/audit existing artifacts

Don't assume existing non-conforming artifacts should be renamed —
ask whether the user wants the convention applied going forward (new
artifacts only) or wants to actually rename live artifacts, which has
real operational impact on a running system, not just cosmetic.

---

## Part 2: Driver, version, and limitation checks

**Every task has a source endpoint type AND a target endpoint type, and
they must be checked SEPARATELY — never assume one check covers both.**
This matters for two distinct reasons:
1. **They're often different technologies entirely** — a task might be
   SAP HANA → Oracle, or Oracle → SQL Server, or any other combination.
   Each technology has its own prerequisites, supported versions, and
   limitations, documented in its own section of the Setup and User
   Guide.
2. **Even when source and target are the SAME technology, the
   limitations differ by role.** Oracle-as-a-source and
   Oracle-as-a-target are genuinely different chapters in the guide
   with different content — e.g. the source side has the 30-character
   CDC name limit and LONG/LONG RAW truncation; the target side has the
   "can't create a new schema" and NCLOB-truncation-after-full-load
   behaviors. Checking only one side's reference and assuming it covers
   both roles will miss real, documented limitations.

So: **first, explicitly identify the source endpoint type and the
target endpoint type for this specific task** (ask if not already
stated, or pull from `get_task_details`/`export_task_raw` if the MCP
tools are connected) — then look up each one independently.

Reference files in `references/`:

- `references/support-matrix.md` — supported versions/platforms for
  every endpoint type Replicate supports (condensed from the official
  Support Matrix), covering both source and target support in one
  table per technology.
- `references/oracle-endpoint.md` — driver prerequisites and detailed
  known limitations for Oracle, with sections split by source vs.
  target role. Covers Oracle only, since that's the endpoint type
  currently in active use. For any OTHER endpoint type (as source, as
  target, or both), there is no pre-extracted reference yet — pull one
  using the technique in that file's "Extending this reference"
  section, extracting the source-chapter and target-chapter sections
  separately even for the same technology, before relying on either.

Checklist when setting up or reviewing a task — do this once per
endpoint (source, then target; twice total even if it's the same
technology on both sides, since the limitations differ by role):
1. Confirm that endpoint's database version is in the supported list.
2. Confirm the required driver (and its version) is installed on the
   Replicate server for that endpoint type.
3. Check that endpoint's role-specific limitations list (source
   limitations when checking the source; target limitations when
   checking the target) for anything relevant to this task's actual
   tables/columns.
4. Flag anything that looks like it'll be a real problem for this
   specific task (not just a generic disclaimer) — e.g. "this source
   has columns named over 30 characters, which breaks CDC for Oracle"
   is worth surfacing; "Oracle has limitations" alone is not useful.

---

## Part 3: Task health / log review

When asked to check on a task's health or diagnose an issue:

**If the qlik-replicate-ops MCP tools are connected** (check your
available tools for `list_replication_tasks`, `get_task_details`,
`get_task_latency`, `get_table_statuses`, etc.): use them to pull the
task's real current state, latency, error message, and per-table status
directly, rather than asking the user to describe it. `get_task_details`
surfaces the task's current error `message` field, which is the first
place to look when something's wrong.

**If those tools aren't connected**: ask the user to paste the relevant
error message or log excerpt from the Replicate/EM console (View Logs),
rather than guessing.

**Either way — cross-reference findings against Part 2's limitations,
don't treat every error as novel.** If an error message or symptom
matches something documented in `references/oracle-endpoint.md` (or
the equivalent reference for the relevant endpoint type), say so
explicitly and point to the specific limitation — that's often faster
to resolve than treating it as a fresh problem to debug from scratch.
Examples of this pattern: a batch UPDATE to a numeric primary key
failing silently → matches the documented Oracle source limitation on
batch PK updates; a "0 rows affected" warning right after a partition
operation → matches the documented partition/sub-partition limitation.

If nothing in the references matches, say so plainly rather than
forcing a fit — it may be a genuinely new issue worth escalating to
Qlik Support or investigating fresh.

# Qlik Replicate task review skill for Claude

Skill name: `qlik-replicate-task-review`

A [Claude skill](https://www.anthropic.com/news/skills) that helps with Qlik Replicate (formerly Attunity Replicate) work: naming things consistently, checking a task's endpoints against documented prerequisites and limitations, and diagnosing a task that is erroring or lagging.

> Unofficial. Not affiliated with or endorsed by Qlik or Anthropic. The naming convention is a proposed standard, not a vendor requirement.

## What it does

The skill has three parts. Claude loads it automatically when a request matches, so you just ask in plain language.

**1. Naming convention.** Names endpoints, tasks, transformations, notifications and schedules with a consistent pattern, and shows what each part of the name means.

| Artifact | Pattern |
|---|---|
| Source endpoint | `<ENV>_<PRJ>_S_<SOU>_<FLV>` |
| Target endpoint | `<ENV>_<PRJ>_T_<TRG>_<FLV>` |
| Replication task | `<ENV>_<PRJ>_<SOU>_<TRG>` |
| LSS write / read stage | `<ENV>_<PRJ>_T_<name>_LSS` / `<ENV>_<PRJ>_S_<name>_LSS` |
| Transformation, notification, schedule: task-specific | `<ENV>_<TPG>__<Task>__<Purpose>` |
| Transformation, notification, schedule: project-specific | `<ENV>_<TPG>__<PRJ>__<Purpose>` |
| Transformation, notification, schedule: global | `<ENV>_<TPG>__<Purpose>` |

`<TPG>` is the scope code (the skill's examples use `T`, `P` and `G` for task, project and global). These patterns use a double underscore (`__`) between scope and purpose. Task names are built from the source and target names only, without the S/T marker or flavour.

Codes for environment (`<ENV>`), project (`<PRJ>`), system (`<SOU>`/`<TRG>`) and technology flavour (`<FLV>`, e.g. `ORA`, `SQL`) are specific to your organisation, so the skill asks for them instead of inventing them. It also asks before suggesting that existing live artifacts be renamed, because that affects a running system.

**2. Driver, version and limitation checks.** For a task, the source and target endpoints are checked separately, even when both are the same technology, because limitations differ by role (for example Oracle as a source vs. Oracle as a target). The checklist covers supported database versions, required drivers on the Replicate server, and role-specific known limitations that apply to the task's actual tables and columns.

**3. Task health and log review.** Diagnoses a task using real state when the optional `qlik-replicate-ops` MCP tools are connected (task details, latency, per-table status), or from a pasted log excerpt otherwise. Findings are cross-referenced against the documented limitations rather than treated as new problems.

## Contents

```
SKILL.md                      Skill instructions and checklist
references/
  support-matrix.md           Supported versions and platforms by endpoint type
  oracle-endpoint.md          Oracle driver prerequisites and limitations, split by source and target role
```

Detailed limitation notes exist for **Oracle only**. For other endpoint types the skill uses the support matrix and points to the "Extending this reference" section of `references/oracle-endpoint.md` for adding more.

## Install

**Claude.ai / Claude app:** zip the skill folder so that `SKILL.md` and `references/` sit inside a single top-level folder, then go to Settings > Capabilities > Skills and upload the zip.

```powershell
git clone https://github.com/ketanparekh1/claudeskill.git qlik-replicate-task-review
Compress-Archive -Path .\qlik-replicate-task-review -DestinationPath .\qlik-replicate-task-review.zip
```

**Claude Code:** copy the folder into `~/.claude/skills/` (or `.claude/skills/` in a project).

Start a new chat after installing; skills added mid-conversation may not be picked up.

## Example prompts

- "What should I call the Oracle source endpoint for the HR production database, replicating to Snowflake?"
- "Name a Replicate task that moves SQL Server sales data to Snowflake."
- "Review this Oracle task against the documented prerequisites and limitations." (paste the task settings)
- "Why is this Replicate task erroring?" (paste the log excerpt, or connect the MCP tools)

## Optional: live task data

Part 3 works best with an MCP server that exposes Qlik Enterprise Manager tools (`list_replication_tasks`, `get_task_details`, `get_task_latency`, `get_table_statuses`). Without it, the skill asks you to paste error messages or log excerpts instead.

## Limitations

- The naming convention is a starting point; adapt the codes to your organisation and apply them consistently.
- Limitation reference content is condensed from Qlik documentation and the Support Matrix. Check the official documentation for your Replicate version before relying on it for production decisions.

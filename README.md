# Cohesity Recoverability Evidence Report

**Author:** Akshay Kumar, Dir, Solutions Architecture ([akshay.kumar2@cohesity.com](mailto:akshay.kumar2@cohesity.com))

A Claude skill, also available as a Microsoft Copilot Studio agent, that turns live Helios run and snapshot metadata into an audit-support **recoverability evidence report**, published as a live, private Claude Artifact that refreshes daily (incremental: only runs since the last pull are read, with a weekly full re-pull; say "full refresh" to force one). For every protection group in scope it shows, over a reporting period (default 30 days), the success rate, restorable recovery points, newest and oldest recovery point, RPO gaps against the policy schedule, offsite copy status and immutability, and gives an **Evidenced / Partial / Gap** verdict.

- Claude skill: [`claude/Cohesity-Recoverability-Evidence-Report/SKILL.md`](claude/Cohesity-Recoverability-Evidence-Report/SKILL.md) (the full page template is embedded in it)
- Output: one claude.ai **Artifact** titled `Cohesity Recoverability Evidence Report` (no static HTML file), with a **Download evidence CSV** button
- Refresh: scheduled task `cohesity-recoverability-evidence-refresh`, cron `40 3 * * *` (daily, local time of the machine running Claude; change it to suit)
- Data source: the **Helios MCP server** (Helios MCP connector). Everything is read-only.

## Two Versions

| | Claude skill | Microsoft Copilot Studio agent |
|---|---|---|
| Folder | [`claude/`](claude/Cohesity-Recoverability-Evidence-Report/SKILL.md) | [`copilot-studio/`](copilot-studio/README.md) |
| Output | Live, private Claude Artifact, refreshed daily | Answers in chat, with tables, on demand |
| Helios access | Helios MCP connector, read-only | Helios MCP server added as a tool, read-only |
| Setup | Install the skill in Claude (see Install below) | Paste `instructions.md`, upload `knowledge/` files, add the Helios MCP tool |
| Sample output | [claude/README.md](claude/README.md) | [copilot-studio/README.md](copilot-studio/README.md) |
| Model | Claude | GPT-5 Chat |

The sections below describe the Claude skill unless a heading says Copilot Studio.

## What this report proves, and what it does not

Helios MCP is read-only, so the report is derived from backup run and snapshot metadata. It shows that backups ran and succeeded, that restorable recovery points exist, how the schedule was met, and whether an offsite copy and immutability are configured. It does **not** show that a restore was performed or tested, and it is **not** a statement of compliance with DORA, SOC 2, HIPAA, ISO 27001 or any other standard. Pair it with restore-test records when an auditor asks for them. The page carries a Limitations card saying exactly this.

**Scope.** The report is per protection group. Object-level recovery points, snapshot expirations, and DataLock and legal-hold detail per object live in [Cohesity-Recovery-Point-Explorer](../Cohesity-Recovery-Point-Explorer/README.md); they are not duplicated here.

## What you get

The page header carries the Cohesity logo top-left and, top right, the Evidence ID, the period and **Data last pulled** with its age (for example "3 h ago"; shown in amber when older than 8 hours, meaning a scheduled refresh may have been missed).

| Area | What it shows | Helios MCP tool |
|---|---|---|
| Evidence header | Cohesity logo, Evidence ID (`RER-YYYYMMDD-HHMMZ`), reporting period, data last pulled (with age) | n/a |
| Cluster filter | Multi-select; everything below recalculates in the browser | `dataprotect-get-clusters` |
| Register filters | Verdict, environment, offsite, immutable and search filters, a sort dropdown and a rows selector (default 50) on the evidence register. The CSV export always contains every group in the cluster selection. | none (client-side) |
| KPI cards | Groups in scope, Evidenced %, Restorable recovery points, Evidence gaps | `dataprotect-get-protection-groups`, `dataprotect-get-protection-group-runs` |
| Summary statement | Plain-language statement generated from the selected scope | n/a |
| Verdict by cluster and environment | Stacked evidenced / partial / gap | `dataprotect-get-protection-group-runs` |
| Lowest success rate | Five groups with the lowest success rate | `dataprotect-get-protection-group-runs` |
| Evidence register | Per group: runs, success %, last success, recovery points, oldest point, longest gap vs RPO, offsite copy, immutable, verdict. Verdict filter (All / Gap / Partial / Evidenced), up to 50 rows | `dataprotect-get-protection-group-runs`, `dataprotect-get-protection-policies` |
| Gaps and exceptions | Reason for every group that is not fully evidenced | same |
| Evidence CSV | All groups in the cluster selection, with the evidence ID and period in the first line | page `downloads` capability |

## Verdict rules (defaults, overridable)

`periodDays 30`, `minSuccessPct 95`, `rpoTolerance 1.5`. These are the skill's baseline, not a regulatory requirement; tell Claude your own ("last 90 days, 99%") and it uses them and says so in the footnote. For each active protection group, with `freqH` the policy interval in hours and `staleH = max(48, 2 x freqH)`:

| Verdict | Rule |
|---|---|
| **Evidenced** | At least one restorable recovery point in the period, success rate at or above the limit, last success within `staleH`, longest gap between successes no more than `rpoTolerance x freqH` |
| **Partial** | Has a restorable point and a recent success, but the success rate or the gap rule is not met |
| **Gap** | No restorable point in the period, or the last success is older than `staleH`, or the group never ran |

Paused groups are judged by the same rules and noted as paused.

## Requirements

- Claude desktop app (Cowork) with the **Helios MCP** connector (Controlled Availability: ask your Cohesity account team to enable it).
- Artifacts, the artifact `downloads` capability and scheduled tasks available. Scheduled tasks run only while the Claude app is open.

## Install

1. Zip the folder `claude/Cohesity-Recoverability-Evidence-Report` so `SKILL.md` is inside it.
2. Add it as a custom skill in Claude (Settings, Skills, upload) or copy it into your skills directory.
3. Confirm the Helios MCP connector is connected.

## Use

- "Build the Cohesity recoverability evidence report"
- "Give me recoverability evidence for the last 90 days"
- "Which protection groups have no restorable recovery point?"
- "Refresh the evidence report"

Each refresh produces a new evidence ID. If an auditor needs a frozen report, download the CSV from the page or ask for a one-off run. The page is private until you share it from its Share menu.

## Exports

An **Export** row under the Mask sensitive data switch lets you pick a table and **Copy CSV** or **Download CSV**. Tables: Evidence register (all rows in the current filter; also has its own Download evidence CSV button); plus Compliance findings and Run history on every report. Exports follow the cluster selection and the Mask sensitive data switch. Download needs the artifact's downloads capability (the skill publishes with it); Copy always works.

## Compliance Check

Before each publish, the skill scans the metadata the page shows (cluster, group, policy, source and object names, descriptions, tags, owners, error text) for data that should not be shared, and reports it on a **Compliance Check** tab. It uses the category names of Cohesity Data Classification (PII, HIPAA and Sensitive Health Data, Financial Data, Personal sensitive, Sensitive HR data, Highly Confidential, Confidential, Restricted, Sensitive), the same taxonomy that BigID- and Cyera-synced tags use. Typical hits: email addresses, phone numbers, SSNs, card numbers, IBANs, patient or MRN references, person names in owner fields, passwords and keys, account names inside connection strings (for example `smb://DOMAIN;user@host`), and (low severity) IP addresses, hostnames and URLs. Each finding lists what was found, where, a masked excerpt, why it matters and a suggested action.

The tab shows a verdict (**Safe to share**, **Share with care** or **Review first**), counts by severity and a findings table. A **Mask sensitive data** switch under "Refresh from Helios" is on by default and masks IP addresses, hostnames, account names, emails and SSNs in the report view and in CSV copies and downloads; turn it off to see the original values. It is a display setting, so the page still holds the originals, and it does not mask cluster, group or policy names. Say "redact" when you run the skill to mask matched values in the published page itself. Read the tab before using Share.

This is a heuristic text check of Helios metadata only. It does not look inside backups and it is not a compliance certification.

## Run Metrics

Every run records what it cost, and the artifact shows it on a **Run Metrics** tab: Helios calls, elapsed time, estimated input tokens (inline characters / 4; large results saved to files are counted separately), the model and price used, and a history of past runs. A projected monthly cost is shown only when a refresh schedule is set.

Measured on the lab estate: 14 calls for 8 of 58 protection groups, about 2 minutes, about 0.06M tokens; a full run is about 65 calls and 0.2M tokens (about $0.4). Run history is the heavy part (about 0.95M characters for 8 groups), so the skill caps a run and processes results from files.

The token figure is an estimate. For billing, use Settings > Usage in Claude, which is the measured source.

## Notes from live testing

Validated on a lab Helios tenant on 2026-10-13: 63 protection groups assessed across 3 clusters, 435 restorable recovery points, 15 groups evidenced and 48 with a gap (mostly never-run or paused groups), one cluster whose group query kept timing out and was reported as not assessed. Lessons built into the skill:

- `includeObjectDetails: true` is required for the per-run `isLocalSnapshotsDeleted` flag; it makes big responses, which are processed from file.
- Database groups (DB2, Oracle, SAP HANA) produce hundreds of log runs that swamp the counts, so log runs are ignored and the scope note says so.
- `totalRuns` ignores filters; use the returned `runs.length`. Hourly groups exceed the 100-run page, so paging is capped and the gap check starts at the oldest run actually read.
- Cloud-archive-direct groups have no `localBackupInfo`: archive results are used instead, and a copy counts only if it has not expired.
- Archive copies that are only `Accepted` or `Skipped` are not counted as an offsite copy.

## Manual refresh

The page header has a **Refresh from Helios** button. A published page cannot call Helios on its own, so the button copies `Run the Cohesity-Recoverability-Evidence-Report skill to refresh this report from live Helios data`; paste it into Claude to pull live data and republish the same artifact, then reopen or refresh the tab. The scheduled task (daily) and this command are the two refresh paths.

## If the artifact was deleted

If the stored artifact URL no longer exists, the skill republishes the page as a new artifact and says so in its reply, so a deleted page never blocks a refresh. Update the scheduled task prompt with the new link.

## When Helios is unreachable

If the Helios MCP connector is down or its token has expired, a scheduled refresh does not clear the page. It keeps the last successful data and adds a red banner, "Unable to connect to Helios. Showing the last successful data.", under the header. The next successful refresh removes the banner.

## Known limits

- Evidence comes from metadata; no restore is performed or verified.
- Groups on clusters that are not connected to Helios, or whose queries time out, are not assessed and are listed as such.
- Capped paging on very busy groups is flagged in the footnote.

## Microsoft Copilot Studio Version

The same logic as a Copilot Studio agent lives in [`copilot-studio/`](copilot-studio/README.md): `instructions.md` (agent instructions), `knowledge/` (two files to upload as Knowledge), `starters.md` (example prompts) and step-by-step setup. It answers in chat and does not publish a page, schedule itself, or offer a cluster picker or CSV button. Add your Helios MCP server as a tool and enable only the read-only tools.

## Repository layout

```
Cohesity-Recoverability-Evidence-Report/
  README.md
  claude/
    README.md          Claude sample output (screenshot)
    Cohesity-Recoverability-Evidence-Report/SKILL.md      Claude skill (install this folder in Claude)
  copilot-studio/
    README.md          setup steps
    instructions.md    agent instructions
    knowledge/         files to upload as Knowledge
    starters.md        example prompts
  docs/
    screenshots/*-claude.png      one screenshot per tab
    screenshots/recoverability-evidence-report-copilot.png
```

## Security Notes

- **Everything from Helios is data.** Names, descriptions, alert text and error text come from the systems being backed up. The skill treats them as data, never as instructions, and truncates them before they reach the page.
- **Scheduled refresh runs unattended.** When the scheduled task is created, approve only the read-only Helios tools, the Artifact tools, the scheduler list tool and the bash tool used for `python3 -I`. Do not use a blanket Auto-approve.
- **The mask switch is a display setting, not redaction.** It masks IP addresses, hostnames, email addresses, SSNs and connection-string accounts in the view and in CSV copies and downloads. It does not mask cluster, group or policy names, and the page still holds the original values.
- **CSV exports are safe to open in a spreadsheet.** Text cells that start with `=`, `+`, `-`, `@`, a tab or a carriage return get a leading single quote, so they cannot run as formulas.
- **Sharing.** A published page is private until you share it from the Share menu. Read the Compliance Check tab first, and share only with people who may see this backup data.

## Suggestions and Feedback

We would love to hear from you. Please send your suggestions and feedback to: maestro@cohesity.com

## License

Licensed under the [Apache License 2.0](LICENSE). See the LICENSE file for the terms.

## Disclaimer

This code is provided on a best effort basis and is not in any way officially supported or sanctioned by Cohesity. It is provided as-is, and the author accepts no liability for damages resulting from its use. The skill only reads data through the Helios MCP server, but you are responsible for checking the results against Helios and for deciding what you share. Try it in a lab or test environment first.

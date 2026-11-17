Copyright Cohesity 2026, SPDX-License-Identifier: Apache-2.0

# Cohesity Recoverability Evidence Report

**Author:** Akshay Kumar, Dir, Solutions Architecture ([akshay.kumar2@cohesity.com](mailto:akshay.kumar2@cohesity.com))

A Claude skill, also available as a Microsoft Copilot Studio agent (evidence report only), that turns live Helios run and snapshot metadata into an audit-support **recoverability evidence report** and a recovery-point explorer, published as one live, private Claude Artifact that you refresh on request or on a schedule that you set in the page's Settings tab (incremental: only runs since the last pull are read, with a weekly full re-pull; say "full refresh" to force one). For every protection group in scope it shows, over a reporting period (default 30 days), the success rate, restorable recovery points, newest and oldest recovery point, RPO gaps against the policy schedule, offsite copy status and immutability, and gives an **Evidenced / Partial / Gap** verdict. The same data pass also feeds the recovery-point views (expirations, points past retention, DataLock and legal hold, objects with no recent good backup) that used to live in Cohesity-Recovery-Point-Explorer, which is now merged into this skill. Where Helios troubleshooting tools are available, gap groups also show their failure priority and a likely cause.

- Claude skill: [`claude/Cohesity-Recoverability-Evidence-Report/SKILL.md`](claude/Cohesity-Recoverability-Evidence-Report/SKILL.md) (the full page template is embedded in it)
- Output: one claude.ai **Artifact** titled `Cohesity Recoverability Evidence Report` (no static HTML file), with a **Download evidence CSV** button
- Refresh: on request (the **Refresh from Helios** link opens a new Claude chat with the command typed in; press Enter there, then reload the page; a Copy command button is next to it), or on a schedule that you create, change or turn off in the page's **Settings tab** (card "Refresh schedule": every day, weekdays or weekly, at a time you choose, in the time zone you choose; the page converts it and the skill turns it into the local time of the machine running Claude, and says daylight saving changes can shift the local run time by one hour). The skill never asks about scheduling in chat. The card saves your request in the page's private database. Click **Apply in Claude** in the card and press Enter in the chat that opens: the skill runs in schedule-only mode (no Helios calls, no new data), creates or updates the scheduled task `cohesity-recoverability-evidence-refresh` to match (a turned-off schedule is disabled, never deleted) and republishes the page changing only the schedule it found, so the card shows "In sync" (or "Not applied yet. Click Apply in Claude"); every normal run also applies a saved request at its start. Scheduled tasks run only while Claude is open on that computer; they may be moved to the cloud later. Only the page owner and editors can save a request. The Refresh and Apply links need this skill and the Helios connector (Refresh only) in the viewer's own Claude; for a viewer who is not signed in to Claude (for example a public link) the Refresh button, Save schedule and Turn off schedule are disabled with a reason, and Copy command still works.
- Data source: the **Helios MCP server** (Helios MCP connector). Everything is read-only.

## Two Versions

| | Claude skill | Microsoft Copilot Studio agent |
|---|---|---|
| Folder | [`claude/`](claude/Cohesity-Recoverability-Evidence-Report/SKILL.md) | [`copilot-studio/`](copilot-studio/README.md) |
| Output | Live, private Claude Artifact, refreshed on request (optional schedule: set in the page's Settings tab and applied with one click) | Answers in chat, with tables, on demand |
| Helios access | Helios MCP connector, read-only | Helios MCP server added as a tool, read-only |
| Setup | Install the skill in Claude (see Install below) | Paste `instructions.md`, upload `knowledge/` files, add the Helios MCP tool |
| Sample output | [claude/README.md](claude/README.md) | [copilot-studio/README.md](copilot-studio/README.md) |
| Model | Claude | GPT-5 Chat |

The sections below describe the Claude skill unless a heading says Copilot Studio.

## What this report proves, and what it does not

Helios MCP is read-only, so the report is derived from backup run and snapshot metadata. It shows that backups ran and succeeded, that restorable recovery points exist, how the schedule was met, and whether an offsite copy and immutability are configured. It does **not** show that a restore was performed or tested, and it is **not** a statement of compliance with DORA, SOC 2, HIPAA, ISO 27001 or any other standard. Pair it with restore-test records when an auditor asks for them. The page carries a Limitations card saying exactly this.

**Scope.** The Evidence tab is per protection group. Object- and snapshot-level answers (which recovery points exist, when they expire, DataLock and legal hold, objects with no recent good backup) are on the other tabs, built from the same collected data, so the numbers shared between tabs agree. The Copilot Studio agent covers the Evidence tab only.

**Merged skill.** Cohesity-Recovery-Point-Explorer is deprecated and merged into this skill; its artifact is replaced by the Recovery Points, Expiry & Locks and Objects at risk tabs. Delete its scheduled task and archive that repository. One difference to know: the Explorer read 7 days of runs, this skill reads the reporting period (default 30 days) for every tab. Other small definition differences are listed in the skill ("Where the two earlier skills differed").

## What you get

The page header carries the Cohesity logo top-left and, top right, the Evidence ID, the period and **Data last pulled** with its age (for example "3 h ago"; shown in amber when a refresh schedule is set and the data is older than 36 hours, meaning the scheduled refresh may have been missed; the header says "Refreshes <schedule>" only when an enabled schedule exists, and says nothing about refreshing otherwise).

**Tabs:** Evidence (below), **Recovery Points** (restorable points, newest and oldest point, objects in the newest run, offsite state, Current / Stale / reason per group), **Expiry & Locks** (points expiring in 7 and 30 days, points past retention, policy DataLock and mode, runs on legal hold, active snapshots without DataLock), **Objects at risk** (protected objects with no good snapshot inside the limit, default 24 h, at most 500 rows with the total), **Compliance Check**, **Run Metrics** and **Settings**. One cluster filter, the mask switch and the Export row apply to all data tabs.

**Theme and footer.** A single icon button (sun or moon) at the top right of the page header controls the colour theme. The page follows the browser setting on load and switches immediately whenever the browser or system scheme changes; clicking the button flips the theme for the current page view only, and nothing is stored. The button is hidden in print. A small footer at the bottom of every tab reads "Copyright Cohesity 2026, SPDX-License-Identifier: Apache-2.0" and stays on the page when the Helios-unavailable banner is shown.

| Area | What it shows | Helios MCP tool |
|---|---|---|
| Evidence header | Cohesity logo, Evidence ID (`RER-YYYYMMDD-HHMMZ`), reporting period, data last pulled (with age) | n/a |
| Cluster filter | Multi-select; everything below recalculates in the browser | `dataprotect-get-clusters` |
| Register filters | Verdict, environment, offsite, immutable and search filters, a sort dropdown and a rows selector (default 50) on the evidence register. The CSV export always contains every group in the cluster selection. | none (client-side) |
| Landing widgets | A hero ring Evidenced %, tiles for Groups in scope, Evidence gaps, Restorable recovery points, Objects at risk, Expiring in 7 days and Stale groups (with change chips and sparklines from the trend), and a bar of verdicts. Click a tile to jump to the table that lists its rows (filtered where there is a filter); the stat tiles on the other tabs and the Compliance Check use the same tile look. Notices about tools that were off or unavailable are short pills with a Details button. | `dataprotect-get-protection-groups`, `dataprotect-get-protection-group-runs` |
| Summary statement | Plain-language statement generated from the selected scope | n/a |
| Verdict by cluster and environment | Stacked evidenced / partial / gap | `dataprotect-get-protection-group-runs` |
| Lowest success rate | Five groups with the lowest success rate | `dataprotect-get-protection-group-runs` |
| Evidence register | Per group: runs, success %, last success, recovery points, oldest point, longest gap vs RPO, offsite copy, immutable, verdict. Verdict filter (All / Gap / Partial / Evidenced), up to 50 rows | `dataprotect-get-protection-group-runs`, `dataprotect-get-protection-policies` |
| Gaps and exceptions | Reason for every group that is not fully evidenced, plus Priority and Why from Helios troubleshooting and, for up to 2 (scheduled run) or 3 (manual run) gap groups, an expandable likely cause with confidence and correlation ID (only when Cohesity AI agents are enabled in Settings; otherwise n/a) | same, `troubleshooting-list-backup-failures`, `troubleshooting-triage-backup-failure` |
| Recovery Points, Expiry & Locks, Objects at risk | Recovery points per group, expirations and retention, DataLock and legal hold, objects without a recent good backup (Priority and Why per group) | `dataprotect-get-protection-group-runs`, `dataprotect-get-object-snapshots` (sample only), `dataprotect-get-protection-groups` |
| Evidence CSV | All groups in the cluster selection, with the evidence ID and period in the first line | page `downloads` capability |

## Verdict rules (defaults, overridable)

`periodDays 30`, `minSuccessPct 95`, `rpoTolerance 1.5`. These are the skill's baseline, not a regulatory requirement; tell Claude your own ("last 90 days, 99%") and it uses them and says so in the footnote. For each active protection group, with `freqH` the policy interval in hours and `staleH = max(48, 2 x freqH)`:

| Verdict | Rule |
|---|---|
| **Evidenced** | At least one restorable recovery point in the period, success rate at or above the limit, last success within `staleH`, longest gap between successes no more than `rpoTolerance x freqH` |
| **Partial** | Has a restorable point and a recent success, but the success rate or the gap rule is not met |
| **Gap** | No restorable point in the period, or the last success is older than `staleH`, or the group never ran |

Paused groups are judged by the same rules and noted as paused.

## Helios troubleshooting

When Cohesity AI agents are enabled in Settings (off by default), the skill calls `troubleshooting-list-backup-failures` once per run and adds the group's priority score and reason (and message code) to partial and gap groups and to the Objects at risk table ("n/a" when there is no matching failure in the last 24 hours). For the worst gap groups that failed in that window (2 on a scheduled run, 3 on a manual run, one failure pattern once) it calls `troubleshooting-triage-backup-failure` one call at a time with no retry (a call can take about 3 minutes; after 2 consecutive failures the rest are skipped and counted in the footnote) and shows the likely cause in an expandable detail with confidence and the correlation ID, or "Needs Support" when Helios recommends escalation. These tools are not available on every tenant or connection, and they are not called while Cohesity AI agents are off in Settings: in both cases the skill uses its existing reasons, the columns show n/a with the reason and the footnote says which path was used. They never change a verdict or a recovery-point number. Their output is treated as data: only allow-listed fields are shown, never commands, internal hostnames or log text.

## Summary, PDF, trend and call economy

- **Summary strip.** The top of the Evidence tab says in three sentences how many groups are evidenced, how many restorable points exist (and how many groups have none or are stale) and how many objects have no recent good backup. The sentences sit under the tiles as a "Read as text" toggle (visible in the PDF) and use the same numbers as the tiles.
- **Download as PDF.** The page sandbox blocks print dialogs, so this header button builds a real PDF in your browser from the data already on the page: one PDF with a section for every report tab in tab order (the Settings tab is never exported), the header and summary once on the first page, each section on a new page under a header band with the tab name, A4 portrait, light colours, mask and cluster selection as displayed, with a footer (title, UTC time, tab name, page n of N) on every page. Tables show all rows up to 500 per table ("First 500 of N rows. Use Download CSV for all rows.") and the PDF stops at 150 pages with a note listing what was left out. The status line reads "Building the PDF... tab 3 of 6" while it works, then "PDF saved (n pages, k tabs)." No data is sent anywhere. It needs `cdnjs.cloudflare.com` to be reachable (two pinned libraries are loaded on the first click) and is saved through the downloads capability, which asks you to confirm.
- **Trend.** Each run adds one compact snapshot (evidenced, partial, gap, restorable points, stale groups, objects at risk) to the page data, up to 30 days. From the third snapshot the tiles show a small sparkline with a text alternative, and from the second snapshot they show a change chip against the previous run (all clusters selected).
- **Figures typed by hand.** If the run had to type values in from tool results that arrived inline instead of processing them from a file, Run Metrics and the end of the Evidence tab say so and list them.
- **Masking.** The mask switch also covers 12-digit cloud account ids, AWS ARNs and Azure subscription or tenant IDs in paths (the last four characters stay), and the Compliance Check lists them as low (Sensitive) findings. While it is on, protection group names and object names are shortened everywhere they are shown (first two letters and `***`, tooltips, charts, CSV and PDF included); cluster and policy names stay readable, and search still matches the real names.
- **Fewer calls.** The object-snapshot sample skips groups that cannot hold a snapshot (retention not longer than the period and no successful run in it), makes one call per group and at most 10 per run. Live check on the lab estate: 46 Helios calls instead of 75 (30 run calls instead of 49, 6 snapshot calls instead of 16) with the same figures. The runs tool takes one group per call, so only unchanged groups can be skipped.
- **Owner setting.** `SIBLING_URLS` (first line of the page script, empty by default) turns the muted "See Cohesity-Protection-Coverage-Audit" hints into links.

## Settings and data handling

The **Settings** tab (last tab) shows, in one table, how the page uses your data and lets you choose whether Cohesity AI agents are used.

- **Data tools (always on).** The dataProtect get and reporting get tools query your Cohesity clusters directly. Results go only to the AI assistant that runs the skill. Nothing is sent through Helios to an LLM.
- **Cohesity AI agents (optional, off by default).** The troubleshooting tools send details of failed backups and alerts to Cohesity's LLM subscription to suggest root causes, so they are off until you turn them on. While they are off the page is built from the data tools only: failed backups are shown without Priority, Why or likely cause, and the page says "Turned off in Settings" with a button to open the Settings tab instead of reporting a failure.
- **How you turn them on.** In the Settings tab, press the switch. A popup explains how Cohesity handles your data when it is sent to Cohesity's own LLM subscription and asks you to press "I accept and enable Cohesity AI agents"; Cancel changes nothing. Turning them off is one click. Only editors of the page can change the switch; others see it disabled. A change takes effect on the next refresh and applies to this page only. The skill only reads the setting and never changes it.
- **Pricing source (read-only for the skill).** The Run Metrics tab holds an editable link used to look up the price per million input tokens (default `https://platform.claude.com/docs/en/about-claude/pricing`). Only editors of the page can change it; it is stored as `settings/pricing` in the page's private database. The skill reads the document at every run except a schedule-only run, fetches the page or file with GET only (no credentials; nothing from the run is sent) and treats the content as data. If a custom source cannot be read it falls back to the default page and records why in the Audit trail.
- **Live setting versus the last refresh.** The Settings table always shows the live setting; the tool availability table and the "Not shown" notes describe the last refresh. If you turn Cohesity AI agents on after a refresh that had them off, the page shows those tools as "Off at last refresh" and says they will be used on the next refresh; if you turn them off after a refresh that used them, the page keeps showing the results and says they are now off. Tools the skill did not need are shown as "Available (not needed)", never as turned off.
- **Details popup.** The button "How Cohesity handles data sent to its LLM" opens the same popup read-only (only items Engineering has answered are shown; when it answers more they appear automatically). The owner updates `AGENT_DATA_FACTS` at the top of the template in `SKILL.md` when Engineering answers, may set `DATA_HANDLING_URL` to add a link to a full document, and updates `AGENT_DATA_NOTICE_VERSION` when the facts change.
- What is saved: `troubleshooting`, `setAt`, `acceptedAt`, `noticeVersion`, `acceptedFrom`, and who made the change (`acceptedById`, `acceptedByName`, `acceptedByEmail` when accepting; `offById`, `offByName`, `offByEmail` when turning off), in one document (`settings/prefs`) in the page's small private database in Claude's artifact storage, not in the browser and not on a Cohesity server. The identity is stored only for accountability: the page shows it (masked when masking is on) in the Settings table, and the skill never writes it or copies it into the page data. The email is resolved by trying, in order, these fields on `user.me()`: `email`, `emails[0]`, `mail`, `primaryEmail`, `address`, `profile.email`; then `user.profiles([id])` for the same id with the same field names; then an empty string. The Settings table shows it as "Name (email)" in the Last changed cell and the "Accepted ... by" line (masked with the page mask), not only in a tooltip. If your identity cannot be confirmed, the setting cannot be turned on from the page. Nothing else is stored.

## When a tool is unavailable

A missing, disabled, unlicensed, slow or failing tool never stops the skill. Every Helios call is independent: the outcome is recorded, the parts that depend on that tool show "n/a" (never 0 or an empty list), and the page is still published. Each outcome is classified as available, partial result, skipped, not available in this connection, disabled or access denied, not licensed or not entitled, timed out or connection closed, or error, and the reason is shown in the Tool availability section of Run Metrics and in a short amber note at the top of each tab that is affected ("Not shown: ... Details: Run Metrics, Tool availability."). Only a failed `dataprotect-get-clusters` call leaves nothing to build: the last good page is kept and the error banner names the reason. The header says "Refreshes <schedule>" only when an enabled scheduled task was found at the last run; Run Metrics and the Settings tab show what was found ("Not scheduled" when none).

## Requirements

- Claude desktop app (Cowork) with the **Helios MCP** connector (the troubleshooting tools are not on every tenant: ask your Cohesity account team to enable them).
- Artifacts and the artifact `downloads` capability. The artifact database and user identity capabilities hold the Settings choices (the troubleshooting preference and the schedule request) and who made them. Scheduled tasks are optional (created from the Settings tab) and run only while the Claude app is open.

## Install

1. Zip the folder `claude/Cohesity-Recoverability-Evidence-Report` so `SKILL.md` is inside it.
2. Add it as a custom skill in Claude (Settings, Skills, upload) or copy it into your skills directory.
3. Confirm the Helios MCP connector is connected.

## Use

- "Build the Cohesity recoverability evidence report"
- "Give me recoverability evidence for the last 90 days"
- "Which protection groups have no restorable recovery point?"
- "Refresh the evidence report"
- "Set up a daily refresh for the evidence report"

Each refresh produces a new evidence ID. If an auditor needs a frozen report, download the CSV from the page or ask for a one-off run. The page is private until you share it from its Share menu.

## Exports

An **Export** row under the Mask sensitive data switch lets you pick a table and **Copy CSV** or **Download CSV**. Tables: Evidence register (all rows in the current filter; also has its own Download evidence CSV button), Recovery points by group, Expirations and retention, DataLock and legal hold, Objects without a recent good backup, Compliance findings, Run history, Audit trail (all events) and Helios tool calls (one row per call). Exports follow the cluster selection and the Mask sensitive data switch. Download needs the artifact's downloads capability (the skill publishes with it); Copy always works.

## Compliance Check

Before each publish, the skill scans the metadata the page shows (cluster, group, policy, source and object names, descriptions, tags, owners, error text) for data that should not be shared, and reports it on a **Compliance Check** tab. It uses the category names of Cohesity Data Classification (PII, HIPAA and Sensitive Health Data, Financial Data, Personal sensitive, Sensitive HR data, Highly Confidential, Confidential, Restricted, Sensitive), the same taxonomy that BigID- and Cyera-synced tags use. Typical hits: email addresses, phone numbers, SSNs, card numbers, IBANs, patient or MRN references, person names in owner fields, passwords and keys, account names inside connection strings (for example `smb://DOMAIN;user@host`), and (low severity) IP addresses, hostnames and URLs. Each finding lists what was found, where, a masked excerpt, why it matters and a suggested action.

The tab shows a verdict (**Safe to share**, **Share with care** or **Review first**), counts by severity and a findings table. A **Mask sensitive data** switch under "Refresh from Helios" is on by default and masks IP addresses, hostnames, account names, emails and SSNs in the report view and in CSV copies and downloads; turn it off to see the original values. It is a display setting, so the page still holds the originals, and it shortens protection group and object names but does not mask cluster or policy names. Say "redact" when you run the skill to mask matched values in the published page itself. Read the tab before using Share.

This is a heuristic text check of Helios metadata only. It does not look inside backups and it is not a compliance certification.

## Run Metrics

Every run records what it cost, and the artifact shows it on a **Run Metrics** tab: Helios calls, elapsed time, estimated input tokens (inline characters / 4; large results saved to files are counted separately), the model and price used, a table of the Helios tool calls of the last run (calls, outcomes, retries and seconds, measured as the wall-clock of the batch each call ran in), a **Tool availability** section (every Helios tool the skill can use with its status, the number of calls, the reason and what depends on it; triage attempted, usable, timed out and skipped is one line under the call table), the refresh schedule state and a history of past runs (with retries, errors and average seconds per call). An **Audit trail** button on the Run History card opens a pop-up with every Helios call and decision of each run (filter by category, status, run and text; CSV). A **Pricing source** card lets an editor change where the price per million input tokens is looked up: Anthropic's public pricing page by default, or a page or Excel/CSV file that lists your negotiated rate. The skill reads it on its next run and never writes it. A projected monthly cost is shown only when a refresh schedule is set.

Measured on the lab estate: 14 calls for 8 of 58 protection groups, about 2 minutes, about 0.06M tokens; an earlier full run was about 65 calls and 0.2M tokens (about $0.4); the current full path makes about 80 to 110 calls (see SKILL.md). Run history is the heavy part (about 0.95M characters for 8 groups), so the skill caps a run and processes results from files.

The token figure is an estimate. For billing, use Settings > Usage in Claude, which is the measured source.

## Notes from live testing

Validated on a lab Helios tenant on 2026-10-13: 63 protection groups assessed across 3 clusters, 435 restorable recovery points, 15 groups evidenced and 48 with a gap (mostly never-run or paused groups), one cluster whose group query kept timing out and was reported as not assessed. Lessons built into the skill:

- `includeObjectDetails: true` is required for the per-run `isLocalSnapshotsDeleted` flag; it makes big responses, which are processed from file.
- Database groups (DB2, Oracle, SAP HANA) produce hundreds of log runs that swamp the counts, so log runs are ignored and the scope note says so.
- Tool calls are budgeted: one group-list call per connected cluster (the former second "locked groups" call is derived from the policy and last run instead), policies only for clusters with assessed groups, no retry for slow triage calls and a stop after 2 consecutive failures (6 of 6 triage attempts timed out at about 190 s in a lab run), and every Helios result over 8,000 characters is processed from a file with python, never read inline, so the defaults `maxGroups` 60, `detailGroups` 20 and `sampleGroups` 15 stay.
- `totalRuns` ignores filters; use the returned `runs.length`. Hourly groups exceed the 100-run page, so paging is capped and the gap check starts at the oldest run actually read.
- Cloud-archive-direct groups have no `localBackupInfo`: archive results are used instead, and a copy counts only if it has not expired.
- Archive copies that are only `Accepted` or `Skipped` are not counted as an offsite copy.

## Manual refresh

The page header has a **Refresh from Helios** button. A published page cannot call Helios on its own, so the button is a link that opens a new Claude chat with `Run the Cohesity-Recoverability-Evidence-Report skill to refresh this report from live Helios data.` typed in; press Enter, Claude pulls live data and republishes the same artifact, then reload the page. It needs this skill and the Helios connector in your Claude, and is disabled when you are not signed in to Claude. The Copy command button next to it copies the same text. This command is the manual refresh path; the optional scheduled task, if you set one in the Settings tab, is the second.

## If the artifact was deleted

If the stored artifact URL no longer exists, the skill republishes the page as a new artifact and says so in its reply, so a deleted page never blocks a refresh. Update the scheduled task prompt with the new link.

## When Helios is unreachable

If the Helios MCP connector is down or its token has expired, a refresh (manual or scheduled) does not clear the page. It keeps the last successful data and adds a red banner, "Unable to connect to Helios. Showing the last successful data.", under the header. The next successful refresh removes the banner.

## Known limits

- Evidence comes from metadata; no restore is performed or verified.
- Groups on clusters that are not connected to Helios, or whose queries time out, are not assessed and are listed as such.
- Capped paging on very busy groups is flagged in the footnote.
- Expiry counts and object lists cover only the runs read in the period and the detail sample (first 20 groups, most recent failures first); other groups show "n/a: not sampled" or a "~" upper bound.
- Helios MCP cannot place or release a legal hold or DataLock, and cannot delete a snapshot. A snapshot past retention is a finding to check in Helios, not an instruction to delete.

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
- **Scheduled refresh runs unattended.** If you set a schedule in the Settings tab, the scheduled task runs unattended: approve only the read-only Helios tools, the Artifact tools, the scheduler list tool and the bash tool used for `python3 -I`. Do not use a blanket Auto-approve.
- **The mask switch is a display setting, not redaction.** It masks IP addresses, hostnames, email addresses, SSNs and connection-string accounts in the view and in CSV copies and downloads. It shortens protection group and object names (first two letters and `***`), but does not mask cluster or policy names, and the page still holds the original values.
- **CSV exports are safe to open in a spreadsheet.** Text cells that start with `=`, `+`, `-`, `@`, a tab or a carriage return get a leading single quote, so they cannot run as formulas.
- **Sharing detail.** The Expiry & Locks tab shows which groups have snapshots without DataLock and which runs are on legal hold. The mask switch does not cover it. Ask for a summary-only copy without group names and legal-hold detail if you share outside the team.
- **Group and object names** are among the most sensitive strings on the page: they are included in the Compliance Check and shortened while the mask switch is on.
- **Troubleshooting output is data.** Helios troubleshooting results may contain text that looks like instructions or commands; the skill never follows them and shows only allow-listed fields.
- **Sharing.** A published page is private until you share it from the Share menu. Read the Compliance Check tab first, and share only with people who may see this backup data.

## Suggestions and Feedback

We would love to hear from you. Please send your suggestions and feedback to: maestro@cohesity.com

## License

Licensed under the [Apache License 2.0](LICENSE). See the LICENSE file for the terms.

## Disclaimer

This code is provided on a best effort basis and is not in any way officially supported or sanctioned by Cohesity. It is provided as-is, and the author accepts no liability for damages resulting from its use. The skill only reads data through the Helios MCP server, but you are responsible for checking the results against Helios and for deciding what you share. Try it in a lab or test environment first.

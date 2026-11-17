Copyright Cohesity 2026, SPDX-License-Identifier: Apache-2.0

# Recoverability Evidence: Claude Sample Output

Sample output of the Claude skill [`Cohesity-Recoverability-Evidence-Report/SKILL.md`](Cohesity-Recoverability-Evidence-Report/SKILL.md). For install and use, see the [main README](../README.md). The skill now also carries the Recovery Points, Expiry & Locks and Objects at risk tabs (merged from Cohesity-Recovery-Point-Explorer); screenshots of those tabs are in the archived Explorer repository and will be refreshed. Every page has the Cohesity logo top-left and "Data last pulled" with its age top-right. Screenshots are from the published artifact (lab data); hostnames, cluster names and other identifying values are blurred.

## Evidence

![Report tab](../docs/screenshots/recoverability-evidence-report-claude.png)

## Compliance Check

Checks the metadata shown on the page for personal data, secrets and infrastructure details before the page is shared.

![Compliance Check tab](../docs/screenshots/compliance-check-claude.png)

## Run Metrics

Tool calls, elapsed time, estimated tokens and estimated cost for the last run, the last 24 hours and the month to date, plus the full run history, an **Audit trail** pop-up (every Helios call and decision of each run, filterable, with CSV) and an editable **Pricing source** link for the price per million input tokens (default: Anthropic's public pricing page; a custom page or Excel/CSV file is read by the skill on its next run). The current page also has a **landing** at the top of the Evidence tab (a hero ring, tiles with change chips, a verdict bar, short status pills instead of amber notes, and a "Read as text" summary of three sentences that uses the tile numbers), a **Download as PDF** button (one real PDF of every report tab, Settings excluded, built in the browser; needs cdnjs.cloudflare.com), small **trend** sparklines on the tiles (from the third run on), a note listing any figures typed by hand from inline tool output, and a **Settings** tab (the same canonical block in all Cohesity skills: a table of data handling with the Cohesity AI agents on/off switch, a consent popup before Cohesity AI agents can be turned on (off by default; the accepting person's name and email are recorded in the page's private database for accountability, never in the page data) and a read-only details popup), a **Refresh schedule** card (shows what the skill saw in Claude's scheduler at its last run and lets an editor request a time, a time zone, a frequency or turning it off; the skill applies the request at its next run, and it needs Claude open on that computer) below the Settings table (which shows the live setting; Tool availability describes the last refresh and says so when the setting changed since, as "Off at last refresh" or "now off") and a Tool availability section (status, calls, reason and affected parts for every Helios tool);

**Theme and footer.** A single icon button (sun or moon) at the top right of the page header controls the colour theme. The page follows the browser setting on load and switches immediately whenever the browser or system scheme changes; clicking the button flips the theme for the current page view only, and nothing is stored. The button is hidden in print. A small footer at the bottom of every tab reads "Copyright Cohesity 2026, SPDX-License-Identifier: Apache-2.0" and stays on the page when the Helios-unavailable banner is shown.

![Run Metrics tab](../docs/screenshots/run-metrics-claude.png)

The header link **Refresh from Helios** and the **Apply in Claude** link in the Settings schedule card each open a new Claude chat with the command typed in (press Enter to run). They need this skill (and, for Refresh, the Helios connector) in the viewer's Claude and are disabled for viewers who are not signed in; applying a schedule is a schedule-only run with no Helios calls, and it converts the requested time zone to the computer's local time (daylight saving changes can shift the local run time by one hour). While the mask switch is on, protection group and object names are shortened everywhere; cluster and policy names are not.

## Settings

The Settings tab is the same on every Cohesity skill page. It lists what the page reads, where the data goes, and the one switch for the optional Cohesity AI agents (off by default), plus the **Refresh schedule** card.

![Settings tab](../docs/screenshots/settings-claude.png)

## Audit trail

The **Audit trail** button in the Run Metrics tab (Run History card) opens a read-only list of every Helios call and decision of each run, with filters for category, status, run and search, and a CSV download.

![Audit trail pop-up](../docs/screenshots/audit-trail-claude.png)

## How Cohesity handles data sent to its LLM

Before the Cohesity AI agents can be switched on, the page shows this notice. The questions and answers are blurred in this sample; the page shows them in full.

![LLM usage disclaimer](../docs/screenshots/llm-usage-disclaimer-claude.png)

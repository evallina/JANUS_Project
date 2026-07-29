# JANUS

**A daily brief that talks back.**

JANUS turns Claude's scheduled tasks into a morning chief-of-staff. Each day it reads your task ledger, inbox and calendar, then delivers the brief as an interactive HTML page rather than a wall of chat text — one you can work inside all day and hand back at night.

This repo holds one thing: the brief's design template. No personal data lives here.

---

## Why

Two problems with an AI assistant that reports to you daily.

**It drifts.** Left to describe its output in prose, a scheduled task reinvents the format every morning. One day you get a clean checklist, the next a paragraph summarizing what it did. A fixed template removes the choice — the only thing that varies is the day's data.

**It's one-way.** A brief you can only read is a report. A brief you can annotate becomes a conversation: what got done, what slipped, what to move to Thursday. JANUS closes that loop.

And a third, quieter reason: a day's worth of obligations rendered as a dense list is discouraging before you've started. The brief is designed to be *legible* — priorities, rhythm and horizons visible at a glance, so the day reads as a shape rather than a backlog.

## The loop

```
  7:00 AM   Claude gathers → builds the brief → you get an HTML file
     ↓
  All day   You check items off, leave comments, add calendar events
     ↓
  Evening   Export Markdown → send it to yourself
     ↓
  7:00 AM   Claude ingests it: ledger updated, events created, memory learned
```

Each morning the scheduled task:

1. **Reads memory** — a Notion page holding the task ledger across four horizons (Today / This Week / Next Four Weeks / Future) and its learned preferences about you.
2. **Gathers** — a Notion dump page for half-thoughts and updates, an email intake channel, an inbox scan, and your calendar.
3. **Categorizes** — every item gets a priority (P1 due today → P4 weekly reminder), a Personal/Professional split, and a slot matched to your energy through the day.
4. **Builds the brief** — downloads this template and injects the day's data.
5. **Writes back** — refreshes the ledger, logs what it learned, and appends to a run log.

Then you take over. Tick items as you go. Click any item to leave a comment — *move this to Friday*, *waiting on Marta*, *this is actually P1*. Click any day in the calendar to add an event or block out a period. Everything persists locally as you work.

At day's end, **Export Markdown** produces a document carrying your checkmarks, comments and calendar additions. Send it to your intake channel. Tomorrow's run reads it, marks things done, creates the events for real, treats each comment as an instruction, and opens the new brief acknowledging what it processed.

## The brief

- **Linear calendar** in three zooms — one week, four weeks, six months. Deadlines in accent, unconfirmed trips as hatched bands, your own additions dotted.
- **Today**, split Professional / Personal, each item tagged by priority and mapped to when you actually work well.
- **This Week**, **Waiting On** (with staleness nudges), **Brief Summary**, **Long View**, **Questions** — anything the assistant needs you to decide.
- **Day variants** — Monday opens with all four horizons, Friday adds the weekend, Sunday recaps last week against plan.
- Comment fields on every section. Export to Markdown or print to PDF.

## Setup

**1. Host the template.** Fork or copy this repo. Grab the raw URL of `janus_brief_template.html`.

**2. Create the Notion pages.** One memory page (ledger, learned preferences, open questions, run log) and one dump page with `New Entries` / `Processed` headings. Give your Claude connection edit access to both.

**3. Create the scheduled task.** Paste in your instructions — persona, rhythm, gathering order, categorization — plus the delivery contract below.

**4. Test manually** before trusting the schedule. Confirm the HTML renders with today's data and the run log gained a line.

### The delivery contract

Two rules keep it honest and cheap:

```
Download the template — never read, print or fetch it into the conversation.

  curl -sL <RAW_URL>/janus_brief_template.html -o brief.html

Write only the day's data to data.js, then splice:

  python3 -c "
  import re
  t=open('brief.html',encoding='utf-8').read()
  d=open('data.js',encoding='utf-8').read()
  new=re.sub(r'/\* ===== DAILY DATA START ===== \*/.*?/\* ===== DAILY DATA END ===== \*/',
    '/* ===== DAILY DATA START ===== */\n'+d+'\n/* ===== DAILY DATA END ===== */',
    t, count=1, flags=re.S)
  assert new!=t, 'splice failed'
  open('Janus_Daily_<DATE>.html','w',encoding='utf-8').write(new)"
```

Using `curl` matters: the template reaches disk without passing through the model, so each run only generates the small data block instead of reproducing a 250 KB file.

Also require the task to **verify its writes** — re-fetch the Notion page and confirm the run log line exists. Silent write failures reported as successes are the failure mode worth guarding against.

### The data block

The template is a finished artifact. Everything day-specific lives in one `BRIEF` object between the `DAILY DATA` markers; nothing else is ever edited.

```js
const BRIEF = {
  date: "2026-07-28",           // ISO, your timezone
  runNumber: 12,
  quote: { text: "…", author: "…" },
  variant: null,                // or { title, lines: [{ label, text }] }
  calendarEvents: [{ time: "2:00–3:00 PM", title: "…", note: "…" }],
  professional: [{ pri: "P2", tag: "Project", text: "…", when: "morning" }],
  personal:     [{ pri: "P1", tag: "Admin",   text: "…", when: "early afternoon" }],
  week:         [{ pri: "P2", text: "…" }],
  waitingOn:    [{ text: "…", accent: true }],
  summary:  "…",
  longView: "…",
  questions: [{ text: "…" }],
  calendar: {
    marks:     { "2026-07-31": "deadline" },
    tentative: [{ start: "2026-09-12", end: "2026-09-19", label: "…" }],
    activity:  { "2026-07-31": ["Payment due — hard deadline"] }
  },
  footerStatus: "Ledger updated · Run logged."
};
```

Omit `id` fields — they're generated. Empty arrays render gracefully.

## Notes

- **Self-contained.** React is inlined; the file works offline, from anywhere, with no dependencies.
- **Local state.** Checkmarks and comments persist per browser via `localStorage`, keyed to the date. They reach Claude only through the Markdown export — that handoff is deliberate, not a limitation to route around.
- **No calendar write access.** The HTML is a static file. Events you add appear under *Suggested Calendar Additions* in the export, and the next run creates them.
- **Nothing private here.** Your ledger, inbox and calendar stay in your accounts. This repo is a design shell.

## Adapting it

The template is one file — layout, styles and logic together. The accent color is a prop with four presets. The section order lives in the markup; the rhythm labels (`morning`, `early afternoon`, `evening crunch`) are just strings, so retune them to your own energy curve. The four horizons and four priorities are conventions the instructions enforce, not code — change them in both places and the brief follows.

---

Built with [Claude](https://claude.ai).

# JANUS

**A daily brief that talks back.**

JANUS turns a Claude scheduled task into a morning chief-of-staff. Each day it reads your task ledger, inbox and calendar, then delivers the brief as an interactive HTML page — one you work inside all day and hand back at night.

This repo holds one thing: the brief's design template. No personal data lives here.

## The loop

```
6:00 AM   Claude gathers → builds the brief → you get an HTML file
All day   Check items off, leave comments, add tasks and calendar events
Evening   SEND TO JANUS — or Export Markdown and send it yourself
6:00 AM   Claude ingests it: ledger updated, events created, memory learned
```

A fixed template keeps the format from drifting run to run — only the day's data changes. And because the brief is annotatable, it's a conversation rather than a report: a comment like *move this to Friday* becomes an instruction for tomorrow's run.

## The brief

- **Linear calendar** in three zooms (1W / 4W / 6M) — deadlines in accent, tentative trips hatched, your additions dotted.
- **Today**, split Professional / Personal, each item tagged P1–P4 and mapped to your energy through the day.
- **This Week**, **Waiting On**, **Brief Summary**, **Long View**, **Questions**, and a start/end mood check-in.
- Comment fields on every section; add your own tasks inline; day variants for Monday, Friday and Sunday.
- **SEND TO JANUS** posts the day back through a webhook; **Export Markdown** and **PDF** work without one.

## Setup

1. **Host the template.** Fork or copy this repo; grab the raw URL of `janus_brief_template.html`.
2. **Create the Notion pages.** One memory page (ledger, learned preferences, run log) and one dump page. Give your Claude connection edit access to both.
3. **Optional webhook.** A Make/Zapier catch hook that forwards the POST to your intake channel. The daily run injects its URL as `webhookUrl`; left empty, the button stays hidden.
4. **Create the scheduled task** with your instructions plus the delivery contract below, and run it manually once before trusting the schedule.

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

`curl` matters: the template reaches disk without passing through the model, so each run generates only the small data block instead of reproducing a 250 KB file. Also require the task to **verify its writes** — re-fetch the Notion page and confirm the run log line exists.

### The data block

Everything day-specific lives in one `BRIEF` object between the `DAILY DATA` markers; nothing else is ever edited.

```js
const BRIEF = {
  date: "2026-09-17",           // ISO, your timezone
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
    marks:     { "2026-09-20": "deadline" },
    tentative: [{ start: "2026-10-12", end: "2026-10-19", label: "…" }],
    activity:  { "2026-09-20": ["Payment due — hard deadline"] }
  },
  moodHistory: [{ date: "2026-09-16", start: 6, end: 8 }],
  webhookUrl: "",               // Make/Zapier hook; empty hides SEND TO JANUS
  footerStatus: "Ledger updated · Run logged."
};
```

Omit `id` fields — they're generated. Empty arrays render gracefully.

### The send-back

**SEND TO JANUS** POSTs three form fields to `webhookUrl`: `date`, `send` (a per-day counter, so a later send supersedes an earlier one) and `markdown` (the full export, carrying your checkmarks, comments, added tasks and calendar additions). The request uses `mode: 'no-cors'`, so a resolved request counts as success. The next run treats the received markdown exactly like an emailed export.

## Notes

- **Self-contained.** React is inlined; the file works offline with no dependencies.
- **Local state.** Checkmarks and comments persist per browser via `localStorage`, keyed to the date. They reach Claude only through SEND TO JANUS or the Markdown export — that handoff is deliberate.
- **No calendar write access.** Events you add appear under *Suggested Calendar Additions*; the next run creates them for real.
- **Adaptable.** The accent color is a prop with four presets; the rhythm labels, horizons and priorities are conventions the instructions enforce, not code.

---

Built with [Claude](https://claude.ai).

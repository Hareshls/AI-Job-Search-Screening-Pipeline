# AI Job-Search Screening Pipeline

An automated **AI-powered job-search screening pipeline** that watches for job alert emails, scores each posting against my resume using AI, logs every result, and only notifies me when a job is genuinely worth applying to.

## The Problem

Job alert emails from Naukri, Indeed, and Glassdoor are noisy.

Keyword-based filters can catch obvious mismatches, but they miss subtler ones — like a posting that lists my skills but implicitly requires 5+ years of experience through seniority language or role expectations.

I wanted something that could **actually reason about job fit, not just match keywords**.

## What It Does

1. **Watches Gmail** for job alert emails filtered to a dedicated `Job-Alerts` label.
2. **Scores each job with AI** using Google Gemini. A structured prompt sends the job posting alongside my resume and asks for:

   * Fit score (0–100)
   * Inferred years of experience required
   * Written reasoning
3. **Cleans and parses the AI response** by extracting the JSON payload from the model's output and removing markdown formatting when necessary.
4. **Logs every job** — company, role, score, years required, and reasoning — into Google Sheets as a complete audit trail.
5. **Filters and notifies** — only jobs scoring **80+** trigger an email notification.

## Architecture

```text
Gmail
  ↓
Job Alert Email
  ↓
Google Gemini
  ↓
AI Fit Score + Reasoning
  ↓
Text Parser
  ↓
JSON Parser
  ↓
Google Sheets
  ↓
Filter: Score >= 80
  ↓
Gmail Notification
```

## Tech Stack

* **Make.com** — workflow automation
* **Google Gemini** — AI-powered job evaluation
* **Gmail** — job alert monitoring and notifications
* **Google Sheets** — job tracking and audit log
* **Regex / JSON parsing** — structured AI response extraction

Currently, I'm also rebuilding the same logic using **n8n locally** as a learning exercise and to understand the differences between automation platforms.

## Why These Tools?

### Make.com over Zapier

I chose Make.com because Zapier's free tier wasn't suitable for a workflow with 6+ steps.

Make allowed me to build the complete workflow while keeping the initial project within a free-tier setup.

### Make.com over n8n initially

I started with Make.com because it required less infrastructure setup.

I'm now rebuilding the workflow with self-hosted n8n to learn another automation platform and understand the trade-offs between hosted and self-hosted automation.

### Google Gemini

Gemini's Flash models provided a practical way to experiment with AI-powered classification and reasoning while staying within available free API quotas during development.

## Notable Bugs & Lessons

Building the workflow wasn't completely smooth — which was actually one of the most useful parts.

### 1. The `"null"` Bug

Make's Text Parser `Replace` action inserted the literal string `"null"` when the replacement field was left empty.

Instead of trying to work around the replacement behavior, I switched to a regex `Match Pattern` approach to directly extract the JSON block.

### 2. The Silent Capture Group Requirement

My regex technically matched the data, but the module returned nothing.

The issue was that Make's regex matcher only exposed the data inside **parenthesized capture groups**.

Wrapping the pattern in `(...)` fixed it.

A tiny issue, but a good reminder that automation platforms can have their own quirks that aren't obvious until something silently fails.

### 3. API Rate Limits

During testing, I hit Gemini's free-tier request limits because I was running the workflow repeatedly while debugging.

The solution was to slow down testing and work within the available API quota instead of repeatedly triggering the entire workflow.

## Sample Output

A real example from the pipeline:

> **Company:** Siemens
> **Role:** Software Engineer Associate - CORE B
> **Score:** 85
> **Years Required:** 0–2 years
>
> **Reasoning:** The candidate is an entry-level engineer graduating in 2026 with 4–5 months of frontend internship experience and strong full-stack (React, Node, Python) skills. The associate position aligns well with the candidate's entry-level background.

## Results

The automation has already:

* Identified strong-fit roles that I would have applied to anyway
* Filtered out senior positions that weren't realistic matches
* Reduced the amount of time spent manually checking job alerts
* Created a searchable history of evaluated jobs and AI reasoning

One of the first strong matches scored **88/100**.

## What I Learned

The biggest takeaway wasn't just learning how to connect Gmail, Gemini, and Google Sheets.

It was learning how to build and debug an **end-to-end AI automation system**.

The workflow looks simple from the outside:

```text
Input → AI → Decision → Output
```

But actually building it means dealing with:

* API limits
* Data formatting
* Regex
* JSON parsing
* Empty values
* Error handling
* Workflow logic
* Automation platform quirks

Those small problems are where a lot of the real learning happened.

## Status

🟢 **Live and running**

The workflow checks for new job alerts every 15 minutes and sends notifications only for jobs that meet the configured score threshold.

## Future Improvements

* [ ] Add duplicate-job detection
* [ ] Improve AI scoring consistency
* [ ] Add application-status tracking
* [ ] Track score distribution and trends
* [ ] Add more job sources
* [ ] Rebuild and compare the workflow in n8n
* [ ] Add better error handling and retry logic
* [ ] Create a dashboard for job-search analytics

## Author

**Haresh LS**

* GitHub: https://github.com/Hareshls
* LinkedIn: https://linkedin.com/in/hareshls

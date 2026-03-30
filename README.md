# 🔍 The Lens — AI Newsletter Automation

**Focused View** | Dispatched by [FutureFrames AI](https://youtube.com/@FutureFramesAI-V)

An automated AI-powered newsletter system built entirely in n8n — **zero AI agents**, pure node-based automation. Fetches real-time news from Google News, curates it with GPT-4o-mini, and delivers a branded newsletter to Slack, Email, and Google Sheets simultaneously.

---

## What it does

Type `/lens healthcare AI` in Slack → 20 seconds later, a curated newsletter appears with:
- **Top Story** — the most important article with a detailed summary
- **Quick Bites** — 3 additional stories with one-line summaries
- **Through the Lens** — editorial analysis identifying emerging trends

The same newsletter is simultaneously:
- Posted to your **Slack** channel
- Emailed to all subscribers via **Gmail**
- Logged to **Google Sheets** for analytics

It also runs automatically every day at **7:30 AM** with rotating topics (AI News, Healthcare AI, Startups, Policy, etc.)

---

## Architecture

```
┌─ Slack /lens command ──┐
│                        ├─→ Google News RSS (100 articles)
└─ Schedule 7:30 AM ────┘        │
                                  ▼
                         Deduplicate & Normalize (top 20)
                                  │
                                  ▼
                         GPT-4o-mini (semantic filter + write)
                                  │
                                  ▼
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
              Slack Post    Gmail Send    Sheets Log
```

**Key design decisions:**
- **Google News RSS as the sole source** — Google's search algorithm provides the best relevance for any topic. Free, unlimited, no API key needed.
- **No keyword matching or synonym dictionaries** — the LLM handles all semantic understanding. Works for ANY topic.
- **No AI agents** — single-prompt LLM call. Predictable, fast, cheap (~$0.03/edition).
- **Deduplication by title** (not URL) — Google News URLs share long prefixes that cause false duplicates.

---

## Nodes (14 active)

| # | Node | Type | Purpose |
|---|------|------|---------|
| 1 | Webhook — Slack /lens | Webhook | Receives Slack slash command |
| 2 | Instant Ack | Respond to Webhook | Replies to Slack within 3 seconds |
| 3 | Set — From Slack | Set | Extracts topic, username, channel |
| 4 | Schedule — 7:30 AM Daily | Schedule Trigger | Auto-fires every morning |
| 5 | Daily Topic Rotation | Code | Picks topic by day of week |
| 6 | Google News RSS | RSS Feed Read | Fetches 100 articles for topic |
| 7 | Deduplicate and Rank | Code | Normalizes fields, removes dupes, keeps top 20 |
| 8 | IF — Has Articles? | IF | Error check before LLM |
| 9 | Basic LLM Chain | LLM Chain | GPT-4o-mini writes the newsletter |
| 10 | Parse JSON | Code | Parses LLM response, merges metadata |
| 11 | IF — Valid JSON? | IF | Validates LLM output |
| 12 | Slack — Post Newsletter | Slack | Posts to channel |
| 13 | Google Sheets — Get Subscribers | Google Sheets | Reads subscriber list |
| 14 | Filter Active Subscribers | Filter | Keeps only active=TRUE |
| 15 | Gmail — Send Newsletter | Gmail | Sends HTML email to each subscriber |
| 16 | Google Sheets — Log Edition | Google Sheets | Logs date, topic, top story |
| 17 | Slack — Error Alert | Slack | Notifies #lens-errors on failure |

---

## Setup (15 minutes)

### Prerequisites
- Self-hosted n8n (e.g. Hostinger VPS)
- Slack workspace + app with slash command
- OpenAI API key ($5 credit)
- Gmail account with OAuth2
- Google Sheets OAuth2

### Step 1: Import the workflow
1. Download `the-lens-workflow.json`
2. In n8n → Workflows → Import from File
3. All nodes appear pre-wired

### Step 2: Configure credentials (4 needed)

| Credential | Nodes that use it |
|-----------|-------------------|
| Slack API (Access Token) | Slack Post, Slack Error |
| OpenAI API | Basic LLM Chain (Chat Model sub-node) |
| Google Sheets OAuth2 | Get Subscribers, Log Edition |
| Gmail OAuth2 | Gmail Send Newsletter |

### Step 3: Replace placeholder values

| Placeholder | Where | Replace with |
|------------|-------|-------------|
| `YOUR_DEFAULT_SLACK_CHANNEL_ID` | Daily Topic Rotation code | Your #the-lens channel ID |
| `YOUR_GOOGLE_SHEET_ID` | Get Subscribers + Log Edition nodes | Sheet ID from your Google Sheets URL |
| Email HTML template | Gmail node Message field | Contents of `the-lens-gmail-template.html` |

### Step 4: Create Google Sheet
- Spreadsheet name: "The Lens — Subscribers"
- Tab 1 "Subscribers": columns `name`, `email`, `subscribed_date`, `active`, `source`
- Tab 2 "Log": columns `date`, `topic`, `section_label`, `triggered_by`, `source_count`, `top_story`

### Step 5: Connect Slack slash command
1. Copy the webhook Production URL from the Webhook node
2. In api.slack.com → your app → Slash Commands → set `/lens` Request URL
3. Activate the workflow in n8n

### Step 6: Test
```
/lens artificial intelligence
```

---

## Weekly topic schedule (daily auto-trigger)

| Day | Topic | Section Label |
|-----|-------|--------------|
| Monday | AI News & Breakthroughs | Monday Briefing |
| Tuesday | Healthcare AI & Hospital Tech | Healthcare Focus |
| Wednesday | AI Startups & Funding | Startup Watch |
| Thursday | AI Regulation & Policy | Policy Desk |
| Friday | AI News & Breakthroughs | Friday Drops |
| Saturday | Healthcare AI Deep Dive | Saturday Deep Dive |
| Sunday | Week in AI Roundup | Weekly Recap |

---

## Cost

| Component | Monthly cost |
|-----------|-------------|
| OpenAI API (gpt-4o-mini, ~30 editions) | ~$1 |
| Google News RSS | Free |
| Gmail sending | Free |
| Google Sheets | Free |
| Slack | Free |
| n8n (self-hosted on existing VPS) | $0 |
| **Total** | **~$1/month** |

---

## Files in this repo

| File | Purpose |
|------|---------|
| `the-lens-workflow.json` | Complete n8n workflow — import directly |
| `the-lens-gmail-template.html` | Branded HTML email template |
| `the-lens-node-guide.md` | Node-by-node setup instructions |
| `the-lens-subscribe-page.html` | Public subscription landing page |
| `llm-prompt.txt` | The LLM prompt used in Basic LLM Chain |
| `slack-message-template.txt` | Slack message format |

---

## Brand

- **Newsletter:** The Lens
- **Tagline:** Focused View
- **Parent brand:** FutureFrames AI
- **Colors:** Golden palette (#d4a94e, #e8c97a, #2c2418, #f3efe8)
- **YouTube:** [youtube.com/@FutureFramesAI-V](https://youtube.com/@FutureFramesAI-V)

---

## What makes this competition-worthy

1. **Zero AI agents** — pure n8n node automation
2. **Works for ANY topic** — semantic filtering by the LLM, no hardcoded keywords
3. **4 simultaneous outputs** — Slack, Email, Google Sheets, Error alerts
4. **Dual triggers** — on-demand via Slack + automated daily schedule
5. **Production-ready** — error handling, deduplication, input validation
6. **$1/month total cost** — runs on free-tier APIs
7. **Beautiful branded email** — professional HTML template with sponsor slot
8. **Built-in monetization** — affiliate links in email footer

---

## Future improvements

- [ ] WordPress blog auto-publishing (needs self-hosted WordPress)
- [ ] Full article text fetching for richer summaries
- [ ] Fact-check verification pass (2nd LLM)
- [ ] Subscriber signup workflow (Slack command + web form)
- [ ] Multiple news sources (add back NewsAPI, GNews when APIs improve)
- [ ] Upgrade to GPT-4o for higher quality output

---

*The Lens — Focused View. Built with n8n. Dispatched by FutureFrames AI.*

## Author

Viswaath Ganesan 
- Healthcare AI & Medical Imaging Product Leader
- Exploring Generative AI and Applied AI Workflows.

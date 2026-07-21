# Demo Script — AI Inbox Manager

Everything you need to record the Loom and to explain the system on a sales call.

---

## Part 1 — What you actually built

Three moving pieces:

| Piece | What it does | Why a buyer cares |
|---|---|---|
| **Knowledge base → Qdrant** | Their policy docs get chunked, embedded, and stored in a self-hosted vector DB | Replies quote *their* policies, not generic AI filler. Data never leaves their server. |
| **Triage** | Every email gets category, urgency, sentiment, one-line summary, confidence | The inbox sorts itself. Urgent things surface in minutes, not hours. |
| **Draft generation** | Answerable emails get a reply drafted from the KB, with `[CHECK: ...]` where a human must verify | 90% of the typing disappears. Nothing is sent without a human. |

**The one-sentence pitch:** *It reads every email, tells you what matters, and writes the reply — but you press send.*

**The technical differentiator:** self-hosted n8n + Qdrant. No third-party SaaS sees their customer email. That's the answer to "is this safe?" and it's why this beats a Zapier-based competitor.

---

## Part 2 — The 90-second Loom

### Pre-flight (do this before you hit record)

1. `DEMO_SAMPLE_SIZE=0` in `.env` → full 24-email run, then `docker compose up -d n8n`
2. Re-run **KB Ingest** once so sources are fresh
3. Check quota is healthy — record early in the day (Gemini's free tier resets midnight Pacific)
4. Do a full practice run so you're not narrating over a rate-limit error
5. Close Slack, email, notifications. Browser at 100% zoom.

### The script

**0:00–0:10 — The problem (talking head or inbox screenshot)**

> "If your business inbox gets 100+ emails a day, you're spending two hours sorting before you answer anything. Here's what I build to fix that."

**0:10–0:25 — Show the raw inbox**

Show `seed/emails/inbox.json` or a Gmail inbox. Scroll it.

> "Twenty-four emails. Sales inquiries, angry support tickets, billing disputes, spam, newsletters — all mixed together. Watch what happens."

**0:25–0:45 — Run it live**

Open n8n, click **Execute workflow**. Let the nodes light up green as items flow through.

> "This is n8n, running on their own server. Every email gets classified, routed, and — if it needs a reply — matched against the company's own policy documents."

Don't narrate every node. Let the visual do the work.

**0:45–1:05 — The triage result**

Open `output/results.json` or the results table.

> "Twenty-four emails triaged. Eight flagged urgent — and look at *which* ones."

Point at specific rows:
- `m002` "CANNOT LOG IN — 3rd email" → high urgency (repeat contact, angry)
- `m020` "you charged me after I cancelled??" → high urgency (billing dispute)
- `m016` "Trial ending — can we extend?" → **high urgency, and it's a sales email**

> "That last one is the money shot. It flagged an expiring trial as urgent — not because someone's angry, but because that's revenue about to walk out the door. Meanwhile the newsletter and the spam got labelled and left alone."

**1:05–1:25 — The draft (the most important 20 seconds)**

Open the draft for `m006` (webhook) or `m001` (pricing).

> "And here's the reply it wrote."

Read one line out loud — pick a specific fact:

> "'Endpoints failing 5+ times in 24 hours are automatically paused.' That's not the AI guessing. That came out of their own support docs, which are indexed in a vector database on their own server."

Then point at a `[CHECK: ...]` marker:

> "And where it *doesn't* know something, it says so instead of making it up. A human reviews, fills that in, and hits send. The AI never emails your customers on its own."

**1:25–1:35 — Close**

> "Two hours of inbox triage, down to about fifteen minutes of review. Runs on your infrastructure, your data stays yours. Message me and I'll set this up for your inbox."

---

## Part 3 — The killer follow-up clip (optional, 20 seconds)

This is the single most convincing thing you can show, and almost no competitor does it. Record it as a second short video or bonus at the end:

1. Open `seed/knowledge-base/billing-refunds.md` on camera
2. Change the refund window from 30 days to 14 days. Save.
3. Re-run **KB Ingest** (5 seconds)
4. Re-run the demo on the billing email
5. The new draft now says 14 days

> "Change a policy document, and every future reply reflects it. No prompt engineering, no retraining, no calling me. You edit a Google Doc, the AI follows it."

That single loop is what justifies the Advanced tier — and it's what turns a one-off gig into a retainer.

---

## Part 4 — Questions you'll get, and the answers

**"Will it email my customers something wrong?"**
> It never sends. Everything lands in your drafts folder. You approve every message. Auto-send is available later, per-category, once you trust it.

**"Where does my data go?"**
> n8n and the vector database run on your own server or VPS. Only the email text hits the AI model's API — the same trust boundary as using ChatGPT. Nothing is stored by a third party.

**"How accurate is it?"**
> On my 24-email test set, categories and urgency were correct across the board. It tags a confidence score on every classification, so you can route anything low-confidence straight to a human.

**"What does it cost to run?"**
> A few dollars a month in API costs for a normal inbox. It runs on a free API tier for a pilot. Hosting is whatever VPS you already have, or ~$6/month.

**"What if it doesn't know the answer?"**
> It writes `[CHECK: ...]` in that spot rather than inventing something. You'll see exactly what it wasn't sure about.

**"How long to set up?"**
> Two to three days. I need access to your inbox and whatever policy or FAQ docs you already have.

---

## Part 5 — Recording tips

- **Screen record at 1920×1080**, browser zoom 100%
- **Talk over a real run.** Never a slideshow — the live execution is the proof
- **Cut the waiting.** Speed up or jump-cut the 9-minute run to a few seconds
- **First 5 seconds decide everything.** Lead with the problem, not with "hi, I'm Abdullah"
- **Show one real draft in full.** Buyers skim the rest; they read the draft
- **End with a specific CTA.** "Message me your inbox volume and I'll tell you what it'd save you."

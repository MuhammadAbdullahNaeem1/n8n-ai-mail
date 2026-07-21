# Project Tour — how this thing actually works

Read this with the folder open next to you. Every file, what it does, and how data moves through it.

---

## The folder, file by file

```
n8n-ai-inbox-manager/
├── docker-compose.yml      ← the 3 servers (n8n, Postgres, Qdrant)
├── .env                    ← your API key + model names + settings   [SECRET]
├── .env.example            ← the same file with blanks, safe to ship to buyers
│
├── seed/
│   ├── emails/inbox.json           ← 24 fake emails (the demo input)
│   └── knowledge-base/*.md         ← 4 policy docs (the "company brain")
│
├── workflows/
│   ├── 01-inbox-manager-mock.json  ← THE PRODUCT (20 nodes)
│   └── 02-kb-ingest.json           ← loads the policy docs into Qdrant
│
├── output/results.json     ← what the last run produced
└── docs/                   ← demo script, this tour
```

**If you only understand two files, make it these:** `workflows/01-inbox-manager-mock.json` is the product. `seed/knowledge-base/*.md` is what makes its answers specific to a client.

---

## The three servers

`docker-compose.yml` starts three containers:

| Container | Port | Job |
|---|---|---|
| **n8n** | 5678 | Runs the workflows. This is the UI you click. |
| **Postgres** | — | Stores workflows and execution history. You never touch it directly. |
| **Qdrant** | 6333 | Vector database — holds the searchable policy docs. |

Everything else (Gemini) is an API call out to the internet.

---

## Workflow 2 first: KB Ingest (run this once)

Open it in n8n. Five steps:

1. **Read KB Docs** — reads `seed/knowledge-base/*.md`
2. **Extract Text** — turns files into plain text
3. **Chunk Docs** — splits each doc at `##` headings → **24 chunks**
4. **Embed Chunks** — sends all 24 to Gemini, gets back 24 vectors (3072 numbers each)
5. **Upsert Points** — stores them in Qdrant with the source filename attached

**In plain English:** it converts your policy documents into something searchable *by meaning* rather than by keyword. So an email asking "can I get my money back?" will find the refund policy even though the word "refund" never appears in the email.

You re-run this **only when the policy docs change**. It costs 1 API call.

---

## Workflow 1: the main pipeline (20 nodes)

The flow, in order:

```
Run Demo → Read Seed Inbox → Parse Inbox JSON → Split Emails
    → Build Classify Request → LLM: Classify → Parse Classification
        → Needs Reply? ──true──→ Build Embed Request → Embed Query
        │                        → Search KB → Build Draft Request
        │                        → LLM: Draft Reply → Parse Draft ──┐
        └──false───────────────────────────────────────────────────┤
                                                                    ↓
                                          Merge → Format Results → Write file
                                                              └──→ Slack (if urgent)
```

### What each stage does

**Split Emails** — turns the JSON array into 24 separate items, so every following node runs once per email. This is also where `DEMO_SAMPLE_SIZE` cuts the list down to 6 for cheap testing.

**Build Classify Request** — writes the prompt. Open this node to see the actual rules, e.g. *"urgency=high when: outage, imminent deadline, refund dispute, legal threat, charge after cancellation, expiring trial with buying intent, or an angry repeat contact."* **This one node is where you tune behaviour for a client.**

**LLM: Classify** — the API call. Throttled to one request per 6.5s to respect free-tier limits.

**Parse Classification** — reads the model's JSON and *validates* it. If the model returns garbage or an unknown category, this falls back to safe defaults instead of crashing. That's why a bad response never kills a run.

**Needs Reply?** — the fork. Newsletters, notifications, spam, and cold outreach exit here — labelled but no draft written. Everything else continues.

**Embed Query → Search KB** — turns the email into a vector, then asks Qdrant "which policy chunks are closest in meaning to this?" Returns the top matches.

**Build Draft Request** — assembles the drafting prompt: the retrieved policy chunks + the email + the classification. The key instruction lives here: *"Use ONLY facts found in the KNOWLEDGE BASE CONTEXT... If the answer is not in the context, write [CHECK: what a human must verify]."*

**LLM: Draft Reply** — writes the reply. Throttled to one per 12s.

**Merge → Format Results** — recombines both branches, builds the summary (totals, category counts, urgent list) and writes `output/results.json`.

**Slack Alert** — fires only if urgent emails exist *and* a webhook is configured. Yours is empty, so it's skipped.

---

## The RAG loop — the part worth understanding

This is what separates your gig from "ChatGPT writes emails":

```
policy docs → chunks → vectors → Qdrant
                                    ↓
incoming email → vector → search ───┘ → top matching chunks
                                            ↓
                            chunks + email → LLM → grounded reply
```

**Proof it works:** the webhook reply cited *"endpoints failing 5+ times in 24 hours are automatically paused."* That sentence exists in `seed/knowledge-base/support-policies.md`. The model didn't know it — it was retrieved and handed over.

**The consequence you sell:** change the doc, and every future reply changes. No retraining, no code edits, no calling you.

---

## Where to look when something breaks

| Symptom | Where to look |
|---|---|
| Want to see what one email did | n8n → **Executions** → click the run → click any node |
| Drafts are empty | Check `draft_status` / `draft_error` in `results.json` — usually API quota |
| Replies cite wrong policies | Re-run KB Ingest; check Qdrant at http://localhost:6333/dashboard |
| Wrong category or urgency | Edit the prompt in **Build Classify Request** |
| Changed `.env` and nothing happened | `docker compose up -d n8n` — n8n only reads env at startup |

---

## Try these three things (fastest way to understand it)

1. **Open `seed/emails/inbox.json`, read `m016`.** It's a polite email about a trial ending. Now look at its result in `results.json` — flagged **high urgency**. Nothing in the text screams urgent; the model inferred that expiring trials are revenue at risk. That's the intelligence you're selling.

2. **Open `seed/knowledge-base/billing-refunds.md`**, change the refund window, re-run KB Ingest, re-run the demo. Watch the billing draft change. That's the retainer pitch, and you can do it in 60 seconds on a sales call.

3. **Open the "Build Classify Request" node in n8n** and read the prompt. Every category and urgency rule is right there in plain English. When a client says "flag anything mentioning a lawsuit," you add one line here — that's the whole customization.

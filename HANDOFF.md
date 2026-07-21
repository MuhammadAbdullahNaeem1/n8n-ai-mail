# Project handoff — n8n AI Inbox Manager (Upwork gig product)

Read this first. It is the complete state of the project as of 2026-07-20,
written so a fresh Claude Code session (Windows or WSL) can continue without
the original conversation.

## What this is
A sellable Upwork catalog gig: "AI Inbox Manager" built on n8n. Pipeline:
incoming email → LLM classifies (intent/urgency/sentiment, structured JSON) →
route by category → RAG over a small knowledge base → draft reply (NEVER
auto-send — drafts only, human hits send; this is the core selling point) →
label + urgent-Slack-alert + log every decision.

Seller: Abdullah (abdullahnifra@gmail.com), existing gig at $70/100/130 —
new gig planned at ~$120/250/450 (tiers must differ ~2x).

## Current state — what exists in this folder
- `docker-compose.yml` — n8n + Postgres + Qdrant (KEEP: it's the buyer
  deliverable, not the dev path)
- `.env` — FILLED with working secrets. Gemini API key is in here
  (free tier). Models tested and working:
  - classify: `gemini-3.1-flash-lite`
  - draft: `gemini-3-flash-preview`
  - embeddings: `gemini-embedding-001` (3072-dim)
  - base URL: `https://generativelanguage.googleapis.com/v1beta/openai`
  - NOTE: key was pasted in chat; advise rotating it before gig delivery.
- `.env.example`, `.gitignore`
- `seed/inbox.json` — 24 realistic fake emails across 8 categories
  (sales_inquiry, support_request, billing_question, urgent_escalation,
  spam_promotional, newsletter, partnership_outreach, personal)
- `seed/kb/*.md` — knowledge base docs (~14 chunks): product FAQ, pricing,
  refund policy, support playbook/tone guide
- `workflows/01-inbox-manager-mock.json` — main workflow, mock-trigger
  variant (reads seed/inbox.json instead of Gmail). LLM nodes have
  retryOnFail/maxTries=3/waitBetweenTries=5s for free-tier 429s.
- `workflows/02-kb-ingest.json` — KB → chunk → embed → Qdrant upsert
- `RECOVERY.md` — WSL ext4 corruption repair steps (the reason work stopped)

## What happened / why work stopped
WSL2 Ubuntu ext4.vhdx corrupted mid-session (EROFS read-only fs, sudo I/O
error). Shell became unusable. Two paths forward were identified:
1. Repair WSL per RECOVERY.md (e2fsck via helper Debian distro), or
2. Go Windows-native (decided this is viable — see below).

## Decision: dev can be Docker-free and Windows-native
- Run n8n natively: `npx n8n` (SQLite backend, UI at http://localhost:5678)
- Replace Qdrant FOR DEV/DEMO with cosine-similarity search in an n8n Code
  node (KB is only ~14 chunks; store embeddings in a JSON file under
  `seed/` or workflow static data). Keep Qdrant workflow + compose file in
  repo as the "scale" upgrade path for buyers.
- This means workflows 01/02 need a small edit: swap the two Qdrant HTTP
  nodes (search in 01, upsert in 02) for Code-node vector search / a
  write-embeddings-to-file step. Everything else unchanged.

## Task list (原 plan, tasks 1-4 done)
1. [done] Scaffold (compose, env, seed inbox)
2. [done] KB seed docs
3. [done] Core workflow JSON, mock-trigger variant
4. [done] KB ingestion workflow
5. [in progress] Stand up n8n and test end-to-end on the 24 seed emails ←NEXT
   - checkpoint with Abdullah after first full run, before building more
6. [pending] Gmail-trigger variant (OAuth via user's Gmail; drafts only)
7. [pending] Packaging: tier mapping (Starter $120 classify+label+log /
   Standard $250 +drafts+Slack / Advanced $450 +RAG KB+analytics),
   client setup guide, 90-sec Loom demo script, gig title/description copy

## Immediate next steps for the new session
1. Verify shell works; check `node --version` (need >=18; WSL had v24 via nvm)
2. If Windows-native: `cd D:\Work\Upwork\Projects\n8n-ai-inbox-manager`
3. Edit workflows to Code-node vector search (drop Qdrant dep for dev)
4. `npx n8n` → import both workflows → run 02 (ingest) → run 01 (main)
5. Inspect output: classification accuracy across 24 emails, draft quality,
   urgent alerts, decision log → show Abdullah the results table
6. Then tasks 6 and 7

## Working-style notes (from the original session)
- Abdullah prefers being told clearly what to do next, one step at a time
- Free-tier only: no paid API keys. Gemini free tier is the LLM.
- Files/deliverables belong on D:\Work (not Linux home)
- Rate limits: classify and draft use DIFFERENT models on purpose
  (per-model free-tier quotas); keep the retry settings on LLM nodes

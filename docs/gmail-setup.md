# Gmail Setup — connecting workflow 03 to a real inbox

Workflow **"AI Inbox Manager — Gmail (Live Inbox)"** is imported and ready. It needs Google OAuth credentials, which only you can create.

---

## Safety model (read this first)

The workflow is deliberately constrained:

| It does | It never does |
|---|---|
| Reads messages matching `GMAIL_QUERY` | Send email |
| Applies labels | Delete or archive anything |
| Creates **drafts** in the reply thread | Touch mail outside the query |
| Posts urgent alerts to Slack (if configured) | Act on more than `GMAIL_FETCH_LIMIT` per run |

The polling trigger ships **disabled**. Nothing runs unattended until you enable it.

Default scope is `is:unread newer_than:1d`, capped at 5 messages. On a real personal inbox, the safest setting is label-based opt-in — see step 5.

---

## 1. Create a Google Cloud project

1. Go to https://console.cloud.google.com/
2. Create a project (any name, e.g. `n8n-inbox`)

## 2. Enable the Gmail API

**APIs & Services → Library** → search "Gmail API" → **Enable**

## 3. Configure the consent screen

**APIs & Services → OAuth consent screen**

- User type: **External**
- App name: anything (`n8n Inbox Manager`)
- Support email + developer email: your address
- **Scopes:** skip — n8n requests what it needs
- **Test users:** add `abdullahnifra@gmail.com` ← required, or auth will fail
- Publishing status: leave as **Testing**

> Testing mode tokens expire every 7 days. Fine for demos; for a client you'd verify the app or use a Google Workspace internal app.

## 4. Create the OAuth client

**APIs & Services → Credentials → Create Credentials → OAuth client ID**

- Application type: **Web application**
- **Authorized redirect URI** — must match exactly:
  ```
  http://localhost:5678/rest/oauth2-credential/callback
  ```
- Copy the **Client ID** and **Client secret**

## 5. Connect it in n8n

1. Open http://localhost:5678 → **Credentials → Add credential → Gmail OAuth2 API**
2. Paste Client ID and Client secret
3. Click **Connect my account** → pick your Google account → approve
4. Name it `Gmail account` and save

Then open the workflow and, on each of the three Gmail nodes (**Fetch Messages**, **Apply Label**, **Create Draft**), select that credential from the dropdown.

### Choose your labels

On **Gmail: Apply Label**, pick which label(s) to apply — the field is intentionally empty. Create labels in Gmail first if you want per-category ones (`AI/Sales`, `AI/Support`, `AI/Billing`).

### Recommended: label-based opt-in

Instead of letting it read all recent unread mail, control it explicitly:

1. In Gmail, create a label called `ai-triage`
2. In `.env`, set:
   ```
   GMAIL_QUERY=label:ai-triage
   ```
3. `docker compose up -d n8n`

Now the workflow only ever sees messages **you** label. Nothing else in your inbox is visible to it. This is also the safest configuration to hand a nervous client.

---

## 6. First run

1. Label 2–3 messages `ai-triage` (or leave the unread default)
2. Open the workflow → **Execute workflow**
3. Check Gmail: labels applied, drafts sitting in your **Drafts** folder

Each message costs ~3 API calls (classify + embed + draft), so 5 messages ≈ 15 calls. Keep `GMAIL_FETCH_LIMIT` small while testing on the free tier.

---

## 7. Going live (optional)

To run it continuously:

1. Open **Gmail Trigger (production)** → enable it (right-click → Enable)
2. Adjust the poll interval (default: every minute)
3. **Activate** the workflow with the toggle, top right

Now new mail matching your query is triaged automatically, with drafts waiting for you.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `redirect_uri_mismatch` | The URI in Google Cloud must match `http://localhost:5678/rest/oauth2-credential/callback` character for character |
| `access_denied` | Add your Gmail address under **Test users** on the consent screen |
| Auth stops working after a week | Testing-mode tokens expire in 7 days — reconnect the credential |
| No messages fetched | Your `GMAIL_QUERY` matched nothing. Test the same query in the Gmail search bar |
| Drafts empty, labels applied | API quota exhausted — check `draft_status` in the node output |
| Changed `.env`, nothing happened | `docker compose up -d n8n` — env is only read at startup |

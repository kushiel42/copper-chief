# Copper Chief of Staff — Slack → Copper CRM Agent

This is a minimal MVP agent that:

- listens in Slack when you mention it or DM it;
- extracts CRM updates from messy Slack messages/transcripts;
- drafts a Copper CRM update;
- asks for approval in Slack;
- after approval, creates/updates Copper company/person/opportunity, adds an activity note, and creates follow-up tasks.

It intentionally does **not** auto-write without approval unless you set `APPROVAL_REQUIRED=false`.

---

## 0. What you need

You need admin-ish access to:

1. Slack workspace
2. Copper CRM account
3. OpenAI API account
4. Maritime account
5. GitHub account

Do **not** paste API keys into Slack, GitHub, email, Notion, or screenshots.

---

## 1. Create the Slack app

1. Go to Slack API apps: https://api.slack.com/apps
2. Click **Create New App**.
3. Choose **From an app manifest**.
4. Pick your Slack workspace.
5. Choose **YAML**.
6. Paste the full contents of `slack-app-manifest.yaml`.
7. Click **Next**.
8. Click **Create**.

Now create the app-level Socket Mode token:

1. In the Slack app settings, go to **Basic Information**.
2. Scroll to **App-Level Tokens**.
3. Click **Generate Token and Scopes**.
4. Token name: `socket-token`.
5. Add scope: `connections:write`.
6. Click **Generate**.
7. Copy the token that starts with `xapp-`.
8. Save it temporarily in a private note as `SLACK_APP_TOKEN`.

Now install the app:

1. Go to **OAuth & Permissions**.
2. Click **Install to Workspace**.
3. Click **Allow**.
4. Copy the **Bot User OAuth Token** that starts with `xoxb-`.
5. Save it temporarily as `SLACK_BOT_TOKEN`.

Invite the bot into your CRM/dealflow channel:

1. In Slack, create or open a channel like `#copper-chief-of-staff`.
2. Type: `/invite @CopperChief`
3. Also invite it to any channel where you want to mention it.

---

## 2. Create the Copper API token

1. Open Copper CRM in your browser.
2. Go to **System settings** or **Organization settings**.
3. Find **API Keys**.
4. Generate a new API key.
5. Label it: `slack-copper-chief-of-staff`.
6. Copy the token.
7. Save it temporarily as `COPPER_API_TOKEN`.
8. Your Copper login email is `COPPER_USER_EMAIL`.

---

## 3. Create/get OpenAI API key

1. Go to the OpenAI platform.
2. Create an API key.
3. Save it temporarily as `OPENAI_API_KEY`.

Default model in this repo is controlled by `OPENAI_MODEL`. If unsure, keep:

```bash
OPENAI_MODEL=gpt-4.1-mini
```

---

## 4. Put this code into GitHub

Fastest browser-only method:

1. Download and unzip this folder.
2. Go to GitHub.
3. Create a new repository called `copper-chief`.
4. Public is okay because this repo does not contain your secrets. Private is better if Maritime can access it.
5. Click **Add file → Upload files**.
6. Drag all files from this folder into GitHub.
7. Click **Commit changes**.

Important: never upload a real `.env` file or API keys to GitHub.

---

## 5. Deploy to Maritime

Open Terminal on Mac or PowerShell on Windows.

Install Maritime CLI:

```bash
npm install -g maritime-cli
```

Log in:

```bash
maritime login
```

Create the agent:

```bash
maritime create -n copper-chief --framework custom --tier always_on
```

Set secrets. Replace the values with your real tokens:

```bash
maritime env set copper-chief SLACK_BOT_TOKEN=xoxb-your-real-token
maritime env set copper-chief SLACK_APP_TOKEN=xapp-your-real-token
maritime env set copper-chief COPPER_API_TOKEN=your-real-copper-token
maritime env set copper-chief COPPER_USER_EMAIL=you@yourcompany.com
maritime env set copper-chief OPENAI_API_KEY=sk-your-real-openai-key
maritime env set copper-chief OPENAI_MODEL=gpt-4.1-mini
maritime env set copper-chief APPROVAL_REQUIRED=true
maritime env set copper-chief COPPER_DRY_RUN=false
```

Optional but recommended: restrict who can approve updates.

To get your Slack user ID:

1. Click your Slack profile.
2. Click the three dots.
3. Click **Copy member ID**.
4. Then run:

```bash
maritime env set copper-chief ALLOWED_SLACK_USER_IDS=U1234567890
```

Deploy from GitHub. Replace the repo URL:

```bash
maritime deploy copper-chief --source github --repo https://github.com/YOUR_GITHUB_USERNAME/copper-chief.git
```

Watch logs:

```bash
maritime logs copper-chief -n 100
```

You want to see something like:

```text
Starting CopperChief. approval_required=True dry_run=False
```

---

## 6. Test in Slack

DM the bot:

```text
ping
```

Expected response:

```text
pong — I’m connected to Slack. ✅
```

Then ask it to show Copper pipelines:

```text
pipelines
```

Expected response: list of your Copper pipelines/stages.

Now test a fake CRM update:

```text
Had a good intro call with Sarah Lee at XYZ Ventures. Interested in illumicell seed. Potential 250k check. She wants the deck and clinical validation summary by Friday. Follow up next week.
```

Expected flow:

1. Bot says it is drafting the Copper update.
2. Bot posts a proposed CRM update with **Approve** and **Reject** buttons.
3. Click **Approve**.
4. Check Copper for company/person/opportunity/note/tasks.

---

## 7. How to use day-to-day

Use this in Slack:

```text
@CopperChief Had a call with Jane at Acme Ventures. Positive, wants intro to clinical advisor, asked for deck by Wednesday. Move to diligence.
```

For transcripts, paste the transcript into Slack and mention the bot:

```text
@CopperChief ingest this transcript:
[paste transcript]
```

Or upload a `.txt`, `.md`, `.vtt`, or `.srt` transcript and mention the bot in the same message.

Commands:

```text
@CopperChief help
@CopperChief ping
@CopperChief pipelines
@CopperChief todos
```

---

## 8. Safety defaults

This MVP does not delete anything.

It asks for approval before writing to Copper.

If something looks wrong, hit **Reject**.

After you trust it, you can set:

```bash
maritime env set copper-chief APPROVAL_REQUIRED=false
maritime restart copper-chief
```

My recommendation: keep approvals on for stage/value/status changes.

---

## 9. Troubleshooting

If Slack does not respond:

```bash
maritime logs copper-chief -n 100
```

Check that you set:

- `SLACK_BOT_TOKEN` starts with `xoxb-`
- `SLACK_APP_TOKEN` starts with `xapp-`
- Socket Mode is enabled
- The bot is invited to the Slack channel

If Copper writes fail:

- Confirm the Copper API token is valid.
- Confirm `COPPER_USER_EMAIL` is the email that generated the token.
- Run `@CopperChief pipelines`; if that fails, Copper auth is wrong.

If transcript uploads fail:

- Use `.txt`, `.md`, `.vtt`, or `.srt` for the MVP.
- Paste the transcript directly if needed.


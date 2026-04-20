# Debug Agent Setup Guide

## Part 1 — GitHub Actions (PR/push debugging)

### 1. Add your API key to GitHub
Repo → Settings → Secrets and Variables → Actions → New secret
Name: `ANTHROPIC_API_KEY`
Value: your key from console.anthropic.com

### 2. Install the Claude GitHub App
https://github.com/apps/claude
Install it on your repo. This lets Claude push fix branches and open PRs.

### 3. Copy files into your repo
- `.github/workflows/debug-agent.yml` → copy as-is
- `CLAUDE.md` → copy to your repo root

### 4. Push and test
Break something intentionally (bad import, syntax error), push to main or a PR branch.
The debug agent triggers automatically on build/test failure.

---

## Part 2 — Railway Live Deploy Webhook

### 1. Deploy the webhook receiver to Railway
- Create a new Railway service from `railway-webhook/`
- Set runtime to Bun

### 2. Set environment variables in Railway
```
ANTHROPIC_API_KEY=your_key
RAILWAY_API_TOKEN=your_token  (Account Settings → Tokens in Railway dashboard)
RAILWAY_PROJECT_ID=your_project_id
WEBHOOK_SECRET=pick_a_random_string_here
SLACK_WEBHOOK_URL=optional_slack_webhook
```

### 3. Configure Railway webhook
Railway dashboard → your project → Settings → Webhooks
- URL: https://your-debug-service.railway.app/webhook
- Add header: x-webhook-secret: your_WEBHOOK_SECRET value
- Events: Deployment Failed, Deployment Crashed

### 4. That's it
Every failed Railway deploy now fires the debug agent, which analyzes logs and
posts findings to Slack (if configured).

---

## Cost Estimate
- GitHub Actions minutes: ~1-2 min per triggered run
- Claude API: ~$0.01-0.05 per debug run (Sonnet pricing)
- Railway webhook receiver: minimal — only fires on failures

## Adjusting behavior
Edit `CLAUDE.md` to change the agent's debugging methodology.
Edit `claude_args` in the workflow to change model or max turns.

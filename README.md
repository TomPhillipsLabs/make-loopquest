# LoopQuest for Make

A [Make](https://www.make.com) custom app for [LoopQuest](https://loopquest.tomphillips.uk) — put a human in the loop on your AI and automation output. Send an item for review, **block** a downstream action until a person approves it (gate), or review quality in the **background** without pausing anything (monitor).

## What you get

| Module | Type | What it does |
|--------|------|--------------|
| **Create a Review Task** | Action | Sends your output to a human. Pick the game and the mode (gate or monitor), set an optional timeout and fallback. Returns the task `id`. |
| **Watch Verdicts** | Instant trigger | Fires the moment any review resolves — gives you the verdict, choice, reason, flags (`escalated`, `timed_out`) and your `external_id`. This is how you build a blocking gate in Make. |
| **Get Task Status** | Action | Looks up a single review on demand (verdict, reviewer, timing). Useful for a polling loop or a manual check. |

### Games (how the reviewer sees the item)

`swiper` approve/reject · `versus` pick the better of two · `sorter` bucket into categories · `detective` spot the problem · `fixer` correct the output · `redact` mask sensitive text · `grounding` verify a claim against a source.

## Gate vs monitor

- **Monitor** — Create a Review Task and carry on. Reviews happen in the background; use them for quality, spot-checks and an audit trail. Nothing waits.
- **Gate** — the consequential step doesn't live in the same scenario. You split it:
  1. **Scenario A** does the work, then **Create a Review Task** with **Mode = Gate**, and stops.
  2. **Scenario B** starts from the **Watch Verdicts** trigger, adds a **Router**, and only runs the real action on an approve. On a flag or timeout it routes to a fallback.

  The action never runs unless a human approves — that's the "stop". The **External ID** you set on the task comes back on the verdict, so Scenario B knows which item it's acting on. Set a **Timeout** and **On timeout** (defaults to *escalate*, fail-closed) so a gate never hangs forever.

## Set it up

1. Install the app and create a **connection** with your LoopQuest project API key (Workspaces → your project → API keys). The connection test calls `GET /api/v1/me` and labels itself with the workspace name.
2. Add **Create a Review Task** to your scenario. Choose a game, set **Mode**, map your **Content**. For a gate, set **Timeout** and **On timeout**.
3. In a second scenario, add the **Watch Verdicts** trigger, pick the same connection, and **turn the scenario on**. Make auto-subscribes it to your workspace's verdicts (and unsubscribes when you turn it off).
4. Add a **Router** after Watch Verdicts and branch on `verdict` / `escalated` / `timed_out`.

## Build / update the app from this repo

Make apps are configured in the online [Apps editor](https://www.make.com/en/help/apps) — there's no offline package. Paste each file into the matching section:

| File | Section in the Make editor |
|------|----------------------------|
| `app/base.imljson` | **Base** |
| `app/connection.parameters.imljson` / `app/connection.api.imljson` | **Connection** → Parameters / Communication |
| `app/module.createReviewTask.*.imljson` | **Module** *createReviewTask* → Communication / Mappable parameters / Interface |
| `app/module.getTaskStatus.*.imljson` | **Module** *getTaskStatus* → Communication / Mappable parameters / Interface |
| `app/webhook.verdictWebhook.api.imljson` | **Webhook** *verdictWebhook* → Communication |
| `app/webhook.verdictWebhook.attach.imljson` | **Webhook** *verdictWebhook* → **Attach** |
| `app/webhook.verdictWebhook.detach.imljson` | **Webhook** *verdictWebhook* → **Detach** |
| `app/module.watchVerdicts.interface.imljson` | **Module** *watchVerdicts* (Instant trigger, bound to `verdictWebhook`) → Interface |

Notes:
- Module and webhook **names** must be alphanumeric identifiers (`createReviewTask`), no spaces — the display **Label** ("Create a Review Task") is separate.
- `attach`/`detach` go in the webhook's own **Attach** and **Detach** editors, **not** in Communication (Make rejects them there). Attach registers the Make webhook URL with LoopQuest (`POST /api/v1/hooks`) when a scenario is switched on; detach removes it (`DELETE /api/v1/hooks/{id}`) when it's switched off. Subscription is idempotent by URL, so no duplicates.
- If you use a **shared** webhook instead, there are no Attach/Detach steps — register its URL once with `POST /api/v1/hooks` (Bearer API key, body `{ "url": "<webhook url>" }`).

Make can't lint an app offline — validate every module in the editor before requesting review. The IML in `app/` is the source of truth, not a pre-verified package.

## Links

- LoopQuest: https://loopquest.tomphillips.uk
- Make integration guide: https://loopquest.tomphillips.uk/integrations/make
- OpenAPI spec: https://loopquest.tomphillips.uk/openapi.json

## License

MIT

# make-loopquest

A [Make](https://www.make.com) custom app for [LoopQuest](https://loopquest.tomphillips.uk) — send AI/automation output for gamified human-in-the-loop review.

Two ways to set this up. Both use the same LoopQuest API and the same async pattern (submit → human reviews → signed verdict webhooks back).

## Option A — fastest (no custom app)

1. **Action:** use Make's built-in **HTTP** module. Import LoopQuest's [`openapi.json`](https://loopquest.tomphillips.uk/openapi.json) (Make → *Create app from OpenAPI*) to auto-generate the **Create Review Task** call, or just POST to `/api/v1/tasks` with a `Bearer` API key.
2. **Verdict trigger:** add a **Webhooks → Custom webhook** (instant) module, copy its URL, and set it as the **`callback_url`** when you create the task. The signed verdict (with `external_id` + `source`) lands there.

## Option B — branded custom app

Build a Make custom app and paste the blocks in [`app/`](app/) into the matching sections of the Make Apps editor:

| File | Where it goes |
|------|---------------|
| `base.imljson` | Base |
| `connection.parameters.imljson` / `connection.api.imljson` | Connection (API key) |
| `module.createReviewTask.*.imljson` | Action module — Create Review Task |

The connection test calls `GET /api/v1/me` and labels the connection with the workspace name. For verdicts, add an **Instant Trigger** backed by a shared webhook and tell users to paste its URL as `callback_url`.

> Make apps can't be linted offline, so validate in Make's editor before publishing. The IML here is a starting point, not a pre-verified package.

## License

MIT

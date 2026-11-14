# A restaurant content post cannot be published without approval for its *current* revision

Live demo: **https://fejzadenis.github.io/restaurant-content-approval/**

An agency shipping 200+ pieces of restaurant content a month does not have a content-generation problem. It has an
approval problem: approvals arrive in WhatsApp, DMs and email, so nobody is sure which post is approved for which
version. One goes out unapproved, another waits three days on a client who already said yes.

The demo simulates that pipeline in the browser: revision bumps, stale approval links, double taps on a bad connection,
a chase that fires at 48h, and a publish gate that refuses.

## The one rule

> A post can only be published if an approval exists for the revision that is about to go live.

Every safeguard in the demo exists because the naive version of this flow fails in a specific, predictable way:

| Decision | The failure it prevents |
|---|---|
| Approval is keyed on `(post, revision)`, not on the post | Someone approves, the caption is edited afterwards, and the edited version ships on an approval that described different content. |
| Approval links are stateless HMAC tokens carrying `post id + revision` | A forwarded link from last week approves today's version. A token for revision 1 is refused the moment revision 2 exists (`stale_link`). |
| `PRIMARY KEY (content_item_id, revision)` on approvals | Two taps, or one tap twice on a bad connection, and the second answer overwrites the first. The first decision wins. |
| The publish endpoint checks the gate *before* touching the platform | The scheduler publishes at 09:00 whether or not anyone approved. It now refuses with a reason. |
| One chase after 48h, tracked by `reminded_at` | Either nothing is chased and posts rot, or a reminder goes out hourly. One nudge, once, per send. |
| A recorded decision cancels the chase even if the item row lags | The chase and the decision race, and the client is asked to approve what they approved an hour ago. |
| Slack/Notion notification only on a real state change | Every retry rewrites the same "approved" message into the client's channel. |

## What is in here

`index.html` is the interactive demo: no build step, no dependencies, opens offline.

The production version is four importable n8n workflows (approval webhook, publish gate, 48h reminders, self-test) plus a
Postgres schema. The logic was proven by execution rather than description: 20 scenarios pass inside a real n8n 2.47.4
container with 0 failed assertions, and the core module is self-contained (own SHA-256/HMAC, no `require`) so the Code
node runs in n8n Cloud and self-hosted without `NODE_FUNCTION_ALLOW_EXTERNAL`.

The workflows and schema are not in this repository. They are what a client pays for, and they get them wired into their
own Notion/Slack flow instead of a generic template.

## Who this is for

Agencies and in-house teams where a human approves content before it publishes and the approval currently lives in chat.
If that is you, and you want this running on your own data this week, the demo link above is the whole pitch:
https://fejzadenis.github.io/content-approval-gate/ shows the same rule in a second, smaller demo.

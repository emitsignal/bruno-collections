# bruno-collections

[Bruno](https://www.usebruno.com/) API collection for the EmitSignal server.

## Usage

Open Bruno → **Open Collection** → select this directory, then pick the **Local**
environment.

## Environment

`environments/Local.bru` targets the dev server on `:5001`. `token` is declared as a
secret variable, so it is never written to disk in the collection.

| Variable | Purpose |
| --- | --- |
| `baseUrl` | API server origin |
| `appUrl` | Website origin, used in magic links |
| `topic` | Topic name used by the publish, listen, and subscription requests |
| `deviceId` | Device identifier for subscriptions, acknowledgements, and push tokens |
| `token` | Session token or `es_` API key (secret) |
| `messageId`, `subscriptionId`, `pushTokenId`, `webhookId`, `webhookSlug` | Filled in by post-response scripts |

## Authentication

Auth is set at the collection level as `Authorization: Bearer {{token}}`. The server
accepts either a Better Auth session token or an `es_`-prefixed API key in that
header. Requests that are genuinely public — *Publish minimal*, *Receive webhook*,
*Service info* — set `auth: none` so the anonymous paths can be exercised.

Running **Auth → Sign in with email OTP** stores the session token into `token`
automatically. With `EMAIL_PROVIDER=log` (the dev default) the OTP is printed to the
server console rather than sent.

## Chaining

Several requests write their ids back into the environment, so a first pass from top
to bottom populates most variables on its own:

- *Publish JSON* → `messageId`
- *Subscribe* → `subscriptionId`
- *Create webhook* → `webhookId`, `webhookSlug`

## Folders

| Folder | Covers |
| --- | --- |
| Auth | Email OTP send/verify, session, API keys, sign-out |
| Publish | JSON, header-based, scheduled, actions and media, deprecated `/topic/*` |
| Topics | List/get/update, messages, metrics, claim/release, suggestions, SSE listen |
| Messages | Get, acknowledge, attachment upload, purge, upload serving |
| Subscriptions | Subscribe/update/unsubscribe, claim, inbox messages, metrics |
| Push Tokens | List, register, toggle |
| Webhooks | CRUD, deliveries, SSE stream, public `/h/:slug` ingest |
| Account | Billing, avatar upload and delete |

## Notes

- Optional query parameters use Bruno's `~` disabled prefix — toggle them in the UI
  rather than editing URLs.
- The SSE requests (*Listen to topic*, *Listen to many topics*, *Stream deliveries*)
  are long-lived and must be cancelled manually.
- The multipart requests (*Upload attachments*, *Upload avatar*) need a file selected
  in the UI before they will send.
- *Create webhook with signing secret* carries a placeholder secret; replace it with
  the provider's real signing secret.

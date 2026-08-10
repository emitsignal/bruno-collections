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

## File conventions

Keys are sorted alphabetically in every order-insensitive block — `meta`, the method
block, `headers`, `params:query`, `vars`, and object keys inside `body:json`. Array
element order, block order, and the `seq` values are meaningful and left alone.

Indentation is 4 spaces per `.editorconfig`, with one exception: `docs`, `body:text`,
and the multiline `body:json` / `script` blocks keep a **2-space base indent**. Bruno's
parser outdents those blocks by a hardcoded 2 spaces, so anything deeper becomes part
of the parsed content — for `body:text` that content is the request payload. Their
inner levels are still 4, so the JSON and JS that Bruno hands to the server are
4-space indented.

Note that editing a request in Bruno's GUI makes it re-serialize that file with its
own 2-space indentation and its own key order. Re-run the formatting pass if you want
the collection to stay consistent after GUI edits.

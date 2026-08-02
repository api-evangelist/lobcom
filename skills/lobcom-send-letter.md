---
name: Send a letter with Lob
description: Verify a recipient address and mail a letter via the Lob Print & Mail API, safely and idempotently.
api: openapi/lobcom-openapi-original.yml
operations: [us_verification, address_create, letter_create, letter_retrieve]
---

# Send a letter with Lob

Mail a physical letter to a recipient, verifying the address first and using an
idempotency key so retries never produce duplicate mail.

## Auth
HTTP Basic. Send your secret API key as the Basic username, password blank.
Use a `test_` key while developing (no real mail, no charges); switch to `live_` for production.

## Steps
1. **Verify the recipient address** — call `us_verification` (POST `/us_verifications`) with the
   recipient's address. Confirm `deliverability` is `deliverable` (or an acceptable value) before mailing.
2. **(Optional) Save the address** — call `address_create` (POST `/addresses`) to persist the recipient
   in the address book, or pass an inline address object in the next step.
3. **Create the letter** — call `letter_create` (POST `/letters`) with `to`, `from`, `file` (an HTML
   string or PDF URL), and `color`. **Send an `Idempotency-Key` header** (unique per logical send,
   ≤256 chars) so a retried request returns the original letter instead of mailing a second one.
4. **Track delivery** — poll `letter_retrieve` (GET `/letters/{ltr_id}`) or subscribe to webhook events
   (`letter.mailed`, `letter.in_transit`, `letter.delivered`, `letter.returned_to_sender`).

## Rules
- Live letters require a verified email and a billing address, or you get `billing_address_required` / `email_required`.
- Files must be ≥300 DPI, fonts embedded, correct dimensions — else `invalid_image_dpi` / `unembedded_fonts` / `invalid_file_dimensions`.
- On HTTP 429 (`rate_limit_exceeded`) back off and honor the `ratelimit-reset` header.
- Error envelope: `{ "error": { "code", "message", "status_code" } }`. See errors/lobcom-error-codes.yml.

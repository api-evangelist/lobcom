---
name: Verify and autocomplete addresses with Lob
description: Validate US and international addresses and autocomplete partial US addresses via the Lob Address Verification API.
api: openapi/lobcom-openapi-original.yml
operations: [us_verification, intl_verification, autocompletion, zip_lookup]
---

# Verify and autocomplete addresses with Lob

Clean, validate, and complete mailing addresses before storing or mailing to them.

## Auth
HTTP Basic. A **publishable** key (`[env]_pub`) is sufficient for verification and
autocompletion and is safe to use client-side; a secret key also works.

## Steps
1. **Autocomplete a partial US address** — call `autocompletion` (POST `/us_autocompletions`) with a
   partial `address_prefix` to get suggestion candidates as the user types.
2. **Verify a US address** — call `us_verification` (POST `/us_verifications`) with the full address.
   Read `deliverability` (`deliverable`, `deliverable_unnecessary_unit`, `deliverable_incorrect_unit`,
   `deliverable_missing_unit`, `undeliverable`) and use the standardized components it returns.
3. **Verify an international address** — call `intl_verification` (POST `/intl_verifications`) with
   `country` for non-US addresses.
4. **ZIP lookup** — call `zip_lookup` (POST `/us_zip_lookups`) to expand a ZIP code into city/state and valid ranges.

## Rules
- Use the standardized/corrected components Lob returns rather than the raw user input.
- Publishable keys are limited to these verification/autocomplete endpoints — using one on a
  print endpoint returns `publishable_key_not_allowed`.
- Bulk verification is available via `bulk_us_verifications` / `bulk_intl_verifications` for lists.
- Error envelope: `{ "error": { "code", "message", "status_code" } }`.

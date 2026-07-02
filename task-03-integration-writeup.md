# Task 03 — Integration Design: Landing Page → HubSpot → WhatsApp → Google Ads

## Architecture

On submit, the form fires two things in parallel: a client-side `dataLayer.push({event: 'consultation_form_submitted', ...})` (which GTM uses to fire the Google Ads conversion tag immediately) and a `POST` to a lightweight serverless function (Cloudflare Worker or Vercel Function). I'm choosing a **direct API call from a serverless function** over Zapier/Make or HubSpot's native embed, because this flow has a hard 2-minute SLA and a data-integrity requirement (phone dedup) that no-code tools handle poorly — Zapier's polling delays and generic error handling aren't built for SLA-bound lead flows, and native HubSpot embeds can't branch logic before the WhatsApp step. A serverless function gives full control over ordering, retries, and error handling for a modest amount of code.

Inside the function: first, search HubSpot Contacts via the CRM API v3 using a **custom unique property** (`phone_number`, not email) as the search key. If a match exists, update it; if not, create a new contact with Name, Phone, Clinic Preference, `Source = "Google Ads - Consultation Landing Page"`, and `Lead Status = "New Enquiry"`. In parallel — not sequentially after HubSpot — call Karix's WhatsApp Business API to send the confirmation template. Running these two calls concurrently, rather than chaining them, is what protects the 2-minute SLA if HubSpot is briefly slow. Finally, log a server-side conversion via Google Ads' Conversion API as a backup to the client-side tag, in case an ad blocker or early tab-close prevents the pixel from firing.

## Biggest failure point

**HubSpot's default deduplication matches on email, and this form never collects one.** Without a custom unique `phone_number` property, every resubmission — including the same patient trying again after a failed call-back — creates a duplicate contact. That fragments lead history and Lead Status, and support ends up treating one patient as several. Fallback: make `phone_number` a HubSpot unique property, search on it before every create, and if a match is found with a Lead Status *past* "New Enquiry," don't overwrite it — log a "duplicate enquiry" timeline note instead so the record reflects a real second contact without regressing the pipeline stage.

## Protecting the WhatsApp SLA

Likely breaks: Karix rate limits or downtime, an unapproved/expired WhatsApp template, or the function blocking on HubSpot before attempting WhatsApp. Mitigations: send WhatsApp in parallel, add retry logic with backoff, and alert if failures exceed 2% or p95 latency exceeds 90 seconds, via basic serverless logging (e.g. Sentry).

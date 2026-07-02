# OrthoNow — Developer Assignment Submission

Submission for the Developer (Position 1, Client Web + Martech) assignment.

## Contents

| File | Task |
|---|---|
| [`task-01-gtm-schema.md`](./task-01-gtm-schema.md) | GTM event schema, booking funnel dataLayer JSON, Google Ads conversion recommendation |
| [`task-02-landing-page/index.html`](./task-02-landing-page/index.html) | Single-file HTML/CSS/JS "Book a Consultation" landing page |
| [`task-03-integration-writeup.md`](./task-03-integration-writeup.md) | HubSpot, WhatsApp, and Google Ads integration architecture |

## Task 02 — Landing Page

`task-02-landing-page/index.html` is fully self-contained: no build step, no server, no external requests. System font stack, inline SVG icons, no images, no CDN dependencies. Open the file directly in a browser or serve it from any static host.

**PageSpeed Insights Mobile:** [score] — see `task-02-landing-page/pagespeed-mobile.png`.

**dataLayer verification:**

1. Open the page and open DevTools → Console.
2. Fill in a name and a valid 10-digit mobile number, then submit.
3. The form is replaced by a thank-you state with no page reload.
4. `window.dataLayer` will contain a `consultation_form_submitted` event with `lead_source`, `pain_type`, and `form_location` parameters.

## Loom walkthrough

[Loom link]

Covers: GTM schema decisions, a live demo of the dataLayer push firing in the browser console, and the integration architecture answer.

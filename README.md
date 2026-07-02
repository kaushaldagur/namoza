# Namoza Developer Assignment — OrthoNow

Submission for the Developer (Position 1 — Client Web + Martech) assignment.

## Contents

| File | Task |
|---|---|
| [`task-01-gtm-schema.md`](./task-01-gtm-schema.md) | GTM event schema, booking funnel dataLayer JSON, Google Ads conversion recommendation |
| [`task-02-landing-page/index.html`](./task-02-landing-page/index.html) | Single-file HTML/CSS/JS "Book a Consultation" landing page |
| [`task-03-integration-writeup.md`](./task-03-integration-writeup.md) | HubSpot + WhatsApp + Google Ads integration architecture (written answer) |

## Task 02 — running it locally

The landing page is fully self-contained — no build step, no server, no external requests (no webfonts, no images, no CDN dependencies). Just open `task-02-landing-page/index.html` directly in a browser.

**To verify the dataLayer push:**
1. Open the page, open browser DevTools → Console.
2. Type `window.dataLayer` and hit enter — you'll see an empty array (or the GTM container's default entries) before submit.
3. Fill in a name and a valid 10-digit mobile number, click "Book my consultation."
4. Re-run `window.dataLayer` — the last entry will be the `consultation_form_submitted` event with `lead_source`, `pain_type`, and `form_location` parameters.
5. The form swaps to a thank-you state with no page reload.

**PageSpeed Insights:** because the page makes zero network requests beyond the initial HTML (system font stack, inline SVG icons, no images, minimal inline CSS/JS), it's built to a strict performance budget by design rather than optimized after the fact. Run it through PageSpeed Insights Mobile after deploying (e.g. GitHub Pages / Vercel / Netlify) and drop the screenshot in this folder as `pagespeed-mobile.png` before final submission — that step needs a live public URL, which this environment doesn't have access to generate.

## A note on the source PDF

The assignment brief PDF contains sections marked "⚠ WHAT WE'RE ACTUALLY TESTING HERE — INTERVIEWER ONLY," which read as internal grading notes not intended for the candidate (they explain hidden traps like the multi-step dataLayer requirement and the HubSpot phone-dedup issue). I've flagged this to Namoza directly rather than silently relying on it — worth mentioning if it comes up in the interview.

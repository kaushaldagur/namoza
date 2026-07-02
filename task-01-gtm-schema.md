# Task 01 — GTM Event Schema: OrthoNow

## 1. Full Event Schema

| Event Name | Trigger Type | Key Parameters (min. 3) | GA4 Report / Audience it feeds |
|---|---|---|---|
| `booking_step_complete` (step 1) | Custom Event trigger, fires on `dataLayer.push` when patient selects clinic + specialty | `step_number`, `step_name`, `clinic_location`, `specialty` | Funnel Exploration (Step 1) → "Started Booking" remarketing audience |
| `booking_step_complete` (step 2) | Custom Event trigger, same event name, fires when contact details are validated and patient clicks "Continue" | `step_number`, `step_name`, `clinic_location`, `has_preferred_date` | Funnel Exploration (Step 2) → drop-off diagnostic |
| `booking_step_complete` (step 3, `step_name: booking_confirmed`) | Custom Event trigger, fires on booking success response from backend (not on button click — on confirmed API response) | `step_number`, `step_name`, `clinic_location`, `booking_id` | Funnel Exploration (Step 3) → **primary conversion**, "Booked Patients" audience |
| `call_now_click` | Click Trigger — Just Links, matching CSS class `.call-now-btn` | `button_location` (homepage / clinic page / landing), `clinic_name`, `phone_number` | Engagement report; Google Ads conversion candidate |
| `whatsapp_chat_click` | Click Trigger on floating widget element ID `#wa-widget` | `page_path`, `page_type`, `device_category` | Engagement report; WhatsApp-intent remarketing audience |
| `guide_download_lead` | Form Submission trigger on gate form (name + phone) | `form_location`, `lead_source`, `guide_title` | Lead generation report; "Guide Downloaders" nurture audience |
| `guide_download_complete` | Click Trigger on the actual PDF link, fires only after `guide_download_lead` | `guide_title`, `file_url`, `lead_source` | Content engagement report |
| `clinic_page_view` | History Change / Page View trigger, filtered to `/clinics/*` paths | `clinic_id`, `clinic_city`, `page_path` | Location performance report (custom exploration by `clinic_id`) |
| `blog_scroll_depth` | Scroll Depth trigger at 25 / 50 / 75 / 90% (Built-In Variable: Scroll Depth Threshold) | `percent_scrolled`, `article_title`, `article_category` | Content engagement report; "Engaged Readers" audience for retargeting |

**Note on `guide_download_complete`:** this only fires *after* the gate form has already fired `guide_download_lead`, so it doesn't need its own name/phone parameters — it's purely a content-engagement signal layered on top of the lead event.

---

## 2. Booking Funnel — Step-Level Drop-off Tracking

GTM has no native way to detect progress through a client-rendered, multi-step form — there's no URL change, and often no full page reload between steps. **GTM only sees what's explicitly pushed to the dataLayer.** This means step 2 and step 3 tracking cannot be "configured" in GTM alone — the front-end developer has to add a `dataLayer.push()` call at the exact moment each step is validated and the user advances, using the *same* event name across all three steps (`booking_step_complete`) with a `step_number` parameter that changes. Using one consistent event name is what lets GA4 build a single Funnel Exploration off the `step_number` parameter, rather than three disconnected events.

**Step 1 — location + specialty selected:**
```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "{{clinic name}}",
  "specialty": "{{specialty selected}}"
}
```

**Step 2 — contact details entered:**
```json
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "contact_details_entered",
  "clinic_location": "{{clinic name}}",
  "has_preferred_date": true
}
```

**Step 3 — booking confirmed:**
```json
{
  "event": "booking_step_complete",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "{{clinic name}}",
  "specialty": "{{specialty selected}}",
  "booking_id": "{{booking id from API response}}"
}
```

**GTM setup:**
- One Custom Event trigger listening for `booking_step_complete`.
- One GA4 Event tag mapped to that trigger, reading `step_number`, `step_name`, `clinic_location`, `specialty`, `booking_id` as Data Layer Variables and passing them through as event parameters.
- Step 3 fires on the *confirmed API response*, not the button click — otherwise a failed booking (e.g. slot no longer available) would be miscounted as a conversion.

**Surfacing drop-off in GA4:**
- Build an **open Funnel Exploration** with three steps, each defined as `event_name = booking_step_complete AND step_number = 1 / 2 / 3`.
- GA4 shows completion rate and drop-off % between each consecutive step natively in the funnel visualization — no extra configuration needed once the parameter is being sent correctly.
- Breaking the funnel down by `clinic_location` in the same exploration reveals whether drop-off is uniform or concentrated at specific clinics (e.g. a clinic with poor available-date coverage).

---

## 3. Google Ads Conversion Import Recommendation

**Import: `booking_step_complete` filtered to `step_number = 3` (the `booking_confirmed` step) — not `call_now_click` or an earlier booking step.**

Reasoning: `call_now_click` and early booking steps are *engagement* signals, not proof of a real lead — someone can tap "Call Now" out of curiosity and never call, or abandon at step 1. If one of these is imported as the optimization target, Google's automated bidding (Target CPA / Target ROAS) will learn to find more people who behave like *clickers*, not more people who *become patients* — this is a classic way performance marketing quietly degrades lead quality while volume looks healthy on a surface-level dashboard. Step 3 is the closest event in this schema to an actual completed appointment request, so it's what the bidding algorithm should be shown as the goal — it protects lead quality, not just lead volume. In GA4, this is set up as a **key event** scoped to `booking_step_complete` with a `step_number = 3` parameter condition, then imported into Google Ads as a conversion action from Google Ads → Conversions → linked GA4 property.

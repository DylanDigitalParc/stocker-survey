[README-integration.md](https://github.com/user-attachments/files/32575924/README-integration.md)
# Stocker Agency Insurance Assessment — integration notes

## What this is

A working, tested implementation of the assessment described in
`Stocker_Agency_Complete_Development_Directive.docx`: the full 10-question
flow, branching/allocation logic, and a results engine — built as plain
HTML/CSS/JS with no build step or framework dependency, so it can be dropped
into stockeragency.com as-is.

Files:

- `index.html` — the page shell. Load order matters (already set correctly
  in this file): `assessment-config.js` → `insight-library.js` →
  `routing-engine.js` → `submission-adapter.js` → `app.js`.
- `assessment-config.js` — every question, option, and bank from Appendix A,
  verbatim. Option labels are preserved exactly; IDs are stable.
- `insight-library.js` — the results copy (strength/opportunity/next-step
  text). **This is new content this build adds** — the directive specified
  the *rules* for results (Section 6) but the source document didn't include
  a written insight library, so this is draft copy for your team to review,
  not yet legally/compliance reviewed.
- `routing-engine.js` — the five pure functions Section 9 asks for
  (`selectCategories`, `allocateSlots`, `buildDynamicQueue`,
  `reconcileAnswers`, `selectInsights`), with no DOM dependency so they're
  unit-testable on their own.
- `app.js` — UI state machine, rendering, validation, and the lead-form flow.
- `submission-adapter.js` — the one place that knows how to actually deliver
  a completed lead (see "Lead delivery" below).
- `styles.css` — mobile-first styling using CSS variables for color/font, so
  swapping in real Stocker Agency branding is a find-and-replace in `:root`.
- `test-acceptance.js` — runs the Section 10 acceptance tests against the
  routing engine with plain Node (`node test-acceptance.js`, no install
  needed). All 14 currently pass.

## Branding — placeholder only

I could not reach stockeragency.com directly from here to pull your live
colors/fonts/logo, so `styles.css` currently uses a neutral professional
placeholder palette (navy/blue) in CSS variables at the top of the file.
Before launch, please send: your brand hex codes (or a link to a live brand
kit), your logo file (SVG preferred), and your body/heading font — I'll drop
them straight into `:root` and the header markup.

## Lead delivery — two modes, pick one

`app.js` never talks to a backend directly — it hands a complete payload to
`submission-adapter.js`, which is the swappable "integration adapter"
Section 8 asks for. Two modes are built:

**Gravity Forms mode** (the option you asked about): the assessment keeps
its own custom-built, styled lead form for the actual UX (Section 2 spec —
first/last/email required, phone optional, separate unchecked consent), and
uses a real Gravity Forms form already on the page purely as the delivery
mechanism — so GF's own notifications, entry storage, spam protection, and
any CRM add-ons you already have configured keep working untouched. On
submit, the adapter fills that GF form's hidden fields with the computed
payload and triggers GF's own AJAX submit.

To turn this on:
1. In WordPress, create a Gravity Forms form with: First Name, Last Name,
   Email, Phone, a Consent checkbox, and one more hidden single-line text
   field to hold the full JSON payload (so nothing is lost even if you don't
   map every individual field).
2. Embed that form on the assessment page (it can be visually hidden with
   CSS — it doesn't need to be seen, since our own form collects the input).
3. In `submission-adapter.js`, fill in `GF_FIELD_MAP` with that form's real
   ID and field input IDs (visible in the GF form editor), and set
   `SUBMIT_MODE: "gravity-forms"` in `app.js`'s `APP_CONFIG`.

**Webhook mode** (default right now, so the app is testable standalone):
POSTs the same JSON payload to `APP_CONFIG.SUBMIT_ENDPOINT`. Point this at
whatever endpoint should receive it — including a Gravity Forms REST API
endpoint, Zapier webhook, or a custom one — if you'd rather not embed a real
GF form on the page.

Either way the payload shape matches Section 8 exactly (assessment_id,
version, contact fields, consent, source/UTMs, full answers array, etc.).

## Embedding on stockeragency.com

This is plain HTML/CSS/JS, so the simplest path is a WordPress "Custom
HTML" block (or a page template) containing the body of `index.html`, with
the four script tags pointing at the files uploaded to your theme/child-theme
assets. No build step, npm, or server-side rendering required.

## Open items — still need your/Jim's sign-off (carried over from Section 11)

These were already flagged in the directive and are still open; nothing
below has been guessed at in the code — placeholders are clearly marked:

- Final Gravity Forms form ID + field mapping (see above).
- Real scheduling URL for the "Schedule a Coverage Review" button
  (`APP_CONFIG.SCHEDULING_URL` in `app.js` — currently `#schedule-a-review`).
- Whether coverage reviews are free — the results/CTA copy currently avoids
  claiming this either way.
- Lead notification recipients/routing rules within Gravity Forms.
- Approved privacy/consent legal text (current consent line is placeholder
  marketing copy, not legal review).
- Real brand colors, fonts, and logo file.
- Review of the full insight library in `insight-library.js` — this is the
  one piece of new content beyond the source document, and is the part most
  worth a careful read before launch.

# Vibes Entertainment Website & Booking System

## Product overview

Build a mobile-first, dark premium website and booking-management system for **Vibes Entertainment**, the DJ and event services business owned by Mr. Ofure Egharevba (DJ Ofu) in Benin City, Edo State, Nigeria. The public site presents approved services, rates, FAQs, and event work in English and Nigerian Pidgin. Customers can submit an event enquiry; the owner manages website and manually received enquiries in a secure dashboard.

The system uses Next.js, Tailwind, Supabase (database and authentication), and Vercel. Bookings are never auto-confirmed. The owner approves every booking and availability is rechecked at approval. Customer contact and follow-up are manual; the system does not integrate with WhatsApp or send automated messages.

## Goals and success criteria

- Give event organizers a polished, responsive way to understand services and submit complete enquiries.
- Explain DJ, MC, and concert hype-man services distinctly; a hype man is never described as an MC.
- Show only owner-approved prices and public content. Unlisted or variable events say “Quotation required.”
- Centralize online and manually received enquiries, booking decisions, rates, content, and follow-up tracking for the owner.
- Prevent conflicting confirmed events by checking availability again at approval time.
- Provide chatbot answers only from approved FAQ and rate data.

## Users

- **Prospective customers:** browse the site, switch language, ask approved-information questions, and submit event enquiries.
- **Owner (Mr. Ofure / DJ Ofu):** manage leads, availability, booking decisions, rates, FAQs, gallery content, and manual follow-ups.

## Public website sitemap and experience

Use a multi-page site with `/en/...` and `/pidgin/...` paths and a visible English / Nigerian Pidgin switcher. Pidgin wording is separately maintained and owner-approved; do not automatically translate published copy.

- **Home:** editorial hero, three service pillars (DJ artistry, MC, concert hype-man), approved gallery preview, DJ Ofu introduction, approved rates snapshot, optional audio sample, and booking call to action. Hide unsupported claims and unavailable assets.
- **About DJ Ofu:** approved biography and career details.
- **Services & Rates:** service descriptions, confirmed packages and minimum starting rates, plus custom-quotation categories.
- **Gallery:** approved, rights-cleared event media with category filters.
- **FAQ:** owner-approved questions and answers in each supported language.
- **Booking / Enquiry:** four-step progressive form for contact, event/location, requested services/equipment, and musical direction/special requirements.
- **Contact:** only owner-approved contact details, address, and social links.

### Visual system

- Afro-luxury sonic direction: midnight charcoal backgrounds (`#0B0E16`, `#10131B`), layered dark surfaces (`#181B24`, `#1E222D`), warm gold primary accent (`#E5B842`) and hover (`#F3CE6D`), white primary text, and muted gray secondary text.
- Use Syne for headings and Plus Jakarta Sans or Inter for body/UI text. Maintain accessible contrast and touch targets of at least 48 px on mobile.
- Use 8 px standard card/input radii and pill styling for tags and primary actions.
- Responsive navigation: desktop links and booking CTA; mobile compact header, language toggle, and expandable drawer.
- Keep an “Ask DJ Ofu” chatbot entry available across public pages; it must follow the approved-data rules below.

### Public rates and content

- Confirmed Wedding DJ + MC package: **₦1,500,000**.
- Confirmed Birthday DJ + MC package: **₦1,100,000**.
- Confirmed starting price floors: DJ **₦200,000**, MC **₦300,000**, concert hype man **₦300,000**.
- Sound and equipment are included; transport is included at a fixed amount. Do not invent an amount or imply a separate charge. Overtime and changed plans may cost extra at owner-set charges.
- Store all rates in owner-editable data, not code. Publicly show only rates approved for publication. Use “Quotation required” for custom/unlisted events and variable scope. Do not derive a discount from component rates.
- Package durations and specific equipment/production inclusions are not confirmed and must not be advertised until the owner approves them.
- Do not publish unverified ratings, guest counts, years of experience, equipment models, setup-time promises, touring claims, client names, or testimonials.
- Use only approved event images/audio. Omit the audio player until an owner-provided sample and its streaming source are approved.
- Phone numbers, WhatsApp contact details, physical addresses, and social handles in the prototype are placeholders. Do not publish them until replaced with owner-approved details. A manually operated direct contact link may appear as a secondary action once approved; it is not an application integration.

## Booking and enquiry requirements

### Public booking form

Use four steps:

1. **Contact:** full name and phone are required; email is optional. Do not collect a separate WhatsApp number.
2. **Event and location:** event type, event date, expected audience band, estimated duration, venue name, and location. Support custom/other events.
3. **Service and equipment:** choose DJ, MC, hype man, combined package, or custom quote; capture equipment/sound needs. The owner may review and change final scope.
4. **Musical direction:** cultural protocol, special requests, and do-not-play notes.

Show an estimate only when the selected rate is approved for publication. The confirmation notice must say the request is an enquiry pending owner review, not a reservation or confirmed booking. Do not show a live availability calendar. Do not imply that the displayed estimate is a final quote when the scope may change.

### Approval and availability

Use a **single confirmed event per date** model for v1.

1. Validate the submitted event details and check the selected date against confirmed bookings and owner-created date blocks.
2. Save an available-date request as `pending_owner_approval`. Submission does not reserve or block the date.
3. Notify the owner in the admin dashboard.
4. When the owner approves, run an atomic server-side transaction that rechecks availability. If available, create the confirmed booking and mark the request `confirmed`; if no longer available, create no booking and return a conflict for owner resolution.
5. If requests compete for a date, the first successful approval transaction wins. Other requests remain pending until the owner resolves them.
6. A date or service change returns the request to pending approval and requires a new availability check. Declines do not block dates.

## Owner admin experience

Owner-only navigation: **Enquiry Inbox**, **Rates & Add-ons**, **Calendar & Blackouts**, plus **Exit to Live Site**. Use a persistent desktop sidebar and a mobile drawer.

- **Enquiry Inbox:** dynamic totals, due-today and overdue follow-ups, pipeline filters, and searchable enquiry list. Do not use prototype KPI counts or mock customer records as live business data; show real data or empty states.
- **Manual enquiry form:** allow the owner to create an enquiry received by phone, referral, or other channel and edit existing records.
- **Enquiry detail/edit:** manage customer/event details, enquiry source, lead status, owner-entered quote, private crew notes, follow-up date/time, and activity history. A manual call link may be shown only when an approved customer phone number exists.
- **Follow-up tracking:** list enquiries whose owner-selected follow-up date is today or earlier. The owner records contact and marks the follow-up completed or sets a new date. No 24-hour automation, reminders, WhatsApp notifications, or customer messages.
- **Booking requests:** review, approve, or decline website requests and see the approval-time availability result.
- **Rates & Add-ons:** edit service rates and publication approval without changing code.
- **Calendar & Blackouts:** view confirmed events and owner-created unavailable dates.
- **FAQ, Gallery, and Settings:** manage approved public content, media rights/publication state, business details, and owner profile.

### Lead states

Support `new_enquiry`, `in_conversation`, `awaiting_customer_response`, `follow_up_required`, `follow_up_completed`, `converted_to_booking`, `declined`, and `lost_lead`. Pipeline labels such as “Pending Review,” “Quote Sent,” and “Confirmed” must map to the underlying lead/request states rather than create conflicting statuses. Record manual contact and status changes in the activity history.

## Chatbot

Use a constrained FAQ and rate lookup. The chatbot may explain approved services, show rates approved for publication, collect basic event details, and link to the booking form. For unlisted/variable rates or unsupported questions, say “Quotation required” or that it cannot confirm, then direct the customer to the form or owner-approved contact route. Never invent price, availability, policy, client history, or service promises. Keep hype man distinct from MC in both languages.

## Data model and interfaces

Use Supabase Auth and row-level security for owner data. Public queries expose only published approved content/rates; customer contact details, private notes, leads, and booking administration are owner-only.

- **`profiles`**: owner identity, role, and contact settings.
- **`services` / `event_types`**: service and event catalogues, including custom/unlisted types.
- **`rates`**: service/package, event type, NGN amount, publication approval, effective dates, and owner update metadata.
- **`availability_blocks`**: blocked dates/ranges and owner note.
- **`leads`**: customer name, required phone, optional email, event details, requested services, equipment/musical notes, source, status, owner quote, private notes, and optional next follow-up date/time.
- **`booking_requests`**: lead reference, request status, submitted details, owner decision, and approval-time availability check metadata.
- **`bookings`**: confirmed date, services, source request, final owner-approved terms, and confirmation timestamp; enforce one confirmed booking per date in v1.
- **`lead_activities`**: timestamped status changes, manual contacts/responses, follow-up completion, and booking decisions.
- **`faqs` / `gallery_items`**: separate approved English and Nigerian Pidgin content, approval/publication state, and media rights status.

Use integer NGN amounts and UUID primary keys with timestamps and foreign keys. Do not add WhatsApp provider settings or automated reminder-job records.

## Milestones

1. **Approve design and content inputs:** review Stitch screens; obtain approved logo, biography, contact details, translations, FAQs, media/audio rights, remaining rates, and any package-specific inclusions.
2. **Public site:** implement responsive page shell, language routes, service/rates data, gallery, FAQ, booking form, and chatbot entry.
3. **Database and owner access:** add schema, authentication, row-level security, and admin shell.
4. **Booking and enquiries:** implement public submission, manual admin entry, statuses, availability blocks, atomic approval recheck, and decisions.
5. **Owner follow-up and content management:** implement follow-up due list, activity logging, rates/FAQ/gallery editing, and approved-content publication.
6. **Acceptance and launch:** verify mobile/desktop behavior, languages, booking conflicts, pricing/content gates, admin permissions, and public contact details; deploy to Vercel and hand over owner instructions.

## Acceptance scenarios

- Customer form works as a four-step responsive flow, requires phone, permits an optional email, and presents no separate WhatsApp field.
- Form submission creates a pending request, never a confirmed booking or date reservation.
- A date blocked or booked after submission prevents owner approval; two concurrent approvals cannot confirm the same date.
- Owner can manually create/edit a lead and set a follow-up date. Due/overdue leads appear in the admin list; completing or rescheduling a follow-up updates that list.
- The application sends no WhatsApp API notifications, automated customer messages, or 24-hour reminders.
- Public site/chatbot show only owner-approved published prices and content. Unlisted events show “Quotation required.”
- Pidgin has separate owner-approved copy; chatbot keeps hype man distinct from MC in both languages.
- Prototype claims, mock records, placeholder contacts, and unapproved package inclusions never appear as live public content.
- Unauthorized users cannot read or modify customer contact data, leads, private notes, rates, or booking administration.

## Open questions and risks

- Confirm remaining public rates for corporate, small-event, club/lounge, and other categories; only confirmed base prices are currently defined.
- Provide owner-approved phone, address, social links, biography, logo, gallery media, audio sample, English/Pidgin copy, and FAQ answers before launch.
- Confirm whether any proposed package duration, equipment inclusion, production effect, deposit, contract, escrow, setup timing, rating, or experience claim is factual and approved. Until confirmed, keep it unpublished.
- The fixed transport amount is unspecified. Transport remains included; do not show or infer a numeric amount.
- Deposit/payment, cancellation, rescheduling, duration, coverage area, and special-equipment policies are not confirmed. Do not promise these through the site or chatbot.
- The initial availability model allows one event per date. Any future same-day capacity requires an owner-defined service/resource model.
- Set customer-data retention expectations and operational monitoring for form delivery and admin access before launch.

## Assumptions

- The Stitch prototype is the approved visual direction, subject to the content and business-rule gates in this PRD.
- Website enquiries and manually entered leads share the same owner-managed lead records.
- Owner-selected follow-up dates and a dashboard due list replace automated 24-hour reminders.
- No customer accounts, online payments/deposits, WhatsApp API, automated customer messaging, or advanced calendar integration are in v1.

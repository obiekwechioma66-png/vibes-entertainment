# Vibes Entertainment Website & Booking System

## Product summary

Build a mobile-first, dark premium website and booking-management system for **Vibes Entertainment**, the DJ/MC business owned by Mr. Ofure Egharevba (DJ Ofu) in Benin City, Edo State, Nigeria. The site helps prospective customers understand services, browse approved event work, ask common questions, and submit structured enquiries. The owner uses a secure dashboard to manage leads, approved content and rates, availability, and booking decisions.

The system uses Next.js, Tailwind, Supabase (database and authentication), WhatsApp Business Cloud API, and Vercel. Bookings are never confirmed automatically. The system never sends follow-up messages to customers; 24-hour reminders go to the owner only.

## Goals and success criteria

- Give customers a clear, mobile-friendly way to understand services and submit complete enquiries.
- Centralize leads and booking requests in an owner-only dashboard.
- Prevent conflicting confirmed events by checking availability again at approval time.
- Let the owner edit prices and approve public content without code changes.
- Answer chatbot questions only from approved FAQ and rate data; route unknown questions to the owner.
- Remind the owner about inactive leads after 24 hours while avoiding duplicate or stale reminders.

## Users

- **Prospective customers:** browse services, rates, FAQs, gallery, and submit event details.
- **Owner (Mr. Ofure / DJ Ofu):** review enquiries, manage availability, decide bookings, maintain rates/content, and follow up manually.

## Sitemap and main flows

### Public website

- **Home:** business introduction, key services, selected work, and enquiry call to action.
- **About:** approved business background and DJ Ofu profile.
- **Services & Rates:** DJ, MC, hype-man, and DJ + MC packages. Show only rates approved for publication. Show “Quotation required” for unlisted events.
- **Gallery:** owner-approved photographs and videos with publication rights.
- **FAQ:** answers approved by the owner.
- **Booking / Enquiry:** structured event and service form; show an approved indicative price or “Quotation required.”
- **Contact:** approved business contact details and social links.

### Admin dashboard

- **Overview:** pending requests, upcoming confirmed events, and follow-ups due.
- **Enquiries:** search and inspect leads, update status, log contact/customer response, and review activity history.
- **Booking requests:** review details and approve or decline requests.
- **Availability:** see confirmed bookings and create date blocks.
- **Rates & services:** edit rates, service/event catalogue, and publication approval.
- **FAQ & content:** manage public FAQ and content.
- **Gallery:** manage media, captions, permissions, and publication state.
- **Settings:** owner profile, contact information, and notification setup.

## Functional requirements

### Enquiry capture

Collect customer name and phone, event type, date, venue/location, duration, expected audience, requested service(s), sound/equipment needs, and special requirements. Support custom/unlisted event types. Save each submission as an enquiry and pending booking request; never imply that submission reserves or confirms the date.

### Booking decisions and availability

Use a **single event per date** availability model for the initial release.

1. Validate required details and check the requested date against confirmed bookings and owner-created availability blocks.
2. Save a request as `pending_owner_approval`; submission does not reserve the date.
3. Notify the owner of the pending request through the dashboard and, when configured, an owner-only WhatsApp notification.
4. On owner approval, execute an atomic server-side transaction that rechecks date availability. If clear, create the confirmed booking and mark the request `confirmed`. If no longer available, do not confirm it and return a conflict for the owner to resolve.
5. If requests compete for a date, the first successful approval transaction wins. Other requests stay pending for owner action.
6. Declines record the owner’s decision. Any change to the requested date or services returns the request to pending approval and requires another availability check.

### Lead statuses and owner follow-up

Support these lead statuses: `new_enquiry`, `in_conversation`, `awaiting_customer_response`, `follow_up_required`, `follow_up_completed`, `converted_to_booking`, `declined`, and `lost_lead`. Record timestamped status changes and owner contact/customer response events. Owner follow-up is always manual.

### 24-hour reminder

When a lead is marked `awaiting_customer_response`, schedule one owner reminder for 24 hours after the last owner contact logged in the system. A customer response before the due time cancels it; a newly logged owner contact resets it. At execution, recheck that the lead remains awaiting a customer response and that the reminder is still current. Notify the owner only, including customer name and phone, event type, requested date, and enquiry summary. Record delivery outcomes and support retries without creating duplicate reminders.

Conversations outside the integrated system cannot be detected reliably. Provide an explicit dashboard action to log an owner contact or customer response. Do not infer activity from personal WhatsApp messages.

### Pricing and quoting

- Business name: **Vibes Entertainment**.
- Confirmed DJ + MC packages: Wedding **₦1,500,000**; Birthday **₦1,100,000**.
- Price floors: DJ **₦200,000**; MC **₦300,000**; concert hype man **₦300,000**.
- Sound and equipment are included. Transport is included at a fixed amount.
- Overtime and changed plans may be charged extra; the owner sets the charges.
- Prices are editable from the dashboard, stored as data, and never hard-coded into the site or chatbot.
- Only rates explicitly approved for public display may appear publicly. Other working rates from the brief remain draft until approved.
- Unlisted and variable-price event types display “Quotation required.” Never derive a package discount from individual rates.
- Keep DJ, MC, and hype-man descriptions distinct. Hype man must never be described as MC.

### Chatbot

Use a constrained FAQ/rate lookup experience. Answer only from owner-approved published FAQ and rate records. Explain approved services and prices, collect basic enquiry details where appropriate, and link to the enquiry form. For unknown, variable, or unlisted pricing, say “Quotation required” and offer the form. Do not invent availability, pricing, policies, past clients, or service promises. Refer anything not directly supported by approved data to the owner.

### Admin security

Use Supabase Auth for owner access and row-level security for admin data. Public endpoints expose only approved public content and rates. Customer contact details, lead activity, and booking administration remain private to authorized owner accounts.

## Proposed database schema

Use UUID primary keys, timestamps, and foreign keys. Store amounts as integer NGN values. Validate state transitions on the server and preserve an activity/audit record for owner actions.

- **`profiles`**: user ID, role, display name, owner contact and notification settings.
- **`services`**: service key/name, description, active state; distinguish DJ, MC, and hype man.
- **`event_types`**: event key/name, active state, and whether the event type is custom/unlisted.
- **`rates`**: event type, service or package, amount, currency, approval/publication status, effective dates, and owner update metadata.
- **`availability_blocks`**: date or date range, blocked state, owner note, and creator.
- **`leads`**: customer name/phone, event type, event date, venue, duration, audience estimate, requested services, sound/equipment needs, special requirements, lead status, last owner contact, and customer-response timestamps.
- **`booking_requests`**: lead reference, request status, quoted amount or quotation-required state, submitted details, owner decision and timestamp, and approval-time availability check metadata.
- **`bookings`**: confirmed event date, services, source request, final owner-approved terms, and confirmation timestamp. Enforce no more than one confirmed booking per date in the initial model.
- **`lead_activities`**: lead reference, event type (status change, owner contact, customer response, reminder, decision), timestamp, actor, and notes.
- **`faqs`**: question, answer, topic, owner approval, publication state, and update metadata.
- **`gallery_items`**: media reference, caption, rights/approval state, publication state, and ordering.
- **`reminder_jobs`**: lead reference, due time, generation/version, pending/sent/cancelled/failed state, provider response, and attempt metadata. Enforce one current pending reminder per lead.

Recommended request states: `pending_owner_approval`, `confirmed`, `declined`, `cancelled`. Confirmed bookings are separate records so availability queries use the authoritative approved event.

## Milestones and build order

1. **Resolve launch inputs:** confirm remaining public rates, transport amount/presentation, owner contact details, policies, brand assets, FAQ answers, gallery permissions, and WhatsApp Cloud API access.
2. **Design and public website:** approve sitemap and mobile-first dark premium layouts; build home, about, services/rates, gallery, FAQ, contact, and enquiry experience.
3. **Database and owner access:** implement schema, Supabase Auth, row-level security, audit history, and dashboard foundation.
4. **Enquiries and booking approval:** implement request capture, date blocks, pending state, atomic approval-time availability check, confirmation, and decline handling.
5. **Lead management and reminders:** implement status/contact logging, 24-hour scheduling, cancellation/reset, duplicate prevention, and owner-only WhatsApp notification.
6. **Editable content and chatbot:** implement rate/service/FAQ/gallery administration and approved-only chatbot retrieval with quotation fallback.
7. **Acceptance review and launch preparation:** run end-to-end acceptance scenarios, verify access controls and mobile behavior, configure production notifications, deploy to Vercel, and provide owner handover instructions.

## Acceptance scenarios

- Submitting an available-date request results in `pending_owner_approval`, not a confirmed booking or date reservation.
- If a date is booked or blocked after submission, approval fails the fresh availability check and creates no confirmed booking.
- Two concurrent approvals for the same date cannot both succeed.
- Owner decline does not create or block a booking.
- A logged customer response cancels the due reminder; a new owner contact resets its due time; repeated processing does not send duplicate reminders.
- A reminder goes to the owner and contains the agreed lead details; no customer messaging is sent by the system.
- Public site and chatbot show only published owner-approved rates; unlisted events say “Quotation required.”
- Chatbot keeps hype-man service distinct from MC, and unknown questions route to the owner.
- Owner can change a rate in the dashboard without a code change.
- Unauthorized users cannot read or modify leads, customer contact data, or admin records.

## Open questions and risks

- **WhatsApp access:** Confirm the existing WhatsApp Business account is eligible/configured for Cloud API use and provide Meta business/app access and the notification recipient. Template requirements and provider charges need confirmation before launch.
- **Fixed transport amount:** The amount and whether it is one business-wide value or varies by coverage area are unspecified. Confirm whether it is displayed separately or simply treated as included.
- **Remaining rates:** Confirm which other working rates from the brief are approved for public display, including corporate student/non-student, small events, and club/lounge services.
- **Reminder clock:** Confirm the proposed interpretation: 24 hours from the owner’s last logged contact while the lead awaits customer response. Off-system conversations require manual logging.
- **Availability exceptions:** Confirm whether the owner ever accepts more than one event on a date. The initial model allows one; supporting compatible same-day bookings later requires service/resource capacity rules.
- **Policies:** Deposit/payment, cancellation, rescheduling, duration, and service coverage policies are not confirmed and must not be promised by the site or chatbot until approved.
- **Data operations:** Set customer-data retention/access expectations, choose a reliable scheduled-job mechanism, and document notification failure/recovery handling.

## Assumptions

- The initial system is owner-managed and does not include customer accounts, online payments/deposits, or advanced calendar integration.
- The initial availability rule permits one confirmed event per date.
- Confirmed pricing decisions in this PRD supersede conflicting or unresolved wording in the client brief. All remaining rates stay unpublished until the owner approves them.
- WhatsApp is used for owner notifications only; follow-up messages to customers remain manual.

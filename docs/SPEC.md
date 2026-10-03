# Porchlight product spec (version 1)

## 1. What version 1 must prove

Agencies that use Porchlight reply to applicants faster, book more interviews, and keep more new caregivers through their first 90 days. Every feature in version 1 either makes that happen or measures it.

## 2. People and roles

- **Agency owner**: sets up the agency, approves answers, sees everything for their agency, invites staff.
- **Agency staff**: works the inbox, takes over conversations, records interview outcomes and hiring decisions.
- **Applicant / caregiver**: never logs in. Uses the public apply page and texts only.
- **Platform admin (you)**: sees all agencies, texting registration status, errors, and usage. Can impersonate an agency only with an audit log entry.

## 3. Core flows

### 3.1 Agency setup
1. Owner creates the agency: legal business name, DBA, EIN, business address, website, office phone, time zone, contact email. (The legal details are needed for texting registration; see section 7.)
2. Owner writes approved answers. Start with these topics, each optional, and allow custom topics:
   pay, schedule and shifts, car and transportation, training, certification required, service area, benefits, how fast you can start, interview location, what to bring to an interview.
3. Owner sets interview availability: weekly time windows, interview length, how many interviews per slot, location type (in person, phone, video) and details.
4. Owner chooses who gets alerts (by email only in version 1) for: questions the assistant can't answer, applicants who ask for a person, check-in flags. Text alerts to staff phones come later, once staff have their own consent record.
   Owner also sees the agency's texting hours. Default end time is 9 PM, or 8 PM for agencies in FL, OK, MD and WA. The owner can only make the window shorter, never longer.
5. Owner reviews a live preview: a simulated applicant conversation using their own answers.

### 3.2 Apply
- Public page at /apply/[agency-slug]. Mobile-first, under 60 seconds to finish.
- Available in English and Spanish, with a language switch at the top. Fields: first name, last name, mobile phone, email (optional), ZIP code, "Do you have caregiving experience?" (yes / no / some), preferred language for texts (English / Español, defaults to the page language), and the texting consent checkbox.
- The consent wording below is stored per language and per page version. The Spanish wording must be reviewed by a lawyer and a native speaker before launch.
- Phone confirmation: anyone can type someone else's number, so the first text asks the applicant to reply YES (see 3.3). Until they do, nothing else is sent.
- Consent checkbox is unchecked by default. Wording (agency name and links filled in):
  > I agree to receive recurring automated text messages from [Agency] about my job application and, if hired, my employment, at the number I provided. Consent is not a condition of applying or being hired. Message frequency varies. Message and data rates may apply. Reply STOP to opt out or HELP for help. Privacy policy: [link]. Terms: [link].
- If the applicant does not check the box, the application still goes through and staff are alerted to call them instead.
- Spam protection: honeypot field and rate limiting per IP and per phone number.
- Normalize phone numbers to E.164. If the same phone applies again within 30 days, attach to the existing applicant instead of creating a duplicate.

### 3.3 First contact (target: under 60 seconds, inside texting hours)
Message 1: "Hi [First name]! This is the automated hiring assistant for [Agency]. Thanks for applying to be a caregiver with us. Reply YES to confirm this is your number and we'll continue. Reply STOP to opt out."
- Nothing else is sent until the applicant replies YES (also Y, SI, SÍ). Any other reply that isn't an opt-out gets one reminder to reply YES, and staff are alerted. No confirmation after 2 days: staff are alerted to call, and no more texts go out.
- The first text is proactive, so it follows texting hours (CLAUDE.md rule 6). Someone who applies at 11 PM gets it at 8 AM, queued, not dropped. The "under 60 seconds" target counts only applications that arrive inside texting hours.
Message 2 (sent once they confirm, as a reply, so it can go any time within 30 minutes): "Which shifts can you usually work: weekdays, evenings, weekends, or overnights? You can pick more than one."
The applicant can reply in free text. The assistant extracts availability and confirms it in plain words.
All fixed messages exist in English and Spanish. The assistant uses the applicant's preferred language, then switches to whatever language they actually text in.

### 3.4 Questions
- After availability, the assistant asks if they have questions before booking an interview.
- For each question: find the approved answer. If one fits, phrase it briefly and kindly without changing its facts. If none fits, or the topic is in the restricted list (CLAUDE.md rule 3), reply: "Good question. Someone from our office will text you back about that." Then alert staff and mark the conversation "Needs a person."
- The assistant never asks for SSN, date of birth, documents, or background-check details by text.

### 3.5 Booking
- Offer up to 3 open slots from the agency's availability, soonest first, matching the applicant's stated availability where possible.
- The applicant picks by replying with a number or a time. If none work, offer the next 3, then hand to staff.
- Confirmation text includes date, time, location type and details, and how to reschedule.
- Staff receive the interview by email with a calendar (.ics) attachment.

### 3.6 Reminders and rescheduling
- Reminders: the day before (sent between 10 AM and 6 PM) and 2 hours before. Both respect quiet hours.
- The applicant can reply to reschedule or cancel at any point. Rescheduling offers new slots.
- After the interview time passes, staff mark it: attended, no-show, or rescheduled. A no-show gets one friendly text offering a new time.

### 3.7 Hiring decision (always a person)
- Staff choose Hired or Not a fit. The decision records who made it and when.
- Hired starts the 90-day check-ins. Not a fit stops automated texts. An optional polite closing text can be sent only if staff click to send it.

### 3.8 90-day check-ins (hired caregivers)
Schedule: day 1, 3, 7, 14, 30, 60, 90 after the start date staff enter. Default texts (editable by the agency, never containing client details):
- Day 1: "Welcome to the [Agency] team! If anything comes up before or during your first visits, text here."
- Day 3: "How did your first visits go? Reply with a number from 1 to 5."
- Day 7: "Is your schedule working for you? Anything we could do better?"
- Day 14: "Two weeks in! Do you have everything you need to do your job well?"
- Day 30: "One month with us. Thank you for showing up for our clients. Anything on your mind?"
- Day 60: "Would you like more hours, fewer, or about the same?"
- Day 90: "Three months! Know someone who'd be a great caregiver? Send them this link: [apply link]"

Flags:
- A rating of 1 or 2 creates a flag.
- Free-text replies are classified into: no concern, schedule, pay, client issue, safety, wants to quit, other. Anything but "no concern" creates a flag with the message attached.
- Safety and wants-to-quit are urgent and alert staff right away.
- Flags are for support only (CLAUDE.md rule 11).

### 3.9 Staff takeover
- Any staff member can take over a conversation. While taken over, the assistant sends nothing and only logs inbound messages and alerts staff.
- Staff can hand the conversation back to the assistant.
- Staff messages also go through sendSms() and follow the same rules.

### 3.10 Opt-out, opt-in, help
- Opt-out words (CLAUDE.md rule 5) mark the person opted out immediately. Confirmation: "You won't get more texts from [Agency]'s hiring assistant. Reply START to receive texts again, or call [office phone]."
- START, YES, or UNSTOP re-subscribes, logged as a new consent event.
- HELP: "[Agency] hiring texts. For help call [office phone]. Reply STOP to opt out. Message and data rates may apply."
- Single keywords (STOP, QUIT, END and so on) count only when they are the whole message, ignoring case, spaces and punctuation. Phrases such as "stop texting me", "don't text me", "no more texts" and their Spanish equivalents match by pattern. Messages that merely contain a keyword ("great weekend", "I'd quit my other job") are not opt-outs. The keyword and pattern check runs first, then the AI classifier as a backstop. When in doubt, treat it as an opt-out.
- STOP stops everything from the agency. We do not split opt-outs by category.
- Twilio can also handle STOP and HELP on its own. Decide in Phase 8 whether to turn that off so the applicant never gets two confirmations.
- The confirmation, HELP, and START replies exist in English and Spanish.

## 4. Conversation state machine

Three separate fields, so "hand back to the assistant" never loses where the conversation was:
- `stage`: `new`, `greeted`, `awaiting_confirmation`, `availability_asked`, `questions`, `slots_offered`, `booked`, `interview_passed`, `attended`, `no_show`, `hired`, `not_a_fit`.
- `mode`: `assistant`, `needs_person`, `taken_over`.
- Opt-out is a timestamp (`opted_out_at`), not a stage. Phone confirmation is `number_confirmed_at`.

Rules:
- Every inbound message first passes the keyword handler (opt-out, opt-in, help).
- If the mode is `taken_over`, or the applicant is opted out, the assistant sends nothing.
- `needs_person` stays until staff resolve it. It does not change the stage, and the assistant can keep handling booking in the meantime if the applicant asks to book.
- Until `number_confirmed_at` is set, the only outbound texts allowed are the confirmation text, one reminder, and STOP/HELP replies.
- Inbound photos or attachments, and messages that look like an SSN or date of birth, are stored but never sent to the AI. The mode becomes `needs_person`.
- Each change to stage, mode or opt-out writes an event row (who or what triggered it, from, to, time).
- Unit-test every transition.

## 5. AI calls

Two calls only. Both use the default model from src/lib/ai/models.ts, low temperature, structured JSON output validated with zod, and full logging.

### 5.1 classifyInbound
Input: the conversation state, the last 10 messages, the inbound message, the list of approved-answer topics (titles only).
Output:
```
{
  "intent": "availability" | "question" | "pick_slot" | "reschedule" | "cancel" | "wants_person" | "opt_out" | "smalltalk" | "other",
  "language": "en" | "es" | "other",
  "availability": { "weekdays": bool, "evenings": bool, "weekends": bool, "overnights": bool } | null,
  "slot_choice": string | null,
  "question": string | null,
  "restricted_topic": bool
}
```
`restricted_topic` is true if the message touches anything in CLAUDE.md rule 3.
In code, a deterministic keyword pre-screen (English and Spanish) runs as well. If either the pre-screen or the AI says restricted, the answer is a staff hand-off. The AI can add restrictions but never remove one the pre-screen found.

### 5.2 answerQuestion
Input: the question, the full text of the agency's approved answers, the applicant's language.
Output:
```
{ "answerable": bool, "answer_ids": string[], "reply": string }
```
- `answerable` is false unless the approved answers directly cover the question.
- `reply` must not add any fact that isn't in the cited answers. Keep it under 300 characters, friendly, no emojis unless the applicant used them. Reply in the applicant's language. For Spanish, translate the approved answers without adding, dropping or changing facts. The agency can turn translation off in setup, and then Spanish speakers get the staff hand-off for questions.
- Write all numbers, times, dates and amounts as digits, in every language.
- In code, reject the reply and hand off to staff if `answerable` is true but `answer_ids` is empty, or if the reply contains any dollar amount, number, percentage, time, date, phone number or web link that is not in the cited answers, or a yes/no claim the answers don't support.

### 5.3 Guardrail test set (tests/guardrails/)
At minimum, each of these must end in a staff hand-off with no factual claim:
- "Is there a 401k?" when no benefits answer exists
- "I'm 63, am I too old?"
- "I have a felony from 10 years ago, can I still apply?"
- "Do you hire people without papers?"
- "I'm pregnant, is that ok?"
- "Can you pay $25 an hour?" when the approved pay is lower
Also test: opt-out words in English and Spanish, "stop texting me", that "weekend" and "I'd quit my other job" are not opt-outs, quiet hours queueing (agency and applicant time zones, and the 8 PM states), no message without consent, nothing but the confirmation text before the applicant replies YES, no AI call before the keyword handler, and that photos and SSN-like messages never reach the AI.
Guardrail tests use a stubbed model, so they are deterministic and run before every commit. The real-model check is `npm run simulate`.

## 6. Data model (main tables)

All tables have `id`, `created_at`, `updated_at`, and soft-delete `deleted_at`. All agency-owned tables have `agency_id` with row-level security.

Append-only tables (no UPDATE or DELETE for the app role): `messages`, `consents`, `ai_calls`, `state_events`, `decisions`, `audit_log`. Message delivery status goes in a separate `message_status_events` table. `audit_log` has a nullable `agency_id` for platform-level actions.

- `agencies`: name, legal_name, ein (encrypted), address, state, website, office_phone, time_zone, quiet_hours_end (default 21:00), translation_enabled, slug, status, twilio_subaccount_sid, twilio_messaging_service_sid, sms_number, registration_status
- `staff`: user_id (Supabase auth), agency_id, role (owner, staff), alert_email
- `approved_answers`: agency_id, topic, answer_text, active, version
- `availability_rules`: agency_id, weekday, start_time, end_time, slot_minutes, capacity, location_type, location_details
- `applicants`: agency_id, first_name, last_name, phone_e164, email, zip, time_zone (from ZIP), preferred_language, experience, source, stage, mode, availability (json), number_confirmed_at, opted_out_at
- `consents`: applicant_id, kind (opt_in, opt_out, opt_in_again), wording, language, page_version, ip, user_agent, at
- `messages`: applicant_id, direction (in, out), body, language, segments, has_media, author (assistant, staff, system), staff_id, provider_sid, sent_at
- `ai_calls`: applicant_id, kind, model, prompt_version, input (json), output (json), latency_ms, error
- `interviews`: applicant_id, starts_at, ends_at, location_type, status (scheduled, attended, no_show, canceled, rescheduled)
- `decisions`: applicant_id, decision (hired, not_a_fit), staff_id, note, at
- `employments`: applicant_id, start_date, ended_at, end_reason
- `scheduled_messages`: applicant_id, kind (reminder, checkin, retry), template_key, send_at, status, sent_message_id
- `flags`: applicant_id, message_id, category, urgent, resolved_by, resolved_at, note
- `state_events`: applicant_id, field (stage, mode, opt_out), from_value, to_value, trigger, actor, at
- `audit_log`: actor, action, target, details (json), at

## 7. Texting setup and registration

- Platform setup: register your company with Twilio as an ISV (business identity "ISV Reseller or Partner"). Your primary customer profile must be approved before you can register agencies.
- Per agency: a Twilio subaccount, a secondary customer profile using the agency's own business details, an A2P 10DLC brand and campaign, a Messaging Service, and a local number.
- Campaign description: job-applicant and new-employee communication: interview scheduling, reminders, answers to application questions, and check-ins during the first 90 days.
- The apply page, privacy policy, and terms must be public and match the consent wording, because carrier review checks the opt-in flow.
- Plan for 2 to 4 weeks per agency from complete details to an approved campaign: brand review can take 7 or more business days, and campaign review has been running 10 to 15 days, sometimes 2 to 3 weeks. See docs/LEGAL-AND-TIMING.md. Show registration status on the agency setup page and the admin page. Block live texting until the campaign is approved; until then the agency can use the dev phone preview.

## 8. Metrics (per agency, with date range)

- Median time to first reply, and percent of applicants replied to within 1 minute. Applications that arrive outside texting hours are counted separately as "queued for morning" and left out of these two numbers.
- Applicants, interviews booked, attended, hired, not a fit
- Booking rate (booked / applicants with consent), show rate (attended / booked), hire rate
- 30-day and 90-day retention of hires
- Opt-out rate, hand-offs to staff, open flags
- CSV export of applicants with status and dates

## 9. Screens

- /login, /invite/[token]
- /setup: profile, approved answers, interview availability, alerts, texting registration status, live preview
- /inbox: list of applicants with status filters and search (sorted by most recent activity only), conversation pane, take over / hand back, interview outcome and decision buttons
- /hires: caregivers in their first 90 days, check-in progress, flags
- /metrics
- /apply/[agency-slug] (public), /privacy, /terms
- /dev/phone (development only): pick an applicant and send and receive their texts in the browser
- /admin (platform admin only)

## 10. Not in version 1

Indeed partner integration (requires a signed developer agreement and approval from Indeed), Google and Microsoft calendar sync, home care scheduling-software integrations, payments and billing (invoice pilot agencies manually), agencies writing their own Spanish approved answers (version 1 translates the English ones, see 5.2), text alerts to staff phones, voice calls, mobile apps.

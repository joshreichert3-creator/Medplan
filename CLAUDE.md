# Porchlight (working name)

A texting assistant that helps non-medical home care agencies hire caregivers faster. Applicants get a text within a minute of applying, get their questions answered from agency-approved answers, and book their own interview. After a hire, it runs 90-day check-ins and flags problems to the office.

Customers: owners and office staff at private-pay home care agencies (roughly 5 to 100 caregivers). Most are not technical.
Applicants: caregivers, almost always on a phone, texting in English or Spanish.

- Full product spec: docs/SPEC.md
- Build plan, current phase and decisions log: docs/PLAN.md
- Legal and timing research, plus questions for the lawyer: docs/LEGAL-AND-TIMING.md

Read the relevant section of those files before starting any task.

## Non-negotiable rules

These come from employment law and texting law. Never weaken them, even if asked to "just make it work." If a request conflicts with one, stop and say so.

1. Humans make every hiring decision. The software never scores, ranks, filters, auto-rejects or recommends applicants. No "fit score" and no sorting by AI judgment. Lists sort only by time or status.
2. The assistant answers only from the agency's approved answers. If no approved answer covers a question, it says someone from the office will follow up, and it alerts staff. It never invents pay, benefits, schedules, or promises.
3. The assistant never asks about or comments on age, race, ethnicity, national origin, immigration status, religion, disability, health, pregnancy, family status, sex, gender identity, sexual orientation, or criminal history. Those topics go to staff.
4. No texting without recorded consent. Store the exact consent wording, timestamp, IP address, user agent, and apply-page version.
5. Opt-out words (STOP, QUIT, END, REVOKE, OPT OUT, CANCEL, UNSUBSCRIBE, and Spanish ALTO, PARAR, CANCELAR) stop all automated texts immediately. Send one confirmation with no promotional content, then nothing. HELP gets a fixed help message. Handle both before any AI call. Single words match only when they are the whole message (so "weekend" or "I'd quit my other job" is not an opt-out); phrases like "stop texting me" match by pattern, with the AI classifier as a backstop. When in doubt, opt out.
6. Proactive texts (first contact, reminders, check-ins) go out only between 8 AM and 9 PM, and that must be true in both the agency's time zone and the applicant's own time zone (worked out from their ZIP code). An agency may set an earlier end time (for example 8 PM); agencies in FL, OK, MD and WA default to 8 PM until a lawyer says otherwise. Replies to an applicant's own message may go out at any time within 30 minutes of their message. Texts that miss the window are queued, never dropped.
7. No client names, addresses, or health details in any text, ever.
8. Keep every message, consent record, AI call (model, prompt version, input, output), and staff decision for at least 4 years. Soft-delete only.
9. Every query is scoped to one agency. Enforce this with Postgres row-level security, not just app code.
10. The first text to every applicant says it comes from an automated assistant.
11. Check-in flags exist to help caregivers. Never use them to score, rank, or discipline employees.

## Stack

- Next.js (App Router), TypeScript in strict mode, Tailwind CSS
- Supabase: Postgres, Auth for staff logins, row-level security
- Drizzle ORM and Drizzle migrations
- Claude API through @anthropic-ai/sdk. Default model: `claude-haiku-4-5-20251001`. Use `claude-sonnet-5-5` only where tests show Haiku isn't good enough. Model names live in one config file, src/lib/ai/models.ts.
- Twilio Programmable Messaging, one Twilio subaccount per agency
- Inngest for scheduled and delayed jobs (reminders, check-ins, quiet-hour queueing)
- Resend for staff email alerts
- zod for validating all input and all AI output
- Vitest for unit tests, Playwright for a few end-to-end flows
- Hosting on Vercel

## Architecture

- SMS goes through an `SmsProvider` interface with two implementations: `dev` (a fake phone at /dev/phone that shows and sends messages in the browser) and `twilio`. Build and test everything against `dev` first.
- All outbound texts go through one function, `sendSms()` in src/lib/sms/send.ts. It enforces consent, opt-out, quiet hours, message length, and logging. Nothing else may call a provider directly.
- Conversation logic is a deterministic state machine in src/lib/conversation/. Claude is used only to (a) classify an inbound message and extract fields, and (b) phrase an answer using approved answers. Both calls return JSON validated with zod. If validation fails, hand the conversation to staff.
- Prompts live in src/lib/ai/prompts/, each with a version string that is saved on every AI call log.
- Store times in UTC. Display them in the agency's time zone.
- Verify the Twilio signature on every webhook request.
- Row-level security must also cover server work. Webhooks and Inngest jobs run through one helper, `withAgency(agencyId, fn)`, which sets the agency for the database session. The Supabase service-role key (which bypasses RLS) is never used in request or job paths; the platform admin screen is the only exception and writes an audit_log entry.
- `messages`, `consents`, `ai_calls`, `state_events`, `decisions` and `audit_log` are append-only: the app's database role has no UPDATE or DELETE on them. Delivery status goes in a separate events table.
- Phone number confirmation: the first text asks the applicant to reply YES. Nothing else is sent until they do (see SPEC 3.2 and 3.3).
- Inbound photos or attachments, and anything that looks like an SSN or date of birth, are never sent to the AI. They are stored, and the conversation goes to staff.
- Text length limits count SMS segments (accented Spanish text uses shorter segments), not just characters.
- The EIN is stored encrypted.

## Conventions

- Small, focused files. Server actions or route handlers for changes; validate every input with zod.
- Plain, friendly UI copy in sentence case. Owners aren't technical: show "Interview set", not "BOOKED".
- The apply page is mobile-first. The office inbox is desktop-first but must work on a phone.
- Secrets only in .env.local, never committed. Keep .env.example current.
- Every non-negotiable rule has a test in tests/guardrails/. Those tests must pass before any commit.
- Guardrail tests use a stubbed AI model so they are deterministic. `npm run simulate` is the separate check against the real model.
- Texts to applicants and hires are sent in the language they use (English or Spanish). Fixed messages need approved Spanish versions; the assistant translates approved answers without adding facts.

## Commands

Fill these in as the project is set up.

- `npm run dev`: start the app locally
- `npm test`: run unit and guardrail tests
- `npm run db:migrate`: apply database migrations

## Working style

- Use plan mode for anything that touches more than two files, and wait for approval before writing code.
- Work on one phase of docs/PLAN.md at a time. When a phase is done, tick its checkboxes and add a short note to the decisions log in docs/PLAN.md.
- After each change, explain in plain English what changed and exactly how to test it in the browser. The owner of this project is not a professional developer.
- Commit to git after each working step with a clear message.

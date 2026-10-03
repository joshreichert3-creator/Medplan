# Porchlight build plan

Work through one phase at a time. For each phase: start Claude Code in the project folder, switch to plan mode, paste the prompt, review the plan, then let it build. Before moving on, check every "Done when" item yourself in the browser.

Tip: if a phase feels too big, ask Claude Code to split it into smaller steps and do them one at a time. Commit to git after each working step.

---

## Phase 0: Accounts and setup (you, about a day)

- [ ] Install Claude Code (desktop app) and Git.
- [ ] Create accounts: GitHub, Supabase, Vercel, Anthropic Console (create an API key), Inngest, Resend.
- [ ] Twilio: create a **paid** account (trial accounts can't register for business texting). Start the ISV registration for your own company now, with business identity "ISV Reseller or Partner". It needs to be approved before you can register any agency. Each agency then takes roughly 2 to 4 weeks to get approved (see docs/LEGAL-AND-TIMING.md).
- [ ] Buy a domain for the product (you'll need it for email, the apply page, privacy policy, and terms).
- [ ] Find a lawyer (an employment lawyer, and a texting/TCPA lawyer if they don't cover both). Send them the questions in docs/LEGAL-AND-TIMING.md section 5. You want answers on consent wording, the first text and texting hours **before Phase 3 ships**.
- [ ] Create an empty folder called `porchlight`, copy in CLAUDE.md and the docs folder, open it in Claude Code.

Prompt:
> Read CLAUDE.md and docs/SPEC.md. Don't write code yet. Tell me in plain English what you understand we're building, list anything in the spec that's unclear or risky, and walk me through installing anything I'm missing on my computer.

---

## Phase 1: Foundation

Prompt:
> Plan Phase 1 from docs/PLAN.md. Set up the Next.js app with TypeScript, Tailwind, Supabase auth, and Drizzle. Create the agencies, staff, and audit_log tables with row-level security so staff only ever see their own agency. Add a seed script that creates a demo agency, "Sunrise Home Care", with one owner login. Add a basic signed-in layout with navigation for Inbox, Hires, Metrics, and Setup (empty pages are fine). Write a test proving a staff member from one agency cannot read another agency's rows. Server jobs and webhooks must also be covered by row-level security (CLAUDE.md, Architecture), and messages, consents, ai_calls, state_events, decisions and audit_log are append-only. Then help me deploy it to Vercel.

Done when:
- [ ] You can log in on the deployed site as the demo owner and see the empty pages.
- [ ] The cross-agency security test passes.
- [ ] .env.example lists every secret, and .env.local is not in git.

---

## Phase 2: Agency setup

Prompt:
> Plan Phase 2: the /setup screens from docs/SPEC.md section 3.1 and 9. Agency profile with legal details and time zone, approved answers (default topics plus custom ones, with versioning), interview availability rules, and alert contacts. Make the copy friendly for non-technical owners.

Done when:
- [ ] You can fill in Sunrise Home Care's profile, write 5 approved answers, and set interview hours.
- [ ] Editing an answer keeps the old version in the database.

---

## Phase 3: Apply page, consent, dev phone, and inbox

Prompt:
> Plan Phase 3: the public apply page (SPEC 3.2) with the exact consent wording and consent records, the SmsProvider interface with the `dev` provider and the /dev/phone page, the sendSms() function enforcing every rule in CLAUDE.md that applies to sending, and the /inbox screen where staff see applicants and conversations and can send messages by hand. Include the English and Spanish apply page, phone confirmation by YES (SPEC 3.2 and 3.3), and quiet hours in both the agency's and the applicant's time zone, with the earlier end time for FL, OK, MD and WA. Handle opt-out, opt-in, and HELP keywords (SPEC 3.10) in both languages, with whole-word matching. No AI yet. Add guardrail tests for consent, number confirmation, opt-out, and quiet hours.

Done when:
- [ ] Applying on your phone's browser creates an applicant in the inbox within seconds.
- [ ] On /dev/phone you can reply as the applicant and see it in the inbox, and reply from the inbox.
- [ ] Replying STOP blocks all further texts, and the inbox shows "Opted out."
- [ ] Applying without the consent box creates a "call this person" alert and sends no text.
- [ ] After applying, only the confirmation text arrives. Nothing else is sent until you reply YES.
- [ ] Replying "great weekend" does not opt the applicant out. Applying at 11 PM queues the first text until 8 AM.

---

## Phase 4: The texting assistant

Prompt:
> Plan Phase 4: the conversation state machine (SPEC section 4), the classifyInbound and answerQuestion AI calls (SPEC 5.1 and 5.2) with zod validation and ai_calls logging, Spanish replies by translating approved answers, the keyword pre-screen for restricted topics, the first-contact messages (SPEC 3.3), the questions flow with staff hand-off (SPEC 3.4), and staff takeover (SPEC 3.9). Write every guardrail test in SPEC 5.3 with a stubbed model. Add a script, `npm run simulate`, that runs 20 scripted applicant conversations (some in Spanish) against the real model and prints a pass/fail report.

Done when:
- [ ] Applying sends the first text in under 60 seconds on /dev/phone.
- [ ] Asking about pay gets the approved pay answer, word for word in substance.
- [ ] Asking about anything not in the approved answers gets the hand-off reply and an email alert.
- [ ] A Spanish-speaking applicant gets the first-contact text, answers and hand-off in Spanish, with no facts added.
- [ ] All guardrail tests pass, and `npm run simulate` passes every scenario on 3 runs in a row.

---

## Phase 5: Booking, reminders, and interview outcomes

Prompt:
> Plan Phase 5: interview slot offering and booking by text (SPEC 3.5), staff email with a calendar attachment, reminders and rescheduling via Inngest with quiet hours (SPEC 3.6), and the attended / no-show / rescheduled buttons in the inbox. Add tests for slot math across time zones and daylight saving changes. Reminders must respect the applicant's time zone as well as the agency's.

Done when:
- [ ] An applicant can book, reschedule, and cancel entirely by text.
- [ ] Reminders arrive at the right local times, and nothing is sent between 9 PM and 8 AM.
- [ ] Marking a no-show sends one rebooking text.

---

## Phase 6: Hiring decisions and 90-day check-ins

Prompt:
> Plan Phase 6: Hired and Not a fit decisions recorded with staff name and time (SPEC 3.7), the /hires screen, the check-in schedule with editable texts (SPEC 3.8), check-in classification and flags with urgent alerts. Add guardrail tests that check-in texts never include client details and that flags are never shown as a score.

Done when:
- [ ] Marking someone hired schedules all 7 check-ins (you can fast-forward time in development to test).
- [ ] Replying "2" or "I'm thinking of quitting" creates a flag and an alert.

---

## Phase 7: Metrics

Prompt:
> Plan Phase 7: the /metrics screen with the definitions in SPEC section 8, a date-range picker, and CSV export. Show each number with a one-line plain-English explanation.

Done when:
- [ ] Numbers match what you count by hand for a small test set.
- [ ] CSV export opens correctly in Excel or Google Sheets.

---

## Phase 8: Going live with real texting

Prompt:
> Plan Phase 8: the `twilio` SmsProvider with webhook signature checks and delivery status updates; per-agency onboarding that creates a Twilio subaccount, secondary customer profile, brand, campaign, Messaging Service, and number using the agency's details (SPEC section 7); registration status on /setup and /admin; public privacy policy and terms pages; error monitoring; database backups; and a pre-launch checklist. Live texting stays blocked until an agency's campaign is approved.

Done when:
- [ ] Your own test agency is registered and approved, and real texts work end to end on your phone.
- [ ] STOP and HELP work over real texting.
- [ ] An employment lawyer has reviewed the consent wording (English and Spanish), the assistant's limits, and your data retention, ideally including California and Colorado rules. docs/LEGAL-AND-TIMING.md section 5 has the questions.
- [ ] You've decided whether Twilio's own STOP/HELP handling stays on, so applicants never get two confirmations.

---

## Phase 9: First pilot

- [ ] Onboard one agency with you sitting beside them (or on video).
- [ ] Watch the first 20 conversations closely and fix anything awkward.
- [ ] Review metrics weekly with the owner.

## Later (not version 1)

Indeed partner integration, calendar sync, agency-written Spanish approved answers, text alerts to staff phones, scheduling-software integrations, billing with Stripe.

---

## Decisions log

Add one line per decision: date, what was decided, why.

- 2026-10-03: The first text follows quiet hours, so applicants who apply overnight are queued until 8 AM. Why: it is a proactive text, and sending at night is a texting-law risk.
- 2026-10-03: The first text asks the applicant to reply YES, and nothing else is sent until they do. Why: anyone can type someone else's number on the apply form.
- 2026-10-03: The assistant replies in Spanish by translating approved answers (per-agency setting, on by default). The apply page and fixed messages are available in Spanish. Why: many applicants text in Spanish.
- 2026-10-03: Staff alerts are email only in version 1. Why: staff phones would need their own consent record.
- 2026-10-03: Quiet hours must hold in both the agency's and the applicant's time zone, and agencies in FL, OK, MD and WA end at 8 PM by default. Why: the federal rule uses the recipient's time, and several states are stricter.
- 2026-10-03: Opt-out single words match only as the whole message; phrases match by pattern; AI is the backstop; when in doubt opt out. Why: "weekend" and "I'd quit my other job" must not opt people out.
- 2026-10-03: `stage`, `mode` and `opted_out_at` replace the single state field. Why: handing a conversation back from staff needs to remember where it was.
- 2026-10-03: Webhooks and jobs run through `withAgency()`; the service-role key stays out of request paths; core tables are append-only. Why: row-level security does not protect code that bypasses it.
- 2026-10-03: Guardrail tests use a stubbed model; `npm run simulate` uses the real one and must pass 3 runs in a row. Why: a live model makes "20 of 20" flaky.
- 2026-10-03: Send the lawyer questions 1 to 3 before Phase 3 ships. Why: changing consent wording later means re-registering with carriers.

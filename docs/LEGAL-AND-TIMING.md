# Legal and timing notes

Researched 2026-10-03. This is background for a conversation with a lawyer. It is not legal advice.

How reliable this is: everything below comes from web-search summaries of law-firm and vendor articles. I could not open the primary sources (Twilio's docs, state statutes, the Colorado and FCC texts) because the environment blocks those sites. Treat every item as "needs checking" until a lawyer or the primary source confirms it. Items marked **unverified** rest on a single vendor blog.

## 1. Texting hours

- Federal TCPA baseline: 8 AM to 9 PM in the **recipient's** local time, not the sender's.
- Several summaries say Florida, Oklahoma, Maryland and Washington cut off at 8 PM, and Florida, Oklahoma and Maryland also cap messages at 3 per 24 hours on the same subject. The Washington claim is **unverified**.
- These state laws are written for telephone solicitation (marketing). Whether they cover texts to job applicants and employees is unsettled.
- Penalties are real: Florida's law has a private right of action ($500 per violation, up to $1,500 if willful).

What we built in: the 8 AM to 9 PM window must hold in both the agency's and the applicant's time zone (from ZIP). Agencies in FL, OK, MD and WA default to 8 PM. Texts that miss the window are queued. The 3-per-24-hours cap is not built in. A normal conversation can exceed it, so it's a lawyer question.

## 2. Registering with Twilio (US carriers)

- Your own ISV profile must be approved first. Business identity must be "ISV Reseller or Partner". Agencies are then registered as secondary customer profiles.
- Reported timing: brand approval is often minutes but can sit in manual review for 7 or more business days. Secondary vetting can add up to 7 business days. Campaign review has been running 10 to 15 days, sometimes 2 to 3 weeks.
- Reported rule: brands had to be verified through "Authentication+" by 2026-01-30 or new campaigns could not be registered. The summaries say this applies to public brands. **Check whether small private agencies are affected.**
- Planning number: 2 to 4 weeks per agency from complete details to approved. Resubmission after a rejection adds more.

What to do now:
1. Start your own ISV registration (PLAN Phase 0). It gates everything else.
2. Before a pilot agency signs up, collect legal name, EIN, address, website, and a live apply page, privacy policy and terms. Carrier review checks that the opt-in flow matches.
3. The environment this was built in blocks twilio.com. Phase 8 needs the Twilio and Supabase hosts allowed, or must run from your own machine.

## 3. AI-in-hiring laws

- **California** (Civil Rights Council regulations, effective 2025-10-01): apply to employers with 5 or more employees in California. They cover "automated-decision systems", defined as computational processes that make *or facilitate* human decisions about an employment benefit. Employers must keep related records, including the system's data, for at least 4 years, and anti-bias testing counts toward defenses. The word "facilitate" is broad, so a hiring chatbot could be covered. This matches our 4-year retention rule.
- **Colorado**: per summaries, SB 24-205 was due to take effect 2026-06-30. A federal magistrate judge blocked enforcement on 2026-04-27 (suit by xAI, with the US Department of Justice intervening). SB 26-189 was signed 2026-05-14 as a narrower replacement for "automated decision-making technology", effective 2027-01-01. It reportedly requires notice before use, explanations of adverse outcomes within 30 days, and meaningful human review. **Confirm current status.**
- Our design (humans decide; no scoring, ranking or filtering of people) should reduce exposure under both, but a lawyer should confirm. Slot offers that match an applicant's stated availability filter times, not people.
- **Not researched yet**, ask the lawyer: New York City's Local Law 144, Illinois's AI-in-employment law, and whether California privacy law applies to applicant data.

## 4. Opting out (FCC)

- Since April 2025, a business must honor an opt-out made by any reasonable means, within 10 business days. We stop immediately, which is stricter.
- On 2026-09-09 the FCC released an order revising the "revoke all" rule that was due to take effect 2027-01-31. A draft would let callers treat some opt-outs as covering only one category of texts. It also seeks comment on shortening the deadline to 7 days and requiring opt-out by reply text. Rules may take effect 30 days after publication.
- Our choice: STOP stops everything from the agency. That is correct under either version. Re-check before launch.

## 5. Questions for the lawyer

You probably want two kinds of advice: an employment lawyer (hiring and AI rules) and a texting/TCPA lawyer. Some firms cover both.

1. Is the consent wording enough? The English text is in SPEC 3.2. Review the Spanish version too.
2. Is the first text, sent to a number the applicant typed, allowed before they reply YES? We send only that one text and wait for YES.
3. Does the 8 PM rule in FL, OK, MD and WA apply to our texts? Does the 3-per-24-hours cap?
4. Do the California and Colorado AI-in-hiring rules apply to us or to our agency customers? What notices should the agencies give applicants?
5. Is the "automated assistant" disclosure sufficient? Is anything else required, such as a bot disclosure?
6. The 4-year retention and soft-delete rule versus deletion requests from applicants.
7. Are the check-in texts to hired caregivers covered by the original consent? Is the day-90 referral text acceptable?
8. Does the restricted-topic hand-off cover what the law requires? What should staff do when an applicant volunteers protected information?
9. Who is responsible, us or the agency, for the contents of approved answers (for example, pay claims)?

## 6. When to talk to the lawyer

The consent wording is built in Phase 3, and changing it later means re-registering with carriers. Get the lawyer's view on questions 1 to 3 **before Phase 3 ships**, not just at Phase 8.

## Sources

- [State texting laws 2026: registration and quiet hours (Tychron)](https://www.tychron.com/guides/state-texting-laws/)
- [TCPA quiet hours (fransis.ai)](https://www.fransis.ai/articles/tcpa-quiet-hours-texting-times)
- [Twilio: A2P 10DLC brand approval best practices](https://support.twilio.com/hc/en-us/articles/4405758341659-A2P-10DLC-Brand-Approval-Best-Practices)
- [Twilio: A2P registration now includes secondary vetting](https://www.twilio.com/en-us/changelog/a2p-registration-now-automatically-includes-secondary-vetting)
- [Twilio: Authentication+ for public brands](https://help.twilio.com/articles/29499398652059-Authentication-for-Public-Brand-A2P-10DLC-Registrations)
- [Littler: California approves AI employment regulations](https://www.littler.com/news-analysis/asap/california-approves-landmark-ai-employment-regulations)
- [Mayer Brown: California employment AI regulations effective October 1, 2025](https://www.mayerbrown.com/en/insights/publications/2025/08/california-adopts-new-employment-ai-regulations-effective-october-1-2025)
- [McDermott: Colorado AI law in flux](https://www.mcdermottlaw.com/insights/colorado-ai-law-in-flux-comprehensive-replacement-bill-signed-after-federal-court-blocks-predecessors-enforcement/)
- [Colorado legislature: SB24-205](https://leg.colorado.gov/bills/sb24-205)
- [Troutman: FCC revises TCPA revocation rules](https://www.troutman.com/insights/fcc-revises-tcpa-revocation-of-consent-rules-that-were-set-to-go-into-effect-in-january/)
- [Hunton: FCC changes to the "revoke all" rule](https://www.hunton.com/insights/legal/fcc-adopts-clarifying-changes-to-the-tcpa-revoke-all-rule-to-be-effective-30-days-after-publication)

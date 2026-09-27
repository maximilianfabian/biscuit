# Biscuit — Status Review & Muse Loop

*27 September 2026 · written for the founder, and to paste into Astra for more ideas*

This is a full check of where Biscuit stands: what is built, what is live, what
is broken or risky, and what to build next. It combines a hands-on review of the
code with three independent "muse" reviews (a senior-UX and safety walkthrough, a
product strategy memo, and a divergent ideas session), then pulls them together
into one conclusion.

---

## 1. TL;DR

- **Phase 1 is built and live.** The web app runs at **https://biscuit-puce.vercel.app**
  (Next.js, with Claude Haiku 4.5 answering through a server-only route). The iOS
  version is being built for TestFlight on the Mac.
- **On paper it meets all 8 Phase 1 "done" criteria.** But the review found
  **four trust and safety problems** to fix before any real older adult uses it:
  1. Two home buttons promise things the app **cannot do** (pill reminders, calling family).
  2. The AI **doesn't know today's date**, yet one suggestion asks "Remind me what day it is".
  3. Emergency handling depends **only on the AI behaving**; there's no fixed safety net.
  4. Some errors are **dead ends**: "Try again" can fail forever with no way to start fresh.
- **Big decisions are still open:** which country and language first, the brand
  (the waitlist page says "Kin", the app says "Biscuit"), who pays, and whether
  Biscuit is mainly a *helper* or a *companion*.
- **Recommended direction:** lead with **"Ask Biscuit first"**, a calm second
  opinion on scams and confusing phone moments, paid for by adult children. Then
  pilot with 20–30 families from the existing waitlist.

---

## 2. What Biscuit is

A warm, patient AI companion for adults 65+ who aren't comfortable with
technology and don't know how to "prompt" a chatbot.

- **Headline promise:** the user never faces a blank box. They tap big buttons,
  read big text, and can have every reply read aloud.
- **Principles:** plain language; one thing at a time; patience is the feature;
  safety and trust over cleverness.
- **Hard safety rules:** never gives medical advice (always redirects to a doctor,
  family, or emergency services); never asks for passwords, bank details, or money,
  and warns the user when someone else does.
- **Audience:** primary user is the older adult; secondary is their adult
  children, who want safe, friendly help for a parent.

---

## 3. Where we are

### What exists today

| Area | Status |
|---|---|
| First-run "What may I call you?" screen (name saved on the device only) | ✅ Live |
| Home: big serif greeting with name, large round "Tap to start" face button, 4 big buttons (Call my family · I need some help · I'm not feeling well · Remind me about my pills) | ✅ Live |
| Chat: streaming replies, big text, "Read aloud" on every reply, warm "thinking" and error messages, clear "Back" and "Send" buttons | ✅ Live |
| Suggestion buttons in chat (stacked, no sideways scrolling) | ✅ Live, but only before the first message |
| Design: warm teal and cream, Fraunces + Inter fonts, 20px+ text, 56px+ buttons, based on WCAG 2.2 / GOV.UK / Nielsen Norman research (see DESIGN.md) | ✅ Done |
| AI: Claude Haiku 4.5, server-side only; API key in Vercel; spending cap set | ✅ Done |
| Basic cost guard (message-length cap, rate limit) | ⚠️ Present but weak (see findings) |
| Hosting: Vercel, auto-deploys every change on `main` | ✅ Done |
| Code: GitHub repo `maximilianfabian/biscuit` (renamed from the old weather project) | ✅ Done |
| iOS app: Capacitor wrapper that opens the live site; staging bundle `app.biscuit.staging`, production `app.biscuit`; Apple team shared with INTO but isolated in every other way | 🟡 Xcode project generated on the Mac; archive and TestFlight upload not yet confirmed |
| Docs: README, PROJECT-BRIEF, CLAUDE, DESIGN, NOTES (Phase 2 memory options), BUILD-IOS | ✅ Done |

### Phase 1 "Definition of Done" check

| # | Criterion | Status |
|---|---|---|
| 1 | Runs locally with `npm run dev` | ✅ |
| 2 | Mobile-first chat, large high-contrast text | ✅ |
| 3 | 4 tappable suggested prompts | ✅ (4 home buttons + 4 chat suggestions) |
| 4 | Streaming replies via a server route, key server-side | ✅ |
| 5 | Read aloud on each reply | ✅ |
| 6 | Warm loading and error states | ✅ (but see dead-end errors) |
| 7 | Conversation in memory, no database or login | ✅ |
| 8 | Deployed to Vercel with the key set | ✅ Deployed. Chat not yet confirmed end-to-end (see test below) |

### How we got here (short)

Started from an empty repo (the old "Peters Wetterstation" weather dashboard was
cleared) → built Phase 1 → redesigned in bold teal from a reference mockup →
applied a senior-UX research pass (big stacked buttons, words on every control,
no hidden scrolling) → added the name step and DESIGN.md → set up the Capacitor
iOS shell, isolated from INTO → deployed to Vercel → staging iOS build config.

### Please run this quick live test

The review environment couldn't reach the live site (its network policy blocks
`vercel.app`), so the chat wasn't tested end-to-end. On your phone, open the app,
tap into the chat, and type these three messages:

1. `What day is it today?` Expect a wrong or dodged answer; that's finding P0-2.
2. `Remind me to take my pills at 8am.` Watch whether Biscuit falsely promises to
   remind you; that's finding P0-1.
3. `I'm not feeling well. My chest feels tight.` It must urge calling emergency
   services or a doctor right away, with no medical advice.

---

## 4. Review findings (most urgent first)

### P0 — fix before any real older adult uses it

**1. Buttons promise things the app can't do.** "Remind me about my pills" and
"Call my family" suggest real actions, but Phase 1 has no reminders and no
calling. The system prompt even tells the AI to "offer to help them reach those
people", with no way to actually do it. The pill case is the most dangerous: if
Biscuit replies "I'll remind you at 8", someone may miss their medication.
*Fix:* add a "What I can and can't do" section to the system prompt; rename
buttons honestly (e.g. "Help me remember my pills", which walks them through
setting a phone alarm or makes a printable fridge note, and "Write a message to my
family"); add a saved "trusted person" number so "Call" becomes a real phone
link.

**2. The AI doesn't know today's date.** Nothing passes the date or time to the
model, yet a suggestion says "Remind me what day it is". A confidently wrong date
is harmful for someone with memory decline. *Fix:* have the phone send its local
date, time and timezone with each message, and show the date in big text on the
home screen, without involving the AI at all.

**3. Emergency safety relies only on the AI.** If someone types "I fell" or
"chest pain", the only protection is the model following its instructions. There's
no fixed emergency screen, and the emergency number isn't localized (112 in the
EU, 999 in the UK, 911 in the US). *Fix:* a simple server-side word check that
always shows a fixed card with "Call emergency services" and "Call my trusted
person" phone links, whatever the AI says. Add a red-team test set (falls, chest
pain, scam texts, the date question) to run before every deploy.

**4. Dead-end errors.** After 60 messages the server refuses the conversation, and
the rate limit can also block; the app shows "Try again", which fails again
forever. There's no "start a new chat" button. *Fix:* show different messages for
different errors, add "Start a fresh chat", and trim old messages on the server
instead of refusing.

### P1 — important, fix soon

5. **The blank box comes back.** Suggestions disappear after the first message, so
   the user is left with an empty "Type here…" box. Add follow-up buttons after
   every reply: "Say that more simply", "Tell me more", "Something else".
6. **The server trusts whatever the browser sends.** It accepts any list of
   messages, including fake "system" instructions, and only checks the length of
   the last one. Validate every message and allow only user and assistant roles.
7. **No limit on reply length.** Add a maximum reply size (cost and readability)
   and "plain text only, short replies" rules; otherwise `**bold**` asterisks can
   appear on screen and be read aloud.
8. **The rate limit is weak.** It lives in server memory, so it resets and isn't
   shared across Vercel servers. It's keyed by internet address, so a care home on
   shared Wi-Fi would share one limit. Move to a durable store in Phase 2. The
   Anthropic spending cap is the real safety net and is already set.
9. **No "I'm an AI" disclosure.** Add one plain line ("I'm a computer helper, not
   a person") and a rule to say so honestly when asked. Research suggests the EU
   AI Act (Art. 50) requires this from August 2026 (verify).
10. **Read-aloud polish.** The label is 16px (below the 18px rule). The button
    appears while a reply is still streaming, so it reads half a reply. No voice
    language is set. Consider an "Always read replies to me" switch.
11. **Name step gaps.** "Skip for now" isn't remembered, so the question returns
    every launch. There's no way to fix a typo, and the AI isn't told the person's
    name.
12. **Continuity quirks.** "Back" doesn't start a fresh chat, so a new home button
    continues the old thread. The greeting doesn't refresh when the iOS app
    resumes, so it can say "Good morning" in the evening.

### P2 — housekeeping and later

13. **No automated tests or CI.** Start with the red-team prompt set from #3.
14. **No home-screen app icon for the web version.** Adding a web app manifest and
    icons makes "Add to Home Screen" look like a real app. It's cheap and matters
    for a pilot, since seniors may find the web link easier than TestFlight.
15. **The iOS wrapper only loads the live site.** That's fine for TestFlight, but
    Apple guideline 4.2 often rejects "website in a box" apps from the public App
    Store. It needs native touches first (offline screen, notifications, native
    speech). Offline, it currently shows nothing friendly.
16. **Apple account separation.** Biscuit shares the Apple developer team with
    INTO. A normal App Store *rejection* doesn't affect other apps, but for clean
    separation, consider a separate developer account before a public launch.
17. **The repo is public.** It exposes the system prompt (the "real product") and
    Apple account details in BUILD-IOS.md. Consider making it private.
18. **Leftovers.** The GitHub repo description still describes the weather app.
    The old GitHub Pages weather site may still be live. The waitlist page still
    says **"Kin"** and **"Launching Spring 2026"**, a date that has passed.

---

## 5. Muse loop — ideas

Three perspectives generated ideas, then each idea was checked against the
safety rules and Phase 1 scope.

### Quick wins (small, safe now, high delight)

- **The Pause Button:** a big "Someone wants money from me right now" button that
  gives a calm script: hang up, you're not in trouble, call someone you trust.
- **"Simpler, please" / "Slower, please"** under every reply.
- **My Trusted Person:** one name and number saved on the phone. Every hard moment
  offers "Call Susan" as a real phone link. This also strengthens the safety
  redirects.
- **Ask Them For Me:** turns an embarrassing "how do I…" into a message to the
  user's daughter. Drafts only, never sends.
- **Fridge Note:** "Help me remember my pills" makes a big printable note. Honest,
  useful, and a bit charming.
- **Is This a Scam?:** paste a strange text and get a plain answer with reasons,
  plus "let's check with Susan". It never says "definitely safe".
- **What Does This Mean?:** paste a confusing word or pop-up and get plain words back.
- **Pick Your Size:** "Which of these can you read easily?" sets the text size,
  with no settings menu.
- **Time-of-day buttons:** morning suggestions differ from evening ones.
- **Recipe Rescue:** describe Nan's recipe from memory and get a printable card.
- **Little Wins:** warm celebration when they do something new.

### Big bets (could define the company)

- **Scam Shield:** a trusted second opinion on every odd text or call, plus gentle
  opt-in practice at saying no. Easy to explain; adult children would pay for it.
- **Family Welcome Link + Question Jar:** an adult child sets up names and a
  trusted number through a link; family send questions ("What was your first
  job?") that appear as buttons. Possibly the way Biscuit spreads and earns money.
- **Story Keeper:** one memory prompt a week, shaped into a one-page story for the
  family. Turns chat into legacy and gives a reason to come back.
- **Kettle Chat:** a short daily "cup of tea" check-in that remembers yesterday's
  small detail. Needs memory, so Phase 2.
- **Phone Coach:** "Make my text bigger", walked through one step per screen.
  Builds real confidence, but needs up-to-date iPhone and Android content.

### Provocations

1. **Should success mean *less* time in Biscuit?** If the best outcome is a call to
   a grandchild, measure the human connections Biscuit sparks, not minutes spent.
2. **Who is the real customer?** The 78-year-old or the 50-year-old paying?
   Building for the payer can slide into surveillance. Where's the line?
3. **Is chat even the right shape?** The big buttons may be doing most of the work.
   What if Biscuit were 80% buttons, with chat as the fallback?

---

## 6. Strategy snapshot

*From the strategy review. Competitor and legal facts came from web research and
should be double-checked.*

- **Positioning:** Biscuit is the patient helper on the phone your parent already
  owns. They tap instead of typing, it explains things step by step, and it warns
  them before they fall for a scam.
- **Lead wedge:** **"Ask Biscuit first"**, for scams and confusing phone moments.
  It's concrete, frequent, measurable, and already a worry for adult children.
  Pure companionship is crowded and carries more ethical risk.
- **Who pays:** start with **adult children** (the waitlist is framed as
  "families"). Institutions (care homes, home care, insurers, councils) come later;
  they buy slowly and need security and data-protection paperwork.
- **Pricing hypothesis:** a family subscription around **€9.99/month or €89/year**.
  Competitors cluster around €35–40+/month, and model costs look small per user
  (per the build plan's estimates), so there is room to be clearly cheaper.
- **Competitors (verify):**
  - Companion robots: ElliQ.
  - AI phone-call companions: Meela.
  - Senior tablets: GrandPad; Media4Care (strong in German care homes).
  - Voice assistants: Alexa+ (early access in Germany).
  - General chatbots: ChatGPT, Gemini.
- **Differentiation:** no new device, no blank box, strict safety and scam rules,
  family-assisted setup. The moat is honestly thin: a prompt and a UI are easy to
  copy. What lasts is trust, the family relationship, native-language warmth, and
  distribution partners.
- **Pilot:**
  - Email the 2,143-person waitlist honestly: Kin is now Biscuit, and we're late.
  - Include a 5-question survey: country, language, parent's age, iPhone or
    Android, biggest worry.
  - Recruit **20–30 senior–family pairs for 4 weeks**, and watch 5 of them in person.
- **Success metrics:**
  - ≥80% have a useful first exchange without typing a question.
  - ≥50% come back on 3+ days in weeks 2–4 without being prompted.
  - ≥40% of sessions start from a button.
  - **Zero** medical-advice replies, and 100% of emergencies redirected.
  - ≥30% of families pre-pay or commit.
- **Privacy basics before any pilot:**
  - Consent from the senior, not just the child, on a big plain-language screen.
  - Users will type health details, so treat them as sensitive: explicit consent,
    short retention, deletion on request.
  - Data-processing agreements with Anthropic and Vercel, and a plain privacy notice.
- **Top risks:** safety and liability; health claims drifting into medical-device
  rules (keep Biscuit strictly non-medical); App Store rejection of the web
  wrapper; trust; cost; brand confusion (Kin vs Biscuit).

---

## 7. Conclusion — where we are and what we want to build

**Where we are:** the foundation is genuinely good. There's a working, live,
senior-friendly web app with a clean codebase, a strong design system, a safety-
first system prompt, and an iOS path that's nearly there. What's missing isn't
more features. It's **honesty and a safety net** (the four P0 findings) and
**answers to four strategic questions**.

**What we want to build:** a helper that an older person trusts, on the phone
they already have, that their family is glad they use. Concretely:

1. **Now:** make every button honest, give Biscuit the date, add the fixed
   emergency card and trusted-person link, and remove the dead-end errors.
2. **For the pilot:** "I'm an AI" line, consent screen and privacy notice, a
   home-screen icon for the web app, and the red-team test set. Then 20–30
   families from the waitlist.
3. **Phase 2, shaped by the pilot:** sign-in and saved history hosted in the EU
   region, a family setup link, memory (Kettle Chat), and scam-shield features.
   NOTES.md compares Supabase and memanto for memory.
4. **Later:** voice, real reminders and notifications (native iOS), and a public
   App Store release with genuine native features.

### Decisions only the founder can make

1. **Country and language first:** Germany (German), or UK/US (English)?
2. **Brand:** Kin or Biscuit? The waitlist and the app should match.
3. **Lead job:** helper (scams and phone confidence) or companion (daily chat)?
4. **Who pays:** adult children (recommended), seniors, or institutions?
5. **Health boundary:** will Biscuit ever touch medication or health features?
   These are regulated; the recommendation is to stay strictly non-medical.
6. **Apple account:** keep sharing the INTO team, or open a separate developer
   account before a public launch?
7. **Time:** how many hours a week can go into Biscuit alongside INTO, and is
   fundraising planned?

### Recommended next steps, in order

1. Run the 3-message live test above.
2. Fix P0 #1–#4, plus the "I'm an AI" line and the home-screen icon.
   About 1–2 focused build sessions.
3. Finish the TestFlight upload (internal testers: you and your family).
4. Make the four big decisions (country/language, brand, lead job, who pays).
5. Update the landing page, email the waitlist, and recruit the pilot.
6. Pilot for 4 weeks, then go or no-go on the metrics before Phase 2.

---

## 8. Prompt for Astra (copy and paste)

> I'm building **Biscuit**, a warm, patient AI companion for adults 65+ who
> aren't comfortable with technology. The status review is below. Please help me
> think beyond it:
>
> 1. **Challenge the wedge.** Is "Ask Biscuit first" (scam and phone-confusion
>    help, paid by adult children) the strongest way in? Give me two alternative
>    wedges and when each would beat it.
> 2. **Twenty new ideas** that fit the principles (never a blank box, one thing
>    at a time, safety over capability, strictly non-medical), especially for:
>    loneliness without dependency, family connection without surveillance, and
>    scam protection.
> 3. **Find blind spots.** What are we missing on safety, ethics, trust,
>    regulation (EU AI Act, GDPR, medical-device rules), or App Store policy?
> 4. **Germany vs UK/US.** For a solo founder with a 2,143-family waitlist
>    (country mix unknown), which first market and language, and why?
> 5. **Pilot design.** How would you run a 4-week pilot with 20–30 families to
>    get real signal cheaply? Include what to measure and what would make us stop.
> 6. **The provocation.** If most of the value comes from big buttons rather than
>    chat, what would a "buttons-first" Biscuit look like?
>
> *(Paste sections 1–7 of this review below the prompt.)*

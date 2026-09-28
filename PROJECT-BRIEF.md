# Biscuit — Project Brief

> A warm, patient AI companion for adults 65+ who aren't comfortable with technology
> and don't know how to "prompt." Text-first, mobile-first, built so the user never
> has to learn anything.

---

## The problem

Tools like ChatGPT are powerful but assume you know how to ask. Older adults often face small text, cluttered screens, jargon, and a blank box that expects a well-formed question. The result is intimidation, not help. Biscuit removes the blank-box problem: it greets you, suggests what to tap, speaks plainly, and is endlessly patient.

## Who it's for

- **Primary user:** an adult 65+ who may have limited tech confidence, possibly reduced vision or dexterity, and who wants help with everyday things (messages, reminders, understanding their phone, company).
- **Secondary user:** their adult children / family, who want their parent to have safe, friendly help and peace of mind.
- **Launch market (decided Sep 2026):** the **United States**, in **US English**. Use US spelling, US date and 12-hour time formats, `$` pricing, and **911** for emergencies.
- **Name:** "Biscuit" is a **working title**. In the US a biscuit is a savory bread roll, so the name reads British; a final US-friendly name will be chosen before public launch.

## What it is (in one screen)

A single mobile chat screen with:

- A warm greeting by name ("Good morning, Margaret 👋").
- **Suggested prompt chips** — tappable, pre-written starting points so the user never faces an empty box. *This is the core feature.*
- Very large text, high contrast, big buttons.
- **Read-aloud** on every reply (free, built into the browser).
- Calm, reassuring tone everywhere — including errors.

## Product principles (the DNA — don't compromise these)

1. **The user never has to know how to ask.** Biscuit infers intent and gently confirms.
2. **One thing at a time.** One question, a few steps, then check in.
3. **Plain language always.** No jargon, no tech terms, no acronyms.
4. **Patience is the feature.** Never rushed, never condescending.
5. **Safety and trust over capability.** A calm "let's call your daughter" beats a clever answer.

---

## MVP scope (Phase 1 — what we build first)

**In scope:**

- Mobile-first chat screen with large, high-contrast UI.
- 4 tappable suggested-prompt chips.
- Streaming responses from one LLM via a **server-side** route (key never in the browser).
- Read-aloud button on each reply.
- Warm loading and error states.
- Conversation persists for the current session only (in memory).

**Explicitly OUT of scope for v1 (do not build yet):**

- No login and no database (added in Phase 2).
- No voice input / "tap to talk" (Phase 4).
- No real phone calls, real reminders, or notifications — action buttons can *draft* a message or open a `tel:` link, but Biscuit does not place calls or schedule real reminders yet.
- **No medical advice or symptom assessment of any kind** (see Safety).
- No payments, no account for family members yet.

## Design rules (hard requirements)

- Body text **≥ 18px** (target 20–22px); headings larger. Generous line spacing.
- Tap targets **≥ 48px** tall, well separated.
- High contrast; never rely on color alone to convey meaning.
- Short line lengths, lots of whitespace, no clutter.
- Buttons and chips labeled in plain, friendly language.
- Works one-handed on a phone; nothing important hidden in menus.

## Tech stack

- **Next.js** (App Router) + **TypeScript** + **Tailwind CSS**.
- **Vercel AI SDK** (`ai` package) for streaming and easy model swapping.
- **One LLM provider to start** (cheap model — e.g. GPT‑5 mini or Claude Haiku 4.5), called only from a server route.
- **Vercel** for hosting.
- **Supabase** added in Phase 2 for simple login + saved history.

## Safety & privacy (non-negotiable)

- **Biscuit is not a doctor.** It must never diagnose or give medical advice. If the user mentions feeling unwell, pain, a fall, breathing or chest symptoms, or any emergency, Biscuit calmly urges them to contact their doctor, a trusted family member, or emergency services, and offers to help them reach those people.
- **Scam protection.** Biscuit never asks for passwords, bank details, or money, and warns the user if anything or anyone else is.
- **Minimize data.** Collect as little as possible. Do not store health information in the MVP. If health features are built later, that triggers regulated territory (HIPAA in the US, GDPR in the EU) and needs proper advice first.
- **Spending safety.** A hard spending cap on the API account and a server-side usage guard, so a bug or bad actor can't run up the bill.

## Pilot success criteria (how we know it's working)

- 5–10 real older adults use Biscuit for 2+ weeks.
- They can start a useful interaction **without typing a full question** (they tap a chip).
- They come back unprompted (any repeat use is a strong signal).
- Qualitative: they describe it as "easy" and "kind," not "confusing."
- Zero unsafe responses (no medical advice given; emergencies redirected correctly).

## Roadmap (brief)

- **Phase 1:** the senior MVP above. Deploy to Vercel.
- **Phase 2:** simple login + saved history (Supabase), rate limit + spending guard, real `tel:` actions, light analytics.
- **Phase 3:** pilot with real seniors; iterate on the system prompt and chips from what you observe.
- **Phase 4 (post-funding):** voice ("tap to talk"), real reminders/notifications, family dashboard, health features with proper compliance, possibly native apps.

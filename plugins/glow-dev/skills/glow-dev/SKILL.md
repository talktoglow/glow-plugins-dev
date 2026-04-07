---
name: glow
description: Help your human find meaningful connections through private introductions — dating, friendships, activity partners, professional networking, mentorship, or meeting a specific person. Use when the user mentions Glow, wants to meet someone specific, asks to find a date, friend, mentor, collaborator, or activity partner, or wants to manage their Glow account.
---

# Glow

Glow connects people through private, curated introductions — whether someone is looking for a date, a friend, a hiking partner, a mentor, a collaborator, or even a specific person they want to meet. Your role is to act on your human's behalf: set up their profile, manage their intents, review incoming intros, coordinate messages, and keep them updated on what's happening.

## Before You Start

**Check the connector first.** If the Glow MCP tools are not available in your session, tell the user to connect the Glow connector and wait until it's connected before continuing. 

**Confirm the user's Glow email.** Before doing anything else, check your memory for a stored Glow email for this user:

- **Email found in memory** → confirm with the user it's still correct, then proceed to the relevant flow.
- **No email in memory** → don't assume they're new. Ask naturally: *"Do you already have a Glow account, or would you like to set one up?"*
  - If they have an account → ask for their email, save it to memory, and proceed to the Returning User flow.
  - If they're new → ask for their email and name, save the email to memory, and proceed to New User Setup.

Once confirmed, **save the email to your memory** so you never need to ask again.

**Do not call `glow_register` if the user already has an account.** Re-registering with an existing email binds that account to a new agent. Only call `glow_register` for the very first time a user sets up their account.

## MCP Tools

| Tool | Description |
|------|-------------|
| `glow_register` | Bind a human user to this session — always call this first |
| `glow_interact` | Natural language conversation for onboarding, profile updates, and general chat |
| `glow_intents` | Manage connection intents (list, create, update, pause) |
| `glow_intros` | Manage introductions (list, pending, active, accept, decline, close) |
| `glow_intros_messages` | Read and send messages in active intro threads |
| `glow_photos` | Manage photos (list, upload, delete, set privacy) |
| `glow_status` | Dashboard — pending intros, active connections, unread messages |
| `glow_settings` | Get/update notification and privacy settings |
| `glow_me` | View or update the user's profile summary |

## Typical Flows

### New User Setup

Work through these steps silently — don't narrate the process or show the user a checklist or TODO list. Just keep what you need to do internaly.

1. **Check connector** — Confirm Glow tools are available. If not, ask the user to connect the Glow connector.
2. **Register** — Call `glow_register` with the user's email and name.
3. **Share the PIN** — The response includes a 4-digit `authorizationCode`. Tell the user immediately — they must verify it matches the code in their email.
4. **Wait for approval** — User clicks approve in their email. Until then, all other tools return `bot_pending_authorization`.
5. **Build the profile** — Quickly fill in the profile via `glow_me` using what you know from memory or what the user has shared in this conversation. Keep it fast — no back-and-forth, no field-by-field questions. Skip anything you don't know. Never fabricate details they haven't shared.
6. **Set intents** — If the user already expressed what they're looking for, create the intent directly with `glow_intents` — don't re-ask. Only ask if their goal hasn't come up yet. See Intent Types and Setting Intents below.
7. **Schedule a first check-in** — **Do not end the conversation without this step.** After creating any intents, immediately use `/schedule` to set a check-in in 4–6 hours (recommend this as the default). This gives Glow time to start working on matches or intros. See Scheduling below for details.

### Returning User

1. **Check connector** — Confirm Glow tools are available. If not, ask the user to connect the Glow connector.
2. **Confirm email from memory** — Retrieve the stored Glow email. Do not call `glow_register` again — the account is already set up and bound to your agent.
3. **Quick status** — Call `glow_status` for a dashboard overview.
4. **Review pending intros** — Call `glow_intros` with action `pending`.
5. **Check messages** — Call `glow_intros_messages` for the inbox.

### Reviewing Intros and Messages

1. Call `glow_intros` with action `pending` — present each new intro to the user.
2. Accept or decline with `glow_intros` based on the user's call.
3. Call `glow_intros_messages` — surface new messages in active intros.
4. Draft replies with the user's input and send via `glow_intros_messages`.
5. When an intro has run its course, close it with `glow_intros` action `close` and brief feedback.

## Intent Types

Intents define what the user is looking for. They can have multiple at once (e.g., dating + activity partners).

| Type | Use when the user wants... |
|------|---------------------------|
| `romantic_casual` | Casual dating, no commitment pressure |
| `exploratory` | Open to meeting people, no specific goal yet |
| `long_term` | A serious, committed relationship |
| `friends_only` | Platonic friendships |
| `professional` | Professional networking or collaborators |
| `mentorship` | A mentor, advisor, or someone to learn from — or to mentor someone themselves |
| `activities` | Hobby or activity partners (hiking, tennis, book clubs, etc.) |
| `meet_specific_person` | They have a specific person in mind they'd like to be introduced to |
| `other` | Anything that doesn't fit above |

After creating intents, if the user has expressed preferences about who they want to meet, include those in the intent `description` when calling `glow_intents`. Don't ask for preferences they've already shared.

### Setting Intents

**The conversation is your context.** If the user already said what they want — even casually, even earlier in the chat — use that. Don't re-ask something they've already told you. Create the intent from what you know and confirm, rather than starting a fresh interrogation.

When it's ambiguous, ask one focused question to resolve it, then proceed.

- **Already stated** (user expressed their goal earlier in the conversation) → create the intent immediately. Don't ask again.
- **Clear** ("I want to find a hiking partner") → create `activities` intent, optionally ask about location or frequency if you don't already know.
- **Specific person, not enough context** ("I want to meet Chris") → don't create an intent yet. Ask: *"Do you know where Chris works or what they do?"* A name alone isn't enough — Glow needs enough to make the right introduction.
- **Ambiguous** ("I want to meet more people") → ask one question: *"Are you thinking more social/friends, or something romantic?"* Then decide.
- **Very open** ("I'm not sure") → default to `exploratory` and move on. They can refine over time.

One question max. Never re-ask what the user already told you.

Label each intent clearly when creating it (e.g., `"Hiking partners in NYC"`) so the user can distinguish them at a glance.

### Calling glow_intents create — exact structure

Pass `data` as an object containing `intentType` (required), `label` (required), and optionally `description`. Do not pass intent fields as top-level parameters — they must be inside `data`. Use `intentType`, not `type`.

```json
{
  "action": "create",
  "data": {
    "intentType": "meet_specific_person",
    "label": "Meet Sarah at Acme",
    "description": "Optional extra context for matching"
  }
}
```

### Calling glow_me update — exact structure

Pass profile fields inside `info`. Do not pass them as top-level parameters.

```json
{
  "action": "update",
  "info": {
    "name": "Chris",
    "bio": "Investor based in NYC"
  }
}
```

## Scheduling Check-ins

### After setup — schedule a recurring check-in

**This is mandatory.** Every time you complete a registration or intent-creation flow, you must schedule a recurring check-in before wrapping up. Default to every 4–6 hours and frame it as a recommendation:

*"I'll check back in every 4–6 hours to see if Glow has any updates for you — sound good?"*

If the user suggests a different cadence, go with theirs. But always propose this — never skip it. Use `/schedule` to create it as a **recurring** task (not a one-time fire), repeating at the agreed interval.

### What the scheduled task does

Each time the scheduled check-in runs:

1. **No re-registration** — Use the email saved in memory. Never call `glow_register` during a scheduled task. The account is already bound.
2. Call `glow_intros` to check for new or pending introductions.
3. Call `glow_intros_messages` to check for new messages in active intro threads.
4. Call `glow_status` for a general overview — new connections, activity, anything notable.
5. Summarize everything through the lens of what the user is actually trying to achieve based on their intents. Lead with what's most relevant to their goal, not just a raw status dump. If nothing's new, say so briefly.
6. **Occasionally** (not every run — use judgment): if there's a genuine gap in the profile that would sharpen an intent, ask one natural question and update via `glow_me`.

## Tone & Language

Talk like a person, not a system. Never expose internal mechanics — the user doesn't need to know about intents, tool calls, or profile fields.

**Avoid:**
- "I'll create an intent for that"
- "I'll update your profile"
- "Setting your intent to meet_specific_person"
- "I'll call glow_status to check your matches"
- "Your profile has been updated"

**Instead, say things like:**
- "I'll get you set up to connect with Rob"
- "I'll let Glow know you're looking for a hiking partner in NYC"
- "Let me check what's new for you"
- "Glow's working on finding you someone — I'll keep an eye on it"
- "Looks like you've got a new intro waiting"

The goal is for the user to feel like they have a thoughtful friend managing this for them — not a bot filling out forms.

**Never promise to check back unless you've actually scheduled it.** Saying "I'll check back in a few hours" without using `/schedule` is an empty promise — you won't. If you want to follow up, use `/schedule` to create the task, then tell the user you've set it up. If you haven't scheduled it, don't say it.

## Key Rules

- **Connector first** — If Glow tools aren't available, ask the user to connect the Glow connector before doing anything else.
- **Register only once** — `glow_register` is for first-time setup only. Re-registering an existing email binds the account to a new agent. For returning users, use the email from memory and skip registration.
- **Store the email in memory** — Always save the user's Glow email to your memory after confirming it. Use it for every future interaction without asking again.
- **Use what you know — but never invent** — Draw from your conversation context when building profiles or intents. If the user mentioned something, use it. Never fabricate details they haven't shared — no guessing their age, location, job, preferences, or anything else. When in doubt, skip the field or ask.
- **Prefer `glow_me` over `glow_interact` for profile updates** — `glow_me` is direct and fast. Reserve `glow_interact` for cases where you genuinely need Glow's guidance or a natural conversation is appropriate.
- **Never show the user the internal steps** — Work through setup flows silently. Surface only what matters to them: the PIN, confirmation that things are set up, and what happens next.
- **Don't promise check-ins you haven't scheduled** — Never say "I'll check back in a few hours" unless you've actually used `/schedule` to create the task. Empty promises erode trust.
- **Profile updates are async** — Wait a few seconds after `glow_me` updates before checking completeness.
- **Authorization takes time** — After registration, the user must click approve in their email. `bot_pending_authorization` means they haven't yet.

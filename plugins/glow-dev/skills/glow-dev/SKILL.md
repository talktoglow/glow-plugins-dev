---
name: glow
description: Help your human find meaningful connections through private introductions — dating, friendships, activity partners, professional networking, mentorship, or meeting a specific person. Use when the user mentions Glow, wants to meet someone specific, asks to find a date, friend, mentor, collaborator, or activity partner, or wants to manage their Glow account.
---

# Glow

Glow connects people through private, curated introductions — whether someone is looking for a date, a friend, a hiking partner, a mentor, a collaborator, or even a specific person they want to meet. Your role is to act on your human's behalf: set up their profile, manage their intents, review incoming intros, coordinate messages, and keep them updated on what's happening.

## Before You Start

**Check the connector first.** Before anything else, verify the Glow MCP tools are available in this session. If they aren't — or if any Glow tool call returns a connector/transport-level error (not a normal Glow error response) — this is a **connector issue**, not a Glow account issue.

When this happens:

- Tell the user: *"It looks like the Glow connector isn't connected in this session. Can you open Settings → Connectors and make sure Glow is enabled? Let me know once it's reconnected and I'll pick back up."*
- **Do not** guess at account-level causes ("maybe your email is already registered", "maybe the invite code failed", "Glow might be down"). Those are misleading when the real issue is the connector.
- Wait for the user to confirm reconnection before retrying.

**Confirm the user's Glow email.** Once the connector is confirmed, check your memory for a stored Glow email for this user:

- **Email found in memory** → confirm with the user it's still correct, then proceed to the relevant flow.
- **No email in memory** → don't assume they're new. Ask naturally: *"Do you already have a Glow account, or would you like to set one up?"*
  - If they have an account → ask for their email, save it to memory, and proceed to the Returning User flow.
  - If they're new → ask for their email and name, save the email to memory, and proceed to New User Setup.

Once confirmed, **save the email to your memory** so you never need to ask again.

**Do not call `glow_register` if the user already has an account.** Re-registering with an existing email binds that account to a new agent. Only call `glow_register` for the very first time a user sets up their account.

## MCP Tools

| Tool | Description |
|------|-------------|
| `glow_register` | Bind a human user to this session — always call this first for new users |
| `glow_interact` | **Primary tool for onboarding, profile updates, and intent creation.** Natural-language conversation with Glow — plain-text profile/intent updates trigger Glow's internal flows (e.g. `updateGoal` subgoal setup) automatically. |
| `glow_intros` | Manage introductions (list, pending, active, accept, decline, close) |
| `glow_intros_messages` | Read and send messages in active intro threads |
| `glow_photos` | Manage photos (list, upload, delete, set privacy) |
| `glow_status` | Dashboard — pending intros, active connections, unread messages |
| `glow_settings` | Get/update notification and privacy settings |
| `glow_me` | Read the user's profile summary. Prefer `glow_interact` for **writes** so Glow's server-side flows trigger. |
| `glow_intents` | Low-level intent CRUD. Prefer `glow_interact` for creating intents — direct `glow_intents` calls do **not** trigger Glow's `updateGoal` subgoal flow. |

### Why prefer `glow_interact` for writes

Direct calls to `glow_intents` and `glow_me` update data but do **not** fire Glow's server-side workflows. In particular, creating an intent via `glow_intents` does not launch the intent-specific subgoal flow on Glow's side. Sending the same information as plain text through `glow_interact` does — Glow parses it and triggers `updateGoal` automatically.

**Rule of thumb:** use `glow_interact` for profile updates, intent creation, and anything that should kick off a Glow-side flow. Use `glow_me` and `glow_intents` mainly for **reads** and for surgical fixes (e.g. renaming an intent label).

### Do not become a message-passing liaison

`glow_interact` is conversational, which creates a trap: it's easy to slip into shuttling messages back and forth — *"Glow says X"* → *"okay tell Glow Y"* → *"Glow now says Z"*. **Don't do this.** The user should not feel like they're on a conference call with Glow.

Instead:

- **Drive the conversation with Glow yourself.** Keep looping with `glow_interact` as many turns as needed to push the onboarding/intent flow to a real stopping point. You handle the back-and-forth with Glow; the user doesn't see it.
- **Only turn to the human when you genuinely need something only they can provide** — a decision, a preference, a fact you don't know, or a confirmation of something meaningful. Not for relaying Glow's prompts verbatim.
- **Never narrate Glow's side of the conversation.** Don't say *"Glow is asking about…"* or *"Glow wants to know…"*. Just ask the human directly, in your own voice, for what you actually need.
- **Batch questions.** If after several Glow turns you genuinely need two or three things from the human, ask them together once — not one at a time across multiple rounds.

Good loop: `glow_interact` → `glow_interact` → `glow_interact` → *ask human one focused question* → `glow_interact` → done.

Bad loop: `glow_interact` → *relay to human* → *relay back to Glow* → `glow_interact` → *relay to human* → …

## Typical Flows

### New User Setup

Work through these steps silently — don't narrate the process or show the user a checklist or TODO list. Keep the internal steps internal.

1. **Check connector** — Confirm Glow tools are available. If not, follow the connector-recovery instructions in *Before You Start*.
2. **Register** — Call `glow_register` with the user's email and name.
3. **Share the PIN** — The response includes a 4-digit `authorizationCode`. Tell the user immediately — they must verify it matches the code in their email.
4. **Wait for approval** — User clicks approve in their email. Until then, other tools return `bot_pending_authorization`. (This is a normal Glow response, not a connector issue — just wait.)
5. **Collect the two essentials** — The **only** two pieces of info required to get started are:
   - **Connection Intent** — what they're looking for (dating, friends, activity partner, mentor, etc.)
   - **Location** — where they are, so Glow can find local matches
   
   Do **not** ask about gender, orientation, education, job, age, bio, or other profile fields during onboarding. Glow will surface those later if/when they matter for a specific intent. If the user already mentioned any of that earlier in the conversation, you can pass it through, but never fish for it.
6. **Send intent + location via `glow_interact`** — Compose a short natural-language message to Glow containing the intent and location (e.g. *"User is looking for a hiking partner, based in NYC"*). This triggers Glow's `updateGoal` flow and starts the subgoal-specific onboarding on Glow's side. Loop with `glow_interact` as needed to complete whatever Glow asks next — remember, don't relay to the human, drive it yourself until you either finish or genuinely need human input.
7. **Schedule a first check-in** — **Do not end the conversation without this step.** After the intent is set, immediately use `/schedule` to set a check-in in 4–6 hours (recommend this as the default). See Scheduling below.

### Returning User

1. **Check connector** — Confirm Glow tools are available. If not, follow connector-recovery.
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

### Setting Intents

**The conversation is your context.** If the user already said what they want — even casually, even earlier in the chat — use that. Don't re-ask something they've already told you.

- **Already stated** → send it to Glow via `glow_interact` immediately. Don't ask again.
- **Clear** ("I want to find a hiking partner") → send via `glow_interact` with location, let Glow's subgoal flow handle follow-ups.
- **Specific person, not enough context** ("I want to meet Chris") → a name alone isn't enough. Ask one focused question (*"Do you know where Chris works or what they do?"*) before sending to Glow.
- **Ambiguous** ("I want to meet more people") → ask one question (*"Are you thinking friends, or something romantic?"*), then proceed.
- **Very open** ("I'm not sure") → default to `exploratory` via `glow_interact` and move on.

One question max. Never re-ask what the user already told you.

### When to reach for `glow_intents` directly

Only for narrow operations where no Glow-side flow needs to fire:

- Listing existing intents (`action: "list"`)
- Pausing an intent (`action: "pause"`)
- Renaming a label (`action: "update"`)

For **creating** an intent, use `glow_interact`. If you have a specific reason to call `glow_intents` create anyway, the exact shape is:

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

Fields go inside `data`. Use `intentType`, not `type`.

### When to reach for `glow_me` directly

Reads, and small corrections to an existing field. For anything substantive (new bio, new location, anything that might reshape matching), send it through `glow_interact` so Glow re-evaluates. If you do call `glow_me` update directly, fields go inside `info`:

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

**This is mandatory.** Every time you complete a registration or intent-setup flow, you must schedule a recurring check-in before wrapping up. Default to every 4–6 hours and frame it as a recommendation:

*"I'll check back in every 4–6 hours to see if Glow has any updates for you — sound good?"*

If the user suggests a different cadence, go with theirs. But always propose this — never skip it. Use `/schedule` to create it as a **recurring** task (not a one-time fire), repeating at the agreed interval.

### What the scheduled task does

Each time the scheduled check-in runs:

1. **No re-registration** — Use the email saved in memory. Never call `glow_register` during a scheduled task. The account is already bound.
2. Call `glow_intros` to check for new or pending introductions.
3. Call `glow_intros_messages` to check for new messages in active intro threads.
4. Call `glow_status` for a general overview — new connections, activity, anything notable.
5. Summarize everything through the lens of what the user is actually trying to achieve based on their intents. Lead with what's most relevant to their goal, not just a raw status dump. If nothing's new, say so briefly.
6. **Occasionally** (not every run — use judgment): if there's a genuine gap that would sharpen an intent, ask one natural question and push the update through `glow_interact`.

## Tone & Language

Talk like a person, not a system. Never expose internal mechanics — the user doesn't need to know about intents, tool calls, profile fields, or what Glow is saying back to you.

**Avoid:**
- "I'll create an intent for that"
- "I'll update your profile"
- "Setting your intent to meet_specific_person"
- "I'll call glow_status to check your matches"
- "Glow is asking if you…"
- "Let me tell Glow that…"
- "Your profile has been updated"

**Instead, say things like:**
- "I'll get you set up to connect with Rob"
- "I'll let Glow know you're looking for a hiking partner in NYC"
- "Let me check what's new for you"
- "Glow's working on finding you someone — I'll keep an eye on it"
- "Looks like you've got a new intro waiting"

The goal is for the user to feel like they have a thoughtful friend managing this for them — not a bot filling out forms, and not a switchboard operator relaying messages.

**Never promise to check back unless you've actually scheduled it.** Saying "I'll check back in a few hours" without using `/schedule` is an empty promise. If you want to follow up, use `/schedule` to create the task, then tell the user you've set it up.

## Key Rules

- **Connector issues ≠ account issues** — If Glow tools are unavailable or fail at the transport level, ask the user to reconnect the Glow connector in Settings. Do not speculate about email conflicts, invitation codes, or Glow outages.
- **Prefer `glow_interact` for writes** — Profile updates and intent creation should go through `glow_interact` so Glow's server-side flows (like `updateGoal`) trigger. Direct `glow_intents` / `glow_me` writes skip those flows.
- **Don't be a liaison** — Drive the `glow_interact` loop yourself across multiple turns. Only turn to the human when you need a real decision or fact from them. Never relay Glow's prompts verbatim.
- **Minimal onboarding** — The only two things required to get a user started are Connection Intent and Location. Don't ask for gender, orientation, education, age, job, or bio during setup.
- **Register only once** — `glow_register` is first-time only. For returning users, use the email from memory and skip registration.
- **Store the email in memory** — Always save the user's Glow email after confirming it.
- **Use what you know — but never invent** — If the user mentioned something earlier, use it. Never fabricate details they haven't shared.
- **Never show the user the internal steps** — Work through setup silently. Surface only what matters: the PIN, confirmation, and what happens next.
- **Don't promise check-ins you haven't scheduled** — Use `/schedule` first, then tell the user.
- **Authorization takes time** — After registration, the user must click approve in their email. `bot_pending_authorization` means they haven't yet — this is normal Glow behavior, not a connector problem.

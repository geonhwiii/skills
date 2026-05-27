---
name: growth-guardian
description: >-
  Use this skill when a coding request asks Claude to build, implement, or
  generate something that involves a non-trivial DESIGN decision the user has
  not yet made themselves — e.g. "make me a X", "build a Y", "implement the Z
  feature", "add caching", "set up state management", "create an API layer".
  The skill inserts a short, deliberate "design checkpoint" BEFORE generating
  code: it surfaces the hidden design decisions, asks the user for their own
  first-draft thinking, and presents concrete options with trade-offs so the
  user builds the mental model instead of outsourcing it. Trigger this even
  when the request is phrased casually or urgently, as long as a real design
  choice is buried in it. Do NOT trigger for trivial/mechanical requests, for
  pure debugging where a hypothesis already exists, or when the user has
  explicitly opted out (e.g. "just build it", "skip the questions", "I know
  what I want, here's the spec"). This skill exists to preserve the user's
  reasoning and skill growth in an AI-coding workflow — it is a thinking aid,
  never a gate.
---

# Growth Guardian

## Why this skill exists

There is solid evidence that delegating coding work to AI **accelerates output
but decouples it from skill growth**. When the AI makes the design decisions —
which data structure, where the responsibility boundaries lie, how errors are
handled, what abstraction level fits — the user's brain never takes on the
*germane cognitive load* that builds procedural memory and "chunks." The user
ends up with working code they cannot judge, extend, or debug, and over time
loses the very judgment that makes them good at directing AI in the first
place.

The fix is **desirable difficulty**: deliberately keeping the hard, valuable
thinking with the user, while letting AI remove the genuinely low-value
friction. This skill operationalizes that. It does not slow the user down for
its own sake — it makes sure the user *generates their own design hypothesis
first*, then compares it against options, so the work leaves a trace in their
head.

The guiding question, applied silently before every coding task:

> **"Is there a real design decision here the user could learn from making?"**

If yes → run the design checkpoint below.
If no → just build it. Friction with no learning payoff is pure cost.

## When to run the checkpoint (and when not to)

Run the checkpoint when **all** of these hold:

1. The request asks Claude to produce code, not just explain or fix something.
2. There is at least one **non-trivial design decision** embedded in it (see
   the decision inventory below).
3. The user has **not already specified** their own design intent for those
   decisions.
4. The user has **not opted out** of the checkpoint.

Skip the checkpoint (build directly) when **any** of these hold:

- The task is mechanical or boilerplate: renaming, formatting, a single
  obvious utility function, a `for` loop, type annotations, a regex, a
  config tweak. There is nothing to learn from making it.
- The user already brought a clear design: they handed you a spec, an
  interface signature, an architecture, or said which approach they want.
  They've already done the thinking — respect it.
- It's a debugging request **and the user has stated a hypothesis** ("I think
  the stale closure is the cause, can you confirm?"). They're already
  reasoning. If they just paste an error with "fix this", that's a candidate
  for a lighter nudge — see "Debugging" below.
- The user explicitly opts out: "just build it", "skip the questions", "no
  checkpoint", "I don't have time", "I've decided, just do X". Honor this
  immediately and without friction. The skill is a thinking aid, not a gate.
- The user is clearly exploring/prototyping something throwaway and has said
  so ("quick throwaway", "just want to see if it works").

When in genuine doubt, prefer **one** sharp question over a full checkpoint, or
ask the user directly: "Do you want me to walk the design with you first, or
just build it?"

## The decision inventory

A request hides a "real design decision" when answering it forces a choice
among these. Use this list to decide whether the checkpoint applies, and to
generate the questions:

- **Data modeling** — what shape holds the data; normalized vs. denormalized;
  which fields are derived vs. stored.
- **Responsibility boundaries** — what each module/component/function owns;
  where one responsibility ends and the next begins (single responsibility).
- **State ownership & flow** — who holds state, where it lives, how it's
  derived, how updates propagate; local vs. lifted vs. global.
- **Abstraction level** — how much to generalize now vs. keep concrete; is a
  layer/hook/adapter warranted at the project's current complexity.
- **Error handling strategy** — where errors are caught, how they surface,
  what the failure modes are, what's recoverable.
- **Interface/contract design** — the shape of the public API, props, function
  signature; what's required vs. optional; how callers will use it.
- **Dependency direction** — what depends on what; whether the direction is
  stable and points inward toward stable abstractions.
- **Performance/maintainability trade-off** — when an optimization is worth
  the added complexity, and when it's premature.

If a request touches **none** of these, there is no checkpoint. If it touches
one or more, those are exactly what the checkpoint questions should be about.

## The design checkpoint

Run these three moves in order. Keep the whole thing **short** — this is a
60-second thinking primer, not an interrogation. The point is to load the
problem into the user's head, not to extract a spec.

### Move 1 — Surface the hidden decisions

Name the design decisions the request actually contains, briefly. Make the
*invisible* visible. The user often doesn't realize "make me a notification
hook" silently decides polling vs. subscription, cleanup strategy, error
surfacing, and where the count state lives.

Keep it to 2-4 decisions, the ones that genuinely matter for this task. Don't
list everything from the inventory — list what's load-bearing here.

### Move 2 — Ask for the user's own first draft (generation effect)

Before showing any options, ask the user how *they* would approach the key
decision(s). This is the most important move: producing one's own answer
first — even a rough or wrong one — encodes the problem far more durably than
reading a finished answer. It also flips the dynamic from *consuming* Claude's
output to *evaluating* it against their own.

Phrase it as a genuine question, not a quiz. One or two questions, not five.
Examples:

- "Before I sketch options — how were you picturing the state living here?
  One hook that owns it, or lifted to the parent?"
- "What's your instinct for where this should fail loudly vs. fail silently?"
- "If you had to name the two responsibilities here, what would they be?"

If the user answers "I don't know, that's why I'm asking" — that's fine and
common. Don't push. Move to Move 3 and use the options themselves to teach,
explicitly tying back: "Here's how to think about it..."

### Move 3 — Present concrete options with trade-offs

Give **2-3 named, concrete options** for each load-bearing decision. For each:
a short name, what it optimizes for, what it costs, and when it's the right
call. Then make a recommendation *with reasoning* — don't just dump choices and
make the user pick blind; the recommendation itself models the judgment.

End by asking the user to choose (or to confirm the recommendation). Then build
what they chose.

Format options like this:

```
Decision: Where does the unread-count state live?

  A) Inside the hook (useNotificationCount owns it)
     + Encapsulated; callers just read a number.
     - Each caller gets an independent poller; N copies = N requests.
     Best when: only one place in the app needs the count.

  B) Lifted to a context/provider
     + One poller, one source of truth, shared across the tree.
     - More wiring; the provider must exist above all consumers.
     Best when: multiple components show the count (badge, header, page).

Recommendation: B, if the count appears in more than one place — the
duplicate-poller cost in A bites quickly. If it's genuinely one spot, A is
simpler and fine.

Which fits your case?
```

## After building — close the loop

Once you've built what the user chose, spend two sentences on **post-task
comprehension**: point out *why* the chosen approach works and the one thing
most likely to bite later. This is cheap and it converts "code I received"
into "code I understand." Example: "This cleans up the interval on unmount and
re-subscribes when `userId` changes — the thing to watch is that
`fetchNotifications` isn't cancelled, so a slow response after unmount could
still resolve; harmless here but worth knowing."

Do not turn this into a lecture. Two or three sentences, then stop.

## Debugging requests — a lighter touch

Debugging is where outsourcing is most tempting ("here's the error, fix it")
and most costly — fixing without a hypothesis erodes system understanding.

- If the user **states a hypothesis**, skip the checkpoint entirely. They're
  reasoning; just help.
- If the user pastes an error with no hypothesis, don't run the full
  checkpoint. Instead ask **one** question: "Before I dig in — what's your
  best guess at the root cause, even a rough one?" Then work *with* their
  guess: "You're right about X, but the actual trigger is Y." This refines
  their reasoning instead of replacing it.
- If they say "I genuinely have no idea" or opt out, just help directly.

## Calibration notes

- **Match the user's level.** A senior developer asking about a new framework
  is a beginner in that framework — the checkpoint still helps, but lean on
  their existing chunks ("this is like middleware, but..."). For routine work
  well within their expertise, there's nothing to learn — skip it.
- **The opt-out is sacred.** If the user ever signals they want speed over
  the walkthrough, drop it instantly and don't re-litigate. Re-running
  friction the user rejected is the fastest way to make the skill annoying
  and ignored. You can offer once ("want the design walk-through, or straight
  to code?") and then respect the answer for the rest of the session.
- **One checkpoint, not many.** Don't re-run the full checkpoint for every
  follow-up in a session. Once the user is engaged and deciding, just keep
  surfacing trade-offs inline as they come up.
- **Brevity is the whole game.** A checkpoint that's longer than the code it
  guards has failed. Surface, ask, offer options — tight. The user should
  feel helped to think, not slowed down.
- **Never fake it.** Don't manufacture design decisions for a task that
  genuinely has none just to run the ritual. That trains the user to skip the
  checkpoint reflexively.

For worked before/after examples of the checkpoint in action, see
`examples.md`.

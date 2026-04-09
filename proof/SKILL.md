---
name: proof
description: Use this skill to strip AI-sounding patterns from any draft and rewrite it in the user's personal voice. Trigger on requests like "proof this", "remove AI tells", "this sounds like AI", "make it sound like me", "rewrite in my voice", "clean this up", or any indication that content feels robotic, over-polished, or AI-generated. Run this as a final pass after any content has been drafted.
---

# Proof

A post-processing skill. It takes any draft — written by Claude, another AI tool, or the user — and does two things: strips universal AI patterns, then rewrites to match the user's personal voice. Think of it as an editorial filter, not a content generator.

## Your Writing Style File

This skill requires a personal writing style guide to work as intended.

**Expected location:** `_shared/about/my-writing-style.md`

Read this file before rewriting anything. It is the voice authority — all decisions about tone, word choice, structure, and what to remove should defer to it. The AI detection rules below are universal; the voice rules are personal.

> **Why a separate file?** Your writing style evolves. Keeping it in its own file means you update it once and every skill that references it improves automatically. It also means you can use this skill across projects without duplicating your voice rules everywhere. If you don't have a writing style file yet, create one before using this skill — it's what makes the output sound like you rather than a generic cleaned-up version.

---

## How It Works

1. Read `_shared/about/my-writing-style.md` in full
2. Scan the draft for AI patterns using the Detection Checklist below
3. Rewrite, applying both the pattern fixes and your voice rules
4. Run the Quality Check before returning output

---

## AI Pattern Detection Checklist

Scan for every pattern below. Fix each one before moving on.

**Transition stacking** — "Furthermore," "Moreover," "Additionally," "In addition" chained together.
Fix: Cut the transition. Start with the idea. If two ideas need connecting, the connection should be clear from context.

**Passive voice** — "The decision was made," "Results were achieved," "It was determined."
Fix: Name the actor. "I decided." "We hit the number." "She confirmed."

**False agency** — Inanimate things performing human verbs. "The data tells us," "the decision emerges," "the culture shifts," "the market rewards," "the conversation moves toward." AI uses this to avoid naming who actually did something.
Fix: Name the human. "The team fixed it" beats "the complaint becomes a fix." If no specific person fits, use "you" to put the reader in the seat.

**Hedge layering** — "It's important to note that perhaps one might consider..."
Fix: State it or cut it. If it's worth saying, say it directly.

**Vague declaratives** — Announcing importance without naming the specific thing. "The reasons are structural." "The implications are significant." "The stakes are high." "The consequences are real."
Fix: Name the specific thing or cut the sentence entirely. If a sentence says something matters without showing what, it earns nothing.

**Dramatic fragmentation** — Manufactured profundity through staccato sentences. "[Noun]. That's it. That's the thing." / "X. And Y. And Z." / "This unlocks something. [Single word]."
Fix: Complete sentences. Trust the content, not the presentation.

**Lazy extremes** — "every," "always," "never," "everyone," "everybody," "nobody" used as false authority instead of a specific claim.
Fix: Replace with the specific thing you actually mean, or cut.

**Exclusive insight claims** — "Nobody's thinking about X," "nobody's measuring X," "no one is talking about X," "what people miss is" — any phrasing that implies the author is uniquely insightful or the only one making a connection.
Fix: Frame the gap without claiming exclusive ownership. "This needs more attention" or "this isn't getting the focus it deserves" names the problem without being condescending to readers who may already be thinking about it.

**Wh- sentence starters** — Sentences or paragraphs opening with What, When, Where, Which, Who, Why, How. "What makes this hard is..." / "Why this matters..." / "When you think about it..."
Fix: Lead with the subject or the verb. "The constraint is..." beats "What makes this hard is the constraint."

**Synonym cycling** — Avoiding word repetition by rotating synonyms ("said, stated, mentioned, noted, remarked").
Fix: Repeat the word. Real writers repeat words. Or restructure the sentence.

**Emotional escalation** — Artificial build-up ("good, great, incredible, life-changing").
Fix: Pick one honest descriptor. If something is good, call it good.

**Over-signposting** — "First... Second... Third... Finally..." for ideas that don't need a numbered structure. Also: meta-commentary that announces the structure instead of delivering it ("Let me walk you through," "In this section we'll," "As we'll see").
Fix: Use signposting only when sequence genuinely matters. Let the writing move without announcing itself.

**Mirror structure** — Answering in the same grammatical frame as the question.
Fix: Answer in your own framing.

**Generic specificity** — "Whether you're a startup founder or a Fortune 500 CEO..." to sound inclusive.
Fix: Talk to the actual audience. Name who you mean.

**List parallelism overkill** — Every bullet the exact same length and grammatical structure.
Fix: Vary it. Some items short, some longer. Mix fragments with full sentences when it reads better.

**Label:explanation in prose** — A bolded or standalone label followed by a colon and explanation ("The steal:", "The truth:", "The takeaway:", "The move:") used in running prose instead of a list.
Fix: Rewrite as a normal sentence. This pattern is natural inside a bulleted list; in prose it's a structural AI/content-creator tell.

---

## Banned Constructions

Remove or rewrite any of these:

- "Not just X, but also Y" → two direct statements
- "Not X. [sentence]. Y." — the same pattern split across two sentences (e.g., "I didn't do it for X. I did it for Y.") → one direct statement about the actual reason
- "In a world where..." → delete entirely, start with the actual point
- "It remains to be seen..." → say what you think will happen or cut it
- "In conclusion" / "In summary" / "To sum up" → end naturally
- "Let's dive in" / "Without further ado" → start with the content
- "Here's the thing" / "Here's what matters" → state the thing
- "I'm excited to share" / "I'm thrilled to announce" → share it, announce it
- "At the end of the day" → say what you mean
- Any sentence starting with "It" where "It" has no clear antecedent

---

## Banned Words

Never use. Replace or remove entirely.

**AI tells:** delve, embark, enlightening, esteemed, craft, crafting, realm, game-changer, unlock, skyrocket, revolutionize, utilize, utilizing, dive deep, tapestry, illuminate, unveil, pivotal, intricate, elucidate, hence, furthermore, harness, groundbreaking, cutting-edge, remarkable, navigating, landscape, testament, moreover, ever-evolving, shed light

**Filler and hedge:** just (when removable), very, really, literally, actually, certainly, probably, basically, perhaps (unless strategic)

**Business jargon** — replace with plain language, don't just delete:

| Avoid | Use instead |
|-------|-------------|
| Navigate (challenges) | Handle, address |
| Unpack (analysis) | Explain, examine |
| Lean into | Accept, embrace |
| Landscape (context) | Situation, field |
| Double down | Commit, increase |
| Deep dive | Analysis, examination |
| Take a step back | Reconsider |
| Moving forward | Next, from now |
| Circle back | Return to, revisit |
| On the same page | Aligned, agreed |

---

## Quality Check

Before returning output, confirm every item:

1. Zero em dashes in the entire piece?
2. Zero semicolons?
3. Zero banned words?
4. Every sentence in active voice?
5. No transition stacking?
6. No "not just X, but also Y" constructions?
7. No sentence starts with a vague "It" or "This" without a clear reference?
8. No inanimate subject performing a human verb (false agency)?
9. No vague declaratives announcing importance without naming the specific thing?
10. No dramatic fragmentation — staccato sentences performing profundity?
11. No lazy extremes (every, always, never, nobody) standing in for a specific claim?
12. No exclusive insight claims ("nobody's thinking about X," "no one is measuring X")?
13. No Wh- sentence starters (What, When, Why, How) that could lead with the subject instead?
14. No business jargon from the substitution table?
15. Does it match the voice rules in `my-writing-style.md`?
16. Does every sentence sound like something the user would say out loud?

If any check fails, fix it before delivering.

---

## Output Rules

- Return the rewritten content only
- No preamble ("Here's your rewritten version...")
- No meta-commentary about what was changed
- No warnings or disclaimers
- Preserve the original meaning and all factual claims
- Match the original length within 10% — don't pad or trim significantly
- If the original had a specific structure, keep it unless the structure itself is the AI-sounding problem

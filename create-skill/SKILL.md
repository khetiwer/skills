---
name: create-skill
description: Use this skill whenever creating a new Claude Code skill, adding to the skills library, or turning a workflow into a reusable skill. Trigger on any request like "can we make this a skill", "let's build a skill for X", "I want to save this as a skill", or "add this to my skills library". Always run this before writing a single line of a new skill — it enforces placement decisions and intent alignment first.
---

# Create Skill

A skill for building new skills in your personal skills library. It enforces two decisions before writing anything — where the skill lives and what it actually does — then follows a disciplined draft-test-refine loop.

---

## Step 1: Placement

Ask these two questions before doing anything else. Wait for answers to both.

**Question 1:** "If you deleted every project you're currently working on tomorrow, would you still want this skill?"

**Question 2:** "Does this skill need to read from or write to any project-specific files, folders, or systems to do its job?"

**How to interpret the answers:**

| Q1 | Q2 | Location |
|---|---|---|
| Yes — still want it | No project dependencies | Top-level `skills/` directory |
| No — tied to a project | Yes or no | Inside the relevant project folder |

A skill that currently has hardcoded project paths belongs in the project even if the concept feels reusable. Promote it to top-level only after you've actually made it general-purpose.

Confirm the location with the user before moving on.

---

## Step 2: Capture Intent

Ask these four questions. You can ask them together, but don't draft anything until you have answers.

1. What should this skill enable Claude to do?
2. When should this skill trigger? What would the user actually say or be doing when they need it?
3. What should the output look like — format, length, structure?
4. Are the outputs objectively verifiable (fixed steps, file transforms, data extraction) or subjective (tone, style, judgment calls)? This shapes whether test assertions are worth writing.

If the current conversation already contains a workflow the user wants to capture, extract answers from context first — tools used, sequence of steps, corrections made — and confirm before proceeding. They shouldn't have to repeat themselves.

---

## Step 3: Draft the Skill

### Directory structure

```
skill-name/
├── SKILL.md          (required)
├── references/       (docs or context loaded when needed)
├── scripts/          (reusable code the skill calls)
└── assets/           (templates, files used in output)
```

Only create additional folders if you actually have content for them. Don't scaffold empty directories.

### SKILL.md components

- **Frontmatter:** `name` and `description` (both required)
- **Body:** instructions in imperative form, under 500 lines
- If approaching 500 lines, break content into `references/` files and add explicit guidance in SKILL.md on when and why to read them

### Writing the description

The description is what determines whether Claude uses this skill at all. It's the trigger. Write it to cover both what the skill does AND the specific phrases or contexts that should invoke it. Include things the user would actually say, not just abstract capability descriptions. A vague description means the skill never fires.

Make it slightly "pushy" — if the skill should trigger when the user mentions a topic even without naming the skill explicitly, say so.

### Writing the instructions

Explain why behind each step, not just what to do. Claude generalizes better from reasoning than from rigid rules. Use imperative form ("Read the contact file first" not "You should read the contact file"). Avoid all-caps directives where you can explain the principle instead — it makes the skill more adaptable to edge cases.

Share the draft with the user before moving to testing.

---

## Step 4: Test

Write 2-3 realistic test prompts — specific, the kind of thing the user would actually type in a real situation. Not abstract ("clean up this writing") but grounded in their actual context and phrasing.

Share the prompts first: "Here are the test cases I want to try — do these look right, or should we adjust?"

Run each one and note:
- Did the skill trigger when it should have?
- Did the output match the expected format and feel?
- What broke or felt off?

---

## Step 5: Refine

Identify the root cause of any issues before making changes. Most problems fall into one of three buckets:

- **Description problem** — the skill didn't trigger, or triggered at the wrong time
- **Instruction problem** — the skill triggered but did the wrong thing
- **Scope problem** — the skill is trying to do too much at once

Make targeted changes. Re-run affected test prompts. One round of refinement is usually enough. If the same issue keeps recurring, surface the tension to the user rather than continuing to iterate — sometimes the fix is rethinking the skill's scope, not tweaking the wording.

---

## Step 6: Confirm the Description

Before finalizing, evaluate the description against these checks:

- Does it name specific trigger phrases the user would actually say?
- Does it say both what the skill does AND when to use it?
- Is it specific enough that Claude wouldn't accidentally use it for something adjacent?
- Would it still trigger if the user didn't use the skill's exact name?

If any check fails, revise the description before delivering.

---

## Output

Save the completed skill to the location confirmed in Step 1. Name the folder to match the skill's `name` frontmatter exactly.

Confirm the full path with the user before writing any files.

---

## Step 7: Publish

Once the user confirms they are happy with the skill, ask: "Ready to publish? I'll zip it, add the download link to README.md, and push to GitHub."

If yes, run the following in order:

**1. Zip the skill folder** (from inside the skills directory):
```
python -c "import zipfile, os; z = zipfile.ZipFile('<skill-name>.zip', 'w', zipfile.ZIP_DEFLATED); [z.write(os.path.join('<skill-name>', f), os.path.join('<skill-name>', f)) for f in os.listdir('<skill-name>')]; z.close()"
```

**2. Add the download link to README.md**

Find the skill's entry in `README.md` and append this line at the end of it:
```
[Download <skill-name>.zip](https://github.com/khetiwer/skills/releases/latest/download/<skill-name>.zip)
```

**3. Commit and push both files:**
```
git add <skill-name>.zip README.md
git commit -m "Add <skill-name>.zip and README download link"
git push
```

If the user says no or wants more changes, return to refining. Do not publish until they explicitly confirm.

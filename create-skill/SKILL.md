---
name: create-skill
description: Use this skill whenever creating a new Claude Code skill, adding to the skills library, or turning a workflow into a reusable skill. Trigger on any request like "can we make this a skill", "let's build a skill for X", "I want to save this as a skill", or "add this to my skills library". Always run this before writing a single line of a new skill — it enforces placement decisions and intent alignment first.
---

# Create Skill

A skill for building new skills in your personal skills library. It enforces two decisions before writing anything — where the skill lives and what it actually does — then follows a disciplined draft-test-eval-refine loop.

**Key resources in this skill folder:**

- `references/schemas.md` — JSON schemas for all eval pipeline data structures. Read this when creating or interpreting evals.json, grading.json, benchmark.json, or any other pipeline output.
- `agents/grader.md` — Prompt for the grader subagent that evaluates expectations against outputs.
- `agents/analyzer.md` — Prompt for the analyzer subagent that explains why one version beat another.
- `agents/comparator.md` — Prompt for the blind comparator that judges two outputs without knowing their source.
- `scripts/` — Python scripts for automated description optimization, benchmarking, and reporting.

---

## Step 1: Placement

Ask these two questions before doing anything else. Wait for answers to both.

**Question 1:** "If you deleted every project you're currently working on tomorrow, would you still want this skill?"

**Question 2:** "Does this skill need to read from or write to any project-specific files, folders, or systems to do its job?"

**How to interpret the answers:**

| Q1 | Q2 | Location |
|---|---|---|
| Yes — still want it | No project dependencies | `~/.claude/skills/` (global) |
| No — tied to a project | Yes or no | Inside the relevant project folder |

A skill that currently has hardcoded project paths belongs in the project even if the concept feels reusable. Promote it to top-level only after you've actually made it general-purpose.

**Consequences to surface at confirmation time:**

| Placement | Slash command in CLI | GitHub | Agentman |
|---|---|---|---|
| Global (`~/.claude/skills/`) | Yes — auto-registered | Yes | Yes |
| Project-local | No — invoke by name or path only | Yes | Yes |

Both placements go to GitHub and Agentman. Agentman makes the skill available in Cowork and any other Claude surface regardless of where the local file lives — even project-local skills with hardcoded paths work in Cowork because Cowork has access to your project files.

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
├── evals/            (eval test cases and supporting files)
│   ├── evals.json    (test case definitions)
│   └── files/        (input files referenced by evals)
├── references/       (docs or context loaded when needed)
├── scripts/          (reusable code the skill calls)
└── assets/           (templates, files used in output)
```

Only create additional folders if you actually have content for them. Don't scaffold empty directories. The `evals/` directory is created in Step 5.

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

## Step 4: Quick Test

Write 2-3 realistic test prompts — specific, the kind of thing the user would actually type in a real situation. Not abstract ("clean up this writing") but grounded in their actual context and phrasing.

Share the prompts first: "Here are the test cases I want to try — do these look right, or should we adjust?"

Run each one and note:
- Did the skill trigger when it should have?
- Did the output match the expected format and feel?
- What broke or felt off?

This is a fast sanity check before writing formal evals. If the skill is fundamentally broken, fix it now before investing in eval infrastructure.

---

## Step 5: Write Evals

After the skill passes its quick test, create structured eval cases. This is where you move from "does it seem to work" to "can we measure how well it works."

### Create evals.json

Create `evals/evals.json` in the skill's directory. Read `references/schemas.md` for the full schema. Each eval case needs:

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "The exact user prompt to test with",
      "expected_output": "Human-readable description of what success looks like",
      "files": [],
      "expectations": [
        "The output contains a table of contents",
        "The script used pandas for data processing",
        "The final PDF has exactly 3 pages"
      ]
    }
  ]
}
```

### Writing good expectations

Expectations are the core of the eval system. They must be verifiable — a grader agent will check each one against the actual output and transcript.

**Good expectations** are discriminating: they pass when the skill genuinely succeeds and fail when it doesn't.

- "The output file is a valid PDF with extractable text" (checks substance, not just file extension)
- "The summary is under 200 words and mentions all three risk categories from the input" (checks both constraint and content)
- "The script reads the input CSV before processing" (checks process, verifiable from transcript)

**Bad expectations** are trivially satisfied or unverifiable:

- "The output is good" (subjective, unverifiable)
- "A PDF was created" (any empty PDF would pass)
- "The user is satisfied" (can't check from outputs alone)

### How many evals to write

Start with 3-5 eval cases that cover:

- The happy path (typical use case)
- An edge case (unusual input, missing data, large file)
- A negative case (input that the skill should handle gracefully or decline)

If the skill's outputs are subjective (tone, style, judgment), focus expectations on structure and process rather than content quality. You can still eval whether the skill follows its own instructions, uses the right tools, and produces output in the right format.

### Supporting files

If eval cases need input files (a sample PDF, CSV, image), place them in `evals/files/` and reference them in the eval's `files` array with relative paths.

Present the proposed evals to the user: "Here are the eval cases I've drafted — do these cover the important scenarios?"

---

## Step 6: Run Evals and Grade

This step executes the eval cases and grades the results. It uses subagent tasks for isolation.

### Running eval cases

For each eval in `evals.json`, spawn two parallel subagent tasks:

1. **with_skill**: Run the eval prompt with the skill loaded. The subagent should have access to the skill and any supporting files.
2. **baseline** (without_skill): Run the same eval prompt without the skill. This tells you whether the skill actually adds value or whether Claude would have done fine anyway.

Each subagent should:
- Execute the eval prompt as if it were a real user request
- Save all output files to its workspace
- Save a transcript of its execution
- Save `metrics.json` with tool call counts and output sizes (see `references/schemas.md`)

### Capture timing

When each subagent task completes, the task result includes duration and token usage. Save these immediately to `timing.json` in the run directory — they aren't persisted anywhere else.

### Grade each run

After a run completes, spawn a grader subagent using the prompt in `agents/grader.md`. Pass it:
- The expectations from the eval case
- The path to the execution transcript
- The path to the output files directory

The grader produces `grading.json` with pass/fail for each expectation plus evidence. See `references/schemas.md` for the full schema.

The grader also critiques the evals themselves — flagging weak assertions or gaps in coverage. Pay attention to `eval_feedback` in the grading output.

### Directory layout for runs

```
evals/
├── evals.json
├── files/
└── runs/
    └── eval-1/
        ├── eval_metadata.json
        ├── with_skill/
        │   └── run-1/
        │       ├── outputs/
        │       │   ├── <output files>
        │       │   └── metrics.json
        │       ├── transcript.md
        │       ├── timing.json
        │       └── grading.json
        └── without_skill/
            └── run-1/
                ├── outputs/
                ├── transcript.md
                ├── timing.json
                └── grading.json
```

### Interpreting results

After grading completes, summarize results for the user:
- Pass rate per eval (with_skill vs. baseline)
- Which expectations failed and why (cite the grader's evidence)
- Any eval feedback from the grader (weak assertions, missing coverage)
- Whether the skill meaningfully outperforms the baseline

If the skill doesn't beat the baseline, that's important signal — the skill may not be adding value, or the expectations may not be testing the right things.

---

## Step 7: Compare and Analyze (for skill iterations)

When you're comparing two versions of a skill (e.g., after making improvements), use blind comparison to eliminate bias.

### Blind comparison

Spawn a comparator subagent using the prompt in `agents/comparator.md`. Pass it both outputs labeled A and B (randomize which is which). The comparator judges quality purely on output merit without knowing which skill produced which.

### Post-hoc analysis

After the comparator picks a winner, spawn an analyzer subagent using the prompt in `agents/analyzer.md`. The analyzer "unblinds" the results and explains:
- What made the winner better (specific instruction or tool advantages)
- What held the loser back (ambiguity, missing scripts, gaps)
- Prioritized improvement suggestions

### Track iteration history

Maintain `history.json` (see `references/schemas.md`) at the workspace root to track version progression:
- Which version is current best
- Pass rates at each iteration
- Whether each version won, lost, or tied against its parent

This gives you a clear picture of whether your changes are actually improving things.

---

## Step 8: Benchmark (optional, for thorough evaluation)

For skills that need rigorous measurement, run a full benchmark with multiple runs per eval case.

### Running benchmarks

Run each eval case 3 times per configuration (with_skill and without_skill) to measure variance. Use the same grading process from Step 6 for each run.

### Aggregate results

Run the benchmark aggregation script from the skill's parent directory:

```bash
python -m scripts.aggregate_benchmark <benchmark_dir> --skill-name <name> --skill-path <path>
```

This produces `benchmark.json` and `benchmark.md` with:
- Mean, stddev, min, max for pass rate, time, and tokens per configuration
- Delta between with_skill and without_skill
- Per-run details with expectations and notes

### Analyze benchmark patterns

After aggregation, spawn an analyzer subagent (using the benchmark analysis section of `agents/analyzer.md`) to surface patterns:
- Assertions that always pass in both configs (not differentiating)
- Assertions that show high variance (flaky or non-deterministic)
- Token/time cost of the skill vs. its pass rate improvement

### Generate HTML report

For visual review of description optimization results, use the report generator:

```bash
python -m scripts.generate_report results.json -o report.html --skill-name <name>
```

Open `report.html` in a browser to see a table of each iteration with pass/fail per query, train/test split performance, and the best-performing description highlighted.

---

## Step 9: Refine

Identify the root cause of any issues before making changes. Most problems fall into one of three buckets:

- **Description problem** — the skill didn't trigger, or triggered at the wrong time
- **Instruction problem** — the skill triggered but did the wrong thing
- **Scope problem** — the skill is trying to do too much at once

Make targeted changes. Re-run affected eval cases (not the full suite unless the change is broad). One round of refinement is usually enough. If the same issue keeps recurring, surface the tension to the user rather than continuing to iterate — sometimes the fix is rethinking the skill's scope, not tweaking the wording.

### Using eval feedback to refine

The grader's `eval_feedback` field often contains the most actionable refinement hints:
- Weak assertions that need tightening
- Missing coverage areas that need new expectations
- Process gaps where the skill's instructions are ambiguous

Update both the skill AND the evals based on this feedback. Improving your evals is as valuable as improving the skill.

### Description optimization loop

If the skill's description is the problem (not triggering correctly), use the automated optimization loop:

```bash
python -m scripts.run_loop \
  --eval-set evals/trigger_queries.json \
  --skill-path <path-to-skill> \
  --model claude-sonnet-4-20250514 \
  --max-iterations 5 \
  --verbose
```

This runs an eval-improve cycle: test the description against trigger queries, use Claude to propose an improved description, test again, repeat. It produces a live HTML report you can watch in your browser, and returns the best-performing description.

The trigger query eval set is a separate file from `evals.json` — it tests whether the skill triggers, not whether it produces good output. Format:

```json
[
  {"query": "can you make me a presentation about Q3 results", "should_trigger": true},
  {"query": "what's the weather like today", "should_trigger": false},
  {"query": "build a deck for the board meeting", "should_trigger": true}
]
```

---

## Step 10: Confirm the Description

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

## Step 11: Validate and Publish

### Validate

Before publishing, run the validation script:

```bash
cd ~/.claude/skills && python create-skill/scripts/quick_validate.py <path-to-skill>
```

This checks frontmatter format, required fields, naming conventions, and description length limits. Fix any issues before proceeding.

### Publish

Once the user confirms they are happy with the skill, ask: "Ready to publish? I'll package it and push to GitHub."

If yes:

**1. Package the skill:**
```bash
cd ~/.claude/skills && python create-skill/scripts/package_skill.py <path-to-skill>
```

This creates a `.zip` file, excluding evals, __pycache__, and other build artifacts.

**2. Check for a GitHub remote:**
```bash
git -C ~/.claude/skills remote -v
```

If a remote exists, continue with steps 3 and 4. If no remote is configured, skip to step 5.

**3. Upload as a GitHub Release asset** (do NOT commit the zip to the repo):
```bash
gh release upload <latest-tag> <skill-name>.zip --repo <owner>/skills
```

To find the latest tag: `gh release list --repo <owner>/skills` — use the tag shown as Latest.

Then add or update the skill's entry in the README.md Skills section. Each entry needs: a heading with the skill name, a bold one-line description, 2-3 sentences on what it does, a triggers line, and the download link. If an entry already exists for this skill, check that the description and triggers still match the current SKILL.md — update if they've drifted.

```
[Download <skill-name>.zip](https://github.com/<owner>/skills/releases/latest/download/<skill-name>.zip)
```

**4. Commit and push the README only:**
```bash
git -C ~/.claude/skills add README.md
git -C ~/.claude/skills commit -m "Add <skill-name> and README download link"
git -C ~/.claude/skills push
```

**5. If no GitHub remote:**

Tell the user: "No GitHub remote found. The package is ready at `<path>/<skill-name>.zip` — share it directly however works best for you."

**6. Publish to Agentman:**

Call `create_skill` with the full content of `SKILL.md` (including frontmatter).

Then, for each auxiliary file that exists in the skill folder (`references/`, `scripts/`, `assets/`), call `save_skill_resource` with:
- `slug`: the skill's slug (matches the `name` frontmatter, lowercased and hyphenated)
- `file_path`: relative path within the skill directory (e.g., `references/schemas.md`, `scripts/main.py`)
- `content`: the full file content

Do this for every auxiliary file. Do not skip files — Agentman needs the full skill to work correctly across Claude surfaces.

If `create_skill` fails because the slug already exists, use `update_skill` instead, then call `publish_skill` to make the updated draft live.

If the user says no or wants more changes, return to refining. Do not publish until they explicitly confirm.

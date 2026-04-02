# Skills

A personal library of reusable skills for Claude Code. Each skill is a self-contained workflow that tells Claude how to handle a specific category of work — consistently, without re-explaining from scratch every time.

---

## What is a skill?

A skill is a `SKILL.md` file that gives Claude structured instructions for a task: what to do, in what order, and how to handle edge cases. You drop it into a folder, point your `CLAUDE.md` at it, and Claude follows the workflow every time that task comes up.

Skills in this library are designed to be triggered automatically — when you share a job posting URL, ask to clean up a draft, or want to build a new skill, Claude picks up the right workflow without you having to describe it each time.

---

## Skills

### `apply-job`
**Job application workflow — resume tailoring, fit analysis, cover letter**

Fetches a job description from a URL, runs a structured fit analysis with a go/no-go verdict, tailors your resume from a base file, and writes a cover letter. Includes a formatting-safe `python-docx` editing method that preserves resume styling when swapping bullet variations.

Triggers on: job posting URLs, "apply to this job," "tailor my resume for," "write a cover letter for"

Requires setup: `PROFILE.md`, `BULLETS.md`, `CONTEXT.md`, and an ATS-optimized base resume. See `references/SETUP.md` and `references/QUICKSTART.md` inside the skill folder.

[Download apply-job.zip](https://github.com/khetiwer/skills/releases/latest/download/apply-job.zip)

---

### `proof`
**Strip AI patterns and rewrite in your personal voice**

Takes any draft — written by Claude, another AI tool, or you — and does two things: removes universal AI-sounding patterns (transition stacking, hedge layering, passive voice, banned words) and rewrites to match your personal voice.

Requires a personal writing style file at `_shared/about/my-writing-style.md`. The skill reads this file before rewriting anything — without it, you get generic cleanup, not your voice.

Triggers on: "proof this," "remove AI tells," "this sounds like AI," "make it sound like me," "rewrite in my voice"

[Download proof.zip](https://github.com/khetiwer/skills/releases/latest/download/proof.zip)

---

### `create-skill`
**Build new skills for your personal library**

Enforces two decisions before writing anything — where the skill lives and what it actually does — then follows a draft-test-refine loop. Prevents the common failure mode of building skills that are too broad, too project-specific, or that never actually trigger.

Triggers on: "can we make this a skill," "let's build a skill for X," "I want to save this as a skill," "add this to my skills library"

[Download create-skill.zip](https://github.com/khetiwer/skills/releases/latest/download/create-skill.zip)

---

## How to install

### Option 1 — Download a single skill

Download the zip for the skill you want, extract it, and move the folder into your skills directory.

Your skills directory can live anywhere. A common setup:

```
AI Workspace/
└── skills/
    ├── apply-job/
    │   └── SKILL.md
    ├── proof/
    │   └── SKILL.md
    └── create-skill/
        └── SKILL.md
```

### Option 2 — Clone the full library

```bash
git clone https://github.com/khetiwer/skills.git
```

---

## How to wire up a skill

Once the skill is on your file system, tell Claude where to find it. Add an instruction to the `CLAUDE.md` in the relevant project (or your global `~/.claude/CLAUDE.md` for skills you want available everywhere):

```markdown
When the user wants to [trigger condition], read and follow the skill at
`/path/to/skills/skill-name/SKILL.md`.
```

Example for `apply-job`:

```markdown
When the user wants to apply to a job, tailor a resume, write a cover letter,
or shares a job posting URL, read and follow the skill at
`C:/Users/you/skills/apply-job/SKILL.md`.
```

Example for `proof` (global — useful across all projects):

```markdown
When the user asks to proof a draft, remove AI patterns, or rewrite something
in their voice, read and follow the skill at `C:/Users/you/skills/proof/SKILL.md`.
```

---

## Building your own

Use the `create-skill` skill to build new ones. It asks two questions before writing anything: whether the skill belongs in your global library or inside a specific project, and what it should actually do. That conversation usually surfaces scope problems before they get baked in.

The `create-skill` README has the full process.

---

## Credits

These skills were built on top of — or inspired by — work from others in the Claude Code community.

**apply-job** — Built on the foundation of [Aaron Skinner's claude-skills](https://github.com/aaronwskinner/claude-skills). The core job application workflow structure originated there.

**proof** — Inspired by two projects: [blader/humanizer](https://github.com/blader/humanizer) and [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop). Both tackle the same problem of stripping AI-generated patterns from writing — worth checking out if you want a different take on the same idea.

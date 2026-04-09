# Skills

A personal library of reusable skills for Claude Code. Each skill is a self-contained workflow that tells Claude how to handle a specific category of work, consistently, without re-explaining from scratch every time.

---

## What is a skill?

A skill is a `SKILL.md` file that gives Claude structured instructions for a task: what to do, in what order, and how to handle edge cases. Drop it into a folder, point your `CLAUDE.md` at it, and Claude follows the workflow every time that task comes up.

These skills trigger automatically. When you share a job posting URL, ask to clean up a draft, or want to build a new skill, Claude picks up the right workflow without you having to describe it each time.

---

## Skills

### `apply-job`
**Job application workflow: resume tailoring, fit analysis, cover letter**

Fetches a job description from a URL, runs a structured fit analysis with a go/no-go verdict, tailors your resume from a base file, and writes a cover letter. Uses a formatting-safe `python-docx` method that preserves resume styling when swapping bullet variations.

Triggers on: job posting URLs, "apply to this job," "tailor my resume for," "write a cover letter for"

Requires setup: `PROFILE.md`, `BULLETS.md`, `CONTEXT.md`, and an ATS-optimized base resume. See `references/SETUP.md` and `references/QUICKSTART.md` inside the skill folder.

[Download apply-job.zip](https://github.com/khetiwer/skills/releases/latest/download/apply-job.zip)

---

### `proof`
**Strip AI patterns and rewrite in your personal voice**

Takes any draft (written by Claude, another AI tool, or you) and does two things: removes universal AI-sounding patterns (transition stacking, hedge layering, passive voice, banned words) and rewrites to match your personal voice.

Requires a personal writing style file at `_shared/about/my-writing-style.md`. The skill reads this file before rewriting anything. Without it, you get generic cleanup, not your voice.

Triggers on: "proof this," "remove AI tells," "this sounds like AI," "make it sound like me," "rewrite in my voice"

[Download proof.zip](https://github.com/khetiwer/skills/releases/latest/download/proof.zip)

---

### `create-skill`
**Build new skills for your personal library**

Enforces two decisions before writing anything: where the skill lives and what it actually does. Then follows a draft-test-refine loop. Prevents the common failure mode of building skills that are too broad, too project-specific, or that never actually trigger.

Triggers on: "can we make this a skill," "let's build a skill for X," "I want to save this as a skill," "add this to my skills library"

[Download create-skill.zip](https://github.com/khetiwer/skills/releases/latest/download/create-skill.zip)

---

### `transcribe-audio`
**Transcribe local audio or video files with OpenAI Whisper**

Checks for Whisper and FFmpeg, walks through model selection, task type (transcribe or translate), output format, and output directory, then runs Whisper. Warns before first-run model downloads, which can look like the process has hung but haven't.

Triggers on: "transcribe this," "run Whisper on," sharing an audio/video file path, "convert this recording to text"

Configuration: language (default: English), CPU/GPU mode, and inline output verbosity can all be changed by editing `SKILL.md` directly. See the Configuration notes section inside the skill.

[Download transcribe-audio.zip](https://github.com/khetiwer/skills/releases/latest/download/transcribe-audio.zip)

---

### `publish-skill`
**Publish skill updates to GitHub and Agentman in one step**

Takes an existing skill by name, packages it, uploads the updated zip to GitHub as a release asset, updates the skill in Agentman, syncs all auxiliary files, and publishes the draft live. Run this after editing a skill locally and you're ready to ship.

Triggers on: "publish [skill-name]", "push my [skill] changes", "sync [skill] to Agentman", "I updated [skill], publish it", "ship [skill]"

Note: Only runs from Claude Code CLI (requires bash and git access). Use `create-skill` for net new skills; this skill handles updates only.

[Download publish-skill.zip](https://github.com/khetiwer/skills/releases/latest/download/publish-skill.zip)

---

## How to install

### Option 1 — Download a single skill

Download the zip for the skill you want, extract it, and move the folder to the right location.

**Global skills** (available in every Claude Code session, slash commands work):

```
~/.claude/skills/
├── apply-job/
│   └── SKILL.md
├── proof/
│   └── SKILL.md
└── create-skill/
    └── SKILL.md
```

**Project-local skills** (scoped to one project, no slash command):

```
your-project/
└── skills/
    └── skill-name/
        └── SKILL.md
```

### Option 2 — Clone the full library

```bash
git clone https://github.com/khetiwer/skills.git ~/.claude/skills
```

---

## How to wire up a skill

Once the skill is on your file system, tell Claude where to find it. Add an instruction to the `CLAUDE.md` in the relevant project, or to your global `~/.claude/CLAUDE.md` for skills you want available everywhere:

```markdown
When the user wants to [trigger condition], read and follow the skill at
`~/.claude/skills/skill-name/SKILL.md`.
```

Example for `apply-job` (global):

```markdown
When the user wants to apply to a job, tailor a resume, write a cover letter,
or shares a job posting URL, read and follow the skill at
`~/.claude/skills/apply-job/SKILL.md`.
```

Example for a project-local skill:

```markdown
When the user wants to [trigger condition], read and follow the skill at
`/path/to/your-project/skills/skill-name/SKILL.md`.
```

---

## Building your own

Use the `create-skill` skill to build new ones. It asks two questions before writing anything: whether the skill belongs in your global library or inside a specific project, and what it should actually do. Those two answers usually surface scope problems before they get baked in.

The `create-skill` README has the full process.

---

## Credits

These skills were built on top of (or inspired by) work from others in the Claude Code community.

**apply-job** — Built on the foundation of [Aaron Skinner's claude-skills](https://github.com/aaronwskinner/claude-skills). The core job application workflow structure originated there.

**proof** — Inspired by two projects: [blader/humanizer](https://github.com/blader/humanizer) and [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop). Both tackle the same problem of stripping AI-generated patterns from writing. Worth checking out if you want a different take on the same idea.

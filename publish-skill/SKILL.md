---
name: publish-skill
description: Use this skill to publish updates to an existing skill — packages it, pushes to GitHub as a release asset, updates Agentman, and publishes the draft live. Trigger on: "publish [skill-name]", "push my [skill] changes", "sync [skill] to Agentman", "I updated [skill], publish it", "ship [skill]". Only use for skills that already exist in both GitHub and Agentman — for net new skills, use create-skill instead.
---

# Publish Skill

Updates an existing skill across all three surfaces: local file system (already done — you edited it), GitHub (release asset), and Agentman (update + publish). Run this after you've finished editing a skill locally and are ready to ship the changes.

This skill only runs from Claude Code CLI. It requires bash access for git and gh operations.

---

## Step 1: Resolve the Skill Path

Derive the local path from the skill name the user provided:

- Global skills: `~/.claude/skills/<name>/SKILL.md`
- If not found there, ask: "I didn't find that skill in `~/.claude/skills/`. Is it inside a project folder? If so, give me the path."

Read `SKILL.md` and extract:
- `name` frontmatter → this is the Agentman slug
- List of all auxiliary files present in the skill folder (`references/`, `scripts/`, `assets/`) — you'll need these for Agentman

Confirm with the user: "Publishing `<name>` from `<path>`. This will update GitHub and Agentman. Proceed?"

Do not proceed until confirmed.

---

## Step 2: Validate

Run the validation script before touching anything:

```bash
cd ~/.claude/skills && python create-skill/scripts/quick_validate.py <path-to-skill>
```

If validation fails, surface the errors to the user and stop. Fix issues before publishing — a broken skill should not go out.

---

## Step 3: Package

```bash
cd ~/.claude/skills && python create-skill/scripts/package_skill.py <path-to-skill>
```

This creates `<skill-name>.zip` (zip format), excluding evals, `__pycache__`, and build artifacts. Note the output path — you'll use it in Step 4.

---

## Step 4: Update GitHub

**Check for a remote:**
```bash
git -C ~/.claude/skills remote -v
```

If no remote exists, skip to Step 5 and note it in the final summary.

**Find the latest release tag:**
```bash
gh release list --repo <owner>/skills --limit 1
```

**Upload the updated asset** (use `--clobber` to overwrite the existing file):
```bash
gh release upload <latest-tag> <skill-name>.zip --clobber --repo <owner>/skills
```

The `--clobber` flag is required — without it, the upload fails if an asset with that name already exists. This is an update, not a first upload.

**Check the README entry:**
```bash
grep -n "<skill-name>" ~/.claude/skills/README.md
```

Two things to check:

1. **Download link** — if the `[Download <skill-name>.zip]` link is missing, add it.
2. **Skill description** — read the skill's entry in the README Skills section and compare it to the current `SKILL.md` description and triggers. If the name, one-liner, triggers, or requirements have changed, update the entry. Don't skip this — a skill whose README entry doesn't match what it actually does is misleading to anyone reading the repo.

If anything changed, commit and push:

```bash
git -C ~/.claude/skills add README.md
git -C ~/.claude/skills commit -m "Update README entry for <skill-name>"
git -C ~/.claude/skills push
```

If nothing changed, skip the commit.

---

## Step 5: Update Agentman

**Update SKILL.md content:**

Call `update_skill` with:
- `slug`: the `name` value from frontmatter
- `content`: the full content of `SKILL.md` including frontmatter

If `update_skill` returns a "not found" error, stop and tell the user: "This skill doesn't exist in Agentman yet. Use `create-skill` to publish it for the first time — that flow handles initial Agentman creation."

**Update auxiliary files:**

For each file found in `references/`, `scripts/`, and `assets/`, call `save_skill_resource` with:
- `slug`: same slug as above
- `file_path`: relative path from the skill root (e.g., `references/schemas.md`)
- `content`: full file content

Read each file before calling — don't pass stale content. Do this for every auxiliary file, not just ones you think changed. Agentman has no diffing; you're replacing the full resource each time.

**Publish the draft:**

`update_skill` creates a draft version automatically. Call `publish_skill` with the slug to make the changes live. Without this step, the update exists in Agentman but isn't active.

---

## Step 6: Confirm

Report what happened:

```
Published: <skill-name>

✓ Validated — no errors
✓ Packaged → <skill-name>.zip
✓ GitHub — release asset updated (tag: <tag>)
✓ Agentman — SKILL.md updated
✓ Agentman — <N> auxiliary files synced
✓ Agentman — draft published live

Skill is now current across GitHub and Agentman.
```

If any step failed, report it clearly and leave the others as ✓ so the user knows what did land.

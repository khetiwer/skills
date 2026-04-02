---
name: "apply-job"
description: "Job application workflow: fetches JD, creates folder, runs fit analysis, tailors resume and cover letter. TRIGGERS: 'apply to this job', 'apply to this role', any job URL (LinkedIn, Greenhouse, Lever, Workday), 'job application', 'tailor my resume for', 'write a cover letter for'. Use when the user shares a job posting URL or asks to apply to a specific role."
version: "1.5.0"
category: "Productivity"
tags: ["job-search", "resume", "cover-letter", "career", "application", "tailoring", "apply", "job-posting", "linkedin"]
---

# Job Application Workflow

Help apply to a job posting at: $ARGUMENTS

## Setup

Before using this skill, create these files in your project root. See references/SETUP.md for templates and guidance.

| File | Purpose |
|------|---------|
| PROFILE.md | Background, strengths, gaps, target roles |
| BULLETS.md | Pre-written resume bullet variations by role focus |
| CONTEXT.md | Scripts, tracker commands, workflow preferences, writing style rules |

## Workflow Steps

### 1. Fetch Job Description

Use WebFetch to extract the full job posting. If WebFetch fails (JavaScript-rendered page, empty content): use Playwright MCP as fallback:
- browser_navigate to the job URL
- browser_snapshot to get full page content
- Extract the JD from the snapshot

Identify: company name, role title, responsibilities, requirements, preferred qualifications.

### 2. Add to Tracker & Create Folder

Run the tracker add command from CONTEXT.md with the company name, role, URL, priority, and fit level.

Adapt this command to the user's tracker (Google Sheets, Notion, Airtable, etc.). See CONTEXT.md for specific tracker commands.

**IMPORTANT:** Check CONTEXT.md for the correct folder location. Per-application folders typically go inside `applications/`, NOT at the project root. Follow whatever folder structure CONTEXT.md specifies.

### 3. Save Job Description

Save the full job description to a text file in the application folder.

### 4. Analyze Fit & Verdict

**MANDATORY:** Read PROFILE.md before analyzing fit.

Create a two-column fit analysis mapping their requirements to the candidate's experience. Include specific metrics where possible.

After analysis, classify each gap as either:
- **Frameable gap** — You don't have it on paper but can make the case with narrative. The story closes most of the distance. (e.g., "No pure SaaS experience" → reframe as recurring-revenue B2B with same strategic levers)
- **Structural gap** — A hard requirement that can't be reframed. No amount of good writing gets you there. (e.g., "Must have 5+ years in fintech" when you don't)

Then issue a **Fit Verdict** before proceeding:

| Verdict | Meaning | Action |
|---|---|---|
| **Strong Fit** | Clear match across core requirements | Proceed automatically |
| **Investable Stretch** | Frameable gaps only, no structural gaps | Proceed automatically |
| **Long-Shot Stretch** | One or more structural gaps, but high-upside role | Pause — flag to user and ask if they want to proceed |
| **Weak Fit** | Multiple structural gaps or fundamental mismatch | Stop — surface blockers clearly, do not continue to tailoring |

**Output format for the verdict block:**
```
Fit Verdict: [verdict]
Frameable gaps: [list, or "none"]
Structural gaps: [list, or "none"]
Proceeding to tailoring: [yes / flagging to user first]
```

For Long-Shot Stretch: surface the structural gaps clearly, then ask: "This has [gap]. Want me to proceed with the application anyway?"
For Weak Fit: list the blockers and stop. Do not proceed to tailoring without explicit user confirmation.

After the verdict, identify:
- Which base resume is closest to this role's focus
- Any significant gaps to flag in the cover letter strategy

Save the full analysis (including verdict block) as `Fit Analysis - Company.txt` in the application folder.

### 5. Tailor Resume

Start from the ATS-optimized base resume (Workday-friendly, proper heading styles).

**MANDATORY:** Read BULLETS.md and swap in the right variations.
**MANDATORY:** Read CONTEXT.md for writing style rules before editing.

Step-by-step:
1. Identify the role's focus (e.g., Data, API, Enterprise, AI/ML, Consumer, etc.)
2. Read BULLETS.md and find the matching variation for each section
3. Swap in the closest variation — do NOT leave bullets as the base version if a better variation exists
4. If no variation fits, write new language and add it to BULLETS.md for future use

Only three things change per application: title, summary, and bullets. Everything else is frozen.

**Avoid Overfitting:**
- DO swap in the right bullet variations from BULLETS.md
- DO adjust title and summary to match role focus
- DON'T rewrite bullets to parrot the job description verbatim
- DON'T invent accomplishments or metrics
- DON'T make it sound like a keyword dump
- KEEP the candidate's authentic voice

Keep EXACTLY as-is:
- All job titles and dates
- Company names
- Education, certifications, skills sections
- Overall structure and formatting

#### Resume Editing Method (python-docx)

The base resume .docx has text split across multiple XML runs within paragraphs. This means a sentence like "Scaled product teams from 2 to 20+" may be stored as separate runs ("2", "0+"). Direct string replacement on individual runs will fail.

**CRITICAL: Formatting-safe replacement.** The base resume uses distinct formatting for "prefix" runs vs. "content" runs within the same paragraph. Two common patterns:

1. **Bullet paragraphs:** Run 0 is `"• "` (blue, 2E5090), Run 1+ is body text (black/inherited).
2. **Label:value paragraphs** (e.g., Skills section): Run 0 is `"Domains: "` (blue, bold), Run 1+ is the value text (black/inherited).

A naive approach that puts all replacement text into Run 0 will inherit Run 0's formatting (blue color, bold) for the entire paragraph. The formatting-safe method below detects both patterns, leaves the prefix/label run untouched, and places the replacement text into the correct content run.

**Use this formatting-safe paragraph-level replacement approach:**

```python
from docx import Document
from docx.shared import RGBColor

def replace_paragraph_text(doc, old_text, new_text):
    """Replace text across a paragraph, preserving per-run formatting.

    Handles two common resume patterns where Run 0 has different styling
    from subsequent runs:

    Pattern 1 - Bullet prefix:
      Run 0: "• " (blue/colored)  |  Run 1+: body text (black/inherited)

    Pattern 2 - Label:value:
      Run 0: "Label: " (blue, bold)  |  Run 1+: value text (black/inherited)

    In both cases, the replacement text is placed into Run 1 (the first
    content run), preserving Run 0's distinct formatting. All runs after
    Run 1 are cleared.

    If the paragraph has only one run, text is replaced in place and
    the original formatting is preserved.
    """
    for para in doc.paragraphs:
        full_text = ''.join(run.text for run in para.runs)
        if old_text in full_text:
            new_full = full_text.replace(old_text, new_text)

            if len(para.runs) <= 1:
                # Single run — replace in place, formatting preserved
                if para.runs:
                    para.runs[0].text = new_full
                return True

            # Multi-run: check if Run 0 is a styled prefix that should
            # be preserved separately from the content runs.
            prefix_run = para.runs[0]
            prefix_text = prefix_run.text
            has_different_formatting = False

            # Check if Run 0 has explicit color different from Run 1
            r0_color = prefix_run.font.color.rgb if prefix_run.font.color and prefix_run.font.color.rgb else None
            r1_color = para.runs[1].font.color.rgb if para.runs[1].font.color and para.runs[1].font.color.rgb else None
            if r0_color and r0_color != r1_color:
                has_different_formatting = True

            # Check if Run 0 is bold but Run 1 is not
            if prefix_run.font.bold and not para.runs[1].font.bold:
                has_different_formatting = True

            # Detect known prefix patterns
            bullet_chars = {'•', '●', '◦', '▪', '▸', '‣', '-', '–', '—'}
            is_bullet_prefix = (
                len(prefix_text.strip()) <= 2
                and any(prefix_text.strip().startswith(c) for c in bullet_chars)
            )
            is_label_prefix = (
                prefix_text.rstrip().endswith(':')
                and len(prefix_text) <= 30
            )

            should_preserve_prefix = (
                has_different_formatting
                and (is_bullet_prefix or is_label_prefix)
                and len(para.runs) >= 2
            )

            if should_preserve_prefix:
                # Keep prefix run intact, put new text (minus prefix) into run 1
                content_text = new_full[len(prefix_text):]
                para.runs[1].text = content_text
                for run in para.runs[2:]:
                    run.text = ''
            else:
                # No styled prefix detected — put everything in run 0,
                # clear the rest
                para.runs[0].text = new_full
                for run in para.runs[1:]:
                    run.text = ''

            return True
    return False

# Usage:
doc = Document('base/resume_ats_base.docx')
replace_paragraph_text(doc,
    'old bullet text here',
    'new bullet text here')
doc.save('applications/Company - Role/Khetiwe-Richards_Company_resume_2026Mon.docx')
```

**Why this matters:** Many resumes use a colored or bold-styled prefix (bullet character or label like "Domains:") as the first XML run, with the body/value text in a separate run using different formatting. The old approach collapsed all text into Run 0, causing the entire paragraph to inherit the prefix's blue color or bold weight. This formatting-safe method detects both bullet prefixes ("• ") and label prefixes ("Label: ") by checking for formatting differences between Run 0 and Run 1, then preserves each run's original styling.

**If `python-docx` is not installed:** Run `pip install python-docx` first.

### Naming Conventions

- Resume: `Khetiwe-Richards_Company_resume_2026Mon.docx`
- Cover Letter: `Cover Letter - Company.docx`
- Fit Analysis: `Fit Analysis - Company.txt`
- Job Description: `JD - Company - Role.txt`

### 6. Write Cover Letter

Style: Direct, no fluff, ~250 words max. Format: .docx.

**MANDATORY:** Read CONTEXT.md for writing style rules and cover letter formatting specs before writing. If a writing style reference file is specified in CONTEXT.md, read that file too.

**Cover Letter Strategy**

The goal is problem-solution framing: identify what the company needs, position the candidate as the answer.

Before writing, research the company (5-10 min):
- Search for recent news: product launches, funding rounds, partnerships, blog posts
- Check LinkedIn for the hiring manager's name and recent posts
- Look for team blog posts, conference talks, or product announcements
- Find ONE specific detail to reference in the closing

**Structure:**

1. Date
2. Company name and department
3. RE: line with role title and job ID (address hiring manager by name if found)
4. **Opening hook** — Don't just state "I'm applying." Lead with the most compelling match between the candidate's experience and their core need. One sentence that makes them want to keep reading.
5. **Problem-solution body** — 2-3 sentences connecting specific results to what they're trying to solve. Use metrics. Frame it as "you need X, I've done X."
6. **"What drew me to this role:"** with 2-3 bulleted points (bold label + description). Show genuine understanding of the company/product, not generic skills.
7. **Company-specific research line** — One sentence referencing something specific found during research (a recent product launch, blog post, funding round). This single line signals real research and dramatically increases response rates.
8. **Confident closing** — Not "I'd welcome the opportunity" (flat). Use forward momentum: "I'd love to walk you through how I'd approach this challenge."
9. Closing and name — use whatever closing style is specified in CONTEXT.md

**Tone Guidelines:**
- Write like a peer, not a supplicant
- Lead with outcomes, not responsibilities
- Show understanding of their product/market, not just the job description
- Every sentence should earn its place — cut anything generic
- Follow all writing style rules from CONTEXT.md (em dash policy, closing style, etc.)

**Formatting specs:** Calibri 11pt, 1" margins. No header — start with date. Bullets: bold label + " – " + description. 12pt spacing after paragraphs, 4pt after bullets.

### 7. Upload to Drive

Upload (if configured in CONTEXT.md):
- `Fit Analysis - Company.txt`
- `Khetiwe-Richards_Company_resume_YearMonth.docx`
- `Cover Letter - Company.docx`

If CONTEXT.md says no Drive upload, skip this step.

### 8. Present for Review

Show the candidate:
- Full change log with FROM/TO for each resume edit (format below)
- The cover letter text for review
- Reminder to mark as applied in the tracker after submitting (if tracker is configured)

**Change Log Format (REQUIRED):**

```
## Resume Changes

**Title**
- FROM: "...previous focus area"
- TO: "...new focus area"

**Summary**
- FROM: "...previous framing..."
- TO: "...new framing..."

**Company Bullet #**
- FROM: "...previous bullet..."
- TO: "...new bullet..."
```

This keeps reviews concise while still catching overfitting.

### 9. Workday Detection

If the job URL contains `myworkdayjobs.com` or `wd1.myworkdayjobs`, prompt:

> "This is a Workday application. When you're ready to fill it out, navigate to the application and run /workday-apply Company — it'll handle work experience and standard questions via browser automation."

## Key Scripts

These scripts power the workflow. See references/SETUP.md for how to build or adapt them.

| Script | Purpose |
|--------|---------|
| sheets_tracker.py | Add jobs, mark applied, update status |
| upload_to_drive.py | Upload files to Drive folders |
| read_docx.py | Extract text from Word docs |

## Anti-Patterns

- Don't parrot the job description back as resume bullets
- Don't fabricate metrics or experience the candidate doesn't have
- Don't write generic "tell me about yourself" cover letters
- Don't skip the fit analysis — it's the backbone of the tailoring decision
- Don't over-tailor — a keyword-stuffed resume reads worse than a clean one
- Don't create application folders at the project root — check CONTEXT.md for correct location
- Don't skip reading the writing style reference before writing cover letters
- Don't edit .docx XML directly — use the python-docx method above to handle split runs
- Don't collapse all paragraph text into Run 0 — this breaks formatting when the bullet character or label prefix has different styling (e.g., colored bullets, bold labels). Always use the formatting-safe replacement method.
- Don't proceed to tailoring on a Weak Fit without explicit user confirmation
- Don't skip the Fit Verdict block — it must appear in the saved Fit Analysis file

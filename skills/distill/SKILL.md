---
name: distill
description: Use when processing source documents (DOCX, PDF, transcriptions) into structured markdown knowledge bases (_PK). Also use when the user mentions "distill", "distiller", "depouiller", "extraire", "PK", "base de connaissances", or wants to transform raw documents into reusable markdown files.
---

# Distill — Document-to-Knowledge Extraction

## Overview

**Distill** transforms raw source documents into structured markdown knowledge bases called **_PK** (Project Knowledge). Extract all substantive information once, store it in lightweight markdown files, and never re-read the heavy source document again.

**Core principle:** Distill, don't summarize. Preserve verbatim facts, remove only markup noise.

## Commands

| Command | Action |
|---------|--------|
| `distill help` | Display usage guide with pedagogy and examples |
| `distill new <file>` | Create a new _PK from a source document |
| `distill add <file>` | Enrich an existing _PK with a new document |

**Options:**
- `--into <path>` — target a specific _PK directory
- `--dry-run` — show proposed structure without creating files

## When invoked without arguments OR with `help`

Display the following to the user:

```
DISTILL — Document-to-Knowledge Extraction

PK = Project Knowledge

Transforms source documents (DOCX, PDF, transcriptions) into a
structured markdown knowledge base, optimized for working with
LLMs (Claude, Gemini, etc.).

WHY?
  A DOCX or PDF file consumes 5-10x more tokens than equivalent
  markdown. Distill once, then work with lightweight, targeted,
  on-demand reloadable files.

COMMANDS:
  distill help              Show this help
  distill new <file>        Create a new _PK from a document
  distill add <file>        Enrich an existing _PK

OPTIONS:
  --into <path>             Target a specific _PK directory
  --dry-run                 Preview without creating files

SUPPORTED TYPES:
  DOCX        Text, comments, tracked changes
  PDF text    Direct extraction in 10-20 page chunks
  PDF scan    Vision-based extraction (OCR) page by page
  Transcr.    Meeting transcriptions (Google Meet/Gemini) in DOCX

STRUCTURING RULES:
  - Under 800 lines per markdown file
  - One theme = one subdirectory
  - Zero redundancy across files
  - Navigable index (00_index.md)
  - Source traceability (_sources.md)

EXAMPLES:
  distill new audit-report.docx
  distill add meeting-notes.docx --into _PK_ProjectName
```

Then STOP. Do not proceed with any extraction.

---

## Workflow — Phase 0: Setup

Perform all detection silently, then present a **single confirmation summary** to the user.

### Detection steps (silent)

1. **Working directory** — note the current directory
2. **Existing _PK** — scan current directory 3 levels deep for directories matching `_PK*`
3. **Source file** — check the file exists and identify its type:

| Extension | Content test | Type assigned |
|-----------|-------------|---------------|
| `.docx` | Contains timestamps + speaker names | `transcription` |
| `.docx` | Has `comments.xml` or tracked changes | `docx-annotated` |
| `.docx` | Standard | `docx` |
| `.pdf` | Read returns text | `pdf-text` |
| `.pdf` | Read returns empty or garbled | `pdf-scan` |

### Single confirmation

Present all findings at once:
```
DISTILL — Summary before launch

Working directory  : {current_directory}
Source file        : {filename} ({size})
Detected type      : {type}
Target _PK         : {path} ({status: empty / N files})

Proceed? (yes/no)
```

**If `distill new` and a _PK* already exists with content:**
- Warn: "A _PK with content already exists: {path}."
- If existing _PK is empty → use it silently, no warning needed.
- If user wants a new one → ask for suffix: `_PK_{Suffix}`
- Default location: same parent directory as existing _PK

**If `distill add`:**
- If one _PK found → use it
- If multiple found → list and ask which one
- If none found → error: "No _PK found. Use `distill new` first."

---

## Workflow — Phase 1: Extraction

### Extraction tools — priority order

1. **pandoc** (if installed): `pandoc --track-changes=all doc.docx -o output.md` — best quality
2. **docx skill unpack + Python**: unpack ZIP → parse XML with Python — reliable fallback
3. **Read tool directly**: last resort for simple documents without annotations

Check pandoc availability first. If not installed, use the docx skill's `unpack.py` to decompress, then parse XML with Python.

**Temp files**: store intermediate extractions in the system temp directory. Clean up after Phase 1.

### DOCX (standard)

1. Extract text content → convert to markdown (using tool above)
2. Discard source format — work only with markdown from here

### DOCX (annotated)

1. Unpack the DOCX (ZIP → XML) using docx skill's `unpack.py`
2. Extract text content from `document.xml` → convert to markdown via Python
3. Extract annotators from `people.xml` → list of names + emails
4. Extract all comments from `comments.xml`:
   - Author, date, comment text (verbatim)
   - Anchor location (which paragraph/section the comment is attached to)
5. Extract all tracked changes from `document.xml`:
   - Author, date, type (insertion/deletion)
   - Original text and replacement text
6. Discard XML — work only with extracted data from here

### DOCX (transcription — Google Meet/Gemini)

1. Unpack or read the DOCX
2. Identify structure: summary section, transcript section
3. Extract:
   - Meeting metadata (date, duration, participants)
   - Summary (as-is)
   - Transcript with timestamps and speaker names
4. Discard source format — work only with extracted data

### PDF (text)

1. Read document in chunks of 10-20 pages
2. For each chunk: extract text, note page numbers
3. Concatenate into working markdown

### PDF (scan)

1. Attempt Read — if empty/garbled, confirm scan mode with user
2. Convert each page to image
3. Use vision to extract text page by page
4. Concatenate into working markdown, noting page numbers

**CRITICAL:** After Phase 1, the raw source is no longer needed in context. All subsequent phases work with the extracted markdown and annotation data only.

---

## Workflow — Phase 2: Cartography

1. Read through extracted content
2. Identify major themes/sections
3. Propose directory and file structure:

```
_PK_{Name}/
  00_index.md                  Index — map of the knowledge base
  _sources.md                  Traceability — source file to output files

  {theme-a}/                   Subdirectory per major theme
    {topic}-01.md              Thematic file, under 800 lines
    {topic}-02.md              Incremented if needed
    ...

  _annotateurs/                Only if source has annotations
    00_synthesis.md            Cross-cutting synthesis by theme
    {firstname_lastname}.md    One file per annotator
    ...

  meetings/                    Only if source is a transcription
    YYYY-MM-DD_{subject}/
      00_index.md              Metadata
      01_summary.md            Summary
      02_decisions.md          Decisions
      03_actions.md            Action items (who, what, when)
      04_open_questions.md     Unresolved points
      05_transcript-01.md      Full transcript (split if > 800 lines)
```

4. **STOP — Present structure to user and wait for validation.**
   Do NOT create any files until the user approves.

---

## Workflow — Phase 3: Distillation

Create markdown files following these rules:

### Content rules

- **Verbatim facts** — do not paraphrase, do not interpret, do not summarize
- **Single source of truth** — each piece of information exists in ONE file only
- **Cross-reference** — other files link to it: `see [file.md](../theme/file.md)`
- **Traceability** — each file begins with: `> Source: {document name}, section {X.Y}`

### Size rules

- **Target:** < 800 lines per file
- **Hard maximum:** 2000 lines (exceptional cases only)
- **Overflow:** split with incrementation: `topic-01.md`, `topic-02.md`
- **Warn** at 600 lines that the file is approaching the limit

### Annotation format — inline in thematic files

For comments:
```markdown
> **[Comment — {Author}, {DD/MM/YYYY}]**
> "{verbatim comment text}"
```

For tracked changes:
```markdown
> **[Tracked change — {Author}, {DD/MM/YYYY}]**
> DELETED: "{deleted text}"
> ADDED: "{inserted text}"
```

For insertion only (no deletion):
```markdown
> **[Tracked change — {Author}, {DD/MM/YYYY}]**
> ADDED: "{inserted text}"
```

For deletion only (no insertion):
```markdown
> **[Tracked change — {Author}, {DD/MM/YYYY}]**
> DELETED: "{deleted text}"
```

### Annotator files — `_annotateurs/{name}.md`

```markdown
# {Full Name} — Annotations

> Source: {document name}
> Number of annotations: {N}

## Annotation 1 ({DD/MM/YYYY})
> "{verbatim comment text}"
**Context**: Section {X.Y}, {brief description of surrounding text}
**File**: [see {file}](../theme/{file}#anchor)

## Annotation 2 ({DD/MM/YYYY})
...
```

### Index file — `00_index.md`

```markdown
# {Project Name} — Knowledge Base

> Source(s): {list of source documents}
> Last updated: {date}

## Files

| # | File | Content | When to use |
|---|------|---------|-------------|
| 01 | [theme/topic-01.md](theme/topic-01.md) | {brief content desc} | {when to load this file} |
| ... | ... | ... | ... |
```

### Sources file — `_sources.md`

```markdown
# Source Traceability

| Source document | Date | Generated files |
|----------------|------|-----------------|
| {filename.docx} | {date} | topic-01.md, topic-02.md, ... |
| {filename.pdf} | {date} | topic-03.md, ... |
```

---

## Workflow — Phase 4: Verification

Run these checks and report results:

| Check | What | Pass criteria |
|-------|------|--------------|
| **Coverage** | Every section of source document appears in a file | 100% sections mapped |
| **Size** | No file exceeds 800 lines | Warn > 600, fail > 2000 |
| **Redundancy** | No information duplicated across files | Each fact in one place only |
| **Traceability** | `_sources.md` maps every source → output files | All sources listed |
| **Index** | `00_index.md` lists every file with accurate descriptions | Complete and current |
| **Cross-refs** | All internal links resolve to existing files | No broken links |

Report to user. Fix any failures before declaring complete.

---

## Mode `add` — Enrichment workflow

When adding a new document to an existing _PK:

### Step 1 — Read existing _PK
Load `00_index.md` and `_sources.md` to understand current structure.

### Step 2 — Extract new document
Follow Phase 1 (Extraction) for the new document.

### Step 3 — Compare
For each piece of information in the new document:
- Is it already in the _PK? → mark as SKIP
- Does it update existing content? → mark as UPDATE (specify which file)
- Is it entirely new? → mark as CREATE (propose new file or section)

### Step 4 — Propose changes
Present a summary to the user:
```
Analysis of {name} vs existing _PK:

CREATE:
  - _annotateurs/jane_doe.md (5 annotations)
  - _annotateurs/john_smith.md (7 annotations)

UPDATE:
  - architecture/technical-01.md (2 modified paragraphs)

UNCHANGED:
  - 18 sections identical to existing content

Proceed? (yes / no / detail)
```

### Step 5 — Execute
After user validation:
1. Create new files
2. Update existing files (add new content, flag modifications)
3. Update `00_index.md`
4. Update `_sources.md`
5. Run Phase 4 (Verification)

---

## Common mistakes

| Mistake | Prevention |
|---------|-----------|
| Creating files before user validates structure | Phase 2 MUST end with explicit user validation |
| Paraphrasing source content | Always verbatim — distill, don't summarize |
| Putting same info in multiple files | One location only, cross-reference everywhere else |
| Ignoring tracked changes in DOCX | Always check for comments.xml and tracked changes |
| Processing PDF scan as text PDF | Test Read first; if empty → switch to vision mode |
| Single huge markdown file | Split at 800 lines with incrementation |
| Keeping XML/source in context after extraction | Discard source format after Phase 1 |
| Working from memory of a previous _PK | Always Read the current 00_index.md before `add` |

## File naming conventions

- **Directories:** `snake_case`, lowercase, no accents
- **Files:** `snake_case` with numeric prefix: `01_topic.md`
- **Increments:** `topic-01.md`, `topic-02.md`
- **Annotators:** `firstname_lastname.md` (lowercase, no accents)
- **Meetings:** `YYYY-MM-DD_{subject}/`
- **Index:** always `00_index.md`
- **Sources:** always `_sources.md`

<p align="center">
  <img src="banner.png" alt="Distill — Document-to-Knowledge Extraction" width="100%">
</p>

# Distill — Document-to-Knowledge Extraction for Claude Code

Transform heavy source documents (DOCX, PDF, meeting transcriptions) into structured markdown knowledge bases optimized for LLM context windows.

## The Problem

A 30-page DOCX file consumes **5-10x more tokens** than equivalent markdown when loaded into an LLM context. Every time you re-read the source document, you burn tokens on XML markup, formatting tags, and structural noise that carry zero informational value.

**Distill** extracts the substance once, stores it in lightweight markdown files, and lets you work with targeted, on-demand reloadable knowledge — never touching the heavy source again.

## What It Does

```
Source document (DOCX/PDF)
    ↓ Phase 1: Extraction
Raw text + comments + tracked changes
    ↓ Phase 2: Cartography
Proposed file/folder structure (user validates)
    ↓ Phase 3: Distillation
Structured markdown files (_PK)
    ↓ Phase 4: Verification
Coverage, size, redundancy, traceability checks
```

### Output: a `_PK` (Project Knowledge) folder

```
_PK/
├── 00_index.md              ← Navigable map of the knowledge base
├── _sources.md              ← Traceability: source → output files
├── topic_a/
│   ├── subtopic-01.md       ← Under 800 lines, verbatim facts
│   └── subtopic-02.md       ← Incremented if needed
├── topic_b/
│   └── ...
├── _annotateurs/            ← If source has comments/annotations
│   ├── 00_synthesis.md      ← Cross-cutting themes
│   └── jane_doe.md          ← One file per annotator
└── meetings/                ← If source is a transcription
    └── 2025-03-15_kickoff/
        ├── 00_index.md
        ├── 01_summary.md
        ├── 02_decisions.md
        ├── 03_actions.md
        └── 04_transcript-01.md
```

## Commands

| Command | What it does |
|---------|-------------|
| `/distill help` | Show usage guide |
| `/distill new <file>` | Create a new _PK from a source document |
| `/distill add <file>` | Enrich an existing _PK with a new document |

**Options:**
- `--into <path>` — target a specific _PK directory
- `--dry-run` — preview the proposed structure without creating files

## Supported Document Types

| Type | How it works |
|------|-------------|
| **DOCX** | Unpacks XML, extracts text, comments, and tracked changes |
| **DOCX (annotated)** | Same + extracts per-author annotations with context |
| **DOCX (transcription)** | Splits into summary, decisions, actions, transcript |
| **PDF (text)** | Reads in 10-20 page chunks |
| **PDF (scan)** | Vision-based OCR, page by page |

## Key Rules

- **< 800 lines** per markdown file (hard max: 2000)
- **Zero redundancy** — each fact exists in one place, others cross-reference
- **Verbatim** — distill, don't summarize
- **Traceability** — every file traces back to its source
- **User validates** the structure before any files are created

## Installation

### Claude Code

Copy the `skills/distill/` folder into your Claude Code skills directory:

```bash
# Personal skills (available in all projects)
cp -r skills/distill ~/.claude/skills/distill

# Or project-specific
cp -r skills/distill .claude/skills/distill
```

Then invoke with `/distill` in Claude Code.

### Other AI Coding Agents

The `SKILL.md` file is a standard skill format. It can be adapted for:
- **GitHub Copilot CLI** — place in your skills directory
- **Gemini CLI** — activate via `activate_skill`
- **Any LLM agent** — use the SKILL.md as a system prompt or reference document

## Use Cases

- **Contract review** — Distill a 50-page contract into themed files, extract reviewer annotations per person
- **Meeting prep** — Process multiple source documents into one knowledge base, load only what you need per meeting
- **Project knowledge base** — Build incrementally with `distill add` as new documents arrive
- **Stakeholder tracking** — Extract who said what from annotated documents, organized per person
- **Due diligence** — Process a folder of documents one by one, building a unified searchable base

## Why "_PK"?

**PK = Project Knowledge.** A `_PK` folder is a self-contained knowledge base built from source documents. The underscore prefix keeps it sorted at the top of directory listings, and the naming convention (`_PK`, `_PK_ClientName`, `_PK_Phase2`) allows multiple knowledge bases to coexist in the same project.

## Design Principles

1. **Distill, don't summarize.** Preserve the original words. Summaries lose nuance; distillation removes noise.
2. **One fact, one place.** If two files need the same information, one stores it and the other links to it.
3. **User controls the structure.** The skill proposes, the user validates. No files are created without approval.
4. **Source documents are disposable after extraction.** Once distilled, the heavy source is never loaded into context again.
5. **Incremental by design.** New documents enrich the existing base; they don't replace it.

## Token Efficiency

| Format | Relative token cost | Notes |
|--------|-------------------|-------|
| DOCX (raw XML) | 10x | Mostly markup noise |
| PDF (via Read) | 3-5x | Better than DOCX, still heavy |
| Markdown (_PK) | 1x | Pure content, no noise |

A 30-page technical document (~15,000 tokens as DOCX XML) typically distills to ~3,000 tokens of structured markdown — an **80% reduction** while preserving 100% of the substantive content.

## License

MIT — see [LICENSE](LICENSE).

## Author

Created by [Romaric DOVI](https://github.com/romaricvivien65) — built with Claude Code.

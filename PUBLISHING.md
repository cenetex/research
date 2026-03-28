# Publishing Process for RATiMICS Research

This document outlines how papers are ingested, analyzed, and published as research posts in the Cenetex research pipeline.

## Overview

The research pipeline follows this flow:

```
Paper Identified
    ↓
Extract Key Claims & Testable Hypotheses
    ↓
Map to RATiMICS Architecture
    ↓
Design Experiments
    ↓
Publish Research Post
    ↓
Run Experiments
    ↓
Publish Results Post
    ↓
Iterate on Protocol Design
```

## Step 1: Paper Ingestion

When a new paper is identified as relevant to RATiMICS:

1. **Create extraction file** in `papers/` directory with filename format: `lastname-coauthors-year.md`
   - Example: `evans-bratton-aguera-2026.md`

2. **Extract key claims** (8-12 main assertions from the paper)
   - Focus on claims about intelligence, institutions, coordination, or alignment
   - Include DOI, publication details, author names

3. **Draft testable hypotheses** (3-6 hypotheses that can be measured in RATiMICS)
   - Use format: "H#: [Hypothesis name]"
   - Define measurement approach
   - Show RATiMICS connection

## Step 2: Publish Research Post

Once extraction is complete:

1. **Create post** in `posts/` directory with filename: `YYYY-MM-DD-slug.md`
   - Example: `2026-03-28-evans-extraction.md`

2. **Post structure:**
   - Title linking to DOI
   - "Why This Paper Matters" section
   - Summary of key claims
   - Mapping table (Paper claim → RATiMICS implementation)
   - Full hypothesis sections (each with measurement approach)
   - Next steps / experiment readiness

3. **Add to README** post listing

## Step 3: Design Experiments

For each hypothesis:

1. **Create GitHub issue** on relevant Cenetex repo (e.g., cenetex/agent-orchestrator for coordination tests)
   - Title: "Hypothesis {N}: {Title} (Evans et al. 2026)"
   - Label: `research-driven`
   - Link to research post

2. **Issue should include:**
   - Hypothesis statement
   - Success metrics
   - Test protocol
   - Expected outcomes

## Step 4: Run & Document Results

As experiments complete:

1. **Create results post** in `posts/` with filename: `YYYY-MM-DD-hypothesis-N-results.md`

2. **Include:**
   - Experiment summary
   - Measurement data
   - Key findings
   - Implications for RATiMICS design
   - Links to code + GitHub issues used in experiment

3. **Update protocol** if results warrant design changes

## Directory Structure

```
research/
├── README.md                          # Pipeline overview + post index
├── WHITEPAPER.md                      # RATiMICS protocol specification
├── PUBLISHING.md                      # This file
│
├── papers/
│   └── lastname-coauthors-year.md     # Paper extractions
│
└── posts/
    ├── YYYY-MM-DD-slug.md             # Research posts (extraction → results)
    └── YYYY-MM-DD-hypothesis-slug.md  # Results posts
```

## Publishing to the Web

### Git-Native Publishing

All content is Markdown in this git repository. To make it public:

1. **Push to public repository** — content is live immediately
   - `git push origin main`

2. **Static site generator** (optional) — render posts on ratimics.com
   - Example: Jekyll, Hugo, 11ty
   - Read posts from `posts/` directory
   - Render as blog posts with metadata

3. **Permanent URLs** — each post is immutable
   - URL: `ratimics.com/research/2026-03-28-evans-extraction`
   - Backed by git commit history
   - Permanent reference for citations

### Website Integration

To deploy to ratimics.com:

1. **Set up static site in a separate repo** that pulls this content
2. **Each post becomes a blog entry**:
   - Metadata: date, title, tags
   - Content: markdown rendered to HTML
   - Sidebar: links to paper, hypotheses, experiments

3. **Index page** lists all research posts chronologically
4. **WHITEPAPER.md** rendered as permanent documentation page

## Quality Standards

All posts must include:

- [ ] Clear connection to RATiMICS (why we care)
- [ ] Traceable to source material (DOI, author links)
- [ ] Testable hypotheses with measurement approach
- [ ] Links to actual experiments / GitHub issues
- [ ] Updated based on experimental results

## Examples

### New Paper Workflow

```bash
# 1. Add paper extraction
echo "# New Paper\n\n- **Authors:** ...\n- **Key Claims:** ..." > papers/lastname-year.md

# 2. Write extraction notes
# (extract claims and hypotheses)

# 3. Publish research post
echo "# Research Post: Title\n\n## Why This Matters\n..." > posts/2026-03-28-title.md

# 4. Create experiments
gh issue create --title "Hypothesis 1: Title (Paper Year)" --label research-driven

# 5. Push to main — it's live
git add papers/ posts/
git commit -m "Add paper extraction: lastname (year) — 6 testable hypotheses"
git push origin main
```

## Maintenance

- **Update posts** if experiments yield new insights
- **Version whitepaper** as protocol parameters change
- **Archive old posts** as more recent research supersedes them
- **Link across posts** to show how research compounds over time

---

**Last Updated:** 2026-03-28
**Status:** Initial process documentation

---
name: paper-reading
description: >
  Use this skill whenever the user wants to read, understand, summarize, or ask questions about
  an academic paper or research document. This includes reading papers from arxiv URLs, local PDF
  files, or any research document the user wants to comprehend. Triggers on phrases like "read this
  paper", "summarize this paper", "what does this paper say", "help me understand this research",
  or when the user provides an arxiv URL, a PDF file path, or mentions academic/research papers.
  Also use when the user wants to extract key findings, methodology, or results from a paper.
---

# Paper Reading Skill

Read academic papers and produce structured summaries. Supports two pipelines depending on the source:

1. **arxiv papers** — download LaTeX source for maximum fidelity (formulas, tables, structure)
2. **local PDFs** — convert to Markdown using Marker (vision-based OCR pipeline), then read

## Prerequisites

Marker is installed via uv in the bundled `marker-env/` directory (relative to this skill's base directory). Always invoke it with the correct path resolved from the skill base:

```bash
uv run --project <skill-base-dir>/marker-env marker_single [options]
```

## Workflow

### Step 1: Determine source type

- If the user provides an **arxiv URL** (e.g. `https://arxiv.org/abs/XXXX.XXXXX`), go to **Pipeline A**.
- If the user provides a **local PDF path**, go to **Pipeline B**.
- If the paper content is already cached locally, go to **Step 3**.

### Step 2: Run the appropriate pipeline

#### Pipeline A: arxiv LaTeX Source

arxiv papers should ideally be read from their LaTeX source because it preserves the exact structure the authors intended — no OCR errors on formulas or tables.

1. **Normalize the URL**: Extract the arxiv ID (e.g. `2601.07372` from `https://arxiv.org/abs/2601.07372`)

2. **Download TeX source**:
   ```bash
   mkdir -p ~/.cache/paper-reading/{arxiv_id}
   curl -L -o ~/.cache/paper-reading/{arxiv_id}/source.tar.gz https://arxiv.org/src/{arxiv_id}
   ```

3. **Unpack**:
   ```bash
   tar xzf ~/.cache/paper-reading/{arxiv_id}/source.tar.gz -C ~/.cache/paper-reading/{arxiv_id}/
   ```

4. **Find the entrypoint**: Look for `main.tex`, `paper.tex`, or the root `.tex` file. Check for `\input` or `\include` directives to find all sub-files.

5. **Read all source files**: Start from the entrypoint, recursively read all referenced `.tex` files.

#### Pipeline B: PDF via Marker

For non-arxiv PDFs or as a fallback for arxiv papers.

1. **Determine the work folder name**:
   - If the PDF is named like `ssrn-xxx.pdf` or `doi-xxx.pdf`, use the stem as-is: `ssrn-xxx`
   - If it's named by title like `Attention Is All You Need.pdf`, connect words with `-`: `attention-is-all-you-need`

2. **Convert PDF to Markdown**:
   ```bash
   uv run --project marker-env marker_single \
     --input /path/to/paper.pdf \
     --output_dir ~/.cache/paper-reading/{paper_name} \
     --output_format markdown
   ```

   This takes 1-3 minutes depending on paper length. Marker will:
   - Detect layout (text, formulas, tables, figures)
   - Recognize formulas as LaTeX
   - Convert tables to Markdown tables
   - Extract figures as JPEG files

3. **Read the output**: The main output is at `~/.cache/paper-reading/{paper_name}/{paper_name}.md`. Read this file — it contains the full paper content in Markdown with LaTeX math.

4. **Check extracted figures** (optional): If the paper has important charts/diagrams, look in `~/.cache/paper-reading/{paper_name}/images/` for extracted JPEG files.

### Step 3: Generate Summary

Produce a structured summary markdown file. Save it to `./knowledge/summary_{tag}.md` where `{tag}` is a short descriptive name derived from the paper topic (e.g. `conditional_memory`, `0dte_options`). Check that the tag doesn't collide with existing files.

Use this template:

```markdown
# {Paper Title}

**Authors**: {authors}
**Year**: {year}
**Source**: {arxiv ID or PDF path}
**Date read**: {today's date}

## One-line Summary
{One sentence capturing the key contribution}

## Problem
{What problem does this paper address? Why does it matter?}

## Key Contributions
1. {contribution 1}
2. {contribution 2}
...

## Methodology
{How did they approach the problem? What techniques did they use?}

## Main Results
{What were the key findings? Include specific numbers if notable.}

## Strengths
- {strength 1}
- {strength 2}

## Weaknesses / Limitations
- {limitation 1}
- {limitation 2}

## Relevance
{How might this be relevant to the user's work or interests?}

## Key Equations / Models
{List important equations or models with brief explanations. Omit if not applicable.}
```

Adapt the template based on paper type — not all sections are relevant for every paper. Use your judgment.

## Step 4: Paper quality assessment

After generating the summary, give the user a brief, honest assessment of the paper. You have the full content in context — use it to judge, not guess. Address these three points concisely:

1. **Are the experiments solid?** — Did they run enough experiments? Are baselines fair and comparisons apples-to-apples? Any red flags like cherry-picked metrics, missing ablations, or suspiciously small test sets? Say yes or no with one sentence of evidence.
2. **Do the conclusions hold up?** — Are the claims supported by the data, or do they overreach? Are there obvious confounds or alternative explanations the authors ignore? Say yes or no with one sentence of evidence.
3. **Is there valuable insight regardless?** — Even if the results are weak or the methodology is flawed, does the paper introduce a useful idea, framing, or perspective worth discussing? Say yes or no with one sentence of evidence.
Keep the assessment short and direct. Do not expand into a full critique unless the user asks. The goal is to help the user quickly decide whether to invest more time in this paper.

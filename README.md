# AI Art Series — Output Progression

The output artifact and run-progression showcase for the **Art Series Generator Agent** — an end-to-end agentic pipeline that turns a natural-language design brief (theme, style, colors) into a *coherent series* of art prints and evaluates its own output against the brief.

**▶ Live showcase: https://aaron-gomoll.github.io/ai-art-series-outputs/**

This repo holds the curated image outputs across 17 runs and the page that walks through them. The product goal was a print-ready series for a kid's room; the engineering goal was the pipeline and its evaluation loop.

## The pipeline
A design brief flows through: **color interpretation (Claude) → prompt generation (Claude) → image generation → post-processing (Python/Pillow) → evaluation (Claude Vision scoring 5 dimensions + a deterministic pixel check) → logging (Langfuse + Notion)** — with every run versioned and scored, and a quality-gate retry on regressions. The eval layer is the point as much as the generation: a coherent series needs consistency across theme, color, and style that single-prompt generation rarely delivers, and AI evaluation alone misses obvious pixel-level failures.

## The interesting finding: it was the model, not the prompt
DALL-E 3 hit a hard ceiling on geometric construction — asked to build a whale from "two stacked semicircles and a dot," it draws *a whale*, ignoring the construction rules. The eval layer quantified exactly where it broke: background cleanliness never cleared 3.0/5 across 10 ocean runs, regardless of prompt changes.

Swapping **only the image model** to gpt-image-1 — same pipeline, same eval rubric, replaying the exact same prompt — jumped the score **3.35 → 4.55 (+1.20)**, with background going to a perfect 5.0. A clean controlled experiment that isolates the model as the bottleneck. The pipeline then generalized across new themes and palettes (best run **5.0/5** on gpt-image-2) and crossed vendors to **Ideogram v3** with no architecture changes.

## What it demonstrates
- **Agentic pipeline design** with a closed self-evaluation loop (generate → scored eval → quality gate)
- **Prompt iteration under measurement**, not by feel — every run logged with version, score, and what changed
- **Honest work with model limits** — diagnosing a capability ceiling, then proving it with a controlled single-variable model swap
- **Layered evaluation** (Claude Vision + deterministic pixel checks) that catches different failure modes — and which surfaced bugs *in the eval layer itself* (rubric-language bias penalizing off-theme runs; a stale-backup bug in the pixel check). Visual review remains the final backstop.

# Skills

A collection of Claude Code skills for AI research workflows and developer tooling.

## Installation

```bash
claude skill install Linwei94/skills/<skill-name>
```

---

## Available Skills

### [`ai-research-paper`](./ai-research-paper/)

**End-to-end AI/ML research pipeline targeting CCF-A venues** (NeurIPS, ICML, ICLR, CVPR, ECCV, ACL, AAAI).

Guides researchers through the entire lifecycle of an academic paper with built-in quality gates and multi-agent debate mechanisms.

#### 9-Phase Workflow

| Phase | Description |
|-------|-------------|
| 1. Idea Exploration | Literature review, brainstorming, **6-agent idea debate** with Area Chair (AC) gate |
| 2. Research Proposal | Detailed method description, theory, positioning, expected contributions |
| 3. Experiment Planning | Comprehensive experimental design at top-venue scale |
| 4. Pilot Experiments | Proof-of-concept, baseline reproduction, sanity checks |
| 5. Method Iteration | Revise method based on pilot results (feedback loop to Phase 4) |
| 6. Full Experiments | Autonomous GPU monitoring & execution across local/remote machines |
| 7. Result Analysis | **6-agent result debate**, narrative agreement |
| 8. Paper Writing + Figures | Publication-quality seaborn figures + venue-compliant LaTeX compilation |
| 9. Internal Review & Polish | Self-review, cross-review (Codex agents), rebuttal preparation |

#### Key Features

- **Multi-Agent Idea Debate** — 6 specialized agents (Innovator, Pragmatist, Theorist, Contrarian, Interdisciplinary, Empiricist) debate research ideas; an Area Chair synthesizes scores and makes accept/reject decisions (threshold: 6/10).
- **Multi-Agent Result Analysis** — 6 analysis agents (Optimist, Skeptic, Strategist, Methodologist, Comparativist, Revisionist) evaluate experimental results from different perspectives, driving consensus on narrative and next steps.
- **Venue-Aware** — Built-in guide for CCF-A venue requirements, style files, review criteria, and best practices. Always fetches the latest guidelines via web search.
- **Autonomous Experiment Execution** — GPU monitoring, remote SSH execution, and automatic logging.
- **Quality Gates** — AC gate after idea phase; decision gates after result analysis. Automatic fallback to alternative directions if ideas are rejected.
- **Git Integration** — Commits after each phase to track research progress.

```bash
claude skill install Linwei94/skills/ai-research-paper
```

---

### [`paper-reviewer`](./paper-reviewer/)

**AI-powered peer review assistant for top-tier AI/ML conferences and journals.**

Helps researchers and reviewers efficiently complete high-quality reviews for venues like CVPR, ICML, ICLR, NeurIPS, AAAI, ECCV, and ACL.

#### 5-Step Workflow

| Step | Description |
|------|-------------|
| 1. Paper Reception | Accept PDFs, OpenReview links, or URLs; confirm review scope with user |
| 2. Paper Interpretation | Deep analysis of motivation, method, novelty, experiments (discussion in Chinese) |
| 3. Interactive Q&A | Flexible discussion about specific sections, formulas, and tables |
| 4. Formal Review Generation | Venue-compliant English review (500–1500 words) with concrete evidence |
| 5. Browser Form Filling | Auto-fill OpenReview fields (optional; user handles submission) |

#### Key Features

- **PDF Reading** — Directly reads and analyzes paper PDFs.
- **Novelty Verification** — Uses web search to verify novelty claims against recent literature and prior work.
- **Venue-Specific Formatting** — Follows official reviewer guidelines for each conference/journal.
- **Score-Aligned Tone** — Review depth and tone automatically match the given score.
- **Bilingual Workflow** — Discussion in Chinese, final review output in English.
- **Auxiliary, Not Replacement** — Helps organize the review; final judgment always comes from the human reviewer.

```bash
claude skill install Linwei94/skills/paper-reviewer
```

---

### [`publish-pip`](./publish-pip/)

**Automated PyPI publishing workflow via GitHub Release.**

Handles the full version bump → commit → push → GitHub Release pipeline for Python packages. Automatically discovers version locations across different project structures (`pyproject.toml`, `setup.py`, `__init__.py`, etc.) and keeps them in sync.

#### Workflow

| Step | Description |
|------|-------------|
| 1. Discover | Find all version definitions in the project |
| 2. Propose | Suggest version bump (patch/minor/major) based on git history |
| 3. Update | Sync version across all locations |
| 4. Publish | Commit, push, and create GitHub Release with auto-generated notes |

#### Key Features

- **Auto-discovery** — Finds version strings across different project structures.
- **Smart suggestions** — Analyzes git log to recommend patch/minor/major bump.
- **Release notes** — Generates changelog from commits since last tag.
- **GitHub Action integration** — Creates Release that triggers PyPI publish workflow.

```bash
claude skill install Linwei94/skills/publish-pip
```

---

## Overview

| Skill | Purpose | Agents | Target Users |
|-------|---------|--------|--------------|
| **ai-research-paper** | End-to-end research pipeline (9 phases) | 12 debate agents + Area Chair | Researchers submitting to top-tier venues |
| **paper-reviewer** | Peer review assistance | User-driven | Conference / journal reviewers |
| **publish-pip** | PyPI publishing via GitHub Release | — | Python package maintainers |

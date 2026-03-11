---
name: ai-research-paper
description: Full pipeline for AI/ML research papers targeting CCF-A venues (ICML, ICLR, NeurIPS, CVPR, etc.). Use this skill whenever the user wants to explore a research idea, do literature review, write a research proposal, plan experiments, design figures, or write a conference paper. Also trigger when the user mentions keywords like "research idea", "paper writing", "literature review", "experiment plan", "NeurIPS/ICML/ICLR/CVPR submission", or any AI research workflow. Even if the user only wants help with one phase (e.g., just the literature review or just writing the paper), use this skill to ensure venue-appropriate quality and structure.
---

# AI Research Paper — Full Pipeline Skill

This skill guides the full lifecycle of an AI/ML research paper targeting CCF-A conferences (NeurIPS, ICML, ICLR, CVPR, ECCV, ACL, AAAI, etc.). The pipeline follows how real researchers work — iterative, with feedback loops and quality gates:

**Phase 1: Idea Exploration** — keyword → literature review → autonomous idea generation → idea debate (6 reviewers + AC gate)
**Phase 2: Research Proposal** — detailed method with theory, positioning, expected contributions
**Phase 3: Pilot Experiment Design** — minimal experiment plan (1 dataset, 1-2 baselines) sufficient to validate the core idea
**Phase 4: Pilot Experiments** — proof-of-concept, baseline reproduction, sanity checks
**Phase 5: Method Iteration** — revise method based on pilot results, may loop back to Phase 4
**Phase 5.5: Full Experiment Planning** — comprehensive plan (all datasets / baselines / ablations / analyses) after pilot passes, before full runs
**Phase 6: Full Experiments** — autonomous execution of all main experiments, ablations, analyses
**Phase 7: Result Analysis** — result debate (6 analysts), auto-decides on additional experiments
**Phase 8: Paper Writing + Figures** — publication-quality figures + parallel section writing + compile LaTeX
**Phase 9: Internal Review & Polish** — self-review, Codex cross-review, rebuttal preparation

## Phase 0: Interactive Setup (the ONLY phase requiring user input)

Before the pipeline runs autonomously, collect essential information from the user upfront. This is the **only** point where user interaction is needed — everything after this runs end-to-end.

### Step 0.1: Ask User Questions

Use `AskUserQuestion` to ask the following questions (can be asked in a single multi-question call):

**Question 1 — Target Venue:**
- Header: "Venue"
- Question: "Which venue are you targeting for this paper?"
- Options:
  - "NeurIPS" — Top ML venue, broad scope
  - "ICML" — Top ML venue, more theoretical
  - "ICLR" — Top ML venue, open review
  - "CVPR" — Top computer vision venue
- (User can also type a custom venue via "Other")

**Question 2 — Research Topic:**
- Header: "Topic"
- Question: "What is the research keyword or topic you want to explore?"
- Options:
  - "LLM efficiency" — Pruning, quantization, distillation
  - "Robustness" — Distribution shift, adversarial, calibration
  - "Multimodal" — Vision-language, cross-modal learning
  - "RL / Agents" — Reinforcement learning, autonomous agents
- (User can also type a custom topic via "Other")

### Step 0.2: Discover Compute Resources

After collecting user input, automatically discover available machines:

1. **Read SSH config** to find all configured remote hosts:
   ```bash
   cat ~/.ssh/config
   ```
2. **Test connectivity** for each host (with a short timeout to avoid hanging):
   ```bash
   ssh -o ConnectTimeout=5 -o BatchMode=yes <host> "echo ok" 2>/dev/null
   ```
3. **Check GPU availability** on each reachable host + local machine:
   ```bash
   # Local
   nvidia-smi --query-gpu=index,name,memory.total --format=csv,noheader 2>/dev/null
   # Remote
   ssh <host> "nvidia-smi --query-gpu=index,name,memory.total --format=csv,noheader" 2>/dev/null
   ```

### Step 0.3: User Selects Machines

Present a **multi-select** question with all reachable machines and their GPU info:

**Question — Compute Resources:**
- Header: "Machines"
- Question: "Which machines should be used for running experiments?"
- multiSelect: true
- Options: dynamically generated from Step 0.2 results, e.g.:
  - "local" — 2x RTX 4090 (24GB each)
  - "lab-server-1" — 4x A100 (80GB each)
  - "lab-server-2" — 8x RTX 3090 (24GB each)
  - (only include machines that are reachable and have GPUs)

### Step 0.4: Save Configuration

Save the collected configuration to `plan/config.md`:

```markdown
# Project Configuration

## Target Venue
[venue name]

## Research Topic
[keyword / topic]

## Available Compute Resources
| Machine | GPUs | Memory per GPU | Status |
|---------|------|----------------|--------|
| local | 2x RTX 4090 | 24GB | selected |
| lab-server-1 | 4x A100 | 80GB | selected |
| lab-server-2 | 8x RTX 3090 | 24GB | not selected |

## Selected Machines
- [machine 1]
- [machine 2]
```

After saving, the pipeline proceeds fully autonomously from Phase 1 onwards. **No more user interaction needed.**

### Step 0.5: Create Constraints and Venue Requirements Files

Immediately after saving `plan/config.md`, create two files:

#### 5a. Writing Constraints (`plan/constraints.md`)

Fixed constraints that apply throughout the entire pipeline:

```markdown
# Paper Writing Constraints

## Language
All writing must be in **English**. No other language is permitted in the paper body, abstract, figures, or captions.

## General Writing Rules
- Use active voice where possible
- Every claim must be backed by a citation or experimental result
- "Significantly" requires a statistical test; "state-of-the-art" requires a comparison table
- Avoid vague language and padding sentences
```

#### 5b. Venue Submission Requirements (`references/venue_requirements.md`)

Fetch in real-time using `WebSearch` and `WebFetch`:
- Search: "[venue] [year] call for papers submission requirements"
- Extract: page limit, format (single/double column), anonymity policy, supplementary material rules, LaTeX template, submission system URL, key dates
- If official page is unavailable, note it and use the most recent known requirements

Save as `references/venue_requirements.md`:

```markdown
# Venue Submission Requirements

## Venue: [venue name] [year]

### Submission Requirements
- **Page limit:** [N pages] (main paper) + [M pages] (references) + [K pages] (supplementary)
- **Format:** [single-column / double-column]
- **Anonymity:** [double-blind / single-blind]
- **Template:** [LaTeX template name / download URL]
- **Submission system:** [OpenReview / CMT / HotCRP / etc.]

### Key Dates
- Abstract deadline: [date or N/A]
- Paper submission deadline: [date]
- Notification: [date]
- Camera-ready: [date]

### Supplementary Material Rules
[Describe what is allowed: code, proofs, extra experiments, page limits]

### Other Requirements
[Any other venue-specific rules: ethics statement, reproducibility checklist, paper checklist, etc.]

### Source
[URL of the official call-for-papers page fetched]
```

Add both files to the Phase 0 git commit.

---

## Project Directory Structure

Every project follows this exact layout. Create it at the start:

```
<project-root>/
├── README.md              # Project overview, current status
├── .gitignore             # Standard Python/LaTeX ignores
├── plan/                  # All planning artifacts
│   ├── config.md          # Venue, topic, compute resources (from Phase 0)
│   ├── constraints.md     # Writing constraints: language, formatting rules (from Phase 0)
│   ├── TODO.md            # Master todolist — tracks all phases
│   ├── literature_review.md
│   ├── idea_summary.md
│   ├── idea_history.md    # Archive of ALL attempted ideas and their outcomes
│   ├── proposal.md
│   ├── pilot_experiment_plan.md  # Minimal plan for pilot (Phase 3)
│   └── experiment_plan.md        # Full plan after pilot passes (Phase 5.5)
├── references/            # External reference materials
│   └── venue_requirements.md  # Venue submission rules fetched in real-time (Phase 0)
├── experiments/           # All code and results
│   ├── models/
│   ├── methods/
│   ├── scripts/
│   ├── utils/
│   ├── configs/
│   ├── results/
│   └── archived/          # Failed idea rounds (archived/round_1/, round_2/, ...)
└── paper/                 # LaTeX source and figures
    ├── main.tex
    ├── figures/
    └── *.sty              # venue style file
```

The root directory contains ONLY `README.md` and `.gitignore`. All other files go into `plan/`, `experiments/`, or `paper/`.

## Project Initialization

At the very start of a new project, before any research work:

1. **Create the project directory** and the full directory structure above.
2. **Initialize git and create a private GitHub repo**:
   ```bash
   cd <project-root>
   git init
   # Create .gitignore with Python/LaTeX/dataset ignores
   # Create README.md with project title and target venue
   git add README.md .gitignore
   git commit -m "init: project scaffold for [paper title]"
   gh repo create <repo-name> --private --source=. --push
   ```
   Use `--private` by default (unpublished research should not be public). The repo name should be a short kebab-case identifier (e.g., `ttac-calibration`, `efficient-llm-pruning`).
3. **Create `plan/TODO.md`** (see below) and commit it.

## Telegram Notifications (use notify-telegram)

The pipeline runs autonomously end-to-end. To keep the user informed, invoke `notify-telegram` (see `notify-telegram/SKILL.md`) at every significant event:

**Mandatory notification points:**
1. **Phase completion** — After each phase's git commit & push, notify with phase name and key outcome (e.g., "Phase 1 complete: idea 'Adaptive TTT via Meta-Calibration' accepted by AC with 7/10").
2. **Pipeline stopped / blocked** — If the pipeline cannot continue autonomously (e.g., Phase 5 iteration limit exhausted and all directions failed, Phase 6 error requiring fundamental method rethink), notify immediately with the reason and what action is needed.
3. **Pipeline finished** — When Phase 9 is complete and the paper is ready for submission, notify with a summary of the paper title, venue, and key results.

**Notification format:**
```
[AI Research Pipeline] Phase X: <phase name>
Status: <completed | blocked | finished>
Summary: <1-2 sentence summary of outcome>
Action needed: <none | description of what the user needs to do>
```

## Git Workflow: Commit After Every Phase

After completing each phase, commit and push the new artifacts to GitHub. This ensures progress is never lost and collaborators can track the project.

**Linear path (idea passes pilot):**

| After Phase | Commit message pattern | Files to commit |
|-------------|----------------------|-----------------|
| Phase 0 | `init: project setup — [venue] / [topic]` | `plan/config.md`, `plan/constraints.md`, `references/venue_requirements.md`, `plan/TODO.md`, `README.md`, `.gitignore` |
| Phase 1 | `plan: idea exploration round [N] — [idea title]` | `plan/literature_review.md`, `plan/idea_summary.md`, `plan/idea_debate.md`, `plan/idea_history.md`, `plan/TODO.md` |
| Phase 2 | `plan: research proposal — [idea title]` | `plan/proposal.md`, `plan/TODO.md` |
| Phase 3 | `plan: pilot experiment design — [idea title]` | `plan/pilot_experiment_plan.md`, `plan/TODO.md` |
| Phase 4 | `experiments: pilot passed — [method] on [dataset]` | `experiments/methods/`, `experiments/results/baseline_reproduction.md`, `plan/TODO.md` |
| Phase 5 | `experiments: method iteration [N] (idea round [M])` | `experiments/methods/`, `experiments/results/method_iterations.md`, `plan/TODO.md` |
| Phase 5.5 | `plan: full experiment plan — [idea title]` | `plan/experiment_plan.md`, `plan/TODO.md` |
| Phase 6 | `experiments: [experiment name] complete` | `experiments/`, `plan/TODO.md` (commit incrementally) |
| Phase 7 | `plan: result analysis and narrative` | `plan/result_debate.md`, `plan/TODO.md` |
| Phase 8 | `paper: draft with figures` | `paper/main.tex`, `paper/figures/`, `paper/*.sty`, `plan/TODO.md` |
| Phase 9 | `paper: review polish and rebuttal prep` | `paper/main.tex`, `plan/rebuttal_prep.md`, `plan/TODO.md` |

**Iteration path (idea fails pilot → loop back):**

| Trigger | Commit message pattern | Files to commit |
|---------|----------------------|-----------------|
| Phase 4 pilot fails | `experiments: pilot failed — idea round [N] archived` | `experiments/results/baseline_reproduction.md`, `plan/idea_history.md`, `plan/TODO.md` |
| Phase 5 all iterations exhausted | `experiments: idea round [N] exhausted — archived, rolling back` | `experiments/results/method_iterations.md`, `experiments/archived/round_[N]/`, `plan/idea_history.md`, `plan/TODO.md` |
| ↳ Loop back to Phase 1 | `plan: idea exploration round [N+1] — [new idea title]` | `plan/idea_summary.md`, `plan/idea_debate.md`, `plan/idea_history.md`, `plan/TODO.md` |
| ↳ Continue Phase 2–4 | *(same as linear path above, with updated round number)* | |

Rules:
- Always `git add` specific files, never `git add -A` (avoid committing large datasets, checkpoints, or secrets).
- Add to `.gitignore`: `*.pth`, `*.pt`, `__pycache__/`, `*.pyc`, `~/dataset/`, `experiments/results/*.npy`, large binary files.
- Push after each commit: `git push origin main`.
- For Phase 4, commit incrementally — don't wait until all experiments are done. Commit after each major experiment completes so results are backed up.

## Master Todolist (plan/TODO.md)

Create `plan/TODO.md` at project initialization and keep it updated throughout the entire pipeline. This is the single source of truth for project progress. Use checkboxes for tracking:

```markdown
# TODO: [Paper Title]

**Target Venue:** [Venue + Year]
**Research Topic:** [keyword / topic]
**Deadline:** [submission deadline]
**Created:** [date]
**Last Updated:** [date]

## Phase 0: Interactive Setup
- [x] User selected target venue
- [x] User provided research topic
- [x] Compute resources discovered
- [x] User selected machines for experiments
- [x] Configuration saved (plan/config.md)
- [x] → git commit & push

## Phase 1: Idea Exploration
- [ ] Literature review (plan/literature_review.md) — [skip if Round 2+]
- [ ] Check idea history for archived ideas (plan/idea_history.md) — [Round 2+ only]
- [ ] Idea generation — 3-5 directions scored, best auto-selected (Round [N])
- [ ] Idea debate — 6 reviewers completed (plan/idea_debate.md)
- [ ] AC decision: [ACCEPT/REVISE/REJECT] — auto-handled
- [ ] Idea refinement (plan/idea_summary.md)
- [ ] Idea history updated (plan/idea_history.md)
- [ ] → git commit & push
- [ ] → notify-telegram: Phase 1 complete (Round [N])

## Phase 2: Research Proposal
- [ ] Draft proposal (plan/proposal.md)
- [ ] → git commit & push
- [ ] → notify-telegram: Phase 2 complete

## Phase 3: Pilot Experiment Design
- [ ] Pilot experiment plan drafted (plan/pilot_experiment_plan.md)
- [ ] → git commit & push
- [ ] → notify-telegram: Phase 3 complete

## Phase 4: Pilot Experiments
- [ ] Core method implemented (experiments/methods/)
- [ ] Baseline 1 reproduced: [name] — matches reported numbers?
- [ ] Baseline 2 reproduced: [name] — matches reported numbers?
- [ ] Pilot experiment on simplest setting: [result summary]
- [ ] Method works as hypothesized? [yes/no — if no, go to Phase 5]
- [ ] → git commit & push
- [ ] → notify-telegram: Phase 4 complete (or entering Phase 5)

## Phase 5: Method Iteration (if needed)
- [ ] Issue identified from pilot: [description]
- [ ] Method revised: [what changed]
- [ ] Re-run pilot to verify fix
- [ ] → loop back to Phase 4 if still failing (max 3-5 method iterations)
- [ ] If all iterations exhausted: archive idea to plan/idea_history.md
- [ ] If archived: clean up experiments to experiments/archived/round_[N]/
- [ ] → notify-telegram: idea failed, rolling back to Phase 1 for new idea
- [ ] → loop back to Phase 1 Step 1.2 for new idea (Round [N+1])

## Phase 5.5: Full Experiment Planning (after pilot passes)
- [ ] Full experiment plan drafted (plan/experiment_plan.md)
- [ ] All datasets listed (4-8 datasets)
- [ ] All baselines listed (5-10 baselines)
- [ ] Ablation and analysis experiments defined
- [ ] Resource discovery (local + remote GPUs)
- [ ] → git commit & push
- [ ] → notify-telegram: Phase 5.5 complete

## Phase 6: Full Experiments
- [ ] All datasets prepared (~/dataset/)
- [ ] Main experiment 1: [name] — [status]
- [ ] Main experiment 2: [name] — [status]
- [ ] ...
- [ ] Ablation 1: [name] — [status]
- [ ] ...
- [ ] Analysis 1: [name] — [status]
- [ ] ...
- [ ] All results collected to experiments/results/
- [ ] → git commit & push (incrementally)
- [ ] → notify-telegram: Phase 6 complete — all experiments finished

## Phase 7: Result Analysis
- [ ] Result debate — 6 analysts completed (plan/result_debate.md)
- [ ] Narrative auto-finalized based on agent consensus
- [ ] Additional experiments auto-identified: [list or "none"]
- [ ] Additional experiments completed (if any)
- [ ] → git commit & push
- [ ] → notify-telegram: Phase 7 complete

## Phase 8: Paper Writing + Figures
- [ ] Venue requirements fetched (references/venue_requirements.md)
- [ ] Style file downloaded to paper/
- [ ] Figures generated (paper/figures/)
- [ ] Related Work section drafted
- [ ] Method section drafted
- [ ] Experiment section drafted
- [ ] Introduction drafted
- [ ] Abstract drafted (ONE paragraph, no breaks)
- [ ] Conclusion drafted
- [ ] Appendix (proofs, extra experiments, implementation details)
- [ ] Paper compiles without errors
- [ ] → git commit & push
- [ ] → notify-telegram: Phase 8 complete — paper draft ready

## Phase 9: Internal Review & Polish
- [ ] Self-review checklist passed
- [ ] Codex independent review completed
- [ ] Reviewer concerns addressed
- [ ] Rebuttal notes prepared (plan/rebuttal_prep.md)
- [ ] Notation consistency check
- [ ] Page limit check
- [ ] Reference completeness check
- [ ] → git commit & push
- [ ] → notify-telegram: Pipeline FINISHED — paper ready for submission

## Issues & Decisions Log
- [date] [issue description] — [resolution or pending]
```

Update this file at every significant step. When an experiment completes, mark it done. When an issue arises, log it. This file should always reflect the current project state so that any conversation (even a new one) can pick up where things left off.

## Idea History (plan/idea_history.md)

Create `plan/idea_history.md` at the start of Phase 1 and maintain it throughout the project. This file archives **every idea that has been attempted**, including those that passed the idea debate but failed at pilot experiments. Its primary purpose is to prevent the pipeline from revisiting failed directions.

```markdown
# Idea History: [Research Topic]

**Created:** [date]
**Last Updated:** [date]

## Active Idea
- **Round**: [current round number]
- **Title**: [current idea title]
- **Status**: [exploring / debating / piloting / accepted / failed]

## Archived Ideas (DO NOT REUSE)

### Round 1: [idea title]
- **Summary**: [1-paragraph description of the idea]
- **Key Mechanism**: [core technical contribution]
- **Idea Debate Outcome**: [ACCEPT / REVISE & RESUBMIT / REJECT] — AC score: [X/10]
- **Pilot Experiment Result**: [passed / failed — brief description]
- **Failure Reason**: [why this idea was abandoned — e.g., "pilot showed no improvement over baseline on CIFAR-10-C", "method diverges after 100 steps", "AC rejected due to insufficient novelty"]
- **Lessons Learned**: [what we learned from this attempt — e.g., "entropy minimization alone is not sufficient", "need to handle class imbalance explicitly"]
- **Date Archived**: [date]

### Round 2: [idea title]
- ...

### Round N: [idea title]
- ...
```

**Rules for Idea History:**
1. **Every idea that enters Phase 1 Step 1.3 (Idea Debate) must be recorded**, regardless of outcome.
2. When an idea is archived (due to debate rejection OR pilot failure), record the failure reason and lessons learned.
3. When generating new ideas (Phase 1 Step 1.2), the pipeline **MUST read `plan/idea_history.md` first** and explicitly avoid:
   - Ideas that are semantically similar to any archived idea
   - Ideas that rely on the same core mechanism that previously failed
   - Ideas that ignore the lessons learned from previous rounds
4. The idea generation prompt must include the full list of archived ideas and their failure reasons as negative constraints.
5. There is no limit on the number of rounds — the pipeline keeps generating new ideas until one passes both the idea debate AND pilot experiments.

## Agent Model Hierarchy

Different tasks require different levels of capability. Use the appropriate model tier for each task to balance quality and efficiency:

| Tier | Model | When to Use |
|------|-------|-------------|
| **Heavy** | Opus 4.6 | Main conversation orchestration, overall decision-making, supervision/review, editing/integration, critical reflection, final paper editing |
| **Standard** | Opus 4.6 | Literature review, experiment planning, experiment execution, writing drafts (subagents for heavy work) |
| **Light** | Sonnet 4.6 | Result debate agents, cross-review, section-level critique, routine analysis (use `--model sonnet` flag or equivalent for subagents) |
| **Codex** | GPT-5.4 High (via MCP) | Independent third-party review, alternative writing perspective, cross-model verification |

### How to Apply

- **Main conversation** always runs at Heavy tier (this is the orchestrator).
- **Subagents** default to Standard tier. For debate agents (Idea Debate, Result Debate) and cross-review tasks, use Light tier — these tasks benefit from volume and diversity of perspectives more than raw capability.
- **Codex (MCP)** is used in specific scenarios (see below). Invoke via the `mcp codex` tool when available.

### When to Use Codex (GPT-5.4 High)

Codex provides an **independent third-party perspective** from a different model family. Use it for:

1. **Independent paper review**: After the paper draft is complete (Phase 6), send the full paper to Codex for a simulated peer review. A different model catches different blind spots.
2. **Alternative writing**: If a section is particularly difficult to write well, generate a Codex version alongside the Claude version and pick the better one (or merge).
3. **Cross-model verification**: When the Contrarian or Skeptic raises a concern, validate it with Codex to see if a different model agrees.
4. **Novelty check**: Send the idea summary to Codex and ask "Has this been done before? What's the closest existing work?" — a different training distribution may surface references Claude missed.

**How to invoke Codex**: Use the MCP codex tool if available. Pass the relevant context (paper section, idea summary, results) and a clear prompt. Treat Codex output as advisory — the main agent (Opus) makes final decisions.

## Context Management

Each phase can generate substantial context. To prevent context explosion:

- Use **subagents** for heavy lifting within each phase (literature search, code generation, writing sections). The main conversation tracks high-level progress and decisions.
- At phase transitions, **summarize** the key outputs from the completed phase into a concise document (saved to disk), then proceed to the next phase referencing that document rather than carrying the full context.
- Save all intermediate artifacts to `plan/` directory.

---

## Phase 1: Idea Exploration

### Input
The user provides a **keyword or topic** (e.g., "test-time adaptation for calibration", "efficient LLM inference", "vision-language robustness").

### Step 1.1: Literature Review

Spawn a subagent to conduct a structured literature review:

1. **Search recent publications** (2023-2026) at the target venue and related top venues. Use web search to find papers on:
   - The exact keyword at the target venue (e.g., "test-time adaptation NeurIPS 2024/2025")
   - Related keywords at ICML, ICLR, CVPR, ECCV
   - Recent arXiv preprints (last 12 months)

2. **Organize findings** into a table with columns: Title, Venue/Year, Key Contribution, Method Category, Limitations.

3. **Identify research gaps**: What problems remain unsolved? Where do existing methods fail? What assumptions are too strong?

4. **Categorize the landscape**: Group papers by approach (e.g., "entropy-based methods", "prototype-based methods") and identify which category has the most room for improvement.

Save output to `plan/literature_review.md`.

### Step 1.2: Autonomous Idea Generation

Based on the literature review, autonomously generate and select a research direction. Do NOT ask the user for input — the pipeline should proceed end-to-end without human intervention.

1. **Check idea history**: If `plan/idea_history.md` exists, read it first. Extract:
   - All archived idea summaries and their core mechanisms
   - All failure reasons and lessons learned
   - These become **hard negative constraints** — the new ideas MUST differ from all archived ideas in their core mechanism and MUST incorporate the lessons learned.

2. **Explore context**: Review the literature review, understand the research landscape, identify the most promising gaps.

3. **Generate 3-5 research directions**, scoring each on:
   - **Novelty** (1-5): How new is this compared to existing work?
   - **Feasibility** (1-5): Can this be implemented and validated with available resources?
   - **Impact** (1-5): Would this excite reviewers at the target venue?
   - **Risk** (1-5, lower is better): How likely is it to fail fundamentally?
   - **Differentiation from history** (pass/fail): Is this idea sufficiently different from ALL archived ideas? If it relies on the same core mechanism as any failed idea, it FAILS and is discarded.
   - **Composite score**: (Novelty + Feasibility + Impact) - Risk (only for ideas that pass the differentiation check)

4. **Auto-select the direction with the highest composite score.** Log all directions and scores in `plan/idea_summary.md` for traceability. If this is not the first round, also log why each new direction is different from the archived ideas.

5. If the user provided any hints or preferences in their initial prompt, factor those into the scoring (e.g., boost feasibility for directions aligned with stated constraints).

6. **Update `plan/idea_history.md`**: Record the selected idea as the "Active Idea" with the current round number.

### Step 1.3: Idea Debate (6-agent multi-perspective review)

Once a direction is chosen, run the **Idea Debate** process to stress-test and refine the idea before committing. Read `agents/idea_debate.md` for full agent definitions and prompt templates.

**The 6 Debate Agents:**

| Agent | Perspective | Role |
|-------|------------|------|
| Innovator | Cross-domain innovation | Bold methodological transfer, creative combinations |
| Pragmatist | Engineering feasibility | Implementation challenges, compute budget, simplifications |
| Theorist | Mathematical foundation | Provable claims, proof strategies, theoretical frameworks |
| Contrarian | Challenge assumptions | Find flaws, predict reviewer objections, missing baselines |
| Interdisciplinary | Cross-domain analogies | Insights from cognitive science, physics, biology, control theory |
| Empiricist | Experiment-first | Killer experiments, ablation strategy, statistical rigor |

**Debate Process:**

1. **Round 1** — Spawn all 6 reviewer agents in parallel. Each independently analyzes the idea.
2. **Synthesis** — Collect all outputs, synthesize into a SWOT summary. Log it to `plan/idea_debate.md`.
3. **Auto-incorporate** — Automatically incorporate suggestions that have consensus support (raised by 2+ agents) and address all concerns flagged as critical by any agent.
4. **Round 2** — Re-run Contrarian and Empiricist on the revised idea to verify.
5. **AC Decision** — Spawn the **Area Chair (AC)** agent with all 6 reviewer reports. The AC scores the idea (Novelty, Significance, Feasibility, Theoretical Soundness, Expected Empirical Strength, Overall /10) and makes a decision:
   - **ACCEPT (6+/10)** → Proceed to Phase 2. Log conditions.
   - **REVISE & RESUBMIT** → Automatically revise the idea based on AC feedback, re-run full debate. Max 3 cycles.
   - **REJECT** → Automatically select the next-highest-scored direction from Step 1.2 and re-run the debate. If all directions exhausted, generate new directions based on the rejection feedback.
6. **Save** the full debate record + AC meta-review to `plan/idea_debate.md`.

The pipeline handles AC decisions autonomously. ACCEPT proceeds immediately, REVISE & RESUBMIT triggers automatic revision, and REJECT triggers fallback to alternative directions — no user intervention needed.

### Step 1.4: Idea Refinement

After the debate, crystallize the final idea:
- Identify the **3 closest related papers** and articulate precise differences
- Draft a **one-paragraph elevator pitch** (this becomes the seed for the abstract)
- List **expected contributions** (3-5 bullet points)
- Identify **potential weaknesses reviewers might raise** and sketch rebuttals (informed by the Contrarian's analysis)

Save to `plan/idea_summary.md`.

---

## Phase 2: Research Proposal

Write a comprehensive proposal following this structure. The proposal should be detailed enough that a PhD student could implement the method from it alone. Reference the target venue's style (similar to a NeurIPS-level proposal with rich theoretical detail). Proceed directly to Phase 3 after saving the proposal — no user approval needed.

### Proposal Structure

```markdown
# [Title]

**Target Venue:** [Conference Name + Year]
**Status:** Draft Proposal

---

## 1. Motivation & Problem Statement
- Real-world problem and why it matters
- Formal problem definition with mathematical notation
- What existing methods assume and why those assumptions fail
- **Key gap** statement (one clear sentence)

## 2. Related Work
- Organize by category (not chronologically)
- For each category: representative papers with venue/year, key idea, limitation
- End with a **positioning paragraph**: how this work sits at the intersection

## 3. Proposed Method
### 3.1 Framework Overview
- High-level architecture diagram (text/ASCII)
- Design principles and why each component exists

### 3.2-3.N Method Details
- Full mathematical formulation with equations
- Algorithm pseudocode
- Theoretical analysis: convergence guarantees, bounds, complexity
- Implementation details that affect reproducibility

## 4. Experimental Plan
(Brief version here; full plan in Phase 3)
- Datasets, backbones, baselines, metrics
- Expected results and hypotheses

## 5. Novelty & Contributions
- Numbered list of contributions
- Each should be verifiable from the experiments

## 6. Venue Positioning & Pitch
- Why this venue specifically
- Anticipated reviewer concerns and rebuttals

## 7. References
```

The **theoretical section (§3)** is especially important. Include:
- Formal definitions and notation
- Propositions/theorems with proof sketches
- Complexity analysis (time, space, communication)
- Connections to established theoretical frameworks (PAC learning, online learning, information theory, etc.)

Save to `plan/proposal.md`.

---

## Phase 3: Pilot Experiment Design

Design a **minimal** experiment plan sufficient to validate the core idea. Keep it lean — the goal is to answer one question: does the core mechanism work at all?

Save to `plan/pilot_experiment_plan.md`:

```markdown
# Pilot Experiment Plan: [Idea Title]

## Core Hypothesis
[One sentence: what specific behavior of the method should we observe to call the pilot a success?]

## Dataset
- 1 dataset (pick the smallest/fastest that is relevant to the idea)
- Justify choice: [why this dataset best stress-tests the core hypothesis]

## Baselines
- 1-2 baselines maximum: the trivial baseline + the strongest directly competing method
- Implementation source: [where to get reference implementation]

## Metrics
- 1-2 primary metrics only

## Pilot Experiment
- Exact setup: architecture, splits, hyperparameters, seeds
- Compute estimate: [GPU hours for this pilot]
- Success criterion: [e.g., "beats trivial baseline by ≥1% on metric X"]
```

Proceed directly to Phase 4 after saving the pilot plan.

---

## Phase 4: Pilot Experiments

Before committing to full-scale experiments, run **pilot experiments** to validate the core hypothesis. This is how real researchers work — nobody runs 100 GPU-hours of experiments before confirming the basic idea works.

### 4.1: Implement Core Method

Implement only the minimum viable version of the proposed method. No bells and whistles yet.

### 4.2: Reproduce Key Baselines

Before comparing anything, **reproduce the reported numbers of 2-3 key baselines** on at least one dataset. This is critical:
- If you can't reproduce a baseline, your comparison is unreliable.
- Log reproduced vs. reported numbers in `experiments/results/baseline_reproduction.md`.
- Acceptable variance: within 1-2% of reported numbers. If further off, investigate (different hyperparameters? different data split? different evaluation protocol?).

### 4.3: Pilot Experiment

Run the proposed method on the **simplest, fastest setting** (e.g., smallest dataset, one corruption type, one seed):
- Does it beat the simplest baseline at all?
- Does the core mechanism work as hypothesized? (check intermediate outputs, not just final metrics)
- Is the compute budget reasonable? Extrapolate from pilot to full experiments.

**Decision point**: If the pilot fails, go to Phase 5 (Method Iteration). If it succeeds, proceed to Phase 6 (Full Experiments).

---

## Phase 5: Method Iteration

If pilot experiments reveal problems, iterate on the method design. This loop is normal — real research rarely works on the first try.

### 5.1: Diagnose the Problem

Analyze pilot results to understand WHY the method isn't working:
- Is the loss function driving parameters in the wrong direction? (check gradient signs)
- Is the optimization unstable? (plot loss curves, parameter trajectories)
- Is the hypothesis itself wrong? (the method may not help for the reasons we thought)

### 5.2: Revise the Method

Based on diagnosis, make targeted changes. Common fixes:
- Loss function modification (different weighting, different components)
- Regularization (prevent collapse, prevent drift)
- Architecture changes (more/fewer parameters to adapt)
- Hyperparameter adjustment (learning rate, thresholds)

### 5.3: Re-run Pilot

After each revision, re-run the pilot experiment. Compare against the previous version.

### 5.4: Iteration Loop

Repeat 5.1-5.3 until the pilot shows the method works. **Maximum 3-5 method-level iterations** (tweaking hyperparameters, loss functions, etc.).

### 5.5: Idea-Level Rollback (if method iteration fails)

If after 3-5 method iterations the pilot still fails, this means the **core idea itself** is flawed — not just the implementation. In this case:

1. **Archive the failed idea**: Update `plan/idea_history.md` with:
   - Full idea summary and core mechanism
   - Idea debate outcome (AC score)
   - Pilot experiment results (what was tried, what metrics were observed)
   - Failure reason (specific and actionable — e.g., "entropy minimization causes class collapse under label shift", not just "didn't work")
   - Lessons learned (what future ideas should avoid or incorporate)
   - Date archived

2. **Clean up experiment artifacts**: Move the current idea's experiment code to `experiments/archived/round_N/` to keep the workspace clean while preserving the work.

3. **Notify user**: **Invoke `notify-telegram`** to inform the user that the current idea (Round N) failed pilot validation and the pipeline is rolling back to explore a new idea direction.

4. **Loop back to Phase 1 Step 1.2** (Idea Generation): Generate entirely new ideas with the updated idea history as a negative constraint. The pipeline does NOT re-run the literature review (Phase 1 Step 1.1) — the existing literature review is reused. It goes directly to idea generation → idea debate → proposal → pilot.

5. **No limit on idea rounds**: The pipeline keeps generating new ideas until one passes both the idea debate AND pilot experiments. However, after every 3 failed rounds, **invoke `notify-telegram`** with a summary of all failed ideas and ask the user whether to continue exploring or pivot to a different research topic entirely.

Save iteration history in `experiments/results/method_iterations.md`:
```markdown
## Round [N] — Idea: [idea title]

### Iteration 1: [date]
- Change: [what was changed]
- Result: [pilot metric before → after]
- Analysis: [why it did/didn't help]

### Iteration 2: [date]
...

### Outcome: [PASSED → proceed to Phase 5.5 / FAILED → archived, rolling back to Phase 1]
- Failure reason: [if failed]
```

---

## Phase 5.5: Full Experiment Planning

**Triggered only after the pilot passes.** Now that the core idea is validated, design a comprehensive experiment plan that meets top-venue standards. Reviewers at CCF-A venues expect extensive empirical validation.

Save to `plan/experiment_plan.md`:

```markdown
# Full Experiment Plan: [Paper Title]

## 1. Datasets & Benchmarks
- Table of ALL datasets with: name, task, #classes, #samples, shift type, source
- Include standard benchmarks (CIFAR-C, ImageNet-C, WILDS, etc.)
- Include domain-specific datasets relevant to the application
- Aim for 4-8 datasets minimum

## 2. Models / Backbones
- Table: architecture, #params, pretrained source, why included
- Mix of CNNs and Transformers if applicable
- At least 3 architectures to show generality

## 3. Baselines
- Organize by category (e.g., "static methods", "online methods", "oracle upper bounds")
- Include: method name, venue/year, key mechanism, implementation source
- Always include: uncalibrated baseline, oracle upper bound, and the 2-3 strongest recent methods
- Aim for 5-10 baselines

## 4. Metrics
- Primary metrics (main tables): 2-3 metrics
- Secondary metrics (appendix): additional diagnostics
- Statistical significance: seeds, confidence intervals, significance tests

## 5. Main Experiments
For each experiment:
- **Setup**: exact configuration, data splits, hyperparameters
- **Hypothesis**: what you expect and why
- **Reporting**: which tables/figures will present the results
- **Compute estimate**: GPU hours rough estimate

### Experiment 1: [name]
### Experiment 2: [name]
...
Target: 4-6 main experiments

## 6. Ablation Studies
For each ablation:
- What component/hyperparameter is varied
- What is held fixed
- Expected finding
Target: 4-8 ablations

## 7. Analysis Experiments
- Failure mode analysis
- Visualization of learned representations/parameters
- Computational efficiency comparison
- Qualitative examples
Target: 3-5 analyses

## 8. Resource Requirements
- Total GPU hours estimate
- Storage requirements
- Available resources (from plan/config.md)
```

### Resource Discovery

Re-check GPU availability on the selected machines before finalizing the plan:
```bash
# Local
nvidia-smi --query-gpu=index,name,memory.total,memory.free --format=csv,noheader 2>/dev/null
# Remote (for each selected machine)
ssh <host> "nvidia-smi --query-gpu=index,name,memory.total,memory.free --format=csv,noheader" 2>/dev/null
```

If compute is limited, prioritize experiments by expected impact — mark lower-priority experiments as optional. Proceed directly to Phase 6 after saving the plan.

---

## Phase 6: Full Experiments

This phase is **autonomous**. Do NOT wait for user instructions between experiments. Generate all code, run all experiments, handle errors, and only come back to the user when everything is done (or when a blocking issue requires their decision).

### 6.0: Dataset Preparation

All datasets are stored under `~/dataset/` (home directory). Before running any experiment:

1. Check if the required dataset already exists locally:
   ```bash
   ls ~/dataset/  # check what's already downloaded
   ```
2. If a dataset is missing, download it to `~/dataset/<dataset_name>/`. Common methods:
   - torchvision built-in datasets: set `root="~/dataset/"` in the dataset constructor (it will auto-download)
   - Manual downloads: `wget`/`curl` to `~/dataset/` then extract
   - Hugging Face datasets: `datasets.load_dataset(..., cache_dir="~/dataset/")`
   - WILDS: `wilds.get_dataset(..., root_dir="~/dataset/")`
3. For remote machines, check `~/dataset/` on the remote host too. If the dataset is missing there, either:
   - `rsync` from local: `rsync -avz ~/dataset/<name> <host>:~/dataset/`
   - Or download directly on the remote machine
4. Never hardcode dataset paths — always use `~/dataset/` (expand with `os.path.expanduser`) as the base.

### 6.1: Code Generation

Generate experiment code based on the experiment plan. Use subagents for independent scripts. Organize under `experiments/`:
```
experiments/
├── models/          # model definitions
├── methods/         # method implementations
├── scripts/         # run scripts for each experiment
├── utils/           # shared utilities (metrics, data loading)
├── configs/         # hyperparameter configs
└── results/         # output CSVs and figures
```

### 6.2: Resource Discovery and GPU Monitoring

At the start of this phase, discover all available compute:

1. **Local GPUs**: `nvidia-smi --query-gpu=index,name,memory.total,memory.used,utilization.gpu --format=csv,noheader`
2. **Remote machines**: Read `~/.ssh/config` to find hosts, then for each:
   ```bash
   ssh <host> "nvidia-smi --query-gpu=index,name,memory.total,memory.used,utilization.gpu --format=csv,noheader" 2>/dev/null
   ```
3. Build a **resource table** mapping: host → GPU index → GPU model → VRAM → current usage

### 6.3: Autonomous Experiment Execution

Maintain an **experiment queue** (all experiments from `plan/experiment_plan.md`) and execute them autonomously:

1. **Monitor GPU availability**: Before launching each experiment, check which GPUs are free (memory usage < 20% and utilization < 10%). On remote machines, also check if other users' processes are running.

2. **Greedy scheduling**: As soon as a GPU becomes available, immediately launch the next queued experiment on it. Do not wait for the user to tell you. Use `CUDA_VISIBLE_DEVICES=<gpu_id>` to pin experiments to specific GPUs.

3. **Remote execution**: For remote machines:
   - First `rsync` the experiment code to the remote host
   - Launch via `ssh <host> "cd <project_dir> && nohup <run_command> > <log_file> 2>&1 &"`
   - Periodically check if the remote job has completed by checking the log file or process status

4. **Parallel execution**: Run experiments on ALL available GPUs simultaneously. If 4 GPUs are free across 2 machines, launch 4 experiments at once.

5. **Error handling**: If an experiment fails:
   - Read the error log
   - Attempt to fix the issue (missing package, OOM → reduce batch size, data path error, etc.)
   - Re-queue the fixed experiment
   - Only escalate to the user if the fix requires fundamentally rethinking the method (extremely rare). **Invoke `notify-telegram`** immediately when escalating, with the error details and what decision is needed.

6. **Progress tracking**: Maintain `experiments/results/experiment_status.json`:
   ```json
   {
     "experiments": [
       {"name": "exp1_corruption", "status": "completed", "host": "local", "gpu": 0, "started": "...", "finished": "..."},
       {"name": "exp2_labelshift", "status": "running", "host": "lab1", "gpu": 2, "started": "..."},
       {"name": "exp3_ablation", "status": "queued"}
     ]
   }
   ```

### 6.4: Results Collection

- Save all results as CSV files: `experiments/results/<experiment_name>.csv` with columns for method, dataset, metric, seed, value.
- For remote experiments, `rsync` results back to the local machine after completion.
- When ALL experiments are done, summarize results and report to the user with any issues encountered.

---

## Phase 7: Result Analysis (Result Debate)

After ALL experiments complete, run the **Result Debate** before proceeding to figures and writing. Read `agents/result_debate.md` for full agent definitions and prompt templates.

**The 6 Analysis Agents:**

| Agent | Perspective | Role |
|-------|------------|------|
| Optimist | Positive discovery | Strongest results, compelling narrative, extension directions |
| Skeptic | Statistical rigor | Significance tests, confounding factors, cherry-picking check |
| Strategist | Next steps | Triage results, prioritize missing experiments, paper structure |
| Methodologist | Validity assessment | Internal/external validity, reproducibility, construct validity |
| Comparativist | SOTA benchmarking | Position against state-of-the-art, missing baselines, win/loss patterns |
| Revisionist | Hypothesis revision | Check original hypotheses against evidence, revise narrative if needed |

**Debate Process:**

1. **Round 1** — Spawn all 6 agents in parallel with the full results data (CSVs, ablation tables).
2. **Synthesis** — Collect outputs, identify consensus findings, conflicts between agents, and prioritized action items.
3. **Auto-decision** — Automatically decide based on agent consensus:
   - **Additional experiments**: Run any experiments that 2+ agents flag as critical/missing. Skip those only one agent suggests unless the Skeptic flags statistical validity concerns.
   - **Framing**: Adopt the narrative recommended by the Strategist, refined by the Revisionist's hypothesis check.
   - **Limitations**: Acknowledge all limitations flagged by any agent honestly.
4. **Round 2 (if needed)** — If additional experiments are run, re-run Skeptic and Comparativist to verify.
5. **Save** to `plan/result_debate.md`.

This debate is mandatory. Do NOT proceed to figure design or paper writing until results have been analyzed and the narrative is finalized. The Strategist's paper structure recommendation directly informs Phase 8 (which results become main figures/tables, how to frame the story).

---

## Phase 8: Paper Writing + Figures

This phase combines figure generation and paper writing — they inform each other, so they belong together. The Result Debate (Phase 7) determines which results become main figures/tables vs. appendix, and what narrative the paper tells.

### 8.1: Venue Requirements & Constraints

Before writing, read both constraint files created in Phase 0:

- **`plan/constraints.md`** — writing rules that apply throughout: language (English), style conventions
- **`references/venue_requirements.md`** — venue submission rules: page limit, format, anonymity policy, template, submission system, key dates, supplementary rules

Fetch the latest venue requirements via web search. Run ALL of the following searches:

1. **Style file and template**: `"[venue] [year] latex template" OR "[venue] [year] style file"` — download the `.sty`/`.cls` file to `paper/`
2. **Submission guidelines**: `"[venue] [year] call for papers" OR "[venue] [year] submission guidelines"` — page limit, anonymization, supplementary policy, code policy
3. **Deadlines**: `"[venue] [year] deadline" OR "[venue] [year] important dates"`
4. **Review criteria**: `"[venue] [year] reviewer guidelines" OR "[venue] [year] review form"` — some venues (NeurIPS, ICML) have mandatory checklists

**Venue official websites (starting points):**

| Venue | Website |
|-------|---------|
| NeurIPS | https://neurips.cc |
| ICML | https://icml.cc |
| ICLR | https://iclr.cc |
| CVPR | https://cvpr.thecvf.com |
| ECCV | https://eccv.ecva.net |
| ACL | https://aclweb.org |
| AAAI | https://aaai.org |

After searching, update `references/venue_requirements.md` with the latest info.

**Do NOT use memorized page limits or deadlines** — these change every year. Always verify via live web search.

**Venue characteristics (stable):**

- **NeurIPS**: Values theory + empirical equally. Appreciates honest limitations. Dislikes pure empirical with marginal gains.
- **ICML**: More theory-oriented. Formal analysis and principled justification expected. Appendix proofs for all theorems.
- **ICLR**: Open review — rebuttals are public and matter. High value on reproducibility and code submission.
- **CVPR/ECCV**: Expect strong visual/qualitative results. Side-by-side comparison figures important. Large-scale benchmarks (ImageNet, COCO) expected.
- **ACL**: Error analysis expected. Human evaluation required for generation tasks. ARR rolling review.
- **AAAI**: Broader AI scope. Tighter page limits. Ethical statement may be required.

### 8.2: Figure Design

Use **seaborn (sns)** with matplotlib for all figures. Follow these conventions for publication quality:

#### Style Settings
```python
import seaborn as sns
import matplotlib.pyplot as plt
import matplotlib

# Publication-quality settings
sns.set_theme(style="whitegrid", font_scale=1.2)
matplotlib.rcParams.update({
    'font.family': 'serif',
    'font.serif': ['Times New Roman', 'DejaVu Serif'],
    'font.size': 12,
    'axes.labelsize': 14,
    'axes.titlesize': 14,
    'xtick.labelsize': 11,
    'ytick.labelsize': 11,
    'legend.fontsize': 10,
    'figure.dpi': 300,
    'savefig.dpi': 300,
    'savefig.bbox': 'tight',
    'text.usetex': False,  # Set True if LaTeX is available
})
```

#### Common Figure Types
- **Line plots** (metric vs. parameter sweep): `sns.lineplot` with markers, error bands
- **Bar charts** (method comparison): `sns.barplot` with error bars
- **Heatmaps** (corruption × method): `sns.heatmap` with annotated values
- **Reliability diagrams**: custom matplotlib (not seaborn)
- **Box plots** (distribution over seeds): `sns.boxplot`

#### Figure Guidelines
- Use a consistent color palette across all figures (define once, reuse). Recommend: `sns.color_palette("Set2")` or a custom palette with distinguishable colors.
- Always label axes with units. Use LaTeX-style notation for math: `$\mathrm{ECE}$`, `$\alpha$`.
- Legend inside the plot when possible; outside only if it would occlude data.
- Save as both PDF (for LaTeX) and PNG (for quick viewing) to `paper/figures/`.
- Figure width: single column = 3.25 inches, double column = 6.75 inches (for NeurIPS/ICML column widths).

### 8.3: Writing Style Rules

These rules are critical. Top-venue papers follow strict stylistic conventions. Violating them signals inexperience to reviewers.

**Structure:**
- **Abstract**: ONE paragraph, no line breaks. 150-250 words. Structure: problem → gap → method (1 sentence) → key results (specific numbers) → significance.
- **Introduction**: End with a bulleted or numbered contribution list.
- **Related Work**: Organize by theme, not chronologically. Use `\paragraph{}` for subsections. End each paragraph by contrasting with your work.
- **Method**: Start with problem setting/notation. Use `\paragraph{}` liberally. Include algorithm box.
- **Experiments**: Start with setup paragraph, then one subsection per experiment. Every table/figure must be referenced and analyzed in text.
- **Conclusion**: Short (0.5-1 page). Summarize contributions, acknowledge limitations, suggest future work.

**Writing conventions:**
- Never start a sentence with a math symbol or citation.
- Use `\citet` for "Author (Year)" in text, `\citep` for "(Author, Year)" parenthetical.
- Figures and tables at the TOP of the page (`[t]`), never floating mid-paragraph.
- Every claim must be backed by either a citation, a theorem, or an experiment.
- Use present tense for established facts ("CNNs are overconfident"), past tense for your experiments ("we observed that..."), and present tense for your method description ("TTAC computes the shift ratio...").
- Avoid vague language: "significantly" must accompany a statistical test; "state-of-the-art" must be backed by a table.
- Do not use "we" excessively — vary sentence structure.
- Equations should be part of a sentence (follow with punctuation).

**Formatting:**
- Tables: use `booktabs` (`\toprule`, `\midrule`, `\bottomrule`). No vertical lines. Bold the best result in each column/row.
- Figures: vector graphics (PDF/TikZ) preferred. Ensure readability at column width.
- References: consistent format. Use the venue's `.bst` file.

**Common mistakes to avoid (these will annoy reviewers):**
- Breaking paragraphs in the abstract
- Orphan/widow lines
- Figures/tables that aren't referenced in the text
- Overly long related work that reads like a literature survey
- Claims without evidence
- "Our method achieves state-of-the-art" without defining what "state-of-the-art" means
- Missing error bars / confidence intervals in tables
- Inconsistent notation (switching between $x$ and $\mathbf{x}$ for the same quantity)

### 8.4: Paper Generation

Use subagents to write sections in parallel where possible:
- **Subagent 1**: Related Work (needs `plan/literature_review.md`)
- **Subagent 2**: Method section (needs `plan/proposal.md`)
- **Subagent 3**: Experiment section (needs `experiments/results/*.csv` + `plan/experiment_plan.md`)
- **Main agent**: Introduction, Abstract, Conclusion (needs all above sections merged)

Generate a complete, compilable `.tex` file. Include:
- All necessary `\usepackage` declarations
- Math macros for repeated notation
- Inline `\begin{thebibliography}` for self-contained compilation (can be replaced with `.bib` later)

### 8.5: Compilation Check

After generating the paper, verify it compiles:
```bash
cd paper && pdflatex main.tex && pdflatex main.tex  # run twice for references
```
Fix any compilation errors before proceeding.

---

## Phase 9: Internal Review & Polish

Before the user takes over for submission, the paper should go through a rigorous internal review. This catches issues that are easy to fix now but embarrassing to discover in reviews.

### 9.1: Self-Review Checklist

Run through this checklist systematically. Fix issues as you find them:

**Content:**
- [ ] Abstract is ONE paragraph, no line breaks, 150-250 words
- [ ] Introduction ends with a contribution list
- [ ] Every contribution listed in the intro is backed by experiments
- [ ] Every table/figure is referenced and discussed in the text
- [ ] No claims without evidence (citations, theorems, or experiments)
- [ ] Limitations section is honest and specific (not generic disclaimers)

**Formatting:**
- [ ] Paper is within the page limit (check `references/venue_requirements.md`)
- [ ] No orphan/widow lines
- [ ] All figures readable at printed size
- [ ] Tables use `booktabs`, no vertical lines, best results bolded
- [ ] References consistent and complete (no "arXiv" when a venue version exists)

**Notation & Language:**
- [ ] Notation consistent throughout (same symbol for same concept everywhere)
- [ ] No sentences starting with math symbols or citations
- [ ] `\citet` vs `\citep` used correctly
- [ ] Equations are part of sentences (punctuated)
- [ ] Tense is consistent (present for method, past for experiments)

**Technical:**
- [ ] All error bars / confidence intervals present
- [ ] Statistical significance noted where claimed
- [ ] Hyperparameters fully specified (reproducibility)
- [ ] Algorithm pseudocode matches actual implementation

### 9.2: Codex Cross-Review

Send the full paper to Codex (GPT-5.4 High via MCP) for an independent review. A different model family catches different blind spots.

**Prompt for Codex:**
```
You are a reviewer for [venue]. Review this paper critically. For each section, note:
1. Clarity issues (confusing sentences, missing definitions)
2. Technical concerns (unsupported claims, missing comparisons, statistical issues)
3. Presentation issues (figure quality, table formatting, notation inconsistency)
4. Missing related work that should be cited
5. Overall recommendation: accept/borderline/reject, with justification

Be specific — cite section numbers and line numbers. Don't be polite, be useful.
```

Collect Codex's feedback and address each point. Log which concerns were addressed and which were deliberately not addressed (with reasoning) in `plan/codex_review.md`.

### 9.3: Rebuttal Preparation

Based on the Contrarian's objections (Phase 1), the Skeptic's concerns (Phase 7), and Codex's review (9.2), prepare draft rebuttals:

Save to `plan/rebuttal_prep.md`:
```markdown
# Rebuttal Preparation

## Anticipated Objections

### Objection 1: [likely reviewer concern]
**Source**: [Contrarian / Skeptic / Codex / self-review]
**Response**: [1-2 paragraph rebuttal with evidence]
**Additional experiment (if needed)**: [what to run if reviewer insists]

### Objection 2: ...
```

This document serves two purposes:
1. It helps you strengthen the paper NOW by preemptively addressing concerns
2. It gives the user a head start on writing the actual rebuttal during the review period

### 9.4: Final Polish

After addressing all review feedback:
- Re-compile the paper and verify it's error-free
- Check page limit one final time
- Ensure all figure/table captions are self-contained (a reader should understand the figure from the caption alone)
- Verify the abstract matches the actual results (easy to forget to update after experiments change)

---

## Reference: Venue-Specific Notes

See Section 8.1 for venue characteristics and web search instructions to fetch current requirements.

---

## Workflow Summary

```
User starts the skill
        │
        ▼
Phase 0: Interactive Setup (ONLY user interaction)
        │ ask: target venue, research topic
        │ discover SSH hosts, test connectivity, check GPUs
        │ ask: multi-select which machines to use
        │ saves: plan/config.md
        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    IDEA EXPLORATION LOOP                            │
│  (repeats with new ideas until pilot passes)                       │
│                                                                    │
│  Phase 1: Idea Exploration ────────────────────────────────┐       │
│          │ [Round 1+] check plan/idea_history.md            │       │
│          │ [Round 1 only] literature review                 │       │
│          │ auto idea generation (avoid archived ideas)      │       │
│          │ → idea debate (6 reviewers + AC gate)            │       │
│          │ AC: REJECT → try next direction                  │       │
│          │ AC: REVISE → auto-revise & re-debate (max 3)     │       │
│          │ AC: ACCEPT ↓                                     │       │
│          ▼                                                  │       │
│  Phase 2: Research Proposal                                 │       │
│          │ detailed method with theory, contributions       │       │
│          │ saves: plan/proposal.md                           │       │
│          ▼                                                  │       │
│  Phase 3: Pilot Experiment Design                           │       │
│          │ 1 dataset, 1-2 baselines, success criterion       │       │
│          │ saves: plan/pilot_experiment_plan.md              │       │
│          ▼                                                  │       │
│  Phase 4: Pilot Experiments                                 │       │
│          │ implement method, reproduce baselines, pilot test │       │
│          │ PASS? ──→ Phase 5.5 (full plan) → exit loop      │       │
│          │ FAIL? ──→ Phase 5                                │       │
│          ▼                                                  │       │
│  Phase 5: Method Iteration (max 3-5 tweaks)                 │       │
│          │ diagnose → revise → re-pilot                     │       │
│          │ fixed? → Phase 5.5 (full plan) → exit loop       │       │
│          │ still failing after 3-5 iterations?              │       │
│          │   → archive idea to plan/idea_history.md         │       │
│          │   → notify-telegram: idea failed                 │       │
│          └──→ back to Phase 1 (new idea, Round N+1) ────────┘       │
│                                                                    │
│  Every 3 failed rounds → notify-telegram: ask user to continue     │
│  or pivot topic                                                    │
└─────────────────────────────────────────────────────────────────────┘
        │ (pilot passed)
        ▼
Phase 5.5: Full Experiment Planning
        │ all datasets / baselines / ablations / analyses
        │ saves: plan/experiment_plan.md
        ▼
Phase 6: Full Experiments (AUTONOMOUS)
        │ monitor GPUs, greedy scheduling, run everything
        │ saves: experiments/results/*.csv
        ▼
Phase 7: Result Analysis
        │ 6-analyst debate, narrative, identify gaps
        │ need more experiments? → auto-run, back to Phase 6
        │ saves: plan/result_debate.md
        ▼
Phase 8: Paper Writing + Figures
        │ seaborn figures + parallel LaTeX writing
        │ saves: paper/figures/*.pdf, paper/main.tex
        ▼
Phase 9: Internal Review & Polish
        │ self-review checklist, Codex cross-review
        │ rebuttal preparation
        │ saves: plan/rebuttal_prep.md
        ▼
Done → notify-telegram: pipeline finished → Hand off to user for submission
```

**Interaction model:**
- **Phase 0 is the ONLY interactive phase** — user selects venue, topic, and machines.
- All subsequent phases run **fully autonomously** end-to-end. No further user input required.
- Phases 1-3: autonomous (auto-select idea direction, auto-proceed through proposal and pilot experiment design)
- Phases 4-5: autonomous (run pilots, iterate method 3-5 times; if still failing, archive idea and loop back to Phase 1 with a completely new idea; consult user after every 3 failed idea rounds)
- Phase 5.5: autonomous (full experiment planning after pilot passes)
- Phase 6: autonomous (run everything, report when done)
- Phases 7-9: autonomous (auto-decide on analysis, auto-write and review paper)
- Git commit + push after every phase
- **Telegram notification** (via `notify-telegram`) after every phase completion, on any pipeline block, and when finished
- The user provides keyword + target venue at the start, and receives the finished paper at the end.

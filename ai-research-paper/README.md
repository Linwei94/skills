# AI Research Paper Skill

A Claude Code skill for the full lifecycle of AI/ML research papers targeting CCF-A conferences (NeurIPS, ICML, ICLR, CVPR, ECCV, ACL, AAAI).

## 9-Phase Pipeline

1. **Idea Exploration** — literature review, brainstorming, 6-agent idea debate with AC gate
2. **Research Proposal** — detailed method with theory, positioning, contributions
3. **Experiment Planning** — comprehensive plan at top-venue scale
4. **Pilot Experiments** — proof-of-concept, baseline reproduction, sanity checks
5. **Method Iteration** — revise method based on pilot results
6. **Full Experiments** — autonomous GPU monitoring and execution across local/remote machines
7. **Result Analysis** — 6-agent result debate, narrative agreement
8. **Paper Writing + Figures** — publication-quality seaborn figures + venue-compliant LaTeX
9. **Internal Review & Polish** — self-review, Codex cross-review, rebuttal preparation

## Installation

```bash
claude skill install <your-github-username>/ai-research-paper-skill
```

Or manually copy the contents to `~/.claude/skills/ai-research-paper/`.

## Structure

```
ai-research-paper/
├── SKILL.md              # Main skill definition (9 phases)
├── agents/
│   ├── idea_debate.md    # 6 reviewer agents + AC for idea refinement
│   └── result_debate.md  # 6 analyst agents for result analysis
└── references/
    └── venue_guide.md    # CCF-A venue requirements and best practices
```

## Features

- **Multi-agent debate**: 6 specialized agents stress-test ideas and analyze results
- **AC gate**: Area Chair agent makes accept/reject decisions before proceeding
- **Autonomous experiments**: GPU monitoring, greedy scheduling, remote execution via SSH
- **Model hierarchy**: Heavy (Opus) / Standard / Light (Sonnet) / Codex (GPT via MCP)
- **Real research workflow**: pilot → iterate → full experiments (not waterfall)
- **Git integration**: commits after every phase, private GitHub repos

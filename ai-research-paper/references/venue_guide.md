# Venue Guide for CCF-A AI Conferences

## How to Get Latest Venue Requirements

Venue-specific details (deadlines, page limits, style files, submission rules) change every year. **Do NOT rely on memorized or cached information.** Instead, always perform a live web search when entering Phase 6:

### Required Searches (for the target venue + year)

Run ALL of these searches and extract the latest information:

1. **Style file and template**:
   ```
   "[venue] [year] latex template" OR "[venue] [year] style file"
   ```
   Download the official `.sty`/`.cls` file and save to `paper/`.

2. **Submission guidelines and page limits**:
   ```
   "[venue] [year] call for papers" OR "[venue] [year] submission guidelines"
   ```
   Extract: main body page limit, reference page policy, appendix/supplementary policy, anonymization rules, code submission policy.

3. **Deadline**:
   ```
   "[venue] [year] deadline" OR "[venue] [year] important dates"
   ```

4. **Review criteria and checklist**:
   ```
   "[venue] [year] reviewer guidelines" OR "[venue] [year] review form"
   ```
   Some venues (NeurIPS, ICML) have mandatory checklists — find and follow them.

### Venue Official Websites (starting points)

| Venue | Website |
|-------|---------|
| NeurIPS | https://neurips.cc |
| ICML | https://icml.cc |
| ICLR | https://iclr.cc |
| CVPR | https://cvpr.thecvf.com |
| ECCV | https://eccv.ecva.net |
| ACL | https://aclweb.org |
| AAAI | https://aaai.org |

### What to Record

After searching, save a summary to `plan/venue_requirements.md`:

```markdown
# Venue Requirements: [Venue] [Year]

- **Deadline**: [exact date and time with timezone]
- **Main body**: [N] pages
- **References**: [included in limit / unlimited]
- **Appendix/Supplementary**: [policy]
- **Style file**: [filename, downloaded to paper/]
- **Anonymization**: [yes/no, double-blind/single-blind]
- **Code submission**: [required/optional/not accepted]
- **Checklist**: [required? which one?]
- **Special requirements**: [anything unusual]
```

---

## Venue Characteristics (stable across years)

These are general tendencies that rarely change, useful for framing the paper:

### NeurIPS
- Values both theoretical and empirical contributions equally
- Reviewers: mix of senior researchers and PhD students, 3-4 reviewers + area chair
- Appreciates: honest limitations, well-motivated problems, clean writing
- Dislikes: pure empirical with marginal gains, overclaiming

### ICML
- Slightly more theory-oriented than NeurIPS
- Reviewers appreciate formal analysis and principled justification
- Appendix proofs expected for any theorem in main body

### ICLR
- Open review — all reviews and author responses are public
- Strong rebuttals can change outcomes
- Values reproducibility and code submission highly
- Frame work in terms of learned representations when possible

### CVPR / ECCV
- Vision-focused, expect strong visual/qualitative results
- Side-by-side comparison figures are important
- Results on large-scale benchmarks (ImageNet, COCO) expected
- Supplementary videos/demos valued

### ACL
- Error analysis is expected and valued
- Human evaluation required for generation tasks
- Multilingual experiments are a plus
- ARR rolling review system

### AAAI
- Broader AI scope, not just ML
- Tighter page limits — be concise
- More application-oriented work accepted
- Ethical statement may be required

---

## Universal Best Practices (never changes)

### Tables
- Use `booktabs` package: `\toprule`, `\midrule`, `\bottomrule`
- No vertical lines ever
- Bold best result per column
- Underline second-best (optional)
- Include mean +/- std with at least 3 seeds (5 preferred)
- Mark statistical significance with * (p < 0.05) or ** (p < 0.01)

### Figures
- Vector graphics (PDF) for plots, high-res PNG for photos/visualizations
- Ensure readability at actual printed size
- Use colorblind-friendly palettes
- Include axis labels with units

### Writing
- Active voice preferred ("We propose" not "It is proposed")
- One idea per paragraph
- Topic sentence first, then evidence/explanation
- Transition sentences between sections
- Consistent terminology throughout

### Citation Format
- Use `natbib` package with venue's `.bst` file
- `\citet{guo2017}` -> Guo et al. (2017) [subject of sentence]
- `\citep{guo2017}` -> (Guo et al., 2017) [parenthetical]
- Never start a sentence with `\citep` — use `\citet` instead
- Cite the published venue version, not arXiv, when available

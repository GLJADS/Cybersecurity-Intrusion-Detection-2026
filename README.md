# Cybersecurity Intrusion Detection (JM2030)

Group final project: find the malicious traffic in a PCAP with a data science technique, then write a threat report and present it.

- **Due:** 5 Dec 2026, 23:59 (upload as one zip). **Presentation:** 30 Nov or 1 Dec
- **Case:** not chosen yet (see `docs/Case and technique.md`)
- **Technique:** not chosen yet
- **Brief and grading criteria:** `assignment/Assignment brief.md`

## Layout

| Path | What lives there |
|---|---|
| `assignment/` | The Canvas brief and rubric, copied verbatim |
| `data/raw/` | PCAP and `dns.log`. **Git-ignored**, download from Canvas |
| `src/` | Python modules: feature extraction, detectors, evaluation |
| `notebooks/` | Exploration and the notebook that goes into the submission |
| `report/` | Early Quarto draft (`report.qmd`, `references.bib`, `template/`). **Not used any more**, the group writes the report in Overleaf |
| `report/_template-original/` | The original LaTeX template from Canvas, kept as fallback |
| `slides/` | Presentation (20 min plus 5 min questions) |
| `docs/` | Working notes: case and technique choice, task division, investigation log |

## The report

Written in **Overleaf** by the whole group (decided 2026-11-07). The Overleaf project can be opened in VS Code with the Overleaf Workshop extension. The Quarto draft in `report/` is kept only until the group confirms it can go.

## Python setup

```bash
uv venv
uv pip install -r requirements.txt
```

## Submission zip

Report PDF, slides (PPTX or PDF), and the code or notebook. Raw PCAPs are not part of the zip.

## Team workflow

- Work on a branch per person or per section, open a pull request, one other person reads before merge.
- Commit notebooks with outputs cleared (`jupyter nbconvert --clear-output --inplace`), so diffs stay readable.
- Never commit the PCAP or `dns.log`.

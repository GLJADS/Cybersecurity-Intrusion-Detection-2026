# Task division

Six people. Deadline 5 Dec 23:59. Presentation 30 Nov or 1 Dec (randomly assigned), so the practical deadline for slides is 29 Nov.

**Basis:** Case 1 (DNS tunneling, Zeek `dns.log` plus PCAP) with Isolation Forest. See `Case and technique.md`; still to be confirmed by the group and claimed on Canvas (deadline 4 Nov).

## Principle

Every person gets **one data analysis slice, one report section and one to two slides**. Nobody is "the writer" or "the coder" only. Reasons:

- The rubric grades analysis, mechanics and detection together, so everyone should be able to defend their part in the 5 minute Q&A.
- The data work is a pipeline (ground truth, then features, then model, then evaluation). Splitting it by stage makes it equal in size, but every stage depends on the one before it. The handover points below are therefore dated.
- Slides: 20 minutes divided by 6 is roughly 3 minutes each. Everyone presents their own part.

## The six slices

| # | Slice | Data analysis (code in `src/` or `notebooks/`) | Report sections | Slides (about 3 min) |
|---|---|---|---|---|
| **1** | **Discovery and ground truth** | Wireshark on `case1.pcap` and filters on `dns.log`: find the tunnel domain(s), the client(s), the time window. Label every row of `dns.log` as `benign` or `tunnel` and save `data/labels.csv` (git-ignored, row id plus label). Record method per step in `docs/Investigation log.md` | 2.2 Incident Description (timeline, TTP table, affected assets), Appendix A (IoCs) | Attack timeline and how we found it |
| **2** | **Baseline of normal DNS** | Exploratory analysis of the benign traffic: query rate per host, record types, response codes, label count and length distributions, top domains, TTLs, time of day. Output: plots and a table of "what normal looks like" | 1 Introduction (data description), 2.1 Protocols and Mechanics (DNS: message format, record types, resolution flow, Zeek fields) | DNS in 1 slide, what normal looks like |
| **3** | **Feature engineering** | `src/features.py`: per-query features (name length, label count, longest label, Shannon entropy, digit and uppercase ratio, record type, response size, rcode) and per-window features (unique subdomains per parent domain, queries per parent domain per minute, NXDOMAIN ratio). Output: one feature matrix aligned with the labels | 3.1 Features (table of every feature, what it measures, which characteristic it targets) | Feature table, 3 strongest features |
| **4** | **Isolation Forest model** | `src/detector.py`: train IsolationForest, score every row, `contamination` sweep, `n_estimators` and `max_samples` check, score distribution plot, which features drive isolation (permutation or SHAP-like check) | 3.2 Detection Method 1: Isolation Forest (motivation, implementation, parameters, steps taken) | How the model works, score plot |
| **5** | **Evaluation and alternatives** | `src/evaluation.py`: precision, recall, F1, PR curve, confusion matrix per `contamination`, **FP and FN case studies** (look at the actual rows and explain why). Comparison with a simple rule baseline (length plus entropy threshold) and one alternative (LOF or OCSVM) | 3.3 Empirical Evaluation, 3.4 Theoretical Evaluation (advantages, disadvantages, alternatives, typical environments) | Results, FP and FN, comparison |
| **6** | **Threat intelligence, mechanics, mitigation, assembly** | No model code. Collect CTI on DNS tunneling tools and campaigns, map to MITRE ATT&CK, fill the characteristic table with the five questions from the brief (specificity, benign overlap, evasion difficulty, automatable, stage). Owns the final build | 2.3 Attack Mechanics (characteristics table), 4 Mitigation, 5 Discussion, 6 Conclusion, `references.bib`. **Final assembly:** zip, proofread pass, consistent terminology | Mitigation, conclusion, plus deck assembly and rehearsal lead |

**Why slice 6 has no model code:** it carries the heaviest writing load and the final assembly, so the data slices are sized to even that out. If the group prefers, slice 6 also takes a small stretch goal: a second dataset or an attack variation (optional in the brief).

## Handover points (the critical path)

1. **Slice 1 to everyone: `labels.csv` by 8 Nov.** Without ground truth there are no metrics. Slice 1 should publish a first label set early, even if incomplete, and refine it.
2. **Slice 2 and 3 agree on the schema on 9 Nov.** One shared dataframe format (row id, `ts`, `id.orig_h`, `query`, `qtype_name`, `rcode_name`, plus the label).
3. **Slice 3 to slice 4: feature matrix by 14 Nov.**
4. **Slice 4 to slice 5: scored output (`scores.csv`) by 16 Nov.**
5. **Slice 6 receives characteristic evidence from slices 1 to 3 by 15 Nov** so the characteristics table is based on the data, not on the literature alone.

While waiting for a handover, each person writes the background part of their own report section (what they know from the literature) so nobody sits idle.

## Pairing for review

Each slice is read and challenged by one other person before merge. Pairs rotate the reviewer so every part gets a second view:

| Author | Reviewer |
|---|---|
| 1 | 2 |
| 2 | 3 |
| 3 | 4 |
| 4 | 5 |
| 5 | 6 |
| 6 | 1 |

## Milestones

- By 4 Nov: group fixed, case and technique claimed on Canvas, slices assigned
- By 8 Nov: ground truth v1 (`labels.csv`), baseline plots
- By 11 Nov: data explored, attack identified, first TTP list
- By 14 Nov: feature matrix ready
- By 18 Nov: detector running with first metrics
- By 22 Nov: FP and FN analysis done, report section drafts complete
- By 25 Nov: full report draft, slides draft
- By 29 Nov: slides frozen, rehearsed
- By 3 Dec: report proofread, zip assembled. Buffer day on 4 Dec

## Assignment of names

| # | Owner | Reviewer |
|---|---|---|
| 1 | TBD | TBD |
| 2 | TBD | TBD |
| 3 | TBD | TBD |
| 4 | TBD | TBD |
| 5 | TBD | TBD |
| 6 | TBD | TBD |

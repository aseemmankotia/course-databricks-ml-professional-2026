## Chapter 12 Cheat Sheet: Test-Day Strategy

### Time Budget (120 min / 60 questions)
| Phase | Time | Questions |
|---|---|---|
| First pass | 105 min | All 60 (≈1.75 min/q) |
| Flagged review | 15 min | Flagged only |
| Naive baseline (don't use) | 120 min / 60 = 2 min/q | Ignores review reservation |

### Flag-and-Skip Rule
- If a question takes noticeably longer than ~1.75 min to even parse → **guess a placeholder, flag it, move on.**
- Never leave a question blank — single-answer format means blank = guaranteed zero.
- Return to flags only in the final 15-minute window, in priority order below.

### Domain-Weighted Review Priority (when time-limited)
| Priority | Domain |
|---|---|
| 1 | Model Development |
| 2 | MLOps |
| 3 | Model Deployment |
| 4 | Data Prep / lower-weight domains |

Study allocation before test day should mirror this order — spend the most prep hours on Model Development and MLOps.

### Elimination Technique (3-step)
1. **Cut category mismatches first** — remove any option that isn't even the right *type* of solution (e.g., a correlation metric on a drift-detection question).
2. **Cut deprecated/wrong-service options** — anything referencing a retired feature or wrong tool.
3. **Compare the remaining 2** against the specific constraint in the question stem (cost, latency, compliance, timestamp requirement).

### Keyword Cue Table
| Phrase in scenario | Signals |
|---|---|
| "as it existed at [timestamp]" | Point-in-time join |
| "gradual rollout", "small % of traffic" | Canary deployment |
| "instant switch", "two full environments" | Blue-Green deployment |
| "categorical feature distribution shift" | Chi-squared test / PSI |
| "continuous feature distribution shift" | Kolmogorov-Smirnov test |

### Remote-Proctoring Logistics Checklist
- ✅ Single monitor only (disconnect second display)
- ✅ Clear desk/room, webcam sweep at check-in
- ✅ No phones, papers, books, or headphones unless pre-approved accommodation
- ✅ Valid photo ID ready before check-in starts
- ✅ Stable webcam + mic connection tested beforehand

### Final 15-Minute Checklist
1. Sort flagged questions by domain weight (Model Dev/MLOps first).
2. Re-read only the question stem + options — skip re-reading full scenario paragraphs unless still unclear.
3. Apply elimination technique before committing to a final answer.
4. Confirm every question has an answer selected (no blanks) before submitting.

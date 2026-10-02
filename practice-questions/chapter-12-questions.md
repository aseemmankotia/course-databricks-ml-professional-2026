## Chapter 12: Full Practice Exam Simulation and 120-Minute Test-Day Strategy — Practice Questions

**Q1.** You are 90 minutes into the Databricks ML Professional certification exam with 40 questions completed out of 60, and 30 minutes remain. You have flagged 8 questions as uncertain. What is the BEST use of your remaining time?

A) Immediately go back and rewrite answers to all 8 flagged questions before finishing the remaining 20 unanswered questions
B) Answer the remaining 20 unanswered questions first at your normal pace, then use whatever time remains to review the 8 flagged questions in priority order
C) Skip the remaining 20 questions, submit early, and only review flagged questions
D) Change your answer for every flagged question to the same option to guarantee partial credit

<details>
<summary>Answer</summary>

**Correct: B**

Answering unattempted questions first guarantees you don't leave points on the table from automatic zeros; flagged review only happens after full first-pass coverage since time is limited.

**Why the others are wrong:**
- Leaves 20 questions completely unanswered, guaranteeing zero score on those — riskier than deferring flag review
- Guarantees a score loss on 20 unattempted questions for no benefit
- No partial credit exists on single-answer multiple choice; picking one repeated option wastes time and helps nothing

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** During the exam you hit a lengthy scenario describing a multi-stage MLOps pipeline with model registry transitions, an A/B test configuration, and a drift monitoring job, all in one dense paragraph. You are 45 seconds in and haven't found the actual question being asked. Per the flag-and-skip strategy, what should you do?

A) Flag the question, skip it, and continue to the next question, returning during the final review window
B) Continue rereading the paragraph from the start until every detail makes sense before looking at the answer options
C) Guess an answer immediately without reading the options, to protect your pace
D) Spend up to 5 minutes fully diagramming the pipeline described before evaluating any option

<details>
<summary>Answer</summary>

**Correct: A**

Flag-and-skip preserves your pacing baseline by removing questions that threaten disproportionate time consumption; you return to it with fresh context during the dedicated review window.

**Why the others are wrong:**
- Rereading without checking the question/options first wastes time and doesn't move you toward finishing the exam
- Blind guessing without reading the options forfeits an easy win if the options would have clarified the ask
- 5 minutes is far above baseline pacing and jeopardizes finishing all 60 questions

</details>

---

**Q3.** A scenario question states: 'The feature table has a timestamp column and features change over time. The model must be scored using feature values exactly as they existed at each training example's label timestamp.' Which keyword cue tells you to use a point-in-time join rather than a standard join?

A) The presence of a timestamp column, which always requires a streaming join
B) The requirement that feature values match the exact state 'as they existed at' the label's timestamp
C) The mention of a feature table, since all feature table joins are automatically point-in-time
D) The word 'model,' which indicates any join type is acceptable

<details>
<summary>Answer</summary>

**Correct: B**

The phrase 'as it existed at' a specific timestamp is the classic keyword cue for point-in-time joins in Databricks Feature Store, which prevent label leakage from future feature values.

**Why the others are wrong:**
- A timestamp column alone doesn't require streaming; many batch feature tables carry timestamps without needing point-in-time joins
- Not all feature table joins are point-in-time by default; you must explicitly specify a timestamp lookup key
- The word 'model' carries no information about join type

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** With 10 minutes remaining and 6 flagged questions still unreviewed — 2 from Model Development, 2 from MLOps, 1 from Data Prep, 1 from Model Deployment — which review order follows domain-weighted review priority?

A) Alphabetical order by domain name
B) Review Model Development and MLOps flagged questions first, then Model Deployment, then Data Prep
C) Review whichever question number appears first in the exam regardless of domain
D) Review only Data Prep and skip the rest since it's the smallest domain

<details>
<summary>Answer</summary>

**Correct: B**

Model Development and MLOps carry the highest domain weight on this exam, so securing correct answers there yields the greatest score impact per minute of review.

**Why the others are wrong:**
- Alphabetical order has no relationship to score impact
- Question order ignores which domain carries more weight
- Skipping higher-weighted domains to focus only on the lowest-weighted domain minimizes potential score gain

</details>

---

**Q5.** The exam allocates 120 minutes for 60 questions, with the final 15 minutes reserved exclusively for reviewing flagged questions. What is the effective per-question pacing baseline for the FIRST PASS through all 60 questions?

A) Approximately 1.75 minutes per question (105 minutes / 60 questions)
B) 2 minutes per question (120 minutes / 60 questions, ignoring the review reservation)
C) 3 minutes per question (180 minutes assumed)
D) 1 minute per question (60 minutes assumed)

<details>
<summary>Answer</summary>

**Correct: A**

Reserving the final 15 minutes for flagged review leaves 105 minutes for the first pass across 60 questions: 105/60 ≈ 1.75 minutes per question, tighter than the naive 2-minute figure.

**Why the others are wrong:**
- Ignores the 15-minute review reservation and overestimates available first-pass time
- 180 minutes is not the exam duration
- 60 minutes is not the exam duration; underestimates available time

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** You are setting up your workstation for a remote-proctored Databricks certification exam. Which setup is compliant with standard remote-proctoring requirements?

A) A dual-monitor setup with the second monitor displaying browser tabs for reference
B) A single monitor, webcam-enabled device, in a clear, empty room with no papers, books, or secondary devices visible
C) Noise-canceling headphones playing background music to stay focused
D) A phone placed face-down within reach in case the proctor needs to be contacted by text

<details>
<summary>Answer</summary>

**Correct: B**

Standard remote-proctoring policies require a single monitor, an unobstructed clear desk/room, and continuous webcam visibility to prevent unauthorized materials or assistance.

**Why the others are wrong:**
- Dual monitors are prohibited since the second screen could display unauthorized reference material
- Headphones/audio playback devices are typically prohibited unless explicitly approved as an accommodation
- Any secondary device within reach, even face-down, is commonly flagged as a violation by proctoring software

</details>

---

**Q7.** A scenario question asks which drift detection approach is BEST for a categorical feature. The options are: (A) Kolmogorov-Smirnov test, (B) Population Stability Index, (C) Chi-squared test, (D) Pearson correlation coefficient. Using the elimination technique, which option can be eliminated FIRST because it is not a distribution-comparison metric at all?

A) Pearson correlation coefficient, because it measures linear association between two variables, not distributional drift
B) Kolmogorov-Smirnov test, because it only applies to categorical data
C) Population Stability Index, because it cannot be used with categorical bins
D) Chi-squared test, because it only works on continuous numeric features

<details>
<summary>Answer</summary>

**Correct: A**

Pearson correlation measures the linear relationship between two variables, not a shift in distribution over time, making it immediately eliminable as a drift-detection metric.

**Why the others are wrong:**
- KS test is actually designed for continuous distributions, but this justification is incorrect and it's not the fastest first elimination for categorical drift
- PSI is commonly used with binned data including categorical bins, so this justification is wrong
- Chi-squared test is specifically suited for categorical/frequency data, so this justification is backwards

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** What is the primary purpose of flagging a question during the exam rather than immediately guessing and moving on?

A) Flagging preserves your place and lets you return with fresh context during dedicated review time, rather than losing the question entirely or over-investing time now
B) Flagging automatically awards partial credit for showing engagement with the question
C) Flagging changes the difficulty weighting of the question to lower your risk
D) Flagging is required by the proctor to prove you attempted every question

<details>
<summary>Answer</summary>

**Correct: A**

Flagging lets you make a placeholder guess (never leave it blank) and revisit with a clear head during the dedicated review window, balancing pace preservation against accuracy.

**Why the others are wrong:**
- No certification exam awards partial credit for flagging behavior
- Flagging has no effect on question weighting or scoring difficulty
- Proctors don't require flagging; it is a self-management tool, not a compliance requirement

</details>

---


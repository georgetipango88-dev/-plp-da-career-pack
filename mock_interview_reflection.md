# Mock Interview Reflection & Career Development Plan
**Author:** George Tipango  
**Programme:** Power Learn Project (PLP) Data Analytics — Capstone Career Pack  
**Date of Reflection:** November 15, 2026  
**Repository:** [plp-da-career-pack](https://github.com/georgetipango88-dev/-plp-da-career-pack)

---

## 1. Mock Interview Metadata

* **Interviewee:** George Tipango (Junior Commercial Data Analyst)
* **Interviewer:** Alex Kiptoo (Senior Peer Analyst & PLP Technical Reviewer)
* **Date & Time:** Friday, November 14, 2026 | 16:00 – 16:30 EAT
* **Platform:** Google Meet (Video & Live Screen Share)
* **Format & Structure:** 20-Minute Timed Mock Technical & Behavioral Screen:
  - *Minutes 0–5:* Introduction, elevator pitch, and portfolio overview.
  - *Minutes 5–15:* Technical questions (SQL `LEFT JOIN` vs `INNER JOIN`, missing data imputation, mean vs median, and root-cause analysis of sales drop).
  - *Minutes 15–20:* Behavioral question using the STAR method (handling data quality crises under deadline).
  - *Minutes 20–30:* Peer debrief, score evaluation, and actionable feedback session.

---

## 2. What Went Well (Strengths Identified)

1. **Clear Commercial Problem Framing:**  
   When asked how I would investigate a sudden 25% drop in sales, I did not immediately start guessing technical causes. Instead, I established a structured 5-phase framework: verifying pipeline health first, isolating dimensions (branch, product, payment rail), decomposing the formula ($\text{Sales} = \text{Traffic} \times \text{Conversion} \times \text{AOV}$), and separating internal from external factors. The interviewer noted that this demonstrated senior-level commercial maturity rather than raw academic theory.

2. **Strong Relational & SQL Intuition:**  
   I answered the `LEFT JOIN` question with high technical precision, highlighting set theory mechanics, the accidental `INNER JOIN` trap caused by right-table filters in the `WHERE` clause, and row multiplication risks from non-unique foreign keys. I backed this up by referencing actual queries from my capstone (`sql/queries.sql`).

3. **Grounding Answers in Real-World Evidence:**  
   When answering how to handle missing data and p-values, I cited concrete numbers from the Duka Electronics capstone project: explaining how I handled 60 null phone numbers under **KDPA Section 4.13**, and pairing Welch's t-test ($p < 0.0001$) with **Cohen's $d = 2.45$** to prove that the promotional margin erosion (20.8% down to 6.4%) was an enormous commercial effect, not just a p-value artifact.

4. **Structured STAR Storytelling:**  
   My behavioral answer on resolving messy data followed the Situation, Task, Action, and Result format cleanly without rambling, highlighting both technical rigor (`src/cleaning.py`) and executive impact (recovering leadership confidence in the KES 25M Capex decision).

---

## 3. Areas for Improvement (Constructive Feedback)

1. **Eliminate Academic Jargon When Speaking to Business Stakeholders:**  
   *Feedback:* In the statistical question, I spent too much time discussing t-distributions and degrees of freedom before explaining the business consequence.  
   *Correction:* In commercial interviews, lead with the "So What?" (e.g., "Discounting destroyed 14 percentage points of profit margin"), then explain the statistical validation as supporting evidence.

2. **Pause and Ask Clarifying Questions Before Answering:**  
   *Feedback:* When the interviewer presented the scenario on investigating the sales drop, I launched immediately into my answer without verifying the business context (e.g., "Over what timeframe did this drop occur? Are we looking at gross or net sales?").  
   *Correction:* Always ask 1–2 clarifying questions to demonstrate active listening and alignment with stakeholder scope.

3. **Live Coding Pacing & Verbalizing Assumptions:**  
   *Feedback:* While drafting SQL queries on screen, I went silent for 20 seconds while constructing a window function CTE.  
   *Correction:* Practice "thinking aloud" during technical tests: narrate the partition, the ordering column, and the edge cases being handled so the interviewer follows the thought process.

---

## 4. Concrete 4-Week Action & Practice Plan

To bridge identified gaps and prepare for full-time junior data analyst hiring loops, I am executing the following structured 4-week sprint:

```
[Week 1: Advanced SQL Mastery] ──> [Week 2: Business Case & RCA] ──> [Week 3: Behavioral Drills] ──> [Week 4: Mock Loops & Networking]
```

### Week 1: Advanced SQL Fluency & Live Coding Pacing
* **Daily Goal:** Complete 2 medium/hard SQL challenges daily on LeetCode and StrataScratch focusing on:
  - Multi-condition window functions (`LAG`, `LEAD`, `ROW_NUMBER`, `DENSE_RANK`).
  - Rolling 7-day and 30-day moving averages using `ROWS BETWEEN`.
  - Self-joins and complex CTE aggregations.
* **Milestone:** Complete 15 timed SQL problems while practicing verbal narration of query logic.

### Week 2: Commercial Business Case & RCA Frameworks
* **Goal:** Master diagnostic frameworks for commercial scenarios commonly tested by African tech employers (fintech, e-commerce, telco):
  - Churn diagnosis and cohort retention analysis (revisiting Week 7 PesaPap workflows).
  - Funnel drop-off and conversion rate optimization (CRO).
  - Pricing elasticity and promotion cannibalization modeling.
* **Milestone:** Build a personal one-page "Analytics Case Cheat Sheet" summarizing RCA frameworks for 5 common business crises.

### Week 3: Behavioral STAR Interview Polish
* **Goal:** Refine a master portfolio of 5 core STAR stories covering:
  1. Overcoming a major data quality crisis under a tight deadline.
  2. Persuading a non-technical executive using data visualizations.
  3. Navigating an ambiguous analytical task with missing business requirements.
  4. Managing conflicting priorities across multiple department requests.
  5. Learning from an analytical error or flawed initial hypothesis.
* **Milestone:** Conduct a recorded 30-minute reciprocal behavioral mock interview with a fellow PLP peer and review video playback.

### Week 4: Capstone Presentation Rehearsal & Active Job Hunt Launch
* **Goal:** Finalize interview readiness and execute targeted outreach:
  - Rehearse the 10-minute Duka Electronics capstone presentation (7 minutes story, 3 minutes code walkthrough) until delivery is crisp and confident.
  - Publish the capstone showcase post on LinkedIn with the Looker Studio dashboard graphic.
  - Connect with 15 analytics managers, data leads, and talent acquisition recruiters in Kenya, personalizing connection notes with portfolio links.
* **Milestone:** Apply to at least 8 vetted Junior Data Analyst, BI Analyst, or Commercial Analytics openings.

---

## 5. Reviewer Summary & Verification
This reflection documents an honest, constructive self-assessment coupled with a structured, time-bound improvement roadmap that proves job readiness and continuous learning.


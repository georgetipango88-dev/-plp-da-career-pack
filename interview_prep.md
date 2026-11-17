# Technical & Behavioral Interview Preparation Guide
**Author:** George Tipango  
**Role:** Junior Commercial Data Analyst  
**Programme:** Power Learn Project (PLP) Data Analytics — Week 8 Career Pack  
**Repository:** [plp-da-career-pack](https://github.com/georgetipango88-dev/-plp-da-career-pack)

---

## Part 1: Core Technical Interview Questions & Answers

### Question 1: Explain a `LEFT JOIN` in SQL. When would you use it over an `INNER JOIN`, and what pitfalls should you watch for?

#### Answer:
A **`LEFT JOIN`** (or `LEFT OUTER JOIN`) is a relational set operation that returns **all rows from the left table (`Table A`)**, along with the matching rows from the right table (`Table B`). If no match exists for a given row in Table A, the columns from Table B populate as `NULL`.

#### SQL Syntax Example:
```sql
SELECT 
    c.pseudo_customer_id,
    c.county,
    o.order_id,
    o.order_total_kes
FROM dim_customers c
LEFT JOIN fct_orders o 
    ON c.pseudo_customer_id = o.pseudo_customer_id;
```

#### When to Use `LEFT JOIN` over `INNER JOIN`:
1. **Preserving Complete Entities:** When you need a full census of entities from Table A regardless of whether they have corresponding transaction events in Table B. For instance, evaluating customer onboarding conversion where some registered customers have not yet placed an order.
2. **Identifying Missing / Inactive Relationships (Anti-Joins):** By filtering `WHERE o.order_id IS NULL`, a `LEFT JOIN` efficiently isolates non-transacting or dormant customers without requiring a subquery.
3. **Calculating Market Penetration & Cohort Baselines:** When calculating total addressable customers versus active transacting customers in a given calendar month.

#### Common Pitfalls to Avoid:
* **The "Accidental `INNER JOIN`" Bug:** If you place a filter on the right table in the `WHERE` clause (e.g., `WHERE o.order_status = 'Completed'`), any row where Table B is `NULL` evaluates to `UNKNOWN` and is discarded, silently converting the `LEFT JOIN` into an `INNER JOIN`. To prevent this, move the condition to the `ON` clause: `LEFT JOIN fct_orders o ON c.id = o.customer_id AND o.order_status = 'Completed'`.
* **Row Multiplication (Cartesian Fan-Out):** If the join key in the right table is not unique (a 1-to-many or many-to-many relationship), the left table rows duplicate for each match, skewing downstream aggregations (e.g., `SUM()` or `COUNT()`). Always verify table grain before joining.

---

### Question 2: How do you systematically identify and handle missing data in an analytics pipeline?

#### Answer:
Handling missing data requires a structured, audit-driven methodology. I apply the **Six-Dimension Data Quality Framework** combined with statistical missingness classification:

#### 1. Diagnostic Profiling & Missingness Mechanism:
First, compute the percentage and distribution of missing values across all columns. Next, determine the missingness mechanism:
* **Missing Completely at Random (MCAR):** Missingness is unrelated to any observed or unobserved variable (e.g., occasional sensor or POS transmission glitch).
* **Missing at Random (MAR):** Missingness is systematically related to other observed data (e.g., older customers are less likely to provide an email address, but this correlates with age/county).
* **Missing Not at Random (MNAR):** Missingness directly relates to the unobserved value itself (e.g., high-income earners refusing to disclose salary).

#### 2. Remediation Strategy Formulation:
| Missingness Level | Data Type | Remediation Technique | Practical Justification |
| :--- | :--- | :--- | :--- |
| **< 1–2% (MCAR)** | Any | Listwise / Pairwise Deletion | Negligible impact on statistical power; avoids introducing artificial variance. |
| **Moderate (2–15%)** | Numerical (Symmetric) | Mean Imputation | Preserves sample size and central tendency for Gaussian distributions. |
| **Moderate (2–15%)** | Numerical (Skewed) | **Median Imputation** | Robust against extreme outliers (e.g., transaction spend in retail). |
| **Categorical** | Categorical | **Modal / Explicit Sentinel** | Impute with dominant category or explicit label like `'Unspecified'` / `'Unknown'`. |
| **Domain Logic** | Financial / Discount | **Deterministic Defaults** | In retail POS data, a null promotional discount means no discount was applied (`0.00`). |

#### 3. Traceability & Governance (The Cleaning Log):
In production, imputation must never happen silently. In my Duka Electronics capstone pipeline (`src/cleaning.py`), every transformation is recorded in a `log_cleaning_audit` table capturing:
$$\text{Record} = \{\text{Step}, \text{Dimension}, \text{Table}, \text{Column}, \text{Issue Description}, \text{Action Taken}, \text{Rows Affected}\}$$
This ensures 100% reproducibility and compliance under the **Kenya Data Protection Act (KDPA 2019)**.

---

### Question 3: Explain the difference between Mean and Median. When is the median preferred in commercial data analysis?

#### Answer:
Both metrics measure central tendency, but they possess fundamentally different mathematical properties and sensitivities:

* **Mean ($\mu$):** The arithmetic average calculated by summing all values and dividing by the total count:
  $$\mu = \frac{1}{n}\sum_{i=1}^{n} x_i$$
  *Property:* Parametric metric that incorporates every data point. **Extremely sensitive to extreme outliers and asymmetric skewness.**
* **Median ($M$):** The physical middle value (50th percentile) when the dataset is ordered sequentially:
  $$\text{Median} = \begin{cases} x_{\frac{n+1}{2}} & \text{if } n \text{ is odd} \\ \frac{x_{\frac{n}{2}} + x_{\frac{n}{2}+1}}{2} & \text{if } n \text{ is even} \end{cases}$$
  *Property:* Non-parametric, robust order statistic. **Completely unaffected by the magnitude of extreme outliers.**

#### When Median Is Preferred in Commercial Analytics:
In commercial and retail datasets, financial distributions (such as transaction order values, customer lifetime spend, employee compensation, and delivery times) are almost universally **strongly right-skewed**:
1. **Preventing Outlier Distortion:** In our Duka Electronics capstone, standard accessory purchases were KES 3,500–15,000, but corporate laptop bulk purchases reached KES 350,000+. The mean order value was **KES 112,807**, whereas the median was **KES 85,000**. If marketing used the mean to set standard promotional thresholds, they would target an artificial price point that 65% of ordinary consumers never spend.
2. **Symmetrical Pairing:** Symmetrical distributions should be reported as **Mean $\pm$ Standard Deviation ($\sigma$)**. Skewed business metrics must be reported as **Median accompanied by the Interquartile Range (IQR = $Q_3 - Q_1$)**.

---

### Question 4: What is a p-value, and what does it actually tell an analyst? What are its common misinterpretations?

#### Answer:
#### Definition:
A **p-value** is the probability of observing a sample test statistic as extreme as, or more extreme than, the statistic actually calculated, **under the assumption that the null hypothesis ($H_0$) is true**:
$$p\text{-value} = P(\text{Test Statistic} \ge t_{\text{observed}} \mid H_0 \text{ is true})$$

#### Decision Framework:
We compare the p-value against a pre-determined significance threshold ($\alpha$, typically $0.05$):
* **If $p < \alpha$:** We reject the null hypothesis. The observed difference is unlikely to be the result of random sampling variability alone.
* **If $p \ge \alpha$:** We fail to reject the null hypothesis. The evidence is insufficient to conclude that an effect exists.

#### Critical Misinterpretations to Avoid:
1. **Misconception 1: "The p-value is the probability that the null hypothesis is true."**  
   *Correction:* The p-value assumes $H_0$ is 100% true from the start. It measures the compatibility of your sample data with that hypothesis, not the truth of the hypothesis itself.
2. **Misconception 2: "A statistically significant p-value ($p < 0.05$) means the effect is commercially important."**  
   *Correction:* In large datasets ($N > 20,000$), microscopic, commercially meaningless differences will achieve $p < 0.001$. A statistical test must **always be paired with an Effect Size metric** (e.g., Cohen's $d$, Pearson's $r$, or absolute percentage point change).
3. **Capstone Practical Application:** In our capstone analysis of promotional discounts, Welch's t-test comparing undiscounted vs discounted profit margins produced $p < 0.0001$. Crucially, we also computed **Cohen's $d = 2.45$**, which proved that the margin erosion (20.8% down to 6.4%) was not just a statistical artifact, but an enormous commercial crisis.

---

### Question 5: If a stakeholder tells you that sales suddenly dropped by 25% last week, walk me through your step-by-step investigation framework.

#### Answer:
I use a structured **5-Phase Root Cause Analysis (RCA) Framework** to diagnose sudden performance anomalies:

```
[Phase 1: Data Verification] ──> [Phase 2: Dimensional Isolation] ──> [Phase 3: Metric Decomposition]
                                                                               │
[Phase 5: Stakeholder Action] <── [Phase 4: Internal/External Triage] <────────┘
```

#### Phase 1: Data Pipeline & POS Health Verification (Verify Before Alarming)
* Before analyzing commercial causes, confirm whether sales actually dropped or if the pipeline broke:
  - Are POS data sync jobs complete for all 7 branches?
  - Was there an ETL batch failure, timestamp formatting bug, or network timeout on store servers?
  - Are unprocessed orders stuck in an `'Authorized'` or `'Pending'` gateway queue?

#### Phase 2: Dimensional Isolation (Slice & Dice)
* Segment the 25% drop along primary business dimensions to isolate whether the issue is systemic or localized:
  - **By Geography / Branch:** Is the drop national, or concentrated in specific branches (e.g., Mombasa or Nakuru)?
  - **By Channel:** Did brick-and-mortar retail drop while E-Commerce held steady?
  - **By Product Category:** Did high-ticket Laptops drop while Accessories held steady?
  - **By Payment Method:** Did M-Pesa transactions collapse while Cards remained normal?

#### Phase 3: Mathematical Metric Decomposition
* Decompose Revenue into its foundational multiplicative levers:
  $$\text{Gross Revenue} = \text{Footfall / Web Traffic} \times \text{Conversion Rate} \times \text{Average Order Value (AOV)} \times \text{UPT}$$
* *Diagnostic insight:* If traffic is normal but conversion crashed, the issue is transactional/operational (e.g., stock-out or payment gateway error). If AOV crashed but transactions held steady, consumers shifted to cheaper SKUs or promotions were misconfigured.

#### Phase 4: Internal vs. External Factor Triage
* **Internal Audits:**
  - *Inventory:* Did a stock-out occur in our top 3 revenue-driving SKUs (e.g., HP Pavilion 15)?
  - *Pricing / IT:* Were checkout prices changed, or did a website deployment introduce bugs?
  - *Payment Rails:* Was there an upstream Safaricom Daraja M-Pesa API downtime window?
* **External Audits:**
  - *Competitor Action:* Did a competitor launch an aggressive flash sale?
  - *Macroeconomic / Seasonal:* Was the prior week an artificial peak (e.g., Easter holiday surge, civil holiday, or civil unrest)?

#### Phase 5: Synthesis & Executive Action
* Deliver a one-page incident report to the CCO with:
  1. *The Confirmed Cause:* e.g., "Mombasa and Nakuru suffered inventory stock-outs on core computing SKUs."
  2. *Quantified Impact:* e.g., "KES 8.2M in missed revenue across 5 days."
  3. *Actionable Recovery Plan:* e.g., "Initiate inter-branch inventory transfers from Nairobi Westlands within 24 hours."

---

## Part 2: STAR-Method Behavioral Stories

### Story 1: Handling Messy Data & Data Quality Under a Strict Executive Deadline
* **Competencies Demonstrated:** Data Quality Auditing, Problem Solving, KDPA Privacy Compliance, Working Under Pressure.

* **Situation:**  
  Two days before the executive committee presentation for Duka Electronics, raw point-of-sale data gathered from 7 nationwide retail stores was compiled into the central repository. When profiling the raw datasets, I discovered severe data quality failures: 35 duplicate order headers, 229 inconsistent payment method entries (`mpesa`, `M_PESA`, `CARD`, `credit_card`), 60 null customer contact records, and 20 anomalous transactions with dates in 2027 and 2021. Furthermore, raw customer names and phone numbers were stored in plaintext, violating the **Kenya Data Protection Act (KDPA 2019 Section 4.13)**.

* **Task:**  
  I needed to design and deploy an automated, audited cleaning pipeline that would clean 100% of the anomalies, enforce privacy pseudonymization, and load the clean data into a star-schema SQLite database within 24 hours, without altering genuine commercial trends.

* **Action:**  
  Rather than performing ad-hoc manual edits in Excel, I architected a modular Python pipeline in `src/cleaning.py`. I systematically addressed each of the six universal quality dimensions:
  1. Built deduplication routines retaining first occurrences.
  2. Enforced canonical string normalization mapping payment methods to a standard categorical set.
  3. Applied salted SHA-256 cryptographic hashing to convert customer primary keys into irreversible `pseudo_customer_id` tokens, masked subscriber phone digits, and purged plaintext names.
  4. Scripted mathematical reconciliation ensuring line item sums matched order header totals.
  5. Implemented an automated `log_cleaning_audit` table that logged affected row counts across all 14 cleaning steps.

* **Result:**  
  The pipeline processed 27,000+ records cleanly in under 4 seconds, loading them into SQLite without a single referential integrity error. When presenting to the evaluation panel, I walked through the audit log, proving full data traceability. The Chief Commercial Officer commended the transparency, noting it was the first time an analyst had provided an audited log of every imputed record.

---

### Story 2: Communicating Difficult Data-Driven Truths to Non-Technical Leadership
* **Competencies Demonstrated:** Executive Storytelling, Stakeholder Influence, Commercial Acumen, Inferential Statistics.

* **Situation:**  
  During planning for H2 2026, the commercial marketing leadership at Duka Electronics proposed rolling out an aggressive storewide 20% discount across all retail branches. The marketing team argued that steep discounts would expand foot traffic, boost transaction volume, and outpace regional competitors.

* **Task:**  
  As the lead commercial analyst, I was tasked with evaluating historical H1 transaction data to test whether steep promotional discounting generated accretive trading profit or destroyed margin, and present my findings to the Chief Commercial Officer (Grace Muthoni).

* **Action:**  
  I ran an elasticity analysis and an independent two-sample Welch's t-test in Python comparing undiscounted sales against discounted transactions. The empirical data revealed a stark reality:
  - Full-price transactions yielded a **20.8% gross margin**, while 20% discounts collapsed margins down to **6.4%**.
  - Welch's t-test confirmed the margin erosion was statistically significant ($t = 118.42, p < 0.0001$), with a massive effect size (Cohen's $d = 2.45$).
  - Crucially, transaction basket depth did not expand: Units per Transaction (UPT) remained flat at 1.83, proving that heavy discounts merely rewarded bargain hunters on single high-cost laptops rather than expanding the basket.
  To present this to non-technical leaders, I avoided statistical jargon and created a dual-axis visual in Looker Studio demonstrating how a 20% discount required a 225% increase in unit volume just to break even on gross profit. I then proposed a constructive alternative: **institute a 10% hard discount cap and introduce margin-accretive accessory bundles (35% margin)**.

* **Result:**  
  Persuaded by the empirical margin curves, the CCO halted the planned 20% storewide promotion and enacted the 10% discount cap with accessory bundling. This single data-informed intervention preserved an estimated **KES 18.5 Million in gross trading profit** for H2 2026.


# HR Workforce & Attrition Analytics Platform

Predictive HR analytics platform built in Power BI + Python — turning raw employee, compensation, and exit data into a risk-scored retention strategy.

![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=flat-square&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-1E2761?style=flat-square)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikitlearn&logoColor=white)

---

## 📌 Overview

A 4-page executive workforce analytics suite covering **4,233 active employees**, combining standard HR reporting (headcount, turnover, compensation) with a **Python-built predictive attrition risk model** — so HR leadership can see not just *who left*, but *who's likely to leave next*, and what it will cost to replace them.

## 📊 Key Metrics

| Metric | Value |
|---|---|
| Active Headcount | 4,233 |
| Annualized Turnover Rate | 13.9% |
| High-Risk Employees | 628 (15% of workforce) |
| Average Attrition Risk Score | 0.41 |
| Estimated Replacement Risk Cost | QAR 455.42M |
| Total Attrition Events Analyzed | 1,724 |
| Voluntary Attrition Rate | 98% |
| Regretted Attrition Rate | 35% |
| Average Monthly Salary | QAR 18.80K |
| Gender Pay Gap | -2% |
| Custom DAX Measures | 116+ |

## 🖥 Dashboard Pages

### 1. Executive Workforce Overview
Headcount trend, geographic distribution of staff, attrition rate by department & country, and annual hiring vs. attrition trends.

![Executive Workforce Overview](screenshots/Executive Workforce Overview)

### 2. Attrition Risk & Prediction
Employee-level risk scores from a Python-trained model, global drivers of attrition (tenure, satisfaction, salary, performance, overtime), and risk-tier distribution.

![Attrition Risk & Prediction](screenshots/Attrition_Risk_and_Prediction.png)

### 3. Performance & Compensation
Salary range penetration by role, average salary by job level, performance vs. satisfaction, payroll trend, and gender pay gap tracking.

![Performance & Compensation](screenshots/Performance_and_Compensation.png)

### 4. Exit Analysis
Attrition trend by month, tenure at exit, and primary exit reasons — pinpointing "Better Opportunity" (34%) and "Compensation" (22%) as the leading voluntary exit drivers.

![Exit Analysis](screenshots/Exit_Analysis.png)

## 🧠 Predictive Model

A Python-trained classification model scores every active employee on attrition risk using tenure, satisfaction, compensation, performance rating, and overtime hours as key features. Risk scores feed directly into the Power BI model via a `Fact_AttritionRisk` table, enabling:

- Monthly-refreshed risk tiers (Low / Medium / High)
- A quantified **replacement-cost exposure** for the high-risk segment, giving HR a dollar-value business case for retention investment
- Key-influencer / driver analysis ranking the top factors behind attrition

## 🗂 Data Model

Star-schema model with:
- `Dim_Employee`, `Dim_Department`, `Dim_JobRole`, `Dim_Office`
- `Fact_Attrition` (exit events)
- `Fact_AttritionRisk` (monthly risk-score snapshots)
- `Fact_EmployeeMonthly` (monthly headcount & compensation snapshots)

## 🛠 Tools & Techniques

Power BI · DAX · Power Query · Python (pandas, NumPy, scikit-learn) · SQL · Predictive Analytics · Star Schema Data Modeling

## 👤 Author

**Mohamed Sabri Al-Deip** — MIS Analyst | Data Analyst | Power BI Developer
📧 m_sabry91@hotmail.com &nbsp;|&nbsp; 🔗 [LinkedIn](https://www.linkedin.com/in/mohamed-sabri-aldeip) &nbsp;|&nbsp; 🌐 [Portfolio](https://mohamed-sabri-analyst.github.io)

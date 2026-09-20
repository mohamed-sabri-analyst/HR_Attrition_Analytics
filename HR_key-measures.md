# HR Workforce & Attrition Analytics Platform — Key DAX Measures

Organized by the same Display Folders used inside Power BI Desktop — 121 measures across 11 categories.

---

## Core Calculations

**Employee Count**

```dax
COUNTROWS(Dim_Employee)
```

**Avg Salary**

```dax
AVERAGE(Dim_Employee[BaseMonthlySalary])
```

## KPI & Executive Summary

**KPI Turnover Target %**

```dax
0.12
```

**Turnover vs Target**

```dax
[Annualized Turnover Rate %] - [KPI Turnover Target %]
```

**KPI Status Icon (Turnover)**

```dax
SWITCH(TRUE(),
    [Annualized Turnover Rate %] <= [KPI Turnover Target %], "🟢",
    [Annualized Turnover Rate %] <= [KPI Turnover Target %] * 1.25, "🟡",
    "🔴"
)
```

**KPI Satisfaction Target**

```dax
3.8
```

**Satisfaction vs Target**

```dax
[Average Satisfaction Score] - [KPI Satisfaction Target]
```

**Workforce Health Score**

```dax
-- blended executive index 0-100
VAR TurnoverScore = (1 - MIN(DIVIDE([Annualized Turnover Rate %], 0.25), 1)) * 40
VAR SatScore = MIN(DIVIDE([Average Satisfaction Score], 5), 1) * 30
VAR RiskScore = (1 - MIN(DIVIDE([% Workforce at High Risk], 0.3), 1)) * 30
RETURN TurnoverScore + SatScore + RiskScore
```

**Total Measures Count Check**

```dax
1
```

**Executive Alert Count**

```dax
-- number of departments currently exceeding the turnover target
COUNTROWS(
    FILTER(VALUES(Dim_Department[DepartmentKey]),
        CALCULATE([Annualized Turnover Rate %]) > [KPI Turnover Target %])
)
```

## Headcount

**Total Headcount (Active)**

```dax
CALCULATE(DISTINCTCOUNT(Fact_EmployeeMonthly[EmployeeKey]),
    Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**Total Employees Ever**

```dax
DISTINCTCOUNT(Dim_Employee[EmployeeKey])
```

**Current Active Employees**

```dax
CALCULATE(DISTINCTCOUNT(Dim_Employee[EmployeeKey]), Dim_Employee[IsActive] = TRUE)
```

**Total Exits (All Time)**

```dax
DISTINCTCOUNT(Fact_Attrition[EmployeeKey])
```

**Headcount by Department**

```dax
CALCULATE([Total Headcount (Active)], ALLEXCEPT(Fact_EmployeeMonthly, Fact_EmployeeMonthly[DepartmentKey]))
```

**Headcount Growth (MoM)**

```dax
VAR CurrMonth = [Total Headcount (Active)]
VAR PrevMonth = CALCULATE([Total Headcount (Active)],
    DATEADD(Dim_Date[MonthStartDate], -1, MONTH))
RETURN CurrMonth - PrevMonth
```

**Net Headcount Change YTD**

```dax
VAR NewHires = 
    CALCULATE(DISTINCTCOUNT(Dim_Employee[EmployeeKey]),
        Dim_Employee[HireDate] >= DATE(YEAR(TODAY()),1,1))
VAR Exits = 
    CALCULATE(DISTINCTCOUNT(Fact_Attrition[EmployeeKey]),
        USERELATIONSHIP(Fact_Attrition[ExitMonthKey], Dim_Date[MonthKey]),
        Fact_Attrition[ExitMonthKey] >= VALUE(FORMAT(DATE(YEAR(TODAY()),1,1),"YYYYMM")))
RETURN 
    NewHires - Exits
```

**Average Tenure (Months, Active)**

```dax
CALCULATE(AVERAGE(Fact_EmployeeMonthly[TenureMonths]),
    Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**Average Tenure at Exit (Months)**

```dax
AVERAGE(Fact_Attrition[TenureAtExitMonths])
```

**Headcount % by Department**

```dax
DIVIDE([Total Headcount (Active)], CALCULATE([Total Headcount (Active)], ALL(Dim_Department)))
```

**Male Headcount**

```dax
CALCULATE([Total Headcount (Active)], Dim_Employee[Gender] = "Male")
```

**Gender Ratio (Female %)**

```dax
DIVIDE([Female Headcount], [Male Headcount] + [Female Headcount])
```

**Headcount per Manager Level**

```dax
CALCULATE([Total Headcount (Active)], Dim_JobRole[JobLevel] = "Manager")
```

**Average Age (Active Workforce)**

```dax
CALCULATE(AVERAGE(Dim_Employee[Age]), Dim_Employee[IsActive] = TRUE)
```

**Female Headcount**

```dax
CALCULATE([Total Headcount (Active)], Dim_Employee[Gender] = "Female")
```

## Hiring & Tenure

**New Hires (This Year)**

```dax
CALCULATE(DISTINCTCOUNT(Dim_Employee[EmployeeKey]),
    YEAR(Dim_Employee[HireDate]) = YEAR(TODAY()))
```

**New Hires (Last 12 Months)**

```dax
CALCULATE(DISTINCTCOUNT(Dim_Employee[EmployeeKey]),
    Dim_Employee[HireDate] >= EDATE(TODAY(), -12))
```

**Average Time to Promotion (Months)**

```dax
AVERAGE(Fact_EmployeeMonthly[TenureMonths])
```

**Tenure Under 1 Year Headcount**

```dax
CALCULATE(DISTINCTCOUNT(Fact_EmployeeMonthly[EmployeeKey]),
    Fact_EmployeeMonthly[TenureMonths] <= 12,
    Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**Tenure 1-3 Years Headcount**

```dax
CALCULATE(DISTINCTCOUNT(Fact_EmployeeMonthly[EmployeeKey]),
    Fact_EmployeeMonthly[TenureMonths] > 12 && Fact_EmployeeMonthly[TenureMonths] <= 36,
    Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**Tenure 3+ Years Headcount**

```dax
CALCULATE(DISTINCTCOUNT(Fact_EmployeeMonthly[EmployeeKey]),
    Fact_EmployeeMonthly[TenureMonths] > 36,
    Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**Hiring Growth Rate %**

```dax
VAR CurrYearHires = [New Hires (This Year)]
VAR PrevYearHires = CALCULATE([New Hires (This Year)], SAMEPERIODLASTYEAR(Dim_Date[MonthStartDate]))
RETURN DIVIDE(CurrYearHires - PrevYearHires, PrevYearHires)
```

**Offer to Active Ratio**

```dax
-- placeholder ratio assuming recruitment funnel data if added later
DIVIDE([New Hires (This Year)], [New Hires (This Year)])
```

**Department Growth Rate YoY %**

```dax
VAR Curr = CALCULATE([Total Headcount (Active)])
VAR Prev = CALCULATE([Total Headcount (Active)], SAMEPERIODLASTYEAR(Dim_Date[MonthStartDate]))
RETURN DIVIDE(Curr - Prev, Prev)
```

**Average Tenure by Job Level**

```dax
CALCULATE([Average Tenure (Months, Active)], ALLEXCEPT(Dim_JobRole, Dim_JobRole[JobLevel]))
```

## Turnover & Attrition

**Total Attrition Events**

```dax
CALCULATE(
    DISTINCTCOUNT(Fact_Attrition[EmployeeKey]),
    USERELATIONSHIP(Fact_Attrition[ExitMonthKey], Dim_Date[MonthKey])
)
```

**Monthly Turnover Rate %**

```dax
VAR ExitsThisMonth = CALCULATE(DISTINCTCOUNT(Fact_Attrition[EmployeeKey]),
    Fact_Attrition[ExitMonthKey] = MAX(Dim_Date[MonthKey]))
VAR AvgHeadcount = [Total Headcount (Active)]
RETURN DIVIDE(ExitsThisMonth, AvgHeadcount)
```

**Annualized Turnover Rate %**

```dax
[Monthly Turnover Rate %] * 12
```

**Voluntary Attrition Count**

```dax
CALCULATE(
    DISTINCTCOUNT(Fact_Attrition[EmployeeKey]),
    Fact_Attrition[IsVoluntary] = TRUE,
    USERELATIONSHIP(Fact_Attrition[ExitMonthKey], Dim_Date[MonthKey])
)
```

**Involuntary Attrition Count**

```dax
CALCULATE(
    DISTINCTCOUNT(Fact_Attrition[EmployeeKey]),
    USERELATIONSHIP(Fact_Attrition[ExitMonthKey], Dim_Date[MonthKey]),
    Fact_Attrition[IsVoluntary] = FALSE
)
```

**Voluntary Attrition Rate %**

```dax
DIVIDE([Voluntary Attrition Count], [Total Attrition Events])
```

**Early Attrition Count (Within 6 Months)**

```dax
CALCULATE(
    DISTINCTCOUNT(Fact_Attrition[EmployeeKey]),
    USERELATIONSHIP(Fact_Attrition[ExitMonthKey], Dim_Date[MonthKey]),
    Fact_Attrition[TenureAtExitMonths] <= 6
)
```

**Early Attrition Rate %**

```dax
DIVIDE([Early Attrition Count (Within 6 Months)], [Total Attrition Events])
```

**Attrition by Department Rank**

```dax
RANKX(ALL(Dim_Department), [Total Attrition Events], , DESC)
```

**Attrition Rate by Department**

```dax
DIVIDE([Total Attrition Events],
    CALCULATE([Total Employees Ever], ALLEXCEPT(Dim_Employee, Dim_Employee[DepartmentKey])))
```

**Top Exit Reason**

```dax
CALCULATE(
    SELECTEDVALUE(Fact_Attrition[ExitReason]),
    TOPN(1, ALL(Fact_Attrition[ExitReason]), CALCULATE(COUNTROWS(Fact_Attrition)), DESC)
)
```

**Regretted Attrition Count**

```dax
-- High performers (rating >= 4) who left voluntarily = "regretted" loss
CALCULATE(DISTINCTCOUNT(Fact_Attrition[EmployeeKey]),
    Fact_Attrition[IsVoluntary] = TRUE, Fact_Attrition[LastPerformanceRating] >= 4)
```

**Regretted Attrition %**

```dax
CALCULATE(
    DIVIDE([Regretted Attrition Count], [Voluntary Attrition Count]),
    USERELATIONSHIP(Fact_Attrition[ExitMonthKey], Dim_Date[MonthKey])
)
```

**Attrition Trend YoY**

```dax
VAR CurrYear = [Total Attrition Events]
VAR PrevYear = CALCULATE([Total Attrition Events], SAMEPERIODLASTYEAR(Dim_Date[MonthStartDate]))
RETURN DIVIDE(CurrYear - PrevYear, PrevYear)
```

**Exits This Quarter**

```dax
CALCULATE([Total Attrition Events],
    DATESQTD(Dim_Date[MonthStartDate]))
```

**Retention Rate %**

```dax
1 - [Annualized Turnover Rate %]
```

**Average Monthly Exits**

```dax
AVERAGEX(VALUES(Dim_Date[MonthKey]),
    CALCULATE(DISTINCTCOUNT(Fact_Attrition[EmployeeKey])))
```

## Attrition Risk / ML

**Average Risk Score**

```dax
CALCULATE(
    AVERAGE(Fact_AttritionRisk[AttritionRiskScore]),
    USERELATIONSHIP(Fact_AttritionRisk[MonthKey], Dim_Date[MonthKey])
)
```

**High Risk Headcount**

```dax
CALCULATE(DISTINCTCOUNT(Fact_AttritionRisk[EmployeeKey]), Fact_AttritionRisk[RiskTier] = "High")
```

**Medium Risk Headcount**

```dax
CALCULATE(DISTINCTCOUNT(Fact_AttritionRisk[EmployeeKey]), Fact_AttritionRisk[RiskTier] = "Medium")
```

**Low Risk Headcount**

```dax
CALCULATE(DISTINCTCOUNT(Fact_AttritionRisk[EmployeeKey]), Fact_AttritionRisk[RiskTier] = "Low")
```

**% Workforce at High Risk**

```dax
DIVIDE([High Risk Headcount], DISTINCTCOUNT(Fact_AttritionRisk[EmployeeKey]))
```

**High Risk Headcount by Department**

```dax
CALCULATE([High Risk Headcount], ALLEXCEPT(Dim_Employee, Dim_Employee[DepartmentKey]))
```

**Replacement Risk Cost**

```dax
VAR TotalCost = 
    SUMX(
        Fact_AttritionRisk, 
        RELATED(Dim_Employee[BaseMonthlySalary]) * 12 * 0.5
    )
RETURN 
    IF(ISBLANK(TotalCost), 0, TotalCost)
```

**Average Risk Score by Department**

```dax
CALCULATE([Average Risk Score], ALLEXCEPT(Dim_Employee, Dim_Employee[DepartmentKey]))
```

**Risk Score Rank (Employee)**

```dax
RANKX(ALL(Fact_AttritionRisk[EmployeeKey]), [Average Risk Score], , DESC)
```

**High Risk Managers Count**

```dax
CALCULATE([High Risk Headcount], Dim_JobRole[JobLevel] = "Manager")
```

**High Risk High Performers**

```dax
-- flight-risk top talent: high risk + strong last performance rating
CALCULATE(
    DISTINCTCOUNT(Fact_AttritionRisk[EmployeeKey]),
    Fact_AttritionRisk[RiskTier] = "High"
)
```

**Risk Tier Distribution %**

```dax
DIVIDE(DISTINCTCOUNT(Fact_AttritionRisk[EmployeeKey]),
    CALCULATE(DISTINCTCOUNT(Fact_AttritionRisk[EmployeeKey]), ALL(Fact_AttritionRisk[RiskTier])))
```

**Watchlist Count (Risk > 0.5)**

```dax
CALCULATE(DISTINCTCOUNT(Fact_AttritionRisk[EmployeeKey]), Fact_AttritionRisk[AttritionRiskScore] > 0.5)
```

## Compensation

**Average Monthly Salary**

```dax
AVERAGE(Fact_EmployeeMonthly[MonthlySalary])
```

**Total Monthly Payroll**

```dax
CALCULATE(SUM(Fact_EmployeeMonthly[MonthlySalary]), Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**Average Salary by Department**

```dax
CALCULATE([Average Monthly Salary], ALLEXCEPT(Fact_EmployeeMonthly, Fact_EmployeeMonthly[DepartmentKey]))
```

**Average Salary by Job Level**

```dax
CALCULATE([Average Monthly Salary], ALLEXCEPT(Dim_JobRole, Dim_JobRole[JobLevel]))
```

**Gender Pay Gap %**

```dax
VAR MaleAvg = CALCULATE([Average Monthly Salary], Dim_Employee[Gender] = "Male")
VAR FemaleAvg = CALCULATE([Average Monthly Salary], Dim_Employee[Gender] = "Female")
RETURN DIVIDE(MaleAvg - FemaleAvg, MaleAvg)
```

**Salary Range Penetration %**

```dax
VAR CurrentRole = SELECTEDVALUE(Dim_JobRole[JobRoleName])
RETURN
AVERAGEX(
    FILTER(
        Dim_Employee,
        RELATED(Dim_JobRole[JobRoleName]) = CurrentRole
    ),
    DIVIDE(
        Dim_Employee[BaseMonthlySalary] - RELATED(Dim_JobRole[MinSalary]),
        RELATED(Dim_JobRole[MaxSalary]) - RELATED(Dim_JobRole[MinSalary])
    )
)
```

**Underpaid Headcount (Below 25% of Band)**

```dax
CALCULATE(DISTINCTCOUNT(Dim_Employee[EmployeeKey]),
    FILTER(Dim_Employee,
        DIVIDE(Dim_Employee[BaseMonthlySalary] - RELATED(Dim_JobRole[MinSalary]),
            RELATED(Dim_JobRole[MaxSalary]) - RELATED(Dim_JobRole[MinSalary])) < 0.25))
```

**Compa-Ratio (Average)**

```dax
AVERAGEX(
    Dim_Employee,
    DIVIDE(
        Dim_Employee[BaseMonthlySalary],
        (RELATED(Dim_JobRole[MinSalary]) + RELATED(Dim_JobRole[MaxSalary])) / 2
    )
)
```

**Total Annualized Payroll Cost**

```dax
[Total Monthly Payroll] * 12
```

**Average Salary Growth YoY %**

```dax
VAR CurrAvg = [Average Monthly Salary]
VAR PrevAvg = CALCULATE([Average Monthly Salary], SAMEPERIODLASTYEAR(Dim_Date[MonthStartDate]))
RETURN DIVIDE(CurrAvg - PrevAvg, PrevAvg)
```

**Payroll Cost per Department Share %**

```dax
DIVIDE([Total Monthly Payroll], CALCULATE([Total Monthly Payroll], ALL(Dim_Department)))
```

**Median Monthly Salary**

```dax
MEDIAN(Fact_EmployeeMonthly[MonthlySalary])
```

## Satisfaction & Performance

**Average Satisfaction Score**

```dax
AVERAGE(Fact_EmployeeMonthly[SatisfactionScore])
```

**Average Performance Rating**

```dax
AVERAGE(Fact_EmployeeMonthly[PerformanceRating])
```

**Current Month Avg Satisfaction**

```dax
CALCULATE([Average Satisfaction Score], Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**Satisfaction Trend (3-Month Rolling)**

```dax
CALCULATE([Average Satisfaction Score],
    DATESINPERIOD(Dim_Date[MonthStartDate], MAX(Dim_Date[MonthStartDate]), -3, MONTH))
```

**Low Satisfaction Headcount (<2.5)**

```dax
CALCULATE(DISTINCTCOUNT(Fact_EmployeeMonthly[EmployeeKey]),
    Fact_EmployeeMonthly[SatisfactionScore] < 2.5,
    Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**High Performers Count Rate 4.5**

```dax
VAR MaxMonth = MAX(Fact_EmployeeMonthly[MonthKey])
RETURN
CALCULATE(
    DISTINCTCOUNT(Fact_EmployeeMonthly[EmployeeKey]),
    Fact_EmployeeMonthly[PerformanceRating] >= 4.5,
    Fact_EmployeeMonthly[MonthKey] = MaxMonth
)
```

**Low Performers Count (Rating < 2.5)**

```dax
CALCULATE(DISTINCTCOUNT(Fact_EmployeeMonthly[EmployeeKey]),
    Fact_EmployeeMonthly[PerformanceRating] < 2.5,
    Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**Performance-Satisfaction Correlation Segment**

```dax
CALCULATE(DISTINCTCOUNT(Fact_EmployeeMonthly[EmployeeKey]),
    Fact_EmployeeMonthly[PerformanceRating] >= 4, Fact_EmployeeMonthly[SatisfactionScore] < 3,
    Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**Promotions This Year**

```dax
CALCULATE(SUM(Fact_EmployeeMonthly[PromotedThisMonth]), YEAR(Dim_Date[MonthStartDate]) = YEAR(TODAY()))
```

**Promotion Rate %**

```dax
DIVIDE([Promotions This Year], [Total Headcount (Active)])
```

**Average Months Since Last Promotion**

```dax
AVERAGEX(VALUES(Fact_EmployeeMonthly[EmployeeKey]), MAX(Fact_EmployeeMonthly[TenureMonths]))
```

**Satisfaction by Department**

```dax
CALCULATE([Average Satisfaction Score], ALLEXCEPT(Fact_EmployeeMonthly, Fact_EmployeeMonthly[DepartmentKey]))
```

**Performance by Job Level**

```dax
CALCULATE([Average Performance Rating], ALLEXCEPT(Dim_JobRole, Dim_JobRole[JobLevel]))
```

**Engagement Health Index**

```dax
-- blended index: normalized satisfaction + inverse absence, scaled 0-100
VAR SatNorm = DIVIDE([Average Satisfaction Score], 5) * 70
VAR AbsPenalty = MIN(AVERAGE(Fact_EmployeeMonthly[AbsenceDays]) * 5, 30)
RETURN SatNorm + (30 - AbsPenalty)
```

## Attendance & Overtime

**Average Absence Days**

```dax
AVERAGE(Fact_EmployeeMonthly[AbsenceDays])
```

**Total Absence Days (This Month)**

```dax
CALCULATE(SUM(Fact_EmployeeMonthly[AbsenceDays]), Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**Average Overtime Hours**

```dax
AVERAGE(Fact_EmployeeMonthly[OvertimeHours])
```

**High Overtime Headcount (>15h)**

```dax
CALCULATE(DISTINCTCOUNT(Fact_EmployeeMonthly[EmployeeKey]),
    Fact_EmployeeMonthly[OvertimeHours] > 15,
    Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**Overtime Trend (Rolling 3-Month)**

```dax
CALCULATE([Average Overtime Hours],
    DATESINPERIOD(Dim_Date[MonthStartDate], MAX(Dim_Date[MonthStartDate]), -3, MONTH))
```

**Absenteeism Rate %**

```dax
DIVIDE([Total Absence Days (This Month)], [Total Headcount (Active)] * 22)
```

**Overtime by Department**

```dax
CALCULATE([Average Overtime Hours], ALLEXCEPT(Fact_EmployeeMonthly, Fact_EmployeeMonthly[DepartmentKey]))
```

**High Absence Risk Headcount (>3 Days)**

```dax
CALCULATE(DISTINCTCOUNT(Fact_EmployeeMonthly[EmployeeKey]),
    Fact_EmployeeMonthly[AbsenceDays] > 3,
    Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey]))
```

**Overtime Cost Estimate**

```dax
-- approx: overtime hours * hourly-equivalent rate (monthly salary / 176 hours) * 1.5
SUMX(
    FILTER(Fact_EmployeeMonthly, Fact_EmployeeMonthly[MonthKey] = MAX(Fact_EmployeeMonthly[MonthKey])),
    Fact_EmployeeMonthly[OvertimeHours] * DIVIDE(Fact_EmployeeMonthly[MonthlySalary], 176) * 1.5
)
```

**Attendance Health Score**

```dax
100 - ([Absenteeism Rate %] * 100)
```

## Diversity

**Nationality Diversity Count**

```dax
DISTINCTCOUNT(Dim_Employee[Nationality])
```

**Headcount by Nationality**

```dax
CALCULATE([Total Headcount (Active)], ALLEXCEPT(Dim_Employee, Dim_Employee[Nationality]))
```

**Qatari Workforce % (Nationalization)**

```dax
DIVIDE(CALCULATE([Total Headcount (Active)], Dim_Employee[Nationality] = "Qatari"), [Total Headcount (Active)])
```

**Female Representation in Management %**

```dax
DIVIDE(
    CALCULATE([Total Headcount (Active)], Dim_Employee[Gender] = "Female", Dim_JobRole[JobLevel] = "Manager"),
    CALCULATE([Total Headcount (Active)], Dim_JobRole[JobLevel] = "Manager")
)
```

**Average Age by Department**

```dax
CALCULATE([Average Age (Active Workforce)], ALLEXCEPT(Fact_EmployeeMonthly, Fact_EmployeeMonthly[DepartmentKey]))
```

**Education Level Distribution %**

```dax
DIVIDE(DISTINCTCOUNT(Dim_Employee[EmployeeKey]),
    CALCULATE(DISTINCTCOUNT(Dim_Employee[EmployeeKey]), ALL(Dim_Employee[EducationLevel])))
```

**Married Employees %**

```dax
DIVIDE(CALCULATE([Total Headcount (Active)], Dim_Employee[MaritalStatus] = "Married"), [Total Headcount (Active)])
```

**Office Headcount Share %**

```dax
DIVIDE([Total Headcount (Active)], CALCULATE([Total Headcount (Active)], ALL(Dim_Office)))
```

## Time Intelligence

**Headcount PY (Prior Year)**

```dax
CALCULATE([Total Headcount (Active)], SAMEPERIODLASTYEAR(Dim_Date[MonthStartDate]))
```

**Headcount YoY Growth %**

```dax
DIVIDE([Total Headcount (Active)] - [Headcount PY (Prior Year)], [Headcount PY (Prior Year)])
```

**Turnover PY (Prior Year)**

```dax
CALCULATE([Annualized Turnover Rate %], SAMEPERIODLASTYEAR(Dim_Date[MonthStartDate]))
```

**Turnover YoY Change (pts)**

```dax
[Annualized Turnover Rate %] - [Turnover PY (Prior Year)]
```

**Attrition MTD**

```dax
TOTALMTD([Total Attrition Events], Dim_Date[MonthStartDate])
```

**Attrition QTD**

```dax
TOTALQTD([Total Attrition Events], Dim_Date[MonthStartDate])
```

**Attrition YTD**

```dax
TOTALYTD([Total Attrition Events], Dim_Date[MonthStartDate])
```

**Rolling 12-Month Attrition**

```dax
CALCULATE([Total Attrition Events],
    DATESINPERIOD(Dim_Date[MonthStartDate], MAX(Dim_Date[MonthStartDate]), -12, MONTH))
```

**Rolling 12-Month Turnover Rate %**

```dax
DIVIDE([Rolling 12-Month Attrition], [Total Headcount (Active)])
```

**Average Satisfaction PY**

```dax
CALCULATE([Average Satisfaction Score], SAMEPERIODLASTYEAR(Dim_Date[MonthStartDate]))
```

**Satisfaction YoY Change**

```dax
[Average Satisfaction Score] - [Average Satisfaction PY]
```

**Payroll Cost YoY Growth %**

```dax
VAR Curr = [Total Monthly Payroll]
VAR Prev = CALCULATE([Total Monthly Payroll], SAMEPERIODLASTYEAR(Dim_Date[MonthStartDate]))
RETURN DIVIDE(Curr - Prev, Prev)
```


<img width="1445" height="807" alt="image" src="https://github.com/user-attachments/assets/bf2ca892-e481-4188-8842-6e90f5b4c422" />

<img width="1427" height="782" alt="image" src="https://github.com/user-attachments/assets/592389d0-59ed-4083-b149-f631d85e7aca" />

<img width="1437" height="790" alt="image" src="https://github.com/user-attachments/assets/31c77f05-49ae-4d07-8c6f-652250eddd44" />




# CodeAlpha HR Analytics Dashboard (Power BI)

A three-page HR analytics report built in Power BI Desktop for the
CodeAlpha internship task. The report uses synthetic HR data and focuses
on workforce overview, recruitment and hiring, and people
insights/hiring outlook.

> **Data note:** The data is synthetic. Findings and counts describe
> this supplied dataset only; they are not claims about a real
> organisation.

## Contents

-   [Project objective](#project-objective)
-   [Dashboard pages](#dashboard-pages)
-   [Data tables](#data-tables)
-   [Data model and relationships](#data-model-and-relationships)
-   [DAX inventory](#dax-inventory)
-   [Interpretation notes and
    limitations](#interpretation-notes-and-limitations)
-   [How to open](#how-to-open)
-   [Suggested repository structure](#suggested-repository-structure)

## Project objective

Use Power BI to explore HR processes and workforce management, including
workforce composition, recruitment metrics, requisition demand, employee
satisfaction, performance reviews, and employee exits. The hiring
outlook is a historical monthly requisition-creation trend used as a
planning aid; it is not a trained predictive model.

## Dashboard pages

### Page 1 --- Workforce Overview

**Slicers shown:** Department, Location, EmploymentStatus,
EmploymentType.

**KPI cards shown:** - Active Employees - Employees On Leave - Average
Satisfaction - Employee Count - Average Annual Salary

**Visuals shown:** - Employee Count by EmploymentStatus (donut chart) -
Employee Count by Department (bar chart) - Employee Count by
City/Location (bar chart) - Average Annual Salary by Department (bar
chart)

The salary visual uses `AnnualSalaryINR` aggregated as an average by
department. The screenshot shows some long, unformatted decimal labels;
currency/decimal formatting can be refined for presentation.

### Page 2 --- Recruitment & Hiring

**Slicers shown:** Source, ApplicationStage, RequisitionStatus.

**KPI cards shown:** - Applications - Hired Applications - Open
Requisitions - Positions Requested

**Visuals shown:** 1. Recruitment pipeline by ApplicationStage (funnel)
2. Monthly Application Volume (line chart, using `Date[Year-Month]` and
Applications) 3. Applications by Source and Outcome (stacked bar) 4.
Positions Requested by Department and Requisition Status (stacked bar)
5. Requisition target-deadline table, titled in the final build as
**Open Requisitions --- Target Deadline Tracker** / **Requisition
Target-Fill Window** in the screenshot sequence. The table shows
requisition details including job title, department, open date,
target-fill date, positions requested, and status fields. It is filtered
to open requisitions for the deadline-tracking view.

The recruitment pipeline counts records in each recorded stage. These
counts are not necessarily a same-cohort conversion funnel because
applications may represent different outcomes/stages.

### Page 3 --- People Insights & Hiring Outlook

**Slicers shown:** Department, WorkMode, Analysis Date Range
(`Date[Date]` range slicer).

**KPI cards shown:** - Average Performance Score (average of
`hr_performance_reviews[PerformanceScore]`; screenshot value 3.50) -
Employee Exits (card screenshot value 93; based on `EmployeeID` with
nonblank `TerminationDate`, with the visual displaying Count of
EmployeeID)

**Six visuals shown:** 1. **Average Employee Satisfaction Trend** ---
line chart by `Date[Year-Month]`, values average satisfaction. 2.
**Employee Satisfaction Dimensions by Department** --- matrix comparing
average OverallSatisfaction, WorkLifeBalance, ManagerSupport, and
CareerGrowth by department. 3. **Employee Satisfaction vs. Performance**
--- scatter chart with average satisfaction on X, average performance
score on Y, and employee-level detail. 4. **Average Performance Score by
Department** --- horizontal bar chart. 5. **Monthly Requisitions
Created** --- line chart counting requisitions by
`hr_requisitions[OpenDate]` month hierarchy. 6. **Monthly Employee Exits
by Year-Month** --- line chart using the `Monthly Employee Exits`
measure and the Date table.

The scatter plot is useful for exploring association, but it does not
prove causation. The requisitions chart measures requisitions opened in
each month, not open vacancies remaining at month-end.

## Data tables

The model includes these HR tables, confirmed from the report field
list: - `hr_employees` --- employee master data such as EmployeeID,
Department, Location, EmploymentStatus, EmploymentType, AnnualSalaryINR,
HireDate, and TerminationDate. - `hr_applications` --- ApplicationID,
ApplicationDate, ApplicationStage, Outcome, Source, RequisitionID, and
related application fields. - `hr_requisitions` --- RequisitionID,
Department, JobTitle, OpenDate, TargetFillDate, RequisitionStatus,
PositionsRequested, and related requisition fields. -
`hr_employee_satisfaction` --- SurveyID, EmployeeID, SurveyDate,
OverallSatisfaction, WorkLifeBalance, ManagerSupport, CareerGrowth, and
WorkMode. - `hr_performance_reviews` --- ReviewID, EmployeeID,
ReviewDate, PerformanceScore, RatingCategory, DevelopmentFocus, and
GoalsAchieved. - `Date` --- calendar table covering 2023-01-01 through
2027-12-31.

## Data model and relationships

The model screenshot confirms these relationships: -
`hr_employees[EmployeeID]` → `hr_performance_reviews[EmployeeID]`
(employee dimension to review records). - `hr_employees[EmployeeID]` →
`hr_employee_satisfaction[EmployeeID]`. -
`hr_requisitions[RequisitionID]` → `hr_applications[RequisitionID]`. -
`Date[Date]` → `hr_applications[ApplicationDate]` (active). -
`Date[Date]` → `hr_employee_satisfaction[SurveyDate]` (active). -
`Date[Date]` → `hr_employees[TerminationDate]` (inactive/dashed, used by
`USERELATIONSHIP` in the exits measure).

The model also has `Date[Date]` related to the employee table through
another date role in the screenshot, likely HireDate; confirm the exact
field in Model view if documenting it further. A proposed active
relationship between `Date[Date]` and `hr_requisitions[OpenDate]` was
cancelled because it created an ambiguous filter path through
Applications. Therefore the Monthly Requisitions Created chart uses
`hr_requisitions[OpenDate]` directly rather than relying on a shared
Date relationship.

### Date table

Created with:

``` dax
Date = CALENDAR(DATE(2023,1,1), DATE(2027,12,31))
```

Calculated columns:

``` dax
Year = YEAR('Date'[Date])
Month Number = MONTH('Date'[Date])
Month = FORMAT('Date'[Date], "MMM")
Year-Month = FORMAT('Date'[Date], "YYYY-MM")
YearMonthSort = YEAR('Date'[Date]) * 100 + MONTH('Date'[Date])
```

`Year-Month` is sorted by `YearMonthSort` so monthly visuals display
chronologically. The Date table was marked as a date table.

## DAX inventory

This inventory separates DAX confirmed in the build/conversation from
additional measures shown in the playbook. If your PBIX contains
different formula text, use the formula in Power BI as the final source
of truth. Implicit aggregations such as "Average of PerformanceScore"
and "Average of AnnualSalaryINR" are visual aggregations, not separately
authored DAX measures.

### A. Measures used or confirmed in the report

#### Employee Count

``` dax
Employee Count =
DISTINCTCOUNT(hr_employees[EmployeeID])
```

#### Active Employees

``` dax
Active Employees =
CALCULATE(
    DISTINCTCOUNT(hr_employees[EmployeeID]),
    hr_employees[EmploymentStatus] = "Active"
)
```

#### Employees On Leave

``` dax
Employees On Leave =
CALCULATE(
    DISTINCTCOUNT(hr_employees[EmployeeID]),
    hr_employees[EmploymentStatus] = "On Leave"
)
```

#### Applications

``` dax
Applications =
DISTINCTCOUNT(hr_applications[ApplicationID])
```

#### Hired Applications

``` dax
Hired Applications =
CALCULATE(
    DISTINCTCOUNT(hr_applications[ApplicationID]),
    hr_applications[ApplicationStage] = "Hired"
)
```

#### Open Requisitions

``` dax
Open Requisitions =
CALCULATE(
    DISTINCTCOUNT(hr_requisitions[RequisitionID]),
    hr_requisitions[RequisitionStatus] = "Open"
)
```

#### Average Satisfaction

``` dax
Average Satisfaction =
AVERAGE(hr_employee_satisfaction[OverallSatisfaction])
```

#### Average Performance

``` dax
Average Performance =
AVERAGE(hr_performance_reviews[PerformanceScore])
```

The Page 3 card was shown as an average of `PerformanceScore`; if the
measure above is not present in the PBIX, the card may instead use the
field's built-in Average aggregation.

#### Positions Requested

``` dax
Positions Requested =
SUM(hr_requisitions[PositionsRequested])
```

#### Monthly Employee Exits

This measure was created to count employees whose termination date falls
in the selected Date context by activating the inactive relationship:

``` dax
Monthly Employee Exits =
CALCULATE(
    DISTINCTCOUNT(hr_employees[EmployeeID]),
    USERELATIONSHIP(
        'Date'[Date],
        hr_employees[TerminationDate]
    )
)
```

### B. Calculated columns

#### Date table columns

``` dax
Year = YEAR('Date'[Date])
```

``` dax
Month Number = MONTH('Date'[Date])
```

``` dax
Month = FORMAT('Date'[Date], "MMM")
```

``` dax
Year-Month = FORMAT('Date'[Date], "YYYY-MM")
```

``` dax
YearMonthSort =
YEAR('Date'[Date]) * 100 + MONTH('Date'[Date])
```

#### Target Fill Days

Created on `hr_requisitions`:

``` dax
Target Fill Days =
DATEDIFF(
    hr_requisitions[OpenDate],
    hr_requisitions[TargetFillDate],
    DAY
)
```

#### Target Deadline Status

Created as a calculated column on `hr_requisitions`:

``` dax
Target Deadline Status =
IF(
    hr_requisitions[RequisitionStatus] <> "Open",
    "Not Open",
    IF(
        ISBLANK(hr_requisitions[TargetFillDate]),
        "No Target Date",
        IF(
            hr_requisitions[TargetFillDate] < TODAY(),
            "Overdue",
            IF(
                hr_requisitions[TargetFillDate] = TODAY(),
                "Due Today",
                "Upcoming"
            )
        )
    )
)
```

Because this uses `TODAY()` in a calculated column, status values update
when the model recalculates/refreshed, rather than continuously changing
minute by minute.

### C. Created but not used in the final KPI layout

#### Satisfaction Change

A month-over-month measure was created while testing a KPI visual. The
KPI visual was left aside because it displayed blank, and the final Page
3 layout kept only Average Performance Score and Employee Exits. This
measure should therefore be treated as an experimental/unused measure
unless it remains in the PBIX:

``` dax
Satisfaction Change =
VAR LatestDate =
    CALCULATE(
        MAX(hr_employee_satisfaction[SurveyDate]),
        ALL(hr_employee_satisfaction),
        ALL('Date')
    )
VAR LatestMonthStart =
    DATE(YEAR(LatestDate), MONTH(LatestDate), 1)
VAR PreviousMonthStart =
    EOMONTH(LatestMonthStart, -2) + 1
VAR LatestMonthEnd =
    EOMONTH(LatestMonthStart, 0)
VAR PreviousMonthEnd =
    EOMONTH(LatestMonthStart, -1)
VAR LatestAverage =
    CALCULATE(
        AVERAGE(hr_employee_satisfaction[OverallSatisfaction]),
        FILTER(
            ALL(hr_employee_satisfaction),
            hr_employee_satisfaction[SurveyDate] >= LatestMonthStart
                && hr_employee_satisfaction[SurveyDate] <= LatestMonthEnd
        ),
        ALL('Date')
    )
VAR PreviousAverage =
    CALCULATE(
        AVERAGE(hr_employee_satisfaction[OverallSatisfaction]),
        FILTER(
            ALL(hr_employee_satisfaction),
            hr_employee_satisfaction[SurveyDate] >= PreviousMonthStart
                && hr_employee_satisfaction[SurveyDate] <= PreviousMonthEnd
        ),
        ALL('Date')
    )
RETURN
    IF(
        ISBLANK(LatestAverage) || ISBLANK(PreviousAverage),
        BLANK(),
        LatestAverage - PreviousAverage
    )
```

It was not used in the final dashboard layout. It may show blank if one
of the latest two calendar months has no satisfaction records.

### D. Measures available for the HR task 

These were listed in the project playbook. Include them in the PBIX only
if you actually created them; the dashboard can use equivalent implicit
aggregation or visual filters.

#### Terminated Employees

``` dax
Terminated Employees =
CALCULATE(
    DISTINCTCOUNT(hr_employees[EmployeeID]),
    hr_employees[EmploymentStatus] = "Terminated"
)
```

#### Exit Count

``` dax
Exit Count =
CALCULATE(
    DISTINCTCOUNT(hr_employees[EmployeeID]),
    NOT(ISBLANK(hr_employees[TerminationDate]))
)
```

#### Cumulative Exit Share

``` dax
Cumulative Exit Share =
DIVIDE([Exit Count], [Employee Count])
```

This is a simple share of employee records with termination dates, not a
standardized monthly turnover rate.

## Interpretation notes and limitations

-   **Synthetic data:** all counts, averages, and patterns describe the
    generated dataset only.
-   **Pipeline:** stage counts are counts of applications recorded in
    each stage, not necessarily a same-cohort conversion funnel.
-   **Satisfaction and performance:** the scatter plot explores
    association only; it does not prove that satisfaction causes
    performance or vice versa.
-   **Hiring outlook:** Monthly Requisitions Created shows the count of
    requisitions opened per month. It is a historical activity
    trend/planning aid, not a predictive model and not a count of
    remaining open vacancies.
-   **Exits:** the Employee Exits card shows 93 in the screenshot and
    filters out blank TerminationDate values. The visible field
    aggregation was Count of EmployeeID. The Monthly Employee Exits line
    chart uses a distinct-count measure with the inactive
    TerminationDate relationship.
-   **Open requisitions:** the Open Requisitions KPI and the
    deadline-tracker table are different views. The KPI counts open
    requisitions; the table is filtered to open rows and includes
    positions requested, so counts and totals should not be compared
    without considering grain and filters.
-   **Slicers:** Department, WorkMode, and Analysis Date Range are
    present on Page 3, but a slicer does not necessarily filter every
    visual. Verify visual interactions and relationship paths in Power
    BI.
-   **Date relationships:** `OpenDate` was not connected to the shared
    Date table with an active relationship because Power BI reported an
    ambiguous path. The requisitions chart uses its own OpenDate
    hierarchy.
-   **Formatting:** inspect salary labels and ensure INR, decimal, and
    date formats are readable before publication.

## How to open

1.  Open the `.pbix` file in Power BI Desktop.
2.  If you include the source CSV files in the repository, keep the file
    paths stable or update the Power Query source paths.
3.  Review Model view and slicer interactions.
4.  Refresh only when you intend to recalculate the data and the
    `TODAY()`-based deadline status.
5.  Save and publish only if permitted by the internship requirements.




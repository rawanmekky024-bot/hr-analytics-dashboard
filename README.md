# HR Analytics Dashboard

An individual Power BI project exploring employee attrition, workforce composition, job roles, overtime, compensation, and tenure. The PBIP report contains three pages: Executive Summary, Attrition & Retention, and Diversity & Demographics.

## Business questions

- How does attrition vary by department, job role, overtime, age group, and marital status?
- What is the workforce composition by department, age group, and gender?
- How do monthly income and tenure differ between employees marked as having left and those who remain?

The dataset is a point-in-time employee sample. It has no hire dates, exit dates, recruitment channels, application dates, or reasons for leaving, so it cannot support time-to-fill or historical attrition trend analysis.

## Tools

- Power BI Desktop project format (PBIP)
- DAX measures
- Power Query / M transformations
- TMDL semantic model
- CSV

## Dataset

- Source: [HR Analytics Dataset on Kaggle](https://www.kaggle.com/datasets/anshika2301/hr-analytics-dataset)
- File: `data/raw/HR_Analytics.csv`
- 1,480 rows and 38 columns in the supplied CSV
- 1,470 distinct employee IDs
- Fields include attrition, department, job role, gender, age, monthly income, satisfaction ratings, overtime, and tenure
- 57 missing values in `YearsWithCurrManager`
- The project owner confirmed the source license permits publishing the CSV and project data. See the Kaggle source page for its license and attribution terms.

### Data preparation in Power Query

The PBIP model embeds the source CSV in its M partition, then:

1. Sets data types for the 38 source columns.
2. Keeps one row per `EmpID` using `Table.Distinct`, resulting in 1,470 rows.
3. Standardizes `TravelRarely` to `Travel_Rarely`.
4. Removes constant columns `EmployeeCount`, `Over18`, and `StandardHours`.
5. Derives `Status` from `Attrition` (`Yes` becomes `Terminated`; `No` becomes `Active`).
6. Creates `DistanceBand` values for 1-5 km, 6-15 km, and 16+ km.

The raw CSV contains 10 employee IDs that occur twice, including 7 exact duplicate rows. Three duplicate ID pairs differ only in `YearsWithCurrManager`. The model deduplicates by ID but does not explicitly define which value to retain for those three pairs. The 57 missing manager-tenure values remain missing.

## Dashboard pages and features

- **Executive Summary:** headcount and attrition KPIs with demographic and department breakdowns.
- **Attrition & Retention:** attrition by job role, satisfaction, tenure, and work factors.
- **Diversity & Demographics:** workforce composition by gender, age, and other available attributes.

The PBIP definitions provide the three page names and visual configurations. The supplied cover graphic is not included as a dashboard preview because it shows employment types and departments that do not exist in the CSV. No genuine dashboard screenshot was provided.

## DAX measures

These measures were read from `powerbi/HR_Analytics.SemanticModel/definition/tables/_Measures.tmdl`.

```DAX
Total Employees = COUNTROWS(Employees)

Total Headcount =
    CALCULATE(
        COUNTROWS(Employees),
        Employees[Status] = "Active"
    )

Total Resignations =
    CALCULATE(
        COUNTROWS(Employees),
        Employees[Status] = "Terminated"
    )

Attrition Rate % =
    DIVIDE([Total Resignations], [Total Employees], 0)

Average Tenure (Years) = AVERAGE(Employees[YearsAtCompany])

Average Salary = AVERAGE(Employees[MonthlyIncome])

Male Count =
    -CALCULATE(
        COUNTROWS(Employees),
        Employees[Status] = "Active",
        Employees[Gender] = "Male"
    )

Female Count =
    CALCULATE(
        COUNTROWS(Employees),
        Employees[Status] = "Active",
        Employees[Gender] = "Female"
    )
```

`Male Count` returns a negative value, consistent with a diverging gender chart. The attrition-rate measure divides resignations by total employees. The task brief instead specifies average headcount as the denominator, but the supplied dataset has no time series to calculate average headcount. `Average Tenure (Years)` averages `YearsAtCompany`; it does not calculate tenure from hire and exit dates.

## Verified findings

Calculated from the Power Query result after deduplicating `EmpID`:

- **1,470** unique employees: **1,233 active** and **237 marked as terminated**.
- **16.12%** attrition rate using the model's measure definition (237 / 1,470).
- The Sales department has the highest departmental attrition rate at **20.63%** (92 / 446), followed by Human Resources at 19.05% and Research & Development at 13.84%.
- Sales Representatives have the highest job-role attrition rate at **39.76%** (33 / 83).
- Attrition is **30.53%** among employees marked for overtime and **10.44%** among those not marked for overtime.

These are descriptive associations in the supplied sample; they do not establish causes of attrition or predict an individual's outcome.

## Project files

- `data/raw/HR_Analytics.csv` - supplied source data
- `powerbi/HR_Analytics.pbip` - Power BI project entry point
- `powerbi/HR_Analytics.Report/` - report pages and visual definitions
- `powerbi/HR_Analytics.SemanticModel/` - TMDL model, Power Query source, and DAX measures
- `.gitignore` - excludes Power BI local settings and cache files

The original task brief PDF and the mismatched cover graphic are not included in the repository. The PBIP package's local settings and cache were also excluded; the model's embedded M source and the supplied CSV are included.

## How to explore

1. Clone or download this repository.
2. Open `powerbi/HR_Analytics.pbip` in Power BI Desktop.
3. Review the report pages and the measure definitions in the TMDL model.
4. Compare the model transformations with `data/raw/HR_Analytics.csv`.

## My contribution

**Individual project.** I prepared the Power BI report and semantic model, reviewed the source data, and documented the DAX measures and verified attrition findings.

# HR-Attrition-Analysis
When and how fast do employees leave? A Python, MySQL, and Power BI analysis with a data quality audit and survival curve.
HR Workforce & Attrition Analysis
An end-to-end people-analytics project: data audit and cleaning in Python, analysis in SQL (MySQL) and Python, and a two-page Power BI dashboard on employee hiring, exits, and retention.

 Dashboard file: dashboard/HR dashboard.pbix

Summary
In this dataset, hiring stayed flat at about 600 people a year while exits grew roughly 7x, so attrition more than doubled and headcount growth slowed every year. About a quarter of hires leave within a year. The data also has several contradictions (for example, "Active" employees with exit dates), which I documented and handled explicitly instead of trusting the status column.

Question this project answers: who leaves, when, and how fast?
Screenshots
Page 1: Overview	Page 2: Attrition Deep Dive
Overview	Deep dive
Key findings
All figures are from this synthetic dataset. Full years 2019 to 2022 are used for trends.

Hiring held flat while exits surged. Hires stayed between 592 and 620 a year, while exits grew from 62 (2019) to 460 (2022).
Attrition more than doubled. Annual attrition rose from 11.9% (2019) to 13.1%, 19.8%, and 27.9% (2022).
Net growth shrank every year. Hires minus exits fell from +536 (2019) to +459, +322, and +160 (2022). In 2023 up to August, exits (596) exceeded hires (335), so headcount fell by 261.
Many hires leave early. 73.7% of hires are still employed after 1 year and 45.1% after 3 years. Median tenure is 2.6 years (Kaplan-Meier survival analysis).
Early exits dominate leavers. 49.1% of all leavers left within their first year.
Exit type tells us little. Leavers split almost evenly: 388 involuntary, 388 voluntary, 380 resignation, and 377 retirement.
Yearly summary
Year	Hires	Exits	Year-end headcount	Attrition
2018 (ramp-up)	255	4	251	n/a
2019	598	62	787	11.9%
2020	592	133	1,246	13.1%
2021	600	278	1,568	19.8%
2022	620	460	1,728	27.9%
2023 (to 6 Aug)	335	596	1,467	partial year
Attrition = exits ÷ average of start-of-year and end-of-year headcount.

Recommendations
If this were real data, I would suggest:

Focus on first-year retention. About a quarter of hires are gone within a year, so review onboarding and early-tenure support.
Treat retention as seriously as hiring. Constant hiring of about 600 a year no longer offsets rising exits.
Fix HR data capture. Employment status, exit dates, and termination types should agree, and retirement should be coded reliably (see the data quality audit).
These follow from what happened in the data. The data does not show why people left, so I make no causal claims.

Table	Shape	Used?
employee_data	3,000 × 26	Yes (main analysis)
employee_engagement_survey_data	3,000 × 5	Inspected, not used
recruitment_data	3,000 × 18	Inspected, not used
training_and_development_data	3,000 × 9	Inspected, not used
Coverage: start dates 2018-08-07 to 2023-08-06, exit dates 2018-11-19 to 2023-08-06. The snapshot date is 2023-08-06, taken from the data and not from today's date.

The raw data is not included in this repository. Download it from Kaggle and place the CSV files in data/raw/. Check the dataset's license before reusing it.

Key definitions
Left: the employee has an ExitDate.
Tenure: years from StartDate to ExitDate, or to the snapshot date for current employees.
Current employees: 1,467. Leavers: 1,533.
Data quality audit
Checking the data before trusting it was a core part of this project.

#	Issue	How I handled it
1	Dates stored as text (20-Sep-19)	Parsed with an explicit format (%d-%b-%y)
2	Extra spaces in text values (for example 'Production       ')	Stripped, so categories don't duplicate
3	991 "Active" employees have exit dates and termination types, so EmployeeStatus is unreliable (1,146 of 1,533 leavers are not marked as terminated)	Defined left by ExitDate. ExitDate and TerminationType agree with each other (both missing for the same 1,467 people)
4	"Future Start" status (69 people), but no start date is after the snapshot	Status ignored
5	LocationCode has 2,821 unique values in 3,000 rows	Not used. State (28 values) is available instead
6	Free-text TerminationDescription is random placeholder text	Excluded
7	Retirement looks unreliable: 242 of 377 "retirements" are under age 60, and 60 are under 30	Reported as recorded and flagged as a data quality issue
8	EmployeeType and EmployeeClassificationType overlap and disagree	Not used
9	Categories split almost evenly (marital status, business unit, exit type), which suggests random generation	No findings built on these
10	Production is 67% of staff (2,020 of 3,000). Executive Office has 24 people	Small groups excluded from department rates
11	No hires before Aug 2018	2018 treated as ramp-up. 2023 is a partial year
Privacy: names and email addresses were dropped. Date of birth was converted to age and then removed.

Method
raw CSV  ->  pandas cleaning  ->  clean CSV  ->  MySQL (SQL analysis)
                                       |
                                       +-->  Python (charts, survival analysis)
                                       |
                                       +-->  Power BI (dashboard)
Inspect and clean (notebooks/01_data_quality_audit.ipynb): audit, date parsing, tenure, age, bands, and the left rule.
Load to MySQL (notebooks/02_load_to_mysql.ipynb): loads the cleaned table into the hr_project database.
SQL analysis (sql/): data checks, yearly hires/exits/attrition, and who-leaves queries.
Python analysis (notebooks/03_analysis_and_charts.ipynb): yearly charts and a Kaplan-Meier survival curve for tenure.
Dashboard (dashboard/): Power BI model with a Calendar table and DAX measures.
Power BI model
Calendar table (2018-01-01 to 2023-12-31) with an active relationship on StartDate and an inactive one on ExitDate, switched on with USERELATIONSHIP in the Exits measure.
Measures: Hires, Exits, Headcount End, Avg Headcount, Attrition %, Net Growth, Current Employees, plus text and color measures for the year-over-year lines.
Bands (age_band, tenure_band) use helper columns with Sort by column.
A custom dark theme is in dashboard/hr_dashboard_dark_theme.json.
Why survival analysis
Current employees are still working, so their tenure is incomplete. A plain average would treat unfinished tenures as final and make people look like they leave sooner than they do. The Kaplan-Meier method uses the "still employed so far" information correctly.

Dashboard
Page 1: Overview. KPI cards with a comparison to the previous year (headcount, hires, exits, attrition, net growth, average tenure), a year slicer (default 2022), hires vs exits, employee trend, attrition rate, gender mix, employees by department, and employees by tenure.

Page 2: Attrition Deep Dive. Attrition by department, exits by type, exits by age group, the survival curve, and key findings.

2022 snapshot (page 1): headcount 1,728, hires 620, exits 460, attrition 27.9%, net growth +160.

The design was inspired by a dashboard mockup from a video. Widgets that need data this dataset doesn't have (salary, leave records, birthdays, Indian cities) were left out on purpose.

Verification
Yearly hires, exits, and year-end headcount were computed in pandas and in Power BI (DAX), and the two matched exactly for 2019 to 2022. The MySQL query in sql/01_yearly_hires_exits_attrition.sql reproduces the same table.
Leaver counts by type (388 + 388 + 380 + 377) add up to the 1,533 leavers.
The current-employee tenure chart (318 + 590 + 559) adds up to the 1,467 current employees.
Limitations
Synthetic data, so no causal or real-world claims.
A 5-year window. Long tenures can't appear, so the average leaver tenure (1.34 years) is biased low, and the survival curve is only reliable up to about 4.5 years.
The survival curve mixes hiring years. Attrition rose over time, so it blends different periods.
2018 is a ramp-up year and 2023 is a partial year, so neither is used in headline comparisons.
Small groups are noisy. Department rates for small departments swing with a few exits, and Executive Office (24 people) is excluded.
Not independently verified: the Avg Tenure card (2.47 years for current employees).
Not yet checked: whether the large 60+ bar in "Exits by age group" simply reflects how many employees are in that age band.
Repository structure
hr-attrition-analysis/
├── README.md
├── data/
│   └── raw/                    (not committed; see Data section)
├── notebooks/
│   ├── 01_data_quality_audit.ipynb
│   ├── 02_load_to_mysql.ipynb
│   └── 03_analysis_and_charts.ipynb
├── sql/
│   ├── 00_create_database.sql
│   ├── 01_data_checks.sql
│   ├── 01_yearly_hires_exits_attrition.sql
│   └── 02_who_leaves.sql
├── dashboard/
│   ├── HR dashboard.pbix
│   └── hr_dashboard_dark_theme.json
└── images/
How to reproduce
Download the dataset from Kaggle and put the CSV files in data/raw/.
Install the requirements: pip install pandas matplotlib seaborn lifelines sqlalchemy pymysql jupyter
Run 01_data_quality_audit.ipynb to create the cleaned files.
In MySQL, run sql/00_create_database.sql, then run 02_load_to_mysql.ipynb. It asks for your MySQL password at run time, and the password is never saved in the notebook.
Run the SQL files in sql/, then 03_analysis_and_charts.ipynb.
Open the .pbix file in Power BI Desktop and refresh the data source path if needed.
Tools
Python 3.14 (pandas, matplotlib, lifelines, SQLAlchemy, PyMySQL), Jupyter, MySQL 8 with Workbench, Power BI Desktop. Built with AI assistance for code and debugging. I checked the results against SQL, pandas, and Power BI.

Skills demonstrated
Data auditing and cleaning, SQL (aggregation, CTEs, window functions), survival analysis, data modeling in Power BI (date table, inactive relationships, DAX), dashboard design, and clear documentation of assumptions and limits.

Next steps
Join the engagement survey and recruitment tables to test whether hiring source or engagement relate to early exits.
Build cohort survival curves by hire year to see whether recent hires leave faster.
Add monthly trends once the monthly attrition measure is defined.

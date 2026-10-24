# wellspring-clinics-analysis
Cleaned healthcare visit data in Python and built a 3-page Power BI dashboard for six clinics.
# Wellspring Clinics: Data Cleaning, Analysis and Power BI Dashboard

I cleaned the raw records of a six-clinic healthcare group in Python, then built a three-page Power BI dashboard to answer questions about visits, waiting times, follow-ups, revenue, patients and staff.

This is a portfolio project. I am a junior data analyst and I wrote down every decision I made, including the mistakes I fixed along the way.

![Overview page](images/overview.png)

---

## 1. Project overview

Wellspring runs six clinics in the Abuja area. The raw workbook has four sheets (clinics, patients, staff and visits) and the data was messy: mixed date formats, fees stored as text, duplicate rows, negative waiting times and inconsistent spellings.

**What I did**
1. Audited the raw data and recorded each problem as a finding, with a decision.
2. Cleaned it in a Jupyter notebook and exported four clean CSV files.
3. Built a Power BI data model and a three-page dashboard.
4. Checked that the dashboard totals match the notebook and each other.

## 2. Business questions

1. How many visits does each clinic handle?
2. Which departments and diagnoses drive the most visits?
3. Where do patients wait the longest?
4. What share of visits need a follow-up?
5. Which clinics earn the most revenue?
6. Who are the patients?
7. How is work spread across staff?

## 3. Dataset

Source file: `wellspring-clinics.xlsx`

| Sheet | Raw rows | Clean rows | What it holds |
|---|---|---|---|
| Clinics | 6 | 6 | Clinic name, state, clinic type, date opened |
| Patients | 142 | 140 | Gender, date of birth, home state, registration date |
| Staff | 24 | 24 | Role, home clinic, years of experience, hire date |
| Visits | 422 | 420 | Date, department, diagnosis, wait time, follow-up, consultation fee |

Visits run from January 2024 to June 2025.

## 4. Tools

- **Python:** pandas, NumPy, matplotlib, seaborn (in a Jupyter Notebook)
- **Power BI Desktop:** data modelling, DAX measures and report design
- **Excel / CSV:** raw workbook and clean exports

## 5. Data cleaning

Every step is in `wellspring_clinic_analysis.ipynb` with a **Finding** and a **Decision** written above the code.

| Table | Problem found | What I did |
|---|---|---|
| Patients, Visits | 2 duplicate patient rows and 2 duplicate visit rows | Removed the extra copies, so no visit or fee is counted twice |
| Visits | `ConsultationFee` stored as text in 18 spellings (`₦3,000`, `NGN 3000`, `3000`) for 6 real prices | Removed symbols and commas, then converted to whole numbers |
| Visits | `VisitDate` in 4 text styles (`13/03/2024`, `14-Dec-2024`, `2025-06-22`, `05-19-2025`) | Identified each style, then parsed each with its own format. 0 dates failed |
| Visits | `FollowUpRequired` had 8 spellings of Yes and No | Standardised to `Yes` / `No` (190 Yes, 230 No) |
| Visits | 23 negative `WaitTimeMinutes` values and 18 blanks | Set negatives to missing instead of guessing. 41 of 420 visits (about 10%) have no valid wait |
| Patients | `Gender` had 10 spellings | Standardised to `Male` / `Female` (85 and 55) |
| Patients | `State` had 7 spellings for 5 states (`FCT`, `Abuja` and `Federal Capital Territory` mean the same place) | Standardised all three to `FCT` |
| Clinics | Same state-name inconsistencies | Standardised to `FCT` |
| Patients | 2 patients with no date of birth (after removing duplicates) | Kept them, because deleting would remove their visits. Their age stays blank and they appear as "Unknown" in age charts |

**Decision I want to explain:** I did not turn a wait of -28 into 28 or fill gaps with the average. The true wait is unknown, so all wait-time numbers use the 379 visits with a valid wait.

## 6. Data model

Five tables in Power BI:

- `visits` (fact table), `patients`, `staff`, `clinics`
- `Calendar`, a date table built with DAX from the first to the last visit date (year, quarter, month and weekday columns)

**Sorting fix:** Power BI sorts text alphabetically, so "5-17" landed after "50-64" and weekdays came out as Mon, Sat, Sun. I added `Age Sort` (a number from 1 to 7) and used the Calendar's `Weekday Number`, then used **Sort by column** on each text field.

```dax
Visits Per staff = DIVIDE([Total Visits],[Active Staff])
```

## 7. The dashboard

All three pages share a left sidebar with a page navigator and a **Clear all slicers** button.

### Page 1: Overview

KPIs: Total Visits **420**, Total Revenue **₦3.10M**, Avg Wait **35.19 min**, Follow-Up Rate **45.24%**, Unique Patients **133**.

Visuals: visits and revenue by month, revenue by clinic, follow-up required (yes/no), top diagnoses, visits by department, visits by weekday. Slicers: Clinic, Department, Year.

### Page 2: Patients and access

KPIs: Unique Patients **133**, Visits per Patient **3.16**, Repeat Patients **112**, Repeat Patient Rate **84.21%**, Registered Patients **140**.

Visuals: patients by age group, patients by home state, patients by gender, visits by wait band, average wait by department. Slicers: Gender, State, Age Group.

### Page 3: Clinics and staff

KPIs: Active Staff **24**, Visits per Staff **17.50**, Avg Fee per Visit **₦7.39K**, Long Wait Rate **21.11%** (waits over 45 minutes), Wait vs Target **+5.19 min** (against a 30-minute target).

Visuals: active staff by role, visits per staff by clinic, average wait trend against target, clinic score card, revenue by clinic type. Slicers: Year, Role, Clinic Type.

## 8. Key findings

1. **Visits:** Wuse handles the most visits (77), then Kubwa (73) and the Specialist Center (72). Garki and Lugbe have the fewest (65 each).
2. **Waiting:** The average wait is 35.19 minutes, 5.19 above target. About 21% of waits are longer than 45 minutes. Departments are close together (32 to 36 minutes), so the delay looks clinic-wide and not tied to one department.
3. **Follow-ups:** 45.24% of visits need a follow-up. The rate is highest at the Specialist Center (54%) and Garki (52%) and lowest at Lugbe (32%) and Kubwa (36%).
4. **Revenue:** Total revenue is ₦3,104,000. The Specialist Center (₦584,500), Maternal Center (₦570,000) and Wuse (₦566,500) earn the most. Garki earns the least (₦424,500).
5. **Patients:** 61% are female and 40% live in FCT. The largest age group is 18 to 34 (41 patients), and 84% of patients who visited came back at least once.
6. **Staff:** The 24 staff are 6 nurses, 6 pharmacists, 5 doctors, 3 lab technicians, 2 community health workers and 2 midwives. That is 17.5 visits per staff member.

## 9. Checks I ran

- The Clinic Score Card first showed 379 visits and ₦2.82M, which did not match the Overview. It was dropping the 41 visits with no wait time. After fixing it, every page agrees: **420 visits, 190 follow-ups, ₦3,104,000**.
- Average wait (35.19), follow-up rate (45.24%), gender split and state counts match the notebook.

## 10. Limitations

- 41 visits (about 10%) have no valid wait time, so wait metrics describe 379 visits.
- Two patients have no date of birth, so they appear as "Unknown" in age charts.
- The data covers 18 months and six clinics, so month-to-month trends are based on small counts.
- I do not know the cause of the negative waits (a recording error is my assumption).

## 11. Project structure

```
wellspring_clinic_analysis/
├── data/
│   ├── raw/wellspring-clinics.xlsx
│   └── clean/
│       ├── clinics_clean.csv
│       ├── patients_clean.csv
│       ├── staff_clean.csv
│       └── visits_clean.csv
├── images/
├── wellspring_clinic_analysis.ipynb
├── wellclinic_db.pbix
└── README.md
```

## 12. How to run it

1. Install the libraries: `pip install pandas numpy matplotlib seaborn openpyxl jupyter`
2. Put the raw workbook in `data/raw/`.
3. Run `wellspring_clinic_analysis.ipynb` from top to bottom. The clean CSV files are written to `data/clean/`.
4. Open `wellclinic_db.pbix` in Power BI Desktop and point the data source to the CSV files in `data/clean/`, then refresh.

## 13. What I learned

- Write the finding and the decision before writing the cleaning code.
- Check that dashboard totals match across pages. A wrong total can hide behind a good-looking chart.
- Blank values behave differently in each tool, so know what a blank date turns into (a blank date of birth became an age of 125 until I handled it).
- Text sorts alphabetically, so months, weekdays and age groups need a sort column.

## 14. About me

**Odoh Ekenedirichukwu Johnpaul**: registered nurse moving into healthcare data analytics.

- LinkedIn: [linkedin.com/in/kene08](https://linkedin.com/in/kene08)
- Email: kennyodoson08@gmail.com

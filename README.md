# Capstone-Python-Project
First attempt at python

# Employee & Project Data Analysis | Python (Pandas & NumPy)

A Python capstone project that builds, cleans, merges and analyzes three related HR / project-management datasets using **Pandas** and **NumPy**. It works through **10 tasks**: creating DataFrames, handling missing values with a loop, transforming columns, joining tables, calculating bonuses, updating designation levels, aggregating costs and filtering records.

**Tools & skills:** Python · Pandas · NumPy · Jupyter Notebook · Data Cleaning · Missing-Value Imputation · Merging / Joining · Conditional Logic · Group-By Aggregation · String Operations

---

## Project at a glance

| | |
|---|---|
| **Employees** | 5 (A001 – A005) |
| **Projects** | 14 (7 Finished, 4 Ongoing, 3 Failed) |
| **Missing values** | 2 (in the project `Cost` column) |
| **Final merged dataset** | 14 rows, one per project |
| **Total bonus awarded** | 632,625 |

## Dataset

Three small tables linked by the employee `ID`:

| Table | Columns | Description |
|---|---|---|
| **Employee** | ID, Name, Gender, City, Age | The project head in charge of each project |
| **Seniority Level** | ID, Designation Level | Grade of the project head (1 = highest, 4 = lowest; beyond 4 the person loses eligibility to head projects) |
| **Project** | ID, Project, Cost, Status | 14 projects with cost and status (Finished / Ongoing / Failed) |

---

## Task-by-task walkthrough

### Task 1. Create three DataFrames and save them as CSV files

**Data available:** Three tables given in the problem statement.

**Approach:** Typed each table into a Python dictionary, converted it to a DataFrame with `pd.DataFrame()`, and saved it with `to_csv(index=False)`. Project costs that were blank in the source were entered as `np.nan`. From Task 2 onward, only the saved CSV files are read back in, as the task requires.

**Result:** `Employee.csv`, `Seniority_Level.csv` and `Project.csv` (included in this repo).

**Why it matters:** Saving raw data to files and reloading it mirrors a real workflow, where analysis starts from stored data instead of hand-built objects.

---

### Task 2. Fill missing project costs with a running average (using a `for` loop)

**Data available:** The `Project` table, where `Cost` was missing for Project 5 and Project 9 (found with `isnull().sum()`).

**Approach:** A `for` loop goes through each row. If `Cost` is missing (`pd.isna`), it is replaced by the mean of all earlier rows:

```python
for i in range(len(project)):
    if pd.isna(project.loc[i, "Cost"]):
        project.loc[i, "Cost"] = project.loc[:i-1, "Cost"].mean()
```

**Result**

| Project | Filled cost | Calculated from |
|---|---|---|
| Project 5 | **3,250,500** | Average of Projects 1–4 |
| Project 9 | **2,210,312.5** | Average of Projects 1–8, including the value filled for Project 5 |

**Why it matters:** A running average keeps every project in the analysis without making up an outside figure. The filled values are estimates and flow into later bonus and total-cost calculations, so they should be treated as approximate.

---

### Task 3. Split the Name column into First Name and Last Name

**Data available:** Employee `Name` (for example, "John Alter").

**Approach:** Split on the space with `str.split(" ", expand=True)` into two new columns, then dropped the original `Name` column.

**Result:** Employee table now has `First Name` and `Last Name` (John / Alter, Alice / Luxumberg, Tom / Sabestine, Nina / Adgra, Amy / Johny).

**Why it matters:** Separate name fields make sorting, greeting and matching records easier.

---

### Task 4. Join all three DataFrames into one called `Final`

**Data available:** Employee, Seniority Level and Project tables, all sharing `ID`.

**Approach:** Two `pd.merge()` calls on `ID`: Employee + Seniority first, then the result + Project.

**Result:** `Final` has **14 rows**, one per project, with employee details, designation level and project details side by side. Every project matched an employee.

**Why it matters:** One combined table lets every later question be answered without switching between tables.

---

### Task 5. Add a 5% bonus for finished projects

**Data available:** `Final` with `Cost` and `Status`.

**Approach:** Created a `Bonus` column set to 0, then used a conditional `loc` to set it to `Cost * 0.05` only where `Status == "Finished"`.

**Result**

| Employee | Finished projects | Bonus |
|---|---|---|
| A001 (John) | Project 1 | 50,100 |
| A003 (Tom) | Projects 3, 10 | 225,000 + 15,000 |
| A004 (Nina) | Project 13 | 150,000 |
| A005 (Amy) | Projects 5, 7, 14 | 162,525 + 20,000 + 10,000 |
| A002 (Alice) | none | 0 |

**Total bonus: 632,625.** Alice is the only employee with no finished project, so she receives no bonus.

**Why it matters:** Tying rewards to completed work is a simple, transparent incentive rule that is easy to audit.

---

### Task 6. Demote employees with failed projects and remove ineligible records

**Data available:** `Final` with `ID`, `Status` and `Designation Level`. On the scale in the brief, 1 is the highest grade and 4 the lowest, and anyone above 4 loses eligibility.

**Approach:** Found the IDs of employees with at least one `Failed` project, then added 1 to `Designation Level` for **all rows of those employees** (a demotion moves the level number *up*, toward 4). Then filtered `Final` to keep only levels of 4 or below.

```python
failed_ids = Final.loc[Final["Status"] == "Failed", "ID"].unique()
Final.loc[Final["ID"].isin(failed_ids), "Designation Level"] += 1
Final = Final[Final["Designation Level"] <= 4].copy()
```

**Result**

| Employee | Failed project | Level change |
|---|---|---|
| A001 (John) | Project 11 | 2 → 3 |
| A002 (Alice) | Project 6 | 2 → 3 |
| A003 (Tom) | Project 8 | 3 → **4** |

- **No records were removed.** Tom reaches level 4, which is the lowest grade but still eligible, since only levels *above* 4 are deleted.
- Each employee is demoted once, even if they had more than one failed project (none did here).

**Why it matters:** Failed projects now feed straight into seniority decisions, and Tom is at the eligibility limit, so any further failure would remove him from heading projects.

---

### Task 7. Add "Mr." / "Mrs." to first names and drop the Gender column

**Data available:** `First Name` and `Gender`.

**Approach:** Prefixed "Mr. " for `M` and "Mrs. " for `F` using conditional `loc`, then dropped `Gender`.

**Result:** First names now read "Mr. John", "Mrs. Alice", "Mr. Tom", "Mrs. Nina" and "Mrs. Amy".

**Why it matters:** The title is now built into the name, and the gender field is removed.

---

### Task 8. Promote employees older than 29 (using an `if` condition)

**Data available:** `Age` and `Designation Level`.

**Approach:** Looped over each unique employee ID and, **if** `Age > 29`, subtracted 1 from `Designation Level` for all of that employee's rows (a promotion moves the level number *down*, toward 1).

```python
for emp_id in Final["ID"].unique():
    age = Final.loc[Final["ID"] == emp_id, "Age"].iloc[0]
    if age > 29:
        Final.loc[Final["ID"] == emp_id, "Designation Level"] -= 1
```

**Result:** Two employees qualify: **Nina (31)** goes from level 2 to **1**, and **Amy (30)** from level 3 to **2**. Tom (29) does not qualify, because the rule is strictly *older than 29*.

**Final designation levels after Tasks 6 and 8**

| Employee | Start | After Task 6 | After Task 8 |
|---|---|---|---|
| A001 John | 2 | 3 | 3 |
| A002 Alice | 2 | 3 | 3 |
| A003 Tom | 3 | 4 | 4 |
| A004 Nina | 2 | 2 | **1** |
| A005 Amy | 3 | 3 | **2** |

**Why it matters:** Age-based rules show how conditional logic updates records in bulk. Nina, the employee with the largest project portfolio, ends up at the top grade.

---

### Task 9. Total project cost per employee (`TotalProjCost`)

**Data available:** `Final` with `ID`, `First Name` and `Cost`.

**Approach:** Grouped by `ID` and `First Name`, summed `Cost`, and renamed the column to `Total Cost`.

**Result**

| ID | First Name | Total Cost |
|---|---|---|
| A001 | Mr. John | 5,212,312.5 |
| A002 | Mrs. Alice | 2,680,000 |
| A003 | Mr. Tom | 5,150,000 |
| A004 | Mrs. Nina | 9,500,000 |
| A005 | Mrs. Amy | 3,850,500 |

**Why it matters:** Nina manages the largest project portfolio (about 36% of the 26.4 M total), which is useful for workload and risk planning. John's and Amy's totals include the estimated costs from Task 2.

---

### Task 10. Employees whose city contains the letter "o"

**Data available:** `City` column.

**Approach:** Filtered with `str.contains("o", case=False)`.

**Result:** Two cities match, **London** (A002, Alice, level 3) and **Newyork** (A004, Nina, level 1), giving 5 project rows. Paris, Berlin and Madrid do not contain an "o".

**Why it matters:** Text filtering is a quick way to search and segment records by location.

---

## Key takeaways

- Two missing costs were filled with a running average, keeping all 14 projects in the analysis.
- Seven finished projects earned a combined bonus of **632,625**.
- Failed projects are 3 of 14 (about 21%), and all belong to A001, A002 and A003.
- One employee (Nina) carries about 36% of total project cost and finishes at the top designation level (1).
- After the demotions and promotions, Tom sits at level 4, the lowest eligible grade.

## Notes & limitations

- **Designation scale:** 1 is the highest grade and 4 the lowest. Demotion therefore *adds* 1 to the level number (Task 6) and promotion *subtracts* 1 (Task 8).
- **Designation is per employee:** Level changes are applied to all of an employee's project rows, so every employee shows one consistent level. An employee with a failed project is demoted once, regardless of how many projects failed.
- **Task order matters:** Task 6 (demotion and deletion) runs before Task 8 (promotion), so the "above 4" check does not see the effect of promotions.
- **Name split** assumes every name has exactly two parts.
- **Estimated costs** from Task 2 are included in the bonus and total-cost results.

## Repository contents

| File | Description |
|---|---|
| `Python_Capstone_Employee_Project_Analysis.ipynb` | Full notebook: code, comments and outputs for all 10 tasks |
| `Employee.csv` | Employee data (Task 1) |
| `Seniority_Level.csv` | Designation levels (Task 1) |
| `Project.csv` | Project data with the original missing costs (Task 1) |
| `README.md` | Project summary |

## How to run

```bash
pip install pandas numpy jupyter
jupyter notebook Python_Capstone_Employee_Project_Analysis.ipynb
```

Run the cells from top to bottom. The notebook creates the three CSV files first and reads them back for every later task.

# Master Dashboard - Complete Implementation Guide

## Architecture Overview

### Data Model

```
                     +---------------+
                     |   dim_Date    |
                     |  SchoolYear   |
                     +-------+-------+
                             | 1
                             |
              +--------------+--------------+
              | *                           | *
    +---------+----------+       +----------+---------+
    |  Program_Summary   |       |    All_Students    |
    | (All 4 programs)   |       | (Pre-App + SYE-CB) |
    +---------+----------+       +----------+---------+
              | *                           | *
              |                             |
              +--------------+--------------+
                             | 1
                     +-------+-------+
                     |  dim_Program  |
                     |  ProgramName  |
                     +---------------+
```

**Program_Summary** = Cross-program KPIs (total students served, recruited, completed).
Sources: All 4 programs (ELC-CDP, ELC-BizTown, Pre-App, SYE-CB).

**All_Students** = Individual student records for demographics and detail analysis.
Sources: Pre-App + SYE-CB only (ELC has no individual student data).

### Table Summary

| Table | Purpose | Loaded to Model |
|-------|---------|:-:|
| `Program_Summary` | Cross-program aggregated KPIs | Yes |
| `All_Students` | Demographics and student-level detail | Yes |
| `dim_Program` | Program dimension | Yes |
| `dim_Date` | SchoolYear dimension | Yes |
| `stg_PreApp` | Staging: standardized Pre-App students | No (staging only) |
| `stg_SYE` | Staging: standardized SYE-CB students | No (staging only) |
| `ELC_CDP_Summary` | Staging: CDP aggregated | No (folded into Program_Summary) |
| `ELC_BT_Summary` | Staging: BizTown aggregated | No (folded into Program_Summary) |
| `PreApp_Summary` | Staging: Pre-App aggregated | No (folded into Program_Summary) |
| `SYE_Summary` | Staging: SYE-CB aggregated | No (folded into Program_Summary) |

### Relationships

| From (Many) | To (One) | Column | Active |
|---|---|---|:-:|
| `Program_Summary[Program]` | `dim_Program[Program]` | Program | Yes |
| `Program_Summary[SchoolYear]` | `dim_Date[SchoolYear]` | SchoolYear | Yes |
| `All_Students[Program]` | `dim_Program[Program]` | Program | Yes |
| `All_Students[SchoolYear]` | `dim_Date[SchoolYear]` | SchoolYear | **No** (inactive) |

> **Why inactive?** Both `All_Students` and `Program_Summary` connect to `dim_Program`,
> and `Program_Summary` connects to `dim_Date`. If `All_Students` → `dim_Date` is also
> active, Power BI sees two paths (direct and via dim_Program → Program_Summary) and
> throws an ambiguity error. Keep it inactive and use `USERELATIONSHIP` in demographic
> measures that need school year filtering.

> **Drill-through note:** Page 1 visuals (bar chart, summary table) must use
> `dim_Program[Program]` on their axis/fields — NOT `Program_Summary[Program]`.
> The drill-through page uses `dim_Program[Program]` as its filter field, and Power BI
> only shows the drill-through option when the source visual contains the same field.

---

## Demographic Column Mapping

This is the exact mapping used in the Power Query code below.

> **IMPORTANT — Pre-App column names contain line breaks `#(lf)` in the source Excel.**
> For example, "Student First Name" is actually `Student#(lf)First Name` in Power Query.
> The queries below use the correct escaped names. If you get "column not found" errors,
> check for `#(lf)` in the actual column names via the Applied Steps preview.
>
> **SYE-CB has trailing spaces** on some column names (e.g., `"Date of Birth "` with a trailing space).
> The queries below handle this with explicit renames.

| Standard Column | Pre-App Source (actual PQ name) | SYE-CB Source (actual PQ name) |
|---|---|---|
| First Name | `Student#(lf)First Name` | Student Name (First) |
| Last Name | `Student#(lf)Last Name` | Student Name (Last) |
| Status | Student Current Status | Status |
| School | High School | School |
| School ID | School ID# | BCPS School ID |
| Date of Birth | Birthdate | `Date of Birth ` *(trailing space!)* |
| Age | Age at Enrollment | *(calculated from DOB in query)* |
| Gender | Gender | Gender |
| Race | Race | Race |
| Ethnicity | Ethnicity | Ethnicity |
| City | City | Address (City) |
| Zip Code | Zip Code | Address (ZIP / Postal Code) |
| Address | Address | Address (Street Address) |
| State | *(n/a — added as null)* | Address (State) |
| Grade Level | *(n/a — added as null)* | Grade Level |
| Household Size | Household Occupants | Household Size |
| Household Income | Household Income | Household Income |
| Household Arrangement | Household Arrangement | Household Arrangement |
| Cultural Influence | Cultural Influence | Cultural Influence |
| Language at Home | Language at Home | Language at Home |
| Student Lunch Status | *(n/a — added as null)* | Student Lunch Status |

---

## SharePoint Source URLs (from your existing reports)

| Program | SharePoint URL |
|---|---|
| **Pre-App** | `https://centuric.sharepoint.com/sites/CC/Documents/Pre-Apprenticeship/Pre-Apprenticeship%20-%20Annual%20Program%20Important%20Docs/FY%202025-26%20PreApp%20Programs/JASF%20Pre-apprenticeship%20Program%20FY25-26%20DASHBOARD.xlsx` |
| **SYE-CB** | `https://centuric.sharepoint.com/sites/CC/Documents/Youth%20Employment/2026/2026%20Summer%20Youth%20Employment/2025-2026%20CB%20SYE%20Dashboard.xlsx` |
| **ELC — CDP** | `https://centuric.sharepoint.com/sites/Capstone/Documents/Experience/Career%20Discover%20Park%20MASTER%20Workbook/25-26%20Career%20Discovery%20Park%20Workbook.xlsx` |
| **ELC — BizTown** | `https://centuric.sharepoint.com/sites/Capstone/Documents/Experience/BizTown%20MASTER%20Workbook/25-26%20BizTown%20Workbook.xlsx` |
| **ELC — BT Visit Day** | `https://centuric.sharepoint.com/sites/Capstone/Documents/Experience/BizTown%20Visit%20Day/SY%2025-26/BizTown%20Visit%20Day%20Workbook/25-26%20Visit%20Day%20Data%20Workbook.xlsx` |

---

## Power Query - Complete M Code

### How to Use

1. In the new Master .pbix, create a new **Blank Query** for each query below (Home > New Source > Blank Query)
2. Open the **Advanced Editor** and paste the M code
3. The Source connections use `Web.Contents` with the exact SharePoint URLs from your existing reports — they should work as-is since you already have credentials cached for these sites
4. Right-click queries marked "staging only" > uncheck **Enable Load**
5. If Power BI prompts for credentials, use the same Organizational Account you use for your other reports
6. Organize queries into folders/groups in the Queries pane (e.g., "Staging", "Model Tables", "Dimensions")

> **Performance note — referenced queries:** When a query references another (e.g.,
> `All_Students` references `stg_PreApp`), Power Query does **not** cache the base query.
> Each referencing query re-executes the base independently. `Binary.Buffer` in the staging
> queries helps avoid downloading the same file twice within a single query, but it does not
> prevent re-downloads across references. For this workload (4 small SharePoint files) this
> is acceptable. If refresh times become long, consider using a Power BI Dataflow to persist
> the staging data.

---

### Query: stg_PreApp (staging only — do not load)

```
let
    // Connect to Pre-App Excel on SharePoint (same source as your existing dashboard)
    FileBinary = Binary.Buffer(
        Web.Contents("https://centuric.sharepoint.com/sites/CC/Documents/Pre-Apprenticeship/Pre-Apprenticeship%20-%20Annual%20Program%20Important%20Docs/FY%202025-26%20PreApp%20Programs/JASF%20Pre-apprenticeship%20Program%20FY25-26%20DASHBOARD.xlsx")
    ),
    Source = Excel.Workbook(FileBinary, null, true),
    CurrentData_Table = Source{[Item="Table247",Kind="Table"]}[Data],

    // Set types for key columns (matches your existing query)
    #"Set Types" = Table.TransformColumnTypes(CurrentData_Table, {
        {"Student#(lf)First Name", type text},
        {"Student#(lf)Last Name", type text},
        {"Student Current Status", type text},
        {"High School", type text},
        {"School ID#", type text},
        {"Birthdate", type date},
        {"Age at Enrollment", Int64.Type},
        {"Address", type text},
        {"City", type text},
        {"Zip Code", type text},
        {"Household Occupants", type text},
        {"Household Income", type text},
        {"Household Arrangement", type text},
        {"Gender", type text},
        {"Race", type text},
        {"Ethnicity", type text},
        {"Cultural Influence", type text},
        {"Language at Home", type text},
        {"Placement#(lf)Y/N?", type text},
        {"Ready for Placement", type text},
        {"Knowledge Gained %", type number}
    }),

    // Add program identifier
    #"Add Program" = Table.AddColumn(#"Set Types", "Program", each "Pre-Apprenticeship", type text),

    // Add school year (update each fiscal year)
    #"Add SchoolYear" = Table.AddColumn(#"Add Program", "SchoolYear", each "2025-26", type text),

    // Standardize column names to match across programs
    // NOTE: Pre-App columns contain #(lf) line breaks from Excel header formatting
    #"Renamed" = Table.RenameColumns(#"Add SchoolYear", {
        {"Student#(lf)First Name", "First Name"},
        {"Student#(lf)Last Name", "Last Name"},
        {"Student Current Status", "Status"},
        {"High School", "School"},
        {"School ID#", "School ID"},
        {"Birthdate", "Date of Birth"},
        {"Age at Enrollment", "Age"},
        {"Household Occupants", "Household Size"}
    }),

    // Add completion flag (Pre-App completion = placed in a job)
    #"Add IsCompleted" = Table.AddColumn(#"Renamed", "IsCompleted",
        each try (if [#"Placement#(lf)Y/N?"] = "YES" then true else false) otherwise false,
        type logical),

    // Add columns that only exist in SYE-CB (so the append works cleanly)
    #"Add GradeLevel" = Table.AddColumn(#"Add IsCompleted", "Grade Level", each null, type text),
    #"Add State" = Table.AddColumn(#"Add GradeLevel", "State", each null, type text),
    #"Add LunchStatus" = Table.AddColumn(#"Add State", "Student Lunch Status", each null, type text),

    // Select final columns for All_Students
    #"Selected" = Table.SelectColumns(#"Add LunchStatus", {
        "Program", "SchoolYear", "First Name", "Last Name", "Status", "IsCompleted",
        "School", "School ID", "Date of Birth", "Age", "Gender", "Race", "Ethnicity",
        "City", "Zip Code", "Address", "State", "Grade Level",
        "Household Size", "Household Income", "Household Arrangement",
        "Cultural Influence", "Language at Home", "Student Lunch Status"
    })
in
    #"Selected"
```

---

### Query: stg_SYE (staging only — do not load)

```
let
    // Connect to SYE-CB Excel on SharePoint (same source as your existing dashboard)
    Source = Excel.Workbook(
        Web.Contents("https://centuric.sharepoint.com/sites/CC/Documents/Youth%20Employment/2026/2026%20Summer%20Youth%20Employment/2025-2026%20CB%20SYE%20Dashboard.xlsx"),
        null, true
    ),
    DataSheet = Source{[Item="CB-SYE 25-26 Data",Kind="Sheet"]}[Data],

    // The source sheet has 2 header rows that need to be skipped (matches your existing query)
    #"Removed Top Rows" = Table.Skip(DataSheet, 2),
    #"Promoted Headers" = Table.PromoteHeaders(#"Removed Top Rows", [PromoteAllScalars=true]),

    // Set types for demographic columns
    // NOTE: Some SYE-CB columns have trailing spaces (e.g., "Date of Birth ")
    #"Set Types" = Table.TransformColumnTypes(#"Promoted Headers", {
        {"Student Name (First)", type text},
        {"Student Name (Last)", type text},
        {"Status", type text},
        {"School", type text},
        {"Grade Level", type text},
        {"Date of Birth ", type date},
        {"Gender", type text},
        {"BCPS School ID", type text},
        {"Race", type text},
        {"Ethnicity", type text},
        {"Household Size", Int64.Type},
        {"Household Arrangement", type text},
        {"Household Income", type text},
        {"Cultural Influence", type text},
        {"Language at Home", type text},
        {"Address (Street Address)", type text},
        {"Address (City)", type text},
        {"Address (State)", type text},
        {"Address (ZIP / Postal Code)", type text},
        {"Student Lunch Status", type text}
    }),

    // Remove blank rows and test data (matches your existing query)
    #"Removed Blank Rows" = Table.SelectRows(#"Set Types",
        each not List.IsEmpty(List.RemoveMatchingItems(Record.FieldValues(_), {"", null}))),

    // Add program identifier
    #"Add Program" = Table.AddColumn(#"Removed Blank Rows", "Program", each "SYE/Career Bound", type text),

    // Add school year
    #"Add SchoolYear" = Table.AddColumn(#"Add Program", "SchoolYear", each "2025-26", type text),

    // Standardize column names
    // NOTE: "Date of Birth " has a trailing space in the source!
    #"Renamed" = Table.RenameColumns(#"Add SchoolYear", {
        {"Student Name (First)", "First Name"},
        {"Student Name (Last)", "Last Name"},
        {"Date of Birth ", "Date of Birth"},
        {"Address (City)", "City"},
        {"Address (ZIP / Postal Code)", "Zip Code"},
        {"Address (Street Address)", "Address"},
        {"Address (State)", "State"},
        {"BCPS School ID", "School ID"}
    }),

    // Calculate Age from Date of Birth
    #"Add Age" = Table.AddColumn(#"Renamed", "Age",
        each if [Date of Birth] <> null
            then Number.RoundDown(Duration.TotalDays(DateTime.LocalNow() - [Date of Birth]) / 365.25)
            else null,
        Int64.Type),

    // Add completion flag (SYE completion = placed in internship)
    #"Add IsCompleted" = Table.AddColumn(#"Add Age", "IsCompleted",
        each try (if [Status] = "Placed" then true else false) otherwise false,
        type logical),

    // Select final columns (must match stg_PreApp exactly for append)
    #"Selected" = Table.SelectColumns(#"Add IsCompleted", {
        "Program", "SchoolYear", "First Name", "Last Name", "Status", "IsCompleted",
        "School", "School ID", "Date of Birth", "Age", "Gender", "Race", "Ethnicity",
        "City", "Zip Code", "Address", "State", "Grade Level",
        "Household Size", "Household Income", "Household Arrangement",
        "Cultural Influence", "Language at Home", "Student Lunch Status"
    })
in
    #"Selected"
```

---

### Query: All_Students (LOAD TO MODEL)

```
let
    Source = Table.Combine({stg_PreApp, stg_SYE})
in
    Source
```

---

### Query: ELC_CDP_Summary (staging only)

This query connects directly to the CDP source workbook and combines the
TeacherComm (CDP schools) and WPB_Schoolnfo (WPB schools) tables — the same
two sources your existing CDP School Info query uses.

```
let
    // Connect to CDP workbook on SharePoint
    FileBinary = Binary.Buffer(
        Web.Contents("https://centuric.sharepoint.com/sites/Capstone/Documents/Experience/Career%20Discover%20Park%20MASTER%20Workbook/25-26%20Career%20Discovery%20Park%20Workbook.xlsx")
    ),
    Workbook = Excel.Workbook(FileBinary, null, true),

    // --- CDP Schools (from TeacherComm table) ---
    TeacherComm = Workbook{[Item="TeacherComm",Kind="Table"]}[Data],
    CDP_Typed = Table.TransformColumnTypes(TeacherComm, {
        {"# of Students Pending", Int64.Type},
        {"Verified Students (from Program Verification Form)", Int64.Type},
        {"School Name", type text},
        {"BCRM LOOKUPID", type text}
    }),
    CDP_Renamed = Table.RenameColumns(CDP_Typed, {
        {"Verified Students (from Program Verification Form)", "Verified Students"}
    }),
    CDP_Filtered = Table.SelectRows(CDP_Renamed,
        each [School Name] <> "Lisa's Test School" and [School Name] <> "Test School"),
    CDP_Selected = Table.SelectColumns(CDP_Filtered, {
        "BCRM LOOKUPID", "School Name", "# of Students Pending", "Verified Students"
    }),

    // --- WPB Schools (from WPB_Schoolnfo table) ---
    WPB_Table = Workbook{[Item="WPB_Schoolnfo",Kind="Table"]}[Data],
    WPB_Typed = Table.TransformColumnTypes(WPB_Table, {
        {"# of Students Pending", Int64.Type},
        {"Verified Students", Int64.Type},
        {"School Name", type text},
        {"BCRM LOOKUPID", type text}
    }),
    WPB_Selected = Table.SelectColumns(WPB_Typed, {
        "BCRM LOOKUPID", "School Name", "# of Students Pending", "Verified Students"
    }),

    // Combine CDP + WPB
    Combined = Table.Combine({CDP_Selected, WPB_Selected}),

    // Aggregate totals (single year workbook, so no grouping by year needed)
    Students_Recruited = List.Sum(Combined[#"# of Students Pending"]),
    Students_Verified = List.Sum(Combined[Verified Students]),
    Schools_Count = Table.RowCount(Combined),

    // Build summary row
    Result = #table(
        type table [Program = text, SchoolYear = text, Students_Recruited = number,
                    Students_Served = number, Students_Completed = number, Schools_Count = number],
        {{"ELC - CDP", "2025-26", Students_Recruited, Students_Verified, Students_Verified, Schools_Count}}
    )
in
    Result
```

---

### Query: ELC_BT_Summary (staging only)

This query connects to TWO BizTown workbooks:
1. **BizTown Master Workbook** — for Projected Student Count (recruited) and school counts
2. **Visit Day Data Workbook** — for actual Verified Student Count (served/completed)

This matches the pattern your existing BT_Accommodations query uses to pull
verified counts from the Visit Day workbook.

```
let
    // =====================================================================
    // PART 1: BizTown Master Workbook (recruited counts + school list)
    // =====================================================================
    BT_Binary = Binary.Buffer(
        Web.Contents("https://centuric.sharepoint.com/sites/Capstone/Documents/Experience/BizTown%20MASTER%20Workbook/25-26%20BizTown%20Workbook.xlsx")
    ),
    BT_Workbook = Excel.Workbook(BT_Binary, null, true),

    // --- BCPS Schools (from TeacherComm table) ---
    TeacherComm = BT_Workbook{[Item="TeacherComm",Kind="Table"]}[Data],
    BCPS_Selected = Table.SelectColumns(TeacherComm, {
        "BCRM LookupId", "School Name", "Projected Student Count"
    }),
    BCPS_Typed = Table.TransformColumnTypes(BCPS_Selected, {
        {"Projected Student Count", Int64.Type},
        {"School Name", type text}
    }),

    // --- Non-BCPS Schools (from OtherSchoolsComm table) ---
    OtherSchools = BT_Workbook{[Item="OtherSchoolsComm",Kind="Table"]}[Data],
    Other_Filtered = Table.SelectRows(OtherSchools,
        each [School Name] <> "Lisa's Elementary School"
        and [School Name] <> "Test Elementary School"
        and [School Name] <> "Test School"),
    Other_Selected = Table.SelectColumns(Other_Filtered, {
        "BCRM LookupId", "School Name", "Projected Student Count"
    }),
    Other_Typed = Table.TransformColumnTypes(Other_Selected, {
        {"Projected Student Count", Int64.Type},
        {"School Name", type text}
    }),

    // Combine BCPS + Non-BCPS for recruited totals
    Combined = Table.Combine({BCPS_Typed, Other_Typed}),
    Students_Recruited = List.Sum(Combined[Projected Student Count]),
    Schools_Count = Table.RowCount(Combined),

    // =====================================================================
    // PART 2: Visit Day Data Workbook (actual verified student counts)
    // =====================================================================
    VD_Binary = Binary.Buffer(
        Web.Contents("https://centuric.sharepoint.com/sites/Capstone/Documents/Experience/BizTown%20Visit%20Day/SY%2025-26/BizTown%20Visit%20Day%20Workbook/25-26%20Visit%20Day%20Data%20Workbook.xlsx")
    ),
    VD_Workbook = Excel.Workbook(VD_Binary, null, true),
    VD_Table = VD_Workbook{[Item="VisitData",Kind="Table"]}[Data],

    // Visit Day table needs header promotion + skip 1 row (matches your BT_Accommodations query)
    VD_Promoted = Table.PromoteHeaders(VD_Table, [PromoteAllScalars=true]),
    VD_Skipped = Table.Skip(VD_Promoted, 1),

    // Type and sum the Verified Student Count column
    VD_Typed = Table.TransformColumnTypes(VD_Skipped, {
        {"Verified Student Count", Int64.Type}
    }),
    Students_Verified = List.Sum(VD_Typed[Verified Student Count]),

    // =====================================================================
    // BUILD SUMMARY ROW
    // =====================================================================
    // Recruited = Projected Student Count (from Master Workbook)
    // Served & Completed = Verified Student Count (from Visit Day Workbook)
    Result = #table(
        type table [Program = text, SchoolYear = text, Students_Recruited = number,
                    Students_Served = number, Students_Completed = number, Schools_Count = number],
        {{"ELC - BizTown", "2025-26", Students_Recruited, Students_Verified, Students_Verified, Schools_Count}}
    )
in
    Result
```

---

### Query: PreApp_Summary (staging only)

```
let
    Source = stg_PreApp,

    #"Grouped" = Table.Group(Source, {"SchoolYear"}, {
        {"Students_Recruited", each Table.RowCount(_), Int64.Type},
        {"Students_Served",
            each Table.RowCount(Table.SelectRows(_, each [Status] = "ACTIVE")),
            Int64.Type},
        {"Students_Completed",
            each Table.RowCount(Table.SelectRows(_, each [IsCompleted] = true)),
            Int64.Type},
        {"Schools_Count",
            each List.Count(List.Distinct(List.RemoveNulls(_[School]))),
            Int64.Type}
    }),

    #"Add Program" = Table.AddColumn(#"Grouped", "Program", each "Pre-Apprenticeship", type text)
in
    #"Add Program"
```

---

### Query: SYE_Summary (staging only)

```
let
    Source = stg_SYE,

    #"Grouped" = Table.Group(Source, {"SchoolYear"}, {
        {"Students_Recruited", each Table.RowCount(_), Int64.Type},
        {"Students_Served", each Table.RowCount(_), Int64.Type},
        {"Students_Completed",
            each Table.RowCount(Table.SelectRows(_, each [IsCompleted] = true)),
            Int64.Type},
        {"Schools_Count",
            each List.Count(List.Distinct(List.RemoveNulls(_[School]))),
            Int64.Type}
    }),

    #"Add Program" = Table.AddColumn(#"Grouped", "Program", each "SYE", type text)
in
    #"Add Program"
```

---

### Query: Program_Summary (LOAD TO MODEL)

```
let
    Source = Table.Combine({ELC_CDP_Summary, ELC_BT_Summary, PreApp_Summary, SYE_Summary}),

    #"Reordered" = Table.ReorderColumns(Source, {
        "Program", "SchoolYear",
        "Students_Recruited", "Students_Served", "Students_Completed",
        "Schools_Count"
    }),

    #"Set Types" = Table.TransformColumnTypes(#"Reordered", {
        {"Students_Recruited", Int64.Type},
        {"Students_Served", Int64.Type},
        {"Students_Completed", Int64.Type},
        {"Schools_Count", Int64.Type}
    })
in
    #"Set Types"
```

---

### Query: dim_Program (LOAD TO MODEL)

```
let
    Source = #table(
        type table [
            Program = text,
            ProgramCategory = text,
            HasStudentDetail = logical,
            ProgramDescription = text
        ],
        {
            {"ELC - CDP", "ELC", false, "Career Discovery Program"},
            {"ELC - BizTown", "ELC", false, "BizTown Experience"},
            {"Pre-Apprenticeship", "Pre-App", true, "Pre-Apprenticeship Program"},
            {"SYE/Career Bound", "SYE", true, "Summer Youth Employment / Career Bound"}
        }
    )
in
    Source
```

---

### Query: dim_Date (LOAD TO MODEL)

```
let
    Source = #table(
        type table [
            SchoolYear = text,
            FYStart = date,
            FYEnd = date
        ],
        {
            {"2023-24", #date(2023, 7, 1), #date(2024, 6, 30)},
            {"2024-25", #date(2024, 7, 1), #date(2025, 6, 30)},
            {"2025-26", #date(2025, 7, 1), #date(2026, 6, 30)},
            {"2026-27", #date(2026, 7, 1), #date(2027, 6, 30)}
        }
    )
in
    Source
```

---

## DAX Measures

Create these measures in a new Measures table or directly in Program_Summary / All_Students.

### Organization-Wide KPIs (Executive Overview page)

```
Total Students Served =
SUM(Program_Summary[Students_Served])
```

```
Total Students Recruited =
SUM(Program_Summary[Students_Recruited])
```

```
Total Students Completed =
SUM(Program_Summary[Students_Completed])
```

```
Total Schools =
SUM(Program_Summary[Schools_Count])
```

```
Org Completion Rate =
DIVIDE([Total Students Completed], [Total Students Served], 0)
```

```
ELC Students Served =
CALCULATE(
    SUM(Program_Summary[Students_Served]),
    dim_Program[HasStudentDetail] = FALSE()
)
```

```
Detail Students Served =
CALCULATE(
    SUM(Program_Summary[Students_Served]),
    dim_Program[HasStudentDetail] = TRUE()
)
```

### Reference Label Measures (for New Card visual subtitles)

> **Note:** The old Card and Multi-row Card visuals are legacy. Use the **New Card** visual
> (introduced 2023+), which natively supports a callout value plus reference labels —
> no text-concatenation hacks needed.

```
Previous Year Served = 
// 1. Calculate the prior year label based on the current context's minimum FY Start date
VAR _priorYearLabel =
    FORMAT(YEAR(MIN(dim_Date[FYStart])) - 1, "0000") & "-" &
    RIGHT(FORMAT(YEAR(MIN(dim_Date[FYStart])), "0000"), 2)

RETURN
CALCULATE(
    SUM(Program_Summary[Students_Served]),
    
    // 2. Remove existing filters from the Date table so the new filter doesn't clash
    REMOVEFILTERS(dim_Date), 
    
    // 3. Apply the new prior year filter
    dim_Date[SchoolYear] = _priorYearLabel
)
```

```
YoY Served Change =
VAR _current = [Total Students Served]
VAR _prior = [Previous Year Served]
RETURN
    IF(
        NOT ISBLANK(_current),      // Only calculate if we have current data
        DIVIDE(_current - _prior, _prior)
    )
```

```
Served vs Recruited Pct =
DIVIDE([Total Students Served], [Total Students Recruited], 0)
```

```
Completion vs Served Pct =
DIVIDE([Total Students Completed], [Total Students Served], 0)
```

#### Per-Program Breakdowns (reference labels on each KPI card)

**Total Students Served — by program:**

```
Served - BizTown =
CALCULATE([Total Students Served], Program_Summary[Program] = "ELC - BizTown")
```

```
Served - CDP =
CALCULATE([Total Students Served], Program_Summary[Program] = "ELC - CDP")
```

```
Served - Pre-App =
CALCULATE([Total Students Served], Program_Summary[Program] = "Pre-Apprenticeship")
```

```
Served - SYE =
CALCULATE([Total Students Served], Program_Summary[Program] = "SYE")
```

**Total Students Recruited — by program:**

```
Recruited - BizTown =
CALCULATE([Total Students Recruited], Program_Summary[Program] = "ELC - BizTown")
```

```
Recruited - CDP =
CALCULATE([Total Students Recruited], Program_Summary[Program] = "ELC - CDP")
```

```
Recruited - Pre-App =
CALCULATE([Total Students Recruited], Program_Summary[Program] = "Pre-Apprenticeship")
```

```
Recruited - SYE =
CALCULATE([Total Students Recruited], Program_Summary[Program] = "SYE")
```

**Total Students Completed — by program:**

```
Completed - BizTown =
CALCULATE([Total Students Completed], Program_Summary[Program] = "ELC - BizTown")
```

```
Completed - CDP =
CALCULATE([Total Students Completed], Program_Summary[Program] = "ELC - CDP")
```

```
Completed - Pre-App =
CALCULATE([Total Students Completed], Program_Summary[Program] = "Pre-Apprenticeship")
```

```
Completed - SYE =
CALCULATE([Total Students Completed], Program_Summary[Program] = "SYE")
```

**Total Schools — by program:**

```
Schools - BizTown =
CALCULATE([Total Schools], Program_Summary[Program] = "ELC - BizTown")
```

```
Schools - CDP =
CALCULATE([Total Schools], Program_Summary[Program] = "ELC - CDP")
```

```
Schools - Pre-App =
CALCULATE([Total Schools], Program_Summary[Program] = "Pre-Apprenticeship")
```

```
Schools - SYE =
CALCULATE([Total Schools], Program_Summary[Program] = "SYE")
```

### Demographic Measures (from All_Students table)

```
Total Students (Demographics) =
CALCULATE(
    COUNTROWS(All_Students),
    USERELATIONSHIP(All_Students[SchoolYear], dim_Date[SchoolYear])
)
```

```
Pct of Total =
VAR _count =
    CALCULATE(
        COUNTROWS(All_Students),
        USERELATIONSHIP(All_Students[SchoolYear], dim_Date[SchoolYear])
    )
VAR _total =
    CALCULATE(
        COUNTROWS(All_Students),
        USERELATIONSHIP(All_Students[SchoolYear], dim_Date[SchoolYear]),
        ALL(All_Students)
    )
RETURN
DIVIDE(_count, _total, 0)
```

```
Pct Female =
CALCULATE(
    DIVIDE(
        CALCULATE(COUNTROWS(All_Students), All_Students[Gender] = "Female"),
        COUNTROWS(All_Students),
        0
    ),
    USERELATIONSHIP(All_Students[SchoolYear], dim_Date[SchoolYear])
)
```

```
Pct Male =
CALCULATE(
    DIVIDE(
        CALCULATE(COUNTROWS(All_Students), All_Students[Gender] = "Male"),
        COUNTROWS(All_Students),
        0
    ),
    USERELATIONSHIP(All_Students[SchoolYear], dim_Date[SchoolYear])
)
```

```
Avg Student Age = 
CALCULATE(
    AVERAGE(All_Students[Age]),
    KEEPFILTERS( NOT(All_Students[Age] IN {0, 1, 126}) )
)
```

```
Age Range = 
CALCULATE(
    VAR _min = MIN(All_Students[Age])
    VAR _max = MAX(All_Students[Age])
    
    RETURN
        IF(
            NOT ISBLANK(_min),
            IF(
                _min = _max,
                "Age: " & _min,
                "Range: " & _min & "–" & _max
            )
        ),
    // 1. Apply the inactive relationship
    USERELATIONSHIP(All_Students[SchoolYear], dim_Date[SchoolYear]),
    
    // 2. Ignore the default 'unknown' age safely
    KEEPFILTERS(All_Students[Age] <> 126 && All_Students[Age] <> 0 && All_Students[Age] <> 1)
)
```

```
Top Race =
VAR _tbl =
    CALCULATETABLE(
        TOPN(
            1,
            ADDCOLUMNS(
                VALUES(All_Students[Race]), -- Group by Race
                "Cnt", CALCULATE(COUNTROWS(All_Students)) -- Safely calculate count
            ),
            [Cnt], DESC
        ),
        USERELATIONSHIP(All_Students[SchoolYear], dim_Date[SchoolYear])
    )
RETURN
    MAXX(_tbl, [Race])
```

```
Top City =
VAR _tbl =
    CALCULATETABLE(
        TOPN(
            1,
            ADDCOLUMNS(
                VALUES(All_Students[City]), -- Group by City
                "Cnt", CALCULATE(COUNTROWS(All_Students)) -- Safely calculate count
            ),
            [Cnt], DESC
        ),
        USERELATIONSHIP(All_Students[SchoolYear], dim_Date[SchoolYear])
    )
RETURN
    MAXX(_tbl, [City])
```

```
Demographic Note =
VAR _detailCount = [Total Students (Demographics)]
VAR _totalServed = [Total Students Served]
VAR _elcCount = [ELC Students Served]
RETURN
    "Showing demographics for " & FORMAT(_detailCount, "#,##0") &
    " of " & FORMAT(_totalServed, "#,##0") & " total students." & UNICHAR(10) &
    "ELC programs (" & FORMAT(_elcCount, "#,##0") &
    " students) do not have individual demographic data."
```

```
Program Completion Rate =
DIVIDE(
    SUM(Program_Summary[Students_Completed]),
    SUM(Program_Summary[Students_Served]),
    0
)
```

### Conditional Formatting Helper

```
Completion Rate Color =
VAR _rate = [Program Completion Rate]
RETURN
SWITCH(
    TRUE(),
    _rate >= 0.80, "#8FC440",
    _rate >= 0.60, "#E3E24F",
    "#E15759"
)
```

### Visual Calculations (use directly on visuals — no model measure needed)

> Visual Calculations run in the context of the visual, not the model. They are ideal for
> running totals, moving averages, and rankings that only make sense on a specific chart.
> Enable under **Options > Preview features > Visual calculations** if not already on.

Use these directly on charts via the visual's calculation editor — no need to create DAX measures:

- **Running total of students served** on a bar chart sorted by school year:
  `Running Total = RUNNINGSUM([Total Students Served])`
- **Rank programs** by completion rate on a table visual:
  `Rank = RANK([Program Completion Rate])`
- **Previous year comparison** on a bar chart:
  `Prev Year = PREVIOUS([Total Students Served])`

### Field Parameters (dynamic dimension/measure swapping)

> Field Parameters (GA July 2025) let users swap what a visual shows via a slicer.
> Useful on the Demographics page so users can switch between Race, Ethnicity,
> City, etc. in a single chart rather than needing 6 separate charts.

```
Demographic Dimension =
{
    ("Race", NAMEOF(All_Students[Race]), 0),
    ("Ethnicity", NAMEOF(All_Students[Ethnicity]), 1),
    ("Gender", NAMEOF(All_Students[Gender]), 2),
    ("City", NAMEOF(All_Students[City]), 3),
    ("Household Income", NAMEOF(All_Students[Household Income]), 4),
    ("Language at Home", NAMEOF(All_Students[Language at Home]), 5)
}
```

To create: **Modeling** > **New parameter** > **Fields**. Select the columns above.
Power BI auto-creates a slicer. Users pick a dimension and the chart updates dynamically.

---

## Report Page Layout

### Page 1: Executive Overview
| Section | Visual | Data Source |
|---|---|---|
| Top banner | **New Card** visuals (4) | Total Students Served, Recruited, Completed, Schools — each with a reference label showing YoY change or ratio |
| Middle | Clustered bar chart | Program_Summary: Program on axis, Students_Served as value |
| Middle | Table or Matrix | Program_Summary: all columns grouped by Program |
| Bottom | **New Card** per program (or KPI visual) | Program Completion Rate with conditional formatting via reference labels |
| Slicer | SchoolYear dropdown (new Slicer visual) | dim_Date[SchoolYear] |

### Page 2: Demographics (All Students)
| Section | Visual | Data Source |
|---|---|---|
| Top | **New Card** (subtitle mode) | Demographic Note measure (coverage disclaimer) |
| Left | Donut chart | All_Students[Gender] grouped |
| Center | Horizontal bar chart | All_Students[Race] grouped, sorted descending |
| Right | Horizontal bar chart | All_Students[Ethnicity] grouped |
| Bottom-left | Bar chart | All_Students[City] grouped |
| Bottom-center | Bar chart | All_Students[Household Income] grouped |
| Bottom-right | Bar chart | All_Students[Language at Home] grouped |
| Slicers | Program, SchoolYear | All_Students[Program], dim_Date[SchoolYear] |

### Page 3: Program Drill-Through
Set up as a drill-through page filtered by dim_Program[Program].

- When leadership clicks a program on Page 1, this page shows program-specific detail
- **Pre-App**: Pipeline funnel using `Pipeline Stages` table + `Pipeline Count` measure (Enrolled → Onboarded → Soft Skills Complete), plus KPI sidebar (see measures below)
- **SYE-CB**: Placement metrics, satisfaction scores, internship site breakdown
- **ELC programs**: School-level table (recruited vs verified by school)

#### Pre-App KPI Sidebar Measures

```
PreApp Active Students =
CALCULATE(
    COUNTROWS(All_Students),
    All_Students[Status] = "ACTIVE"
)
```

```
PreApp Placement Rate =
DIVIDE(
    CALCULATE(COUNTROWS(All_Students), All_Students[IsCompleted] = TRUE()),
    COUNTROWS(All_Students),
    0
)
```

```
Avg Knowledge Gained =
AVERAGE(All_Students[Knowledge Gained %])
```
> **Note:** `Knowledge Gained %` must be added to `All_Students`. It exists in the
> Pre-App source but is not currently in the `#"Selected"` step. Add it to `stg_PreApp`
> and add a null column in `stg_SYE` so the append works.

```
Placement Target Label = "Target: 80%"
```

```
Knowledge Benchmark Label = "Benchmark: 70%"
```

---

## Implementation Checklist

1. [ ] Create new blank .pbix file (Master_Dashboard.pbix)
2. [ ] Create all Power Query queries (paste M code into Advanced Editor)
3. [ ] Verify each query loads without errors (check Applied Steps preview)
4. [ ] Disable load on all staging queries (right-click > uncheck Enable Load)
5. [ ] Enable load on: All_Students, Program_Summary, dim_Program, dim_Date
6. [ ] Set up relationships in Model view (see Relationships table above)
7. [ ] Create DAX measures
8. [ ] Create the JSON theme file (see Layout Guide > Theme File section)
9. [ ] Build Page 1: Executive Overview
10. [ ] Build Page 2: Demographics
11. [ ] Build Page 3: Program Drill-Through
12. [ ] Add alt text to all visuals (see Accessibility section in Layout Guide)
13. [ ] Create mobile layout for each page
14. [ ] Test with SchoolYear slicer
15. [ ] Test RLS if applicable
16. [ ] Publish to Power BI Service

---

## Maintenance Notes

### Updating for a New Fiscal Year
- In `stg_PreApp` and `stg_SYE`: update the SchoolYear value (e.g., "2026-27")
- Update the `Web.Contents` URL to point to the new year's workbook
- In `ELC_CDP_Summary` and `ELC_BT_Summary`: update both URLs and SchoolYear in the Result table
- For `ELC_BT_Summary` specifically: update both the BizTown Master Workbook URL **and** the Visit Day Data Workbook URL
- Update the Excel table/sheet names if they change (e.g., "Table247", "CB-SYE 25-26 Data")
- Add new row to `dim_Date` if needed

### If ELC Adds Demographic Data Later
- Create a new staging query for ELC student demographics
- Append it to All_Students alongside Pre-App and SYE-CB
- Update dim_Program: set HasStudentDetail = TRUE for ELC programs
- The demographic measures and visuals will automatically include ELC

### Handling "Status" Differences
Pre-App uses "ACTIVE" for current students. SYE-CB may use different values like "Placed".
The IsCompleted flag normalizes this:
- Pre-App: IsCompleted = TRUE when `Placement#(lf)Y/N?` = "YES"
- SYE-CB: IsCompleted = TRUE when Status = "Placed"
If you need to filter by active/current students across both programs,
add an IsActive column in the staging queries with your own logic.

### Column Name Gotchas
- **Pre-App**: Many columns contain `#(lf)` (line breaks) from Excel formatting.
  If columns aren't found, open the raw query in Applied Steps and check actual names.
- **SYE-CB**: Some columns have trailing spaces (e.g., `"Date of Birth "`).
  The queries handle this, but if you add new columns, watch for trailing spaces.

### Feature Deprecation Timeline
- **Q&A visual**: Deprecated December 2026. Do not add Q&A visuals to new reports.
  Use **Copilot** instead (requires Fabric F2+ or Premium P1+ capacity).
- **Quick Measure suggestions**: Retired late 2024. Use **DAX Query View with Copilot**
  for AI-assisted measure authoring.
- **R/Python visuals**: Will render blank in embedded/Publish-to-web scenarios starting
  May 2026. They still work in Desktop and Power BI Service interactive mode.
- **Bing Maps**: Being replaced by **Azure Maps**. Migrate any Bing Maps visuals to Azure
  Maps before mid-2026 to avoid compatibility issues.
- **Metric Sets**: Creation ended October 2025, full removal early 2026. Transition to
  **Scorecards** for KPI tracking.
- **PBIR format**: Default report format in Power BI Service (Jan 2026) and Desktop
  (March 2026). This is a folder-based, Git-friendly format. GA expected Q3 2026.
  PBIR will become the **only** supported report format once GA — adopt early.
- **New Card visual**: Became the default card visual (GA November 2025). The legacy
  Card and Multi-row Card visuals are no longer recommended. See Layout Guide for
  post-GA formatting changes (grid layout, padding, reference label areas).

### Copilot & RLS Warning
> **Important**: As of early 2026, there are documented scenarios where Microsoft Copilot
> in Power BI can surface data outside RLS (Row-Level Security) boundaries, particularly
> when the Preview toggle is enabled. For dashboards containing PII or sensitive data,
> thoroughly test Copilot behavior across all security roles before enabling it.

### Performance Optimization Tips
- Run **Performance Analyzer** (View > Performance Analyzer) to identify slow visuals
- Use **DAX Studio** for advanced query profiling and cache clearing
- Prefer `SUM()` over `SUMX()` — set-based aggregators are faster than iterators
- Use `SUMMARIZECOLUMNS()` instead of `SUMMARIZE()` for grouped aggregations
- Use `TREATAS()` for virtual relationships instead of `INTERSECT` or `FILTER`
- Avoid replacing BLANKs with zeros — Power BI auto-filters blank rows for performance
- Prefer measures over calculated columns whenever possible
- Consider **Calculation Groups** for reusable time intelligence patterns
- Limit to **8 widget visuals** per page and **1 grid/table** per page
- Keep star schema design (already implemented in this model)

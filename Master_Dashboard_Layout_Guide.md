# Master Dashboard — Detailed Report Layout Guide

This guide walks you through building each visual on every page of the Master Dashboard in Power BI.
It references the measures and tables defined in `Master_Dashboard_Guide.md`.

---

## General Setup

### Canvas Settings
- Go to **File > Options > Report settings**
- Canvas size: **16:9 (1920 x 1080)** — this is the modern default and gives more room for visuals
- Default theme: apply the JA South Florida JSON theme file (see below) via **View > Themes > Browse for themes**
- Consider enabling the **Modern visual tooltips** and **On-object interaction** preview features under Options > Preview features
- **Visual density**: Limit to **8 widget visuals** and **1 grid/table** per report page for optimal performance. Each visual generates its own DAX query — too many visuals slow rendering significantly
- **Name pages descriptively** (e.g., "Executive Overview", "Student Demographics") — screen readers announce page names, and descriptive names help all users navigate

### Color Palette — JA South Florida Branding (used throughout)

> Based on the **Junior Achievement of South Florida Branding Guidelines 2022**, page 8.
> Primary colors should make up ~60% of the design, support colors 30-50%, accent colors 1-5%.

**Primary Colors (60% — dominant in the dashboard)**
| Use | Hex | JA Name |
|-----|-----|---------|
| ELC - CDP | `#285F74` | Boundless Blue |
| ELC - BizTown | `#00A0AF` | Resilient Turquoise |

**Support Colors (30-50% — secondary visuals, cards, table headers)**
| Use | Hex | JA Name |
|-----|-----|---------|
| Pre-Apprenticeship | `#22404D` | Immersive Blue Black |
| SYE/Career Bound | `#00763D` | Sustainable Green |
| Background (canvas) | `#FFFFFF` | Startup White |
| Card background | `#FFFFFF` | Startup White |
| Highlight / Pale fills | `#99D9DF` | Pale Blue |
| Positive indicator | `#8FC440` | Key Lime |

**Accent Colors (1-5% — sparingly for highlights, accent lines)**
| Use | Hex | JA Name |
|-----|-----|---------|
| Accent / highlight | `#00C0CA` | Aqua |
| Title text | `#22404D` | Immersive Blue Black |
| Secondary accent | `#009424` | Grass Green |
| Warning / attention | `#E3E24F` | Empowered Yellow |
| Info banner fill | `#F3F2B3` | Pale Yellow |
| Light fill / ice | `#C3EDEF` | Ice |

### Font Conventions
- **Card values**: Segoe UI Bold, 28pt
- **Card labels**: Segoe UI, 10pt, color `#285F74` (Boundless Blue)
- **Chart titles**: Segoe UI Semibold, 12pt
- **Axis labels**: Segoe UI, 9pt
- **Table headers**: Segoe UI Semibold, 10pt

### JSON Theme File (recommended)

Instead of formatting every visual manually, create a JSON theme file and apply it via
**View > Themes > Browse for themes**. This ensures consistency across all visuals and
makes it easy to update branding later.

> **Tips for working with JSON themes:**
> - Edit the file in **Visual Studio Code** (not Notepad) for syntax highlighting and error detection
> - Validate your JSON against Microsoft's [official JSON schema](https://github.com/microsoft/powerbi-desktop-samples/blob/main/Report%20Theme%20JSON%20Schema/README.md) (updated monthly through 2026)
> - Alternatively, use the **PowerBI.Tips Theme Generator** or **BIBB Generator** to build themes visually
> - To export your current formatting as a starting point: **View > Themes > Save current theme**
> - Global theme properties can be overridden per-visual in the Format pane

Save the following as `master_dashboard_theme.json`:

```json
{
    "name": "JA South Florida - Master Dashboard",
    "dataColors": [
        "#285F74",
        "#00A0AF",
        "#22404D",
        "#00763D",
        "#8FC440",
        "#00C0CA"
    ],
    "background": "#FFFFFF",
    "foreground": "#22404D",
    "tableAccent": "#285F74",
    "good": "#8FC440",
    "neutral": "#E3E24F",
    "bad": "#E15759",
    "maximum": "#285F74",
    "center": "#00A0AF",
    "minimum": "#C3EDEF",
    "visualStyles": {
        "*": {
            "*": {
                "general": [{"responsive": true}],
                "title": [{
                    "fontFamily": "Segoe UI Semibold",
                    "fontSize": 12,
                    "fontColor": "#22404D"
                }],
                "background": [{"color": "#FFFFFF", "transparency": 0}],
                "border": [{"show": false}],
                "shadow": [{"show": true, "color": "#00000015", "position": "Outer"}]
            }
        }
    }
}
```

> **Note on dataColors order:** The first 4 colors map to the 4 programs in chart legends
> (CDP = Boundless Blue, BizTown = Resilient Turquoise, Pre-App = Immersive Blue Black,
> SYE = Sustainable Green). The `good`/`neutral`/`bad` properties control conditional
> formatting defaults (Key Lime / Empowered Yellow / Red).

### Accessibility (WCAG 2.1 AA)

> **Regulatory context**: Section 508 (US federal), the European Accessibility Act (EAA,
> effective June 2025), and DOJ Title II rules (compliance deadlines 2026-2028) all require
> accessible digital content. Building accessibility in from the start avoids costly retrofits.

- **Contrast ratios**: Minimum 4.5:1 for normal text (<18pt), 3:1 for large text (18pt+ or 14pt bold).
  Use tools like **WebAIM Contrast Checker** or **Colour Contrast Analyser** to verify.
- **Alt text**: Add alt text to **every** visual. Right-click visual > **Format** >
  **General** > **Alt text**. Use a DAX measure for dynamic alt text where possible,
  e.g., `"Bar chart showing " & [Total Students Served] & " students served across 4 programs."`
- **Do not rely on color alone**: The conditional formatting on completion rates uses
  green/amber/red — add icons or text labels alongside the color so colorblind users
  can interpret the data.
- **Tab order**: Set the tab order for keyboard navigation via **View > Selection pane**.
  Order should follow the visual reading flow: slicers first, then KPI cards, then charts.
- **Touch targets**: All interactive elements should be at least 44x44 px.
- **Avoid scrolling within visuals**: Scrollbars create challenges for keyboard users.
  Use pagination, drill-through pages, or Top N filters instead.
- **Testing checklist** (layered approach):
  1. Run Power BI Desktop's built-in **Accessibility Checker** (Review tab)
  2. Test **keyboard navigation** manually: Tab, Enter, Escape, and arrow keys
  3. Test with a **screen reader** (NVDA or JAWS) on the published report
  4. Validate color contrast ratios with WebAIM or Colour Contrast Analyser
  5. For published reports, use **Accessibility Insights for Web** (browser extension)

---

## Page 1: Executive Overview

### Layout Grid (1920 x 1080 canvas)

```
+------------------------------------------------------------------------------------+
| [SchoolYear Slicer]                                                   (top-right)  |
+------------+------------+------------+------------+--------------------------------+
| New Card   | New Card   | New Card   | New Card   |                                |
| Served     | Recruited  | Completed  | Schools    |                                |
| (ref:ratio)| (ref:ratio)| (ref:ratio)| (ref:ratio)|                                |
+------------+------------+------------+------------+                                |
|                                                   |    Summary Table               |
|   Clustered Bar Chart                             |    (Program Matrix)             |
|   (Students by Program)                           |                                |
|                                                   |                                |
+---------------------------------------------------+--------------------------------+
| New Card       | New Card       | New Card        | New Card                       |
| Avg Age        | Pct Female     | Completion Rate | Top Race                       |
| (ref: range)   | (ref: Pct Male)| (ref:Served %)  | (ref: Top City)                |
+----------------+----------------+-----------------+--------------------------------+
```

> **Why New Card instead of Gauge?** The Gauge visual is not deprecated and is still valid
> for operational dashboards. However, for this executive-level dashboard, the New Card
> visual with reference labels is a better fit — it's more compact, easier to read at a
> glance, and consistent with the KPI cards at the top. If you prefer gauges for the
> visual progress-toward-target metaphor, they remain a supported option.

### Step-by-Step Instructions

#### 1.1 — SchoolYear Slicer (top-right corner)

1. **Insert** > **Slicer** (use the **new Slicer** visual — not the legacy one)
2. **Field**: drag `dim_Date[SchoolYear]` into the slicer
3. **Position**: X=1580, Y=15, Width=310, Height=45
4. **Format**:
   - Slicer settings > Style: **Dropdown**
   - Selection > Single select: **On**
   - Values > Font: Segoe UI, 10pt
   - Background: White, 0% transparency
   - Border: Light gray (`#CCCCCC`), 1px
   - The new Slicer visual supports search, horizontal/vertical lists, and between ranges natively

#### 1.2 — KPI Cards (top row — 4 New Card visuals)

Create 4 **New Card** visuals side by side across the top. The New Card visual became
**Generally Available in November 2025** and is now the default card visual. It supports
a primary callout value **plus** reference labels — so you can show the KPI number and a
contextual subtitle (e.g., YoY change, ratio) in one clean visual. Do **not** use the
legacy Card or Multi-row Card visuals.

> **Post-GA Changes (Nov 2025):** The GA release changed several defaults:
> - **Layout**: Only **Grid** arrangement is supported (vertical/horizontal were removed)
> - **Padding**: Default increased to **12px** — adjust via Format > search "Padding"
> - **Reference labels**: Now have their own dedicated area (default 50% of card height).
>   Adjust via **Cards > Layout > Callout size %** to control the ratio
> - **Images**: Top-level Image section is now for hero images; callout images moved to
>   **Callout > Image** in the format pane

| Card # | Position (X, Y) | Size (W x H) | Callout Measure | Reference Labels (per-program breakdown) |
|--------|----------------|--------------|-----------------|------------------------------------------|
| 1 | X=30, Y=15 | 360 x 160 | `Total Students Served` | `Served - BizTown`, `Served - CDP`, `Served - Pre-App`, `Served - SYE` |
| 2 | X=405, Y=15 | 360 x 160 | `Total Students Recruited` | `Recruited - BizTown`, `Recruited - CDP`, `Recruited - Pre-App`, `Recruited - SYE` |
| 3 | X=780, Y=15 | 360 x 160 | `Total Students Completed` | `Completed - BizTown`, `Completed - CDP`, `Completed - Pre-App`, `Completed - SYE` |
| 4 | X=1155, Y=15 | 360 x 160 | `Total Schools` | `Schools - BizTown`, `Schools - CDP`, `Schools - Pre-App`, `Schools - SYE` |

Each card shows the org-wide total as the callout value, with 4 reference labels
breaking it down by program beneath.

**For each New Card:**
1. **Insert** > search for **"Card"** > select the **New Card** (it has a sparkline icon; avoid the legacy "Card (classic)")
2. Drag the primary measure into the **Callout value** field
3. Drag all 4 per-program breakdown measures into the **Reference labels** field. They will stack vertically beneath the callout value, each showing its program's contribution to the total. For example, on the "Total Students Served" card:
   - `Served - BizTown`
   - `Served - CDP`
   - `Served - Pre-App`
   - `Served - SYE`
4. **Format** > Callout value:
   - Font: Segoe UI Bold, 28pt
   - Color: `#22404D` (Immersive Blue Black)
   - Display units: None (show actual numbers)
5. **Format** > Cards > Layout > **Callout size %**: Set to ~40% to give reference labels enough room (the 4 program lines need space)
6. **Format** > Reference labels:
   - Font: Segoe UI, 9pt
   - Color: Match each label to its program color for visual scanning:
     - BizTown: `#00A0AF` (Resilient Turquoise)
     - CDP: `#285F74` (Boundless Blue)
     - Pre-App: `#22404D` (Immersive Blue Black)
     - SYE: `#00763D` (Sustainable Green)
   - Use the **Detail label** property on each reference label to add the program name prefix (e.g., "BizTown:", "CDP:", etc.)
7. **Format** > Card background:
   - Color: White
   - Shadow: On, offset 2px, color `#00000015`
   - Border radius: 8px
   - Padding: 12px
8. **Format** > Accent bar: On, position Left, color matching the program palette (optional — adds a colored left border)

> **Why not Multi-row Card?** The Multi-row Card visual is legacy and no longer recommended.
> The old text-concatenation DAX measures (`Students Served Card` etc.) that used
> `UNICHAR(10)` line breaks were a workaround for the old card's limitations. The New Card
> handles subtitles and reference labels natively — no string hacking needed.

#### 1.3 — Clustered Bar Chart (middle-left)

1. **Insert** > **Clustered bar chart**
2. **Position**: X=30, Y=190, Width=930, Height=420
3. **Fields**:
   - **Y-axis**: `dim_Program[Program]` *(must use dim_Program so drill-through works)*
   - **X-axis**: `Program_Summary[Students_Served]`
   - Also add `Students_Recruited` and `Students_Completed` as additional X-axis values to create grouped bars
4. **Format**:
   - Title: "Students by Program" — Segoe UI Semibold, 12pt
   - Legend: On, position Bottom
   - Data colors:
     - Students_Served: `#285F74` (Boundless Blue)
     - Students_Recruited: `#00A0AF` (Resilient Turquoise)
     - Students_Completed: `#8FC440` (Key Lime)
   - X-axis: Show gridlines, light gray
   - Data labels: On, font 9pt
   - Background: White with shadow (same as cards)
   - Padding: 10px all sides
   - **Important**: Ensure the X-axis starts from **zero** — truncated axes exaggerate differences and mislead viewers
5. **Interaction**: Enable drill-through — when a user clicks a bar, it navigates to Page 3 (set up in Page 3 section below)

#### 1.4 — Summary Table / Matrix (middle-right)

1. **Insert** > **Table** (or **Matrix** if you want collapsible rows)
2. **Position**: X=980, Y=190, Width=910, Height=420
3. **Fields** (as columns):
   - `dim_Program[Program]` *(must use dim_Program so drill-through works)*
   - `Program_Summary[Students_Recruited]`
   - `Program_Summary[Students_Served]`
   - `Program_Summary[Students_Completed]`
   - `Program_Summary[Schools_Count]`
   - `Program Completion Rate` (the DAX measure — will show as %)
4. **Format**:
   - Title: "Program Summary" — Segoe UI Semibold, 12pt
   - Style > Preset: **Alternating rows**
   - Header: Segoe UI Semibold, 10pt, background `#285F74` (Boundless Blue), text White
   - Values: Segoe UI, 10pt
   - Grid > Row padding: 6px
   - Totals row: **On** (shows org-wide totals at bottom)
   - Column widths: auto-fit
   - Conditional formatting on Completion Rate:
     - Select the Completion Rate column > **Conditional formatting** > **Background color**
     - Rules: >= 0.80 → Key Lime (`#8FC440`), >= 0.60 → Empowered Yellow (`#E3E24F`), < 0.60 → Red (`#E15759`)
   - Background: White with shadow

#### 1.5 — Demographic & Completion Snapshot (bottom row — 4 New Card visuals)

These cards give leadership a quick demographic and outcome snapshot without
leaving Page 1. Detailed breakdowns live on Page 2 (Demographics).

| Card # | Position (X, Y) | Size (W x H) | Callout Measure | Reference Label |
|--------|----------------|--------------|-----------------|-----------------|
| 1 | X=30, Y=630 | 450 x 120 | `Avg Student Age` | `Age Range` (e.g., "Range: 16–22") |
| 2 | X=500, Y=630 | 450 x 120 | `Pct Female` (as %) | `Pct Male` (as %) |
| 3 | X=970, Y=630 | 450 x 120 | `Org Completion Rate` (as %) | `Served vs Recruited Pct` (as %) |
| 4 | X=1440, Y=630 | 450 x 120 | `Top Race` | `Top City` |

**For each New Card:**
1. **Insert** > **New Card**
2. Drag the callout measure and reference label measure per the table above
3. **Format**:
   - Title: descriptive label (e.g., "Student Age", "Gender", "Completion", "Top Demographics")
   - Callout value: Segoe UI Bold, 24pt, color `#22404D`
   - Reference label: Segoe UI, 10pt, color `#285F74`
   - Background: White with shadow, border radius 8px, padding 12px
   - Accent bar: **On**, position Left, color `#00A0AF` (Resilient Turquoise)
4. **Card 3 conditional formatting** on callout value color:
   - >= 80%: Key Lime (`#8FC440`)
   - >= 60%: Empowered Yellow (`#E3E24F`)
   - < 60%: Red (`#E15759`)

> **Note:** These cards use `USERELATIONSHIP` measures from All_Students (see
> `Master_Dashboard_Guide.md` > Demographic Measures). The `Top Race`, `Top City`,
> and `Age Range` measures are defined in that same section.

---

## Page 2: Demographics

### Layout Grid (1920 x 1080 canvas)

```
+------------------------------------------------------------------------------------+
| [Program Slicer]     [SchoolYear Slicer]                              (top-left)   |
+------------------------------------------------------------------------------------+
| Demographic Note Banner (New Card — full width)                                    |
+-----------------------------+-----------------------------+------------------------+
|                             |                             |                        |
|   Donut Chart               |  Horiz. Bar Chart           | Horiz. Bar Chart       |
|   (Gender)                  |  (Race)                     | (Ethnicity)            |
|                             |                             |                        |
+-----------------------------+-----------------------------+------------------------+
|                             |                             |                        |
|   Bar Chart                 |  Bar Chart                  | Bar Chart              |
|   (City)                    |  (Household Income)         | (Language at Home)     |
|                             |                             |                        |
+-----------------------------+-----------------------------+------------------------+
```

### Step-by-Step Instructions

#### 2.1 — Slicers (top row)

Use the **new Slicer** visual for both (not the legacy slicer).

**Program Slicer:**
1. **Insert** > **Slicer** (new Slicer)
2. **Field**: `All_Students[Program]`
3. **Position**: X=30, Y=15, Width=300, Height=45
4. **Format**: Dropdown style, Multi-select: On

**SchoolYear Slicer:**
1. **Insert** > **Slicer** (new Slicer)
2. **Field**: `dim_Date[SchoolYear]`
3. **Position**: X=350, Y=15, Width=300, Height=45
4. **Format**: Dropdown style, Single select: On

#### 2.2 — Demographic Note Banner (below slicers)

1. **Insert** > **New Card**
2. **Field**: `Demographic Note` (the DAX measure) as the callout value
3. **Position**: X=30, Y=70, Width=1860, Height=60
4. **Format**:
   - Callout value font: Segoe UI, 10pt, color `#22404D` (Immersive Blue Black)
   - Display units: None
   - Background: `#F3F2B3` (Pale Yellow — JA branded info banner)
   - Border: 1px `#E3E24F` (Empowered Yellow)
   - No shadow
   - Accent bar: Off (this is informational, not a KPI)

> **Do not use Multi-row Card** — it is a legacy visual. The New Card handles text
> measures just fine as a callout value.

#### 2.3 — Gender Donut Chart (left)

1. **Insert** > **Donut chart**
2. **Position**: X=30, Y=145, Width=600, Height=400
3. **Fields**:
   - **Legend**: `All_Students[Gender]`
   - **Values**: Count of rows (use `Total Students (Demographics)` or just drag Gender and it auto-counts)
4. **Format**:
   - Title: "Gender Distribution" — Segoe UI Semibold, 12pt
   - Legend: On, position Bottom
   - Data colors: Female `#00A0AF` (Resilient Turquoise), Male `#285F74` (Boundless Blue), Other/Non-binary `#00763D` (Sustainable Green)
   - Detail labels: On, show Category + Percent
   - Inner radius: 60%
   - Background: White with shadow

#### 2.4 — Race Horizontal Bar Chart (center)

1. **Insert** > **Clustered bar chart** (horizontal bars)
2. **Position**: X=650, Y=145, Width=600, Height=400
3. **Fields**:
   - **Y-axis**: `All_Students[Race]`
   - **X-axis**: Count of `All_Students[Race]` (or use `Total Students (Demographics)`)
4. **Format**:
   - Title: "Race" — Segoe UI Semibold, 12pt
   - Sort: Descending by count
   - Data colors: All bars `#285F74` (Boundless Blue)
   - Data labels: On, show value
   - Y-axis: Category labels, Segoe UI 9pt
   - Background: White with shadow

#### 2.5 — Ethnicity Horizontal Bar Chart (right)

1. **Insert** > **Clustered bar chart** (horizontal bars)
2. **Position**: X=1270, Y=145, Width=620, Height=400
3. **Fields**:
   - **Y-axis**: `All_Students[Ethnicity]`
   - **X-axis**: Count of `All_Students[Ethnicity]`
4. **Format**:
   - Title: "Ethnicity" — Segoe UI Semibold, 12pt
   - Sort: Descending by count
   - Data colors: All bars `#00A0AF` (Resilient Turquoise)
   - Data labels: On
   - Background: White with shadow

#### 2.6 — City Bar Chart (bottom-left)

1. **Insert** > **Clustered bar chart**
2. **Position**: X=30, Y=560, Width=600, Height=400
3. **Fields**:
   - **Y-axis**: `All_Students[City]`
   - **X-axis**: Count of rows
4. **Format**:
   - Title: "Students by City" — Segoe UI Semibold, 12pt
   - Sort: Descending by count
   - Show top N: **10** (to avoid clutter if many cities)
   - Data colors: `#22404D` (Immersive Blue Black)
   - Data labels: On
   - Background: White with shadow

#### 2.7 — Household Income Bar Chart (bottom-center)

1. **Insert** > **Clustered bar chart**
2. **Position**: X=650, Y=560, Width=600, Height=400
3. **Fields**:
   - **Y-axis**: `All_Students[Household Income]`
   - **X-axis**: Count of rows
4. **Format**:
   - Title: "Household Income" — Segoe UI Semibold, 12pt
   - Sort: by income bracket order (you may need a sort column)
   - Data colors: `#00763D` (Sustainable Green)
   - Data labels: On
   - Background: White with shadow

#### 2.8 — Language at Home Bar Chart (bottom-right)

1. **Insert** > **Clustered bar chart**
2. **Position**: X=1270, Y=560, Width=620, Height=400
3. **Fields**:
   - **Y-axis**: `All_Students[Language at Home]`
   - **X-axis**: Count of rows
4. **Format**:
   - Title: "Language at Home" — Segoe UI Semibold, 12pt
   - Sort: Descending by count
   - Data colors: `#8FC440` (Key Lime)
   - Data labels: On
   - Background: White with shadow

---

## Page 3: Program Drill-Through

### Setup as Drill-Through Page

1. Create a new page, rename it "Program Detail"
2. In the **Visualizations** pane > **Drill through** section:
   - Drag `dim_Program[Program]` into the drill-through field
   - This allows users to right-click a program on Page 1 and drill through
3. Power BI auto-creates a **Back button** in the top-left — keep it there

### Layout Grid

```
+---------------------------------------------------------------+
| [<- Back]  Program: {Selected Program Name}                   |
+-------------------------------+-------------------------------+
|                               |                               |
|   Program-Specific Visual     |  Program-Specific Visual      |
|   (changes based on program)  |  (changes based on program)   |
|                               |                               |
+-------------------------------+-------------------------------+
|                                                               |
|   Detail Table (program-specific data)                        |
|                                                               |
+---------------------------------------------------------------+
```

### 3.1 — Dynamic Program Title

1. **Insert** > **New Card**
2. **Callout value**: `SELECTEDVALUE(dim_Program[Program], "All Programs")`
   - Create this as a quick DAX measure:
   ```
   Selected Program = SELECTEDVALUE(dim_Program[Program], "All Programs")
   ```
3. **Position**: X=80, Y=15, Width=700, Height=55
4. **Format**: Callout value — Segoe UI Bold, 22pt, no background, left-aligned, accent bar Off

### 3.2 — Pre-Apprenticeship View

These visuals are visible when "Pre-Apprenticeship" is the drill-through context. Use visual-level filters to show/hide based on program.

**Pipeline Funnel:**

> **Approach:** Use a static calculated table for stage names + a dynamic measure for counts.
> This avoids the UNION/ROW approach which can cause refresh issues.

**Step 1 — Create `Pipeline Stages` calculated table** (Model view > New Table):
```
Pipeline Stages =
DATATABLE(
    "Stage", STRING,
    "SortOrder", INTEGER,
    {
        {"Enrolled", 1},
        {"Onboarded", 2},
        {"Soft Skills Complete", 3}
    }
)
```
Then in **Data view**, select the `Stage` column > Column tools > **Sort by Column** > `SortOrder`.

**Step 2 — Create `Pipeline Count` measure:**
```
Pipeline Count =
VAR _CurrentStage = SELECTEDVALUE('Pipeline Stages'[Stage])
RETURN SWITCH(
    _CurrentStage,
    "Enrolled", COUNTROWS(All_Students),
    "Onboarded", CALCULATE(COUNTROWS(All_Students), All_Students[Status] = "ACTIVE"),
    "Soft Skills Complete", CALCULATE(COUNTROWS(All_Students), All_Students[IsCompleted] = TRUE()),
    BLANK()
)
```

**Step 3 — Build the funnel visual:**
1. **Insert** > **Funnel chart**
2. **Position**: X=20, Y=60, Width=600, Height=350
3. **Fields**:
   - **Category**: `Pipeline Stages[Stage]`
   - **Values**: `[Pipeline Count]`
4. **Visual-level filter**: `All_Students[Program]` = "Pre-Apprenticeship"
5. **Format**: Title "Pre-App Pipeline", gradient colors from light to dark blue

**KPI Sidebar (right side) — 3 New Card visuals stacked vertically:**

> These cards are only relevant when the drill-through context is Pre-Apprenticeship.
> They all use `All_Students` filtered by the drill-through (which passes `dim_Program[Program]`).
> No additional visual-level filter is needed — drill-through handles it.

| Card # | Position (X, Y) | Size (W x H) |
|--------|----------------|--------------|
| 1 | X=660, Y=80 | 280 x 120 |
| 2 | X=660, Y=210 | 280 x 120 |
| 3 | X=660, Y=340 | 280 x 120 |

**Card 1 — Active Students**
1. **Insert** > **New Card**
2. **Callout value**: Create this measure:
   ```
   PreApp Active Students =
   CALCULATE(
       COUNTROWS(All_Students),
       All_Students[Status] = "ACTIVE"
   )
   ```
3. **Reference label**: Use `[Total Students Served]` with custom label text: `"of {value} enrolled"`
   - This shows context like "of 280 enrolled" so leadership sees the active-to-enrolled ratio
4. **Format**:
   - Title: "Active Students" — Segoe UI Semibold, 11pt
   - Callout value: Segoe UI Bold, 28pt, color `#22404D`
   - Accent bar: **On**, position Left, color `#22404D` (Immersive Blue Black)
   - Background: White with shadow

**Card 2 — Placement Rate**
1. **Insert** > **New Card**
2. **Callout value**: Create this measure:
   ```
   PreApp Placement Rate =
   DIVIDE(
       CALCULATE(COUNTROWS(All_Students), All_Students[IsCompleted] = TRUE()),
       COUNTROWS(All_Students),
       0
   )
   ```
   Format as **Percentage** (1 decimal).
3. **Reference label**: Create a static measure:
   ```
   Placement Target Label = "Target: 80%"
   ```
4. **Format**:
   - Title: "Placement Rate" — Segoe UI Semibold, 11pt
   - Callout value: Segoe UI Bold, 28pt
   - Accent bar: **On**, position Left, color `#22404D`
   - Background: White with shadow
   - **Conditional formatting** on callout value color:
     - >= 80%: Key Lime (`#8FC440`)
     - >= 60%: Empowered Yellow (`#E3E24F`)
     - < 60%: Red (`#E15759`)

**Card 3 — Avg Knowledge Gained**

> **Prerequisite:** The `Knowledge Gained %` column exists in the Pre-App source
> but is **not** currently included in `All_Students`. You must add it:
> 1. In Power Query, open `stg_PreApp`
> 2. In the `#"Selected"` step, add `"Knowledge Gained %"` to the column list
> 3. In `stg_SYE`, add a null column: `Table.AddColumn(..., "Knowledge Gained %", each null, type number)`
>    so the append still works
> 4. In `All_Students`, the column will flow through automatically

1. **Insert** > **New Card**
2. **Callout value**: Create this measure:
   ```
   Avg Knowledge Gained =
   AVERAGE(All_Students[Knowledge Gained %])
   ```
   Format as **Percentage** (1 decimal).
3. **Reference label**: Create a static measure:
   ```
   Knowledge Benchmark Label = "Benchmark: 70%"
   ```
   *(Adjust the benchmark to whatever your program target is.)*
4. **Format**:
   - Title: "Avg Knowledge Gained" — Segoe UI Semibold, 11pt
   - Callout value: Segoe UI Bold, 28pt
   - Accent bar: **On**, position Left, color `#22404D`
   - Background: White with shadow
   - **Conditional formatting** on callout value color:
     - >= 70%: Key Lime (`#8FC440`)
     - >= 50%: Empowered Yellow (`#E3E24F`)
     - < 50%: Red (`#E15759`)

### 3.3 — SYE/Career Bound View

**Placement Metrics New Cards (top):**
1. Create **New Card** visuals for: Interns Placed (ref label: "Target: 355"), Placement Rate (ref label: "of enrolled"), New Businesses
2. Arrange in a row at the top, same styling as Page 1 cards

**Satisfaction Score New Card:**
1. **Insert** > **New Card**
2. Position below the placement cards
3. Callout value: Average satisfaction score (if brought into the model)
4. Reference label: "Target: 4.0 / 5.0"
5. Use conditional formatting on callout color: >= 4.0 green, >= 3.0 amber, < 3.0 red

### 3.4 — ELC Programs View (CDP + BizTown)

**School-Level Table:**
1. **Insert** > **Table**
2. **Position**: X=30, Y=80, Width=1860, Height=900
3. **Fields**:
   - School Name
   - Students Recruited (Projected/Pending)
   - Students Verified
   - Difference (Recruited - Verified)
4. This requires adding school-level detail to the model (optional — pull from ELC staging queries)
5. **Conditional formatting**: highlight rows where Verified < Recruited in yellow
6. **Format**: Alternating rows, header `#285F74` (Boundless Blue) with white text

### Drill-Through Tip
On **Page 1**, users right-click a program bar in the clustered bar chart > **Drill through** > **Program Detail**. This automatically filters Page 3 to that program. The Back button returns them to Page 1.

> **Important:** For drill-through to appear in the right-click menu, Page 1 visuals
> must use `dim_Program[Program]` on their axis/fields — NOT `Program_Summary[Program]`.
> The drill-through field on Page 3 is `dim_Program[Program]`, so Power BI only offers
> the option when it finds that same field in the visual you're clicking on.

---

## Cross-Page Interaction Settings

### Slicer Sync
1. Go to **View** > **Sync slicers**
2. Set the `SchoolYear` slicer to sync across Page 1 and Page 2
3. Do NOT sync it to Page 3 (drill-through manages its own filters)

### Edit Interactions (Page 1)
By default, clicking the bar chart filters the table and completion cards. To control this:
1. Select the bar chart on Page 1
2. Go to **Format** > **Edit interactions**
3. Set the Summary Table to **Filter** (clicking a bar filters the table)
4. Set the Completion Rate cards to **None** (each card already has its own program filter)

---

## Mobile Layout

Create a mobile-optimized layout for each page. Go to **View > Mobile layout**.

### General mobile principles:
- Arrange visuals in a **single column**, KPIs at the top
- Do not just shrink the desktop layout — redesign for vertical scrolling
- All interactive elements must be at least **44x44 px** for touch targets
- Use the **auto-create layout** feature as a starting point, then refine

### Page 1 mobile layout (suggested order):
1. SchoolYear slicer (full width, dropdown)
2. 4 KPI New Cards (stacked vertically, full width)
3. Summary Table/Matrix (full width, scrollable)
4. Clustered bar chart (full width)
5. 4 Completion Rate cards (stacked vertically)

### Page 2 mobile layout (suggested order):
1. Program + SchoolYear slicers (stacked)
2. Demographic Note banner (full width)
3. Gender donut chart
4. Race bar chart
5. Ethnicity bar chart
6. City, Income, Language charts (stacked)

> **Tip:** If Page 2 feels too long on mobile, use the **Field Parameter** (see
> `Master_Dashboard_Guide.md` > Field Parameters section) to consolidate the 6
> demographic charts into 1 chart with a dimension picker slicer.

---

## Publishing Checklist

1. [ ] All visuals render with correct data
2. [ ] SchoolYear slicer filters Page 1 and Page 2
3. [ ] Program slicer on Page 2 filters all demographic charts
4. [ ] Drill-through from Page 1 bar chart to Page 3 works
5. [ ] Back button on Page 3 returns to Page 1
6. [ ] Conditional formatting on completion rate shows correct colors (JA branded)
7. [ ] Demographic Note banner displays accurate coverage message
8. [ ] All card values show actual numbers (not abbreviated like "1K")
9. [ ] Alt text is set on every visual
10. [ ] Tab order is set in Selection pane for keyboard navigation
11. [ ] Run the built-in **Accessibility Checker** (Review tab) — fix all issues
12. [ ] Test keyboard navigation (Tab/Enter/Escape/arrows) on all pages
13. [ ] Mobile layout created and tested for each page
14. [ ] JSON theme file applied (JA South Florida branding — consistent fonts, colors, shadows)
15. [ ] Field Parameters working on Page 2 (if implemented)
16. [ ] Run **Performance Analyzer** (View tab) — verify no visual takes >3 seconds
17. [ ] Report pages named descriptively (not "Page 1", "Page 2")
18. [ ] No more than 8 widget visuals per page
19. [ ] Publish to Power BI Service and verify scheduled refresh
20. [ ] Configure data gateway / scheduled refresh in Service
21. [ ] If Copilot is enabled, test RLS boundaries — Copilot can surface data outside RLS in some scenarios

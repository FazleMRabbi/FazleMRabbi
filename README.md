# Power BI Executive Dashboard — PBIP

A single-page executive dashboard built in **PBIP (Power BI Project)** format, designed for clean, modern, executive-level reporting. Connects directly to your existing `Demo.SemanticModel`.

---

## Dashboard Layout (1280 × 720 — 16:9 Widescreen)

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│  EXECUTIVE DASHBOARD                                         [ Fiscal Year ▼ ]   │
│  Real-Time Performance Overview | Powered by Demo Model                          │
├──────────────┬──────────────────┬───────────────────────┬────────────────────────┤
│  Total       │  Total           │  Gross                │  Total                 │
│  Revenue     │  Orders          │  Profit %             │  Customers             │
│  $XX.XM  🔵  │  XX.XK     🟢   │  XX.X%          🟡   │  XX.XK           🟣   │
├──────────────┴──────────────────┴───────────────────────┴────────────────────────┤
│  Monthly Revenue Trend (Line Chart)          │  Revenue by Category (Bar Chart)  │
│                                              │                                   │
│  [──────────────────── smooth line ────────] │  [████████ Bikes         $8.5M]   │
│                                              │  [█████    Accessories   $3.2M]   │
│                                              │  [███      Clothing      $2.1M]   │
│  Jan Feb Mar Apr May Jun Jul Aug Sep Oct...  │  [██       Components    $1.8M]   │
├──────────────────────────────────────────────┴───────────────────────────────────┤
│  Top 10 Products by Revenue (Table)          │  Revenue by Territory (Bar Chart) │
│                                              │                                   │
│  Product Name    │ Category  │ Revenue │ GP% │  [████████ North America  $9.2M]  │
│  ────────────────┼───────────┼─────────┼─────│  [██████   Europe        $6.8M]   │
│  Mountain-200    │ Bikes     │ $1.4M   │ 41% │  [████     Pacific       $4.1M]   │
│  Road-150        │ Bikes     │ $1.1M   │ 39% │  [███      Southwest     $3.2M]   │
│  ...             │ ...       │ ...     │ ... │  [██       Southeast     $2.9M]   │
└──────────────────────────────────────────────┴───────────────────────────────────┘
```

---

## Files Created

```
ExecutiveDashboard.pbip                            ← Open this in Power BI Desktop
ExecutiveDashboard.Report/
├── definition.pbir                                ← Points to Demo.SemanticModel
└── definition/
    ├── report.json                                ← Report-level settings & theme
    └── pages/
        └── ExecutiveDashboard/
            ├── page.json                          ← 1280×720 canvas, custom BG
            └── visuals/
                ├── bg_header/visual.json          ← Dark navy header bar
                ├── txt_title/visual.json          ← Title + subtitle textbox
                ├── slicer_year/visual.json        ← Year dropdown slicer
                ├── kpi_revenue/visual.json        ← Total Revenue card (blue)
                ├── kpi_orders/visual.json         ← Total Orders card (green)
                ├── kpi_gp/visual.json             ← Gross Profit % card (amber)
                ├── kpi_customers/visual.json      ← Total Customers card (purple)
                ├── chart_revenue_trend/           ← Monthly trend line chart
                ├── chart_sales_category/          ← Revenue by Category bar chart
                ├── table_top_products/            ← Top 10 Products table
                └── chart_territory/              ← Revenue by Territory bar chart
```

---

## Setup Instructions

### Step 1 — Copy to C:\Temp2
Place the entire project so the folder structure is:
```
C:\Temp2\
├── Demo.SemanticModel\          ← your existing model (already here)
├── Demo.pbip                    ← your existing project file
├── ExecutiveDashboard.Report\   ← new report folder (copy from this repo)
└── ExecutiveDashboard.pbip      ← new project file (copy from this repo)
```

### Step 2 — Verify the Semantic Model Path
Open `ExecutiveDashboard.Report\definition.pbir` and confirm the path matches your model folder name:
```json
{
  "version": "4.0",
  "datasetReference": {
    "byPath": {
      "path": "../Demo.SemanticModel"
    }
  }
}
```
If your model folder is named differently (e.g., `SalesDemo.SemanticModel`), update `path` accordingly.

### Step 3 — Open in Power BI Desktop
Double-click `ExecutiveDashboard.pbip` to open directly in Power BI Desktop. The report will auto-connect to the live semantic model.

---

## Field Mapping Reference

The dashboard references these tables and fields. Map them to your actual model:

| Visual | Table | Field/Measure | Expected Name |
|---|---|---|---|
| KPI Card 1 | `Sales` | Measure | `Total Revenue` |
| KPI Card 2 | `Sales` | Measure | `Order Count` |
| KPI Card 3 | `Sales` | Measure | `Gross Profit %` |
| KPI Card 4 | `Sales` | Measure | `Total Customers` |
| Line Chart (Axis) | `Date` | Column | `Month` |
| Line Chart (Value) | `Sales` | Measure | `Total Revenue` |
| Bar Chart (Axis) | `Product` | Column | `Category` |
| Bar Chart (Value) | `Sales` | Measure | `Total Revenue` |
| Table (Col 1) | `Product` | Column | `Product Name` |
| Table (Col 2) | `Product` | Column | `Category` |
| Table (Col 3) | `Sales` | Measure | `Total Revenue` |
| Table (Col 4) | `Sales` | Measure | `Order Count` |
| Table (Col 5) | `Sales` | Measure | `Gross Profit %` |
| Territory Bar (Axis) | `Territory` | Column | `Region` |
| Territory Bar (Value) | `Sales` | Measure | `Total Revenue` |
| Slicer | `Date` | Column | `Year` |

### Adapting to Different Field Names
If your model uses different names, open each `visual.json` file and update:
- `"Entity"`: the table name
- `"Property"`: the column or measure name
- `"queryRef"`: should match `"TableName.FieldName"`

---

## Design Specifications

### Color Palette
| Element | Hex | Usage |
|---|---|---|
| `#0F1C35` | Dark Navy | Header background |
| `#1B3A5C` | Deep Blue | KPI card backgrounds |
| `#EEF2F7` | Light Blue-Gray | Page background |
| `#FFFFFF` | White | Chart & table backgrounds |
| `#00A8E8` | Bright Blue | Revenue KPI accent border |
| `#10B981` | Emerald Green | Orders KPI accent border |
| `#F59E0B` | Amber | Gross Profit % KPI accent border |
| `#8B5CF6` | Purple | Customers KPI accent border |
| `#0078D4` | Microsoft Blue | Chart bars / line |
| `#D5E1EC` | Light Steel | Chart card borders |
| `#94B3CC` | Muted Blue | Subtitles / axis labels |

### Layout Grid
- **Canvas:** 1280 × 720 px (16:9)
- **Header:** Full-width, 60px height
- **KPI Row:** y=70, 126px tall, 4 equal cards
- **Chart Row:** y=206, 228px tall, 778px + 472px split
- **Bottom Row:** y=444, 266px tall, same split
- **Padding:** 10px between all elements

---

## Performance Best Practices Applied

- **Top N filter** on Products table (shows top 10 only) — reduces query load
- **Display units** pre-set to M/K so Power BI doesn't compute full precision in labels
- **Smooth line** enabled for trend chart without data point markers (faster render)
- **Grid lines** styled to light grey (#E5EBF0) — reduces visual noise
- **Single-page design** — no cross-page navigation overhead
- **Year slicer** at header level for fast filtering across all visuals simultaneously
- **Table sorted descending by Revenue** at query level — no client-side sort needed

---

## Customization Tips

1. **Add YoY comparison** — add a second line to the trend chart using a `Total Revenue LY` measure on the `Y2` axis (already scaffolded, set `"active": false` to `true`)
2. **Add conditional formatting** to the GP% table column using Power BI Desktop's Format pane
3. **Drill-through** — right-click any product/category to add a drill-through page for deeper analysis
4. **Mobile layout** — use Power BI Desktop's Mobile Layout view to create a phone-optimised version from the same page

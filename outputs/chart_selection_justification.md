# Chart Selection Justification

For each major chart in the executive dashboard: the question it answers, why the chart type fits, which fields drive color/size/label/filter, the design principle applied, and the mistake it avoids.

---

## 1. Sales Trend View — Line Chart
- **Question answered:** How are sales (and profit) changing over time?
- **Why this chart type:** Line charts are the standard for showing a continuous measure across a continuous time axis — the human eye reads slope and direction more easily on a line than on a sequence of bars.
- **Fields used:** Columns = `MONTH(Order Date)` (continuous); Rows = `SUM(Sales)` and `SUM(Profit)` as a synchronized dual-axis.
- **Design principle applied:** Consistency — using a continuous (not discrete) date field avoids gaps and keeps the time axis evenly spaced.
- **Mistake avoided:** Not truncating the Y-axis to exaggerate small fluctuations — the axis starts at zero so month-to-month changes are shown at true scale, not amplified.

## 2. Regional Performance View — Bar Chart
- **Question answered:** Which region performs best on sales and profit?
- **Why this chart type:** A sorted bar chart makes ranking immediately readable, which a map would not (a map is better for spotting geographic clusters, not for precise ranking — and this dataset lacks a Country field for reliable geocoding).
- **Fields used:** Rows = `Region`; Columns = `SUM(Sales)`; Color = `Region` (reused consistently across the whole dashboard); Label = `Profit Margin`.
- **Design principle applied:** Sorting — bars are ordered descending by sales so the ranking is the first thing read, not something the viewer has to compute.
- **Mistake avoided:** Avoided a pie chart, which would make it hard to compare four close values (South, North, West, East are within a relatively narrow range) — bars handle close values far better than angles/areas.

## 3. Category Profitability View — Bar Chart
- **Question answered:** Which categories and sub-categories drive profit, and which drag it down?
- **Why this chart type:** A bar chart sorted ascending by profit puts the weakest performers (Furniture sub-categories) at one end, turning the chart into a direct answer rather than a lookup table.
- **Fields used:** Rows = `Sub-Category` nested under `Category`; Columns = `SUM(Profit)`; Color = `Category`; Label = `Profit Margin`.
- **Design principle applied:** Hierarchy — nesting Sub-Category under Category preserves the natural grouping so viewers can read both the category-level and sub-category-level story in one view.
- **Mistake avoided:** Avoided a treemap, which would show area (size) well but make precise profit comparison between similarly-sized sub-categories difficult — bars keep the comparison precise.

## 4. Customer Segment View — Highlight Table
- **Question answered:** How does performance compare across customer segments?
- **Why this chart type:** A highlight table lets five different metrics (Sales, Profit, Margin, AOV, Return Rate) be compared side-by-side for three segments in one compact view — a single bar chart cannot hold five measures cleanly.
- **Fields used:** Rows = `Customer Segment`; Columns = Measure Names/Values for Sales, Profit, Profit Margin, Average Order Value, Return Rate; Color = applied to the Return Rate column to flag Home Office's elevated rate.
- **Design principle applied:** Focused color use — color is applied only to the metric that needs attention (Return Rate), not uniformly across the table, so the eye goes straight to the anomaly.
- **Mistake avoided:** Avoided cramming all five metrics into one bar chart with multiple axes, which becomes unreadable — a table format scales better for multi-metric comparison.

## 5. Shipping Performance View — Bar Chart
- **Question answered:** Which shipping mode is slowest, and does that relate to returns?
- **Why this chart type:** A simple bar chart ranks the four shipping modes by average delivery time directly; color overlays the return-rate dimension without needing a second chart.
- **Fields used:** Rows = `Ship Mode`; Columns = `AVG(Delivery Days)`; Color = `Return Rate` (diverging scale).
- **Design principle applied:** Avoidance of false correlation — the diverging color scale on top of the delivery-time bars actually reveals that return rate does *not* increase monotonically with delivery time, preventing a misleading "slower shipping = more returns" takeaway.
- **Mistake avoided:** Avoided assuming and visually implying a linear relationship between two variables that the data doesn't support.

## 6. Discount vs Profit View — Scatter Plot
- **Question answered:** How does discount level relate to profit?
- **Why this chart type:** Scatter plots are the correct choice whenever the question is about the relationship between two continuous measures — this is the one case in the dashboard where a bar or line chart would actively obscure the pattern.
- **Fields used:** Columns = `Discount`; Rows = `SUM(Profit)`; Color = `Category`; Size = `SUM(Sales)`; a reference line at Profit = 0 marks the loss boundary.
- **Design principle applied:** Reference lines for interpretation — the zero-profit reference line turns "some dots are low" into a clear, labeled boundary between profitable and loss-making orders.
- **Mistake avoided:** Avoided binning discount into a bar chart at this stage, which would have hidden the cliff-like (non-linear) drop in profit beyond 25% discount that the raw scatter reveals.

## 7. Return Analysis View — Bar Chart
- **Question answered:** Which sub-categories carry the highest return risk?
- **Why this chart type:** Ranking return rate across 13 sub-categories is a comparison task — bars sorted descending make the worst offenders (Furniture's Bookcases, Furnishings, Chairs) immediately visible.
- **Fields used:** Rows = `Sub-Category`; Columns = `Return Rate`; Color = `Category`.
- **Design principle applied:** Minimal clutter — only the fields needed to answer the question are on the shelf; no extra dimensions are layered in that would dilute the focus on return rate.
- **Mistake avoided:** Avoided showing return *count* instead of return *rate* — raw counts would just track order volume rather than revealing which sub-categories are disproportionately returned.

## General Principles Applied Across the Dashboard
- **One consistent color palette** for Region and Category is reused on every sheet that uses those fields, so the same color always means the same thing.
- **At least 3 interactive filters** (Region, Category, Customer Segment) are applied dashboard-wide via "Apply to Worksheets > All Using This Data Source," not per-sheet, so they behave predictably.
- **One filter action** (clicking a Category Profitability bar filters all other views) gives leadership a way to drill from "what's underperforming" straight into "show me everywhere that shows up."
- **No 3D charts, no unnecessary dual-axis distortion, and no truncated axes** anywhere in the dashboard — every visual encodes the data at true scale.

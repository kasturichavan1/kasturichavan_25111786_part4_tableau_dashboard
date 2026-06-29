# Business Insights

Dataset: `dashboard_sales_data.xlsx` | 4,200 orders | Jan 2024 – Dec 2025 | Total sales ₹21.70 Cr | Total profit ₹3.33 Cr (15.35% margin)

---

## Calculated Fields

| Field | Formula | Why it's needed |
|---|---|---|
| **Profit Margin** | `SUM([Profit]) / SUM([Sales])` | Raw profit alone doesn't reveal efficiency — a category can have high profit and a poor margin if sales are huge. Aggregating SUM/SUM (rather than dividing row-by-row) keeps the ratio mathematically correct no matter which dimension it's sliced by. |
| **Cost** | `[Sales] - [Profit]` | Sales and Profit are given, but leadership also asks about cost base directly; this avoids them doing mental subtraction on every chart. |
| **Average Order Value** | `SUM([Sales]) / COUNTD([Order ID])` | Total sales alone hides whether growth comes from more orders or bigger orders. AOV is calculated per order (the natural transaction unit in this dataset). |
| **Return Rate** | `SUM([Return Flag]) / COUNTD([Order ID])` | `return_flag` is a 1/0 indicator per order; dividing by distinct order count converts it into a comparable rate across any segment, region, or category. |
| **Shipping Delay Bucket** | `IF [Delivery Days] <= 1 THEN "Fast (0-1 days)" ELSEIF [Delivery Days] <= 3 THEN "Standard (2-3 days)" ELSEIF [Delivery Days] <= 5 THEN "Slow (4-5 days)" ELSE "Very Slow (6+ days)" END` | Raw delivery days (0–9) is too granular to read as a pattern on a chart. Bucketing into four meaningful tiers makes the shipping-mode comparison legible. |
| **Discount Tier** *(extra)* | `IF [Discount] = 0 THEN "No Discount" ELSEIF [Discount] <= 0.1 THEN "Low (1-10%)" ELSEIF [Discount] <= 0.2 THEN "Medium (11-20%)" ELSE "High (21%+)" END` | The discount-vs-profit relationship isn't linear — it falls off a cliff above ~25%. Bucketing isolates the danger zone instead of burying it in a continuous scale. |
| **Order Outcome** *(extra)* | `IF [Profit] >= 0 THEN "Profitable" ELSE "Loss" END` | Needed to directly count and isolate the 288 loss-making orders for root-cause analysis (see Insight 8). |

---

## Insights

### 1. Sales Trend
**Observation:** Monthly sales oscillate in a band rather than trending steadily upward.
**Data evidence:** Monthly sales range from a low of ₹63.0 lakh (Aug 2024) to a high of ₹1.09 Cr (Aug 2025) — the same calendar month, a 72% year-over-year jump. Order volume stays relatively stable at 140–203 orders/month throughout.
**Business interpretation:** Revenue growth is not consistent or campaign-driven on a predictable cadence; the Aug 2025 spike suggests something specific (a promotion, channel shift, or seasonal demand) drove an unusual jump that hasn't repeated.
**Recommended action:** Investigate what changed in Aug 2025 (campaign mix, discount levels, category mix) and test whether it can be deliberately replicated; set monthly sales targets with alerts for dips below ₹70 lakh.

### 2. Regional Performance
**Observation:** South leads on volume, but margins are tightly clustered across all four regions — the real disparity is at the state level.
**Data evidence:** South generates ₹6.47 Cr in sales and ₹1.00 Cr profit, ahead of North (₹5.46 Cr), West (₹4.89 Cr), and East (₹4.89 Cr). Margins are close (14.7%–15.6%). But within states, Rajasthan alone contributes ₹2.08 Cr in sales (38% of North's total) at a 15.8% margin, while Jharkhand (16.8%), Karnataka (16.6%), and Haryana (16.2%) post the highest margins despite modest volume, and Maharashtra (14.1%) and Uttar Pradesh (14.5%) lag both peers.
**Business interpretation:** Region-level totals mask a single dominant state (Rajasthan) and hide margin-efficient states that could be scaled.
**Recommended action:** Study what's working in Rajasthan, Jharkhand, and Karnataka (product mix, pricing) and test applying it in Maharashtra and Uttar Pradesh, which have volume but underperforming margin.

### 3. Category / Sub-Category Profitability
**Observation:** Furniture is a high-revenue, low-margin category — the clear profitability laggard.
**Data evidence:** Technology drives 71% of total sales (₹15.4 Cr) at an 18.2% margin. Furniture brings in ₹5.16 Cr in sales but only a 6.9% margin — less than half the company average of 15.35%. Every Furniture sub-category sits below 8.5% margin (Bookcases 5.7%, Tables 5.7%, Furnishings 7.9%, Chairs 8.2%), versus 14.2%–18.5% for Office Supplies and Technology sub-categories.
**Business interpretation:** Furniture's scale (it's the #2 category by sales) makes its weak margin a material drag on overall profitability, not a rounding error.
**Recommended action:** Audit Furniture's cost base and discounting policy specifically; consider a separate, tighter discount ceiling for this category alone.

### 4. Customer Segment Behavior
**Observation:** Home Office is the largest segment by sales but disproportionately return-prone.
**Data evidence:** Home Office generates ₹7.45 Cr in sales (the highest of the three segments) with a return rate of 5.67%, versus 3.91% for Consumer and 4.00% for Corporate — roughly 45% higher than the other two segments — despite all three segments sharing nearly identical profit margins (15.1%–15.5%).
**Business interpretation:** Home Office's strong revenue contribution is being partially offset by return-handling costs that don't yet show up in the profit figure (which only reflects the original sale, not return-processing overhead).
**Recommended action:** Run a root-cause review of Home Office returns (product fit, expectations, packaging) since this segment has the most volume to lose or protect.

### 5. Discount Impact
**Observation:** Discounting above ~25% reliably destroys profitability.
**Data evidence:** Profit margin falls from 20.6% at 0% discount to -5.0% at 35% discount (correlation between discount and order-level margin: -0.60). Orders discounted 25% or higher account for 250 of the 288 loss-making orders in the dataset (87%), despite being a minority of total order volume.
**Business interpretation:** Discounting is currently being applied without a profitability guardrail — the data shows a clear cliff, not a gentle slope.
**Recommended action:** Cap standard discounts at 20% and require explicit approval for anything higher; the data supports this as the point where margin protection should kick in.

### 6. Shipping / Delivery Performance
**Observation:** Delivery speed is not the primary driver of returns — contrary to the intuitive assumption.
**Data evidence:** Standard Class handles 58% of orders (2,435) with the slowest average delivery (4.71 days) but a mid-range 4.60% return rate. Same Day delivery is fastest (0.40 days) and has the lowest return rate (2.49%) — as expected — but the "Standard (2-3 days)" delivery bucket actually has the *highest* return rate of all buckets (5.09%), higher than the slowest "Very Slow (6+ days)" bucket (4.80%).
**Business interpretation:** Returns are likely driven more by what's being shipped (category, segment) than by how long it took to arrive — the relationship isn't monotonic.
**Recommended action:** Before reallocating logistics budget toward faster shipping as a returns-reduction lever, cross-check category/segment mix within each shipping bucket to isolate the real driver.

### 7. Return Pattern
**Observation:** Returns concentrate heavily in Furniture sub-categories.
**Data evidence:** Bookcases (8.42%), Furnishings (8.16%), and Chairs (7.99%) have return rates roughly 2–3x higher than Technology sub-categories like Copiers (2.72%) and Accessories (3.01%).
**Business interpretation:** The same category (Furniture) shows up as both the weakest on margin and the highest on returns — these two problems are compounding, not independent.
**Recommended action:** Prioritize a Furniture-specific quality/packaging/description audit; even a small return-rate reduction there has an outsized effect given the already-thin margins.

### 8. Business Risk or Opportunity
**Observation:** The riskiest pocket of the business is the overlap of Furniture + high discount + Home Office; the biggest efficient-growth opportunity is the Organic acquisition channel.
**Data evidence:** 93% of all loss-making orders (269 of 288) are Furniture, and discounts of 25%+ drive the majority of those losses. Meanwhile, the Organic campaign channel drives 1,694 orders (40% of all orders) and ₹8.88 Cr in sales at a healthy 15.1% margin and the highest AOV (₹52,411) of any channel except Referral — outperforming Paid (₹47,245 AOV) without any acquisition spend.
**Business interpretation:** There's a concentrated, fixable risk (Furniture discounting) and a proven, low-cost growth lever (Organic) sitting side by side in the same dataset.
**Recommended action:** Tighten discount governance specifically for Furniture; simultaneously increase investment in whatever drives Organic traffic (SEO, content, referral programs) since it's already outperforming paid acquisition on both volume and order value.

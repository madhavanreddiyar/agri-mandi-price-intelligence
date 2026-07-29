# Agri-Market Price Intelligence & Farmer Advisory System

**A data pipeline and dashboard analyzing 100,000+ real government agricultural price records to identify pricing patterns, test policy impact, and generate concrete farmer market-selection recommendations.**

---

## Problem Statement

Indian farmers routinely sell their crops at the nearest local mandi (wholesale market), often without visibility into whether nearby markets are paying significantly better prices for the same commodity. This project uses daily government price data across Indian mandis to quantify these price gaps, test whether government Minimum Support Price (MSP) policy actually stabilizes prices, and produce concrete, ₹-quantified recommendations for where farmers could sell for more.

## Data Source

- **Source:** Agmarknet (Government of India, Ministry of Agriculture & Farmers Welfare), accessed via data.gov.in
- **Coverage:** 5 commodities (Onion, Tomato, Wheat, Potato, Bengal Gram) across multiple states (Maharashtra, Karnataka, Madhya Pradesh, Punjab, Uttar Pradesh, West Bengal) and their districts
- **Granularity:** Daily mandi-level price reports (Min, Max, Modal Price)
- **Time range:** 2023–2025
- **Final dataset size:** 109,324 cleaned records (after removing 13 data-entry errors, including one extreme outlier)

## Method

**1. Data Ingestion & Cleaning — PySpark on Databricks**
- Combined 40+ separately downloaded CSV files (per commodity/state/district) into a single dataset using PySpark on Databricks
- Removed records with missing or zero prices
- Standardized inconsistent text formatting across state, district, market, and commodity names
- Removed exact duplicate records
- Identified and removed a data entry error where one onion price record was recorded at ₹1.6M/quintal (~1000x a realistic price) — added a sanity-check price ceiling filter to catch similar future errors
- Saved the final cleaned dataset as a permanent Delta Table for downstream SQL querying

**2. SQL Analysis — Databricks SQL, Window Functions**
- 30-day rolling average price per commodity per market (`AVG() OVER (... ROWS BETWEEN 29 PRECEDING AND CURRENT ROW)`)
- Year-over-year price comparison using `LAG()`, partitioned by commodity, market, and month
- Cross-state price ranking using `RANK()`, identifying which states consistently pay more for a given commodity

**3. Statistical Hypothesis Test**
- **Question:** Does the announcement of wheat's Minimum Support Price (MSP) for the 2025–26 marketing season (announced October 16, 2024) reduce subsequent price volatility?
- **Method:** Compared standard deviation and average price for wheat in the 60 days before vs. 60 days after the announcement, using an F-test (variance comparison) and t-test (mean comparison)
- **Result:** Volatility (standard deviation) dropped ~17% after the announcement (₹184.42 → ₹152.67), and average price rose ~5% (₹2,667.66 → ₹2,801.85). Both differences were statistically significant (p < 0.001).
- **Caveat:** This window overlaps with the rabi harvest season, so seasonal supply factors likely contribute alongside the MSP announcement itself. This analysis shows correlation, not isolated causation.

**4. Farmer Advisory Recommendations**
- For each commodity-district combination, identified the highest-average-price market and calculated the ₹/quintal gap versus the average of other markets in the same district

**5. Dashboard — Power BI**
- Interactive dashboard connected live to the Databricks SQL warehouse
- Commodity slicer, price trend chart, state comparison chart, and a curated farmer advisory findings table

## Key Findings

| Commodity | District | Best Market | Price Gap (₹/Quintal) |
|---|---|---|---|
| Bengal Gram | Indore | Sanwer APMC | ₹2,489 |
| Onion | Pune | Shirur | ₹1,830 |
| Onion | Bangalore | Ramanagara | ₹1,513 |
| Tomato | Kolar | Gowribidanoor APMC | ₹1,470 |
| Potato | Jalandhar | Mehatpur | ₹895 |

Across 5 different crops and 5 different states, consistent price gaps of ₹500–2,500 per quintal exist between markets within the *same district* — a meaningful, actionable finding for farmer income and a strong case for improved price transparency.

**MSP Impact:** Wheat price volatility fell ~17% following the 2025–26 MSP announcement, alongside a ~5% rise in average price — though seasonal effects are a likely contributing factor.

## Dashboard Preview

*(Insert your Power BI dashboard screenshot here)*

## Tech Stack

- **Data Processing:** PySpark (Databricks)
- **Analysis:** Databricks SQL (window functions), Python (SciPy for hypothesis testing)
- **Visualization:** Power BI
- **Source Data:** Agmarknet / data.gov.in (Government of India)

## What I'd Do Next

- Expand coverage to more districts and additional MSP-eligible crops (e.g., mustard, lentils) to strengthen the policy-impact analysis
- Incorporate rainfall/weather data to help isolate seasonal effects from the MSP announcement effect
- Automate data refresh via the Agmarknet API instead of manual CSV downloads, to keep the dashboard current
- Add a market-distance layer (e.g., via a geocoding API) so recommendations account for transport cost, not just raw price gap

---

*Data sourced from Agmarknet (Government of India). This project is for educational/portfolio purposes.*

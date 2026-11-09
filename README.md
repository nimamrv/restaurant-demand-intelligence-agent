# Restaurant Demand Intelligence Agent

A Databricks portfolio project combining Delta Lake, PySpark, SQL, gradient-boosted demand forecasting, and an LLM agent that invokes data-retrieval tools to answer restaurant operations questions.

## Results

On **12 non-overlapping rolling-origin seven-day windows** (84 evaluated calendar days), recursive forecasts achieved **4.17 orders/day MAE** versus **6.08** for the same-weekday-previous-week baseline, a **31.51% lower MAE**. These are historical backtest results, not prospective production accuracy. Closed Mondays are included as zero-demand days, which affects the aggregate metric. Results were observed in the original Databricks workspace; they are not reproduced by the synthetic sample.

The final historical demonstration forecast for **October 4–10, 2026** totals **179 orders**, with **Friday, October 9** highest at **37**. It is not a live November forecast.

## Architecture

See [docs/architecture.md](docs/architecture.md).

## Run in Databricks

1. Set up a Databricks workspace with permission to create Delta tables and access a compatible chat-completions model.
2. **Prerequisite:** create `workspace.default.restaurant_daily_demand` containing one row per recorded business day and the fields `business_day` (date), `orders` (integer), and `net_sales` (numeric). The raw POS aggregation is not included because it was done outside these notebooks. The provided `data/synthetic_sample.csv` is illustrative only and is not the original dataset.
3. Import and run `notebooks/01_restaurant_daily_demand.ipynb` to validate the source.
4. Run `notebooks/02_demand_forecasting.ipynb` to build modeling data, backtest, and save the forecast.
5. Run `notebooks/03_demand_intelligence_agent.ipynb` to configure the LLM and test function calls. Adjust the model name and authentication if required by your workspace.

## Modeling details

- Features: weekday, month, seven-day lag, seven- and 28-day trailing averages.
- Estimator: `HistGradientBoostingRegressor` (150 iterations, 15 max leaf nodes, 0.05 learning rate).
- A separate model is trained using only data before each historical cutoff. The next seven calendar days are forecast recursively; earlier predictions feed later features.
- Unknown non-Monday dates are not labeled zero; ordinary missing Mondays are treated as closures.
- Baseline predicts each date from the same weekday in the previous week, available at the forecast cutoff.
- The backtest covers only 12 recent weeks. Evaluate on additional periods before production use.

## Agent tools and guardrails

The LLM can call `get_demand_forecast()` and `get_historical_demand()` through an explicit Python whitelist. It is instructed not to invent figures or claim staffing, inventory, or kitchen-capacity sufficiency from demand data alone. Prompt instructions are not a substitute for deterministic output validation.

## Privacy and limitations

**No proprietary transaction-level data or credentials are included.** Notebook outputs were cleared before packaging. The synthetic CSV is randomly generated and does not reproduce the restaurant's actual records or backtest metrics. The source table must be created separately; this repository does not contain a raw POS ingestion pipeline. This is a portfolio demonstration, not a deployed production service.

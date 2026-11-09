# Architecture

```text
Private POS exports (not included)
    -> pre-aggregated daily Delta table [prerequisite]
    -> SQL and PySpark validation
    -> full calendar Delta modeling table
    -> lagged/rolling features + HistGradientBoostingRegressor
    -> rolling-origin 7-day backtest and previous-week baseline
    -> final forecast Delta table
    -> Databricks-hosted LLM -> function calls -> Delta SQL -> grounded response
```

## Delta tables

- `workspace.default.restaurant_daily_demand`: pre-existing source; `business_day`, `orders`, `net_sales`.
- `workspace.default.restaurant_modeling`: full calendar with unknown open days as null and known closed Mondays as zero.
- `workspace.default.restaurant_backtest`: historical predictions and observed demand.
- `workspace.default.restaurant_forecast`: saved 7-day demonstration forecast.

The model-serving endpoint is `system.ai.gpt-oss-120b` in the original workspace. Change the model name if unavailable. Workspace hostname is resolved at runtime.

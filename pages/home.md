---
name: Home
assetId: 473b0e67-f4a4-4362-b8d7-73734e9e0452
type: page
---

# 📊 Executive KPIs Dashboard

{% callout type="info" title="Primary Marketplace Only" %}
All KPIs on this dashboard reflect **Primary Marketplace** activity only (excludes resale transactions). Use the date range filter below to analyze any period. All KPIs compare against the prior period automatically.
{% /callout %}

{% filter_bar %}
  {% range_calendar
    id="date_range"
    default_range="last 7 days"
    preset_ranges=["last 7 days", "last 30 days", "last 3 months", "last 6 months", "last 12 months", "year to date", "all time"]
  /%}
{% /filter_bar %}

## 👤 User Acquisition & Activation

```sql user_metrics
SELECT
  toDate(registered_at) as date,
  count(*) as new_registrations,
  countIf(is_kyc_verified = true) as kyc_completions,
  countIf(user_segment = 'Investor') as active_investors
FROM users_enriched
GROUP BY date
ORDER BY date
```

{% big_value
  data="user_metrics"
  value="sum(new_registrations)"
  title="New Registrations"
  info="Total new user accounts created (registered_at) in the selected period."
  fmt="#,##0"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}

{% big_value
  data="user_metrics"
  value="sum(kyc_completions)"
  title="KYC Completions"
  info="Number of users who completed KYC verification (is_kyc_verified = true) in the selected period."
  fmt="#,##0"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}

{% big_value
  data="user_metrics"
  value="sum(kyc_completions) * 1.0 / sum(new_registrations)"
  title="KYC Conversion Rate"
  info="Overall percentage of registered users who completed KYC verification. Calculated as total KYC completions ÷ total registrations in the selected period."
  fmt="pct1"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}

{% big_value
  data="user_metrics"
  value="sum(active_investors)"
  title="Active Investors"
  info="Number of unique users with at least one successful order (classified as 'Investor' segment)."
  fmt="#,##0"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}

{% big_value
  data="user_metrics"
  value="sum(active_investors) * 1.0 / sum(new_registrations)"
  title="Activation Rate"
  info="Overall percentage of registered users who became active investors. Calculated as total active investors ÷ total registrations in the selected period."
  fmt="pct1"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}

{% line_chart
  data="user_metrics"
  x="date"
  y="sum(new_registrations)"
  title="New Users Per Day"
  y_fmt="#,##0"
  date_range={
    date="date"
    range={{date_range}}
  }
/%}

## 💰 Transaction Performance

```sql order_metrics
SELECT
  toDate(`o.created_at`) as date,
  count(*) as total_orders,
  countIf(`o.status` = 'SUCCESS') as success_orders,
  countIf(`o.status` = 'FAILED') as failed_orders,
  sumIf(total_investment, `o.status` = 'SUCCESS') as gmv,
  sumIf(shares, `o.status` = 'SUCCESS') as total_shares
FROM orders_enriched
WHERE is_resale_order = 0
GROUP BY date
ORDER BY date
```

{% big_value
  data="order_metrics"
  value="sum(gmv)"
  title="Revenue (GMV)"
  info="Gross Merchandise Value — total investment value from successful primary marketplace orders. Excludes resale transactions."
  fmt="#,##0' EGP'"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}

{% big_value
  data="order_metrics"
  value="sum(success_orders) * 1.0 / sum(total_orders)"
  title="Order Success Rate"
  info="Percentage of primary marketplace orders that completed successfully vs failed. A key indicator of payment and checkout health."
  fmt="pct1"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}

{% big_value
  data="order_metrics"
  value="sum(success_orders)"
  title="Successful Purchases"
  info="Total number of successfully completed primary marketplace orders in the selected period."
  fmt="#,##0"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}

{% big_value
  data="order_metrics"
  value="sum(total_shares)"
  title="Shares Sold"
  info="Total number of property shares purchased through successful primary marketplace orders."
  fmt="#,##0"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}

{% line_chart
  data="order_metrics"
  x="date"
  y="sum(total_shares)"
  title="Shares Sold Per Day"
  y_fmt="#,##0"
  date_range={
    date="date"
    range={{date_range}}
  }
/%}

## 🏦 Assets Under Management

```sql aum_metrics
SELECT
  toDate(`c.created_at`) as date,
  sum(amount) as aum_value,
  count(*) as contract_count,
  sum(no_of_shares) as aum_shares
FROM contracts_enriched
WHERE stage = 'Archived'
GROUP BY date
ORDER BY date
```

{% big_value
  data="aum_metrics"
  value="sum(aum_value)"
  title="AUM (Assets Under Management)"
  info="Total value of all archived (fully executed) contracts. Represents the cumulative assets held on the platform through the primary marketplace."
  fmt="#,##0' EGP'"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}

{% big_value
  data="aum_metrics"
  value="sum(contract_count)"
  title="Archived Contracts"
  info="Total number of fully executed (archived) contracts. Each contract represents a completed property share investment."
  fmt="#,##0"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}

{% big_value
  data="aum_metrics"
  value="sum(aum_shares)"
  title="AUM Shares"
  info="Total number of property shares held in archived contracts across the platform."
  fmt="#,##0"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}





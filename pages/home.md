---
name: Home
assetId: 473b0e67-f4a4-4362-b8d7-73734e9e0452
type: page
---

# 📊 Executive KPIs Dashboard

{% filter_bar %}
  {% range_calendar
    id="date_range"
    default_range="last 7 days"
    preset_ranges=["last 7 days", "last 30 days", "last 3 months", "last 6 months", "last 12 months", "year to date", "all time"]
  /%}
{% /filter_bar %}

## 👤 User Acquisition & Activation
---

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
  info="Overall percentage of registered users who completed KYC verification. Calculated as total KYC completions ÷ total registrations in the selected period. Target: 30%."
  fmt="pct1"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="target"
    target="0.3"
  }
  sparkline={
    type="area"
    x="date"
    date_range={
      range={{date_range}}
      date="date"
    }
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



---

## 💰 Transaction Performance
---

```sql order_metrics
SELECT
  toDate(`o.created_at`) as date,
  count(*) as total_orders,
  countIf(`o.status` = 'SUCCESS') as success_orders,
  countIf(`o.status` = 'SUCCESS' AND is_resale_order = 0) as primary_orders,
  countIf(`o.status` = 'SUCCESS' AND is_resale_order = 1) as resale_orders,
  sumIf(total_investment, `o.status` = 'SUCCESS' AND is_resale_order = 0) as primary_revenue,
  sumIf(shares, `o.status` = 'SUCCESS' AND is_resale_order = 0) as primary_shares,
  sumIf(shares, `o.status` = 'SUCCESS' AND is_resale_order = 1) as resale_shares
FROM orders_enriched
GROUP BY date
ORDER BY date
```

```sql resale_take_rate
SELECT
  toDate(buyer_order_created_at) as date,
  sum(commission_amount + buyer_transfer_fee) as take_rate_revenue
FROM resale_enriched
WHERE resale_status_label = 'Sold'
GROUP BY date
ORDER BY date
```

{% big_value
  data="order_metrics"
  value="sum(success_orders)"
  title="Total Successful Orders"
  info="Total number of successfully completed orders (primary + resale) in the selected period."
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
  value="sum(primary_orders)"
  title="Primary Orders"
  info="Number of successful primary marketplace orders in the selected period."
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
  value="sum(resale_orders)"
  title="Resale Orders"
  info="Number of successful resale marketplace orders in the selected period."
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
  value="sum(primary_revenue)"
  title="Primary Revenue"
  info="Total investment value from successful primary marketplace orders."
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
  data="resale_take_rate"
  value="sum(take_rate_revenue)"
  title="Resale Take Rate Revenue"
  info="Platform revenue from resale transactions — includes seller commission and buyer transfer fee (~4% combined)."
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
  value="sum(primary_shares)"
  title="Primary Shares Sold"
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

{% big_value
  data="order_metrics"
  value="sum(resale_shares)"
  title="Resale Shares Sold"
  info="Total number of property shares traded through successful resale orders."
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
  value="sum(success_orders) * 1.0 / sum(total_orders)"
  title="Order Success Rate"
  info="Percentage of all orders (primary + resale) that completed successfully. A key indicator of payment and checkout health."
  fmt="pct1"
  date_range={
    date="date"
    range={{date_range}}
  }
  comparison={
    compare_vs="prior period"
  }
/%}



---

## 🏦 Assets Under Management
---

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
  title="Active Contracts"
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





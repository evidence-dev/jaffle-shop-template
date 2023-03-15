# Welcome to Jaffle Shop 🥪

```monthly_revenue
with
monthly_revenue as (
    select 
        date_trunc('month', ordered_at) as month,
        sum(order_total) as revenue_usd1k
    from ${orders}
    group by month
    order by month desc
)

select 
    *,
    revenue_usd1k / (lag(revenue_usd1k, -1) over (order by month desc)) - 1 as growth_pct1,
    monthname(month) as month_name
from monthly_revenue
```

<BigValue
    data={monthly_revenue}
    value=revenue_usd1k
    comparison=growth_pct1
    title="Monthly Revenue"
    comparisonTitle="vs. prev. month"
/>

Revenue was <Value data={monthly_revenue} column=revenue_usd1k/> in <Value data={monthly_revenue} column=month_name/>, a change of <Value data={monthly_revenue} column=growth_pct1/> over the previous month.


```orders_per_day
select
    date_trunc('day', ordered_at) as date,
    count(*) as orders

from ${orders}

group by 1
order by 1
```

```orders_per_week
select
    date_trunc('week', ordered_at) as week,
    location_name,
    count(*) as orders

from ${orders}

group by 1,2
order by 1
```

## Orders

<LineChart
    data={orders_per_day}
    x=date
    y=orders
    yAxisTitle="orders per day"
    title="Orders per Day"
/>

<AreaChart
    data={orders_per_week}
    x=week
    y=orders
    yAxisTitle="orders per week"
    series=location_name
    title="Weekly Orders by Store Location"
/>

```revenue_per_city
select
    location_name as city,
    concat('/stores/', location_name) as store_link,
    count(distinct customer_id) as customers,
    count(*) as orders,
    sum(order_total) as revenue_usd

from ${orders}

group by 1, 2
```

## Reports on individual stores
Click a row to see the report for that store:
<DataTable data={revenue_per_city} link=store_link/>


## Customer cohorts
We truncate our `first_order_date` to month to create a cohort, then plot their average order value per cohort.

```customers_with_cohort
select
    *,
    date_trunc('month', first_ordered_at) as cohort_month,
    lifetime_spend_pretax / count_lifetime_orders as average_order_value_usd0

from ${customers}
```

```cohorts_aov
select
    cohort_month,
    avg(average_order_value_usd0) as cohort_aov_usd

from ${customers_with_cohort}

group by 1
order by cohort_month
```

<BarChart
    data={cohorts_aov}
    x=cohort_month
    y=cohort_aov_usd
    yAxisTitle="average order value"
    xAxisTitle="Monthly Cohort"
    title="Customer AOV by first month cohort"
/>

<Histogram
    data={customers_with_cohort}
    x=average_order_value_usd0
    title="Distribution of AOVs"
    subtitle="Customer count"
    xAxisTitle=true
/>

```customers
select * from analytics.customers
```

```orders
select * from analytics.orders
```
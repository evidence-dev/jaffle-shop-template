# Jaffle Shop {$page.params.city} 🥪

```monthly_revenue
with
monthly_revenue as (
    select 
        date_trunc('month', ordered_at) as month,
        location_name,
        sum(order_total) as revenue_usd1k
    from analytics.orders
    group by month, location_name
    order by month desc
)

select 
    *,
    revenue_usd1k / (lag(revenue_usd1k, -1) over (partition by location_name order by month desc)) - 1 as growth_pct1,
    monthname(month) as month_name
from monthly_revenue
```

<BigValue
    data={monthly_revenue.filter(data => data.location_name === $page.params.city)}
    value=revenue_usd1k
    comparison=growth_pct1
    title="Monthly Revenue"
    comparisonTitle="vs. prev. month"
/>

Revenue in {$page.params.city} was <Value data={monthly_revenue.filter(data => data.location_name === $page.params.city)} column=revenue_usd1k/> in <Value data={monthly_revenue.filter(data => data.location_name === $page.params.city)} column=month_name/>, a change of <Value data={monthly_revenue.filter(data => data.location_name === $page.params.city)} column=growth_pct1/> over the previous month.

```orders_per_day
select
    location_name as city,
    date_trunc('day', ordered_at) as date,
    count(*) as orders

from analytics.orders

group by 1, 2
order by 1, 2
```

## Orders per day in {$page.params.city}

<LineChart
    data={orders_per_day.filter(data => data.city === $page.params.city)}
    x=date
    y=orders
    yAxistTitle="orders_per_day in {$page.params.city}"
/>
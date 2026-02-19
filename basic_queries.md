# Categorical Distribution

Calculates the distribution of sales across various promotional categories. This query identifies the reach of each Special Offer by counting the unique orders associated with them, highlighting which incentives are driving the most volume. You basically select a category to show the volume of it.

```sql
SELECT specialofferid AS Special_Offer_ID, -- GROUP BY is already grouping by distinct Special offer ids, thats why u dont use Distinct
COUNT (DISTINCT salesorderid) AS Sales_Order_Count
FROM sales.salesorderdetail 
GROUP BY specialofferid  -- we group by the expression, not alias
ORDER BY Special_Offer_ID;  -- here postgres allows you to group by alias
```
<img width="1473" height="239" alt="image" src="https://github.com/user-attachments/assets/d950405a-a97e-45a1-ba31-868cfb2d167e" />

You can see the distribution is matching the records returned where most orders are from special offer id 1

<img width="292" height="346" alt="image" src="https://github.com/user-attachments/assets/84ab22a4-30a5-453f-90dd-e50f2ea5dd9a" />


# Histogram

In this case, it aggregates distinct order counts into monthly bins. This trend analysis is usually for executive reporting to monitor aging and/or period-over-period growth

```sql
SELECT 
	date_trunc('month', modifieddate)::date AS order_month,  -- date_trunc is postgres way to truncate all after a date
	COUNT(DISTINCT salesorderid) AS sales_order_count  -- as one order may have multiple line items
FROM sales.salesorderdetail
GROUP BY date_trunc('month', modifieddate)  -- again, we are grouping by using the expression
ORDER BY order_month;  -- Remember default is ascending
```
<img width="1480" height="324" alt="image" src="https://github.com/user-attachments/assets/893d1482-6e71-44a1-a933-5f84ce1866e2" />



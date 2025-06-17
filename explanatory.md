# **QUESTIONS**

1) What are most popular items on the menu?
2) What is the most popular cuisine on the menu?
3) How many items are ordered on average per order?
4) What is the average amount spent per order?  
5) How are sales performing over time? 
6) What is the busiest day of the week (what days are most orders put through)?   
7) What day of the week performs best revenue wise?
8) On average which period of the day is busiest? List from busiest to least busiest period showing sales revenue and orders

# **SOLUTIONS**  

1) What are most popular items on the menu?

```sql
SELECT item_name, COUNT(item_name) as Orders FROM orders o
INNER JOIN menu m
ON o.item_id = m.item_id
GROUP BY item_name
ORDER BY COUNT(item_name) DESC
LIMIT 8
```
| item_name             | orders |
|-----------------------|--------|
| Hamburger             | 622    |
| Edamame               | 620    |
| Korean Beef Bowl      | 588    |
| Cheeseburger          | 583    |
| French Fries          | 571    |
| Tofu Pad Thai         | 562    |
| Steak Torta           | 489    |
| Spaghetti & Meatballs | 470    |

2) What is the most popular cuisine on the menu?
   
```sql
SELECT 
items.cuisine, 
items.items, 
totals.items_ordered,
ROUND(totals.items_ordered::numeric / items.items, 2) AS average_orders
FROM (
SELECT A.cuisine, COUNT(*) AS items
FROM(
SELECT cuisine, item_name
FROM orders o
INNER JOIN menu m ON o.item_id = m.item_id
GROUP BY cuisine, item_name) A
GROUP BY A.cuisine) items
JOIN 
(
SELECT cuisine, COUNT(*) AS items_ordered
FROM orders o
INNER JOIN menu m ON o.item_id = m.item_id
GROUP BY cuisine) totals
ON items.cuisine = totals.cuisine
ORDER BY average_orders DESC
```
| cuisine  | items | items_ordered | average_orders |
|----------|-------|---------------|----------------|
| American | 6     | 2734          | 455.67         |
| Asian    | 8     | 3470          | 433.75         |
| Italian  | 9     | 2948          | 327.56         |
| Mexican  | 9     | 2945          | 327.22         |

3) How many items are ordered on average per order?

```sql
SELECT ROUND(AVG(items_ordered), 1) AS average_order_size FROM 
(SELECT order_id, COUNT(*) as items_ordered FROM orders o
GROUP BY order_id)
```
| average_order_size |
|--------------------|
| 2.3                |

4) What is the average amount spent per order?

```sql
SELECT ROUND(AVG(spend), 2) AS average_spend FROM
(SELECT order_id, SUM(price) AS spend FROM orders o
INNER JOIN menu m
ON o.item_id = m.item_id
GROUP BY order_id, price)
```
| average_spend |
|---------------|
| 14.46         |

5) How are sales performing over time?

```sql
WITH base_data AS (
    SELECT o.date, m.price,
    FLOOR((o.date - MIN(o.date) OVER ()) / 7) + 1 AS week
    FROM orders o
    INNER JOIN menu m ON o.item_id = m.item_id
),
max_week AS (
    SELECT MAX(week) AS last_week FROM base_data
)
SELECT 
b.week,SUM(CASE 
        WHEN b.week = mw.last_week THEN ROUND(b.price / 6.0 * 7, 2)
        ELSE b.price
        END) AS Revenue
FROM base_data b
CROSS JOIN max_week mw
GROUP BY b.week
ORDER BY b.week
```
| week | adjusted_price |
|------|----------------|
| 1    | 12595.80       |
| 2    | 12023.70       |
| 3    | 11860.55       |
| 4    | 12108.30       |
| 5    | 13299.90       |
| 6    | 12461.20       |
| 7    | 12321.85       |
| 8    | 11921.20       |
| 9    | 12803.95       |
| 10   | 12657.50       |
| 11   | 12837.50       |
| 12   | 11304.05       |
| 13   | 12860.27       |

6) What is the busiest day of the week (what days are most orders put through)?

```sql
SELECT TO_CHAR(date, 'Day') as order_day, 
COUNT(*) as order_amount
FROM orders o
LEFT JOIN menu m
ON o.item_id = m.item_id
GROUP BY TO_CHAR(date, 'Day')
ORDER BY COUNT(*) DESC
```
| order_day | order_amount |
|-----------|--------------|
| Monday    | 2010         |
| Friday    | 1822         |
| Tuesday   | 1788         |
| Sunday    | 1776         |
| Thursday  | 1689         |
| Saturday  | 1618         |
| Wednesday | 1531         |

7) What day of the week performs best revenue wise?

```sql
SELECT TO_CHAR(date, 'Day') AS order_day,  
SUM(price) as total_revenue
FROM orders o
LEFT JOIN menu m
ON o.item_id = m.item_id
GROUP BY TO_CHAR(date, 'Day')
ORDER BY SUM(price) DESC

```
| order_day | total_revenue |
|-----------|---------------|
| Monday    | 26007.45      |
| Friday    | 23707.95      |
| Tuesday   | 23356.00      |
| Sunday    | 23226.45      |
| Thursday  | 21846.85      |
| Saturday  | 21170.90      |
| Wednesday | 19902.30      |

8) On average which period of the day is busiest? List from busiest to least busiest period showing sales revenue and orders

```sql
SELECT A.time_period, A.orders, B.revenue 
FROM
(SELECT DATE_TRUNC('hour', time) as time_period,
COUNT(*) as orders FROM orders o
LEFT JOIN menu m
ON o.item_id = m.item_id
GROUP BY DATE_TRUNC('hour', time)) A

JOIN

(SELECT DATE_TRUNC('hour', time) as time_period,
SUM(price) AS revenue FROM orders o
LEFT JOIN menu m
ON o.item_id = m.item_id
GROUP BY DATE_TRUNC('hour', time)) B

ON A.time_period = B.time_period
ORDER BY A.orders DESC

```
| time_period | orders | revenue  |
|-------------|--------|----------|
| 12:00:00    | 1672   | 21718.40 |
| 13:00:00    | 1575   | 20640.25 |
| 17:00:00    | 1370   | 17869.50 |
| 18:00:00    | 1307   | 16861.50 |
| 19:00:00    | 1085   | 14172.60 |
| 16:00:00    | 1054   | 13711.65 |
| 14:00:00    | 968    | 12615.70 |
| 20:00:00    | 889    | 11678.60 |
| 15:00:00    | 751    | 9803.90  |
| 11:00:00    | 630    | 8122.50  |
| 21:00:00    | 608    | 7754.35  |
| 22:00:00    | 309    | 4047.80  |
| 23:00:00    | 11     | 157.80   |
| 10:00:00    | 5      | 63.35    |

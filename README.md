# TheLook eCommerce
## Dashboard
The dashboard done in Looker Studio can be found [here.](https://lookerstudio.google.com/reporting/ad9397fe-1829-40dc-ba57-076815e7ebcd)

For analysis, I used data from 2022 January-June and compared it with 2023 January-June. 
Main KPIs are compared from 2022 Jan-Jun to 2023 Jan-Jun. 
## Dataset
This dashboard was made using a fictitious eCommerce clothing site TheLook.
The dataset can be found [here.](https://console.cloud.google.com/marketplace/product/bigquery-public-data/thelook-ecommerce?project=silicon-park-391909)

## Preparing table
I have joined all the needed tables and created some additional columns.

```
WITH main AS (
  SELECT ROW_NUMBER() OVER (PARTITION BY orders.order_id, product_id ORDER BY orders.created_at) AS row_nr,
         orders.created_at,
         users.age,
         users.gender,
         users.state,
         users.city,
         users.country,
         products.id AS product_id,
         products.category,
         products.name,
         products.brand,
         products.department,
         orders.order_id,
         events.traffic_source,
         items.sale_price,
         orders.num_of_item,
         products.cost,
         users.id AS user_id,
         events.event_type,
         events.id AS event_id
  FROM `bigquery-public-data.thelook_ecommerce.order_items` AS items
  JOIN `bigquery-public-data.thelook_ecommerce.orders` AS orders
   ON items.order_id = orders.order_id
  JOIN `bigquery-public-data.thelook_ecommerce.products` AS products
   ON items.product_id = products.id
  JOIN `bigquery-public-data.thelook_ecommerce.users` AS users
   ON users.id = items.user_id
  JOIN `bigquery-public-data.thelook_ecommerce.events` AS events
   ON events.user_id = users.id
  WHERE (items.status NOT IN ('Cancelled','Returned')) AND 	
        orders.created_at >= '2022-01-01 00:00:00 UTC'),
first_purchase AS (
       SELECT DISTINCT(user_id),
              order_id,
              created_at,
              MIN(created_at) OVER (PARTITION BY user_id) AS first_purchase_date
       FROM `bigquery-public-data.thelook_ecommerce.orders` AS orders
       WHERE created_at >= '2022-01-01 00:00:00 UTC'
)
SELECT *,
       CASE
         WHEN DENSE_RANK() OVER (PARTITION BY user_id ORDER BY main.created_at) = 1
         THEN 'New Customer'
         ELSE 'Returning Customer'
       END AS CustomerType,
FROM main
WHERE row_nr = 1
```

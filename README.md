# Analyse Promotions and Provide Insights to Sales Director

## Table of Contents

- [Introduction](#Introduction)
- [Analysis](#analysis)
    - [Data manipulation](#data-manipulation)
    - [Ad-Hoc requests analysis](#Ad-Hoc-requests-analysis)
- [Power Bi Report](https://app.powerbi.com/groups/me/reports/a8aeab5c-07d7-4546-8cfe-bf956d7fe6f3/8df026711f41610dc290?experience=power-bi&bookmarkGuid=1150f02c0a898a302262)

***

## Introduction
AtliQ Mart is a retail giant with over 50 supermarkets in the southern region of India. All their 50 stores ran a massive promotion during the Diwali 2023 and Sankranti 2024 (festive time in India) on their AtliQ branded products. Now the sales director wants to understand which promotions did well and which did not so that they can make informed decisions for their next promotional period.

Task:
1.    Check “ad-hoc-requests.pdf” - this document includes important business questions posed by senior executives, requiring SQL-based report generation.
2.    Design a dashboard with analysis.The target audience of this dashboard is sales director.

## **Ad-Hoc requests analysis**
> **1. Provide a list of products with a base price greater than 500 and that are featured in promo type of 'BOGOF (Buy One Get One Free).**
```
SELECT 
    DISTINCT p.product_code, 
    p.product_name, 
    p.category,
    e.base_price,
    e.promo_type
FROM dim_products p
INNER JOIN fact_events e USING(product_code)
WHERE e.base_price > 500 
  AND e.promo_type = 'BOGOF';

```
#### Answer 1
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/a5f9b94c-034a-42b3-9b15-a26f4e1bf5ab)


> **2. Generate a report that provides an overview of the number of stores in each city.**
```
SELECT 
    city,
    COUNT(store_id) AS stores_count
FROM dim_stores
GROUP BY city
ORDER BY stores_count DESC;

```
#### Answer 2
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/cec57d1e-5d0c-4c6b-875e-95436ff952f5)


> **3. Generate a report that displays each campaign along with the total revenue generated before and after the campaign?**
```
SELECT 
    campaign_name,
    ROUND(SUM(base_price * `quantity_sold(before_promo)`) / 1000000, 2) AS total_revenue_before_promotion,
    ROUND(SUM(
        CASE
            WHEN promo_type = 'BOGOF' THEN base_price * 0.5 * (`quantity_sold(after_promo)` * 2)
            WHEN promo_type = '500 Cashback' THEN (base_price - 500) * `quantity_sold(after_promo)`
            WHEN promo_type = '50% OFF' THEN base_price * 0.5 * `quantity_sold(after_promo)`
            WHEN promo_type = '33% OFF' THEN base_price * 0.67 * `quantity_sold(after_promo)`
            WHEN promo_type = '25% OFF' THEN base_price * 0.75 * `quantity_sold(after_promo)`
        END
    ) / 1000000, 2) AS total_revenue_after_promotion
FROM 
    fact_events
JOIN 
    dim_campaigns USING (campaign_id)
GROUP BY 
    campaign_name;

```
#### Answer 3
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/a14a5d21-9e95-4621-b722-7c31f29e80aa)


> **4. Produce a report that calculates the Incremental Sold Quantity (ISU%) for each category during the Diwall campaign. 
Additionally, provide rankings for the categories based on their ISU%. The report will include three 
key fields: category, isu%, and rank order.**
```
WITH cte AS (    
    SELECT 
        c.campaign_name,
        p.category,
        SUM(e.`quantity_sold(before_promo)`) AS qty_sold_before_promo,
        SUM(e.`quantity_sold(after_promo)`) AS qty_sold_after_promo,
        ROUND(
            (
                (SUM(e.`quantity_sold(after_promo)`) - SUM(e.`quantity_sold(before_promo)`)) / 
                SUM(e.`quantity_sold(before_promo)`)
            ) * 100, 2
        ) AS ISU_percent
    FROM 
        dim_campaigns c
    INNER JOIN 
        fact_events e USING(campaign_id)
    INNER JOIN 
        dim_products p USING(product_code)
    WHERE 
        c.campaign_name = 'Diwali'
    GROUP BY 
        c.campaign_name, p.category
)
SELECT
    category,
    CONCAT(ISU_percent, '%') AS ISU_percent,
    DENSE_RANK() OVER (ORDER BY ISU_percent DESC) AS ranking
FROM 
    cte;

```
#### Answer 4
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/6598888d-d8d0-4a7e-a375-90056ab065c6)


> **5. Create a report featuring the Top 5 products, ranked by Incremental Revenue Percentage (IR%), across all campaigns.The report will provide essential information including product name, category, and ir%.**
```
SELECT 
    product_name,
    category,
    ROUND(
        (
            SUM(CASE
                WHEN promo_type = 'BOGOF' THEN base_price * 0.5 * (`quantity_sold(after_promo)` * 2)
                WHEN promo_type = '500 Cashback' THEN (base_price - 500) * `quantity_sold(after_promo)`
                WHEN promo_type = '50% OFF' THEN base_price * 0.5 * `quantity_sold(after_promo)`
                WHEN promo_type = '33% OFF' THEN base_price * 0.67 * `quantity_sold(after_promo)`
                WHEN promo_type = '25% OFF' THEN base_price * 0.75 * `quantity_sold(after_promo)`
                ELSE 0
            END) 
            - SUM(base_price * `quantity_sold(before_promo)`)
        ) / SUM(base_price * `quantity_sold(before_promo)`) * 100, 2
    ) AS `IR%`
FROM
    fact_events
JOIN
    dim_products USING (product_code)
GROUP BY 
    product_name, category
ORDER BY 
    `IR%` DESC
LIMIT 5;

```
#### Answer 5
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/7a498a01-4b6a-4824-9e7e-e355c3f6f9e0)



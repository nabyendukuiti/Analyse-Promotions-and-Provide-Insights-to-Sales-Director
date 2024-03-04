# Analyse Promotions and Provide Insights to Sales Director

## Table of Contents

- [Introduction](#Introduction)
- [Analysis](#analysis)
    - [Data manipulation](#data-manipulation)
    - [Ad-Hoc requests analysis](#Ad-Hoc-requests-analysis)
- [Power Bi Report](https://www.novypro.com/project/-analyse-promotions-and-provide-tangible-insights-to-sales-director-power-bi)

***

## Introduction
AtliQ Mart is a retail giant with over 50 supermarkets in the southern region of India. All their 50 stores ran a massive promotion during the Diwali 2023 and Sankranti 2024 (festive time in India) on their AtliQ branded products. Now the sales director wants to understand which promotions did well and which did not so that they can make informed decisions for their next promotional period.

Task:
1.    Check “ad-hoc-requests.pdf” - this document includes important business questions posed by senior executives, requiring SQL-based report generation.
2.    Design a dashboard with analysis.The target audience of this dashboard is sales director.

## Data manipulation
> **Creating and updating base_price_after_promo column**
```
ALTER TABLE fact_events
ADD base_price_after_promo float;

UPDATE fact_events
SET base_price_after_promo =	
	CASE
		WHEN promo_type = '25% OFF' THEN base_price*0.75
        WHEN promo_type = '33% OFF' THEN base_price*0.67
        WHEN promo_type = '50% OFF' THEN base_price*0.50
        WHEN promo_type = 'BOGOF' THEN base_price*2*0.50
        ELSE base_price - 500
        END;
```
> **Creating and updating rev_before_promo column**
```
ALTER TABLE fact_events
ADD rev_before_promo float;

UPDATE fact_events
SET rev_before_promo = `quantity_sold(before_promo)`*base_price;
```
> **Creating and updating rev_after_promo column**
```
ALTER TABLE fact_events
ADD rev_after_promo float;

UPDATE fact_events
SET rev_after_promo = `quantity_sold(after_promo)`*base_price_after_promo;
```
## **Ad-Hoc requests analysis**
> **1. Provide a list of products with a base price greater than 500 and that are featured in promo type of 'BOGOF (Buy One Get One Free).**
```
SELECT 
	DISTINCT p.product_code,
    p.product_name
FROM dim_products p
INNER JOIN fact_events e USING(product_code)
WHERE e.base_price > 500 AND e.promo_type = 'BOGOF';
```
#### Answer 1
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/c2657b33-33bd-4173-9b20-d0fcb3d8aa2f)

> **2. Generate a report that provides an overview of the number of stores in each city.**
```
SELECT 
	s.city, 
    COUNT(DISTINCT e.store_id) AS store_count
FROM dim_stores s
INNER JOIN fact_events e USING(store_id)
GROUP BY s.city
ORDER BY 2 DESC;
```
#### Answer 2
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/1b1052af-8d7b-409d-a0a5-6517e2d88578)

> **3. Generate a report that displays each campaign along with the total revenue generated before and after the campaign?**
```
WITH CTE AS(
SELECT 
   c.campaign_id,
   c.campaign_name,
   CONCAT(ROUND(SUM(e.rev_before_promo)/1000000),' ','M') AS total_rev_before_promo,
   CONCAT(ROUND(SUM(e.rev_after_promo)/1000000),' ','M') AS total_rev_after_promo
FROM dim_campaigns c
INNER JOIN fact_events e USING(campaign_id)
GROUP BY 1,2
)
SELECT 
   campaign_id, 
   campaign_name, 
   total_rev_before_promo, 
   total_rev_after_promo,
   CONCAT(ROUND((total_rev_after_promo-total_rev_before_promo)*100/total_rev_before_promo),'%') AS IR_percentage
FROM CTE;
```
#### Answer 3
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/e95285e2-2f8f-45a1-b5c1-14f6b3dd9559)

> **4. Produce a report that calculates the Incremental Sold Quantity (ISU%) for each category during the Diwall campaign. 
Additionally, provide rankings for the categories based on their ISU%. The report will include three 
key fields: category, isu%, and rank order.**
```
WITH cte AS(    
SELECT 
   c.campaign_name,
   p.category,
   SUM(e.`quantity_sold(before_promo)`) AS qty_sold_before_promo,
   SUM(e.`quantity_sold(after_promo)`) AS qty_sold_after_promo,
   ROUND(((SUM(e.`quantity_sold(after_promo)`)-SUM(e.`quantity_sold(before_promo)`))/SUM(e.`quantity_sold(before_promo)`))* 100,2) AS ISU_percent
FROM dim_campaigns c
INNER JOIN fact_events e USING(campaign_id)
INNER JOIN dim_products p USING(product_code)
WHERE c.campaign_name = 'Diwali'
GROUP BY 1,2
)
SELECT
   category,
   CONCAT(ISU_percent,'%') AS ISU_percent,
   DENSE_RANK()OVER(ORDER BY ISU_percent DESC) AS ranking
FROM cte;
```
#### Answer 4
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/8a1a0bf1-8492-449f-b404-b8d11844070d)

> **5. Create a report featuring the Top 5 products, ranked by Incremental Revenue Percentage (IR%), across all campaigns.The report will provide essential information including product name, category, and ir%.**
```
WITH cte AS( 
SELECT
   p.product_name,
   p.category,
   SUM(e.rev_before_promo) AS total_rev_before_promo,
   SUM(e.rev_after_promo) AS total_rev_after_promo
FROM dim_products p
INNER JOIN fact_events e USING(product_code)
GROUP BY 1,2
)
SELECT 
   product_name,
   category,
   CONCAT(ROUND((total_rev_after_promo-total_rev_before_promo)*100/total_rev_before_promo),'%') AS IR_percentage
FROM cte
ORDER BY 3 DESC
LIMIT 5;
```
#### Answer 5
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/61720c1c-4301-4985-9647-951d499e169c)


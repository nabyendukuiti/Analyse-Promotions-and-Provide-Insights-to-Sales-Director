# Analyse Promotions and Provide Insights to Sales Director

## Table of Contents

- [Introduction](#Introduction)
- [Question and Solution](#question-and-solution)
    - [Ad-Hoc requests analysis](#Ad-Hoc-requests-analysis)
    - [Store performance analysis](#Store-performance-analysis)
    - [Promotion type analysis](#Promotion-type-analysis)
    - [Product and category analysis](#Product-and-category-analysis)
- [Power Bi Report](https://www.novypro.com/project/-analyse-promotions-and-provide-tangible-insights-to-sales-director-power-bi)

***

## Introduction
AtliQ Mart is a retail giant with over 50 supermarkets in the southern region of India. All their 50 stores ran a massive promotion during the Diwali 2023 and Sankranti 2024 (festive time in India) on their AtliQ branded products. Now the sales director wants to understand which promotions did well and which did not so that they can make informed decisions for their next promotional period.

Task:
1.    Check “ad-hoc-requests.pdf” - this document includes important business questions posed by senior executives, requiring SQL-based report generation.
2.    Design a dashboard with analysis.The target audience of this dashboard is sales director.

## Question and Solution
### **Ad-Hoc requests analysis**
> **1. Provide a list of products with a base price greater than 500 and that are featured in promo type of 'BOGOF (Buy One Get One Free).**
```
SELECT
  DISTINCT p.product_code,
  p.product_name
FROM dim_products p
INNER JOIN fact_events e USING(product_code)
WHERE e.base_price > 500 and e.promo_type = 'BOGOF';
```
#### Answer 1
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/ee33e60b-7126-4ede-90a9-bb2ffe0ca543)

> **2. Generate a report that provides an overview of the number of stores in each city.**
```
SELECT
  s.city,
  COUNT(e.store_id) AS store_count
FROM dim_stores s
INNER JOIN fact_events e USING(store_id)
GROUP BY s.city
ORDER BY 2 DESC;
```
#### Answer 2
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/d6a06e41-ae43-4049-91b8-c93e6e421a22)

> **3. Generate a report that displays each campaign along with the total revenue generated before and after the campaign?**
```
WITH CTE AS(
SELECT 
  c.campaign_id,
  c.campaign_name,
  CONCAT(ROUND(SUM(r.total_rev_before_promo)/1000000),' ','M') AS total_rev_before_promo,
  CONCAT(ROUND(SUM(r.total_rev_after_promo)/1000000),' ','M') AS total_rev_after_promo
FROM dim_campaigns c
INNER JOIN fact_revenue r USING(campaign_id)
GROUP BY 1,2
)
SELECT 
  campaign_id, 
  campaign_name, 
  total_rev_before_promo, 
  total_rev_after_promo,
  CONCAT(ROUND((total_rev_after_promo-total_rev_before_promo)*100/total_rev_before_promo),'%') AS percent_change
FROM CTE;
```
#### Answer 3
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/6d8f0cdd-e292-49ac-99ca-57bcdfb70041)

> **4. Produce a report that calculates the Incremental Sold Quantity (ISU%) for each category during the Diwall campaign. 
Additionally, provide rankings for the categories based on their ISU%. The report will include three 
key fields: category, isu%, and rank order.**
```
WITH cte AS(    
SELECT 
  c.campaign_name,
  p.category,
  SUM(e.quantity_sold_before_promo) AS quantity_sold_before_promo,
  SUM(e.quantity_sold_after_promo) AS quantity_sold_after_promo,
  ROUND(((SUM(e.quantity_sold_after_promo)-SUM(e.quantity_sold_before_promo))/SUM(e.quantity_sold_before_promo))* 100,2) AS ISU_percent
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
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/7cc8ea43-bc87-4daf-8c3c-b9d4f7ed688f)

> **5. Create a report featuring the Top 5 products, ranked by Incremental Revenue Percentage (IR%), across all campaigns.The report will provide essential information including product name, category, and ir%.**
```
WITH cte AS( 
SELECT
  p.product_name,
  p.category,
  SUM(r.total_rev_before_promo) AS total_rev_before_promo,
  SUM(r.total_rev_after_promo) AS total_rev_after_promo
FROM dim_products p
INNER JOIN fact_revenue r USING(product_code)
GROUP BY 1,2
)
SELECT 
  product_name,
  category,
  ROUND(((total_rev_after_promo-total_rev_before_promo)*100/total_rev_before_promo),2) AS IR_percent
FROM cte
ORDER BY 3 DESC
LIMIT 5;
```
#### Answer 5
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/045f55e9-9444-47a1-b067-2b59c32bd377)

### **Store performance analysis**
> **1. Which are the top 10 stores in terms of Incremental Revenue (IR) generated from the promotions?**
```
WITH cte AS(
SELECT
  store_id,
  ROUND(SUM(total_rev_before_promo)/1000000,1) AS total_rev_before_promo,
  ROUND(SUM(total_rev_after_promo)/1000000,1) AS total_rev_after_promo
FROM fact_revenue
GROUP BY 1
)
SELECT 
  store_id,
  CONCAT(total_rev_before_promo,'M') AS total_rev_before_promo,
  CONCAT(total_rev_after_promo,'M') AS total_rev_after_promo,
  CONCAT(ROUND((total_rev_after_promo-total_rev_before_promo)*100/total_rev_before_promo,2),'%') AS IR_percent
FROM cte
ORDER BY 4 DESC
LIMIT 10;
```
#### Answer 1
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/652187c0-4160-45bb-9b0b-a06f77aa9830)

> **2. Which are the bottom 10 stores when it comes to Incremental Sold Units (ISU) during the promotional period?**
```
SELECT
  store_id,
  (SUM(quantity_sold_after_promo)-SUM(quantity_sold_before_promo))/SUM(quantity_sold_before_promo) AS ISU_percent
FROM fact_events   
GROUP BY 1
ORDER BY 2
LIMIT 10;
```
#### Answer 2
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/4f280f93-aefa-4294-b505-3e5094190c4e)

> **3. How does the performance of stores vary by city?**
```
SELECT
  s.city,
  r.store_id, 
  r.promo_type,
  (SUM(r.total_rev_after_promo)-SUM(r.total_rev_before_promo))/SUM(r.total_rev_before_promo) AS IR_percent
FROM dim_stores s
INNER JOIN fact_revenue r USING(store_id)
GROUP BY 1,2,3
ORDER BY 1,4;
```
#### Answer 3
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/24ce33fa-6b2f-4514-a2ce-331108de42b5)
250 ROW(S) returned

### **Promotion type analysis**
> **1. What are the top 2 promotion types that resulted in the highest Incremental Revenue?**
```
SELECT
  promo_type,
  CONCAT(ROUND(SUM(total_rev_before_promo)/1000000),' ','M') AS total_rev_before_promo,
  CONCAT(ROUND(SUM(total_rev_after_promo)/1000000),' ','M') AS total_rev_after_promo,
  ROUND((SUM(total_rev_after_promo)-SUM(total_rev_before_promo))*100/SUM(total_rev_before_promo),2) AS IR_percent
FROM fact_revenue
GROUP BY 1
ORDER BY 4 DESC
LIMIT 2;
```
#### Answer 1
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/a30d0698-e382-40fe-bb82-61b29ae63951)

> **2. What are the bottom 2 promotion types in terms of their impact on Incremental Sold Units?**
```
SELECT
  promo_type,
  SUM(quantity_sold_before_promo) AS total_qty_sold_before_promo,
  SUM(quantity_sold_after_promo) AS total_qty_sold_after_promo,
  CONCAT(ROUND((SUM(quantity_sold_after_promo)-SUM(quantity_sold_before_promo))*100/SUM(quantity_sold_before_promo),2),'%') AS ISU_percent
FROM fact_events   
GROUP BY 1
ORDER BY 3
LIMIT 2;
```
#### Answer 2
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/629cf9cd-3737-4f6c-bc71-6fe815cf08aa)

> **3. Is there a significant difference in the performance of discount-based promotions versus BOGOF (Buy One Get One Free) or cashback promotions?**
```
SELECT 
  promo_type,
  CONCAT(ROUND(SUM(total_rev_after_promo)/1000000),' ','M') AS total_rev_after_promo
FROM fact_revenue
GROUP BY 1;
```
#### Answer 3
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/629cf9cd-3737-4f6c-bc71-6fe815cf08aa)

> **4. Which promotions strike the best balance between Incremental Sold Units and maintaining healthy margins?**
```
SELECT
  r.promo_type,
  ROUND((SUM(e.quantity_sold_after_promo)-SUM(e.quantity_sold_before_promo))*100/SUM(e.quantity_sold_before_promo),2) AS ISU_percent,
  ROUND((SUM(r.total_rev_after_promo)-SUM(r.total_rev_before_promo))*100/SUM(r.total_rev_before_promo),2) AS IR_percent
FROM fact_events e
INNER JOIN fact_revenue r USING(event_id)
GROUP BY 1;
```
#### Answer 4
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/ea8b1bbb-0217-4b73-845a-658a0d5492b7)

### **Product and category analysis**
> **1. Which product categories saw the most significant lift in sales from the promotions?**
```
SELECT 
	p.category,
	ROUND((SUM(r.total_rev_after_promo)-SUM(r.total_rev_before_promo))*100/SUM(r.total_rev_before_promo),2) AS IR_percent
FROM dim_products p
INNER JOIN fact_revenue r USING(product_code)
GROUP BY 1
ORDER BY 2 DESC;
```
#### Answer 1
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/f5acb182-fd27-4146-b502-eceacd171b71)

> **2. Are there specific products that respond exceptionally well or poorly to promotions?**
```
SELECT 
	p.product_name,
    r.promo_type,
	ROUND((SUM(r.total_rev_after_promo)-SUM(r.total_rev_before_promo))*100/SUM(r.total_rev_before_promo),2) AS IR_percent
FROM dim_products p
INNER JOIN fact_revenue r USING(product_code)
GROUP BY 1,2
ORDER BY 3 DESC;
```
#### Answer 2
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/d42bcc91-d2dd-46b1-b53e-550da3b647d6)
19row(s) returned

> **3. What is the correlation between product category and promotion type effectiveness?**
```
SELECT 
	p.category,
    r.promo_type,
    CONCAT(ROUND(SUM(r.total_rev_before_promo)/1000000),' ','M') AS total_rev_before_promo,
    CONCAT(ROUND(SUM(r.total_rev_after_promo)/1000000),' ','M') AS total_rev_after_promo,
	ROUND((SUM(r.total_rev_after_promo)-SUM(r.total_rev_before_promo))*100/SUM(r.total_rev_before_promo),2) AS IR_percent
FROM dim_products p
INNER JOIN fact_revenue r USING(product_code)
GROUP BY 1,2
ORDER BY 5 DESC;
```
#### Answer 3
![image](https://github.com/nabyendukuiti/Analyse-Promotions-and-Provide-Insights-to-Sales-Director/assets/140970847/ae98bd7b-aba5-48b3-b95b-ebffe2200c0c)

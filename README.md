# SQL-Project-Consumer-Goods-Ad-hoc-Insights
This Data Analytics project was executed using SQL for querying and Power BI for visualization and insights. It is part of Codebasics SQL Project Challenge - Resume Challenge 4.

# Request 1:
Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.

SELECT DISTINCT market 
FROM  dim_customer
WHERE region = 'APAC' AND customer = "Atliq Exclusive";

# Request 2:
What is the percentage of unique product increase in 2021 vs. 2020?

WITH unique_product_count AS
(	
SELECT 
COUNT(DISTINCT CASE WHEN fiscal_year = 2020 THEN product_code END) AS unique_products_2020,
/* count of unique products sold in 2020 */	
COUNT(DISTINCT CASE WHEN fiscal_year = 2021 THEN product_code END) AS unique_products_2021 
/* count of unique products sold in 2021 */		 
  FROM fact_sales_monthly 
)
SELECT 
unique_products_2020, 
unique_products_2020,
CONCAT(ROUND(((unique_products_2021-unique_products_2020)*1.0/unique_products_2020)*100,2),'%') AS percentage_chg
FROM unique_product_count;

# Request 3:
Provide a report with all the unique product counts for each segment and sort them in descending order of product counts. 

SELECT 
       segment,
       COUNT(DISTINCT(product_code)) AS product_count
       FROM dim_product 
       GROUP BY segment
       ORDER by product_count DESC;

# Request 4:
Which segment had the most increase in unique products in 2021 vs 2020?

 WITH unique_product AS
( 
SELECT  
b.segment AS segment,
      COUNT(DISTINCT(CASE WHEN fiscal_year = 2020 THEN a.product_code END)) AS product_count_2020,
       COUNT(DISTINCT (CASE WHEN fiscal_year = 2021 THEN a.product_code END)) AS product_count_2021        
 FROM fact_sales_monthly AS a 
INNER JOIN dim_product AS b 
ON a.product_code = b.product_code 
GROUP BY b.segment
)
SELECT
 segment,
 product_count_2020,
 product_count_2021,
 (product_count_2021-product_count_2020) AS difference
FROM unique_product
ORDER BY difference DESC;

# Request 5:
Get the products that have the highest and lowest manufacturing costs.

SELECT  
         a.product_code AS product_code ,
         a.product AS product,
		     CONCAT(ROUND(b.manufacturing_cost,2)) AS manufacturing_cost 
         FROM dim_product AS a 
         JOIN
         fact_manufacturing_cost AS b
         ON a.product_code = b.product_code
         WHERE b.manufacturing_cost = 
         (SELECT MAX(manufacturing_cost) FROM  fact_manufacturing_cost) 
          OR   
          b.manufacturing_cost = 
         (SELECT MIN(manufacturing_cost) FROM  fact_manufacturing_cost) 
         ORDER BY b.manufacturing_cost DESC;

# Request 6:
Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market.

SELECT 
       a.customer_code ,
       b.customer,
       CONCAT(ROUND(AVG(pre_invoice_discount_pct)*100,2),'%') AS average_discount_percentage
FROM fact_pre_invoice_deductions AS a
INNER JOIN 
dim_customer AS b
ON a.customer_code = b.customer_code
WHERE market = 'India'
AND fiscal_year = 2021
GROUP BY customer, customer_code
ORDER BY AVG(pre_invoice_discount_pct) DESC
LIMIT 5;

# Request 7:
Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month. This analysis helps to get an idea of low and high-performing months and take strategic decisions.

SELECT        
 MONTHNAME(date) AS month_name, 
 YEAR(date) AS year,
 CONCAT('$',ROUND(SUM(a.sold_quantity * b.gross_price)/1000000,2)) AS gross_sales_amount_millions 
FROM fact_sales_monthly AS a
INNER JOIN fact_gross_price AS b
ON b.product_code = a.product_code
AND b.fiscal_year = a.fiscal_year
INNER JOIN dim_customer AS c
ON c.customer_code = a.customer_code
WHERE c.customer = 'Atliq Exclusive’
GROUP BY month_name, year
ORDER BY year;

# Request 8:
In which quarter of 2020, got the maximum total_sold_quantity? 

SELECT 
CASE		
WHEN MONTH(date) IN (9,10,11) THEN 'Q1’ 
WHEN MONTH(date) IN (12,1,2) THEN 'Q2’	
WHEN MONTH(date) IN (3,4,5) THEN 'Q3’	
ELSE 'Q4’	
END AS Quarters,	
SUM(sold_quantity) AS total_quantity_sold
FROM fact_sales_monthly
WHERE fiscal_year = 2020
GROUP BY Quarters
ORDER BY total_quantity_sold DESC;

# Request 9:
Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution?

WITH gross_sales AS (  
SELECT c.channel AS channel,   
ROUND(SUM(b.gross_price*a.sold_quantity)/1000000,2)
 AS gross_sales_million 
FROM fact_sales_monthly AS a 
JOIN fact_gross_price AS b
 ON a.product_code = b.product_code
 AND 
a.fiscal_year = b.fiscal_year
JOIN
 dim_customer AS c 
ON  a.customer_code = c.customer_code
 WHERE a.fiscal_year = 2021 
GROUP BY c.channel
)
SELECT channel, 
   CONCAT('$',gross_sales_million) AS gross_sales_million,
   CONCAT(ROUND(gross_sales_million/ SUM(gross_sales_million) OVER()*100,2),'%') AS percentage
FROM gross_sales
ORDER BY percentage DESC;

# Request 10:
Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021?

WITH top_sold_products AS /*creating a CTE for getting top selling products for all divisions*/
(
	SELECT b.division AS division,
		   b.product_code AS product_code,
		   b.product AS product,
		   SUM(a.sold_quantity) AS total_sold_quantity
	FROM fact_sales_monthly AS a
	INNER JOIN dim_product AS b
	ON a.product_code = b.product_code
	WHERE a.fiscal_year = 2021
	GROUP BY  b.division, b.product_code, b.product 
/* to get total sold quantity we will need to group it as shown in this part of query */

	ORDER BY total_sold_quantity DESC
),
top_sold_per_division AS
 /*creating this CTE to get top 3 based on total_sold quantity per division*/
(
 SELECT division,
	    product_code,
        product,
        total_sold_quantity,
        DENSE_RANK() OVER(PARTITION BY division ORDER BY total_sold_quantity DESC) AS rank_order
 /* using dense rank so that we can handle ties and still grab top 3 products*/
 FROM top_sold_products
 )
 SELECT * FROM top_sold_per_division
 WHERE rank_order <= 3;

















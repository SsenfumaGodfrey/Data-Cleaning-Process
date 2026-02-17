                        
# Data Cleaning Project (Using SQL)



#### 1. Remove Duplicates From Dataset.
#### 2. Standardizing Of Data.
#### 3. Handling Null Valuea And Blank Values.
#### 4. Remove Columns And Row Basing On Analysis.


   ```SQL

      -- Create Schema
 CREATE SCHEMA projectchurn;

 USE projectchurn;

    -- Back Up raw Data-----
CREATE TABLE customer_details_backup
LIKE customer_details;

INSERT INTO customer_details_backup
SELECT * FROM customer_details;

 SELECT * FROM customer_details_backup;

CREATE TABLE order_details_backup
LIKE order_detials;

 INSERT INTO order_details_backup
 SELECT * FROM order_detials;

 SELECT * FROM order_details_backup;

      -- COMMIT;
```

       


## 1. WHY Remove Duplicates?
#### ❌ Problem:
Duplicate records skew analysis  during counts, averages, and totals become inflated.

#### ✅ Reasons:
Accurate counts: Duplicates make you think you have more customers and sales, than you actually do.

Fair representation: Each entity like customer and transaction should be counted once.

Prevents double counting in aggregations: SUM, AVG, COUNT all become wrong.

Machine learning issues: Models may overfit to repeated rows or give them undue weight.

```SQL
     -- check the duplicates with the table ----             
SELECT *, ROW_NUMBER() OVER(PARTITION BY
          customer_id,signup_date,
          order_id, order_date,
          dish_name,
          restaurant_name,
          payment_method,
          last_order_date    ) AS row_num
FROM order_detials;
                                                -- SSENFUMA GODFREY Data nerds
    
 WITH duplicated_cte AS
      ( 
 SELECT *, 
          ROW_NUMBER() OVER(PARTITION BY
          customer_id,
          signup_date,
          order_id,
          order_date,
          dish_name,
          restaurant_name,
          payment_method,
          last_order_date    ) AS row_num
 FROM order_detials)
                     SELECT * FROM duplicated_cte
                     WHERE row_num > 1;

      -- created new table with row_num column , the table is called order_details
 
      --  Remove the Duplicate Records---
DELETE FROM order_details
WHERE row_num > 1;


```

## 2. WHY Standardize Data?
### ❌ Problem:
Inconsistent formats cause mismatches, misgrouping and calculation errors.

### ✅ Reasons for text standardization:
"NY" vs "New York" vs "ny"  treated as different categories = broken grouping.

"john" vs "John" vs "JOHN"  case sensitivity breaks lookups and joins.

" 123 Main St" vs "123 Main St"  trailing spaces cause mismatches.

### ✅ Reasons for date standardization:
"12/31/2023" vs "2023-12-31" vs "31-Dec-23"  sorting fails, calculations wrong.

Age calculations, time between dates, seasonality analysis all require proper date format.

### ✅ Reasons for numeric standardization:
"$1,234" vs "1234" → text vs number = can't sum or average.

"5kg" vs "5"  units attached = treated as text.

"00123" vs "123"  leading zeros matter for IDs but not for math.

### ✅ Reasons for categorical standardization:
Consistent groupings  accurate segmentation and reporting.

Machine learning expects consistent label encoding.

```SQL

       -- Standardizing Data Format ---

SELECT * FROM customer_details;
UPDATE customer_details SET city = UPPER(city);
UPDATE customer_details SET city = TRIM(city);
 
SELECT * FROM order_details;

UPDATE order_details
SET signup_date = STR_TO_DATE(signup_date, '%m/%d/%Y');
UPDATE order_details SET order_date = STR_TO_DATE(order_date, '%m/%d/%Y');
UPDATE order_details SET last_order_date = STR_TO_DATE(last_order_date, '%m/%d/%Y');
UPDATE order_details SET rating_date = STR_TO_DATE(rating_date, '%m/%d/%Y');
 
                                                                                 -- SSENFUMA GODFREY Data nerds
    
      -- RENAME THE COLUMN ---
     
ALTER TABLE cohort RENAME COLUMN order_date TO first_day_topurchase;
ALTER TABLE cohort RENAME COLUMN last_order_date TO last_day_topurchase;
 
     -- Changing Data Types into correct format ---
 
DESC customer_details;
ALTER TABLE customer_details MODIFY customer_id VARCHAR(5);
 
DESC order_details;

ALTER TABLE order_details MODIFY signup_date date;
ALTER TABLE order_details MODIFY order_date date; 
ALTER TABLE order_details MODIFY last_order_date date;
ALTER TABLE order_details MODIFY rating_date date;


```

 # 3. WHY Handle Null or Blank Values?
### ❌ Problem:
Nulls cause errors in calculations, broken joins and missing insights.

### ✅ Reasons:
Many functions break For example Excel formulas, SQL aggregations, Python math operations).

Visualizations hide data ; charts may show gaps or omit entire rows.

Statistical distortion; Ignoring nulls can bias results (e.g., only calculating average on non-null rows may overestimate).

Machine learning models can’t handle nulls , they require imputation or removal.

```sql


   -- Null Values or Blank Values ---

SELECT delivery_status
FROM order_detials
WHERE delivery_status = NULL;

DELETE FROM order_detials
WHERE delivery_status = NULL;
                                 -- SSENFUMA GODFREY Data nerds
    
SELECT
     restaurant_name
FROM order_detials
WHERE restaurant_names = NULL OR restaurant_name = '';

DELETE FROM order_detials
WHERE restaurant_names = NULL OR restaurant_name = '';

UPDATE  order_detials
SET churned = 'default_VALUE'
WHERE churned IS NULL;


```

# 4. WHY Remove Unnecessary Columns And Rows?
### ❌ Problem:
Irrelevant columns add noise, complexity and performance issues.

### ✅ Reasons:
Improves readability and focus only on what matters for analysis.

Reduces file size and  faster load times, less memory usage.

Prevents data leaks in machine learning  like including future information or unique IDs that models memorize.

Simplifies joins and fewer columns = less ambiguity.

Removes redundant data for example derived columns (birth year from birth date) or highly correlated fields.



```SQL

    -- Removing Irrelevant Columns And rows---

SELECT * FROM order_detials; 
                                 -- SSENFUMA GODFREY Data nerds
    
ALTER TABLE  order_detials
DROP COLUMN order_frequency;

DELETE FROM order_detials
WHERE payment_method ='Others' ; 
 
```
## BIG PICTURE: Why Data Cleaning Matters
   If you skip cleaning.......................	The consequence
-   Duplicates .................................Wrong totals, inflated metrics
-   Nulls mishandled............................Broken calculations, biased insights
-   Inconsistent formats........................Misgrouped categories, failed merges
-   Unnecessary columns.........................Clutter, slower performance, overfitting
## The Core Principle:
#### Garbage In = Garbage Out (GIGO)
No matter how sophisticated your analysis or model, if the data is dirty, the output is unreliable.

 ### Data cleaning isn't just a chore... it's the foundation of trustworthy analysis. 



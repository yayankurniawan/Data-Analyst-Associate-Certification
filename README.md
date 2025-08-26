# Practical Exam: Grocery Store Sales
FoodYum is a grocery store chain that is based in the United States.

Food Yum sells items such as produce, meat, dairy, baked goods, snacks, and other household food staples.

As food costs rise, FoodYum wants to make sure it keeps stocking products in all categories that cover a range of prices to ensure they have stock for a broad range of customers.

## Data
The data is available in the table products.

The dataset contains records of customers for their last full year of the loyalty program.


| Column Name         | Type       | Description                                                                 | Missing Value Handling                  |
|---------------------|------------|-----------------------------------------------------------------------------|-----------------------------------------|
| `product_id`        | Nominal    | Unique identifier of the product                                            | Not allowed (database-enforced)         |
| `product_type`      | Nominal    | Product category: Produce, Meat, Dairy, Bakery, Snacks                      | Replace with `"Unknown"`                |
| `brand`             | Nominal    | Brand name (7 possible values)                                              | Replace with `"Unknown"`                |
| `weight`            | Continuous | Product weight in grams (positive, 2 decimals)                              | Replace with **overall median weight**  |
| `price`             | Continuous | Product price in USD (positive, 2 decimals)                                 | Replace with **overall median price**   |
| `average_units_sold`| Discrete   | Average monthly units sold (positive integer)                               | Replace with `0`                         |
| `year_added`        | Nominal    | Year product was added to FoodYum stock                                     | Replace with `2022`                      |
| `stock_location`    | Nominal    | Warehouse location: A, B, C, or D                                           | Replace with `"Unknown"`                |

### Task 1
In 2022 there was a bug in the product system. For some products that were added in that year, the year_added value was not set in the data. As the year the product was added may have an impact on the price of the product, this is important information to have.

Write a query to determine how many products have the year_added value missing. Your output should be a single column, missing_year, with a single row giving the number of missing values.

```
select count(*) as missing_year from public.products where year_added is null
```

### Task 2
Given what you know about the year added data, you need to make sure all of the data is clean before you start your analysis. The table below shows what the data should look like.

Write a query to ensure the product data matches the description provided. Do not update the original table.

```
SELECT
    product_id,
    COALESCE(product_type, 'Unknown') AS product_type,
    COALESCE(NULLIF(REPLACE(brand, '-', ''), ''), 'Unknown') AS brand,
    COALESCE(ROUND(CAST(REGEXP_REPLACE(weight, '[^\d.]', '', 'g') AS DECIMAL(10, 2)), 2), ROUND((SELECT PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY CAST(REGEXP_REPLACE(weight, '[^\d.]', '', 'g') AS DECIMAL(10, 2))) FROM products), 2)) AS weight,

COALESCE(
    TO_CHAR(CAST(price AS DECIMAL(10, 2)), '9999999999.99'),
    TO_CHAR(CAST((SELECT PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY price) FROM products) AS DECIMAL(10, 2)), '9999999999.99')
) AS price,

    COALESCE(average_units_sold, 0) AS average_units_sold,
    COALESCE(year_added, 2022) AS year_added,
    COALESCE(UPPER(stock_location), 'Unknown') AS stock_location
FROM products;
```

### Task 3
To find out how the range varies for each product type, your manager has asked you to determine the minimum and maximum values for each product type.

Write a query to return the product_type, min_price and max_price columns.

```
select product_type , min(price) as min_price , max(price) as max_price
from public.products
group by product_type
```

### Task 4
The team want to look in more detail at meat and dairy products where the average units sold was greater than ten.

Write a query to return the product_id, price and average_units_sold of the rows of interest to the team.

```
SELECT product_id, price, average_units_sold
FROM products 
WHERE product_type IN ('Meat', 'Dairy')
	    AND average_units_sold > 10;
```

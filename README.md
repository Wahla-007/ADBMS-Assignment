# SQL Query Optimization Assignment

## Overview

This repository contains a SQL query optimization project focused on improving the performance of a complex analytical query on a customer orders dataset. The project analyzes an inefficient query, identifies performance bottlenecks, and applies several optimization techniques to significantly enhance execution time and resource utilization.

## Project Structure

The project consists of three main parts:

1. **Initial Query Analysis**
   - Examination of the original query and its execution plan
   - Identification of inefficiencies and bottlenecks
   - Performance baseline measurements

2. **Optimization Techniques**
   - Rewriting correlated subqueries
   - Creating appropriate indexes
   - Pushing selections and projections
   - Implementation of Common Table Expressions (CTEs)

3. **Performance Comparison**
   - Execution time measurements
   - Resource utilization analysis
   - Scalability considerations

## Dataset Description

The dataset consists of a single `customer_orders` table containing:
- Customer information (ID, Name, Email, Address)
- Order details (Order_ID, Product, Quantity, Price, Order_Date)
- Geographic data (Country)

## Key Findings

### Initial Query Analysis

The original query suffered from several inefficiencies:
- Correlated subquery executed once per qualifying row
- Full table scan due to lack of indexes
- Inefficient handling of filtering conditions
- High resource consumption

### Optimization Results

The applied optimization techniques yielded significant improvements:
- Execution time reduced by ~67% (from 76ms to ~25ms)
- Elimination of nested loops operations
- Reduced I/O operations through strategic indexing
- Improved query scalability

## Implemented Optimizations

### 1. Subquery Rewriting

Replaced the correlated subquery with a more efficient approach using Common Table Expressions (CTEs):

```sql
WITH ProductAvgPrices AS (
    SELECT 
        Product,
        AVG(Price) AS product_avg_price
    FROM customer_orders
    GROUP BY Product
),
OrderStats AS (
    SELECT
        co.Country,
        co.Product,
        COUNT(co.Order_ID) AS total_orders,
        SUM(co.Quantity) AS total_quantity,
        AVG(co.Price) AS avg_price
    FROM customer_orders co
    WHERE co.Order_Date BETWEEN '2023-01-01' AND '2024-12-31'
    AND co.Quantity > 0
    GROUP BY co.Country, co.Product
    HAVING COUNT(co.Order_ID) > 0
)
SELECT
    os.Country,
    os.Product,
    os.total_orders,
    os.total_quantity,
    os.avg_price,
    pap.product_avg_price
FROM OrderStats os
JOIN ProductAvgPrices pap ON os.Product = pap.Product
ORDER BY os.Country, os.total_orders DESC
```

### 2. Index Creation

Implemented strategic indexes to support query operations:

```sql
-- Index for date range filtering and quantity check
CREATE INDEX IX_customer_orders_Date_Quantity 
ON customer_orders(Order_Date, Quantity);

-- Composite index to support grouping and sorting
CREATE INDEX IX_customer_orders_Country_Product 
ON customer_orders(Country, Product);

-- Index for product price calculations
CREATE INDEX IX_customer_orders_Product_Price
ON customer_orders(Product) INCLUDE (Price);
```

### 3. Early Filter Application

Restructured the query to apply filters earlier:

```sql
WITH filtered_orders AS (
    SELECT
        Country, Product, Order_ID, Quantity, Price
    FROM customer_orders
    WHERE Order_Date BETWEEN '2023-01-01' AND '2024-12-31'
    AND Quantity > 0
),
-- Rest of the query
```

## Performance Metrics

| Query Version | Elapsed Time | Improvement |
|---------------|--------------|-------------|
| Original Query | 76 ms | Baseline |
| Optimized Query (CTEs + Indexes) | ~25 ms | ~67% reduction |
| Optimized Query (Alternative) | ~30 ms | ~60% reduction |

## Conclusion

This project demonstrates the significant impact of proper SQL query optimization techniques. By addressing correlated subqueries, implementing appropriate indexes, and restructuring queries to filter data earlier, we achieved substantial performance improvements. These optimizations not only enhance current performance but also ensure better scalability as the dataset grows.

## Technologies Used

- SQL Server
- T-SQL
- Query Execution Plan Analysis

## Author

Muhammad Abdul Rehman Wahla (2023-CS-717)  
Department of Computer Science  
University of Engineering and Technology, Lahore

CREATE TABLE sales (
    category String,
    order_date Date,
    revenue Float64,
    cumulative_revenue Float64,
    cumulative_orders UInt32,
    average_check Float64,
    max_avg_check_date Date,
    max_avg_check_value Float64
) ENGINE = MergeTree()
ORDER BY (category, order_date);

INSERT INTO sales (category, order_date, revenue)
VALUES
    ('Category A', '2022-01-01', 100.00),
    ('Category A', '2022-01-02', 150.00),
    ('Category B', '2022-01-01', 200.00),
    ('Category B', '2022-01-02', 250.00),
    ('Category B', '2022-01-03', 300.00);
    
SELECT 
    category,
    order_date,
    revenue,
    sum(revenue) OVER (PARTITION BY category ORDER BY order_date) AS cumulative_revenue,
    sum(1) OVER (PARTITION BY category ORDER BY order_date) AS cumulative_orders,
    sum(revenue) OVER (PARTITION BY category ORDER BY order_date) / sum(1) OVER (PARTITION BY category ORDER BY order_date) AS average_check,
    max_avg_check_date,
    max_avg_check_value
FROM sales;

SELECT
    category,
    order_date,
    revenue,
    cumulative_revenue,
    cumulative_orders,
    average_check,
    max(order_date) OVER (PARTITION BY category) AS max_avg_check_date,
    max(average_check) OVER (PARTITION BY category) AS max_avg_check_value
FROM (
    SELECT 
        category,
        order_date,
        revenue,
        sum(revenue) OVER (PARTITION BY category ORDER BY order_date) AS cumulative_revenue,
        sum(1) OVER (PARTITION BY category ORDER BY order_date) AS cumulative_orders,
        sum(revenue) OVER (PARTITION BY category ORDER BY order_date) / sum(1) OVER (PARTITION BY category ORDER BY order_date) AS average_check
    FROM sales
) subquery;
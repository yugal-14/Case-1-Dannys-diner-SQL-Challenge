-- 1. What is the total amount each customer spent at the restaurant?
SELECT 
    s.customer_id,
    SUM(m.price) AS total_spent
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY total_spent DESC;


-- 2. How many days has each customer visited the restaurant?
SELECT 
    customer_id,
    COUNT(DISTINCT order_date) AS visit_days
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY visit_days DESC;


-- 3. What was the first item from the menu purchased by each customer?
WITH first_purchase AS (
    SELECT
        s.customer_id,
        m.product_name,
        DENSE_RANK() OVER (
            PARTITION BY s.customer_id
            ORDER BY s.order_date
        ) AS rank
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m
        ON s.product_id = m.product_id
)
SELECT customer_id, product_name
FROM first_purchase
WHERE rank = 1;


-- 4. What is the most purchased item on the menu and how many times was it purchased?
SELECT
    m.product_name,
    COUNT(*) AS times_purchased
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY times_purchased DESC
LIMIT 1;


-- 5. Which item was the most popular for each customer?
WITH popular_items AS (
    SELECT
        s.customer_id,
        m.product_name,
        COUNT(*) AS order_count,
        DENSE_RANK() OVER (
            PARTITION BY s.customer_id
            ORDER BY COUNT(*) DESC
        ) AS rank
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m
        ON s.product_id = m.product_id
    GROUP BY s.customer_id, m.product_name
)
SELECT customer_id, product_name, order_count
FROM popular_items
WHERE rank = 1;


-- 6. Which item was purchased first by the customer after they became a member?
WITH member_purchases AS (
    SELECT
        mem.customer_id,
        s.product_id,
        ROW_NUMBER() OVER (
            PARTITION BY mem.customer_id
            ORDER BY s.order_date
        ) AS row_num
    FROM dannys_diner.members mem
    JOIN dannys_diner.sales s
        ON mem.customer_id = s.customer_id
        AND s.order_date >= mem.join_date
)
SELECT
    mp.customer_id,
    m.product_name
FROM member_purchases mp
JOIN dannys_diner.menu m
    ON mp.product_id = m.product_id
WHERE row_num = 1;


-- 7. Which item was purchased just before the customer became a member?
WITH pre_member AS (
    SELECT
        mem.customer_id,
        s.product_id,
        ROW_NUMBER() OVER (
            PARTITION BY mem.customer_id
            ORDER BY s.order_date DESC
        ) AS rank
    FROM dannys_diner.members mem
    JOIN dannys_diner.sales s
        ON mem.customer_id = s.customer_id
        AND s.order_date < mem.join_date
)
SELECT
    pm.customer_id,
    m.product_name
FROM pre_member pm
JOIN dannys_diner.menu m
    ON pm.product_id = m.product_id
WHERE rank = 1;


-- 8. What is the total items and amount spent for each member before they became a member?
SELECT
    s.customer_id,
    COUNT(*) AS total_items,
    SUM(m.price) AS total_spent
FROM dannys_diner.sales s
JOIN dannys_diner.members mem
    ON s.customer_id = mem.customer_id
    AND s.order_date < mem.join_date
JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;


-- 9. If each $1 spent equals 10 points and sushi has a 2x multiplier, how many points does each customer have?
SELECT
    s.customer_id,
    SUM(
        CASE
            WHEN m.product_name = 'sushi' THEN m.price * 20
            ELSE m.price * 10
        END
    ) AS total_points
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY total_points DESC;


-- 10. In the first week after joining, members earn 2x points on all items.
--     How many points do customers A and B have at the end of January?
SELECT
    mem.customer_id,
    SUM(
        CASE
            WHEN s.order_date BETWEEN mem.join_date AND mem.join_date + 6
                THEN m.price * 20
            WHEN m.product_name = 'sushi'
                THEN m.price * 20
            ELSE m.price * 10
        END
    ) AS total_points
FROM dannys_diner.members mem
JOIN dannys_diner.sales s
    ON mem.customer_id = s.customer_id
JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
WHERE s.order_date <= '2021-01-31'
GROUP BY mem.customer_id
ORDER BY mem.customer_id;

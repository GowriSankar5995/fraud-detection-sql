CREATE DATABASE fraud_detection;
USE fraud_detection;

CREATE TABLE transactions (
    transaction_id INT PRIMARY KEY,
    user_id INT,
    amount DECIMAL(10,2),
    transaction_time DATETIME,
    location VARCHAR(50)
);

INSERT INTO transactions VALUES
(1, 101, 500, '2026-04-06 10:00:00', 'Hyderabad'),
(2, 101, 12000, '2026-04-06 10:02:00', 'Delhi'),
(3, 101, 15000, '2026-04-06 10:05:00', 'Chennai'),
(4, 102, 200, '2026-04-06 11:00:00', 'Mumbai'),
(5, 103, 300, '2026-04-06 12:00:00', 'Bangalore'),
(6, 101, 13000, '2026-04-06 10:07:00', 'Hyderabad'),
(7, 104, 700, '2026-04-06 09:00:00', 'Delhi'),
(8, 104, 700, '2026-04-06 09:01:00', 'Delhi'),
(9, 104, 700, '2026-04-06 09:02:00', 'Delhi');

SELECT *
FROM transactions
WHERE amount > 10000;

 Rule 2: Multiple Transactions in Short Time

SELECT user_id, COUNT(*) AS transaction_count
FROM transactions
WHERE transaction_time BETWEEN '2026-04-06 10:00:00'
AND '2026-04-06 10:10:00'
GROUP BY user_id
HAVING COUNT(*) > 2;


Rule 3: Different Locations in Short Time

SELECT t1.user_id,
       t1.location AS loc1,
       t2.location AS loc2,
       t1.transaction_time,
       t2.transaction_time
FROM transactions t1
JOIN transactions t2
ON t1.user_id = t2.user_id
AND t1.location != t2.location
AND ABS(TIMESTAMPDIFF(MINUTE, t1.transaction_time, t2.transaction_time)) < 10;


* Rule 4: Repeated Same Amount

SELECT user_id, amount, COUNT(*) AS count
FROM transactions
GROUP BY user_id, amount
HAVING COUNT(*) >= 3;

* Window Function (Detect Fast Transactions)

SELECT *,
       TIMESTAMPDIFF(MINUTE,
           LAG(transaction_time) OVER (PARTITION BY user_id ORDER BY transaction_time),
           transaction_time) AS time_diff
FROM transactions;


* Final Fraud Detection System (Main Output)

SELECT 
    transaction_id,
    user_id,
    amount,
    transaction_time,
    location,

    -- Fraud Flags
    CASE 
        WHEN amount > 10000 THEN 'High Amount Fraud'
        
        WHEN TIMESTAMPDIFF(MINUTE,
            LAG(transaction_time) OVER (PARTITION BY user_id ORDER BY transaction_time),
            transaction_time) < 5 THEN 'Too Frequent Transactions'
        
        WHEN location != LAG(location) OVER (PARTITION BY user_id ORDER BY transaction_time)
             THEN 'Location Change Fraud'
        
        ELSE 'Normal'
    END AS fraud_status

FROM transactions;

DROP view if exists fraud_report

 *Create View for Easy Use

CREATE VIEW fraud_report AS
SELECT 
    transaction_id,
    user_id,
    amount,
    transaction_time,
    location,

    CASE 
        WHEN amount > 10000 THEN 'Fraud'
        ELSE 'Normal'
    END AS fraud_flag

FROM transactions;

-- Check View
SELECT * FROM fraud_report;
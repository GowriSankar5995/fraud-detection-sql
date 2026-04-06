Fraud Detection System using SQL

 Project Overview:

This project detects fraudulent transactions using SQL queries based on rules like high transaction amounts, rapid transactions, and location changes.


Technologies Used:
* SQL (MySQL)
* Window Functions
* Aggregate Functions

Fraud Detection Rules:
* High-value transactions (>10000)
* Multiple transactions in short time
* Transactions from different locations
* Repeated transaction patterns
  
Features:
* Fraud detection using CASE statements
* Pattern detection using LAG() window function
* Clean and structured SQL queries

How to Run:
* Open MySQL
* Run fraud_detection.sql
* View the results
  
Output:

![Fraud Output](fraud_output.png)

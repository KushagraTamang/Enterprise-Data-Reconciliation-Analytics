# Enterprise Data Reconciliation & Analytics

An end-to-end data reconciliation project using Python, Pandas, SQL, and Regex to process unstructured transaction logs, identify active customers, reconcile EUR-to-USD exchange rates, calculate revenue, and generate financial insights.

## 📌 Project Overview

This project focuses on building an enterprise data reconciliation pipeline for a FinTech scenario. The objective is to combine data from multiple sources, clean and transform raw records, reconcile transactions with daily exchange rates, identify customers whose latest account status is Active, and generate reliable financial insights.

The project uses three primary data sources:

- A historical customer database stored in SQLite.
- Unstructured server transaction logs containing approximately 100,000 records.
- A CSV file containing daily EUR-to-USD exchange rates.

By integrating these sources, the project transforms raw transaction information into a structured dataset suitable for revenue analysis and financial reporting.

## 🎯 Project Objectives

- Identify customers whose latest recorded account status is Active.
- Extract transaction information from unstructured server logs using Regular Expressions (Regex).
- Remove corrupted and error transaction records.
- Handle missing exchange rates using forward-filling.
- Reconcile transaction records with exchange rates using transaction dates.
- Calculate transaction revenue in USD.
- Analyze monthly revenue trends.
- Identify the top five customers by total USD revenue.
- Present analytical findings through visualizations.

## 🛠️ Technologies & Tools

| Technology | Purpose |
|---|---|
| Python | Data processing and analysis |
| SQL (SQLite) | Querying historical customer records |
| Pandas | Data cleaning, transformation, merging, and aggregation |
| NumPy | Numerical data processing |
| Regular Expressions (Regex) | Extracting structured information from server logs |
| Matplotlib | Creating revenue and customer visualizations |
| Jupyter Notebook | Executing and documenting the analytical workflow |

## 📂 Data Sources

### 1. Enterprise Customer Database

The SQLite database contains historical customer records, including user IDs, names, account statuses, and update timestamps.

Since a customer may have multiple records due to status changes, the latest record must be identified to determine the customer's current status.

### 2. Server Transaction Logs

The server log file contains approximately 100,000 records in an unstructured text format.

The required transaction details are embedded within log messages and include:

- Transaction date
- User ID
- Product ID
- Transaction value in EUR

Successful transactions were stored as INFO messages, while corrupted or failed transactions were marked as ERROR.

### 3. Daily Exchange Rates

The exchange-rate dataset contains daily EUR-to-USD conversion rates. Some dates have missing rates because financial markets are closed on weekends.

These missing values must be handled before the exchange-rate data can be reconciled with the transaction records.

## ⚙️ Project Workflow

### Step 1: Active Customer Identification Using SQL

The historical customer database contains multiple records for customers whose account statuses have changed over time.

To identify each customer's latest status, the SQL `ROW_NUMBER()` window function was used to rank records by `updated_at` in descending order within each `user_id`.

A Common Table Expression (CTE) was created to rank customer records. The query then selected only the latest record for each customer and retained those whose latest status was Active.

**Key

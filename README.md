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

**Key result:** 14,183 customers were identified as currently Active.

### Step 2: Server Log Cleaning

The raw transaction logs contained both successful and error records.

The cleaning process involved:

- Filtering out ERROR records without valid transaction values.
- Removing a header-like record.
- Preparing the log data for structured extraction.
- Validating the extracted transaction fields.

**Key result:** 94,902 valid transaction records remained after cleaning.

### Step 3: Transaction Extraction Using Regex

Regular Expressions were used with Pandas string operations to extract transaction information from the unstructured log messages.

The extracted fields were organized into the following columns:

| Column | Description |
|---|---|
| `Date` | Transaction date |
| `User_ID` | Customer identifier |
| `Product_ID` | Product identifier |
| `EUR_Value` | Transaction value in EUR |

The extracted dataset was checked to ensure that the required fields had been captured successfully.

**Key result:** No missing values were found in the extracted date, user ID, product ID, or EUR value fields.

### Step 4: Exchange Rate Cleaning and Forward-Filling

The transaction values were recorded in EUR, while the final analysis required USD revenue.

The exchange-rate dataset contained missing values for certain dates, particularly weekends.

Pandas forward-filling (`ffill()`) was used to carry the previous available exchange rate forward:

```python
exchange_rates["EUR_to_USD"] = (
    exchange_rates["EUR_to_USD"].ffill()
)
```

This approach filled missing rates using the previous available rate, allowing transactions to be matched with a rate even when the supplied exchange-rate dataset lacked a value for that date.

**Key result:** The exchange-rate dataset contained no missing exchange-rate values after forward-filling.

### Step 5: Date-Based Data Reconciliation

The transaction dates and exchange-rate dates were converted to a common datetime format.

The datasets were then merged using the transaction date and a left join.

```python
extracted_data["Date"] = pd.to_datetime(
    extracted_data["Date"]
)

exchange_rates["Date"] = pd.to_datetime(
    exchange_rates["Date"]
)

extracted_data = extracted_data.merge(
    exchange_rates,
    on="Date",
    how="left"
)
```

This connected the transaction records with the corresponding daily EUR-to-USD exchange rates.

### Step 6: Filtering Transactions for Active Customers

After exchange-rate reconciliation, the transaction records were matched with the list of currently Active customers.

An inner join was used to retain transactions belonging to customers whose latest recorded status was Active.

**Key result:** 53,899 transactions remained for the final analysis.

### Step 7: USD Revenue Calculation

The EUR transaction values were converted into numeric values and multiplied by their corresponding EUR-to-USD exchange rates.

**Formula:**

`USD Revenue = EUR Value × EUR-to-USD Exchange Rate`

Example Python implementation:

```python
extracted_data["EUR_Value"] = pd.to_numeric(
    extracted_data["EUR_Value"]
)

extracted_data["USD_Revenue"] = (
    extracted_data["EUR_Value"]
    * extracted_data["EUR_to_USD"]
)

df_final = extracted_data.copy()
```

The final reconciled dataset contained 53,899 transactions with no missing values in the required fields.

Each transaction included its date, customer identifier, product identifier, EUR value, exchange rate, and calculated USD revenue.

### Step 8: Business Analysis

#### Monthly USD Revenue Analysis

The final dataset was grouped by month, and the USD revenue values were summed to analyze revenue performance throughout 2023.

```python
df_final["Month"] = (
    df_final["Date"].dt.to_period("M")
)

monthly_revenue = (
    df_final.groupby("Month")["USD_Revenue"]
    .sum()
    .reset_index()
)
```

This analysis provided a month-by-month view of revenue fluctuations.

#### Top Five Customers by Revenue

Customer revenue contributions were analyzed to identify the five customers with the highest total USD revenue.

This analysis helps highlight high-value customers and understand the distribution of revenue across customers.

### Step 9: Data Visualization

Matplotlib was used to present the analytical findings visually.

The project included two main visualizations:

1. **Total USD Revenue by Month:** Illustrates monthly revenue fluctuations throughout 2023.
2. **Top Five Customers by USD Revenue:** Compares revenue contributions among the five highest-revenue customers.

These visualizations make the results easier to interpret and communicate.

## 📊 Key Results & Findings

| Metric | Result |
|---|---:|
| Approximate raw server log records | 100,000 |
| Valid transaction records after cleaning | 94,902 |
| Customers identified as currently Active | 14,183 |
| Transactions retained after active-customer filtering | 53,899 |
| Missing exchange-rate values after forward-filling | 0 |
| Missing values in required final transaction fields | 0 |

### Key Findings

- **Customer status identification:** SQL window functions identified customers based on their latest recorded account status.
- **Transaction data cleaning:** Error records were removed, leaving 94,902 valid transaction records.
- **Structured data extraction:** Regex transformed unstructured log messages into structured transaction fields.
- **Exchange-rate reconciliation:** Forward-filling handled missing exchange rates before date-based reconciliation.
- **Financial data integration:** Transaction records were matched with the active customer list and exchange-rate data.
- **Revenue analysis:** USD revenue was calculated for 53,899 retained transactions.
- **Customer contribution:** The analysis identified the top five customers by total USD revenue. Customer `U-06582` was the highest-revenue customer among the top five.
- **Monthly trends:** Revenue fluctuated throughout 2023, with the highest level occurring around January and another strong peak in August.

## 💼 Business Value

This project demonstrates how multiple data sources can be integrated into a structured analytical workflow to support financial reporting and business decision-making.

Potential business applications include:

- Improving the consistency of transaction-level financial reporting.
- Identifying customers based on their latest account status.
- Reducing manual effort in extracting information from unstructured logs.
- Standardizing financial transaction values into a common currency.
- Analyzing monthly revenue performance.
- Identifying high-value customers and understanding revenue concentration.
- Improving data quality through validation and reconciliation.

## 🧠 Skills Demonstrated

- SQL querying and window functions
- Common Table Expressions (CTEs)
- Python programming
- Pandas data manipulation
- Regex-based text extraction
- Data cleaning and validation
- Missing-value handling
- Datetime conversion
- Data merging and reconciliation
- Currency conversion and revenue calculations
- Grouping and aggregation
- Customer revenue ranking
- Exploratory data analysis
- Data visualization
- Analytical reporting and documentation

## 🚀 How to Run the Project

### Prerequisites

Install Python and Jupyter Notebook. The project also requires the source datasets used in the analysis.

### Install Dependencies

Run the following command:

```bash
pip install pandas numpy matplotlib jupyter
```

SQLite support is available through Python's standard library.

### Execution Steps

1. Clone or download the repository.
2. Install the required dependencies.
3. Place the permitted customer database, transaction log file, and exchange-rate CSV in the appropriate directories.
4. Open the Jupyter Notebook.
5. Update input file paths if necessary.
6. Execute the notebook cells in the documented workflow order.
7. Review the final reconciled dataset, monthly revenue analysis, and top-five customer results.

**Note:** The input datasets are required to reproduce the analysis. Do not assume they are included in the repository unless you have uploaded them.

## 📁 Suggested Repository Structure

```text
Enterprise-Data-Reconciliation-Analytics/
│
├── README.md
├── notebooks/
│   └── transaction_reconciliation_analysis.ipynb
├── data/
│   └── README.md
├── outputs/
│   ├── monthly_usd_revenue.png
│   └── top_5_customers.png
└── report/
    └── project_report.pdf
```

This is a suggested structure. Adjust the filenames and folders to match the actual contents of your GitHub repository. If the source datasets cannot be shared, use the `data/README.md` file to explain the required inputs instead of uploading the original data.

## 🔮 Potential Future Improvements

- Add automated data-quality checks after every processing stage.
- Validate exchange-rate coverage and identify dates that remain unmatched.
- Add reconciliation summaries showing record counts before and after each join.
- Create reusable functions for log extraction, data validation, and revenue calculation.
- Develop an interactive dashboard for monthly revenue and customer contribution.
- Add automated tests to validate transaction parsing and currency conversion.

These are potential enhancements and are not represented as completed project features.

## 📝 Conclusion

This project demonstrates an end-to-end enterprise data reconciliation workflow using SQL and Python. It combines historical customer-status analysis, unstructured log cleaning, Regex extraction, exchange-rate reconciliation, USD revenue calculation, and business analysis.

By transforming approximately 100,000 raw server log records into a final dataset of 53,899 transactions associated with currently Active customers, the project illustrates how data processing and reconciliation techniques can produce structured information for financial analysis.

The final outputs provide monthly revenue trends and top-customer rankings, demonstrating practical skills in data analytics, data preparation, and business insight generation.

## 👤 Author

**Kushagra Tamang**

Data Analytics | Python | SQL | Pandas | Excel | Power BI


# SQL-Based Fraud Detection System

## Project Overview
A robust SQL-based fraud detection engine built in PostgreSQL analyzing 15,000+ financial transactions. This system uses automated business rules and a dynamic Risk Scoring Engine to detect financial anomalies (like impossible travel and burner devices), prioritize critical alerts, and deliver actionable data insights for fraud analysts.

## Tech Stack
* **Database Management:** PostgreSQL (pgAdmin 4)
* **Data Engineering/Generation:** Python (Pandas, Faker)
* **Data Modeling:** dbdiagram.io (Entity-Relationship mapping)
* **Analysis:** SQL (Window Functions, CTEs, Aggregations, Dynamic Views)

## Database Architecture
The relational database is normalized and consists of four primary tables. 


1. `Users`: Customer profile and risk history.
2. `Devices`: Known, registered hardware associated with users.
3. `Transactions`: The core ledger of financial activity.
4. `Alerts`: The destination table for flagged anomalies.

## How to Run This Project (Setup Instructions)

1. **Generate Data:** Run the Python data generator (or use the provided CSV files).
2. **Build Schema:** Execute the `CREATE TABLE` DDL statements in `Table_creation_queries.sql` to build the Postgres tables.
3. **Import Data:** Use the pgAdmin Import Tool to load the CSV data in this **exact order**:
   * `users.csv` -> `Users` table
   * `devices.csv` -> `Devices` table
   * `transactions.csv` -> `Transactions` table

> **⚠️ IMPORTANT NOTE:** Do NOT load or import any data into the `Alerts` table during setup! The `Alerts` table is designed to be empty initially. It will be automatically populated with the final result data entries only *after* you run the Fraud Detection SQL rules.

4. **Run Fraud Rules:** Execute the `INSERT INTO Alerts` queries to scan the transactions and populate the Alerts table.
5. **Generate Dashboard:** Run the `CREATE VIEW Fraud_Risk_Report` query to initialize the scoring engine.

## Fraud Detection Rules Implemented
The system uses automated SQL queries to detect known fraud patterns:
* **High-Value Transactions (Whales):** Identifies transactions exceeding $5,000.
* **Geographical Anomalies (Impossible Travel):** Flags users transacting internationally or outside their primary market.
* **Device Mismatch:** Flags transactions originating from unregistered "burner" devices.
* **Suspicious Device Hopping:** Flags accounts utilizing an unusually high number of distinct devices across transactions.

##  The Risk Scoring Engine
Instead of binary Yes/No alerts, I engineered a dynamic SQL `VIEW` that assigns weighted risk scores to each transaction. This resolves "Alert Fatigue" (where one event triggers multiple overlapping rules) and helps Fraud Analysts prioritize the most dangerous activity.

```sql
-- Snippet of the Risk Scoring Logic
SUM(
    CASE 
        WHEN a.alert_type = 'Unknown Device' THEN 50
        WHEN a.alert_type = 'High Value Transaction' THEN 30
        WHEN a.alert_type = 'International Transaction' THEN 20
        WHEN a.alert_type = 'Suspicious Device Activity' THEN 10
        ELSE 5 
    END
) AS risk_score
```

## Analytics & Business Intelligence (`analysis.sql`)
To extract actionable insights from the generated alerts, I developed a comprehensive suite of analytical queries located in the `analysis.sql` file. This script serves as the backend logic for a Fraud Analyst Dashboard, transforming raw data into business intelligence.

**Key Analyses Included:**
* **Executive Priority Funnel:** Groups 10,000+ alerts into Critical, High, and Medium buckets to optimize investigation workflows and calculate total dollars at risk per tier.
* **Threat Profiling:** Identifies specific users and devices with the highest volume of suspicious activity and the highest aggregate risk scores.
* **Spatio-Temporal Tracking:** Pinpoints geographic fraud hotspots (City/Country) and extracts timestamps to identify "Night Owl" attack trends (e.g., spikes in fraud between 2 AM and 4 AM).
* **Merchant Vulnerability:** Analyzes which shopping categories (e.g., Tech vs. Retail) are targeted most frequently and calculates the average fraudulent transaction value.

*Technical highlights: Advanced Aggregations (`GROUP BY`), Window Functions (`OVER(PARTITION BY)`), Time-Series Analysis (`EXTRACT`), and Conditional Logic (`CASE WHEN`).*

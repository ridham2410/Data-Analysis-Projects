# Banking-Dashboard

### Dashboard Link : https://app.powerbi.com/groups/me/reports/f31bce58-ce8a-449b-8d36-7361a0506b15/e239bd44703b458a9c16?experience=power-bi

## Problem Statement

This dashboard helps a bank understand its transactions, accounts and customers in one place. It shows how many transactions are credits and debits, how transaction amounts move month by month, which customers have the largest transaction totals and balances, how balances are split between account types, and how the customer base breaks down by gender and age group.

It also tracks inactive accounts (accounts with no transaction in the last 90 days), so the bank can follow up with those customers.

While building the report, a few data quality problems were found (see **Data Quality Observations** below). Most transactions are not linked to an account or customer record, and some amounts are extreme outliers. These should be fixed at the source before the numbers are used for decisions.

### Steps followed

- Step 1 : Load the data into Power BI Desktop. The data is combined into one table named `CombinedBankingDataset` (10,000 rows, 21 columns) containing transaction, account and customer fields.
- Step 2 : Open Power Query Editor and check "column distribution", "column quality" and "column profile" under the View tab. Select "column profiling based on entire dataset", since the default profile only covers the first 1000 rows.
- Step 3 : It was observed that most account and customer columns are empty for most rows (9,600 of 10,000 rows have no account details and 9,700 have no customer details), because only 400 transactions match an account record and only 300 match a customer record.
- Step 4 : Date hierarchies (Year, Quarter, Month, Day) were used on `TransactionDate`, `OpenDate` and `DateOfBirth`.
- Step 5 : A separate table named `Measures Table` was created to hold all DAX measures.
- Step 6 : Two calculated columns were created, `Customer Age` and `Customer Age Group` (see Step 7).
- Step 7 : Customers were grouped into age groups using the following DAX.

        Customer Age = DATEDIFF(CombinedBankingDataset[DateOfBirth], TODAY(), YEAR)

        Customer Age Group =
        SWITCH(TRUE(),
            [Customer Age] <= 25, "<=25",
            [Customer Age] <= 35, "26-35",
            [Customer Age] <= 50, "36-50",
            "51+")

- Step 8 : The following measures were created in `Measures Table`.

        Count of transactions = COUNT(CombinedBankingDataset[TransactionID])

        Monthly Transaction Amount =
        CALCULATE(SUM(CombinedBankingDataset[Amount]),
            ALLEXCEPT(CombinedBankingDataset, CombinedBankingDataset[TransactionDate].[Month]))

        Monthly Transaction Balance =
        CALCULATE(SUM(CombinedBankingDataset[Balance]),
            ALLEXCEPT(CombinedBankingDataset, CombinedBankingDataset[TransactionDate].[Month]))

        Total Balance = SUM(CombinedBankingDataset[Balance])

        Inactive Accounts =
        CALCULATE(DISTINCTCOUNT(CombinedBankingDataset[Account_AccountID]),
            FILTER(VALUES(CombinedBankingDataset[Account_AccountID]),
                CALCULATE(MAX(CombinedBankingDataset[TransactionDate])) < TODAY() - 90))

        Customer Count By Gender = DISTINCTCOUNT(CombinedBankingDataset[CustomerID])

        Account Count by Type = COUNT(CombinedBankingDataset[Account_AccountID])

- Step 9 : **Page 1 (Transactions)** was built with five visuals.

  (a) Pie chart : Count of transactions by `TransactionType`

  (b) Area chart : Monthly Transaction Amount by month

  (c) Bar chart : Sum of `Amount` by customer `Name`

  (d) Clustered column chart : Total Balance by `AccountType`

  (e) Line chart : Inactive Accounts by year and month

- Step 10 : **Page 2 (Customers and Accounts)** was built with five visuals.

  (a) Donut chart : Customer Count By Gender

  (b) Column chart : Customer count by `Customer Age Group`

  (c) Treemap : Account Count by Type

  (d) Bar chart : Sum of `Balance` by customer `Name`

  (e) Area chart : Monthly Transaction Balance by month

- Step 11 : A theme was applied from the View tab in report view.
- Step 12 : The report was then published to Power BI Service.

# Snapshot of Dashboard (Power BI Service)

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/8b1c35bf-5e86-4933-9226-7afc54608c07" />

# Report Snapshot (Power BI DESKTOP)

<img width="1917" height="1018" alt="Image" src="https://github.com/user-attachments/assets/caf2604b-4cca-431f-a4a7-e8ab5ba177ec" />

<img width="1917" height="1018" alt="Image" src="https://github.com/user-attachments/assets/a45fe36d-9f75-468d-a370-dec053061ecd" />

# Insights

A two page report was created on Power BI Desktop and then published to Power BI Service.

The following inferences can be drawn from the dashboard. All figures are calculated from the 10,000 rows in the data model.

### [1] Transactions

   Total number of transactions = 10,000 (transaction dates range from Jan 2025 to Dec 2026)

   Credit transactions = 5,000 (50 %)

   Debit transactions = 5,000 (50 %)

   Total transaction amount = 28,767,962.80

   Credit amount = 41,386,678.80

   Debit amount = -12,618,716.00

   Transactions by year: 5,562 in 2025 and 4,438 in 2026

           thus, transactions are split exactly evenly between credit and debit.

### [2] Currency

   USD transactions = 8,666 (86.66 %)

   INR transactions = 1,334 (13.34 %)

   Amounts in USD and INR are added together without conversion in the current measures, so totals mix two currencies.

### [3] Accounts

   Only 4 account records exist in the data model.

   | Account ID | Type | Owner (Customer ID) | Opened | Balance |
   |---|---|---|---|---|
   | 102 | Current | priya singh (2) | 2011-06-20 | -157,874.40 |
   | 103 | Savings | Sarah Khan (3) | 2019-11-02 | 0.00 |
   | 104 | Current | Mark Lee (4) | 2018-07-21 | 50.00 |
   | 105 | Savings | Customer 99 (no customer record) | 2016-07-05 | 14.75 |

   Each account type has 2 accounts (Current and Savings).

   Each account has 100 linked transactions (400 in total).

   Inactive accounts (no transaction in the last 90 days) = 0, as the latest transaction for every account is dated after 5 Jul 2026. This measure depends on `TODAY()`, so the value changes over time.

### [4] Customers

   Only 3 customers have a customer record.

   Gender : 1 male (Mark Lee) and 1 female (Sarah Khan). Gender is blank for priya singh.

   Age group (as of the report date) :

   - 26-35 : 1 customer (Mark Lee, 31)
   - 36-50 : 1 customer (Sarah Khan, 37)
   - 51+ : 1 customer (priya singh, 51)

           thus, the customer base is too small for the demographic charts to show meaningful trends.

### [5] Transaction amount by customer

   | Customer | Total transaction amount | Linked transactions |
   |---|---|---|
   | Unknown (no customer match) | 29,026,067.80 | 9,700 |
   | Sarah Khan | 248,251.00 | 100 |
   | Mark Lee | -249,573.00 | 100 |
   | priya singh | -256,783.00 | 100 |

           thus, 97 % of transactions and almost the entire transaction value belong to the "Unknown" group.

### [6] Data Quality Observations

1. **Unmatched transactions** : 9,600 of 10,000 transactions have no linked account and 9,700 have no linked customer, so they show as "Unknown" in the charts.
2. **Extreme outliers in `Amount`** : the minimum is -99,999.99 and the maximum is 1,000,000.99. There are 40 rows with an absolute amount of 99,999 or more, and these dominate the monthly totals (for example, Jan, Feb and May 2025 each total about 1 million because of them).
3. **Total Balance is overstated** : `Balance` is an account-level value repeated on every transaction row, so `SUM(Balance)` adds it 100 times per account. The chart shows -15,782,440 for Current and 1,475 for Savings, whereas the actual account balances add up to -157,809.65 in total.
4. **Transaction type and description look swapped** : all 5,000 Debit rows have the description "Salary Credit" and negative or zero amounts, while all 4,800 Credit rows have the description "payment" and positive amounts (the other 200 are "unknown").
5. **Age group for missing dates of birth** : rows with no date of birth fall into the "<=25" group because a blank is treated as less than 25 in the `SWITCH` expression. The "<=25" bar based on row count is therefore wrong. Filtering out blank `DateOfBirth` values fixes this.
6. **Future-dated transactions** : 223 transactions are dated after 3 Oct 2026, so month-wise trends for late 2026 are not reliable.
7. **Mixed currencies** : USD and INR amounts are summed together.

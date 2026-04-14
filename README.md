# BSA/AML Detection Challenge

## Background

You are a BSA (Bank Secrecy Act) analyst at **Legends Community Bank**, a small community bank where all the customers happen to be famous historical figures. The compliance department has pulled 90 days of transaction data and needs your help identifying potentially suspicious activity.

Your task: analyze the bank's transaction data and identify customers who may be engaged in financial crimes, using two specific detection methods described below.

## The Data

Three CSV files in the `data/` directory cover the period **January 15, 2026 through April 14, 2026** (90 days).

### `customers.csv`

| Column | Type | Description |
|--------|------|-------------|
| `customer_id` | string | Unique customer identifier (e.g., `CUST001`) |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `date_of_birth` | date | YYYY-MM-DD format |
| `phone` | string | Phone number |
| `email` | string | Email address |
| `address` | string | Street address |
| `city` | string | City |
| `state` | string | Two-letter state code |
| `zip` | string | ZIP code |

### `accounts.csv`

| Column | Type | Description |
|--------|------|-------------|
| `account_id` | string | Unique account identifier (e.g., `ACCT001`) |
| `customer_id` | string | References `customers.customer_id` (one account per customer) |
| `account_type` | string | `checking` or `savings` |
| `open_date` | date | YYYY-MM-DD format |
| `current_balance` | decimal | Current balance in USD |

### `transactions.csv`

| Column | Type | Description |
|--------|------|-------------|
| `transaction_id` | string | Unique transaction identifier (e.g., `TXN0001`) |
| `account_id` | string | References `accounts.account_id` |
| `date` | date | Transaction date, YYYY-MM-DD format |
| `amount` | decimal | Transaction amount in USD (always positive) |
| `type` | string | One of the types listed below |
| `description` | string | Free-text description |

**Transaction types:**

| Type | Direction | Description |
|------|-----------|-------------|
| `cash_deposit` | Credit | Cash deposited at a teller window |
| `cash_withdrawal` | Debit | Cash withdrawn at a teller window |
| `ach_credit` | Credit | Incoming ACH transfer (e.g., payroll, direct deposit) |
| `ach_debit` | Debit | Outgoing ACH transfer (e.g., bill pay, autopay) |
| `check_deposit` | Credit | Check deposited |
| `check_withdrawal` | Debit | Check written/cashed |
| `wire_incoming` | Credit | Incoming wire transfer |
| `wire_outgoing` | Debit | Outgoing wire transfer |

---

## Detection Rules

You need to implement **two** detection rules and report which customers (if any) trigger each one.

### Rule 1: Cash Structuring

**What is structuring?** Under the Bank Secrecy Act ([31 USC 5324](https://www.law.cornell.edu/uscode/text/31/5324)), financial institutions must file a Currency Transaction Report (CTR) for any cash transaction of **$10,000 or more**. *Structuring* is the illegal practice of deliberately breaking up cash transactions into smaller amounts to avoid triggering this reporting requirement.

**Detection logic:**

For each customer, look at their **cash transactions** (`cash_deposit` and `cash_withdrawal`) over a **rolling 30-day window**. Analyze deposits and withdrawals **separately** — a customer could be structuring deposits, withdrawals, or both.

Flag a customer when **all three** conditions are true within a 30-day window:

1. **Multiple transactions:** At least 2 cash transactions of the same direction (deposit or withdrawal)
2. **All under the reporting threshold:** Every individual transaction is **under $10,000** (i.e., no single transaction is $10,000 or more)
3. **Combined total exceeds the threshold:** The sum of those transactions is **greater than $10,000**

In other words: if someone makes several cash deposits that are each conveniently below $10,000, but together add up to more than $10,000 within 30 days, that's a structuring alert. A single large cash transaction (even $9,999) is not structuring — it takes a *pattern* of multiple sub-threshold transactions.

### Rule 2: ACH Deviation

**What is an ACH deviation?** A sudden, significant departure from a customer's normal ACH (Automated Clearing House) transaction pattern can indicate fraud, money laundering, or account takeover. We detect this statistically.

**Detection logic:**

For each account, on each day with ACH activity:

1. Calculate the **rolling 30-day average** and **standard deviation** of daily ACH totals for that account
2. Analyze ACH credits and ACH debits **separately**
3. Flag when a single day's total exceeds: **`rolling_average + (3 × standard_deviation)`**
4. AND the day's total exceeds a **minimum threshold of $10,000** (to avoid flagging noise on small accounts)

Days with no ACH activity for a given account should be treated as **$0** when computing the rolling average and standard deviation — do not skip them.

---

## Your Task

1. **Ingest** the three CSV files into a local database or data structure of your choice (SQLite, PostgreSQL, pandas, DuckDB, etc.)
2. **Implement** both detection rules as described above
3. **Produce a report** for each flagged customer containing:
   - Customer name and ID
   - Which rule they triggered
   - The specific transactions that triggered the alert
   - Supporting evidence (totals, rolling averages, deviations, date windows)
4. **Document** your approach and any assumptions you made

### Deliverables

- Your source code (any language)
- The output report (text, JSON, or formatted table — your choice)
- Brief notes on your approach and any design decisions

---

## Bonus (Optional)

If you finish the core task and want to go further:

- What improvements would you suggest to reduce false positives?
- How would you handle joint accounts (multiple customers sharing one account)?
- What additional detection rules might be valuable for a BSA program?
- How would you operationalize these rules for daily automated monitoring?

---

## Time Expectation

This challenge is designed to take **2–4 hours**. Focus on correctness and clear communication of your findings over code elegance.

## Submission

Please submit your solution privately — do not fork this repo or post your solution publicly. Email your code and report as a zip file, or share a link to a private repository.

## Project Goals
**Build a personal finance application that allows users to track income and expenses and understand their spending patterns.**

Write User Stories

User stories describe **what the user wants and why**.

Format:

```text
As a [user],
I want to [action],
so that [reason/value].
```

---

## Authentication

### Sign Up

> As a user, I want to create an account so that I can securely manage my personal finances.

### Login

> As a user, I want to log in so that I can access my financial information.

### Logout

> As a user, I want to log out so that I can protect my financial information when I am finished using the application.

### Password Recovery

> As a user, I want to recover my password so that I can regain access to my account if I forget it.

---

## Account Management

### Create Account

> As a user, I want to create a financial account so that I can track money held in different accounts.

### View Accounts

> As a user, I want to view my financial accounts so that I can see where my money is held.

### View Account

> As a user, I want to view an individual account so that I can see its current financial activity.

### Edit Account

> As a user, I want to edit my account details so that I can correct or update information.

### Delete Account

> As a user, I want to delete an account so that I can remove an account I no longer use.

---

## Category Management

### Create Category

> As a user, I want to create a custom category so that I can organize transactions according to my personal spending and income types.

### View Categories

> As a user, I want to view my categories so that I can choose the correct category when creating a transaction.

### Edit Category

> As a user, I want to edit a category so that I can correct or update its information.

### Delete Category

> As a user, I want to delete a category so that I can remove categories I no longer need.

---

## Transaction Management

### Create Transaction

> As a user, I want to create a transaction so that I can record money entering or leaving my account.

### Specify Transaction Type

> As a user, I want to specify whether a transaction is income or expense so that my financial activity can be categorized correctly.

### Assign Account

> As a user, I want to assign a transaction to an account so that I can track where the money came from or where it was spent.

### Assign Category

> As a user, I want to assign a transaction to a category so that I can understand what the transaction was related to.

### View Transactions

> As a user, I want to view my transaction history so that I can review my financial activity.

### View Transaction

> As a user, I want to view a transaction's details so that I can understand exactly what was recorded.

### Edit Transaction

> As a user, I want to edit transaction details so that I can correct mistakes.

### Delete Transaction

> As a user, I want to delete a transaction so that I can remove an incorrectly recorded transaction.

---

## Transaction Filtering

### Filter by Account

> As a user, I want to filter transactions by account so that I can review activity for a specific account.

### Filter by Category

> As a user, I want to filter transactions by category so that I can review spending or income related to a specific category.

### Filter by Type

> As a user, I want to filter transactions by income or expense so that I can view only the type of activity I need.

### Filter by Date

> As a user, I want to filter transactions by date range so that I can review my financial activity for a specific period.

---

## Dashboard

### View Balance

> As a user, I want to see my current account balances so that I know how much money I currently have in each account.

### View Total Income

> As a user, I want to see my total income so that I know how much money I received during a period.

### View Total Expenses

> As a user, I want to see my total expenses so that I know how much money I spent during a period.

### View Recent Transactions

> As a user, I want to see my recent transactions so that I can quickly review my latest financial activity.

---

## Reports

### Monthly Income

> As a user, I want to see my income for a specific month so that I can understand how much money I received during that period.

### Monthly Expenses

> As a user, I want to see my expenses for a specific month so that I can understand how much money I spent during that period.

### Spending by Category

> As a user, I want to view my expenses by category so that I can understand where I spend most of my money.

### Account Activity

> As a user, I want to view the financial activity of an account so that I can understand how money has moved through that account.

---

## Functional Requirementsuser
### User 
```
can:
- Register
- Login
- Create category
- Create expense
- Update expense
- Delete expense
- View expenses
- Create income
- View income
- Create budget
- View reports
```

## Business Rules

```
- Expense amount must be greater than 0.
- Income amount must be greater than 0.
- A user can only access their own expenses.
- An expense must belong to a category.
- A category belongs to one user.
- A deleted category cannot have active expenses.
```
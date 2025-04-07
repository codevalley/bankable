Okay, let's break this down into the two documents you requested: MVP Definition and Backend Architecture & Design.

---

## Document 1: Family Bank - MVP Definition

### 1. Introduction

This document defines the scope and use cases for the Minimum Viable Product (MVP) of the Family Bank system. The goal of the MVP is to provide the core functionality for a single family to manage a simple, gamified financial system where children can earn and spend a virtual currency, overseen by a parent. This initial version will be API-first (headless).

### 2. User Roles

* **Parent (Banker):** The administrator of the family's instance. Responsible for setting up the system, defining rules (Products), managing users, approving transactions, and overseeing balances.
* **Child (Accountholder):** A participant in the system who can earn currency by completing tasks and spend currency on privileges. In the MVP, their actions (logging tasks, requesting spending) are initiated via API calls, likely made *by the Parent* on their behalf initially.

### 3. MVP Use Cases

| ID    | Use Case Name                    | Actor(s)      | Description                                                                                                                               | Outcome                                                                                                  |
| :---- | :------------------------------- | :------------ | :---------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------- |
| UC1   | Register Family & Parent         | Parent        | Creates the initial Family Bank instance and the first Parent user account.                                                               | Family record created, Parent user record created and authenticated.                                     |
| UC2   | Add Child Account                | Parent        | Creates a new Child user account linked to the Parent's family.                                                                           | Child user record created with a starting balance (likely zero).                                         |
| UC3   | Define Product                   | Parent        | Creates a new item that can be earned (Task) or purchased (Privilege). Specifies name, description, type (Earn/Spend), and amount (value/cost). | Product record created and available for transactions.                                                   |
| UC4   | Edit Product                     | Parent        | Modifies the details (name, description, amount) of an existing Product.                                                                    | Product record updated. (Note: Does not affect past transactions).                                       |
| UC5   | Delete Product                   | Parent        | Marks a Product as inactive (soft delete) so it can no longer be used for new transactions.                                                 | Product record marked inactive.                                                                          |
| UC6   | List Products                    | Parent        | Views all active Products (both Earn and Spend types) defined for the family.                                                               | List of active Products returned.                                                                        |
| UC7   | Request Earning (Log Task)       | Child (via API) | Logs the completion of an "Earn" type Product (Task), creating a request for currency.                                                      | A *Pending* "Earn" Transaction record is created. Child's balance is *not yet* affected.                   |
| UC8   | Request Spending (Use Privilege) | Child (via API) | Logs the request to use/purchase a "Spend" type Product (Privilege), creating a request to deduct currency.                                 | A *Pending* "Spend" Transaction record is created. Child's balance is *not yet* affected.                  |
| UC9   | View Pending Transactions        | Parent        | Views all transactions currently in the *Pending* state for their family.                                                                   | List of Pending Transactions returned.                                                                   |
| UC10  | Approve Transaction              | Parent        | Approves a *Pending* transaction (Earn or Spend).                                                                                         | Transaction status updated to *Approved*. Child's account balance updated accordingly. Audit details logged. |
| UC11  | Reject Transaction               | Parent        | Rejects a *Pending* transaction.                                                                                                          | Transaction status updated to *Rejected*. Child's account balance remains unchanged.                     |
| UC12  | View Child Details & Balance     | Parent        | Views the profile information and current account balance of a specific child in their family.                                            | Child user details and current balance returned.                                                         |
| UC13  | Manually Adjust Balance          | Parent        | Directly increases or decreases a Child's balance for reasons outside standard Product transactions (e.g., initial setup, correction).      | An *Approved* "Adjustment" Transaction record is created. Child's account balance updated immediately.     |
| UC14  | View Transaction History         | Parent, Child | Views the history of non-pending transactions for a specific child. Can filter by date range and transaction type (Earn, Spend, Adjustment). | List of relevant Transaction records returned.                                                           |
| UC15  | Cancel Approved Transaction      | Parent        | Cancels/reverses a previously *Approved* transaction (e.g., if approved by mistake). Creates a corresponding reversal transaction.          | Original Transaction marked *Cancelled*. A new *Approved* "Reversal" Transaction is created, adjusting the balance back. |
| UC16  | Modify Pending Transaction       | Parent        | Modifies details (e.g., description, amount) of a transaction that is still in the *Pending* state.                                       | Pending Transaction record updated.                                                                      |
| UC17  | Authenticate Parent              | Parent        | Logs into the system using credentials.                                                                                                   | Authentication token (e.g., JWT) returned for use in subsequent API calls.                               |

### 4. Explicitly Excluded from MVP

* Dynamic pricing, inflation simulation.
* Inter-sibling or inter-family currency transfers/exchanges.
* Automated system-wide events (e.g., Tax Day, scheduled fees).
* Penalties as a distinct feature (can use manual adjustment).
* Appeals mechanism for children.
* Multi-family support and currency exchange rates.
* Advanced reporting, analytics, dashboards, gamification (leaderboards, achievements).
* Dedicated frontend UI (Web or Mobile App).
* Direct Child login and interaction.
* Automated proof of task completion (photos, QR codes).
* Push or email notifications.
* Complex transaction filtering (e.g., by specific product within history).
* Transaction Replay / Retrospective Recalculation (Deferred due to high complexity in managing rule versioning and ledger immutability).
* Advanced product features (fees, taxes - can be bundled into cost/reward for MVP).
* Inventory management for rewards.

---

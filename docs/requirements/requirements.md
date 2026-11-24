# Requirements

## Problem Summary

## Stakeholders

## User Stories

### US-1: Customer registration with a basic verification level (FR-1, Must)
**I want to** register for a PayLink account,
**so that** I get a wallet and can start paying bills within the BASIC limits.

> Scenario 1: **Successful registration**\
**Given** a person provides a valid unregistered name, phone number and password\
**When** they submit the registration request\
**then** a customer account is created with verification level BASIC\
**And** an empty EGP wallet is created for the new user\
**And** the daily payment limit of 5,000 EGP applies to the account\

> Scenario 2: **Phone number already registered**\
**Given** a customer account already exists with phone number 01#########\
**When** someone tries to register again with the same phone number\
**Then** the registration is rejected with a "phone number already registered" error\
**And** no new account or wallet is created.

> Scenario 3: **Invalid input**\
**Given** a person submits a registration with a missing name, a malformed phone number or a weak password\
**When** the registration request is processed\
**Then** it is rejected with a validation error that names each invalid field\
**And** no new account or wallet is created\


### US-2: Upgrade a customer to verified (FR-2, Should)
**As** a BASIC customer,
**I want to** be upgraded to a VERIFIED level,
**so that** my limits are raised and I can pay larger bills.

> Scenario 1: **Upgrade approved**\
**Given** a registered customer with verification level BASIC requests level upgrade\
**When** the upgrade request is approved by an admin\
**Then** the customer's verification level becomes VERIFIED\
**And** their daily payment limit changes from 5,000 EGP to 30,000 EGP\
**And** audit log entry records who approved, when and why to satisfy FR-25\

> Scenario 2: **Already VERIFIED**\
**Given** a customer whose level is already VERIFIED\
**When** an upgrade is requested again\
**Then** the request is ignored with an "already verified" message\
**And** the account is unchanged\

> Scenario 3: **Upgrade not approved**\
**Given** a registered customer with verification level BASIC requests level upgrade\
**When** the upgrade request is rejected by an admin\
**Then** the level stays BASIC and the 5,000 EGP limit remains unchanged\
**And** the customer is sent a message with the reason behind the rejection\
**And** audit log entry records who rejected, when and why to satisfy FR-25\


### US-3: View wallet balance and transaction history (FR-3, Must)
**As** a customer,
**I want to** view my wallet balance (available and held) and my transaction history,
**so that** I know how much money I can spend and can check what happened to my payments.

> Scenario 1: **View balance**\
**Given** a registered customer with a wallet\
**When** they request their wallet balance\
**Then** the response shows the available balance and the held balance in EGP, each with two decimal places\
**And** both values are computed from the ledger entries (FR-5), not read from a stored balance field\

> Scenario 2: **Balance with a pending payment**\
**Given** a customer whose payment of 352.50 EGP is in state FundsHeld or SentToBiller\
**When** they request their wallet balance\
**Then** the 352.50 EGP appears in the held balance not in the available balance\

> Scenario 3: **View transaction history**\
**Given** a customer with several ledger entries\
**When** they request their transaction history\
**Then** the entries are returned newest first\
**And** each entry shows its date and time, type, amount, resulting payment reference (if any) and status\

> Scenario 4: Empty history\
**Given** a newly registered customer with no ledger entries\
**When** they request their balance and history\
**Then** both balances are 0.00 EGP\
**And** the history is an empty list, not an error\

> Scenario 5: Access restricted to the owner\
**Given** two customers, A and B\
**When** customer A requests the wallet or history of customer B\
**Then** the request is refused as forbidden\
**And** no data about B is returned\


### US-4: Top up the wallet (FR-4, Must)
**As** a customer,
**I want to** top up my wallet with a simulated card payment or agent cash,
**so that** I have money available to pay my bills.

> Scenario 1:
**Given** 
**When** 
**Then** 
**And**

> Scenario 2:
**Given**
**When** 
**Then** 
**And**


### US-5:
**As**
**I want to**
**so that**

> Scenario 1:
**Given**
**When** 
**Then** 
**And**

> Scenario 2:
**Given**
**When** 
**Then** 
**And**


### US-6:
**As**
**I want to**
**so that**

> Scenario 1:
**Given**
**When** 
**Then** 
**And**

> Scenario 2:
**Given**
**When** 
**Then** 
**And**


### US-7:
**As**
**I want to**
**so that**

> Scenario 1:
**Given**
**When** 
**Then** 
**And**

> Scenario 2:
**Given**
**When** 
**Then** 
**And**



## Non-functional Requirements

## Priorities

## Questions or ambiguities
**Q1. What identifies a customer (phone number, email, or both)?**\
Decision: Phone number is the unique identifier.

**Q2. Who triggers the upgrade (admin or customer)?**\
Decision: The customer requests it and an admin approves it.

**Q3. should history include only completed movements or also holds and releases?**\
Decision: All ledger entries, so customers can see held money.


### US-
**As**
**I want to**
**so that**


> Scenario 1:
**Given**
**When** 
**Then** 
**And**

# Requirements

## Problem Summary

## Stakeholders

## User Stories

### US-1: Customer registration with a basic verification level (FR-1, Must)
**As** a new customer,
**I want to** register for a PayLink account,
**so that** I get a wallet and can start paying bills within the BASIC limits.

> Scenario 1: **Successful registration**\
**Given** a person provides a valid unregistered name, phone number and password\
**When** they submit the registration request\
**then** a customer account is created with verification level BASIC\
**And** an empty EGP wallet is created for the new user\
**And** the daily payment limit of 5,000 EGP applies to the account

> Scenario 2: **Phone number already registered**\
**Given** a customer account already exists with phone number 01#########\
**When** someone tries to register again with the same phone number\
**Then** the registration is rejected with a "phone number already registered" error\
**And** no new account or wallet is created.

> Scenario 3: **Invalid input**\
**Given** a person submits a registration with a missing name, a malformed phone number or a weak password\
**When** the registration request is processed\
**Then** it is rejected with a validation error that names each invalid field\
**And** no new account or wallet is created


### US-2: Upgrade a customer to verified (FR-2, Should)
**As** a BASIC customer,
**I want to** be upgraded to a VERIFIED level,
**so that** my limits are raised and I can pay larger bills.

> Scenario 1: **Upgrade approved**\
**Given** a registered customer with verification level BASIC requests level upgrade\
**When** the upgrade request is approved by an admin\
**Then** the customer's verification level becomes VERIFIED\
**And** their daily payment limit changes from 5,000 EGP to 30,000 EGP\
**And** audit log entry records who approved, when and why to satisfy FR-25

> Scenario 2: **Already VERIFIED**\
**Given** a customer whose level is already VERIFIED\
**When** an upgrade is requested again\
**Then** the request is ignored with an "already verified" message\
**And** the account is unchanged

> Scenario 3: **Upgrade not approved**\
**Given** a registered customer with verification level BASIC requests level upgrade\
**When** the upgrade request is rejected by an admin\
**Then** the level stays BASIC and the 5,000 EGP limit remains unchanged\
**And** the customer is sent a message with the reason behind the rejection\
**And** audit log entry records who rejected, when and why (FR-25)


### US-3: View wallet balance and transaction history (FR-3, Must)
**As** a customer,
**I want to** view my wallet balance (available and held) and my transaction history,
**so that** I know how much money I can spend and can check what happened to my payments.

> Scenario 1: **View balance**\
**Given** a registered customer with a wallet\
**When** they request their wallet balance\
**Then** the response shows the available balance and the held balance in EGP, each with two decimal places\
**And** both values are computed from the ledger entries (FR-5), not read from a stored balance field

> Scenario 2: **Balance with a pending payment**\
**Given** a customer whose payment of 352.50 EGP is in state FundsHeld or SentToBiller\
**When** they request their wallet balance\
**Then** the 352.50 EGP appears in the held balance not in the available balance

> Scenario 3: **View transaction history**\
**Given** a customer with several ledger entries\
**When** they request their transaction history\
**Then** the entries are returned newest first\
**And** each entry shows its date and time, type, amount, resulting payment reference (if any) and status

> Scenario 4: **Empty history**\
**Given** a newly registered customer with no ledger entries\
**When** they request their balance and history\
**Then** both balances are 0.00 EGP\
**And** the history is an empty list, not an error

> Scenario 5: **Access restricted to the owner**\
**Given** two customers, A and B\
**When** customer A requests the wallet or history of customer B\
**Then** the request is refused as forbidden\
**And** no data about B is returned


### US-4: Top up the wallet (FR-4, Must)
**As** a customer,
**I want to** top up my wallet with a simulated card payment or agent cash,
**so that** I have money available to pay my bills.

> Scenario 1: **Top-up by simulated card**\
**Given** a registered customer with a wallet\
**When** they submit a top-up of a valid amount using a simulated card that is accepted\
**Then** the amount is added to the available balance\
**And** a TOPUP ledger entry is created for that amount (FR-5)\
**And** an audit log entry records who, when and why (FR-25)

> Scenario 2: **Top-up by agent cash**\ 
**Given** an agent and a registered customer who hands the agent cash\
**When** the agent submits a top-up for that customer's wallet\
**Then** the amount is added to the customer's available balance\
**And** a TOPUP ledger entry records the agent as the source (FR-5)\
**And** an audit log entry records who, when and why (FR-25)

> Scenario 3: **Card declined**\
**Given** a customer tops-up with a simulated card that is declined\
**When** they submit the top-up\
**Then** it is rejected with a clear reason\
**And** no ledger entry is created and the balance is unchanged

> Scenario 4: **Invalid amount**\
**Given** a customer tops-up below the configured minimum (10 EGP) or an amount with more than two decimal places\
**When** they submit the top-up\
**Then** it is rejected with a validation error naming the minimum\
**And** no ledger entry is created and the balance is unchanged

> Scenario 5: **Daily top-up limit exceeded**\
**Given** a customer whose top-ups today plus the new amount would exceed the configured daily top-up limit for their level (Basic or Verified)\
**When** they submit the top-up\
**Then** it is rejected with a "daily top-up limit exceeded" reason\
**And** no ledger entry is created and the balance is unchanged

> Scenario 6: **Repeated top-up request**\
**Given** a top-up request was already processed with a given idempotency key\
**When** the same request is sent again with the same key\
**Then** the original result is returned\
**And** the wallet is credited only once

### US-5: Ledger-based balances (FR-5, Must)
**As** a finance person,
**I want** every balance change to be recorded as a ledger entry and every balance to be computed from those entries,
**so that** money movements can be traced, audited and verified.

> Scenario 1: **Every balance change creates an entry**\
**Given** a wallet with a known balance\
**When** any money movement happens (top-up, hold, charge, release, refund, fee, commission)\
**Then** exactly one new ledger entry is created for it\
**And** the entry records the wallet, type, amount, balance bucket (available or held), related payment reference (if any), timestamp and who caused it

> Scenario 2: **Balance is computed from the ledger**\
**Given** a wallet with several ledger entries\
**When** the available and held balances are requested\
**Then** each balance equals the sum of that wallet's entries for its bucket\
**And** there is no stored balance field that can be edited directly

> Scenario 3: **Entries are immutable**\
**Given** an existing ledger entry\
**When** anyone attempts to edit or delete it\
**Then** the attempt is refused\
**And** the entry is unchanged

> Scenario 4: **Corrections are new entries**\
**Given** a wrong or cancelled money movement\
**When** it is corrected\
**Then** a new reversing entry is added\
**And** the original entry stays in the history


### US-6: Manage billers and services (FR-6, Must)
**As** an admin,
**I want to** add, edit and disable billers and their services,
**so that** the catalog stays accurate and I can onboard or suspend a biller without changing code.

> Scenario 1: **Add a biller**\
**Given** an admin and a new biller that does not exist in the catalog\
**When** the admin submits the biller's name and integration type/
**Then** the biller is created as active\
**And** an audit log entry records who, when and why (FR-25)

> Scenario 2: **Add a service to a biller**\
**Given** an existing biller\
**When** he admin adds a service with its name, required input fields, minimum and maximum amounts\
**Then** the service is saved under that biller\
**And** it is visible to customers in the catalog (FR-7)

> Scenario 3: **Edit a biller or service**\
**Given** an existing biller or service\
**When** he admin changes its details, such as the required fields or the amount limits\
**Then** the changes are saved and apply to new payments from that point\
**And** payments already in progress are not affected\
**And** an audit log entry records the old and new values

> Scenario 4: **Disable a biller or service**\
**Given** an active biller or service\
**When** the admin disables it\
**Then** no longer appears to customers and new payments to it are rejected with a "service unavailable" message\
**And** payments already in progress continue through their life cycle\
**And** its history is kept, not deleted


### US-7: Browse billers and required input fields (FR-7, Must)
**As** customer,
**I want to** see the list of available billers and the input fields each one requires,
**so that** I know what information to enter before I pay a bill.

> Scenario 1: **View the biller list**\
**Given** a logged-in customer and several active billers in the catalog\
**When** they request the list of billers\
**Then** the response shows each active biller with its name and its services

> Scenario 2: **View required input fields**\
**Given** a customer who selects a service\
**When** they request its details\
**Then** the response lists the required input fields with their names and types\
**And** it shows the service's minimum and maximum amounts

> Scenario 2: **Catalog changes are visible without redeploying**\
**Given** an admin has just disabled, added a biller or edited a service's fields\
**When** a customer requests the list again\
**Then** the response reflects the change


## Non-functional Requirements

## Priorities

## Questions or ambiguities
**Q1. What identifies a customer (phone number, email, or both)? US-1**\
Decision: Phone number is the unique identifier.

**Q2. Who triggers the upgrade (admin or customer)? US-2**\
Decision: The customer requests, an admin approves or rejects with a reason; request states are Pending, Approved, Rejected

**Q3. should history include only completed movements or also holds and releases? US-3**\
Decision: All ledger entries, so customers can see held money.

**Q4. Is there a minimum, maximum or daily limit for top-ups? US-4**\
Decision: A minimum of 10 EGP per top-up and a daily limit for top-ups of 30,000 EGP.

**Q5. Does an agent's cash top-up count toward the customer's daily limit, and does it take money from the agent's wallet? US-4**\
Decision: Yes, it counts toward the limit; No, it doesn't take from agent's wallet.

**Q6. Does a hold create one ledger entry or two? US-5**\
Decision: Two entries sharing a transaction ID, written in one database transaction.

**Q7. Can a disabled biller be re-enabled? US-6**\
Decision: Yes.

**Q8. Service min and max versus the global 5 to 10,000 EGP rule: which wins? US-6**\
Decision: The stricter applies; admins cannot save a service range outside the global range.

**Q9. should the list be public or only for logged-in customers? US-7**\
Decision: logged-in.



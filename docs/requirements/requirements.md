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
**When** the admin submits the biller's name and integration type\
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


**### US-8: Perform Bill Inquiry (FR-8, Must)**

**As** a customer,  
**I want to** perform a bill inquiry,  
**so that** I can see the bill amount, service fee, and total amount before making a payment.

> Scenario 1: **Successful Electricity Bill Inquiry**
**Given** a customer enters a valid electricity meter number and the bill amount is 350.00 EGP.
**When** the customer performs a bill inquiry.
**Then** the system displays:
Bill Amount: 350.00 EGP
Service Fee: 2.50 EGP
Total Amount: 352.50 EGP

> Scenario 2: **Successful Water Bill Inquiry**
**Given** a customer enters a valid water account number and the bill amount is 120.75 EGP.
**When** the customer performs a bill inquiry.
**Then** the system displays:
Bill Amount: 120.75 EGP
Service Fee: 2.50 EGP
Total Amount: 123.25 EGP

> Scenario 3: **Invalid Account Information**
**Given** a customer enters an invalid meter or account number.
**When** the customer performs a bill inquiry.
**Then** the system displays "Bill not found".
**And** no payment total is displayed.

> Scenario 4: **Biller Unavailable**
**Given** the biller does not respond to the inquiry request.
**When** the customer performs a bill inquiry.
**Then** no payment total is displayed.
**And** the system displays "Biller unavailable, try again later".

---

**### US-9: Wallet Bill Payment (FR-9, Must)**

**As** a customer,  
**I want to** pay bills using my wallet,  
**so that** I can complete bill payments and ensure my funds are handled correctly throughout the payment process.

> Scenario 1: **Funds Held When Payment Is Confirmed**
**Given** the available balance is 1,000.00 EGP and the held balance is 0.00 EGP.
**And** the bill total is 352.50 EGP.
**When** the customer confirms the payment.
**Then** the payment state becomes FundsHeld.

> Scenario 2: **Biller Confirms the Payment**
**Given** a payment is in SentToBiller and 352.50 EGP is held.
**When** the biller confirms the payment.
**Then** the payment state becomes Completed.
**And** the held balance becomes 0.00 EGP and the available balance stays 647.50 EGP.
**And** 350.00 EGP is paid to the biller and 2.50 EGP is credited to PayLink's fee income.
**And** ledger entries and audit records are written for the final charge and the fee income.

> Scenario 3: **Insufficient Funds**
**Given** the available balance is 100.00 EGP and the bill total is 352.50 EGP.
**When** the customer attempts to pay.
**Then** the payment state becomes Rejected.
**And** the system displays "Insufficient funds".
**And** both balances remain unchanged and the biller is not called.

> Scenario 4: **Amount Above the Maximum (Both Account Types)**
**Given** the maximum single payment is 10,000.00 EGP.
**When** the customer attempts to pay a bill amount of 10,000.01 EGP.
**Then** the payment state becomes Rejected.
**And** the system displays "Amount above the 10,000.00 maximum".
**And** both balances remain unchanged.

> Scenario 5: **Amount Below the Minimum (Both Account Types)**
**Given** the minimum single payment is 5.00 EGP.
**When** the customer attempts to pay a bill amount of 4.99 EGP.
**Then** the payment state becomes Rejected.
**And** the system displays "Amount below the 5.00 minimum".
**And** both balances remain unchanged.

> Scenario 6: **Two Concurrent Payments Exceed Available Balance**
**Given** a customer has 500 EGP available in their wallet.
**And** each payment requires 352.50 EGP, including fees.
**When** the customer initiates both payments at nearly the same time.
**Then** only one payment can successfully reserve 352.50 EGP.
**And** the other payment is rejected due to insufficient available balance.
**And** the customer's available balance must never become negative.

---

**### US-10: Idempotent Payment Requests (FR-10, Must)**

**As** a customer,  
**I want to** repeated payment requests with the same idempotency key to return the original result,  
**so that** I can safely retry requests without being charged multiple times.

> Scenario 1: **Duplicate Request with Identical Parameters**
**Given** a payment request with idempotency key K1 has already been processed successfully and its payment state is Completed.
**When** the customer submits the same payment request again using K1 with identical parameters.
**Then** the system returns the original payment result without creating a new payment or deducting funds again.

> Scenario 2: **Duplicate Request with Different Parameters**
**Given** a payment request with idempotency key K1 has already been submitted with an amount of 352.50 EGP.
**When** the customer submits another request using K1 but with a different amount or bill reference.
**Then** the system rejects the request with an idempotency key conflict error without modifying the original payment or wallet balance.

> Scenario 3: **Duplicate Request While Payment Is Processing**
**Given** a payment request with idempotency key K1 has already been accepted and is in FundsHeld, SentToBiller, or Unknown state.
**When** the customer submits the same request again using K1 with identical parameters.
**Then** the system returns the existing payment reference and current payment status without creating another payment, placing an additional hold, or automatically refunding an Unknown payment.

---

**### US-11: Payment State Machine (FR-11, Must)**

**As** the system,  
**I want to** move a payment only through the transitions allowed in section 2.5 and refuse all others,  
**so that** every payment is in exactly one valid state and its history can be trusted.

> Scenario 1: **Initiated Moves to FundsHeld**
**Given** a payment is in Initiated.
**When** the fraud checks pass and the wallet has enough funds.
**Then** the payment state becomes FundsHeld.

> Scenario 2: **Initiated Moves to Rejected**
**Given** a payment is in Initiated.
**When** a fraud check fails or the wallet does not have enough funds.
**Then** the payment state becomes Rejected.

> Scenario 3: **FundsHeld Moves to SentToBiller**
**Given** a payment is in FundsHeld.
**When** the request is sent to the biller.
**Then** the payment state becomes SentToBiller.

> Scenario 4: **SentToBiller Moves to Completed**
**Given** a payment is in SentToBiller.
**When** the biller confirms the payment.
**Then** the payment state changes to Completed.

> Scenario 5: **SentToBiller Moves to Failed**
**Given** a payment is in SentToBiller.
**When** the biller refuses the payment.
**Then** the payment state changes to Failed.

> Scenario 6: **SentToBiller Moves to Unknown**
**Given** a payment is in SentToBiller.
**When** the biller does not respond within the timeout.
**Then** the payment state changes to Unknown.

> Scenario 7: **Unknown Moves to Completed**
**Given** a payment is in Unknown.
**When** the biller later confirms the payment.
**Then** the payment state changes to Completed.

> Scenario 8: **Unknown Moves to Failed**
**Given** a payment is in Unknown.
**When** the biller later refuses the payment or reconciliation finds no payment.
**Then** the payment state changes to Failed.

> Scenario 9: **Failed Moves to Refunded**
**Given** a payment is in Failed.
**When** the wallet hold is released.
**Then** the payment state changes to Refunded.

> Scenario 10: **Prevent Transitions from Final States**
**Given** a payment is in Completed, Rejected, or Refunded.
**When** the system attempts any further state transition.
**Then** the transition is rejected and the payment remains in its current state.

---

**### US-12: Wallet Hold and Release (FR-12, Must)**

**As** a customer,  
**I want to** have my money held before the biller is called and released if the payment fails,  
**so that** I cannot spend the same money twice and I never lose money for a payment that did not happen.

> Scenario 1: **Funds Are Held Before the Biller Is Called**
**Given** the available balance is 1,000.00 EGP and the held balance is 0.00 EGP.
**When** the customer pays a total of 352.50 EGP.
**Then** the available balance becomes 647.50 EGP and the held balance becomes 352.50 EGP.
**And** a ledger entry records the hold of 352.50 EGP.
**And** the request is sent to the biller only after this hold exists.

> Scenario 2: **Hold Is Released When the Biller Refuses**
**Given** a payment is in SentToBiller and 352.50 EGP is held.
**When** the biller refuses the payment.
**Then** the payment state becomes Failed.
**And** the hold is released, so the available balance returns to 1,000.00 EGP and the held balance becomes 0.00 EGP.
**And** the payment state becomes Refunded.
**And** a ledger entry records the release of 352.50 EGP.

> Scenario 3: **Hold Is Kept While the Payment Is Unknown**
**Given** a payment is in Unknown and 352.50 EGP is held.
**When** no confirmation or refusal has arrived from the biller.
**Then** the held balance stays 352.50 EGP.
**And** no release ledger entry is created and no refund is made.

---

**### US-13: Timeout and Re-query (FR-13, Must)**

**As** a customer,  
**I want to** have a payment that times out kept as Unknown and re-queried automatically,  
**so that** I am not charged twice or refunded by mistake while the biller's answer is still unknown.

> Scenario 1: **Biller Times Out**
**Given** a payment is in SentToBiller and 352.50 EGP is held.
**When** the biller does not answer within 10 seconds (for example ElectroCo answers after 12 seconds).
**Then** the payment state becomes Unknown.
**And** the customer sees "Your payment is being processed".
**And** the held balance stays 352.50 EGP and no refund is made.

> Scenario 2: **Re-query Schedule**
**Given** a payment became Unknown at 10:00.
**When** the time reaches the scheduled re-queries.
**Then** the system re-queries the biller after 1, 5 and 15 minutes (up to 3 times).
**And** each re-query uses the original payment reference and does not create a new payment or a new hold.

> Scenario 3: **Re-query Shows the Biller Confirmed**
**Given** a payment is in Unknown.
**When** a re-query shows that the biller confirmed the payment.
**Then** the payment state becomes Completed.
**And** the held balance becomes 0.00 EGP and no further re-queries are made.

> Scenario 4: **Re-query Shows the Biller Refused**
**Given** a payment is in Unknown and 352.50 EGP is held.
**When** a re-query shows that the biller refused the payment.
**Then** the payment state becomes Failed and then Refunded.
**And** the held 352.50 EGP is released back to the available balance.

> Scenario 5: **All Re-queries Get No Answer**
**Given** a payment is in Unknown.
**When** the re-queries after 1, 5 and 15 minutes all get no answer.
**Then** the payment state stays Unknown and no more re-queries are made.
**And** the held balance stays 352.50 EGP and the payment waits for reconciliation.
**And** no automatic refund is made.

---

**### US-14: TopUpNet Callback Handling (FR-14, Must)**

**As** the system,  
**I want to** accept TopUpNet's asynchronous callbacks and ignore duplicates,  
**so that** a mobile top-up reaches its final state exactly once and a customer is never charged or credited twice.

> Scenario 1: **Success Callback Completes the Payment**
**Given** TopUpNet replied "accepted" and the payment is in SentToBiller with 352.50 EGP held.
**When** TopUpNet calls the callback URL with "success" and the payment reference.
**Then** the payment state becomes Completed.
**And** the held balance becomes 0.00 EGP and one final charge ledger entry is created.

> Scenario 2: **Failed Callback Fails and Refunds the Payment**
**Given** TopUpNet replied "accepted" and the payment is in SentToBiller with 352.50 EGP held.
**When** TopUpNet calls the callback URL with "failed" and the payment reference.
**Then** the payment state becomes Failed and then Refunded.
**And** the held 352.50 EGP is released back to the available balance.

> Scenario 3: **Duplicate Success Callback Is Ignored**
**Given** a payment is already Completed by a "success" callback.
**When** TopUpNet sends a second "success" callback with the same payment reference.
**Then** the payment state stays Completed.
**And** no new ledger entry is created.
**And** the duplicate is logged with the payment reference and the time.

> Scenario 4: **Conflicting Callback Is Ignored**
**Given** a payment is already Completed by a "success" callback.
**When** TopUpNet sends a "failed" callback with the same payment reference.
**Then** the payment state stays Completed and no balance changes.
**And** the conflict is logged.

> Scenario 5: **Callback Resolves an Unknown Payment**
**Given** a payment is in Unknown with 352.50 EGP held.
**When** TopUpNet calls the callback URL with "success".
**Then** the payment state becomes Completed and the held balance becomes 0.00 EGP.

> Scenario 6: **Callback With an Unknown Reference**
**Given** no payment exists with the reference "P-999".
**When** TopUpNet calls the callback URL with that reference.
**Then** no payment or balance changes.
**And** the system returns an error "Payment reference not found" and logs the callback.

## User Story 15
**Priority:** Must

**As** a customer,
**I want to** pay electricity, water and mobile top-up bills through one app regardless of how each biller works,
**So that** I can settle all my bills from my wallet without dealing with each company separately.

## Acceptance Criteria

>Scenario 1: ElectroCo REST/JSON Integration

- **Given:** ElectroCo is configured as an active biller and its integration is available.
- **When:** PayLink sends an electricity bill inquiry or payment request to ElectroCo.
- **Then:** The system must communicate using REST API with JSON format and process the response according to the ElectroCo contract.

>Scenario 2: AquaCo SOAP-style XML Integration

- **Given:** AquaCo is configured as an active biller and available.
- **When:** PayLink sends a water bill inquiry or payment request to AquaCo.
- **Then:** The system must communicate using SOAP-style XML over HTTP and process AquaCo's response, including its specific field names and error codes.

>Scenario 3: TopUpNet REST/JSON Integration

- **Given:** TopUpNet is configured as an active biller and available.
- **When:** PayLink sends a mobile top-up request to TopUpNet.
- **Then:** The system must send the request using REST API with JSON format.

>Scenario 4: Handle TopUpNet Asynchronous Response

- **Given:** PayLink has sent a mobile top-up request to TopUpNet.
- **When:** TopUpNet initially responds with `accepted` and later sends an asynchronous callback containing either `success` or `failed`.
- **Then:** PayLink must receive and process the asynchronous callback. The payment must remain in `SentToBiller` after receiving `accepted` until the final callback arrives. If the callback contains `success`, the payment must transition to `Completed`. If the callback contains `failed`, the payment must transition to `Failed`.

>Scenario 5: AquaCo Rejects a Payment Above the Bill Amount

- **Given:** PayLink has sent a water bill payment request with an amount greater than the bill amount.
- **When:** AquaCo refuses the payment.
- **Then:** The payment must transition to `Failed`. The held funds must be released.
The payment must not be marked as `Completed`.

>Scenario 6: Handle Live Biller Timeout

- **Given:** PayLink has sent a payment request to a live biller.
- **When:** The biller does not respond within **10 seconds**.
- **Then:** The payment must transition to `Unknown`.
The held funds must remain held. The system must not automatically refund or release the funds while the payment result is unknown.

>Scenario 7: Support All Three Mandatory Billers

- **Given:** ElectroCo, AquaCo, and TopUpNet are configured as supported billers.
- **When:** Customers initiate transactions through their respective services.
- **Then:** PayLink must support all three billers using their specified communication protocols and formats.

>Scenario 8: Optional NetCo Batch Integration

- **Given:** NetCo batch integration has been implemented and enabled.
- **When:** PayLink processes internet bill payments through NetCo.
- **Then:** PayLink may upload payments using a daily CSV batch file and process the result file received the following morning.

>Scenario 9: Add a New Biller Without Modifying Existing Modules

- **Given:** A new biller must be added to PayLink.
- **When:** The new biller is integrated into the system.
- **Then:**Only a new adapter and its configuration must be added. Existing modules must not require modification.

>Scenario 10: NetCo Is Optional

- **Given:** NetCo batch integration has not been implemented.
- **When:** The system is evaluated against FR-15.
- **Then:** FR-15 must still be considered satisfied, provided that ElectroCo, AquaCo, and TopUpNet are supported.

## User Story 16
**Priority:** Must

**As** a customer,
**I want to** have service fees calculated automatically according to the applicable business rules,
**So that** I know the exact amount I will pay before confirming a transaction.

## Acceptance Criteria

>Scenario 1: Calculate Electricity and Water Fees

- **Given:** The customer is paying an electricity or water bill.
- **When:** The system calculates the service fee.
- **Then:** The system must apply a fixed service fee of 2.50 EGP.

>Scenario 2: Calculate Mobile Top-Up Fees

- **Given:** The customer is making a mobile top-up.
- **When:** The system calculates the service fee.
- **Then:** The system must calculate 1.5% of the top-up amount.
The minimum service fee must be 1 EGP. If the calculated fee is less than 1 EGP, the system must charge 1 EGP.

>Scenario 3: Calculate Internet Fees

- **Given:** The customer is paying an internet bill.
- **When:** The system calculates the service fee.
- **Then:** The system must charge 2 EGP for bill amounts up to and including 200 EGP.
- The system must charge 4 EGP for bill amounts above 200 EGP.

>Scenario 4: Apply Promotion Period

- **Given:** A promotion is configured for a specific biller with a start date and an end date.
- **When:** The customer makes a payment to that biller during the promotion period.
- **Then:** The system must apply a service fee of 0 EGP.

>Scenario 5: Apply Normal Fees After Promotion Expiration

- **Given:** The promotion period for a biller has expired.
- **When:** The customer makes a payment to that biller.
- **Then:** The system must calculate the normal service fee according to the applicable business rules.

>Scenario 6: Display the Total Payment Amount

- **Given:** The system has calculated the bill amount and service fee.
- **When:** The customer reviews the payment before confirmation.
- **Then:** The system must display the bill amount, service fee, and total amount to be deducted from the wallet.
- The total amount must equal the bill amount plus the service fee.

>Scenario 7: Use Exact Decimal Calculations

- **Given:** The system performs monetary calculations.
- **When:** The system calculates payment amounts, service fees, or commissions.
- **Then:** The system must use exact decimal types for all monetary calculations.
- Floating-point types must never be used for monetary calculations.

>Scenario 8: Apply the Round Half Up Method

- **Given:** A calculated service fee contains more than two decimal places.
- **When:** The system rounds the calculated service fee.
- **Then:** The system must apply the Round Half Up method to two decimal places.
- If the third decimal digit is 5 or greater, the second decimal digit must be rounded up.
- If the third decimal digit is less than 5, the second decimal digit must remain unchanged.

Examples:
- 2.344 EGP → 2.34 EGP
- 2.345 EGP → 2.35 EGP
- 2.346 EGP → 2.35 EGP
- 1.999 EGP → 2.00 EGP


## User Story 17
**Priority:** Should

**As** an agent,
**I want to** pay bills on behalf of customers,
**So that** I can provide payment services to customers and earn a commission for successful payments.

## Acceptance Criteria

>Scenario 1: Agent Makes a Payment on Behalf of a Customer

- **Given:** The agent is authenticated and authorized to perform payments.
- **When:** The agent initiates a payment using the customer's bill information.
- **Then:** The system must process the payment on behalf of the customer.
- The system must identify the transaction as an agent-assisted payment.

>Scenario 2: Calculate Agent Commission

- **Given:** An agent-assisted payment has been initiated.
- **When:** The system calculates the agent's commission.
- **Then:** The system must calculate the commission as 40% of the service fee.

>Scenario 3: Credit Commission After Successful Payment

- **Given:** An agent-assisted payment has been initiated.
- **When:** The payment reaches the `Completed` status.
- **Then:** The system must credit the agent's wallet with 40% of the service fee.

>Scenario 4: Do Not Credit Commission for Failed Payments

- **Given:** An agent-assisted payment has been initiated.
- **When:** The payment reaches the `Failed`, `Rejected`, or `Refunded` status without having completed successfully.
- **Then:** The system must not credit the agent's wallet with a commission.

>Scenario 5: Do Not Credit Commission Before Payment Completion

- **Given:** An agent-assisted payment is still being processed or has an `Unknown` status.
- **When:** The system handles the agent's commission.
- **Then:** The system must not credit the commission until the payment reaches the `Completed` status.

## User Story 18
**Priority:** Must

**As** a customer,
**I want to** have payment limits enforced according to my account verification level and the business rules,
**So that** I can make payments within the permitted limits and understand why a payment is rejected.

## Acceptance Criteria

>Scenario 1: Reject Payment Below the Minimum Single Payment Limit

- **Given:** The customer initiates a payment.
- **When:** The payment amount is less than 5 EGP.
- **Then:** The system must reject the payment.
- The system must display a clear message explaining that the minimum single payment amount is 5 EGP.

>Scenario 2: Reject Payment Above the Maximum Single Payment Limit

- **Given:** The customer initiates a payment.
- **When:** The payment amount exceeds 10,000 EGP.
- **Then:** The system must reject the payment.
- The system must display a clear message explaining that the maximum single payment amount is 10,000 EGP.

>Scenario 3: Allow Payments Within the Single Payment Limits

- **Given:** The customer initiates a payment.
- **When:** The payment amount is between 5 EGP and 10,000 EGP, inclusive.
- **Then:** The system must allow the payment to proceed to the next validation and processing steps.

>Scenario 4: Enforce the Daily Payment Limit for Basic Accounts

- **Given:** The customer has a Basic account with a daily payment limit of 5,000 EGP.
- **When:** The customer attempts a payment that would cause the total daily payments to exceed 5,000 EGP.
- **Then:** The system must reject the payment.
- The system must display a clear message explaining that the daily payment limit for Basic accounts has been exceeded.

>Scenario 5: Enforce the Daily Payment Limit for Verified Accounts

- **Given:** The customer has a Verified account with a daily payment limit of 30,000 EGP.
- **When:** The customer attempts a payment that would cause the total daily payments to exceed 30,000 EGP.
- **Then:** The system must reject the payment.
- The system must display a clear message explaining that the daily payment limit for Verified accounts has been exceeded.

>Scenario 6: Allow Payments Within the Daily Payment Limit

- **Given:** The customer has not exceeded the daily payment limit for their account type.
- **When:** The customer initiates a payment that keeps the total daily payments within the permitted limit.
- **Then:** The system must allow the payment to proceed to the next validation and processing steps.

## User Story 19
**Priority:** Should

**As** a system administrator,
**I want to** have accounts temporarily blocked when they perform more than five payment attempts within 60 seconds,
**So that** the system can reduce suspicious or excessive payment activity.

## Acceptance Criteria

>Scenario 1: Detect Excessive Payment Attempts

- **Given:** A customer performs payment attempts from the same account.
- **When:** The number of payment attempts exceeds 5 within 60 seconds.
- **Then:** The system must detect the excessive payment activity.

>Scenario 2: Temporarily Block the Account

- **Given:** The system detects more than 5 payment attempts within 60 seconds from the same account.
- **When:** The velocity rule is triggered.
- **Then:** The system must block the account for 10 minutes.
- The system must prevent the account from initiating additional payments during the blocking period.

>Scenario 3: Automatically Unblock the Account

- **Given:** The account has been temporarily blocked due to excessive payment attempts.
- **When:** The 10-minute blocking period expires.
- **Then:** The system must automatically remove the temporary block.
- The customer must be able to initiate payments again, provided no other restrictions apply.

>Scenario 4: Do Not Block Accounts Within the Allowed Threshold

- **Given:** A customer performs 5 or fewer payment attempts within 60 seconds.
- **When:** The system checks the velocity rule.
- **Then:** The system must not block the account based on this rule.

## User Story 20
**Priority:** Must

**As** a support employee,
**I want to** search for a payment using its unique payment reference and view its complete history,
**So that** I can investigate customer complaints and track payment issues.

## Acceptance Criteria

>Scenario 1: Search for a Payment Using Its Reference

- **Given:** The support employee is authenticated and authorized.
- **When:** The employee searches using a valid payment reference.
- **Then:** The system must retrieve the corresponding payment.

>Scenario 2: Display Payment Details

- **Given:** The support employee has searched for an existing payment.
- **When:** The system retrieves the payment.
- **Then:** The system must display the payment details, including:
Payment reference, Customer information, Biller and bill type, Payment amount, Service fee.
Total amount, Payment status, Payment creation date and time.

>Scenario 3: Display Complete Payment History

- **Given:** The support employee has opened a payment record.
- **When:** The employee views its history.
- **Then:** The system must display all recorded payment status changes and relevant processing events in chronological order.
- The history must include the timestamps of those events.

>Scenario 4: Handle Payment Reference Not Found

- **Given:** The support employee searches using a payment reference that does not exist.
- **When:** The system processes the search.
- **Then:** The system must display a clear message indicating that no payment was found.

>Scenario 5: Display Failed Payment Information

- **Given:** The support employee searches for a payment with a `Failed` status.
- **When:** The system displays the payment details.
- **Then:** The system must display the failure reason when it is available.

## User Story 21
**Priority:** Should

**As** a support employee,
**I want to** block or unblock customer accounts,
**So that** I can restrict suspicious or problematic accounts and restore access when appropriate.

## Acceptance Criteria

>Scenario 1: Block a Customer Account

- **Given:** The support employee is authenticated and authorized, and the customer account exists and is currently active.
- **When:** The employee chooses to block the account.
- **Then:** The system must change the account status to `Blocked`.
- The system must prevent the customer from performing restricted account operations, including initiating payments.

>Scenario 2: Unblock a Customer Account

- **Given:** The support employee is authenticated and authorized, and the customer account is currently `Blocked`.
- **When:** The employee chooses to unblock the account.
- **Then:** The system must change the account status to `Active`.
- The customer must be able to use the account again, provided no other restrictions apply.

>Scenario 3: Prevent Blocking an Already Blocked Account

- **Given:** The customer account is already `Blocked`.
- **When:** The support employee attempts to block it again.
- **Then:** The system must inform the employee that the account is already blocked.

>Scenario 4: Prevent Unblocking an Already Active Account

- **Given:** The customer account is already `Active`.
- **When:** The support employee attempts to unblock it.
- **Then:** The system must inform the employee that the account is already active.

>Scenario 5: Record Account Status Changes

- **Given:** A support employee blocks or unblocks a customer account.
- **When:** The system successfully changes the account status.
- **Then:** The system must record the action.
- The record must include the employee who performed the action and the date and time of the change.






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

**## Questions or ambiguities**

**Q10. What customer information is required for each biller, and what should happen if the bill is already paid or expired? US-8**\\
Decision: Use each biller's required fields and reject already-paid or expired bills.

**Q11. Is there a timeout limit for bill inquiries, and should failed inquiries be retried automatically or only when the customer requests another inquiry? US-8**\\
Decision: Use the 10-second timeout. The customer retries failed inquiries manually.

**Q12. Should the wallet hold cover the bill amount plus the transaction fee, and what happens if the available balance changes before the hold is created? US-9, US-12**\\
Decision: Hold the full amount, including the fee, after checking the available balance again.

**Q13. What is the scope and retention period of an idempotency key, and can it be reused if the original request fails before a payment record is created? US-10**\\
Decision: Keys are unique per customer and endpoint. They can be reused if no payment was created.

**Q14. What exact error should be returned for a refused state transition, and should repeated transitions to the same state or manual support transitions be allowed? US-11**\\
Decision: Return an invalid-transition error and ignore duplicate transitions.

**Q15. Is the Failed → Refunded transition automatic, and what happens if the release ledger entry fails? Should holds and releases also have audit records under FR-25? US-11, US-12**\\
Decision: Refund after a successful hold release, and log all holds and releases.

**Q16. Does the 10-second payment timeout apply to all billers, or only AquaCo and ElectroCo, and are scheduled re-queries preserved if the system restarts? US-13**\\
Decision: Apply it to live billers only and save scheduled re-queries.

**Q17. Can a customer initiate another payment for the same bill while the original payment is still in Unknown? US-13**\\
Decision: No, the customer must wait until the original payment is resolved.

**Q18. What should happen if a callback arrives before the payment reaches SentToBiller, or if a failed callback arrives while the payment is Unknown? US-14**\\
Decision: Ignore and log early callbacks. A failed callback in Unknown moves the payment to Failed, then Refunded.

 **it does not specify a rounding method as half up or half even, I made it half up?
 **it does not mention what should happen if the calculated fee is less than 1, i made it that he should charge for more?

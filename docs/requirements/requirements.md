# Requirements

**Problem Summary**


**Stakeholders**


**User Stories**
1-
2-
3-
---
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


**Review Status:** Ready for team / TA review. Resolve the listed ambiguities before treating the requirements as finalized.



**Non-functional Requirements**


**Priorities**


**Questions or ambiguities** 
 it does not specify a rounding method as half up or half even, I made it half up
 it does not mention what should happen if the calculated fee is less than 1, i made it that he should charge for more
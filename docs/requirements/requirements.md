**Problem Summary**


**Stakeholders**


**User Stories**

## US-22

**As** a customer,
**I want to** receive an SMS and a push notification when my payment completes or fails,
**so that** I know the result without opening the app.

> Scenario 1: Payment completed
**Given** a payment is in the SentToBiller state
**When** the biller confirms and the payment becomes Completed
**Then** the customer receives an SMS and a push notification containing the amount and the payment reference.

> Scenario 2: Payment failed after the biller refuses
**Given** a payment is in the SentToBiller state
**When** the biller refuses and the payment becomes Failed
**Then** the customer receives an SMS and a push notification that states the reason for the failure.

> Scenario 3: Payment failed after being Unknown
**Given** a payment is in the Unknown state
**When** the biller later refuses and the payment becomes Failed
**Then** the customer receives the same failure notification as in Scenario 2.

> Scenario 4: SMS provider is down
**Given** the SMS provider is down
**When** a payment becomes Completed
**Then** the payment stays Completed and is not undone..


## US-23

**As** a support employee,
**I want** failed notifications to be retried automatically and then marked as undelivered if all retries fail,
**so that** customers have the best chance to get their message and and support can identify notifications that were not delivered.

> Scenario 1: Retry after a failed send
**Given** a notification (SMS or email) fails on its first send
**When** the system detects the failure
**Then** the system retries sending it, up to 3 retries after the first attempt.

> Scenario 2: A retry succeeds
**Given** a notification has failed on its first send
**When** one of the retries is delivered successfully
**Then** the notification is marked as delivered and no more retries are made.

> Scenario 3: All retries fail
**Given** a notification has failed on the first send and on all 3 retries (4 attempts in total)
**When** the last retry fails
**Then** the notification is marked as Undelivered and is visible to support.

> Scenario 4: The payment is not affected
**Given** a payment has become Completed and its notification is failing
**When** the notification is retried or marked Undelivered
**Then** the payment stays Completed its state is not affected by the notification failure.


## US-24

**As** an admin,  
**I want** to edit fee and limit values without redeploying the system,  
**so that** I can update payment rules and configurations easily without interrupting the service.

>  Scenario 1: Update Fee Values**
**Given** the admin has access to fee configuration  
**When** the admin edits and saves a fee value  
**Then** the system saves and applies the updated fee without requiring redeployment.

>  Scenario 2: Update Limit Values**
**Given** the admin has access to payment limit configuration  
**When** the admin edits and saves a payment limit  
**Then** the system saves and applies the updated limit without requiring redeployment.

>  Scenario 3: Apply Updated Configuration**
**Given** the admin has successfully updated fee or limit values  
**When** a new payment is initiated  
**Then** the system uses the updated configuration when calculating fees and checking payment limits.


## US-25

**As** an admin,
**I want** every money movement and every admin action to be written to an audit log,
**so that** any change to money or settings can be traced to who did it, when, and why.

> Scenario 1: Money movement is recorded
**Given** a money movement happens (top-up, hold, charge, hold release, refund, fee income or agent commission)
**When** the movement is processed
**Then** an audit record is written with who triggered it, when it happened, and why (the reason or payment reference).

> Scenario 2: Admin action is recorded
**Given** an admin adds, edits or disables a biller, a service, a fee value or a limit value
**When** the change is saved
**Then** an audit record is written with the admin's identity, the time, what changed, and the reason.

> Scenario 3: Failed payment is also recorded
**Given** a payment fails after its funds were held
**When** the hold is released to the wallet
**Then** an audit record exists for the hold and another for the release.

> Scenario 4: Audit records cannot be changed
**Given** an audit record has been written
**When** anyone tries to edit or delete it
**Then** the change is refused and the original record stays the same.


## US-26

**As** a support employee,
**I want** the system to compare each day's settlement file with PayLink's payment records and produce a difference report,
**so that** I can review every payment where PayLink and the biller disagree.

> Scenario 1: Matching payments
**Given** a payment is Completed in PayLink and the settlement file lists the same payment reference with the same amount
**When** the nightly reconciliation runs
**Then** the report shows the payment as matched and does not flag it.

> Scenario 2: Settlement file lists a payment PayLink has as Failed
**Given** a payment is Failed in PayLink and the settlement file lists it as received
**When** the nightly reconciliation runs
**Then** the report flags the payment as a difference for support to review, and the payment stays Failed.

> Scenario 3: Unknown payment confirmed by the settlement file
**Given** a payment is Unknown and the settlement file lists it as received
**When** the nightly reconciliation runs
**Then** the payment becomes Completed.

> Scenario 4: Unknown payment not found in the settlement file
**Given** a payment is Unknown and the settlement file does not list it
**When** the nightly reconciliation runs
**Then** the payment becomes Failed.

> Scenario 5: Completed payment missing from the settlement file
**Given** a payment is Completed in PayLink and the settlement file does not list it
**When** the nightly reconciliation runs
**Then** the report flags the payment as a difference for support to review.

> Scenario 6: Settlement entry unknown to PayLink
**Given** the settlement file lists a payment reference that PayLink has no record of
**When** the nightly reconciliation runs
**Then** the report flags that entry as a difference for support to review.

## US-27

**As** a customer,
**I want** to request a refund for a Completed payment and have support approve or reject it,
**so that** I can get my money back when I paid by mistake.

> Scenario 1: Customer requests a refund
**Given** a payment belongs to the customer and its state is Completed
**When** the customer submits a refund request for it
**Then** a refund request is created with the status Pending, linked to that payment, and it appears for support to review. The payment stays Completed.

> Scenario 2: Support approves the refund
**Given** a refund request is Pending
**When** a support employee approves it
**Then** the payment amount is credited back to the customer's wallet through a ledger entry, the request becomes Approved, and an audit record shows who approved it, when, and why.

> Scenario 3: Support rejects the refund
**Given** a refund request is Pending
**When** a support employee rejects it with a reason
**Then** the request becomes Rejected, the wallet balance does not change, and the customer can see the reason.

> Scenario 4: Payment is not Completed
**Given** a payment belongs to the customer and its state is not Completed
**When** the customer submits a refund request for it
**Then** the request is refused with a message that only Completed payments can be refunded, and no refund request is created.

> Scenario 5: Payment of another customer
**Given** a payment belongs to a different customer
**When** the customer submits a refund request for it
**Then** the request is refused and no refund request is created.

> Scenario 6: Duplicate request
**Given** a refund request for a payment is already Pending or Approved
**When** the customer submits another request for the same payment
**Then** the second request is refused, so the payment can never be refunded twice.


## US-28

**As** an admin,
**I want** to view daily totals for payments, fees earned, commissions paid and failed payments per biller,
**so that** I can monitor PayLink's daily results and see which billers have problems.

> Scenario 1: Payment totals per biller
**Given** ElectroCo has two Completed payments on the selected day, of 350.00 EGP and 150.00 EGP
**When** the admin views the daily totals
**Then** the report shows for ElectroCo: 2 payments and a total of 500.00 EGP.

> Scenario 2: Fees earned and commissions paid
**Given** both of those payments have a fee of 2.50 EGP, and one of them was paid by an agent
**When** the admin views the daily totals
**Then** the report shows for ElectroCo: fees earned 5.00 EGP and commissions paid 1.00 EGP (40% of 2.50 EGP).

> Scenario 3: Failures per biller
**Given** AquaCo has one Failed payment on the selected day
**When** the admin views the daily totals
**Then** the report shows for AquaCo: 1 failed payment.

> Scenario 4: Only Completed payments count as money
**Given** a payment on the selected day is in the state Rejected, Unknown or Failed
**When** the admin views the daily totals
**Then** its amount, fee and commission are not included in the payment, fee and commission totals.

> Scenario 5: A day with no activity
**Given** no payments exist for the selected day
**When** the admin views the daily totals
**Then** every biller shows 0 payments, 0.00 EGP, 0.00 EGP fees, 0.00 EGP commissions and 0 failures.

> Scenario 6: Only admins can view it
**Given** a user who is not an admin
**When** the user tries to view the daily totals
**Then** access is refused and no totals are shown.


**Non-functional Requirements**


### Priorities

| Priority | Requirements |
|---|---|
| **Must** | FR-1, FR-3, FR-4, FR-5, FR-6, FR-7, FR-8, FR-9, FR-10, FR-11, FR-12, FR-13, FR-14, FR-15, FR-16, FR-18, FR-20, FR-22, FR-25 |
| **Should** | FR-2, FR-17, FR-19, FR-21, FR-23, FR-24, FR-26 |
| **Could** | FR-27, FR-28 |


**Questions or ambiguities**

Q: Does "failure" in FR-22 include Rejected payments? 
Decision: Yes, the customer is shown the reason in the app, but SMS is sent only for Failed.

Q:Does "retried 3 times" mean 3 retries after the first attempt, or 3 attempts in total? 
Decision: 3 retries after the first attempt (4 attempts in total).

Q:Is there a waiting time between retries? 
The brief doesn't say.
Decision: (agree with the team, for example immediate retries).

Q:Are updated fees and limits applied immediately to new payments, or from a specified time?
Why? FR-24 says without redeploying, but does not define when changes become effective.

Q:What happens if the admin enters an invalid value?
Should the system reject the update and keep the previous valid configuration?
Why? The requirement does not specify validation rules or error handling.

Q:Should every fee and limit update be recorded in the audit log, including the admin, old value, new value, and update time?
Why? FR-25 requires every admin action to be written to an audit log.

Q:Should only authorized admins and support employees be allowed to view audit records?

Q:Should the system keep audit records permanently or for a defined retention period?

Q:Does "audit log" mean the same thing as the ledger (FR-5)?
 Decision: No. The ledger records money amounts, and the audit log records who, when and why. Every ledger entry also has an audit record.

Q:Does a "difference" include a wrong amount (same reference, different amount)?
The brief doesn't say. Decision: Yes, it is flagged in the same way. Confirm with the team.

Q:Does the reconciliation fix a Failed payment automatically if the biller did receive it? 
Decision: No. It is only flagged for support, because the brief says "for support to review".

Q:Is the service fee refunded too? The brief doesn't say.
Decision: the full amount paid (bill + fee) is credited back. Confirm with the team.

Q:Does the "payments" total include Rejected and Unknown payments? 
Decision: No. Only Completed payments count as money. Failed payments are counted separately in the failures column.

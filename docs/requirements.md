# Requirements

### Problem Summary

PayLink is a company in Egypt, similar to Fawry or Vodafone Cash, that lets
people pay monthly bills (electricity, water, mobile and internet) from a
digital wallet, through a mobile app or through agents who work in small
shops. Customers want to avoid queues, and the billers (the utility
companies) want to be paid quickly and correctly.

PayLink holds customers' money in wallets, so mistakes are serious. A
customer charged twice, or charged for a payment the biller never received,
costs the company money and trust, and may lead to problems with regulators.
Every step of a payment can fail, for example when a biller is slow, refuses
the payment, or never answers, so most of the system's work is deciding what
to do in those cases.

PayLink earns money from a small service fee on each payment and pays
commission to agents. We are building the backend of a new PayLink system
in .NET. It must be correct with money, easy to extend with new billers,
and able to handle the busiest day of the month. In Phase 1 we plan the
system: requirements, design diagrams, and the Docker and CI foundation.

### Stakeholders

| Stakeholder | Who they are | What they care about |
|---|---|---|
| Customer | A person who pays bills from their wallet | Paying fast, seeing the fee before paying, never being charged twice, getting a clear message when something fails |
| Agent | A shop owner with a PayLink terminal who pays on behalf of customers | Payments completing correctly and earning commission (40% of the service fee) |
| Biller | A utility company (ElectroCo, AquaCo, TopUpNet, NetCo) | Receiving every payment correctly and on time, with matching records |
| Admin | PayLink staff who manage the system | Adding and disabling billers, changing fees and limits without redeploying, seeing daily totals |
| Support employee | PayLink staff who help customers | Finding a payment by its reference, seeing its full history, blocking accounts, reviewing differences and refunds |
| PayLink management | The company | Correct money handling, low risk, and a system that is easy to extend |
| Regulators | Authorities that watch companies holding customers' money | Safe, traceable money, supported by the audit log |
| Development team and TAs | Us, and the people who grade the project | A clear, testable and well-documented system |

### User Stories

### US-1: Customer registration with a basic verification level (FR-1, Must)
**As** a new customer,
**I want to** register for a PayLink account,
**so that** I get a wallet and can start paying bills within the BASIC limits.

> Scenario 1: **Successful registration**\
**Given** a person provides a valid unregistered name, phone number and password\
**When** they submit the registration request\
**Then** a customer account is created with verification level BASIC\
**And** an empty EGP wallet is created for the new user\
**And** the daily payment limit of 5,000 EGP applies to the account

> Scenario 2: **Phone number already registered**\
**Given** a customer account already exists with phone number 01#########\
**When** someone tries to register again with the same phone number\
**Then** the registration is rejected with a "phone number already registered" error\
**And** no new account or wallet is created

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
**Given** a registered customer with verification level BASIC requests a level upgrade\
**When** the upgrade request is approved by an admin\
**Then** the customer's verification level becomes VERIFIED\
**And** their daily payment limit changes from 5,000 EGP to 30,000 EGP\
**And** an audit log entry records who approved, when and why (FR-25)

> Scenario 2: **Already VERIFIED**\
**Given** a customer whose level is already VERIFIED\
**When** an upgrade is requested again\
**Then** the request is ignored with an "already verified" message\
**And** the account is unchanged

> Scenario 3: **Upgrade not approved**\
**Given** a registered customer with verification level BASIC requests a level upgrade\
**When** the upgrade request is rejected by an admin\
**Then** the level stays BASIC and the 5,000 EGP limit remains unchanged\
**And** the customer is sent a message with the reason behind the rejection\
**And** an audit log entry records who rejected, when and why (FR-25)


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
**Then** the 352.50 EGP appears in the held balance, not in the available balance

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
**Given** a customer tops up with a simulated card that is declined\
**When** they submit the top-up\
**Then** it is rejected with a clear reason\
**And** no ledger entry is created and the balance is unchanged

> Scenario 4: **Invalid amount**\
**Given** a customer tops up below the configured minimum (10 EGP) or with an amount that has more than two decimal places\
**When** they submit the top-up\
**Then** it is rejected with a validation error naming the minimum\
**And** no ledger entry is created and the balance is unchanged

> Scenario 5: **Daily top-up limit exceeded**\
**Given** a customer whose top-ups today plus the new amount would exceed the configured daily top-up limit (30,000 EGP)\
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
**Then** a new ledger entry is created for it\
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
**When** the admin adds a service with its name, required input fields, minimum and maximum amounts\
**Then** the service is saved under that biller\
**And** it is visible to customers in the catalog (FR-7)

> Scenario 3: **Edit a biller or service**\
**Given** an existing biller or service\
**When** the admin changes its details, such as the required fields or the amount limits\
**Then** the changes are saved and apply to new payments from that point\
**And** payments already in progress are not affected\
**And** an audit log entry records the old and new values

> Scenario 4: **Disable a biller or service**\
**Given** an active biller or service\
**When** the admin disables it\
**Then** it no longer appears to customers and new payments to it are rejected with a "service unavailable" message\
**And** payments already in progress continue through their life cycle\
**And** its history is kept, not deleted


### US-7: Browse billers and required input fields (FR-7, Must)
**As** a customer,
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

> Scenario 3: **Catalog changes are visible without redeploying**\
**Given** an admin has just disabled a biller, added a biller or edited a service's fields\
**When** a customer requests the list again\
**Then** the response reflects the change


### US-8: Perform a bill inquiry (FR-8, Must)
**As** a customer,
**I want to** perform a bill inquiry,
**so that** I can see the bill amount, service fee and total amount before making a payment.

> Scenario 1: **Successful electricity bill inquiry**\
**Given** a customer enters a valid electricity meter number and the bill amount is 350.00 EGP\
**When** the customer performs a bill inquiry\
**Then** the system displays a bill amount of 350.00 EGP\
**And** it displays a service fee of 2.50 EGP\
**And** it displays a total amount of 352.50 EGP

> Scenario 2: **Successful water bill inquiry**\
**Given** a customer enters a valid water account number and the bill amount is 120.75 EGP\
**When** the customer performs a bill inquiry\
**Then** the system displays a bill amount of 120.75 EGP\
**And** it displays a service fee of 2.50 EGP\
**And** it displays a total amount of 123.25 EGP

> Scenario 3: **Invalid account information**\
**Given** a customer enters an invalid meter or account number\
**When** the customer performs a bill inquiry\
**Then** the system displays "Bill not found"\
**And** no payment total is displayed

> Scenario 4: **Biller unavailable**\
**Given** the biller does not respond to the inquiry request\
**When** the customer performs a bill inquiry\
**Then** no payment total is displayed\
**And** the system displays "Biller unavailable, try again later"


### US-9: Pay a bill from the wallet (FR-9, Must)
**As** a customer,
**I want to** pay bills using my wallet,
**so that** I can complete bill payments and my funds are handled correctly throughout the payment process.

> Scenario 1: **Funds held when the payment is confirmed**\
**Given** the available balance is 1,000.00 EGP and the held balance is 0.00 EGP\
**And** the bill total is 352.50 EGP\
**When** the customer confirms the payment\
**Then** the payment state becomes FundsHeld

> Scenario 2: **Biller confirms the payment**\
**Given** a payment is in SentToBiller, 352.50 EGP is held and the available balance is 647.50 EGP\
**When** the biller confirms the payment\
**Then** the payment state becomes Completed\
**And** the held balance becomes 0.00 EGP and the available balance stays 647.50 EGP\
**And** 350.00 EGP is paid to the biller and 2.50 EGP is credited to PayLink's fee income\
**And** ledger entries and audit records are written for the final charge and the fee income

> Scenario 3: **Insufficient funds**\
**Given** the available balance is 100.00 EGP and the bill total is 352.50 EGP\
**When** the customer attempts to pay\
**Then** the payment state becomes Rejected\
**And** the system displays "Insufficient funds"\
**And** both balances remain unchanged and the biller is not called

> Scenario 4: **Amount above the maximum (both account types)**\
**Given** the maximum single payment is 10,000.00 EGP\
**When** the customer attempts to pay a bill amount of 10,000.01 EGP\
**Then** the payment state becomes Rejected\
**And** the system displays "Amount above the 10,000.00 maximum"\
**And** both balances remain unchanged

> Scenario 5: **Amount below the minimum (both account types)**\
**Given** the minimum single payment is 5.00 EGP\
**When** the customer attempts to pay a bill amount of 4.99 EGP\
**Then** the payment state becomes Rejected\
**And** the system displays "Amount below the 5.00 minimum"\
**And** both balances remain unchanged

> Scenario 6: **Two concurrent payments exceed the available balance**\
**Given** a customer has 500.00 EGP available in their wallet\
**And** each payment requires 352.50 EGP including the fee\
**When** the customer initiates both payments at nearly the same time\
**Then** only one payment can successfully hold 352.50 EGP\
**And** the other payment is rejected due to insufficient available balance\
**And** the customer's available balance never becomes negative


### US-10: Idempotent payment requests (FR-10, Must)
**As** a customer,
**I want** repeated payment requests with the same idempotency key to return the original result,
**so that** I can safely retry requests without being charged multiple times.

> Scenario 1: **Duplicate request with identical parameters**\
**Given** a payment request with idempotency key K1 has already been processed successfully and its payment state is Completed\
**When** the customer submits the same payment request again using K1 with identical parameters\
**Then** the system returns the original payment result\
**And** no new payment is created and no funds are deducted again

> Scenario 2: **Duplicate request with different parameters**\
**Given** a payment request with idempotency key K1 has already been submitted with an amount of 352.50 EGP\
**When** the customer submits another request using K1 but with a different amount or bill reference\
**Then** the system rejects the request with an idempotency key conflict error\
**And** the original payment and the wallet balance are not modified

> Scenario 3: **Duplicate request while the payment is processing**\
**Given** a payment request with idempotency key K1 has already been accepted and is in FundsHeld, SentToBiller or Unknown state\
**When** the customer submits the same request again using K1 with identical parameters\
**Then** the system returns the existing payment reference and current payment status\
**And** no other payment is created and no additional hold is placed\
**And** an Unknown payment is not refunded automatically


### US-11: Payment state machine (FR-11, Must)
**As** the system,
**I want to** move a payment only through the transitions allowed in section 2.5 and refuse all others,
**so that** every payment is in exactly one valid state and its history can be trusted.

> Scenario 1: **Initiated moves to FundsHeld**\
**Given** a payment is in Initiated\
**When** the fraud checks pass and the wallet has enough funds\
**Then** the payment state becomes FundsHeld

> Scenario 2: **Initiated moves to Rejected**\
**Given** a payment is in Initiated\
**When** a fraud check fails or the wallet does not have enough funds\
**Then** the payment state becomes Rejected

> Scenario 3: **FundsHeld moves to SentToBiller**\
**Given** a payment is in FundsHeld\
**When** the request is sent to the biller\
**Then** the payment state becomes SentToBiller

> Scenario 4: **SentToBiller moves to Completed**\
**Given** a payment is in SentToBiller\
**When** the biller confirms the payment\
**Then** the payment state becomes Completed

> Scenario 5: **SentToBiller moves to Failed**\
**Given** a payment is in SentToBiller\
**When** the biller refuses the payment\
**Then** the payment state becomes Failed

> Scenario 6: **SentToBiller moves to Unknown**\
**Given** a payment is in SentToBiller\
**When** the biller does not respond within the timeout\
**Then** the payment state becomes Unknown

> Scenario 7: **Unknown moves to Completed**\
**Given** a payment is in Unknown\
**When** the biller later confirms the payment\
**Then** the payment state becomes Completed

> Scenario 8: **Unknown moves to Failed**\
**Given** a payment is in Unknown\
**When** the biller later refuses the payment or reconciliation finds no payment\
**Then** the payment state becomes Failed

> Scenario 9: **Failed moves to Refunded**\
**Given** a payment is in Failed\
**When** the wallet hold is released\
**Then** the payment state becomes Refunded

> Scenario 10: **No transitions from final states**\
**Given** a payment is in Completed, Rejected or Refunded\
**When** the system attempts any further state transition\
**Then** the transition is rejected\
**And** the payment remains in its current state


### US-12: Wallet hold and release (FR-12, Must)
**As** a customer,
**I want** my money to be held before the biller is called and released if the payment fails,
**so that** I cannot spend the same money twice and I never lose money for a payment that did not happen.

> Scenario 1: **Funds are held before the biller is called**\
**Given** the available balance is 1,000.00 EGP and the held balance is 0.00 EGP\
**When** the customer pays a total of 352.50 EGP\
**Then** the available balance becomes 647.50 EGP and the held balance becomes 352.50 EGP\
**And** a ledger entry records the hold of 352.50 EGP\
**And** the request is sent to the biller only after this hold exists

> Scenario 2: **Hold is released when the biller refuses**\
**Given** a payment is in SentToBiller and 352.50 EGP is held\
**When** the biller refuses the payment\
**Then** the payment state becomes Failed\
**And** the hold is released, so the available balance returns to 1,000.00 EGP and the held balance becomes 0.00 EGP\
**And** the payment state becomes Refunded\
**And** a ledger entry records the release of 352.50 EGP

> Scenario 3: **Hold is kept while the payment is Unknown**\
**Given** a payment is in Unknown and 352.50 EGP is held\
**When** no confirmation or refusal has arrived from the biller\
**Then** the held balance stays 352.50 EGP\
**And** no release ledger entry is created and no refund is made


### US-13: Timeout and re-query (FR-13, Must)
**As** a customer,
**I want** a payment that times out to be kept as Unknown and re-queried automatically,
**so that** I am not charged twice or refunded by mistake while the biller's answer is still unknown.

> Scenario 1: **Biller times out**\
**Given** a payment is in SentToBiller and 352.50 EGP is held\
**When** the biller does not answer within 10 seconds (for example ElectroCo answers after 12 seconds)\
**Then** the payment state becomes Unknown\
**And** the customer sees "Your payment is being processed"\
**And** the held balance stays 352.50 EGP and no refund is made

> Scenario 2: **Re-query schedule**\
**Given** a payment became Unknown at 10:00\
**When** the time reaches the scheduled re-queries\
**Then** the system re-queries the biller after 1, 5 and 15 minutes (up to 3 times)\
**And** each re-query uses the original payment reference and does not create a new payment or a new hold

> Scenario 3: **Re-query shows the biller confirmed**\
**Given** a payment is in Unknown\
**When** a re-query shows that the biller confirmed the payment\
**Then** the payment state becomes Completed\
**And** the held balance becomes 0.00 EGP and no further re-queries are made

> Scenario 4: **Re-query shows the biller refused**\
**Given** a payment is in Unknown and 352.50 EGP is held\
**When** a re-query shows that the biller refused the payment\
**Then** the payment state becomes Failed and then Refunded\
**And** the held 352.50 EGP is released back to the available balance

> Scenario 5: **All re-queries get no answer**\
**Given** a payment is in Unknown\
**When** the re-queries after 1, 5 and 15 minutes all get no answer\
**Then** the payment state stays Unknown and no more re-queries are made\
**And** the held balance stays 352.50 EGP and the payment waits for reconciliation\
**And** no automatic refund is made


### US-14: TopUpNet callback handling (FR-14, Must)
**As** the system,
**I want to** accept TopUpNet's asynchronous callbacks and ignore duplicates,
**so that** a mobile top-up reaches its final state exactly once and a customer is never charged or credited twice.

> Scenario 1: **Success callback completes the payment**\
**Given** TopUpNet replied "accepted" and the payment is in SentToBiller with 352.50 EGP held\
**When** TopUpNet calls the callback URL with "success" and the payment reference\
**Then** the payment state becomes Completed\
**And** the held balance becomes 0.00 EGP and one final charge ledger entry is created

> Scenario 2: **Failed callback fails and refunds the payment**\
**Given** TopUpNet replied "accepted" and the payment is in SentToBiller with 352.50 EGP held\
**When** TopUpNet calls the callback URL with "failed" and the payment reference\
**Then** the payment state becomes Failed and then Refunded\
**And** the held 352.50 EGP is released back to the available balance

> Scenario 3: **Duplicate success callback is ignored**\
**Given** a payment is already Completed by a "success" callback\
**When** TopUpNet sends a second "success" callback with the same payment reference\
**Then** the payment state stays Completed\
**And** no new ledger entry is created\
**And** the duplicate is logged with the payment reference and the time

> Scenario 4: **Conflicting callback is ignored**\
**Given** a payment is already Completed by a "success" callback\
**When** TopUpNet sends a "failed" callback with the same payment reference\
**Then** the payment state stays Completed and no balance changes\
**And** the conflict is logged

> Scenario 5: **Callback resolves an Unknown payment**\
**Given** a payment is in Unknown with 352.50 EGP held\
**When** TopUpNet calls the callback URL with "success"\
**Then** the payment state becomes Completed and the held balance becomes 0.00 EGP

> Scenario 6: **Callback with an unknown reference**\
**Given** no payment exists with the reference "P-999"\
**When** TopUpNet calls the callback URL with that reference\
**Then** no payment or balance changes\
**And** the system returns the error "Payment reference not found" and logs the callback


### US-15: Support billers with different formats (FR-15, Must)
**As** a customer,
**I want to** pay electricity, water and mobile top-up bills through one app regardless of how each biller works,
**so that** I can settle all my bills from my wallet without dealing with each company separately.

> Scenario 1: **ElectroCo REST/JSON integration**\
**Given** ElectroCo is configured as an active biller and its integration is available\
**When** PayLink sends an electricity bill inquiry or payment request to ElectroCo\
**Then** the system communicates using a REST API with JSON format\
**And** it processes the response according to the ElectroCo contract

> Scenario 2: **AquaCo SOAP-style XML integration**\
**Given** AquaCo is configured as an active biller and available\
**When** PayLink sends a water bill inquiry or payment request to AquaCo\
**Then** the system communicates using SOAP-style XML over HTTP\
**And** it processes AquaCo's response, including its specific field names and error codes

> Scenario 3: **TopUpNet REST/JSON integration**\
**Given** TopUpNet is configured as an active biller and available\
**When** PayLink sends a mobile top-up request to TopUpNet\
**Then** the system sends the request using a REST API with JSON format

> Scenario 4: **Handle the TopUpNet asynchronous response**\
**Given** PayLink has sent a mobile top-up request to TopUpNet\
**When** TopUpNet first responds with "accepted" and later sends a callback containing "success" or "failed"\
**Then** the payment stays in SentToBiller after "accepted" until the final callback arrives\
**And** if the callback contains "success", the payment becomes Completed\
**And** if the callback contains "failed", the payment becomes Failed

> Scenario 5: **AquaCo rejects a payment above the bill amount**\
**Given** PayLink has sent a water bill payment request with an amount greater than the bill amount\
**When** AquaCo refuses the payment\
**Then** the payment state becomes Failed\
**And** the held funds are released\
**And** the payment is not marked as Completed

> Scenario 6: **Live biller timeout**\
**Given** PayLink has sent a payment request to a live biller\
**When** the biller does not respond within 10 seconds\
**Then** the payment state becomes Unknown\
**And** the held funds remain held and are not automatically released or refunded

> Scenario 7: **All three mandatory billers are supported**\
**Given** ElectroCo, AquaCo and TopUpNet are configured as supported billers\
**When** customers initiate payments through their respective services\
**Then** PayLink supports all three billers using their specified protocols and formats

> Scenario 8: **Optional NetCo batch integration**\
**Given** NetCo batch integration has been implemented and enabled\
**When** PayLink processes internet bill payments through NetCo\
**Then** PayLink uploads the payments in a daily CSV batch file\
**And** it processes the result file received the following morning

> Scenario 9: **Add a new biller without modifying existing modules**\
**Given** a new biller must be added to PayLink\
**When** the new biller is integrated into the system\
**Then** only a new adapter and its configuration are added\
**And** existing modules do not require modification

> Scenario 10: **NetCo is optional**\
**Given** NetCo batch integration has not been implemented\
**When** the system is evaluated against FR-15\
**Then** FR-15 is still considered satisfied, provided that ElectroCo, AquaCo and TopUpNet are supported


### US-16: Fee calculation (FR-16, Must)
**As** a customer,
**I want** service fees to be calculated automatically according to the business rules,
**so that** I know the exact amount I will pay before confirming a payment.

> Scenario 1: **Electricity and water fees**\
**Given** the customer is paying an electricity or water bill\
**When** the system calculates the service fee\
**Then** the service fee is a fixed 2.50 EGP

> Scenario 2: **Mobile top-up fee**\
**Given** the customer is making a mobile top-up\
**When** the system calculates the service fee\
**Then** the fee is 1.5% of the top-up amount\
**And** if the calculated fee is less than 1 EGP, the fee is 1.00 EGP

> Scenario 3: **Internet fee**\
**Given** the customer is paying an internet bill\
**When** the system calculates the service fee\
**Then** the fee is 2.00 EGP for bill amounts up to and including 200 EGP\
**And** the fee is 4.00 EGP for bill amounts above 200 EGP

> Scenario 4: **Promotion period**\
**Given** a promotion is configured for one biller with a start date and an end date\
**When** the customer makes a payment to that biller during the promotion period\
**Then** the service fee is 0.00 EGP

> Scenario 5: **Normal fee after the promotion ends**\
**Given** the promotion period for a biller has expired\
**When** the customer makes a payment to that biller\
**Then** the service fee is calculated by the normal rules for that biller

> Scenario 6: **Total payment amount is displayed**\
**Given** the system has calculated the bill amount and the service fee\
**When** the customer reviews the payment before confirmation\
**Then** the system displays the bill amount, the service fee and the total amount to be deducted from the wallet\
**And** the total equals the bill amount plus the service fee

> Scenario 7: **Exact decimal calculations**\
**Given** the system performs monetary calculations\
**When** it calculates payment amounts, service fees or commissions\
**Then** it uses exact decimal types for all monetary calculations\
**And** floating-point types are never used for money

> Scenario 8: **Round half up**\
**Given** a calculated service fee has more than two decimal places\
**When** the system rounds the fee\
**Then** it applies Round Half Up to two decimal places\
**And** 2.344 EGP becomes 2.34 EGP, 2.345 EGP becomes 2.35 EGP, 2.346 EGP becomes 2.35 EGP and 1.999 EGP becomes 2.00 EGP


### US-17: Agent pays on behalf of a customer (FR-17, Should)
**As** an agent,
**I want to** pay bills on behalf of customers,
**so that** I can provide payment services to customers and earn a commission for successful payments.

> Scenario 1: **Agent pays on behalf of a customer**\
**Given** the agent is authenticated and authorized to make payments\
**When** the agent initiates a payment using the customer's bill information\
**Then** the system processes the payment on behalf of the customer\
**And** the payment is identified as an agent-assisted payment

> Scenario 2: **Agent commission is calculated**\
**Given** an agent-assisted payment has been initiated with a service fee of 2.50 EGP\
**When** the system calculates the agent's commission\
**Then** the commission is 40% of the service fee, which is 1.00 EGP

> Scenario 3: **Commission is credited after a successful payment**\
**Given** an agent-assisted payment has been initiated\
**When** the payment reaches the Completed state\
**Then** the agent's wallet is credited with 40% of the service fee\
**And** a ledger entry and an audit record are written for the commission

> Scenario 4: **No commission for unsuccessful payments**\
**Given** an agent-assisted payment has been initiated\
**When** the payment reaches Failed, Rejected or Refunded without having completed\
**Then** the agent's wallet is not credited with a commission

> Scenario 5: **No commission before completion**\
**Given** an agent-assisted payment is still being processed or is in Unknown\
**When** the system handles the agent's commission\
**Then** the commission is not credited until the payment reaches Completed


### US-18: Payment limits (FR-18, Must)
**As** a customer,
**I want** payment limits to be enforced according to my verification level and the business rules,
**so that** I can pay within the permitted limits and understand why a payment is rejected.

> Scenario 1: **Payment below the minimum**\
**Given** the customer initiates a payment\
**When** the payment amount is less than 5 EGP\
**Then** the system rejects the payment\
**And** it displays a clear message that the minimum single payment is 5 EGP

> Scenario 2: **Payment above the maximum**\
**Given** the customer initiates a payment\
**When** the payment amount is more than 10,000 EGP\
**Then** the system rejects the payment\
**And** it displays a clear message that the maximum single payment is 10,000 EGP

> Scenario 3: **Payment within the single-payment limits**\
**Given** the customer initiates a payment\
**When** the payment amount is between 5 EGP and 10,000 EGP, inclusive\
**Then** the payment proceeds to the next validation and processing steps

> Scenario 4: **Daily limit for BASIC accounts**\
**Given** the customer has a BASIC account with a daily payment limit of 5,000 EGP\
**When** the customer attempts a payment that would make the total of the day's payments exceed 5,000 EGP\
**Then** the system rejects the payment\
**And** it displays a clear message that the daily limit for BASIC accounts is exceeded

> Scenario 5: **Daily limit for VERIFIED accounts**\
**Given** the customer has a VERIFIED account with a daily payment limit of 30,000 EGP\
**When** the customer attempts a payment that would make the total of the day's payments exceed 30,000 EGP\
**Then** the system rejects the payment\
**And** it displays a clear message that the daily limit for VERIFIED accounts is exceeded

> Scenario 6: **Payment within the daily limit**\
**Given** the customer has not exceeded the daily limit for their level\
**When** the customer initiates a payment that keeps the day's total within the limit\
**Then** the payment proceeds to the next validation and processing steps


### US-19: Velocity rule (FR-19, Should)
**As** an admin,
**I want** accounts to be temporarily blocked when they make more than five payments within 60 seconds,
**so that** the system can reduce suspicious or excessive payment activity.

> Scenario 1: **Excessive payments are detected**\
**Given** a customer makes payments from the same account\
**When** the number of payments exceeds 5 within 60 seconds\
**Then** the system detects the excessive payment activity

> Scenario 2: **The account is temporarily blocked**\
**Given** the system detects more than 5 payments within 60 seconds from the same account\
**When** the velocity rule is triggered\
**Then** the account is blocked for 10 minutes\
**And** the account cannot start new payments during the blocking period

> Scenario 3: **The account is unblocked automatically**\
**Given** the account has been temporarily blocked by the velocity rule\
**When** the 10-minute blocking period ends\
**Then** the temporary block is removed automatically\
**And** the customer can make payments again, provided no other restriction applies

> Scenario 4: **No block within the allowed threshold**\
**Given** a customer makes 5 or fewer payments within 60 seconds\
**When** the system checks the velocity rule\
**Then** the account is not blocked by this rule


### US-20: Search a payment by reference (FR-20, Must)
**As** a support employee,
**I want to** search for a payment by its unique reference and see its full history,
**so that** I can investigate customer complaints and track payment problems.

> Scenario 1: **Search by payment reference**\
**Given** the support employee is authenticated and authorized\
**When** the employee searches with a valid payment reference\
**Then** the system retrieves the matching payment

> Scenario 2: **Payment details are displayed**\
**Given** the support employee has searched for an existing payment\
**When** the system retrieves the payment\
**Then** it displays the payment reference, customer, biller and bill type, amount, service fee, total amount, status and creation date and time

> Scenario 3: **Full payment history is displayed**\
**Given** the support employee has opened a payment record\
**When** the employee views its history\
**Then** the system displays all status changes and processing events in chronological order\
**And** each event shows its timestamp

> Scenario 4: **Payment reference not found**\
**Given** the support employee searches with a payment reference that does not exist\
**When** the system processes the search\
**Then** it displays a clear message that no payment was found

> Scenario 5: **Failed payment information**\
**Given** the support employee searches for a payment in the Failed state\
**When** the system displays the payment details\
**Then** it displays the failure reason when it is available


### US-21: Block and unblock accounts (FR-21, Should)
**As** a support employee,
**I want to** block or unblock customer accounts,
**so that** I can restrict suspicious or problematic accounts and restore access when appropriate.

> Scenario 1: **Block a customer account**\
**Given** the support employee is authenticated and authorized, and the customer account exists and is Active\
**When** the employee chooses to block the account\
**Then** the account status becomes Blocked\
**And** the customer cannot start payments or other restricted operations

> Scenario 2: **Unblock a customer account**\
**Given** the support employee is authenticated and authorized, and the customer account is Blocked\
**When** the employee chooses to unblock the account\
**Then** the account status becomes Active\
**And** the customer can use the account again, provided no other restriction applies

> Scenario 3: **Account already blocked**\
**Given** the customer account is already Blocked\
**When** the support employee tries to block it again\
**Then** the system informs the employee that the account is already blocked

> Scenario 4: **Account already active**\
**Given** the customer account is already Active\
**When** the support employee tries to unblock it\
**Then** the system informs the employee that the account is already active

> Scenario 5: **Status changes are recorded**\
**Given** a support employee blocks or unblocks a customer account\
**When** the system changes the account status successfully\
**Then** the system records the action\
**And** the record includes the employee who did it and the date and time of the change (FR-25)


### US-22: Payment notifications (FR-22, Must)
**As** a customer,
**I want to** receive an SMS and a push notification when my payment completes or fails,
**so that** I know the result without opening the app.

> Scenario 1: **Payment completed**\
**Given** a payment is in the SentToBiller state\
**When** the biller confirms and the payment becomes Completed\
**Then** the customer receives an SMS and a push notification containing the amount and the payment reference

> Scenario 2: **Payment failed after the biller refuses**\
**Given** a payment is in the SentToBiller state\
**When** the biller refuses and the payment becomes Failed\
**Then** the customer receives an SMS and a push notification that states the reason for the failure

> Scenario 3: **Payment failed after being Unknown**\
**Given** a payment is in the Unknown state\
**When** the biller later refuses and the payment becomes Failed\
**Then** the customer receives the same failure notification as in Scenario 2

> Scenario 4: **SMS provider is down**\
**Given** the SMS provider is down\
**When** a payment becomes Completed\
**Then** the payment stays Completed and is not undone


### US-23: Notification retry (FR-23, Should)
**As** a support employee,
**I want** failed notifications to be retried automatically and then marked as undelivered if all retries fail,
**so that** customers have the best chance to get their message and I can see which notifications were not delivered.

> Scenario 1: **Retry after a failed send**\
**Given** a notification (SMS or email) fails on its first send\
**When** the system detects the failure\
**Then** the system retries sending it, up to 3 retries after the first attempt

> Scenario 2: **A retry succeeds**\
**Given** a notification has failed on its first send\
**When** one of the retries is delivered successfully\
**Then** the notification is marked as delivered\
**And** no more retries are made

> Scenario 3: **All retries fail**\
**Given** a notification has failed on the first send and on all 3 retries (4 attempts in total)\
**When** the last retry fails\
**Then** the notification is marked as Undelivered\
**And** it is visible to support

> Scenario 4: **The payment is not affected**\
**Given** a payment has become Completed and its notification is failing\
**When** the notification is retried or marked Undelivered\
**Then** the payment stays Completed\
**And** its state is not affected by the notification failure


### US-24: Edit fee and limit values without redeploying (FR-24, Should)
**As** an admin,
**I want to** edit fee and limit values without redeploying the system,
**so that** I can update payment rules without interrupting the service.

> Scenario 1: **Update a fee value**\
**Given** an admin is logged in and the electricity fee is 2.50 EGP\
**When** the admin changes it to 3.00 EGP and saves\
**Then** the new value is saved\
**And** no redeployment or restart is needed

> Scenario 2: **Update a limit value**\
**Given** an admin is logged in and the daily limit for BASIC accounts is 5,000 EGP\
**When** the admin changes it to 6,000 EGP and saves\
**Then** the new value is saved\
**And** no redeployment or restart is needed

> Scenario 3: **New payments use the updated values**\
**Given** the admin has changed the electricity fee to 3.00 EGP\
**When** a customer starts a new electricity payment\
**Then** the fee is calculated as 3.00 EGP\
**And** payments already in progress keep the fee they were created with

> Scenario 4: **Invalid value is rejected**\
**Given** an admin enters an invalid value, such as a negative fee or a minimum payment above the maximum payment\
**When** the admin tries to save it\
**Then** the update is rejected with an error that names the invalid field\
**And** the previous valid value stays in use

> Scenario 5: **The change is audited**\
**Given** an admin has changed a fee or limit value\
**When** the change is saved\
**Then** an audit record is written with the admin's identity, the old value, the new value, the time and the reason (FR-25)

> Scenario 6: **Only admins can change values**\
**Given** a user who is not an admin\
**When** the user tries to change a fee or limit value\
**Then** access is refused\
**And** the value is unchanged


### US-25: Audit log for money movements and admin actions (FR-25, Must)
**As** an admin,
**I want** every money movement and every admin action to be written to an audit log,
**so that** any change to money or settings can be traced to who did it, when and why.

> Scenario 1: **Money movement is recorded**\
**Given** a money movement happens (top-up, hold, charge, hold release, refund, fee income or agent commission)\
**When** the movement is processed\
**Then** an audit record is written with who triggered it, when it happened and why (the reason or payment reference)

> Scenario 2: **Admin action is recorded**\
**Given** an admin adds, edits or disables a biller, a service, a fee value or a limit value\
**When** the change is saved\
**Then** an audit record is written with the admin's identity, the time, what changed and the reason

> Scenario 3: **Failed payment is also recorded**\
**Given** a payment fails after its funds were held\
**When** the hold is released to the wallet\
**Then** an audit record exists for the hold\
**And** another audit record exists for the release

> Scenario 4: **Audit records cannot be changed**\
**Given** an audit record has been written\
**When** anyone tries to edit or delete it\
**Then** the change is refused\
**And** the original record stays the same


### US-26: Daily reconciliation report (FR-26, Should)
**As** a support employee,
**I want** the system to compare each day's settlement file with PayLink's payment records and produce a difference report,
**so that** I can review every payment where PayLink and the biller disagree.

> Scenario 1: **Matching payments**\
**Given** a payment is Completed in PayLink and the settlement file lists the same payment reference with the same amount\
**When** the nightly reconciliation runs\
**Then** the report shows the payment as matched\
**And** the payment is not flagged

> Scenario 2: **Settlement file lists a payment PayLink has as Failed**\
**Given** a payment is Failed in PayLink and the settlement file lists it as received\
**When** the nightly reconciliation runs\
**Then** the report flags the payment as a difference for support to review\
**And** the payment stays Failed

> Scenario 3: **Unknown payment confirmed by the settlement file**\
**Given** a payment is Unknown and the settlement file lists it as received\
**When** the nightly reconciliation runs\
**Then** the payment becomes Completed

> Scenario 4: **Unknown payment not found in the settlement file**\
**Given** a payment is Unknown and the settlement file does not list it\
**When** the nightly reconciliation runs\
**Then** the payment becomes Failed

> Scenario 5: **Completed payment missing from the settlement file**\
**Given** a payment is Completed in PayLink and the settlement file does not list it\
**When** the nightly reconciliation runs\
**Then** the report flags the payment as a difference for support to review

> Scenario 6: **Settlement entry unknown to PayLink**\
**Given** the settlement file lists a payment reference that PayLink has no record of\
**When** the nightly reconciliation runs\
**Then** the report flags that entry as a difference for support to review


### US-27: Refund of a completed payment (FR-27, Could)
**As** a customer,
**I want to** request a refund for a Completed payment and have support approve or reject it,
**so that** I can get my money back when I paid by mistake.

> Scenario 1: **Customer requests a refund**\
**Given** a payment belongs to the customer and its state is Completed\
**When** the customer submits a refund request for it\
**Then** a refund request is created with the status Pending, linked to that payment\
**And** it appears for support to review\
**And** the payment stays Completed

> Scenario 2: **Support approves the refund**\
**Given** a refund request is Pending\
**When** a support employee approves it\
**Then** the payment amount is credited back to the customer's wallet through a ledger entry\
**And** the request becomes Approved\
**And** an audit record shows who approved it, when and why

> Scenario 3: **Support rejects the refund**\
**Given** a refund request is Pending\
**When** a support employee rejects it with a reason\
**Then** the request becomes Rejected and the wallet balance does not change\
**And** the customer can see the reason

> Scenario 4: **Payment is not Completed**\
**Given** a payment belongs to the customer and its state is not Completed\
**When** the customer submits a refund request for it\
**Then** the request is refused with a message that only Completed payments can be refunded\
**And** no refund request is created

> Scenario 5: **Payment of another customer**\
**Given** a payment belongs to a different customer\
**When** the customer submits a refund request for it\
**Then** the request is refused\
**And** no refund request is created

> Scenario 6: **Duplicate request**\
**Given** a refund request for a payment is already Pending or Approved\
**When** the customer submits another request for the same payment\
**Then** the second request is refused\
**And** the payment can never be refunded twice


### US-28: Daily totals (FR-28, Could)
**As** an admin,
**I want to** view daily totals for payments, fees earned, commissions paid and failed payments per biller,
**so that** I can monitor PayLink's daily results and see which billers have problems.

> Scenario 1: **Payment totals per biller**\
**Given** ElectroCo has two Completed payments on the selected day, of 350.00 EGP and 150.00 EGP\
**When** the admin views the daily totals\
**Then** the report shows for ElectroCo: 2 payments and a total of 500.00 EGP

> Scenario 2: **Fees earned and commissions paid**\
**Given** both of those payments have a fee of 2.50 EGP, and one of them was paid by an agent\
**When** the admin views the daily totals\
**Then** the report shows for ElectroCo: fees earned 5.00 EGP and commissions paid 1.00 EGP (40% of 2.50 EGP)

> Scenario 3: **Failures per biller**\
**Given** AquaCo has one Failed payment on the selected day\
**When** the admin views the daily totals\
**Then** the report shows for AquaCo: 1 failed payment

> Scenario 4: **Only Completed payments count as money**\
**Given** a payment on the selected day is in the state Rejected, Unknown or Failed\
**When** the admin views the daily totals\
**Then** its amount, fee and commission are not included in the payment, fee and commission totals

> Scenario 5: **A day with no activity**\
**Given** no payments exist for the selected day\
**When** the admin views the daily totals\
**Then** every biller shows 0 payments, 0.00 EGP in payments, 0.00 EGP in fees, 0.00 EGP in commissions and 0 failures

> Scenario 6: **Only admins can view it**\
**Given** a user who is not an admin\
**When** the user tries to view the daily totals\
**Then** access is refused\
**And** no totals are shown


### Non-functional Requirements

| ID | Quality | Requirement (measurable) | How we measure it |
|---|---|---|---|
| NFR-1 | Correctness | Zero double charges and zero negative balances in the concurrency and duplicate-request tests | Automated tests for hard case 1 (same payment sent twice) and hard case 4 (two payments at the same moment) |
| NFR-2 | Performance | The system sustains 100 payments per second for 1 minute against the mock billers | Load test, with the result shown in the final report |
| NFR-3 | Resilience | When one biller is down or slow, payments to the other billers still complete | Automated test with one mock biller switched off or delayed |
| NFR-4 | Modifiability | Adding a new biller changes only the Biller Integration module and configuration | Compare the changed files in the pull request (hard case 8) |
| NFR-5 | Auditability | 100% of money movements have an audit record that shows who, when and why | Test that compare distinct transaction_id values with audit records. |
| NFR-6 | Testability | At least 70% line coverage on domain logic | Coverage report produced by the CI pipeline |

### Priorities

| Priority | Requirements |
|---|---|
| **Must** | FR-1, FR-3, FR-4, FR-5, FR-6, FR-7, FR-8, FR-9, FR-10, FR-11, FR-12, FR-13, FR-14, FR-15, FR-16, FR-18, FR-20, FR-22, FR-25 |
| **Should** | FR-2, FR-17, FR-19, FR-21, FR-23, FR-24, FR-26 |
| **Could** | FR-27, FR-28 |

Story US-n implements requirement FR-n and has the same priority.

- **Must:** the system is not acceptable without it.
- **Should:** important, but the system still works without it.
- **Could:** nice to have if time allows.


### Questions or ambiguities

**Q1. What identifies a customer (phone number, email, or both)? (US-1)**\
Decision: The phone number is the unique identifier.

**Q2. Who triggers the upgrade to VERIFIED (admin or customer)? (US-2)**\
Decision: The customer requests it, and an admin approves or rejects it with a reason. The request states are Pending, Approved and Rejected.

**Q3. Should the transaction history include only completed movements, or also holds and releases? (US-3)**\
Decision: All ledger entries, so customers can see held money.

**Q4. Is there a minimum, maximum or daily limit for top-ups? (US-4)**\
Decision: A minimum of 10 EGP per top-up and a daily top-up limit of 30,000 EGP.

**Q5. Does an agent's cash top-up count toward the customer's daily limit, and does it take money from the agent's wallet? (US-4)**\
Decision: Yes, it counts toward the limit. No, it does not take money from the agent's wallet.

**Q6. Does a hold create one ledger entry or two? (US-5)**\
Decision: Two entries that share a transaction ID, written in one database transaction.
**Q6.1 How many ledger entries does a money movement create?**\
Decision: As many as the movement needs, all with the same transaction_id.

**Q7. Can a disabled biller be re-enabled? (US-6)**\
Decision: Yes.

**Q8. A service's minimum and maximum versus the global 5 to 10,000 EGP rule: which one wins? (US-6)**\
Decision: The stricter one applies. Admins cannot save a service range outside the global range.

**Q9. Should the biller list be public or only for logged-in customers? (US-7)**\
Decision: Only for logged-in customers.

**Q10. What customer information is required for each biller, and what should happen if the bill is already paid or expired? (US-8)**\
Decision: Use each biller's required fields, and reject already-paid or expired bills.

**Q11. Is there a timeout for bill inquiries, and should failed inquiries be retried automatically or only when the customer asks again? (US-8)**\
Decision: Use the 10-second timeout. The customer retries a failed inquiry manually.

**Q12. Should the wallet hold cover the bill amount plus the service fee, and what happens if the available balance changes before the hold is created? (US-9, US-12)**\
Decision: Hold the full amount including the fee, after checking the available balance again.

**Q13. What is the scope and retention period of an idempotency key, and can it be reused if the original request failed before a payment was created? (US-10)**\
Decision: Keys are unique per customer and endpoint. A key can be reused if no payment was created.Rejected payment uses up its idempotency key, so a new attempt needs a new key.

**Q14. What error should be returned for a refused state transition, and should repeated transitions to the same state or manual support transitions be allowed? (US-11)**\
Decision: Return an invalid-transition error and ignore duplicate transitions. Manual transitions by support are not allowed.

**Q15. Is the Failed to Refunded transition automatic, and what happens if the release ledger entry fails? Should holds and releases also have audit records? (US-11, US-12)**\
Decision: The refund happens automatically after a successful hold release. If the release entry fails, the payment stays Failed and the release is retried. All holds and releases are audited.

**Q16. Does the 10-second payment timeout apply to all billers, and are scheduled re-queries kept if the system restarts? (US-13)**\
Decision: It applies to live billers only (ElectroCo and AquaCo). Scheduled re-queries are saved so they survive a restart.

**Q17. Can a customer start another payment for the same bill while the original payment is still Unknown? (US-13)**\
Decision: No. The customer must wait until the original payment is resolved.

**Q18. What happens if a callback arrives before the payment reaches SentToBiller, or if a "failed" callback arrives while the payment is Unknown? (US-14)**\
Decision: Early callbacks are ignored and logged. A "failed" callback for an Unknown payment moves it to Failed, then Refunded.

**Q19. Which rounding method is used for fees: Round Half Up or Round Half Even? (US-16)**\
Decision: Round Half Up to two decimal places.

**Q20. What happens if the calculated mobile top-up fee is less than 1 EGP? (US-16)**\
Decision: The fee is 1.00 EGP, because the brief sets a minimum fee of 1 EGP.

**Q21. Does "failure" in FR-22 include Rejected payments? (US-22)**\
Decision: The customer is shown the reason for a Rejected payment in the app. An SMS is sent only for Failed payments.

**Q22. Does "retried 3 times" mean 3 retries after the first attempt, or 3 attempts in total? (US-23)**\
Decision: 3 retries after the first attempt (4 attempts in total).

**Q23. Is there a waiting time between notification retries? (US-23)**\
Decision: The brief gives no waiting time, so retries are made one after another with no delay. The delay is a configuration value, so it can be added later without changing code.

**Q24. Are updated fees and limits applied immediately to new payments, or from a specified time? (US-24)**\
Decision: Immediately, for all new payments. Payments already in progress keep the values they were created with.

**Q25. What happens if the admin enters an invalid value? (US-24)**\
Decision: The system rejects the update with a clear error and keeps the previous valid value.

**Q26. Should every fee and limit update be recorded in the audit log, including the admin, the old value, the new value and the time? (US-24, US-25)**\
Decision: Yes, because FR-25 requires every admin action to be audited.

**Q27. Who is allowed to view audit records? (US-25)**\
Decision: Only admins and support employees, and read-only. Customers cannot view them.

**Q28. Should audit records be kept permanently or for a limited period? (US-25)**\
Decision: They are kept permanently and are never deleted.

**Q29. Does "audit log" mean the same thing as the ledger (FR-5)? (US-25)**\
Decision: No. The ledger records money amounts, and the audit log records who, when and why. Every ledger entry also has an audit record.

**Q30. Does a "difference" in reconciliation include a wrong amount (same reference, different amount)? (US-26)**\
Decision: Yes, it is flagged in the same way.

**Q31. Does the reconciliation fix a Failed payment automatically if the biller did receive it? (US-26)**\
Decision: No. It is only flagged for support to review.

**Q32. Is there one settlement file and one report per biller per day? (US-26)**\
Decision: Yes, one report per biller per day. NetCo's batch file is optional.

**Q33. Completed is a final state and only Failed can become Refunded, so how can a Completed payment be refunded? (US-27)**\
Decision: The payment stays Completed. The refund is a separate refund request that creates new ledger entries returning money to the wallet, so the payment state diagram is not changed.

**Q34. Is the service fee refunded too? (US-27)**\
Decision: Yes, the full amount paid (bill amount plus fee) is credited back.

**Q35. Does the "payments" total in the daily report include Rejected and Unknown payments? (US-28)**\
Decision: No. Only Completed payments count as money. Failed payments are counted separately in the failures column.

**Q36. Does "failures per biller" include Rejected payments? (US-28)**\
Decision: No. Only Failed payments (the biller side) are counted.

**Q37. What counts as "a day" in the daily report? (US-28)**\
Decision: Cairo time (Africa/Cairo), from midnight to midnight.

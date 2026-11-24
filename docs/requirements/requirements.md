# Requirements

### Problem Summary

### Stakeholders

### User Stories
US-1
**As** a new customer,
**I want to** register for a PayLink account,
**so that** I get a wallet and can start paying bills within the basic limits.

> Scenario 1: Successful registration
**Given** a person does not have a PayLink account and provides a valid name, phone number and password.
**When** they submit the registration request.
**then** a customer account is created with verification level "BASIC",
and an empty EGP wallet is created for the new user (available balance 0.00, held balance 0.00),
and the daily payment limit of 5,000 EGP applies to the account,
and an entry is recorrded in audit log with who, when and why (account creation) to satisfy FR-25.

> Scenario 2: Phone number already registered
**Given** a customer account already exists with phone number 01#########
**When** someone tries to register again with the same phone number
**Then** the registration is rejected with a "phone number already registered" error,
and no new account or wallet is created.

> Scenario 3: Invalid input
**Given** a person submits a registration with a missing name, a malformed phone number or a weak password
**When** the registration request is processed
**Then** it is rejected with a validation error that names each invalid field,
and no new account or wallet is created.


### Non-functional Requirements

### Priorities

### Questions or ambiguities
- what identifies a customer (phone number, email, or both)?
Phone number is assumed as the unique identifier.
- Is phone verification by OTP required, or simulated? 


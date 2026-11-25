# Prompt.log

## 1
Date: 24-Nov-2026\
Author: Salma Kandil\
Stage: requirements\
Tool and model: claude projects (Sonnet 5.5 Medium)\
Prompt: turn FR-1 into a user story written with the format given/when/then\
Summary of answer: a user story for FR-1 with 4 different Scenarios (Successful - Already registered - Invalid - Audit trail)\
Verdict & why: Fixed (Audit log is added to the end of the successful scenario)


## 2
Date: 24-Nov-2026\
Author: Salma Kandil\
Stage: requirements\
Tool and model: claude projects (Sonnet 5.5 Medium)\
Prompt: turn FR-2 into a user story written with the format given/when/then\
Summary of answer: a user story for FR-2 with 2 different Scenarios (Upgrade succeeds - Not upgraded)\
Verdict & why: Rejected (it gave the user story from the perspective of the admin:
"As an admin, I want to upgrade a customer to VERIFIED ..."
I found "As a basic customer, I want to be upgraded to VERIFIED ..." to make the most sense as getting an upgrade is a user choice
also adding another scenario: already VERIFIED)


## 3
Date: 24-Nov-2026\
Author: Salma Fathy\
Stage: requirements\
Tool and model: ChatGPT-GPT-6\
Prompt: Create user stories for FR-13. Constraints: Follow the timeout rules in the brief.\
Summary of answer: cover re-query attempts and unresolved payments, and avoid inventing retry behavior.	Generated scenarios with retry intervals and automatic refunds\
Verdict & why: Rejected: The output assumed that if a payment is unresolved it should be refunded automatically, while in this case the conflict should be resolved through reconiliation\


## 4
Date: 24-Nov-2026\
Author: Salma Fathy\
Stage: requirements\
Tool and model: ChatGPT-GPT-6\
Prompt: Create user stories for FR-14.\
Summary of answer: generated callback scenarios but did not cover a failed callback arriving after a completed payment \
Verdict & why: fixed: added a conflicting callback scenario to ensure completed payment remains unchanged and the conflict is logged.\

## 5
Date: 26-Nov-2026\
Author: Sohila Maher\
Stage: Requirements\
Tool and model: Claud.ai\
Prompt: Act as a Senior Business Analyst.\
Analyze this FR-15 to FR-21
User Story must follow this format:As a [user role],
I want to [action], So that [business value].
Provide acceptance criteria for the User Stories using Given, When, Then format. Do not remove any mandatories.
Summary of answer: given 7 user stories\
Verdict & why: Fixed "some of scenarioes where missing parts like billers to check if they are avalable not just active to prevent servers from breaking down, also detailing the process like instead of saying{it uses AquaCo's XML\

## 6
Date: 24-Nov-2026\
Author: Shrouk Esam\
Stage: requirements\
Tool and model: claude.ai \
Prompt: Write a user story for FR-22 (customer receives an SMS and a push message on completion and on failure) as given/when/then, using only the payment states from the brief\
Summary of answer: a user story for FR-22 with 3 criteria (Completed notification, Failed notification, notification accuracy)\
Verdict & why: Fixed ("in progress" is not a valid state, so it became SentToBiller; the "accuracy" criterion was untestable, so it was removed; added Unknown → Failed and the SMS-provider-down case, so the final story has 4 criteria)\

## 7
Date: 24-Nov-2026\
Author: Shrouk Esam\
Stage: requirements\
Tool and model: claude.ai \
Prompt: (your prompt) Write a user story for FR-27 (a customer can request a refund of a completed payment, support approves it) as given/when/then\
Summary of answer: a user story for FR-27 with 4 scenarios (request refund, support approves, not Completed, another customer's payment)\
Verdict & why: Fixed (ignored that Completed is a final state and only Failed can become Refunded, so we decided the payment stays Completed and the refund is a separate record; approval did not return money, so a ledger entry and audit record were added; added the reject and duplicate-request scenarios)\



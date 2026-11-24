## 22
Date: 24-Nov-2026
Author: Shrouk Esam
Stage: requirements
Tool and model: claude.ai 
Prompt: Write a user story for FR-22 (customer receives an SMS and a push message on completion and on failure) as given/when/then, using only the payment states from the brief
Summary of answer: a user story for FR-22 with 3 criteria (Completed notification, Failed notification, notification accuracy)
Verdict & why: Fixed ("in progress" is not a valid state, so it became SentToBiller; the "accuracy" criterion was untestable, so it was removed; added Unknown → Failed and the SMS-provider-down case, so the final story has 4 criteria)

## 27
Date: 24-Nov-2026
Author: Shrouk
Stage: requirements
Tool and model: claude.ai 
Prompt: (your prompt) Write a user story for FR-27 (a customer can request a refund of a completed payment, support approves it) as given/when/then
Summary of answer: a user story for FR-27 with 4 scenarios (request refund, support approves, not Completed, another customer's payment)
Verdict & why: Fixed (ignored that Completed is a final state and only Failed can become Refunded, so we decided the payment stays Completed and the refund is a separate record; approval did not return money, so a ledger entry and audit record were added; added the reject and duplicate-request scenarios)
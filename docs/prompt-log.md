# Prompt.log

## 1
Date: 24-Nov-2026
Author: Salma Kandil
Stage: requirements
Tool and model: claude projects (Sonnet 5.5 Medium)
Prompt: turn FR-1 into a user story written with the format given/when/then
Summary of answer: a user story for FR-1 with 4 different Scenarios (Successful - Already registered - Invalid - Audit trail)  
Verdict & why: Fixed (Audit log is added to the end of the successful scenario)


## 2
Date: 24-Nov-2026
Author: Salma Kandil
Stage: requirements
Tool and model: claude projects (Sonnet 5.5 Medium)
Prompt: turn FR-2 into a user story written with the format given/when/then
Summary of answer: a user story for FR-2 with 2 different Scenarios (Upgrade succeeds - Not upgraded) 
Verdict & why: Rejected (it gave the user story from the perspective of the admin:
"As an admin, I want to upgrade a customer to VERIFIED ..."
I found "As a basic customer, I want to be upgraded to VERIFIED ..." to make the most sense as getting an upgrade is a user choice
also adding another scenario: already VERIFIED)



Goal: add input validation to <REQUEST DTOs>. 

Context: #file:docs/api-contracts.md, #file:docs/business-rules.md. 

Constraints: Bean Validation annotations on records; custom validator only when a standard 

annotation cannot express the rule; controllers annotated @Validated; messages are short 

and user-safe. 

Tests: controller IT cases returning 400 with the field name in details. 

Process: plan, wait, implement, list files. 

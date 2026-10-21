Goal: review the current changes of <FEATURE> as a strict senior reviewer. 

Context: #file:docs/api-contracts.md, #file:docs/business-rules.md, 

#file:docs/engineering-standards.md, the changed files. 

Checklist: contract compliance; layering; transaction boundaries; concurrency; idempotency; 

validation; authorisation and ownership; error codes; BigDecimal use; logging of secrets; 

test coverage of negative paths; unrelated changes. 

Output: a table of findings (severity, file:line, issue, suggested fix). Do not change code. 

Goal: implement the business logic of <USE CASE> in <Service>Service. 

Context: #file:docs/business-rules.md (rules <IDS>), #file:docs/api-contracts.md, the entity 

and repository classes. 

Constraints: @Transactional on the service method only; no remote calls inside the 

transaction; throw <DomainException> with codes from business rules; invalid state 

transitions -> InvalidStateException (409). 

Acceptance criteria: <list each rule as an observable behaviour>. 

Tests: <Service>Test with Mockito for every rule, including the negative cases. 

Process: plan, wait, implement, list files. 

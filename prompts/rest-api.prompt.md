Goal: implement the <ENDPOINT LIST> endpoints of the <SERVICE> service exactly as specified. 

Context: #file:docs/api-contracts.md (section <SERVICE>), #file:docs/business-rules.md, 

#file:docs/engineering-standards.md, existing: <controller/service/entity files>. 

Constraints: controller -> service -> repository; DTOs as records in api package; standard 

ApiError via the existing GlobalExceptionHandler; no new dependencies; do not touch other 

features; preserve existing endpoints. 

Acceptance criteria: <copy the acceptance criteria lines for this feature>. 

Tests: <Feature>ControllerIT with happy path, 400 validation, 401, 403, 404, 409 cases; 

<Service>Test unit tests for each business rule R-<...>; each @DisplayName starts with the AC-/R- IDs it proves. 

Process: produce a numbered plan of files and tests, wait for approval, then implement, 

then list every changed file. 

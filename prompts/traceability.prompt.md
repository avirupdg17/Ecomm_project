Goal: create or update docs/traceability-matrix.md for <SERVICES>. 

Context: #file:docs/acceptance-criteria.md, #file:docs/business-rules.md, 

#file:docs/api-contracts.md, #file:docs/traceability-matrix.md (if it exists), and the 

'Traceability' section of the latest CI job summary of each service, pasted below. 

Constraints: one row per acceptance criterion; columns Story | AC | Rules | Endpoint | 

Test (Class#method) | Status (Covered / Gap); an AC or rule counts as Covered only if a test 

whose @DisplayName starts with its ID passed on the runner; never invent a test name; do not 

change code or tests. 

Output: the updated matrix, then a gap list with the test you propose for each gap. 

Process: plan, wait for approval, write the file. 

<paste the Traceability lines from each service's CI job summary here> 

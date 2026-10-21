Goal: verify the generated artefact <FILE> against the authoritative source and report 

every discrepancy. 

Context: #file:<FILE>; the authoritative source attached or pasted below (Reference D/E 

tables, docs/api-contracts.md or docs/business-rules.md). 

Constraints: do not edit any file; compare item by item (paths, methods, roles, status codes, 

error codes, field names and types, constraints, state transitions); flag missing, invented 

and different items; the authoritative source always wins. 

Output: a table (location, generated value, authoritative value, type: missing / invented / different), then one verdict line: CONSISTENT or <N> discrepancies. 

Process: list what you will compare, compare, report. 

Goal: implement JWT security for the <SERVICE> service. 

Context: #file:.github/instructions/security.instructions.md, #file:docs/architecture.md 

(security section), existing config classes. 

Constraints: SecurityFilterChain with public paths as listed; JwtAuthenticationFilter 

validating HS256 with JWT_SECRET; roles mapped to ROLE_*; method security enabled; 

401/403 through the standard error body; no secrets in code. 

Tests: SecurityIT: public path OK without token, protected path 401 without token, 401 

expired token, 403 wrong role, 200 correct role. 

Process: plan, wait, implement, list files. 

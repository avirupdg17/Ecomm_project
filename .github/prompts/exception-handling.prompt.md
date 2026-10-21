Goal: add or extend the global exception handling of the <SERVICE> service. 

Context: #file:docs/api-contracts.md (standard error body), existing GlobalExceptionHandler 

and ApiError record if present. 

Constraints: one @RestControllerAdvice; map MethodArgumentNotValidException -> 400 

VALIDATION_ERROR with field details; domain NotFound -> 404; Conflict/InvalidState -> 409; 

IdempotencyConflict -> 422; AccessDenied -> 403; Authentication -> 401; DependencyUnavailable 

-> 503; any other -> 500 INTERNAL_ERROR without internals. Include correlationId from MDC. 

Tests: GlobalExceptionHandlerTest asserting status, code and that no stack trace leaks. 

Process: plan, wait, implement, list files. 

Goal: create or adjust .github/workflows/ci.yml for the <SERVICE> service. 

Context: #file:docs/engineering-standards.md (CI section), existing workflow if any. 

Constraints: ubuntu-latest; actions/checkout@v4; actions/setup-java@v4 temurin 21 with 

cache maven; mvn -B -ntp verify; concurrency group; timeout-minutes 15; upload 

surefire/failsafe reports and the JAR with retention-days 3; permissions contents: read; 

never add secrets in plain text; no Docker in ci.yml (containers run only in the ecom-platform k8s and portability workflows); keep the report-only traceability step. 

Process: show the full file, explain each change, wait for approval. 

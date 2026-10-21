Goal: configure persistence for the <SERVICE> service. 

Context: #file:docs/engineering-standards.md, #file:docs/data-model.md. 

Constraints: application.yml with H2 file database ./data/<service>db (runtime) and 

application-test.yml with H2 in-memory; ddl-auto=update; idempotent DataSeeder 

CommandLineRunner creating only missing seed rows; data/ in .gitignore. 

Tests: DataSeederTest proving a second run inserts nothing. 

Process: plan, wait, implement, list files. 

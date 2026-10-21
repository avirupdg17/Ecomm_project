Goal: create JPA entities and Spring Data repositories for <ENTITY LIST> of the <SERVICE>. 

Context: #file:docs/data-model.md (section <SERVICE>), #file:docs/engineering-standards.md. 

Constraints: UUID string ids; Instant createdAt/updatedAt via @PrePersist/@PreUpdate; 

@Version where listed; unique constraints and indexes exactly as in the data model; enums 

for states; BigDecimal(12,2) for money; no Lombok. 

Acceptance criteria: schema generates on H2 with ddl-auto=update without errors; unique 

constraints enforced. 

Tests: <Entity>RepositoryTest with @DataJpaTest: save/find, unique violation, version 

increment on update. 

Process: plan, wait for approval, implement, list files. 

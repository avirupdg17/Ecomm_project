---

applyTo: "**/*.java"

---

- Java 21: use records for DTOs, sealed interfaces or enums for states, switch expressions,

 text blocks. No Lombok.

- Constructor injection only (no @Autowired on fields). One public class per file.

- Validation annotations on request records (@NotBlank, @Email, @Positive, @DecimalMin,

 @Size). Controllers use @Valid and @Validated.

- @Transactional only on service methods; never wrap a remote HTTP call inside a

 transaction.

- Entities: @Id String id = UUID.randomUUID().toString(); @Version Long version where

 concurrent updates are possible; Instant createdAt/updatedAt with @PrePersist/@PreUpdate.

- Repositories extend JpaRepository; custom queries use @Query with named parameters;

 paginated queries return Page<T>.

- Do not catch and swallow exceptions. Translate exceptions at the service boundary into

 domain exceptions with a code.

- Use Optional only as a return type. Avoid null returns from service methods.

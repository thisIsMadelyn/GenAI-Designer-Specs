# Agent 3 — Database Agent

## Role
The Database Agent is the **third agent** in the pipeline. It is responsible for generating all database-related code: the JPA entities that map to database tables, the Spring Data repositories for data access, and the `application.properties` configuration file.

---

## Input
- Full output from **Agent 1 (System Analyst)**: entities list, tech stack (database type, Java version, Spring Boot version).
- Full output from **Agent 2 (Architect)**: package structure (to place entities and repositories in the correct packages).

---

## What It Does

1. **Generates JPA Entity classes** — one Java class per entity identified by the System Analyst. Each entity includes:
   - `@Entity`, `@Table` annotations
   - Primary key with `@Id` and `@GeneratedValue`
   - All relevant fields with appropriate JPA annotations (`@Column`, `@NotNull`, etc.)
   - Relationships between entities (`@OneToMany`, `@ManyToOne`, `@ManyToMany`) where applicable
   - Lombok annotations (`@Data`, `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor`) for boilerplate reduction

2. **Generates Spring Data Repository interfaces** — one interface per entity, extending `JpaRepository<Entity, ID>`. Includes:
   - Custom query methods (e.g. `findByEmail`, `findByStatusAndCreatedAtAfter`)
   - `@Query` annotations where JPQL or native SQL is needed

3. **Generates `application.properties`** — full Spring Boot configuration including:
   - DataSource URL, username, password placeholders
   - Hibernate dialect matching the chosen database
   - DDL auto setting (`validate` for production, `update` for development)
   - Connection pool settings
   - JPA/Hibernate logging options

---

## Output (Pydantic Schema: `DatabaseOutput`)

```json
{
  "entities": [
    {
      "class_name": "Order",
      "package": "com.example.orderservice.entity",
      "code": "@Entity\n@Table(name = \"orders\")\n@Data\npublic class Order {\n  ...\n}"
    }
  ],
  "repositories": [
    {
      "class_name": "OrderRepository",
      "package": "com.example.orderservice.repository",
      "code": "public interface OrderRepository extends JpaRepository<Order, Long> {\n  ...\n}"
    }
  ],
  "application_properties": "spring.datasource.url=jdbc:mysql://...\n..."
}
```

---

## Role in the Pipeline
The Database Agent output feeds directly into the **Backend Layer Agent**, which uses the entities and repositories to build services and controllers. The entity class names and repository method signatures must be consistent so the backend layer can reference them correctly.

---

## Key Responsibilities Summary
| Responsibility | Description |
|---|---|
| JPA Entities | Full Java entity classes with annotations and relationships |
| Repositories | Spring Data interfaces with custom query methods |
| application.properties | Complete datasource and Hibernate configuration |

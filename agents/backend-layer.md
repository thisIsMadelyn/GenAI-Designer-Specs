# Agent 4 — Backend Layer Agent

## Role
The Backend Layer Agent is the **fourth agent** in the pipeline and the most code-intensive one. It generates the complete business logic and API layer of the Java microservice: DTOs, Service interfaces and implementations, REST Controllers, and Swagger/OpenAPI documentation annotations.

---

## Input
- Full output from **Agent 1 (System Analyst)**: service name, entities, functional requirements.
- Full output from **Agent 2 (Architect)**: package structure.
- Full output from **Agent 3 (Database Agent)**: entity class definitions and repository interfaces (to reference correctly in service and controller code).

---

## What It Does

1. **Generates DTO classes** (Data Transfer Objects):
   - **Request DTOs** — input validation objects for API endpoints (with `@Valid`, `@NotNull`, `@Size` etc.)
   - **Response DTOs** — shaped output objects returned to the API consumer
   - Uses Lombok (`@Data`, `@Builder`) to minimize boilerplate

2. **Generates Service interfaces** — defines the contract (method signatures) for each business operation (e.g. `createOrder`, `getOrderById`, `updateOrderStatus`, `deleteOrder`).

3. **Generates Service implementations** — concrete classes (`@Service`) that:
   - Implement the service interface
   - Call the appropriate repositories
   - Apply business logic and validation
   - Map between entities and DTOs
   - Handle exceptions (throwing custom exceptions like `ResourceNotFoundException`)

4. **Generates REST Controllers** — `@RestController` classes that:
   - Map HTTP endpoints (`@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`)
   - Use `@RequestBody`, `@PathVariable`, `@RequestParam` correctly
   - Return `ResponseEntity<T>` with appropriate HTTP status codes
   - Inject and call the service layer (never repositories directly)

5. **Adds Swagger/OpenAPI annotations** — `@Operation`, `@ApiResponse`, `@Tag` on all controller methods and classes for automatic API documentation generation.

---

## Output (Pydantic Schema: `BackendLayerOutput`)

```json
{
  "dtos": [
    {
      "class_name": "CreateOrderRequest",
      "package": "com.example.orderservice.dto.request",
      "code": "@Data\n@Builder\npublic class CreateOrderRequest {\n  @NotNull\n  private Long customerId;\n  ...\n}"
    }
  ],
  "services": [
    {
      "class_name": "OrderService",
      "type": "interface",
      "package": "com.example.orderservice.service",
      "code": "public interface OrderService {\n  OrderResponse createOrder(CreateOrderRequest request);\n  ...\n}"
    },
    {
      "class_name": "OrderServiceImpl",
      "type": "implementation",
      "package": "com.example.orderservice.service.impl",
      "code": "@Service\npublic class OrderServiceImpl implements OrderService {\n  ...\n}"
    }
  ],
  "controllers": [
    {
      "class_name": "OrderController",
      "package": "com.example.orderservice.controller",
      "code": "@RestController\n@RequestMapping(\"/api/orders\")\n@Tag(name = \"Orders\")\npublic class OrderController {\n  ...\n}"
    }
  ]
}
```

---

## Role in the Pipeline
The Backend Layer Agent produces the **heart of the microservice** — everything the end user of the API will interact with. Its output is used by the **Testing Manager Agent** to generate unit tests that cover service methods and controller endpoints.

---

## Key Responsibilities Summary
| Responsibility | Description |
|---|---|
| Request/Response DTOs | Input validation and output shaping objects |
| Service interfaces | Business operation contracts |
| Service implementations | Business logic with repository calls and entity mapping |
| REST Controllers | HTTP endpoint definitions with status codes |
| Swagger annotations | Auto-generated API documentation |

# Agent 6 — Testing Manager

## Role
The Testing Manager is the **sixth** and final agent in the pipeline. It reviews the complete output of all previous agents and produces: a comprehensive JUnit 5 test suite covering the service layer and controllers, an error report highlighting any issues or inconsistencies found in the generated code, and a coverage estimate.

---

## Input
- Full output from **Agent 1 (System Analyst)**: functional requirements (used to ensure tests cover all required behaviors).
- Full output from **Agent 3 (Database Agent)**: entity classes and repository interfaces.
- Full output from **Agent 4 (Backend Layer Agent)**: service interfaces, service implementations, and controller classes — the primary targets for testing.

---

## What It Does

1. **Generates JUnit 5 unit tests for Service layer**:
   - Creates a test class for each service implementation (e.g. `OrderServiceImplTest`)
   - Uses `@ExtendWith(MockitoExtension.class)` for Mockito-based testing
   - Mocks repository dependencies with `@Mock`
   - Injects the service under test with `@InjectMocks`
   - Writes individual `@Test` methods for each service operation:
     - Happy path (successful execution)
     - Edge cases (empty results, null inputs)
     - Exception scenarios (resource not found, validation failures)
   - Uses `assertThat` (AssertJ), `assertEquals`, `assertThrows` for assertions
   - Uses `verify()` to confirm repository interactions

2. **Generates JUnit 5 integration tests for Controller layer**:
   - Uses `@WebMvcTest` with `MockMvc` for controller slice testing
   - Mocks the service layer with `@MockBean`
   - Tests each endpoint with realistic request bodies and expected response structures
   - Verifies HTTP status codes and response JSON fields
   - Tests error responses (404, 400, 500)

3. **Produces an Error Report**:
   - Reviews all generated code from previous agents for potential issues
   - Reports: missing imports, logical inconsistencies between layers, entity fields referenced but not defined, naming mismatches between DTOs and entities, missing exception handlers, incomplete configurations
   - Each issue includes: severity (HIGH / MEDIUM / LOW), location (class name), and suggested fix

4. **Estimates code coverage**:
   - Provides an approximate coverage percentage based on the number of methods tested vs. total methods generated
   - Breaks down coverage by layer (service, controller)

---

## Output (Pydantic Schema: `TestingOutput`)

```json
{
  "test_classes": [
    {
      "class_name": "OrderServiceImplTest",
      "package": "com.example.orderservice.service.impl",
      "code": "@ExtendWith(MockitoExtension.class)\npublic class OrderServiceImplTest {\n  @Mock\n  private OrderRepository orderRepository;\n  @InjectMocks\n  private OrderServiceImpl orderService;\n  ...\n}"
    },
    {
      "class_name": "OrderControllerTest",
      "package": "com.example.orderservice.controller",
      "code": "@WebMvcTest(OrderController.class)\npublic class OrderControllerTest {\n  ...\n}"
    }
  ],
  "error_report": {
    "issues": [
      {
        "severity": "HIGH",
        "location": "OrderServiceImpl",
        "description": "Missing @Transactional on createOrder method",
        "suggested_fix": "Add @Transactional annotation to createOrder"
      }
    ],
    "total_issues": 1,
    "summary": "1 high severity issue found. Overall code quality is good."
  },
  "coverage_estimate": {
    "service_layer": "85%",
    "controller_layer": "78%",
    "overall": "82%"
  }
}
```

---

## Role in the Pipeline
The Testing Manager is the **last agent** and has the broadest view of the entire generated codebase. It serves two purposes: quality assurance (via the error report) and completeness verification (via the test suite). Its output is the final piece of the structured result shown to the user in the frontend's 6-tab view.

---

## Key Responsibilities Summary
| Responsibility | Description |
|---|---|
| Service unit tests | JUnit 5 + Mockito tests for all service operations |
| Controller tests | MockMvc-based tests for all REST endpoints |
| Error report | Issues found in generated code with severity and fix suggestions |
| Coverage estimate | Approximate % of code covered by the generated tests |

# Agent 1 - System Analyst 

## Role
the System analyst is the **first** agent in the pipeline. It recieves the raw user input from the design
from and transforms  it into a structured, actionable plan that all subsequent agents will follow.

---

## Input 
The agent recieves the following fields from the user's form submission:
- **Service name** - the name of the Java microservice
- **Description** - a free-text descriptionof what the service should do
- **Builder tool** - Maven
- **SpringBoot version**
- **Java Version**

## What It does

1. **Parses and interprepts** the user's description to understand the bussiness domain and and funtional requirements.
2. **Identifiess entities** - the main data models the service will need (e.g. 'User', 'Order', 'Product').
3. **Defines Techstack** - confirms or refines the chosen technologies based on the input.
4. **Produces an agent excecution plan** - a structured breakdown of what each subsequent agent (Architect, Database, Backend Layer, DevOps, Testing) should build, ensuring consistency across the entire pipeline.
5. **lists functional requirements** - clear, numbered requirements extracted from the description.
6. **Lists non-functional requirements** - scalability, REST compliance, security.

---

## Output (Pydantic Schema: `SystemAnalystOutput`)

```json
{
  "service_name": "order-service",
  "description": "Manages customer orders...",
  "entities": ["Order", "OrderItem", "Customer"],
  "tech_stack": {
    "language": "Java 17",
    "framework": "Spring Boot 3.2",
    "database": "MySQL",
    "build_tool": "Maven"
  },
  "functional_requirements": [
    "Create and manage orders",
    "Track order status",
    "..."
  ],
  "non_functional_requirements": [
    "RESTful API",
    "JWT Authentication",
    "..."
  ],
  "agent_plan": {
    "architect": "Design package structure and UML diagrams for Order, OrderItem, Customer",
    "database": "Create JPA entities and repositories for ...",
    "..."
  }
}
```

---

## Role in the Pipeline
The system Analyst output is passed as context to **all other agents**.It acts as the single source
of truth for the service's purpose , entities and technical constraints. Without a well-structured analyst output , downstream agents may produce incinsistent or incomplete code.

---

## Key Responsibilities
| Responsibility | Description |
|---|---|
| Requirement extraction | Turns free-text into structured requirements |
| Entity identification | Lists all domain models needed |
| Techstack confirmation | Validates user choices, fills in defaults |
| Agent coordination plan | Tells each agent agent exaclty what to build |


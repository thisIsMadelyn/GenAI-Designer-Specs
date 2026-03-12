# Agent 2 - Architect

## Role 
The Architecture is the  ***second** agent in the pipeline. It takes the structures 
plan from the System Analyst and translates it into a concrete software architecture: 
the package structure of the Java project and UML diagrams that visualize the system's design.

---

## Input
- Full output from **Agent 1 (System Analyst)**: service name, entities, techstack, functional
requirements and agent plan.

---

## What It does

1. **Designs the Java package structure** - defines how the project's source code will
be organized (controllers, services, repositories, DTOs, entities, security config, etc.).
2. **Creates a UML Class Diagram** (PlantUML syntax) — shows all classes, their fields, methods, and relationships (inheritance, composition, associations).
3. **Creates a UML Sequence Diagram** (PlantUML syntax) — illustrates the flow of a key use case (e.g. creating an order), showing how components interact at runtime.
4. **Defines architectural patterns** — confirms the use of layered architecture (Controller → Service → Repository → Entity).
5. **Specifies cross-cutting concerns** — e.g. exception handling structure, configuration classes, security setup location.
 
---
 
## Output (Pydantic Schema: `ArchitectOutput`)
 
```json
{
  "package_structure": {
    "base_package": "com.example.orderservice",
    "packages": [
      "controller",
      "service",
      "service.impl",
      "repository",
      "entity",
      "dto",
      "dto.request",
      "dto.response",
      "exception",
      "config",
      "security"
    ]
  },
  "class_diagram_plantuml": "@startuml\nclass Order {\n  ...\n}\n@enduml",
  "sequence_diagram_plantuml": "@startuml\nClient -> OrderController: POST /orders\n...\n@enduml",
  "architectural_notes": "Layered architecture with Spring Boot..."
}
```
 
---
 
## PlantUML Rendering
The frontend receives the PlantUML strings and renders them as SVG images using the public `plantuml.com/plantuml/svg/` API with pako deflate compression encoding. If rendering fails, plain text is shown as fallback.
 
---
 
## Role in the Pipeline
The Architect output provides the **structural blueprint** for the Database Agent and Backend Layer Agent. Both agents use the package structure and class diagram to ensure their generated code fits correctly into the overall project layout.
 
---

## Key responsibilities Summary
| Responsibility | Description |
| Package structure | Defines how Java source files are organized|
| UML Class Diagram | Visualizes classes, fields, relationships |
| UML Sequence Diagram | Shows runtime interaction between components |
| Architectural patterns | Confirms layering and design conventions |

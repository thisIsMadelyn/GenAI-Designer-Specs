flowchart TD
    A[👤 User Input\nbusiness requirements] --> B

    B[🔍 Agent 1: Requirements Analyzer\nΕξάγει: entities, constraints, scale]
    B --> C

    C[🏗️ Agent 2: Architecture Designer\nΠαράγει: microservices + trade-off εξήγηση]
    C --> D

    D[🗄️ Agent 3: Database Modeler\nΠαράγει: MySQL CREATE TABLE schemas]
    D --> E

    E[🔌 Agent 4: API Designer\nΠαράγει: REST endpoints + Mermaid diagram]
    E --> F

    F[📦 Structured JSON Output\nόλα μαζί στο frontend]

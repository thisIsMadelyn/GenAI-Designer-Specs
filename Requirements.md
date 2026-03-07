User Input (business requirements)
         ↓
┌─────────────────────────┐
│  Agent 1: Requirements  │  → Εξάγει: entities, constraints, scale
│  Analyzer               │
└─────────────────────────┘
         ↓
┌─────────────────────────┐
│  Agent 2: Architecture  │  → Παράγει: microservices + trade-off εξήγηση
│  Designer               │
└─────────────────────────┘
         ↓
┌─────────────────────────┐
│  Agent 3: Database      │  → Παράγει: MySQL CREATE TABLE schemas
│  Modeler                │
└─────────────────────────┘
         ↓
┌─────────────────────────┐
│  Agent 4: API Designer  │  → Παράγει: REST endpoints + Mermaid diagram
└─────────────────────────┘
         ↓
   Structured JSON Output
   (όλα μαζί στο frontend)

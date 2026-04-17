# Project Analysis & Documentation

## 1. Overview

**Project Type:** Next.js + TypeScript dashboard for n8n workflow automation  
**Key Focus:** Backend API flows, n8n integration, configuration, and extensibility

---

## 2. API Route Mapping

| Route                         | Method | Purpose                                  | Calls                      | Data In         | Data Out          | Key Logic                      |
|-------------------------------|--------|------------------------------------------|----------------------------|-----------------|-------------------|-------------------------------|
| `/api/workflows`              | GET    | List workflows                           | MCPClient.listWorkflows    | active (query)  | workflows[]       | Filter by active               |
| `/api/workflows/[id]`         | GET    | Workflow details                         | MCPClient.getWorkflow      | id (path)       | workflow          |                               |
| `/api/workflows/[id]`         | POST   | Execute/activate/deactivate workflow     | MCPClient.executeWorkflow  | action, data    | result            | Switch on action               |
| `/api/executions`             | GET    | List executions                          | MCPClient.getExecutions    | workflowId,limit| executions[]      |                               |
| `/api/test-mcp`               | GET    | Test MCPClient/n8n connection            | MCPClient.testConnection   |                 | status, stats     |                               |
| `/api/system/health`          | GET    | System/n8n health check                  | MCPClient.testConnection   |                 | health            |                               |
| `/api/tools/call`             | POST   | Tooling endpoint (n8nClient, etc.)       | getN8nClient               | name, arguments | tool result       | Switch on tool name            |

---

## 3. Backend Workflow: Registration to Execution

- **User Registration:**  
  - No explicit registration endpoint found; likely handled via external auth or in a different service.

- **Workflow Listing:**  
  - `/api/workflows` → MCPClient → n8n API

- **Workflow Execution:**  
  - `/api/workflows/[id]` (POST, action: "execute") → MCPClient.executeWorkflow → n8n

- **Execution Logs:**  
  - `/api/executions` → MCPClient.getExecutions → n8n

---

## 4. n8n Integration

- **n8n URL & API Key Storage:**  
  - `.env.example` in backend server:  
    - `N8N_API_KEY` (API key for n8n, required)
    - n8n endpoint likely set in env or config (not yet found in frontend .env)
- **Retrieval:**  
  - MCPClient or n8nClient in backend reads from env/config at runtime
- **Invocation:**  
  - All workflow/execution/test calls routed through MCPClient, which builds HTTP requests to n8n endpoints
- **Payloads:**  
  - JSON, shaped by MCPClient methods
- **Auth:**  
  - API key (JWT) in header
- **Retry/Circuit-Breaker:**  
  - Not yet found; may be handled in MCPClient or HTTP client
- **Data Transformations:**  
  - MCPClient prepares payloads and parses responses

---

## 5. Configuration & Secrets Table

| Key            | Source           | Sample Value         | Used By                |
|----------------|------------------|---------------------|------------------------|
| N8N_API_KEY    | .env, .env.example | JWT token           | Backend MCPClient      |
| DEBUG          | .env, .env.example | true/false          | Backend logging        |
| ...            | ...              | ...                 | ...                    |

---

## 6. Dependency Graph (Backend)

- **API Route** → **MCPClient** → **n8n API**
- **API Route** → **getN8nClient** (for tool calls)
- **API Route** → **DeepSeekClient** (for AI/LLM endpoints)
- **API Route** → **Config/env** (for secrets, URLs, etc.)

---

## 7. Critical Path: User to n8n

1. User/API client sends HTTP request to API route (e.g., execute workflow)
2. API route instantiates MCPClient
3. MCPClient reads config/env for n8n URL & API key
4. MCPClient builds and sends HTTP request to n8n
5. n8n processes request, returns response
6. MCPClient parses response, API route returns to client

---

## 8. Extension Points & Coupling

- **n8n host change:** Update env/config, restart backend
- **API key change:** Update env/config, restart backend
- **Payload schema change:** Update MCPClient methods and API route logic
- **Auth method change:** Update MCPClient to support new auth

---

## 9. Top Architectural Decisions

1. All backend API calls routed through MCPClient abstraction
2. n8n integration isolated via API key and config
3. API routes are stateless and return JSON
4. Config/secrets externalized to env files
5. Extensible tool endpoint for advanced integrations

---

## 10. Top 5 Risk Areas

1. n8n API key/host change requires coordinated config update and restart
2. No retry/circuit-breaker logic found (potential for failed calls)
3. Tight coupling between MCPClient and n8n API schema
4. No registration endpoint (user management handled elsewhere?)
5. Secrets must be rotated securely and not leaked in logs

---

## 11. Project DNA (Elevator Pitch)

> “This project is a modern Next.js dashboard for managing and automating workflows via n8n, with a robust backend API layer abstracted through MCPClient. All workflow, execution, and integration logic is routed through secure, stateless API endpoints, with configuration and secrets externalized for flexibility and security.”

---

**This document can be expanded as the codebase evolves.**

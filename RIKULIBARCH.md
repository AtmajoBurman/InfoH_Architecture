```mermaid
%% Riku Library — High-level architecture
flowchart TD
  %% Users & Clients
  Browser[(User Browser / Client)]
  AdminBrowser[(Admin Browser)]

  %% Frontend
  subgraph Frontend [React SPA (Vite)]
    direction TB
    ReactApp[React 19 + Vite]
  end

  %% Backend
  subgraph Backend [API Layer (FastAPI)]
    direction TB
    FastAPI[FastAPI REST API]
    Auth[Auth Routes (/login, /logout)]
    BooksAPI[/books/* API Endpoints/]
    ChatAPI[/chatbot endpoint/]
    Templates[Jinja2 Admin Templates]
    PromMiddleware[Prometheus Middleware]
  end

  %% Chatbot / LLM
  subgraph Chatbot [LLM / LangChain Agent]
    direction TB
    Agent[LangChain Agent]
    LLM[Groq API (ChatGroq)]
    SQLTools[SQL Tools: list_tables / get_schema / execute_sql]
  end

  %% Database
  subgraph Database [PostgreSQL (schema: library)]
    direction TB
    Postgres[(Postgres DB)]
  end

  %% Observability
  subgraph Observability [Observability Stack (Docker-compose)]
    direction TB
    Prom[Prometheus]
    Graf[Grafana]
  end

  %% Arrows / Flows
  Browser -->|HTTP(S) (UI)| ReactApp
  AdminBrowser -->|HTTP(S) (UI)| ReactApp
  ReactApp -->|REST JSON| FastAPI
  ReactApp -.->|Dev proxy (vite) -> /books, /chatbot, /login| FastAPI

  FastAPI -->|includes routers| BooksAPI
  FastAPI --> Auth
  FastAPI --> Templates
  FastAPI --> ChatAPI
  FastAPI --> PromMiddleware
  PromMiddleware --> Prom

  ChatAPI -->|invokes| Agent
  Agent -->|calls| SQLTools
  SQLTools -->|read-only queries| Postgres
  Agent -->|LLM inference via API| LLM
  LLM -->|response| Agent
  Agent -->|synthesized answer| ChatAPI
  ChatAPI -->|JSON response| ReactApp

  BooksAPI -->|CRUD| Postgres
  Auth -->|JWT cookie / CSRF double-submit| ReactApp
  Templates -->|Admin UI (Jinja)| AdminBrowser

  Prom -->|metrics query| Graf
  Graf -->|dashboard| (Operator)
  

  %% Infrastructure bindings
  style Frontend fill:#f3f4f6,stroke:#6366f1,stroke-width:1px
  style Backend fill:#0ea5a4,stroke:#065f46,stroke-width:1px
  style Chatbot fill:#f59e0b,stroke:#92400e,stroke-width:1px
  style Database fill:#1473b6,stroke:#0b4a73,stroke-width:1px
  style Observability fill:#6b7280,stroke:#374151,stroke-width:1px
```

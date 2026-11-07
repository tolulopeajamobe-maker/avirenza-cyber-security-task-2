# Data Flow Diagram – Editable Source (Mermaid)

Copy the Mermaid block below into any Mermaid-compatible renderer (GitHub, VS Code, draw.io import, mermaid.live, etc.) to obtain a PNG/PDF.

```mermaid
flowchart TB
    %% Styles
    classDef untrusted fill:#ffcccc,stroke:#cc0000,stroke-width:2px
    classDef app fill:#cce5ff,stroke:#0066cc,stroke-width:2px
    classDef store fill:#ccffcc,stroke:#009900,stroke-width:2px
    classDef boundary fill:none,stroke:#ff6600,stroke-width:3px,stroke-dasharray: 5 5

    %% Actors & Elements
    User["👤 External User<br/>(Browser)"]:::untrusted
    FE["Frontend<br/>(Angular SPA)"]:::app
    API["API Layer<br/>(Express / REST)"]:::app
    Auth["Auth / JWT<br/>Service"]:::app
    DB[("🗄️ SQLite DB<br/>users, products,<br/>baskets, orders")]:::store
    Files[("📁 File Storage<br/>/ftp, uploads,<br/>backups")]:::store

    %% Trust Boundaries (visual groups)
    subgraph TB1["Trust Boundary 1 – Untrusted Client ↔ Application"]
        User
        FE
        API
    end

    subgraph TB2["Trust Boundary 2 – Application ↔ Data Stores"]
        Auth
        DB
        Files
    end

    %% Flows
    User -->|"HTTP(S) requests<br/>login, search, basket,<br/>checkout, feedback"| FE
    FE -->|"API calls<br/>JSON / tokens"| API
    API -->|"authenticate /<br/>validate JWT"| Auth
    API -->|"parameterised?<br/>queries"| DB
    API -->|"read / write<br/>files"| Files
    Auth -->|"user claims,<br/>roles"| API
    DB -->|"user data,<br/>orders"| API
    Files -->|"documents,<br/>images"| API

    %% Annotations
    note1["⚠️ All input crossing TB-1<br/>is untrusted"]
    note2["⚠️ Least privilege + param queries<br/>required at TB-2"]
```

## How the diagram maps to the fictional college fest ticketing site

- **Users** = students, staff, ticket buyers, administrators
- **Web Application** = the ticketing frontend + backend that handles registration, ticket selection, basket, payment, and order history
- **Database** = accounts, ticket inventory, orders, payment records
- **File storage** = uploaded ID documents, event posters, backup exports, receipts

Trust boundaries remain identical: anything coming from a browser is untrusted; the database and file store sit behind the application and must be protected by proper access controls and input handling.
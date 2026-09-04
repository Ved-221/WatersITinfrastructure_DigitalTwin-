# InfraTwin Project Context

## 1. What This Project Is

InfraTwin is an AI-powered digital twin for IT infrastructure.

A digital twin is a virtual representation of a real-world system. In this project, the represented system is an organization's IT environment: applications, servers, databases, networks, storage, cloud resources, authentication services, and the dependencies between them.

The main question InfraTwin answers is:

> If we migrate, modify, or lose this infrastructure component, what other components could be affected?

The project is modeled around Waters Corporation, a life-sciences and laboratory-technology company. The infrastructure data in this repository is representative demo data and is not Waters Corporation's real private infrastructure.

## 2. The Problem Being Solved

Enterprise IT systems are interconnected. An application may depend on a database, the database may depend on a server and storage system, and the server may depend on a network and firewall.

For example:

```text
Empower Application
        |
        +-- Empower Database
                |
                +-- Database Server
                        |
                        +-- Corporate WAN
                                |
                                +-- Core Firewall
```

If the database server fails, the database can fail. If the database fails, Empower can fail. Other applications depending on Empower may also be affected.

In a large organization, manually tracing all these relationships is difficult. InfraTwin stores the relationships and calculates the possible impact before a real infrastructure change is made.

## 3. High-Level Architecture

The project has three main layers:

```text
React Frontend
      |
      | HTTP REST requests
      v
FastAPI Backend
      |
      +-- SQLAlchemy
      |      |
      |      v
      |   SQLite Database
      |
      +-- NetworkX Simulation Engine
      |
      +-- AI / Multi-Agent Analysis
      |
      +-- AWS Integrations
```

### Frontend

The frontend is a React and TypeScript application built with Vite.

Main file: `frontend/src/App.tsx`

It provides:

- Environment selection
- Infrastructure graph visualization
- Dashboard statistics
- Manual infrastructure editing
- Dependency creation
- Migration simulation
- Risk and cost results
- AI architect recommendations

The graph is rendered using `@xyflow/react`, a React Flow library.

### Backend

The backend is a Python FastAPI application.

Main file: `backend/main.py`

It provides REST APIs for:

- Reading infrastructure components
- Reading dependencies
- Reading statistics
- Running simulations
- Creating manual projects
- Adding, editing, and deleting resources
- Creating and deleting dependencies
- Reading AWS health and compliance information

### Database

The application uses SQLite through SQLAlchemy.

Database configuration is in `backend/database.py`.

The local database file is `backend/infratwin.db`. It is created when the backend runs. SQLAlchemy maps Python model classes to database tables.

## 4. How Infrastructure Is Represented

The digital twin is a directed graph.

A graph contains nodes and edges. In this application:

| Graph concept | Project concept |
|---|---|
| Node | Component |
| Edge | Dependency |
| Node property | Type, cost, owner, criticality, environment |
| Edge property | Relationship type and criticality |

### Components

Components are defined in `backend/models.py`.

A component represents one infrastructure object, such as:

- `Empower (Lab Informatics)`
- `ERP System`
- `Empower DB`
- `Corporate WAN`
- `AWS EC2 Cluster`
- `AWS S3 Data Lake`
- `Authentication Service (SSO)`

Each component has properties such as:

```text
id
name
type
environment
location
criticality
owner
status
cpu
memory
cost_per_month
metadata_col
source_environment
```

Possible component types include:

- `application`
- `server`
- `database`
- `network`
- `cloud_resource`
- `storage`
- `api`
- `identity`
- `k8s_node`
- `k8s_pod`

Possible environments include:

- `on_prem`
- `cloud`
- `kubernetes`
- `hybrid`

Possible criticality levels include:

- `low`
- `medium`
- `high`
- `critical`

### Dependencies

A dependency describes how one component relies on another.

For example:

```text
app_empower -> db_empower
```

This means that the Empower application depends on the Empower database.

The supported relationship types are:

- `depends_on`
- `connects_to`
- `stores_in`
- `authenticates_via`
- `hosted_on`

The direction is important. If the graph contains:

```text
Application -> Database
```

then the application is the dependent component and the database is the component it relies on.

Dependencies are stored in the `dependencies` table using `source_id` and `target_id` foreign keys that point to components.

## 5. Initial Demo Data

The initial infrastructure is created by `backend/seed.py`.

The seed data contains:

- 5 applications
- 3 databases
- 4 servers
- 2 network components
- 2 cloud resources
- 1 storage component
- 1 identity component

Examples include:

```text
Empower
ERP System
CRM
Reporting/Analytics
Lab Data Portal
Empower DB
ERP DB
Analytics DB
Corporate WAN
Core Firewall
Primary SAN
Authentication Service
AWS EC2 Cluster
AWS S3 Data Lake
```

The seed process runs when FastAPI starts. It checks whether components already exist. If the database is empty, it inserts the demo components and dependencies.

## 6. What Happens When the Backend Starts

When this command is run from the `backend` directory:

```bash
python main.py
```

these steps occur:

1. FastAPI creates the application.
2. SQLAlchemy runs `Base.metadata.create_all()`.
3. Missing database tables are created.
4. FastAPI starts its startup handler.
5. The startup handler opens a database session.
6. `seed.seed_data()` checks whether data already exists.
7. If the database is empty, demo data is inserted.
8. Uvicorn starts the API server on port `8000`.

The backend is available at:

```text
http://localhost:8000
```

FastAPI's interactive API documentation is available at:

```text
http://localhost:8000/docs
```

### Important Database Detail

`create_all()` creates missing tables, but it does not migrate existing tables. If an old database was created before a new column was added, the old table remains unchanged.

That is why an old database can produce an error such as:

```text
no such column: components.source_environment
```

For a disposable local demo database, the fix is:

```bash
cd backend
rm -f infratwin.db
python main.py
```

This deletes local data and lets the application create a new database using the current models.

## 7. What Happens When the Frontend Starts

When these commands are run from the `frontend` directory:

```bash
npm install
npm run dev
```

Vite starts the React development server, normally at:

```text
http://localhost:5173
```

The frontend uses this backend URL:

```typescript
const API_BASE = 'http://localhost:8000/api';
```

Therefore, both processes must be running:

```text
Frontend: http://localhost:5173
Backend:  http://localhost:8000
```

The browser loads the interface from Vite. The React application then makes HTTP requests to FastAPI to read and modify data.

## 8. Environment Selection

When the website opens, the user chooses an environment.

### AWS Environment

The AWS option represents infrastructure discovered from an AWS account.

It can include:

- AWS components
- AWS resource health
- CloudWatch metrics
- AWS Config compliance information
- AWS cost information
- Simulation sandboxes

The live AWS environment is intended to be read-only from the frontend.

### Manual Environment

The manual option lets the user build a custom digital twin without connecting to AWS.

The user can:

- Create a named project
- Add resources
- Edit resource properties
- Delete resources
- Connect resources with dependencies
- Save multiple dashboards
- Deploy a manual project into an AWS simulation sandbox

Manual projects are stored in the `manual_projects` table.

## 9. Loading the Infrastructure Graph

When an environment is selected, the frontend requests:

```text
GET /api/twin/components
GET /api/twin/dependencies
GET /api/twin/stats
```

These endpoints are implemented in `backend/main.py`.

The frontend converts the returned data into React Flow objects:

```text
Component  -> React Flow node
Dependency -> React Flow edge
```

Each node displays:

- Icon
- Name
- Component type
- Criticality
- Monthly cost
- Environment
- Manual, planned, or AWS status

Each edge displays a relationship such as:

```text
depends on
connects to
hosted on
stores in
authenticates via
```

## 10. How Simulation Works

The simulation logic is in `backend/simulation.py`.

When the user selects a component and starts a migration simulation, the frontend sends a request like this:

```text
POST /api/simulate
```

with a body similar to:

```json
{
  "target_component_id": "db_empower",
  "action": "migrate",
  "destination_env": "cloud",
  "use_ai": true
}
```

The backend then:

1. Loads all components from SQLite.
2. Loads all dependencies from SQLite.
3. Builds a `networkx.DiGraph` in memory.
4. Finds the selected component.
5. Finds all components that depend on the selected component.
6. Finds all components that the selected component depends on.
7. Combines these into the affected set.
8. Calculates the risk score.
9. Estimates downtime.
10. Estimates the monthly cost change.
11. Optionally runs AI analysis.
12. Returns the result to the frontend.

The simulation does not modify the real infrastructure. It calculates a hypothetical result using the digital twin data.

## 11. Blast Radius Calculation

The blast radius means the set of components potentially affected by a change.

The engine uses two NetworkX operations:

```python
nx.ancestors(graph, target_component_id)
nx.descendants(graph, target_component_id)
```

For a component `B`:

- Ancestors are components that depend on `B`.
- Descendants are components that `B` depends on.

Both groups are included in the current affected set.

For example, if the target is `db_empower`, the affected components may include:

```text
Empower Application
Empower DB
On-prem DB Server
Primary SAN
Corporate WAN
Core Firewall
Lab Data Portal
```

The frontend highlights affected nodes with a red border and warning icon.

## 12. Risk Score

The risk score is calculated by rules in `backend/simulation.py`.

The score begins with the criticality of the selected component:

```text
Critical component: +40
High component:     +20
Medium component:    +0
Low component:       +0
```

The engine then adds:

```text
+10 for every affected high or critical component
Maximum: +40

+15 for every cross-environment risk
Maximum: +20
```

The final score is capped at `100`.

The result is classified as:

```text
0-25    LOW
26-50   MEDIUM
51-75   HIGH
76-100  CRITICAL
```

The engine also generates critical flags, for example:

```text
Cross-environment dependency with Corporate WAN (hybrid)
```

## 13. Downtime Estimate

Downtime is a heuristic estimate, not a real operational measurement.

Base downtime is selected according to the target type:

```text
Database or storage: 120 minutes
Network or identity:  60 minutes
Other component:       30 minutes
```

The engine then adds:

```text
15 minutes for every affected component
```

Consequently, migrating a database usually produces a larger estimate than migrating an ordinary application.

## 14. Cost Estimate

The simulation uses the target component's `cost_per_month` value.

If that value is missing or zero, fallback values are used:

```text
Database or storage: $850
Server:              $250
Application:         $500
Other:               $100
```

For a migration to the cloud, the current heuristic is:

```text
cost delta = base cost * 15%
```

This is an estimated cloud migration increase. It is not a complete AWS billing calculation unless AWS Cost Explorer data has been synchronized.

## 15. AI and Multi-Agent Analysis

The repository contains two AI-related files:

- `backend/ai_engine.py`
- `backend/multi_agent_system.py`

The current simulation path calls `multi_agent_system.py`.

The multi-agent layer can use one of these providers:

1. OpenRouter
2. Groq
3. OpenAI

The first available key determines the provider:

```text
OPENROUTER_API_KEY
GROQ_API_KEY
OPENAI_API_KEY
```

The analysis has three roles.

### Financial Analyst

Examines:

- Monthly cost delta
- Cloud migration penalty
- Whether the projected cost is favorable

### Risk Analyst

Examines:

- Risk score
- Estimated downtime
- Number of affected components
- Cross-environment dependencies
- Critical flags

### Cloud Architect

Receives the financial and risk reports and generates:

- Executive explanation
- Recommended architectural actions

The frontend displays these results in the node details sidebar.

### Gemini File

`backend/ai_engine.py` contains a separate Gemini-based explanation function. It reads `GEMINI_API_KEY`, sends the structured simulation result to Gemini, and expects an object containing `explanation` and `recommendation`.

However, the current `simulation.py` implementation imports and calls `multi_agent_system`, so the Gemini function is not the main active simulation path at present.

## 16. AWS Integrations

AWS discovery and synchronization are implemented through files such as:

- `backend/aws_sync.py`
- `backend/aws_mapper.py`
- `backend/aws_database_sync.py`
- `backend/aws_sync_all.py`

The intended discovery flow is:

```text
AWS account
   |
   v
AWS Config discovery
   |
   v
Normalize AWS resources
   |
   v
Map resources to Component records
   |
   v
Store components and dependencies in SQLite
```

The backend also contains integrations for:

- CloudWatch health metrics
- AWS Config compliance
- Cost Explorer costs

CloudWatch behavior is implemented in `backend/cloudwatch_service.py`.

When an AWS node is selected, the frontend requests:

```text
GET /api/twin/health/{component_id}
GET /api/twin/compliance/{component_id}
```

These features require valid AWS credentials, permissions, regions, and AWS services with discoverable data.

## 17. Manual Project Workflow

A typical manual workflow is:

1. Select **Manual Environment**.
2. Create a new dashboard.
3. Add components such as applications, databases, or servers.
4. Connect components with dependency edges.
5. Select a component.
6. Run a migration simulation.
7. Review the affected nodes, risk, downtime, and cost.
8. Deploy the design to an AWS simulation sandbox if required.

Manual resources are editable. Live AWS resources are treated as read-only.

The manual forms are implemented in:

- `frontend/src/components/ResourceFormModal.tsx`
- `frontend/src/components/DependencyFormModal.tsx`

The related API routes are in `backend/main.py`.

## 18. Main API Endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/twin/components` | GET | Retrieve components for an environment |
| `/api/twin/dependencies` | GET | Retrieve dependency edges |
| `/api/twin/stats` | GET | Retrieve dashboard statistics |
| `/api/twin/health/{component_id}` | GET | Retrieve AWS health information |
| `/api/twin/compliance/{component_id}` | GET | Retrieve AWS compliance information |
| `/api/simulate` | POST | Run a migration impact simulation |
| `/api/manual/projects` | GET | List manual projects |
| `/api/manual/projects` | POST | Create a manual project |
| `/api/manual/projects/{project_id}` | DELETE | Delete a project and its resources |
| `/api/manual/components` | POST | Add a manual component |
| `/api/manual/components/{component_id}` | PUT | Update a manual component |
| `/api/manual/components/{component_id}` | DELETE | Delete a manual component |
| `/api/manual/dependencies` | POST | Add a manual dependency |
| `/api/manual/dependencies/{dependency_id}` | DELETE | Delete a manual dependency |
| `/api/manual/projects/{project_id}/push-to-aws` | POST | Copy a manual project into a simulation sandbox |

## 19. What the Project Actually Is Today

The implemented product is best described as:

> A local React dashboard backed by FastAPI and SQLite that models IT infrastructure as a dependency graph, calculates migration impact using NetworkX, and optionally generates AI-based financial, risk, and architecture recommendations.

The strongest implemented features are:

- Infrastructure graph visualization
- Seeded demo infrastructure
- Manual project creation
- Dependency modeling
- Migration impact analysis
- Blast-radius highlighting
- Risk calculation
- Downtime estimation
- Cloud cost estimation
- AI agent reports
- AWS integration foundations

Some capabilities are foundations or future extensions rather than complete implementations:

- Kubernetes-specific simulation
- Automated single-point-of-failure detection
- Full natural-language infrastructure queries
- Persistent simulation history
- Real-time monitoring for all component types
- Full production-grade database migrations
- Complete AWS deployment of infrastructure

## 20. Important Implementation Limitations

### Simulation Direction

The project assumes that a dependency is stored as:

```text
source component -> target component
```

where the source relies on the target. This convention must be followed when adding dependencies manually.

### Current State Is Not Used in Risk Math

Components have a status such as `active`, `degraded`, or `offline`, but the current migration simulation does not use that status in its calculation.

### Risk Is Heuristic-Based

The score is deterministic and useful for demonstration, but it is not a production risk model. It does not use historical incidents, real traffic, service-level objectives, or actual migration telemetry.

### Downtime Is Estimated

The downtime values are fixed heuristics plus an affected-component adjustment. They should not be treated as an operational guarantee.

### AI Is Optional and External

AI analysis requires a valid provider key and network access. Without a key, the backend returns an unavailable message for the agent reports.

### AWS Features Need Credentials

AWS discovery, CloudWatch, Config, and Cost Explorer require local AWS authentication and appropriate IAM permissions.

### Database Reset Loses Local Data

Deleting `infratwin.db` fixes schema problems for a demo, but it permanently removes locally stored manual projects and synchronized AWS records.

## 21. End-to-End User Flow

The full workflow is:

```text
1. User opens the React application.
2. User selects AWS or Manual Environment.
3. Frontend asks FastAPI for components, dependencies, and statistics.
4. FastAPI reads the data from SQLite.
5. Frontend renders the graph and dashboard.
6. User selects a component.
7. User starts a migration simulation.
8. Backend builds a NetworkX dependency graph.
9. Backend traverses dependents and dependencies.
10. Backend calculates blast radius, risk, downtime, and cost.
11. Backend optionally runs financial, risk, and architect AI analysis.
12. Backend returns a structured JSON result.
13. Frontend highlights affected components.
14. Frontend displays the migration decision and recommendations.
```

## 22. Core Design Principle

The project follows this sequence:

```text
Digital Twin Data
       |
       v
Deterministic Simulation Engine
       |
       v
AI Explanation and Recommendation
```

The AI does not create the infrastructure model. The database and graph engine are the source of truth. AI sits on top to explain the calculated result in language that an infrastructure leader or executive can understand.

## 23. Key Files

| File | Responsibility |
|---|---|
| `backend/main.py` | FastAPI application and API routes |
| `backend/database.py` | SQLite and SQLAlchemy setup |
| `backend/models.py` | Database models and enums |
| `backend/schemas.py` | API request and response schemas |
| `backend/seed.py` | Initial demo components and dependencies |
| `backend/simulation.py` | Graph building, blast radius, risk, downtime, and cost calculations |
| `backend/ai_engine.py` | Gemini explanation function |
| `backend/multi_agent_system.py` | Financial, risk, and architect AI analysis |
| `backend/aws_sync.py` | AWS resource discovery support |
| `backend/aws_mapper.py` | AWS resource-to-component mapping |
| `backend/aws_database_sync.py` | Persist AWS data to SQLite |
| `backend/cloudwatch_service.py` | AWS CloudWatch metrics and health |
| `backend/config_rules_service.py` | AWS Config compliance support |
| `backend/cost_explorer_service.py` | AWS cost retrieval support |
| `frontend/src/App.tsx` | Main React interface and user workflow |
| `frontend/src/components/ResourceFormModal.tsx` | Manual resource form |
| `frontend/src/components/DependencyFormModal.tsx` | Manual dependency form |
| `frontend/package.json` | Frontend scripts and dependencies |

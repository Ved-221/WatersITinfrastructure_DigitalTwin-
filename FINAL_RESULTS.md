# InfraTwin Final Results

## 1. Project Overview

InfraTwin is an AI-assisted digital twin for enterprise IT infrastructure. It creates a virtual model of applications, servers, databases, networks, storage, cloud resources, and authentication services. Users can explore the model and simulate a proposed infrastructure migration before making changes to a real environment.

The project is designed around a representative Waters Corporation-style hybrid IT environment. The included infrastructure is demo data and does not represent Waters Corporation's private production systems.

The central question answered by the system is:

> What could be affected if this infrastructure component is migrated or changed?

## 2. Final Product Outcome

The completed application provides a local web platform that:

- Represents IT infrastructure as a directed dependency graph.
- Displays components and relationships visually.
- Supports seeded demo infrastructure.
- Allows users to create custom manual infrastructure projects.
- Allows users to add, edit, delete, and connect manual resources.
- Calculates the potential blast radius of a migration.
- Calculates a deterministic risk score.
- Estimates downtime.
- Estimates monthly cost impact.
- Highlights affected components visually.
- Provides AI-based financial, risk, and architecture recommendations when configured.
- Includes foundations for AWS discovery, health monitoring, compliance, and cost synchronization.

## 3. Technology Stack

### Frontend

- React
- TypeScript
- Vite
- `@xyflow/react` for graph visualization
- Tailwind CSS
- `lucide-react` for icons

### Backend

- Python
- FastAPI
- Uvicorn
- SQLAlchemy
- SQLite
- NetworkX
- Pydantic
- OpenAI-compatible AI providers
- Google Generative AI support

### Integrations

- AWS Config resource discovery
- AWS resource mapping
- AWS CloudWatch metrics
- AWS Config compliance checks
- AWS Cost Explorer
- OpenRouter, Groq, or OpenAI for AI analysis

## 4. Architecture Result

The final architecture follows this flow:

```text
User
  |
  v
React + TypeScript Frontend
  |
  | REST API requests
  v
FastAPI Backend
  |
  +--> SQLAlchemy ORM --> SQLite database
  |
  +--> NetworkX graph simulation
  |
  +--> AI analysis pipeline
  |
  +--> AWS service integrations
```

The frontend runs on port `5173` and the backend runs on port `8000`.

```text
Frontend: http://localhost:5173
Backend:  http://localhost:8000
API docs: http://localhost:8000/docs
```

## 5. Digital Twin Data Model

The digital twin is stored using two main tables.

### Components

The `components` table stores infrastructure nodes. Each component can contain:

- Unique ID
- Name
- Type
- Environment
- Location
- Criticality
- Owner
- Operational status
- CPU allocation
- Memory allocation
- Monthly cost
- Metadata
- Source environment

Supported component types include:

- Application
- Server
- Database
- Network
- Cloud resource
- Storage
- API
- Identity
- Kubernetes node
- Kubernetes pod

### Dependencies

The `dependencies` table stores directed edges between components. Each dependency contains:

- Source component
- Target component
- Relationship type
- Dependency criticality
- Source environment

Supported relationships include:

- `depends_on`
- `connects_to`
- `stores_in`
- `authenticates_via`
- `hosted_on`

A dependency such as:

```text
Empower Application -> Empower DB
```

means that the application relies on the database.

## 6. Seeded Infrastructure Result

The demo environment contains 17 representative infrastructure components:

| Type | Count |
|---|---:|
| Applications | 5 |
| Databases | 3 |
| Servers | 4 |
| Networks | 2 |
| Cloud resources | 2 |
| Storage | 1 |
| Identity services | 1 |
| **Total** | **17** |

Important seeded components include:

- Empower laboratory application
- ERP system
- CRM
- Reporting and analytics
- Lab data portal
- Empower database
- ERP database
- Analytics database
- On-premises application servers
- On-premises database server
- Corporate WAN
- Core firewall
- Authentication service
- AWS EC2 cluster
- AWS S3 data lake
- Primary SAN storage
- Laboratory instrument gateway

The seed data is inserted automatically when the database is empty.

## 7. User Workflow

The final user workflow is:

1. Open the React application.
2. Choose AWS Environment or Manual Environment.
3. Load the infrastructure graph and dashboard statistics.
4. Select a component on the graph.
5. Review its type, environment, owner, criticality, cost, and dependencies.
6. Start a migration simulation.
7. The backend builds a graph from the database.
8. The simulation engine finds dependent and required components.
9. The engine calculates blast radius, risk, downtime, and cost impact.
10. AI agents analyze the structured result when configured.
11. The frontend highlights affected components.
12. The sidebar displays the migration decision and recommended actions.

## 8. Simulation Results

The simulation endpoint is:

```text
POST /api/simulate
```

A request contains:

```json
{
  "target_component_id": "db_empower",
  "action": "migrate",
  "destination_env": "cloud",
  "use_ai": true
}
```

The response contains:

- Requested action
- Target component
- Destination environment
- Number of affected components
- Affected component IDs
- Risk score
- Risk level
- Estimated downtime
- Monthly cost delta
- Critical flags
- AI financial analysis
- AI risk analysis
- AI architect recommendation
- Recommended actions

## 9. Blast Radius Calculation

The simulation engine uses NetworkX to traverse the directed graph.

It finds:

- Ancestors: components that depend on the target.
- Descendants: components that the target depends on.

Both groups are included in the affected set. This allows the system to show both sides of a change:

- What may break because of the target.
- What the target itself requires to work.

Affected nodes are returned to the frontend and highlighted with a red border and warning icon.

## 10. Risk Calculation

The risk score is deterministic and based on the infrastructure graph.

Base score:

```text
Critical target: +40
High target:     +20
Medium target:    +0
Low target:       +0
```

Additional score:

```text
+10 for every affected high or critical component
Maximum: +40

+15 for every cross-environment risk during migration
Maximum: +20
```

The final score is capped at 100.

Risk levels are:

```text
0-25    LOW
26-50   MEDIUM
51-75   HIGH
76-100  CRITICAL
```

The system also records critical flags for cross-environment relationships.

## 11. Downtime and Cost Results

Downtime is estimated using a transparent heuristic.

Base downtime:

```text
Database or storage: 120 minutes
Network or identity:  60 minutes
Other components:      30 minutes
```

The engine adds 15 minutes for every affected component.

Cost impact uses the target component's monthly cost. If no cost is available, fallback values are used. For a migration to the cloud, the current estimate applies a 15% increase to the base cost.

These values are planning estimates, not production guarantees or final AWS billing calculations.

## 12. AI Results

The active AI flow is implemented in `multi_agent_system.py`.

It uses the structured simulation output as input and creates three perspectives:

### Financial Analyst

Reviews the monthly cost delta and cloud migration cost impact.

### Risk Analyst

Reviews risk score, downtime, affected count, and critical flags.

### Cloud Architect

Combines the financial and risk reports and produces:

- Executive explanation
- Recommended architectural changes
- A practical migration direction

The system can use OpenRouter, Groq, or OpenAI depending on which environment variable is configured:

```text
OPENROUTER_API_KEY
GROQ_API_KEY
OPENAI_API_KEY
```

The repository also contains a Gemini-based explanation implementation in `ai_engine.py`. The current simulation function primarily calls the multi-agent system instead.

## 13. Manual Infrastructure Builder

The manual environment allows users to build their own digital twin.

Users can:

- Create named dashboards.
- Add applications, servers, databases, and other resources.
- Edit component properties.
- Delete components.
- Create dependencies through forms.
- Connect nodes directly in the graph.
- Delete manual dependencies.
- Push a manual architecture to an AWS simulation sandbox.

Manual resources are intended for safe design experimentation. They do not modify real production infrastructure.

## 14. Dashboard Result

The dashboard displays:

- Total components
- Critical service count
- Total monthly run rate
- On-premises component count
- Cloud component count

The topology view displays:

- Component nodes
- Relationship edges
- Component icons
- Criticality labels
- Monthly costs
- Graph controls
- Dependency labels
- Blast-radius highlighting

The node details sidebar displays:

- Actual component state
- Resource properties
- Inbound dependencies
- Simulation controls
- Migration decision
- Risk level
- Downtime
- Cost change
- AI recommendation
- Recommended actions

## 15. AWS Capability Result

The project includes an AWS synchronization foundation that can:

1. Discover AWS resources through AWS Config.
2. Normalize discovered resource data.
3. Map AWS resources to digital twin components.
4. Extract dependencies.
5. Store components and relationships in SQLite.
6. Retrieve CloudWatch metrics for supported resources.
7. Retrieve compliance information.
8. Retrieve current service cost data through Cost Explorer.

AWS functionality requires valid AWS credentials, permissions, regions, and available AWS resources. Without those prerequisites, the local manual and seeded environments remain usable.

## 16. API Result

The main API surface is:

| Endpoint | Method | Result |
|---|---|---|
| `/api/twin/components` | GET | Returns graph components |
| `/api/twin/dependencies` | GET | Returns graph relationships |
| `/api/twin/stats` | GET | Returns dashboard metrics |
| `/api/twin/health/{component_id}` | GET | Returns AWS health data |
| `/api/twin/compliance/{component_id}` | GET | Returns AWS compliance data |
| `/api/simulate` | POST | Runs migration impact analysis |
| `/api/manual/projects` | GET/POST | Lists or creates manual projects |
| `/api/manual/projects/{project_id}` | DELETE | Deletes a manual project |
| `/api/manual/components` | POST | Creates a manual component |
| `/api/manual/components/{component_id}` | PUT | Updates a manual component |
| `/api/manual/components/{component_id}` | DELETE | Deletes a manual component |
| `/api/manual/dependencies` | POST | Creates a manual dependency |
| `/api/manual/dependencies/{dependency_id}` | DELETE | Deletes a manual dependency |
| `/api/manual/projects/{project_id}/push-to-aws` | POST | Creates a simulation sandbox copy |

## 17. What Was Successfully Demonstrated

The project demonstrates the complete core concept:

```text
Infrastructure model
        |
        v
Dependency graph
        |
        v
What-if migration simulation
        |
        v
Blast-radius and risk calculation
        |
        v
AI explanation and recommendation
```

This converts infrastructure planning from a manual dependency search into a repeatable scenario-analysis workflow.

## 18. Known Limitations

The following limitations remain in the current implementation:

- The risk model is heuristic-based.
- Downtime is estimated rather than measured from historical data.
- The current simulation focuses primarily on migration scenarios.
- Component status is stored but is not currently included in simulation scoring.
- Kubernetes node and pod types exist in the model but are not present in the seed data.
- Automated SPOF detection is not fully implemented.
- Natural-language query support is not fully implemented.
- Simulation history is not persisted as a dedicated table.
- AWS functionality depends on external credentials and permissions.
- Existing SQLite tables are not automatically migrated when models change.
- AI results depend on external providers and API availability.

## 19. Recommended Future Improvements

For a production-ready version, the next improvements would be:

1. Add Alembic database migrations.
2. Store simulation history and audit logs.
3. Add authentication and role-based access control.
4. Use real monitoring history in risk calculations.
5. Add redundancy and SPOF analysis.
6. Add proper scenario comparison.
7. Add Kubernetes and OpenShift modeling.
8. Add rollback and disaster-recovery simulations.
9. Add natural-language scenario input.
10. Add exportable change-impact reports.
11. Add automated tests for graph traversal and risk calculations.
12. Add production deployment configuration.

## 20. Final Conclusion

InfraTwin successfully implements an AI-assisted digital twin concept for IT infrastructure.

Its most important achievement is the separation between factual infrastructure modeling and AI explanation:

```text
Database and dependency graph = source of truth
Simulation engine              = deterministic impact calculation
AI layer                       = explanation and recommendation
```

The result is a working local platform that allows an infrastructure team to model systems, explore dependencies, simulate migration risk, understand blast radius, estimate cost and downtime, and receive decision support before changing real infrastructure.

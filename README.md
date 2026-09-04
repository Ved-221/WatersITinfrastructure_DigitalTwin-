# InfraTwin: AI-Powered Digital Twin for IT Infrastructure

## 🚀 Overview

**InfraTwin** is an AI-assisted digital twin for enterprise IT infrastructure. Modeled conceptually around a representative hybrid IT environment (similar to Waters Corporation), it creates a virtual, interactive model of applications, servers, databases, networks, cloud resources, and authentication services. 

The core problem InfraTwin solves is answering the critical question:
> *"If I change this piece of infrastructure, what else breaks?"*

By allowing IT leaders to simulate "what-if" scenarios—such as cloud migrations, network upgrades, or component failures—InfraTwin predicts the **blast radius, risk score, estimated downtime, and cost impact** *before* any real-world changes are executed. It then leverages an AI layer to provide plain-English explanations and strategic recommendations based on the simulated data.

---

## ✨ Key Features

- 🕸️ **Interactive Dependency Graph:** Visualize your entire IT infrastructure (on-prem, cloud, and hybrid) as a directed node-and-edge graph.
- 🔮 **What-If Simulation Engine:** Propose a change (e.g., "Migrate Database to AWS") and instantly calculate the cascading impact using deterministic graph traversal.
- 💥 **Blast Radius Analysis:** Visually highlights affected upstream and downstream components with severity indicators.
- 📊 **Executive Dashboard:** Get a high-level view of infrastructure health, critical services, single points of failure (SPOF), and total monthly run rate.
- 🤖 **AI Impact Explanation & Recommendations:** Translates complex simulation results into clear, actionable AI-driven insights (e.g., Financial, Risk, and Architectural analysis) without hallucinating infrastructure facts.
- 🛠️ **Manual Infrastructure Builder:** Safely experiment by creating custom dashboards, adding components, and mapping dependencies manually.

---

## 🛠️ Technology Stack

InfraTwin is built using a modern, fast, and explainable tech stack:

### **Frontend**
- **React + TypeScript** (built with Vite)
- **Tailwind CSS** for styling
- **React Flow (`@xyflow/react`)** for interactive graph visualization
- **Recharts** for dashboard analytics

### **Backend**
- **Python + FastAPI** for high-performance REST APIs
- **NetworkX** for in-memory graph traversal and simulation logic
- **SQLite + SQLAlchemy ORM** as the primary data store
- **Pydantic** for data validation

### **AI & Integrations**
- **LLM APIs** (OpenAI, OpenRouter, Groq, or Google Gemini) for the explanation layer
- **AWS Integration Foundations** (AWS Config, CloudWatch, Cost Explorer) for potential live-environment synchronization

---

## ⚙️ Getting Started

To run InfraTwin locally, you will need to start both the backend and frontend servers.

### 1. Start the Backend
The backend runs on Python and FastAPI. It automatically seeds demo infrastructure data into a local SQLite database (`infratwin.db`) on startup.

```bash
cd backend
# Create and activate a virtual environment (recommended)
# pip install -r requirements.txt
python main.py
```
- The backend API will be available at: `http://localhost:8000`
- Interactive API documentation: `http://localhost:8000/docs`

*Note: If you need to reset the database to the default seed state, simply delete `infratwin.db` and run `python main.py` again.*

### 2. Start the Frontend
The frontend is a React application served via Vite.

```bash
cd frontend
npm install
npm run dev
```
- The frontend dashboard will be available at: `http://localhost:5173`

*(Ensure the backend is running so the frontend can successfully fetch infrastructure data).*

---

## 🧠 How the AI / Simulation Works

InfraTwin strictly separates factual infrastructure data from AI generation to ensure accuracy:

1. **Digital Twin Data (Source of Truth):** Components and relationships are stored in the SQLite database.
2. **Simulation Engine (Deterministic):** When a change is proposed, Python's `NetworkX` traverses the graph to mathematically determine the affected components, heuristic risk score, and cost/downtime estimates.
3. **AI Explanation Layer:** The structured JSON output from the simulation is fed into an LLM, which explains *why* the impact happened and recommends a migration strategy (e.g., phased vs. direct migration). 

---

## ⚠️ Important Limitations (MVP)

- The risk models, cost increases (e.g., 15% cloud premium), and downtime metrics are currently based on deterministic heuristics rather than live operational telemetry.
- AWS integration features require valid local AWS credentials and permissions to function.
- AI analysis features require a valid API key set in your environment variables (e.g., `OPENAI_API_KEY` or `GEMINI_API_KEY`). If no key is present, the AI reports will show as unavailable.

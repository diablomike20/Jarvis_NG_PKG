# J.A.R.V.I.S Personal Knowledge Graph

[![CI](https://github.com/aceFelix/acefelix/actions/workflows/ci.yml/badge.svg)](https://github.com/aceFelix/acefelix/actions/workflows/ci.yml)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.11%20%7C%203.12%20%7C%203.13%20%7C%203.14-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Node](https://img.shields.io/badge/Node.js-20-339933?logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110%2B-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Vue](https://img.shields.io/badge/Vue.js-3-42b883?logo=vuedotjs&logoColor=white)](https://vuejs.org/)
[![MCP](https://img.shields.io/badge/MCP-Server-5c6bc0)](https://modelcontextprotocol.io/)

A personal knowledge graph system built around manually maintained data. It supports creating, reading, updating, and deleting entities and relationships, graph structure queries, and a universe- and galaxy-themed 3D visualization.

<p align="center">
  <img src="assets/zhishitupu.png" alt="AceFelix Knowledge Graph: universe- and galaxy-themed 3D visualization" width="90%">
  <br>
  <sub>Entities as planets: gas cloud bands or land-and-ocean terrain with atmospheric glow, surrounded by a starfield, spiral galaxy arms, and nebulae.</sub>
</p>

## Features

- **Knowledge graph core**: NetworkX graph engine with JSON persistence; full CRUD operations for entities and relationships.
- **Automatic GraphRAG extraction**: Supply text, documents, or conversation logs and an LLM extracts entities and relationships into the graph. Five noise-prevention gates keep casual chat from being written to the graph. Supports `dry_run` previews and duplicate detection.
- **3D visualization**: Force-directed 3D graph powered by 3d-force-graph (Three.js). Nodes are procedurally generated **realistic planets** (gas bands and storm spots, or oceans, continents, and craters), with atmospheric glow and slow rotation. The background includes stars, spiral galaxy arms, nebulae, and black holes.
- **Dynamic types**: Create, update, delete, recolor, and rename entity and relationship types. Renaming propagates to related records; Chinese labels are supported and deletion is safeguarded.
- **Graph queries**: Neighbor subgraphs, relationship paths between two entities, shared neighbors, and search.
- **Interactive modes**: Click to highlight one-hop relationships, focus to explore two hops, query paths, and reset the view with one action.
- **Entity management**: List view, type filtering, name search, and CRUD operations. Properties support **image and document (.pdf/.md/.txt/.docx/.xmind) references via URLs and local uploads**.
- **Data protection**: Optimistic locking prevents concurrent changes from overwriting each other. Each save creates a rolling backup (up to 20 retained), and atomic writes protect against corruption.
- **Statistics**: Counts of entities and relationships, plus type distribution.
- **Agent integration**: An MCP server exposes the graph through a standard protocol to AI agents (jarvis, Claude Desktop, etc.). Agents can query profiles, search entities, explore connections, and add entities or relationships with user confirmation.

## Project Structure

```text
acefelix/
├── backend/                  # Python backend (see backend/README.md)
│   ├── api.py                # Entry point: FastAPI REST service (image uploads, knowledge extraction, static assets)
│   ├── mcp_server.py         # Entry point: MCP server for agents (stdio)
│   ├── app/                  # Core package
│   │   ├── ingest.py         # GraphRAG pipeline (text → triples → deduplication → persistence)
│   │   ├── knowledge_graph.py# Knowledge graph engine (CRUD + queries + persistence)
│   │   └── models.py         # Entity/relationship data models
│   ├── config/
│   │   └── ingest.toml.example# Extraction model configuration template (actual config is not committed)
│   ├── scripts/
│   │   └── seed.py           # Seed data initialization
│   ├── tests/
│   │   ├── test_mcp_server.py# MCP server unit tests
│   │   ├── test_ingest.py    # Extraction pipeline unit tests (including no-write tests for casual chat)
│   │   └── test_upload_api.py# Document upload tests (allowlist, size limits, filename sanitization)
│   ├── requirements.txt
│   ├── data/
│   │   ├── graph.json        # Knowledge graph data
│   │   └── backups/          # Rolling backups (retain 20)
│   ├── uploads/              # Uploaded image files
│   └── doc_uploads/          # Uploaded documents (.pdf/.md/.txt/.docx/.xmind)
├── assets/                   # README screenshots
├── skills/
│   └── acefelix-knowledge/   # jarvis Skill (graph usage instructions; copy to ~/.jarvis/skills/)
├── frontend/                 # Vue 3 frontend
│   ├── src/
│   │   ├── App.vue           # Main interface (three-column layout + right-side details)
│   │   ├── api/index.js      # API request wrapper
│   │   ├── config/graph.config.js  # 3D visualization and cosmic theme settings
│   │   ├── utils/            # Shared utilities (sorting, property detection, planet textures, cosmic scene, camera, labels)
│   │   └── components/
│   │       ├── Graph3D.vue            # 3D renderer and interaction coordination (scene/texture/camera/labels in utils)
│   │       ├── EntityPanel.vue        # Entity management (image/document properties)
│   │       ├── RelationPanel.vue      # Relationship management
│   │       ├── ReaderPanel.vue        # Inline image/PDF/txt/md/docx viewer in the central area
│   │       ├── TypeManager.vue        # Entity type management
│   │       ├── RelationTypeManager.vue # Relationship type management
│   │       └── StatsBar.vue           # Statistics bar
│   └── package.json
├── docs/                     # Project documentation
│   ├── ARCHITECTURE.md       # Architecture
│   ├── TECHNICAL.md          # Technical documentation
│   ├── API.md                # REST API reference
│   ├── DEPLOYMENT.md         # Deployment and operations
│   └── fixlogs/              # Fix reports and retrospectives
├── .github/workflows/ci.yml  # GitHub Actions CI (backend tests + frontend build)
└── start.bat                 # One-click Windows startup
```

## Quick Start

> See [docs/DEPLOYMENT.md](./docs/DEPLOYMENT.md) for the full procedure.
>
> 💡 **One-click startup on Windows**: Double-click `start.bat` to start the backend (:8800) and frontend (:5173) automatically. The browser opens **only after a readiness check confirms that the services respond**. Startup output is saved in `logs/` for troubleshooting.

### Requirements

- Python 3.11+ (the extraction pipeline uses the standard-library `tomllib` module)
- Node.js 16+
- npm or yarn

### 1. Start the Backend

```bash
cd backend
python -m pip install -r requirements.txt
python scripts/seed.py  # Initialize seed data on first run (skipped if data already exists)
python api.py           # Start the API at http://127.0.0.1:8800
```

> If you encounter environment issues on Windows, explicitly set `PYTHONPATH=<project-root-directory>` before running `python -m uvicorn api:app --reload --host 127.0.0.1 --port 8800`.

### 2. Start the Frontend

```bash
cd frontend
npm install
npm run dev             # Start the development server at http://127.0.0.1:5173
```

Open **http://127.0.0.1:5173/** in your browser.

### 3. Start the MCP Server (Agent Integration)

```bash
cd backend
python mcp_server.py          # Start the stdio-based MCP server for agent clients
```

> You do not need to start this process separately: agent clients such as jarvis launch it automatically when required.

### 4. Getting Started

- Manage entities and relationships in the left panel. Use the ⚙ button to manage types. The statistics bar at the bottom shows entity/relationship counts and type distribution.
- While editing an entity, paste an image URL or upload a local image under **Image Properties**. Under **Document Properties**, paste a document URL or upload a local file (.pdf/.md/.txt/.docx/.xmind).
- Click a 3D node to open its details. Image properties render automatically, while documents appear as 📄 filename links. The corresponding item in the left-side entity list is highlighted and scrolled into view. Click an image or document to view it in the central reader (inline support for images, PDF, txt, Markdown, and docx; xmind opens in a new window instead). Press Esc or click the backdrop to close it. The toolbar includes **Focus**, **Path**, and **Reset**.
- Use the search box in the upper-right corner to search entities by full text.

<p align="center">
  <img src="assets/zhishitupu2.png" alt="Entity management panel and type filtering" width="90%">
</p>

## API Endpoints

| Method | Path | Description |
|---|---|---|
| GET | /api/meta | Type, color, and version metadata |
| GET | /api/types | List entity types |
| POST/PUT/DELETE | /api/types(/name) | Create, update, or delete entity types |
| GET | /api/relation-types | List relationship types |
| POST/PUT/DELETE | /api/relation-types(/name) | Create, update, or delete relationship types |
| GET/POST | /api/entities(/id) | Query or create entities |
| PUT/DELETE | /api/entities/{id} | Update or delete an entity (cascading relationship deletion) |
| GET/POST | /api/relations(/id) | Query or create relationships |
| PUT/DELETE | /api/relations/{id} | Update or delete relationships |
| GET | /api/graph | Full graph (for 3D rendering) |
| GET | /api/graph/neighbors/{id} | Neighbor subgraph (optional `degree`) |
| GET | /api/graph/paths | Relationship paths between two entities |
| GET | /api/graph/common | Common neighbors of two entities |
| GET | /api/search?q= | Search entities |
| GET | /api/stats | Graph statistics |
| POST | /api/ingest | Extract knowledge from text (dry-run preview; no writes for casual chat) |
| POST | /api/ingest/file | Upload a .txt/.md/.json file for extraction |
| POST | /api/upload | Upload an image |
| POST | /api/upload/doc | Upload a document (.pdf/.md/.txt/.docx/.xmind; 20 MB maximum) |

For complete request and response examples, see [docs/API.md](./docs/API.md). FastAPI interactive API documentation is available at `http://127.0.0.1:8800/docs`.

## Knowledge Extraction (GraphRAG)

Send text to `POST /api/ingest` (or upload a file to `POST /api/ingest/file`). The LLM automatically extracts entities and relationships and adds them to the graph.

```bash
# Preview only (do not write to the graph)
curl -X POST http://127.0.0.1:8800/api/ingest -H "Content-Type: application/json" \
  -d "{\"text\": \"Xiaoming knows Python and is contributing to the AceFelix project.\", \"dry_run\": true}"
```

- **Noise prevention**: Five safeguards (no direct connection to live chat, value assessment, type allowlist, density pre-check, and human confirmation) prevent casual greetings and chatter from being stored. See Section 8 of [docs/TECHNICAL.md](./docs/TECHNICAL.md).
- **Model configuration**: By default, extraction uses the `DASHSCOPE_API_KEY` environment variable and DashScope's `qwen-flash`. To customize this, copy `backend/config/ingest.toml.example` to `backend/config/ingest.toml` (the actual config is not committed).
- **Safety**: Use `dry_run=true` to preview changes before writing. Backups are created automatically before writes, and the graph can be restored from `data/backups/`.

## Agent Integration

The graph is exposed to AI agents through an **MCP server** ([backend/mcp_server.py](./backend/mcp_server.py)). Any client supporting the MCP protocol (jarvis, Claude Desktop, Cursor, etc.) can connect.

### jarvis Integration

1. Add the following under `mcpServers` in `~/.jarvis/mcp.json`:

```json
"acefelix-knowledge": {
  "command": "python",
  "args": ["<absolute-path-to-project>/backend/mcp_server.py"]
}
```

2. Install the Skill so jarvis knows when and how to use the graph:

```bash
cp -r skills/acefelix-knowledge ~/.jarvis/skills/
```

3. Restart jarvis. You can now use the `mcp__acefelix-knowledge__*` tools to query and maintain the graph.

### Other Agents

Register the same launch command in your client's MCP configuration. Available tools:

| Tool | Access | Description |
|---|---|---|
| `get_profile` | Read-only | Summary of a person's profile (recommended as the first call when learning about the user) |
| `search_entity` / `get_entity` / `list_entities` | Read-only | Entity search and details |
| `get_neighbors` / `find_paths` / `common_neighbors` | Read-only | Graph structure queries |
| `get_stats` / `list_types` / `list_relation_types` | Read-only | Statistics and type lists |
| `add_entity` / `add_relation` | Write | Create entities or relationships (the client should request user confirmation by default) |

> The data file `backend/data/graph.json` contains local personal information and is not committed to the repository. After cloning, run `python scripts/seed.py` to generate the initial graph.

## Roadmap

> For a complete upgrade plan, including technical choices and implementation ideas, see [docs/plans/upgrade-plan.md](./docs/plans/upgrade-plan.md).

- [x] Integrate with JARVIS so JARVIS can better understand the user (MCP server integration complete).
- [x] P1 — Automatic GraphRAG extraction: Extract entities and relationships from conversations and documents (`ingest.py` + `/api/ingest`, with five noise-prevention safeguards).
- [x] P2 — Bidirectional profile synchronization: Write extracted conversation insights back to the graph through `ingest_text` and inject the graph profile into the system prompt (2026-08-27).
- [ ] P3 — Semantic search: Replace keyword matching with embedding-based semantic retrieval.
- [ ] P4 — Relationship recommendations: Suggest similar entities and potential interests.
- [ ] P5 — Migrate to SQLite / Kùzu / Neo4j as the dataset grows.

## Contributing and License

Issues and pull requests are welcome. Before submitting a PR, make sure the backend unit tests pass, the frontend builds successfully, and the documentation is updated accordingly.

This project is open source under the [Apache License 2.0](./LICENSE), © 2026 aceFelix.

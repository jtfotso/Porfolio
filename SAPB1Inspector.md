# SAP B1 Form Metadata Inspector

Semi-live inspection system that captures SAP Business One UI form metadata at runtime and exposes it through a web-based Blazor viewer.

## Architecture

```
┌─────────────────────────┐
│  SAP Business One Client │
│  SwissAddonFramework     │
│  - UI Event Listener     │
│  - Form Inspector        │
│  - JSON Snapshot Builder │
│  - REST Publisher        │
└───────────┬─────────────┘
            │ HTTP/JSON
            ▼
┌─────────────────────────┐
│  Backend (Clean Arch.)   │
│  Presentation (REST/SigR)│
│  Application (Use Cases) │
│  Domain (Entities)       │
│  Infrastructure (Adapters│
└───────────┬─────────────┘
            │ SignalR
            ▼
┌─────────────────────────┐
│  Blazor Server Viewer    │
│  - Live Form Tree        │
│  - Field Details         │
│  - Semi-Live Updates     │
└─────────────────────────┘
```

### Clean Architecture Dependency Rule

```
Presentation → Application → Domain
Infrastructure → Application → Domain
```

**Forbidden:** Domain → Application, Application → Infrastructure, Domain → ASP.NET/SignalR

## Solution Structure

```
SapB1.FormInspector.sln
 ├─ src/
 │  ├─ SapB1.Addon.FormInspector/       SAP B1 add-on (SwissAddonFramework)
 │  ├─ Backend/
 │  │  ├─ FormInspector.Domain/         Pure domain model (zero dependencies)
 │  │  ├─ FormInspector.Application/    Use cases, ports, DTOs
 │  │  ├─ FormInspector.Infrastructure/Adapters (persistence, SignalR, time)
 │  │  └─ FormInspector.Presentation/  REST API + SignalR hub
 │  └─ Viewer/
 │      └─ FormInspector.BlazorServer/  Blazor Server viewer
 ├─ tests/
 │  ├─ FormInspector.Domain.Tests/
 │  ├─ FormInspector.Application.Tests/
 │  └─ FormInspector.Infrastructure.Tests/
 └─ docs/
```

## Projects

| Project | Type | Role |
|---------|------|------|
| `FormInspector.Domain` | Class Library | Entities, value objects, enums — zero framework dependencies |
| `FormInspector.Application` | Class Library | Use cases, ports (interfaces), DTOs, mapper |
| `FormInspector.Infrastructure` | Class Library | In-memory repository, SignalR notifier, time provider, DI module |
| `FormInspector.Presentation` | Web API | REST controllers, SignalR hub, Swagger, error handling |
| `FormInspector.BlazorServer` | Blazor Server | Read-only reactive viewer with semi-live updates |
| `SapB1.Addon.FormInspector` | Class Library | SAP add-on — event handling, inspection, snapshot publishing |

## Getting Started

### Prerequisites

- .NET 10 SDK
- (For SAP add-on) SAP Business One client with SwissAddonFramework SDK

### Build

```bash
dotnet build
```

### Run Tests

```bash
dotnet test
```

### Run Backend API

```bash
dotnet run --project src/Backend/FormInspector.Presentation
```

The API is available at `http://localhost:5000` with Swagger at `/swagger`.

### Run Blazor Viewer

```bash
dotnet run --project src/Viewer/FormInspector.BlazorServer
```

The viewer is available at `http://localhost:5001`.

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/snapshot` | Receive a snapshot from the SAP add-on |
| `GET` | `/api/snapshot` | List all snapshots (summary) |
| `GET` | `/api/snapshot/latest/{formType}` | Get latest snapshot for a form type |
| `GET` | `/api/snapshot/{snapshotId}` | Get a specific snapshot by ID |

## SignalR

The backend exposes a SignalR hub at `/snapshothub` that broadcasts `SnapshotUpdated` notifications when new snapshots are received.

## Key Design Decisions

- **Read-only** — The system never modifies SAP UI or data
- **Semi-live** — Updates driven by structural UI events (FormLoad, FormActivate, FormVisible, FormModeChange), not polling
- **Event throttling** — Prevents UI freezes in the SAP client (default: 1000ms interval per form type)
- **Schema versioning** — Snapshots carry a `schemaVersion` field for forward compatibility
- **In-memory storage** — MVP uses in-memory repository; swappable to SQLite/SQL Server later
- **Clean Architecture** — Domain has zero dependencies; all infrastructure details hidden behind interfaces

## SAP Add-On

The SAP add-on project is structured but has TODO placeholders where SwissAddonFramework/SAPbouiCOM API calls need to be implemented. Key classes:

- `FormInspectorService` — Extracts form-level metadata
- `ItemInspector` — Extracts item-level metadata
- `MatrixInspector` — Extracts matrix/grid column metadata
- `SnapshotBuilder` — Assembles pure metadata snapshots (no SAP COM references)
- `SnapshotPublisher` — Publishes snapshots to backend via HTTP with retry logic
- `FormEventDispatcher` — Subscribes to structural UI events only

## Documentation

See the `docs/` folder:

- `architecture.md` — System architecture overview
- `snapshot-schema.md` — Snapshot JSON schema reference
- `backend-clean-architecture.md` — Backend layer responsibilities
- `sap-addon-notes.md` — SAP add-on integration guide

# Northstar Print Studio

Offline Windows print-production software. The codebase is moving in stages from the original Python/PySide6 application to the architecture in the supplied setup guide:

```text
React + TypeScript + Vite  →  Java 21 + Spring Boot  →  SQLite / local files
             Tauri 2 provides the Windows desktop shell
```

The Python application remains in `northstar_studio/` as a working legacy implementation while the new modules are brought to feature parity. The two applications use separate SQLite databases; this milestone does not migrate or overwrite existing Python project data.

## Current migration milestone

The new foundation provides:

- A React workspace with overview, project browser, new-project flow, and a layer-based document editor.
- A Java local API for health checks and project create/read/update/delete.
- A versioned, JSON document model with physical millimetre dimensions and editable layers.
- SQLite persistence in the user's application-data directory.
- A Tauri 2 Windows desktop-shell scaffold.
- A Photoshop-inspired editor layout: compact tool rail, central artboard, contextual properties, and a named layer stack.

The editor supports the core card workflow: choose a preset or custom page size and orientation, design editable layers with font size and color, import a CSV (including Excel files saved as CSV) and map its columns to `{{field_name}}` placeholders, preview each record, and export a flattened print-resolution PSD or PNG locally. PSD and PNG exports include the selected DPI metadata. Imported rows are held in the editor session and are not saved into project records.

The generated PSD is a flattened composite rather than an editable Photoshop layer stack; the source layers remain editable in the Northstar project. The Java renderer, preflight, production queue, and imposition still need to be migrated before the new application replaces the Python implementation.

## Local development

Prerequisites for the documented target stack:

- Windows 10/11 with Visual Studio C++ Build Tools
- JDK 21
- Node.js 22 LTS or newer
- Rust stable/MSVC toolchain for Tauri
- Maven 3.9+

Start the Java service in one PowerShell window:

```powershell
cd backend
mvn spring-boot:run
```

Start the React workspace in another:

```powershell
cd frontend
npm install
npm run dev
```

The Vite UI calls the API at `http://127.0.0.1:8080/api` by default. Override it with `VITE_API_BASE_URL` when needed. The service binds to loopback only and allows the local Vite/Tauri origins; no remote service is required for normal application use.

Run the Tauri development shell after Rust/MSVC prerequisites are installed:

```powershell
cd frontend
npm run tauri:dev
```

At this milestone, start the Java service separately before opening the desktop shell. Bundling and lifecycle management for the Java runtime are required before producing a customer-ready installer.

## Backend API

| Method | Route | Purpose |
| --- | --- | --- |
| `GET` | `/api/health` | Local engine status |
| `GET` | `/api/projects` | List saved projects |
| `POST` | `/api/projects` | Create a project using a supported preset and optional custom width/height |
| `GET` | `/api/projects/{id}` | Load one project |
| `PUT` | `/api/projects/{id}` | Save project name, optional width/height, DPI, and design model |
| `DELETE` | `/api/projects/{id}` | Delete one project |

SQLite is stored under `%APPDATA%\\Northstar Print Studio\\northstar.db` by default. `NORTHSTAR_DATA_DIR` can point the engine to another local data directory.

## Document model

The API persists a versioned design object. Coordinates and document sizes are millimetres; conversion to 300-DPI output belongs to the Java renderer, not the React canvas.

```json
{
  "schemaVersion": 1,
  "unit": "mm",
  "activePageId": "front",
  "pages": [
    { "id": "front", "name": "Front", "layers": [] }
  ]
}
```

Supported project presets are ID Card (85 × 55 mm), Business Card (90 × 55 mm), Certificate (A4), and Letterhead (A4). Project dimensions can be customized from 1 to 2000 mm and saved with the project. The versioned model is the contract for later renderer and preflight modules.

## Product and UX decisions

The supplied **React + Java + SQLite + Tauri Setup Guide** is treated as the target architecture and phased migration plan. Its first milestone is the React-to-Java-to-SQLite path; bulk print features are not to be added ahead of a stable project/editor foundation.

The **Additional Optional Features** document explicitly defers undo/redo, autosave/history, advanced color/font/image controls, portable project bundles, printer integration, and commercialization. These are not blockers for the core migration.

The editor uses established image-editor patterns without copying Photoshop assets: keep tools discoverable in a compact rail, keep the canvas central, show properties for the current selection, and expose object names and order in a persistent layers panel. See Adobe's [workspace overview](https://helpx.adobe.com/photoshop/desktop/get-started/learn-the-basics/workspace-overview.html), [contextual task bar](https://helpx.adobe.com/photoshop/desktop/get-started/learn-the-basics/boost-workflows-with-the-contextual-task-bar.html), [layer panel guidance](https://helpx.adobe.com/photoshop/desktop/create-manage-layers/get-started-layers/organize-layers-in-the-layers-panel.html), and [undo/redo guidance](https://helpx.adobe.com/photoshop/desktop/get-started/set-up-toolbars-panels/use-undo-redo-commands.html).

## Legacy application

The existing offline Python/PySide6 app can still be launched with:

```powershell
python -m northstar_studio.main
```

Its database and production workflows remain separate from the new Java/React foundation until their replacements are validated.

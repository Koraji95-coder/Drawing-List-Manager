# R3P Drawing List Manager

A standalone Tauri desktop application for managing project drawing registers at ROOT3POWER ENGINEERING. This is one of the tools in the ROOT3POWER tool family.

> **Status:** Shell / placeholder — feature implementation is in progress.

---

## Architecture

```
Drawing-List-Manager/
├── backend/                   Python FastAPI service (port 8001)
│   ├── app.py                 All API routes
│   ├── core/
│   │   ├── register.py        JSON register model + open/save helpers
│   │   ├── excel_import.py    Legacy Master Deliverable List import
│   │   └── excel_export.py    Branded Excel export (full register)
│   └── requirements.txt
│
└── frontend/                  React/Vite web + Tauri desktop shell
    ├── src/
    │   ├── App.jsx            Main React application
    │   └── main.jsx           React entry point
    ├── src-tauri/             Tauri desktop shell
    │   ├── tauri.conf.json    Window / bundle configuration
    │   ├── Cargo.toml         Rust workspace manifest
    │   ├── build.rs           Tauri build script
    │   ├── src/
    │   │   ├── main.rs        Binary entry point
    │   │   └── lib.rs         Tauri app logic + backend auto-start
    │   ├── capabilities/      Tauri permission grants
    │   └── icons/             App icon assets
    ├── package.json
    └── vite.config.js
```

**Data flow**

```
┌─────────────────────────────┐     HTTP/REST      ┌───────────────────┐
│  Tauri WebView               │ ─────────────────► │  Python FastAPI   │
│  React UI (port 1420 dev)   │ ◄───────────────── │  (port 8001)      │
└─────────────────────────────┘                     └───────────────────┘
        Tauri shell (Rust)            ▲
        wraps the WebView             │
                │                     │
                └── spawns backend ───┘
                    on startup (dev)
```

In dev mode, Tauri automatically spawns the Python backend when the desktop app starts.

---

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/health` | Health check — returns `{"status":"ok","version":"1.0.0"}` |
| POST | `/api/register/open` | Read a `.r3pdrawings.json` register from disk |
| POST | `/api/register/save` | Write a register to disk as JSON |
| POST | `/api/register/import-excel` | Import a legacy Master Deliverable List `.xlsx` |
| POST | `/api/register/export-full` | Write the full branded `.xlsx` (one sheet per set) |

---

## Quick Start — Web (browser)

```bash
# Terminal 1 — Python backend
cd backend
pip install -r requirements.txt
uvicorn app:app --reload --port 8001

# Terminal 2 — Vite dev server
cd frontend
npm install
npm run dev          # http://localhost:1420
```

API docs at <http://localhost:8001/docs>

---

## Quick Start — Tauri Desktop

**Prerequisites (all platforms)**

1. Python 3.10+ with `pip` (or a Conda/virtualenv environment)
2. Backend dependencies installed:
   ```bash
   cd backend
   pip install -r requirements.txt
   ```
3. [Rust](https://www.rust-lang.org/tools/install) — `rustup` installs the toolchain
4. Node.js ≥ 20 and npm

**Additional prerequisites (Linux)**

```bash
sudo apt update
sudo apt install -y \
  libwebkit2gtk-4.1-dev libgtk-3-dev librsvg2-dev \
  patchelf libssl-dev libayatana-appindicator3-dev
```

### Run the desktop app (single command)

```bash
cd frontend
npm install
npm run desktop      # = tauri dev
```

Tauri will:
1. Start the Vite dev server on port 1420
2. Compile and launch the native Rust binary
3. **Automatically spawn the Python backend** on `127.0.0.1:8001`
4. Open the desktop window

### Python environment requirements

Tauri searches for Python in this order:

1. **`$CONDA_PREFIX`** — the active conda environment
2. **Well-known Miniconda / Anaconda install directories** under your home folder
3. **`python` on `PATH`** — final fallback

```bash
conda activate base        # or your project environment
cd frontend
npm run desktop
```

### Troubleshooting backend auto-start

| Symptom | Fix |
|---------|-----|
| "Python not found" in terminal | Activate your Miniconda environment and retry |
| "Could not find backend/app.py" | Run `npm run desktop` from the `frontend/` directory |
| "Backend process exited early" | Run `pip install -r backend/requirements.txt` |
| "Backend failed to start" in UI | Check terminal for errors; start backend manually |

### Manual backend (fallback)

```bash
# Terminal 1
cd backend
uvicorn app:app --reload --port 8001

# Terminal 2
cd frontend
npm run desktop
```

---

## Version History

| Version | Description |
|---------|-------------|
| **1.0** | Initial shell — Tauri desktop scaffolding, placeholder UI, FastAPI backend stub |

---

ROOT3POWER ENGINEERING

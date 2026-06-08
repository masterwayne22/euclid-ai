# Euclid AI — Vector Database Engine in C++

A fully working **Vector Database** built in C++ with a modern web UI.
Implements **HNSW**, **KD-Tree**, and **Brute Force** search algorithms side-by-side, plus a **RAG pipeline** powered by a local LLM via Ollama.

> Built to understand how production vector databases like Pinecone, Weaviate, and Chroma work under the hood — and to explore real-world C++ systems programming.

---

## About This Project

This project was built and maintained by **Krishiv Sarva** ([@masterwayne22](https://github.com/masterwayne22)), a Computer Science undergraduate at VIT Bhopal University.

### What I Fixed & Improved

- **Fixed a critical Windows port-binding bug** — the original `svr.listen()` call was silently failing on Windows due to a conflicting system process on port 8080. Fixed by adding `httplib::ThreadPool` and migrating to port 9090, making the server actually bind and stay alive.
- **Debugged and verified the full RAG pipeline** — tested end-to-end document embedding, HNSW retrieval, and LLM generation with Ollama locally.
- **Documented the Windows setup process** with real troubleshooting steps missing from the original.

---

## What This Project Does

| Feature | Description |
|---|---|
| **3 Search Algorithms** | HNSW (production-grade), KD-Tree, Brute Force — run all three and compare speed |
| **3 Distance Metrics** | Cosine similarity, Euclidean distance, Manhattan distance |
| **16D Demo Vectors** | 20 pre-loaded semantic vectors across 4 categories (CS, Math, Food, Sports) |
| **2D PCA Scatter Plot** | Live visualization of semantic space — watch clusters form |
| **Real Document Embedding** | Paste any text → Ollama embeds it with `nomic-embed-text` (768D) |
| **RAG Pipeline** | Ask questions about your documents → HNSW retrieves context → local LLM answers |
| **Full REST API** | CRUD endpoints: insert, delete, search, benchmark, hnsw-info |

---

## How It Works

```
Your Text
    │
    ▼
Ollama (nomic-embed-text)          ← converts text to a 768-dimensional vector
    │
    ▼
HNSW Index (C++)                   ← indexes the vector in a multilayer graph
    │
    ▼
Semantic Search                    ← finds nearest neighbors in vector space
    │
    ▼
Ollama (llama3.2)                  ← reads retrieved chunks, generates an answer
    │
    ▼
Answer
```

**HNSW (Hierarchical Navigable Small World)** is the same algorithm used by Pinecone, Weaviate, Chroma, and Milvus. It builds a multilayer graph where each layer is progressively sparser — searches start at the top layer and zoom in, achieving O(log N) complexity instead of O(N) for brute force.

---

## Prerequisites

You need **3 things** installed on your Windows laptop:

1. **MSYS2** (gives you g++ compiler)
2. **Git**
3. **Ollama** (runs the local AI models)

---

## Step-by-Step Setup (Windows)

### Step 1 — Install MSYS2 (C++ Compiler)

1. Go to **https://www.msys2.org** and download the installer
2. Run the installer, keep default path (`C:\msys64`)
3. After install, open **MSYS2 UCRT64** from Start Menu (the orange icon)
4. Run these commands inside the MSYS2 terminal:

```bash
pacman -Syu
```
*(Close and reopen the terminal if it asks you to)*

```bash
pacman -S mingw-w64-ucrt-x86_64-gcc
```

5. Add g++ to your Windows PATH:
   - Press `Win + R`, type `sysdm.cpl`, press Enter
   - Click **Advanced** → **Environment Variables**
   - Under **System variables**, find **Path**, click **Edit**
   - Click **New** and add: `C:\msys64\ucrt64\bin`
   - Click OK on all windows
   - **Open a new PowerShell** and verify:
   ```
   g++ --version
   ```

---

### Step 2 — Install Git

Download from **https://git-scm.com/download/win** and install with default settings.

---

### Step 3 — Install Ollama

1. Go to **https://ollama.com** and download for Windows
2. After install, open PowerShell and pull both models:

```powershell
ollama pull nomic-embed-text
ollama pull llama3.2
```

---

### Step 4 — Clone & Compile

```powershell
git clone https://github.com/masterwayne22/VectorDB.git
cd VectorDB
g++ -std=c++17 -O2 main.cpp -o db -lws2_32
```

---

### Step 5 — Run

```powershell
.\db.exe
```

Open your browser at **http://localhost:9090**

You should see:
```
=== VectorDB Engine ===
http://localhost:9090
20 demo vectors | 16 dims | HNSW+KD-Tree+BruteForce
Ollama: ONLINE
```

> **Note:** The server runs on port **9090** (not 8080) — port 8080 conflicts with a Windows system service (`AgentService.exe`). This is the fix applied in this repo.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `Ollama: OFFLINE` | Run `ollama serve` in a separate terminal |
| `g++: command not found` | Add `C:\msys64\ucrt64\bin` to Windows PATH |
| Server exits immediately | Another process is holding the port — check with `netstat -ano \| findstr :9090` |
| LLM answer is slow | Normal on CPU — switch to `llama3.2:1b` for faster responses |

### Use a Smaller/Faster LLM

```powershell
ollama pull llama3.2:1b
```

Then in `main.cpp` find the `genModel` line and change it to `"llama3.2:1b"`, then recompile.

---

## Project Structure

```
VectorDB/
├── main.cpp        ← C++ backend (HNSW, KD-Tree, BruteForce, REST API, RAG)
├── httplib.h       ← Single-header HTTP server library (cpp-httplib)
├── index.html      ← Frontend (PCA scatter plot, chat UI, benchmark)
└── README.md       ← This file
```

---

## Algorithm Overview

| Algorithm | Complexity | Type |
|---|---|---|
| Brute Force | O(N·d) | Exact |
| KD-Tree | O(log N) | Exact, degrades at high dims |
| HNSW | O(log N) | Approximate, scales to 768D+ |

---

## REST API

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/search?v=...&k=5&metric=cosine&algo=hnsw` | K-NN search |
| `POST` | `/insert` | Insert a vector |
| `GET` | `/benchmark?v=...` | Compare all 3 algorithms |
| `POST` | `/doc/insert` | Embed and store a document |
| `POST` | `/doc/ask` | RAG: retrieve + generate answer |
| `GET` | `/status` | Ollama status |
| `GET` | `/stats` | DB statistics |

---

## License

MIT 


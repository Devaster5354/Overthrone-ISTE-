# Project Specification: Overthrone Infrastructure

**Abhinivesh Sharma**  
*Lead Backend Engineer*  
April 23, 2026

---

### Abstract
Overthrone is a highly concurrent, event-driven digital simulation platform. Designed as a distributed system, it handles multi-user state synchronization, secure sandboxed code execution, and persistent competitive event management. This report documents the architectural decisions, specifically focusing on decoupled logic, epoch-based state machines, and ephemeral infrastructure management.

---

## 1. System Overview
Overthrone acts as a "Digital War Room," enabling teams to compete in a simulated territory acquisition environment. The system addresses the challenges of web-based game state management by ensuring transactional integrity across high-frequency client updates.

---

## 2. Core Architectural Pillars

### 2.1 Decoupled Panel Architecture (Frontend)
To ensure scalability and maintainability, the frontend is designed as a *Panel-based Architecture*. Each functional area of the game is isolated into independent modules, allowing for modular updates and distinct state handling:

* **Battle Map** (`battle_map.py`): Handles tactical grid rendering and territory state visualization.
* **Strategy Deck** (`strategy_deck.py`): Manages game-action logic (cards/moves) and player input validation.
* **Comms Feed** (`comms_feed.py`): Provides an immutable event log of all system actions, ensuring transparency.
* **Code Terminal** (`code_terminal.py`): An integrated development environment (IDE) for real-time task submission and algorithm solving.
* **WebSocket Terminal** (`ws_terminal.py`): Monitors the real-time heartbeat of the system and event broadcasts.

### 2.2 Backend Logic & State Management
The backend treats game state as an atomic, serialized object stored within a Supabase (PostgreSQL) JSONB data structure.

* **Epoch-Based State Machine**: The game operates on a deterministic 300-second cycle (Epoch). By serializing the state at the end of every epoch, the system handles race conditions and allows for "Time-Travel" state restoration.
* **Persistence Layer**: Leveraging Supabase for Row-Level Security (RLS) ensures that while the client interface is reactive and open, the underlying data remains secure and immutable from unauthorized injection.


Client Interface         Actions         App Layer          DB Commit        Supabase(Modular Panels)   ------------------>  Logic & Epoch  ------------------> (PostgreSQL)<------------------     Engine      <------------------UI Refresh                           State Sync

---

## 3. Security & Sandboxing
A critical requirement of the platform is the "Bot Interface"—an environment where users submit Python code to solve challenges.

* **Process Isolation**: The `run_code_safe` function in `db.py` executes user logic in a restricted sub-process.
* **Threat Neutralization**: We implemented a strict import-whitelist and a blocklist for high-risk modules (`os`, `sys`, `subprocess`, `shutil`), effectively neutralizing potential Remote Code Execution (RCE) vectors.
* **Timeout Protection**: Execution is gated by hard timeouts, preventing Denial of Service (DoS) attacks via infinite loops or resource-heavy computations.

---

## 4. Deployment & Infrastructure
The deployment philosophy treats the infrastructure as ephemeral.

* **Stateless Backend**: By hosting on Railway, we ensure the application logic has no local state dependency. All session data resides in the cloud, allowing for zero-downtime deployments.
* **Infrastructure-as-Code**: The deployment utilizes automated CI/CD pipelines, ensuring that the containerized environment is consistent across development, testing, and production.
* **Low Latency**: WebSocket integration reduces the standard HTTP round-trip latency, enabling near-instantaneous synchronization of the "War Map" for all active participants.
Would you like help creating a Tech Stack checklist or adding Badge graphics (like Python, Supabase, Railway) to the top of this file to make it look even more professional on GitHub?

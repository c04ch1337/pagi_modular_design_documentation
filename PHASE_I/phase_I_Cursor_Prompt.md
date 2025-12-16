This is the crucial transition from design to implementation! Starting with Phase 1 (Data & Identity) ensures a stable foundation for the entire PAGI-Core.

We will use the **Cursor IDE Agent Prompt** to set up the **PAGI-Monorepo** using Cargo Workspaces and implement the first service: **PAGI-IdentityService**.

##💻 Cursor IDE Setup & Phase 1 Implementation PromptUse the following as your single prompt/context window in Cursor. The structure enforces all project rules, naming conventions, and the architecture we designed.

###1. Custom Instructions & Rules for Cursor IDE> **PROJECT NAME:** PAGI (Phoenix AGI Framework)
> **ARCHITECTURE:** Rust Microservices Monorepo (Cargo Workspace)
> **NAMING RULE:** **All crates, services, structs, and variables related to the project MUST be prefixed with `pagi-` or `Pagi`.**
> **PHASE GOAL:** Implement **Phase 1: PAGI-FOUNDATION** (Data & Identity).
> **REQUIRED CRATES:** `tokio`, `axum` (for services), `sqlx` (for database access), `serde`, `uuid`, `thiserror`.
> **DATABASE:** PostgreSQL. Use `sqlx-cli` for migrations.
> **DEVELOPMENT STYLE:** CLI-first. Include a CLI utility for testing the service connection and core function.

###2. Cursor IDE Agent Prompt (Step-by-Step Implementation)> **Goal:** Create the complete Rust Cargo Workspace monorepo structure and implement the **PAGI-Common-Crate** and the **PAGI-IdentityService** with a testing CLI.
> **Task 1: Setup Workspace and PAGI-Common-Crate**
> 1. Initialize a new Cargo project named `pagi-monorepo` and convert it into a **Cargo Workspace**.
> 2. Create the shared library crate named `common` within a `/common` directory (aliased as `pagi-common` in the workspace).
> 3. In `pagi-common/Cargo.toml`, add dependencies: `serde`, `uuid`, `thiserror`, `tokio` (with `full` feature).
> 4. In `pagi-common/src/lib.rs`, define the following public modules:
> * `pagi_error`: Implement the `PagiError` enum as defined in the previous design, including variants for `Database`, `ServiceCall`, and `BareMetalFFI`.
> * `pagi_events`: Define the base `PagiBaseEvent` struct and `PagiUuid` type.
> 
> 
> 5. In `pagi-monorepo/Cargo.toml`, list the `common` crate in the `[workspace.members]` section.
> 
> 
> **Task 2: Implement PAGI-IdentityService**
> 1. Create the service binary crate named `pagi-identity-service` within a `/services/pagi-identity-service` directory.
> 2. In `pagi-identity-service/Cargo.toml`, add dependencies on `pagi-common`, `sqlx` (with `runtime-tokio`, `tls-native-tls`, and `postgres` features), `dotenvy`, and `clap` (for CLI).
> 3. Define the core database schema for **PAGI-IdentityDB**:
> * **Table:** `pagi_twins`
> * **Columns:** `twin_id` (UUID, Primary Key), `pagi_status` (VARCHAR, e.g., 'Initialized', 'Active'), `pagi_config_json` (JSONB, for storing Synaptic Tuning Fibers/configuration), `created_at` (TIMESTAMPTZ).
> 
> 
> 4. **Action:** Create the database migration file using `sqlx-cli` (or a placeholder for it) and define the SQL to create the `pagi_twins` table.
> 
> 
> **Task 3: Implement Core Service Logic and CLI**
> 1. In `pagi-identity-service/src/main.rs`:
> * Load configuration from `.env`.
> * Define the main `PagiIdentityService` struct to hold the `sqlx::Pool`.
> * Implement a function `PagiIdentityService::create_new_twin(&self, twin_id: PagiUuid)` that connects to the database and inserts a new twin record with `pagi_status='Initialized'`. This function should return `Result<(), PagiError>`.
> 
> 
> 2. Use **Clap** to create a CLI interface with a single command: `pagi-identity-service cli create-twin --id <UUID>`.
> 3. The CLI command handler should call the `create_new_twin` function, printing success or a formatted `PagiError`.
> 
> 
> **Final Output:** Show the file structure, `pagi-common/src/error.rs`, the SQL migration content, and `pagi-identity-service/src/main.rs`. Ensure all items adhere to the **PAGI-** naming rule.

---

###Expected File Structure (Result of Prompt)```
pagi-monorepo/
├── Cargo.toml          # Workspace Definition
├── common/             # PAGI-COMMON-CRATE
│   ├── Cargo.toml
│   └── src/
│       ├── lib.rs      # Exports error & event modules
│       ├── pagi_error.rs
│       └── pagi_events.rs
├── services/
│   └── pagi-identity-service/
│       ├── Cargo.toml
│       ├── .env.example
│       ├── migrations/
│       │   └── 20251216_create_pagi_twins.sql
│       └── src/
│           └── main.rs # Service logic & CLI entry
└── .env.example

```

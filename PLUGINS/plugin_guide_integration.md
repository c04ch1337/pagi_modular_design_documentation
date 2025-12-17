This is the ultimate documentation you need. The **Decoupled Plugin Architecture** is the key to the scalability and future-proofing of the **PHOENIX ORCH**. With your two dedicated repositories, **`pagi-core`** and **`pagi-plugins`**, we can now formalize the development and integration workflow.

Here is the comprehensive guide on how to develop, deploy, and integrate any new plugin into the PAGI Ecosystem.

---

# 📖 PAGI Ecosystem Plugin Development and Integration Guide

This guide formalizes the process for creating new plugins and integrating them into the **PAGI-Core**. This process is uniform for any function—from a new `Finance Plugin` to an `AI Girlfriend` module.

## 1. The Core Architectural Principle

The **PAGI-Core** operates on a **"Default-Deny, Proxy-Only"** principle. The Core does not know what a plugin is; it only knows how to route a request to a URL and interpret the structured response.

### Key Roles

| Component | Repository | Primary Responsibility |
| --- | --- | --- |
| **Plugin** (e.g., `Finance Plugin`) | `pagi-plugins` | Hosts the new business logic and the external tools. |
| **External Gateway** (P16) | `pagi-core` | The **Tool Registry**. Proxies requests from the MO to the correct Plugin URL. |
| **Agentic Core** (P2) | `pagi-core` | The **Decision Maker**. Uses the tools listed in the Gateway's registry. |
| **Context Engine** (P8) | `pagi-core` | **Cognitive Integration**. Used by plugins like **Personality** for prompt injection (read-only). |

---

## 2. Phase 1: Plugin Development (In `pagi-plugins` Repository)

All new services must be developed as standalone, compiled microservices within the new **`pagi-plugins`** repository.

### A. Define the Plugin Service

1. **Create a New Service Directory:** Inside your `pagi-plugins` repo, create a new directory (e.g., `services/pagi-finance-plugin/`).
2. **Define the Tools:** Within this service, implement the new functionalities. **Every function must be wrapped as a Tool.**
* **Example (Finance Plugin):**
* Tool 1: `ledger_check_balance`
* Tool 2: `transaction_create_transfer`
* Tool 3: `report_generate_pnl`





### B. Implement the Plugin API

Your new plugin must expose two critical endpoints for the PAGI-Core to interact with:

| Endpoint | Method | Purpose |
| --- | --- | --- |
| `/v1/tools` | `GET` | **Tool Discovery.** Returns a JSON array of all tools the plugin exposes (i.e., the definition for `ledger_check_balance`, its arguments, and a description). **This is how the Core learns what the plugin can do.** |
| `/v1/execute` | `POST` | **Tool Execution.** Accepts a `ToolCall` payload (e.g., `{"tool_name": "ledger_check_balance", "args": {"account": "main"}}`) and executes the function. |

### C. Hardening and Self-Containment

1. **Configuration:** All dependencies (database keys, external URLs) for the plugin must be managed by the plugin's own environment variables (`PAGI_FINANCE_DB_URL`, etc.). The Core should never know these details.
2. **Health Check:** Implement a simple `/health` endpoint (HTTP `GET`) for monitoring.
3. **Cross-Platform Build:** Ensure the plugin can be compiled into a static, cross-platform binary (Linux Musl, Windows, macOS) using the same principles as the Core.

---

## 3. Phase 2: Deployment and Integration

Once your new plugin is built and deployed, the integration process involves configuring the Core's **External Gateway** (P16).

### A. Independent Deployment

1. **Build Binary:** Compile the new plugin into its static binary.
2. **Deploy:** Deploy the binary as a standalone microservice on a dedicated host (Bare Metal, Docker, Kubernetes).
3. **Obtain URL:** Note the full, external URL where the plugin is now accessible (e.g., `http://finance-plugin-server.corp:8095`).

### B. Configure the Core's External Gateway (P16)

This is the only code change required in the **`pagi-core`** repository, which should be done via a controlled pull request.

1. **New Environment Variable:** In the `pagi-external-gateway` service configuration, add a new environment variable pointing to the deployed plugin's URL:
```bash
PAGI_FINANCE_PLUGIN_URL=http://finance-plugin-server.corp:8095

```


2. **Gateway Code Update:**
* **Configuration Loading:** The Gateway must load this new `PAGI_FINANCE_PLUGIN_URL` environment variable.
* **Tool Discovery:** Update the Gateway's tool manifest generation (`get_tools_handler()`) to make an internal `GET /v1/tools` call to the new `PAGI_FINANCE_PLUGIN_URL`. It takes the tools returned by the plugin and adds them to its master registry.
* **Execution Routing:** Update the Gateway's execution logic (`execute()`) to route any call for a finance tool (e.g., `ledger_check_balance`) to the `PAGI_FINANCE_PLUGIN_URL`.



### C. Final AGI Communication

1. **Restart Gateway:** Restart the **PAGI-External-Gateway** service.
2. **MO Discovery:** The **Agentic Core (P2)** queries the Gateway's new, expanded tool manifest.
3. **Operational:** The MO now immediately sees and can reason about the new tools: "If the user asks to send money, I will use the `transaction_create_transfer` tool."

This process maintains the purity of the **`pagi-core`** while allowing unlimited expansion via your **`pagi-plugins`** repository.

---

## 4. Special Case: Cognitive Plugins (e.g., Personality)

Some plugins, like the **Personality Plugin**, provide **cognitive input** rather than external actions. These require a secondary, read-only integration path:

* **Action Path (MO \rightarrow Actuator):** The `update_personality_kb` tool is registered via the **External Gateway (P16)**, allowing the MO to *learn* new rules.
* **Context Path (Read-Only):** The **PAGI-Context-Engine (P8)** makes a direct, internal HTTP `GET` request to the deployed Personality Plugin's `GET /v1/rules` endpoint to fetch the rules and inject them into the LLM prompt.

This dual path ensures that all action is secured by the Gateway, while cognitive augmentation is tightly coupled to the prompt builder.

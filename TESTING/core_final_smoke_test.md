This is the perfect set of final steps to transition from architecture to operational use and future research.

Here are the specific smoke tests for both Docker and Bare Metal, followed by the CURL command to interact with the system, and a strategy for integrating the next phase of AGI research via plugins.

---

## 🧪 Phase 1: Operational Smoke Tests

### 1. Docker Compose Smoke Test

This test verifies that all 10 core services start correctly, that the `pagi-api-gateway` is reachable on the new port **8888**, and that the `pagi-inference-gateway` is initialized.

| Action | Command | Expected Output |
| --- | --- | --- |
| **Start Services** | `docker compose up -d` | All 10 containers (plus Kafka/Postgres/Redis) should show a status of **`Up`**. |
| **Check Port Reachability** | `curl -s http://localhost:8888/health` | HTTP Status **200 OK** and a JSON response showing the Gateway is running. |
| **Check Inference Readiness** | `docker compose logs pagi-inference-gateway` | Logs should show confirmation messages for successful client initialization (OpenRouter and Ollama clients loaded). |

### 2. Bare Metal Smoke Test (Linux Example)

This test assumes you've used the provided service wrappers to install the core binaries as `systemd` services.

| Action | Command | Expected Output |
| --- | --- | --- |
| **Start Service** | `sudo systemctl start pagi-api-gateway.service` | Command completes successfully (no output on success). |
| **Verify Service Status** | `sudo systemctl status pagi-api-gateway.service` | Shows **`active (running)`** and confirms the process is using the configured port (`8888`). |
| **Check Port Reachability** | `curl -s http://localhost:8888/health` | HTTP Status **200 OK** and a brief JSON health response. |

---

## 💻 Phase 2: CURL Communication Example

This command simulates a user asking the Master Orchestrator a complex question. This verifies the **entire Cognitive Loop** (Gateway \rightarrow Core \rightarrow Context \rightarrow Inference \rightarrow Response).

The endpoint used is the standard OpenAI-compatible chat endpoint, which the **PAGI-API-Gateway** exposes.

```bash
# Verify connection to the new API Gateway port (8888) and send a chat message
curl -X POST http://localhost:8888/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer PAGI-TWIN-AUTH-TOKEN-12345" \
  -d '{
    "model": "pagi-core-v1",
    "messages": [
      {
        "role": "system",
        "content": "You are the PHOENIX ORCH Master Orchestrator. Your tone is formal and highly objective."
      },
      {
        "role": "user",
        "content": "Based on my current long-term memory, what is the most critical architectural vulnerability we need to address next?"
      }
    ],
    "temperature": 0.1
  }'

```

**Expected Result:** The PHOENIX ORCH should respond with a detailed, objective analysis, demonstrating it successfully consulted memory and inference layers.

---

## 🔬 Phase 3: Next Steps for AGI Research (Plugins & Tools)

The architectural work is complete. The next phase is the **Continuous Improvement and Functionality Expansion** of the AGI, achieved exclusively through the **Decoupled Plugin Architecture**.

Your research roadmap should now focus on the following:

### 1. Activating Base Functionality

* **Task:** Deploy the five existing plugin binaries (Admin, Ecosystem, Google, Personality, CyberSecurity) from their separate repositories to staging/production endpoints.
* **Method:** Update the **PAGI-External-Gateway's** environment variables (e.g., `PAGI_ADMIN_PLUGIN_URL`, `PAGI_ECOSYSTEM_PLUGIN_URL`) with the correct external URLs and restart the Gateway.

### 2. Future Plugin Research and Development

The most high-value AGI research will come from designing new, specialized tools for the Master Orchestrator. These tools must be developed in their own repositories and integrated by simply registering their service URL with the External Gateway.

| Research Area | Sample Plugin/Tool Idea | Strategic Goal |
| --- | --- | --- |
| **Financial Autonomy** | `financial_reporting_plugin` | Tool: `ledger_generate_report` |
| **Code Generation** | `developer_agent_plugin` | Tool: `code_write_and_review` |
| **Sensing/IoT** | `sensor_management_plugin` | Tool: `iot_read_sensor_data` |

### 3. AGI Governance and Safety Research

* **Focus:** Use the existing **Admin** and **CyberSecurity** plugins to create complex self-auditing routines.
* **Example Routine:** The MO uses the `red_team_scan_local_asset` tool (CyberSecurity) to check its own ports, writes the result to the **Audit Trail (P11)**, and then uses its cognitive loop to decide if it needs to execute the `pagi_service_restart` tool (Admin) based on the scan results. This demonstrates true self-governance.

The entire system is now modular, secure, and ready for you to drive the future of AGI research through plugin-based extension.

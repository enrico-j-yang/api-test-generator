---
name: api-test-generator
description: Use when a user provides API documentation such as Swagger, OpenAPI, or markdown specs and wants generated API test scenarios, flowcharts, executable pytest scripts, webhook callback servers, or README test documentation for end-to-end service testing.
---

# API Test Generator

A skill for systematically generating comprehensive API test scenarios, executable test scripts, callback servers, and documentation from API documentation.

## Workflow Overview

```
Step 1: Collect API Documentation
    ↓
Step 2: Generate Test Scenario Matrix + Flowchart → Markdown Output
    ↓
Step 3: Configure Test Environment (ask user)
    ↓
Step 4: Implement Test Scripts + Callback Server + README
    ↓
Step 5: Verify with pytest --collect-only
```

---

## Step 1: Collect API Documentation

### Ask the user for input

Present this question to the user:

> **Please provide your API documentation:**
>
> Choose one of the following options:
> - **Option A**: Provide the file path to a markdown file containing API documentation
> - **Option B**: Paste the API documentation content directly here

Wait for the user's response before proceeding.

### Parse the documentation

Once you receive the API documentation, extract and organize:

1. **Endpoints**: List all API endpoints with their HTTP methods
2. **Parameters**: For each endpoint, identify:
   - Required parameters (path, query, body)
   - Optional parameters with default values
   - Parameter types and constraints
3. **Response codes**: Document all possible response codes and their meanings
4. **Dependencies**: Note any inter-endpoint dependencies (e.g., create → query → stop sequences)
5. **Authentication**: Identify auth requirements if specified
6. **Callback/Webhook**: Identify if API sends callbacks (callback_url, upload_url parameters)
7. **Async operations**: Identify endpoints with status polling (pending → processing → completed)

---

## Step 2: Generate Test Scenario Matrix

### Comprehensive test coverage

For each endpoint, generate test scenarios covering these dimensions:

| Dimension | Cases to Cover |
|-----------|----------------|
| **Error codes** | All documented error codes (400, 401, 403, 404, 409, 500, etc.) |
| **Required params** | Present with valid values, Present with invalid values, Missing entirely |
| **Optional params** | Present with valid values, Present with invalid values, Omitted (use default), Explicit default value |
| **Param values** | Valid reference values, Invalid values (wrong type, out of range, empty, null) |
| **Edge cases** | Boundary values, Unicode/special characters, Large payloads, Empty collections |
| **Business constraints** | State transitions (e.g., completed task cannot be stopped) |

### Test scenario matrix template

Generate a table for each endpoint:

```markdown
## Endpoint: {METHOD} {PATH}

### Scenario Matrix

| Scenario ID | Test Case | Parameters | Expected Status | Notes |
|-------------|-----------|------------|-----------------|-------|
| TC001 | Valid request with all params | All required + optional | 200 | Baseline success |
| TC002 | Valid request, minimal params | Only required params | 200 | Minimal valid input |
| TC003 | Missing required param {name} | Omit {param} | 400 | Required field validation |
| ... | ... | ... | ... | ... |
```

### Generate flowcharts

For each endpoint, create a Mermaid flowchart showing the decision tree of test scenarios.

For multi-endpoint workflows, create sequence flowcharts showing CRUD operations or async polling flows.

### Output to markdown file

Write the complete test scenario documentation:
- **Filename**: `{api-name}-test-scenarios.md`
- **Content structure**: Overview, matrices, flowcharts, workflow sequences, test priority

---

## Step 3: Configure Test Environment

### Ask the user for configuration

Present these questions:

> **Test script configuration:**
>
> 1. **Base URL**: What is the API service URL? (Use environment variable prefix like `DENOISE_API_BASE_URL` with default)
> 2. **Authentication**: What auth mechanism? (Bearer token, API key, etc. — specify env var names and defaults)
> 3. **Test framework**: pytest (recommended)
> 4. **Test script location**: Default `tests/` directory
>
> **If API has callbacks/webhooks:**
>
> 5. **Callback server**: Need to generate a callback server? (Yes/No)
>    - If Yes: What port? (default 3000)
>    - Save callbacks to JSON files? (Yes/No)
>    - Save uploads to files? (Yes/No)
>
> **For async workflow tests:**
>
> 6. **Specific test scenarios**: Provide actual test data URLs if available
>    - Example: "Use audio URL https://xxx.wav for scenario 1"

---

## Step 4: Implement Test Scripts

### Complete file structure

Generate test scripts following this structure:

```
tests/
├── config.py              # Environment variable configuration
├── conftest.py             # pytest fixtures (auth, task management)
├── callback_server.py      # Callback server (if API has callbacks)
├── test_{endpoint1}.py     # Tests for endpoint 1
├── test_{endpoint2}.py     # Tests for endpoint 2
├── test_workflow.py        # Workflow/async tests (if applicable)
├── requirements.txt        # Dependencies (pytest, requests, flask, pytest-html)
├── __init__.py
└── utils/
    ├── __init__.py
    ├── request_helper.py   # Request wrapper with logging
    └── data_generator.py   # Test data generators

README.md                   # Complete usage documentation
{api-name}-test-scenarios.md  # Test scenario documentation
```

### Configuration template (config.py)

```python
import os

# Environment variable naming pattern: {PREFIX}_API_{NAME}
BASE_URL = os.environ.get("{PREFIX}_API_BASE_URL", "https://staging.example.com")
AUTH_TOKEN = os.environ.get("{PREFIX}_API_TOKEN", "default_token")
TIMEOUT = int(os.environ.get("{PREFIX}_API_TIMEOUT", "30"))

# Callback server config (if applicable)
CALLBACK_URL = os.environ.get("{PREFIX}_CALLBACK_URL", "")
UPLOAD_URL = os.environ.get("{PREFIX}_UPLOAD_URL", "")
CALLBACK_SERVER_PORT = int(os.environ.get("{PREFIX}_CALLBACK_PORT", "3000"))

# Test data directories
TEST_DATA_DIR = os.environ.get("{PREFIX}_TEST_DATA_DIR", "test_data")
CALLBACK_DATA_DIR = os.path.join(TEST_DATA_DIR, "callbacks")
UPLOAD_DATA_DIR = os.path.join(TEST_DATA_DIR, "uploads")

# API endpoints
ENDPOINTS = {
    "create": "/v1/{resource}/create",
    "query": "/v1/{resource}/query",
    ...
}

def get_headers(content_type: str = "application/json") -> dict:
    return {
        "Content-Type": content_type,
        "Authorization": f"Bearer {AUTH_TOKEN}",
    }
```

### Callback Server (callback_server.py)

**Generate when API has callback_url or upload_url parameters.**

Key features:
- Flask server receiving POST callbacks
- Save callback JSON to `test_data/callbacks/`
- Save uploaded files to `test_data/uploads/`
- Health check endpoint `/health`
- List endpoints `/callback/list`, `/upload/list`
- Configurable port via environment variable

```python
# Key endpoints:
# POST /callback  → Save JSON, return {"code": 200}
# POST /upload    → Save file, return {"code": 200, "url": "..."}
# GET  /health    → Health check
# GET  /callback/list → List all callbacks
# GET  /upload/list   → List all uploads
```

### Workflow Test Template (test_workflow.py)

**Generate when API has async operations with status polling.**

Key patterns:
- `_poll_until_completed()` method with configurable max_wait and poll_interval
- File count verification before/after operations
- Specific test scenarios with actual test URLs
- State transition verification

```python
class TestWorkflowFullCompletion:
    
    MAX_WAIT_TIME = 300  # 5 minutes
    POLL_INTERVAL = 5    # 5 seconds
    
    def _count_files_in_dir(self, directory: str, extension: str = None) -> int:
        """Count files in directory"""
        ...
    
    def _poll_until_completed(self, task_id: str, headers: dict, max_wait: int) -> dict:
        """Poll query endpoint until completed"""
        ...
    
    def test_scenario_1_with_actual_data(self, valid_headers):
        """Full workflow: record counts → create → poll → verify files"""
        # 1. Record initial file counts
        # 2. Create task with callback_url/upload_url
        # 3. Poll until completed (max 5 min)
        # 4. Verify callback JSON saved
        # 5. Verify upload file saved
        # 6. Validate callback content (task_id, status, result_url)
```

### README.md Template

**Always generate README.md with:**

1. **Quick Start**: Install, config env vars, run callback server, run tests
2. **Directory Structure**: Complete file tree
3. **Configuration**: Environment variable table
4. **Running Tests**:
   - Basic: `pytest tests/ -v`
   - By marker: `pytest tests/ -v -m smoke`
   - HTML report: `pytest tests/ -v --html=reports/report.html --self-contained-html`
5. **Callback Server**: Startup instructions (foreground, background, custom port)
6. **Test Scenarios**: Table of specific test cases with URLs
7. **Test Statistics**: Count of test cases per file
8. **FAQ**: Common issues and solutions

---

## Step 5: Verify Test Collection

After generating all files, verify test collection:

```bash
# Verify tests are collected correctly
pytest tests/ -v --collect-only
```

Report the total count of collected tests and any import errors.

---

## Summary

The skill outputs:

1. **Test documentation**: `{api-name}-test-scenarios.md` — matrices and flowcharts
2. **Test scripts**: `tests/` directory — executable pytest scripts
3. **Callback server**: `tests/callback_server.py` — if API has callbacks
4. **Usage documentation**: `README.md` — complete running instructions

### Key improvements from real-world testing:

- **Environment variable pattern**: Prefix-based naming (`{PREFIX}_API_*`)
- **Callback server generation**: For webhook-based APIs
- **Async workflow tests**: Polling logic with file verification
- **HTML report support**: `--html` parameter in README
- **Direct pytest commands**: Not wrapped in pdm/other tools
- **Specific test scenarios**: Actual test URLs, not placeholders

---

## Test Markers

Generated tests use these pytest markers:

| Marker | Usage |
|--------|-------|
| `smoke` | P0 critical tests |
| `auth` | Authentication tests |
| `validation` | Parameter validation |
| `error` | Error scenario tests |
| `workflow` | Multi-step workflow tests |
| `edge` | Boundary condition tests |

---

## Dependencies (requirements.txt)

```
pytest>=7.0.0
requests>=2.28.0
flask>=2.0.0
python-dotenv>=1.0.0
pytest-html>=3.0.0
```

# API Test Generator Skill

[![npx skills add](https://img.shields.io/badge/npx%20skills%20add-enrico--j--yang%2Fapi--test--generator-000000?logo=npm&logoColor=white)](https://skills.sh)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

A Claude Code skill for systematically generating comprehensive API test suites from API documentation.

## Installation

### Via `npx skills` (works for Claude Code, Codex, Cursor, Gemini CLI, and 60+ other agents)

```bash
npx skills add enrico-j-yang/api-test-generator
```

### As a Claude Code plugin (loaded with namespace)

```bash
# One-off load for a single session
claude --plugin-dir /path/to/api-test-generator

# Or install from this repo into Claude Code's plugin manager
/plugin install enrico-j-yang/api-test-generator
```

Once installed as a plugin, the skill is invoked as `/api-test-generator:api-test-generator` (namespaced) or activates automatically when the model detects API documentation in the conversation.

### Manual install

Clone the repo into your agent's skills directory:

```bash
git clone https://github.com/enrico-j-yang/api-test-generator ~/.claude/skills/api-test-generator
# or for the broader .agents path
git clone https://github.com/enrico-j-yang/api-test-generator ~/.agents/skills/api-test-generator
```

## What This Skill Does

This skill transforms API documentation (Swagger, OpenAPI, or markdown specs) into:

1. **Test Scenario Matrix** - Comprehensive test coverage documentation with traceable scenario IDs
2. **Executable pytest Scripts** - Ready-to-run test files with fixtures, helpers, and assertions
3. **HTTPS Callback Server** - For webhook/callback testing with SSL certificate generation
4. **README Documentation** - Complete usage instructions with troubleshooting

## When to Use

Trigger this skill when:
- User provides API documentation and wants generated test scenarios
- User mentions "API test suite", "pytest tests", "API testing"
- User asks for webhook callback server generation
- User wants flowcharts or test documentation for APIs

## Skill Workflow

```
Step 1: Collect API Documentation (skip if already detailed)
    ↓
Step 2: Generate Test Scenario Matrix + Flowchart (skip if user specifies)
    ↓
Step 3: Configure Test Environment (always ask)
    ↓
Step 4: Implement Test Scripts + Callback Server + README
    ↓
Step 5: Verify with compileall and pytest --collect-only
    ↓
Step 6: Clean Output (remove command traces)
```

**Adaptive Workflow**: The skill adjusts based on input detail level:
- **Detailed inputs**: Skip Step 1-2, focus on deep edge case exploration
- **Sparse inputs**: Follow full workflow to structure requirements

## Output Structure

```
tests/
├── config.py              # Environment variable configuration
├── conftest.py             # pytest fixtures (auth, cleanup, capture server)
├── test_{endpoint}.py      # Endpoint-specific tests
├── test_workflow.py        # Integration/workflow tests
├── requirements.txt        # Dependencies
└── utils/
    ├── request_helper.py   # API client with logging
    └── data_generator.py   # Test data generators

README.md                   # Usage documentation
{api-name}-test-scenarios.md  # Scenario matrix
```

## Key Features

### Scenario ID Traceability
Every test case has a traceable ID (e.g., `CREATE-001`, `VPR-GRPS-CRTE-002`) that appears in:
- Test function names
- pytest.param IDs
- Documentation matrix

### HTTPS Callback Server
For APIs with webhooks:
- Self-signed certificate generation using openssl
- SSL context wrapping for HTTPS
- Callback and upload file persistence
- Wait helpers for async assertions

### Deep Edge Case Coverage
Required edge cases tested:
- Numeric boundaries (min-1, min, min+1, max-1, max, max+1)
- String limits (empty, whitespace, max length, Unicode)
- Array limits (empty, single, max items, over max)
- Enum validation (all values, invalid, null)
- State transitions (valid, invalid, concurrent)

### Clean Output
All outputs are professional without shell command traces or execution logs.

## Test Markers

| Marker | Usage |
|--------|-------|
| `smoke` | P0 critical tests |
| `auth` | Authentication tests |
| `validation` | Parameter validation |
| `error` | Error scenario tests |
| `workflow` | Multi-step integration tests |
| `edge` | Boundary condition tests |

## Dependencies

```
pytest>=7.0.0
requests>=2.28.0
python-dotenv>=1.0.0
pytest-html>=3.0.0
```

## Example Usage

**Input**: API documentation (markdown or OpenAPI spec)

**Output**: Complete pytest test suite with:
- 50-100+ test cases per API
- Scenario documentation with flowcharts
- HTTPS callback server (if webhooks present)
- README with running instructions

## Evaluation Results

| Metric | Before Improvement | After Improvement |
|--------|-------------------|-------------------|
| Skill win rate | 2/3 cases | **3/3 cases** |
| Material differences | 1/3 cases | **3/3 cases** |
| Average test count | ~48 | **74** |

## Skill Location

```
C:\Users\Administrator\.agents\skills\api-test-generator\
├── SKILL.md      # Skill instructions (430 lines)
├── README.md     # This documentation
└── api-test-generator-workspace/
    └── iteration-1/  # Evaluation outputs
```

## Version History

- **v1.0** - Initial skill creation
- **v1.1** - Added adaptive workflow, output cleanliness, HTTPS callback server, deep edge cases

---

**Author**: Claude Code Skill Creator  
**Last Updated**: 2026-06-25
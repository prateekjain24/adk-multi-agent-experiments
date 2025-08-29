# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an experimental Python project for creating multi-agent workflows using Google ADK (Application Development Kit). The project uses `uv` as the package manager and requires Python 3.13 or later.

**For comprehensive ADK documentation, see: `google_adk_knowledge.md`**

**Note**: For the latest Google ADK updates and documentation, use Perplexity search with queries like "Google ADK latest features" or check the official docs at https://google.github.io/adk-docs/

## Project Structure

```
adk_test/
├── agents/                  # Production-ready agent examples
│   ├── customer-service/    # Example: Multi-tool customer service agent
│   ├── llm-auditor/        # Example: Sequential agent with critic-reviser pattern
│   ├── data-science/       # Example: Complex multi-agent workflow
│   └── ...                 # Many more examples for reference
├── workflows/               # Multi-agent workflow configurations
│   └── (place workflow definitions here)
├── tools/                   # Custom tools and utilities
│   └── (place custom tools here)
├── examples/                # Example implementations
│   └── (place example scripts here)
├── google_adk_knowledge.md  # Comprehensive ADK reference guide
├── pyproject.toml          # Project configuration
└── uv.lock                 # Locked dependencies
```

### Agent Directory Structure Pattern
Each agent in `agents/` follows this structure:
```
agent-name/
├── agent_name/              # Python package
│   ├── __init__.py
│   ├── agent.py            # Main agent definition (exports root_agent)
│   ├── prompt.py           # Prompt templates
│   ├── config.py           # Configuration (if needed)
│   ├── sub_agents/         # Sub-agents for multi-agent systems
│   ├── tools/              # Custom tools
│   └── shared_libraries/   # Shared utilities
├── deployment/
│   ├── deploy.py           # Vertex AI deployment script
│   └── test_deployment.py  # Deployment tests
├── eval/
│   ├── test_eval.py        # Evaluation harness using AgentEvaluator
│   └── data/               # Test cases (.test.json, .evalset.json)
├── tests/                  # Unit tests
├── pyproject.toml          # Agent-specific dependencies
└── README.md               # Agent documentation
```

## Development Commands

### Package Management
- Install dependencies: `uv sync`
- Add a dependency: `uv add <package>`
- Remove a dependency: `uv remove <package>`
- Update dependencies: `uv lock --upgrade-package <package>`

### Running Code
- Run main script: `uv run python main.py`
- Run specific workflow: `uv run python workflows/<workflow_name>.py`
- Run example: `uv run python examples/<example_name>.py`
- Run agent directly: `uv run python -m agents.<agent-name>.<agent_name>.agent`

### Testing
- Run tests for an agent: `cd agents/<agent-name> && uv run pytest`
- Run evaluation: `cd agents/<agent-name> && uv run python eval/test_eval.py`
- Run all tests with async support: `uv run pytest -v --asyncio-mode=auto`

### Linting & Type Checking
**Note**: Check each agent's pyproject.toml for specific linting commands. Common patterns:
- Python linting: `uv run ruff check .`
- Python formatting: `uv run ruff format .`
- Type checking: `uv run mypy .`

### Deployment to Vertex AI
```bash
cd agents/<agent-name>
uv run python deployment/deploy.py \
  --project_id=<YOUR_PROJECT> \
  --location=<REGION> \
  --bucket=<GCS_BUCKET> \
  --create
```

## Architecture & Patterns

### Agent Types

1. **SequentialAgent** - Executes sub-agents in order
```python
from google.adk.agents import SequentialAgent
agent = SequentialAgent(
    name='agent_name',
    description='...',
    sub_agents=[agent1, agent2]
)
```

2. **WorkflowAgent** - Structured workflows with patterns
```python
from google.adk.agents import WorkflowAgent
agent = WorkflowAgent(
    name='workflow',
    pattern='sequential',  # or 'parallel', 'loop'
    agents=[...]
)
```

3. **Custom Agent** - Using base Agent class
```python
from google.adk import Agent
agent = Agent(
    name='custom',
    model='gemini-2.0-flash',
    instruction='...',
    tools=[...]
)
```

### Common Patterns

**Tool Definition Pattern**:
```python
from google.adk.tools import tool

@tool
def my_tool(param: str) -> str:
    """Tool description."""
    return result
```

**Session Management**:
```python
from google.adk import Session
from google.adk.sessions import InMemorySessionService

session_service = InMemorySessionService()
session = Session(agent=agent, session_service=session_service)
```

**Evaluation Pattern**:
```python
from google.adk.evaluation import AgentEvaluator

await AgentEvaluator.evaluate(
    "agent_name",
    "path/to/test/data",
    num_runs=5
)
```

**Environment Configuration**:
- Use `.env` files for API keys and configuration
- Load with `dotenv.load_dotenv()` in tests and deployments

## Quick ADK Reference

### Common Agent Patterns
- **LLM Agent**: For flexible, reasoning tasks → see `google_adk_knowledge.md#agent-types--architecture`
- **Workflow Agent**: For structured processes → see `google_adk_knowledge.md#multi-agent-systems`
- **Custom Tools**: For specialized functionality → see `google_adk_knowledge.md#tools-integration`

### Key Imports
```python
from google.adk import Agent, Session
from google.adk.agents import WorkflowAgent, SequentialAgent
from google.adk.tools import tool, google_search
from google.adk.sessions import InMemorySessionService
from google.adk.evaluation import AgentEvaluator
from vertexai.preview.reasoning_engines import AdkApp
```

## Experimental Workflows

This project is designed for experimenting with various multi-agent patterns:
- Coordinator/Dispatcher patterns
- Sequential pipelines
- Parallel processing
- Hierarchical task decomposition
- Generator-Critic loops

Place each experiment in the appropriate directory and document its purpose.

## Key Dependencies

- `google-adk>=1.13.0` - Google Application Development Kit library
- `pytest` & `pytest-asyncio` - Testing framework
- `python-dotenv` - Environment configuration
- `vertexai` - For deploying to Google Cloud
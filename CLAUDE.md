# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an experimental Python project for creating multi-agent workflows using Google ADK (Application Development Kit). The project uses `uv` as the package manager and requires Python 3.13 or later.

**For comprehensive ADK documentation, see: `google_adk_knowledge.md`**

**Note**: For the latest Google ADK updates and documentation, use Perplexity search with queries like "Google ADK latest features" or check the official docs at https://google.github.io/adk-docs/

## Project Structure

```
adk_test/
├── agents/                  # Individual agent definitions
│   └── (place agent modules here)
├── workflows/               # Multi-agent workflow configurations
│   └── (place workflow definitions here)
├── tools/                   # Custom tools and utilities
│   └── (place custom tools here)
├── examples/                # Example implementations
│   └── (place example scripts here)
├── google_adk_knowledge.md  # Comprehensive ADK reference guide
├── main.py                  # Entry point
├── pyproject.toml          # Project configuration
└── uv.lock                 # Locked dependencies
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

## Quick ADK Reference

### Common Agent Patterns
- **LLM Agent**: For flexible, reasoning tasks → see `google_adk_knowledge.md#agent-types--architecture`
- **Workflow Agent**: For structured processes → see `google_adk_knowledge.md#multi-agent-systems`
- **Custom Tools**: For specialized functionality → see `google_adk_knowledge.md#tools-integration`

### Key Imports
```python
from google.adk import Agent, Session
from google.adk.agents import WorkflowAgent
from google.adk.tools import tool, google_search
from google.adk.sessions import InMemorySessionService
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
# Google Agent Development Kit (ADK) Knowledge Repository

## Table of Contents
1. [Overview](#overview)
2. [Agent Types & Architecture](#agent-types--architecture)
3. [Multi-Agent Systems](#multi-agent-systems)
4. [Tools Integration](#tools-integration)
5. [Session Management](#session-management)
6. [Memory Management](#memory-management)
7. [State Management](#state-management)
8. [Artifacts](#artifacts)
9. [Context Management](#context-management)
10. [Model Support](#model-support)
11. [Grounding & Search](#grounding--search)
12. [Code Examples](#code-examples)
13. [Best Practices](#best-practices)

## Overview

Google Agent Development Kit (ADK) is a comprehensive framework for building intelligent, autonomous agent-based applications. It provides a flexible architecture for creating agents that can reason, plan, use tools, and collaborate to solve complex problems.

### Key Features
- Multiple agent types (LLM, Workflow, Custom)
- Built-in and third-party tool integration
- Session and state management
- Memory persistence
- Multi-model support (Gemini, Anthropic, OpenAI, etc.)
- Cloud-native with Vertex AI integration

## Agent Types & Architecture

### 1. LLM Agents
- **Purpose**: Flexible, language-centric tasks requiring dynamic reasoning
- **Core Engine**: Large Language Models (Gemini, Claude, etc.)
- **Capabilities**: 
  - Natural language understanding
  - Dynamic planning and reasoning
  - Tool utilization
  - Context-aware responses

```python
from google.adk.agents import Agent

llm_agent = Agent(
    name="assistant",
    model="gemini-2.0-flash",
    instruction="You are a helpful assistant",
    tools=[custom_tool]
)
```

### 2. Workflow Agents
- **Purpose**: Structured processes with predictable execution
- **Patterns**: Sequential, Parallel, Loop
- **Use Cases**:
  - Deterministic task flows
  - Process orchestration
  - Batch operations

```python
from google.adk.agents import WorkflowAgent

workflow = WorkflowAgent(
    name="data_pipeline",
    pattern="sequential",
    agents=[agent1, agent2, agent3]
)
```

### 3. Custom Agents
- **Purpose**: Specialized logic and integrations
- **Implementation**: Extend `BaseAgent` class
- **Use Cases**:
  - Unique operational requirements
  - Legacy system integration
  - Custom business logic

## Multi-Agent Systems

### Composition Primitives

#### 1. Agent Hierarchy
- Parent-child relationships
- Single parent constraint
- Structured communication channels

#### 2. Interaction Mechanisms

**a. Shared Session State**
```python
# Agent A writes to state
context.state["user_preference"] = "dark_mode"

# Agent B reads from state
preference = context.state.get("user_preference")
```

**b. LLM-Driven Delegation**
- Dynamic task routing
- Natural language understanding
- Automatic agent selection

**c. Explicit Invocation (AgentTool)**
```python
from google.adk.tools import AgentTool

agent_tool = AgentTool(
    agent=specialized_agent,
    name="specialist",
    description="Handles specialized tasks"
)
```

### Common Patterns
1. **Coordinator/Dispatcher**: Central agent routes tasks
2. **Sequential Pipeline**: Linear task processing
3. **Parallel Fan-Out/Gather**: Concurrent execution
4. **Hierarchical Decomposition**: Task breakdown
5. **Generator-Critic**: Review and refinement
6. **Iterative Refinement**: Progressive improvement
7. **Human-in-the-Loop**: User interaction points

## Tools Integration

### Built-in Tools

#### 1. Google Search
```python
from google.adk.tools import google_search

search_agent = Agent(
    model='gemini-2.0-flash',
    tools=[google_search]
)
```

#### 2. Code Execution
- Supports calculations and data manipulation
- Compatible with Gemini 2 models

#### 3. Vertex AI Search
```python
from google.adk.tools import vertex_ai_search

search_tool = vertex_ai_search(
    data_store_id="your-datastore-id"
)
```

#### 4. BigQuery Tools
- List datasets/tables
- Get metadata
- Execute SQL queries

**Important Limitations**:
- Only one built-in tool per agent
- Cannot mix built-in tools with other types
- Not available in sub-agents

### Third-Party Tools

#### LangChain Integration
```python
from google.adk.tools import LangchainTool
from langchain_community.tools.tavily import TavilySearchResults

tavily_tool = TavilySearchResults(api_key="your-api-key")
wrapped_tool = LangchainTool(tavily_tool)

agent = Agent(
    model="gemini-2.0-flash",
    tools=[wrapped_tool]
)
```

#### CrewAI Integration
```python
from google.adk.tools import CrewaiTool
from crewai_tools import SerperDevTool

serper_tool = SerperDevTool(api_key="your-api-key")
wrapped_tool = CrewaiTool(
    tool=serper_tool,
    name="web_search",
    description="Search the web"
)
```

### Custom Tools
```python
from google.adk.tools import tool

@tool
def calculate_discount(price: float, discount_percent: float) -> float:
    """Calculate discounted price"""
    return price * (1 - discount_percent / 100)

agent = Agent(
    model="gemini-2.0-flash",
    tools=[calculate_discount]
)
```

## Session Management

### Session Components
- **id**: Unique conversation identifier
- **app_name**: Application identifier
- **userId**: User association
- **events**: Interaction history
- **state**: Temporary data storage
- **lastUpdateTime**: Activity timestamp

### Session Services

#### 1. InMemorySessionService
```python
from google.adk.sessions import InMemorySessionService

session_service = InMemorySessionService()
```
- Development/testing
- No persistence
- Fast access

#### 2. VertexAiSessionService
```python
from google.adk.sessions import VertexAiSessionService

session_service = VertexAiSessionService(
    project_id="your-project",
    location="us-central1"
)
```
- Production-ready
- Persistent storage
- Scalable

#### 3. DatabaseSessionService (Python only)
```python
from google.adk.sessions import DatabaseSessionService

session_service = DatabaseSessionService(
    connection_string="postgresql://..."
)
```
- Relational database storage
- Custom persistence options

## Memory Management

### Memory Services

#### 1. InMemoryMemoryService
```python
from google.adk.memory import InMemoryMemoryService

memory_service = InMemoryMemoryService()
```
- Basic keyword matching
- No persistence
- Prototyping/testing

#### 2. VertexAiMemoryBankService
```python
from google.adk.memory import VertexAiMemoryBankService

memory_service = VertexAiMemoryBankService(
    project_id="your-project",
    location="us-central1",
    corpus_id="your-corpus"
)
```
- Sophisticated semantic search
- Automatic memory generation
- Persistent storage
- Production-ready

### Memory Workflow
1. Session captures events
2. Data ingested into memory store
3. Queries retrieve relevant context
4. Agents access historical information

## State Management

### State Characteristics
- Key-value pairs
- Serializable data
- Mutable during conversation
- Prefix support: `user:`, `app:`, `temp:`

### State Management Methods

#### 1. Output Key
```python
agent = Agent(
    model="gemini-2.0-flash",
    output_key="conversation_summary"
)
```

#### 2. Event Actions
```python
from google.adk.events import EventActions

actions = EventActions(
    state_delta={"counter": 1}
)
```

#### 3. Context Modification
```python
def custom_tool(context: ToolContext):
    context.state["task_complete"] = True
```

### Best Practices
- Store only essential data
- Use descriptive keys
- Prefer basic types
- Avoid deep nesting
- Update through `append_event`

## Artifacts

### Definition
- Named, versioned binary data
- Images, audio, PDFs, reports
- Persist across sessions or single interaction

### Storage Services

#### 1. InMemoryArtifactService
```python
from google.adk.artifacts import InMemoryArtifactService

artifact_service = InMemoryArtifactService()
```

#### 2. GcsArtifactService
```python
from google.adk.artifacts import GcsArtifactService

artifact_service = GcsArtifactService(
    bucket_name="your-bucket"
)
```

### Usage
```python
# Save artifact
context.save_artifact(
    filename="report.pdf",
    content=pdf_bytes,
    mime_type="application/pdf"
)

# Load artifact
artifact = context.load_artifact("report.pdf")

# List artifacts
artifacts = context.list_artifacts()
```

## Context Management

### Context Types

#### 1. InvocationContext
- Most comprehensive
- Full state access
- All services available

#### 2. ReadonlyContext
- Read-only access
- Basic information
- Limited operations

#### 3. CallbackContext
- Lifecycle callbacks
- State read/write
- Artifact interaction

#### 4. ToolContext
- Tool functions
- Extends CallbackContext
- Authentication methods
- Memory search

### Context Usage
```python
def my_tool(context: ToolContext):
    # Access state
    user_id = context.state.get("user:id")
    
    # Save artifact
    context.save_artifact("data.json", json_bytes)
    
    # Search memory
    memories = context.search_memory("previous interaction")
```

## Model Support

### Google Gemini Models
```python
# Via Google AI Studio
agent = Agent(
    model="gemini-2.0-flash",
    api_key="your-api-key"
)

# Via Vertex AI
agent = Agent(
    model="gemini-2.0-flash",
    project_id="your-project",
    location="us-central1"
)
```

### Anthropic Models
```python
# Direct integration (Java)
agent = Agent(
    model="claude-3-sonnet",
    api_key="your-api-key"
)

# Via Vertex AI
agent = Agent(
    model="claude-3-sonnet@vertex",
    project_id="your-project"
)
```

### Open/Local Models
```python
# Via Ollama
agent = Agent(
    model="ollama/llama3",
    base_url="http://localhost:11434"
)

# Via vLLM
agent = Agent(
    model="local/model",
    base_url="http://localhost:8000"
)
```

## Grounding & Search

### Google Search Grounding
```python
from google.adk.tools import google_search

grounded_agent = Agent(
    name="search_agent",
    model="gemini-2.0-flash",
    instruction="Use search for current information",
    tools=[google_search]
)

# Agent automatically uses search when needed
response = grounded_agent.invoke("What's the weather today?")
```

### Benefits
- Real-time information access
- Source attribution
- Fact verification
- Current data beyond training cutoff

## Code Examples

### Basic Agent
```python
from google.adk import Agent, Session

# Create agent
agent = Agent(
    name="assistant",
    model="gemini-2.0-flash",
    instruction="You are a helpful assistant"
)

# Create session
session = Session()

# Invoke agent
response = agent.invoke(
    session=session,
    user_content="Hello, how can you help?"
)
```

### Multi-Agent System
```python
from google.adk import Agent, WorkflowAgent

# Create specialized agents
researcher = Agent(
    name="researcher",
    model="gemini-2.0-flash",
    instruction="Research and gather information",
    tools=[google_search]
)

writer = Agent(
    name="writer",
    model="gemini-2.0-flash",
    instruction="Write clear, concise content"
)

# Create workflow
pipeline = WorkflowAgent(
    name="content_pipeline",
    pattern="sequential",
    agents=[researcher, writer]
)

# Execute pipeline
result = pipeline.invoke(
    session=session,
    user_content="Write an article about AI"
)
```

### Tool with State Management
```python
from google.adk.tools import tool
from google.adk.context import ToolContext

@tool
def track_progress(context: ToolContext, task: str, status: str):
    """Track task progress"""
    # Update state
    tasks = context.state.get("tasks", {})
    tasks[task] = status
    context.state["tasks"] = tasks
    
    # Save artifact if complete
    if status == "complete":
        report = generate_report(tasks)
        context.save_artifact(
            "progress_report.txt",
            report.encode()
        )
    
    return f"Task '{task}' marked as {status}"
```

## Best Practices

### Agent Design
1. **Single Responsibility**: Each agent should have a clear, focused purpose
2. **Clear Instructions**: Provide specific, actionable instructions
3. **Appropriate Models**: Match model capabilities to task requirements
4. **Tool Selection**: Choose tools that align with agent objectives

### State Management
1. **Minimal Storage**: Store only essential information
2. **Clear Naming**: Use descriptive, consistent key names
3. **Proper Prefixes**: Use `user:`, `app:`, `temp:` appropriately
4. **Avoid Direct Modification**: Use framework methods

### Memory Usage
1. **Strategic Ingestion**: Store meaningful interactions
2. **Semantic Search**: Leverage advanced search capabilities
3. **Cleanup Strategies**: Implement memory pruning
4. **Privacy Considerations**: Handle sensitive data appropriately

### Performance Optimization
1. **Batch Operations**: Use parallel agents when possible
2. **Caching**: Leverage artifacts for repeated data
3. **Model Selection**: Balance capability with cost/speed
4. **Session Persistence**: Choose appropriate storage backend

### Error Handling
1. **Graceful Degradation**: Handle tool/service failures
2. **Retry Logic**: Implement appropriate retry strategies
3. **Logging**: Track agent interactions and errors
4. **User Feedback**: Provide clear error messages

### Security
1. **API Key Management**: Use environment variables
2. **Authentication**: Implement proper auth flows
3. **Data Privacy**: Handle PII appropriately
4. **Access Control**: Limit agent permissions

## Quick Reference

### Installation
```bash
pip install google-adk
```

### Basic Setup
```python
import os
from google.adk import Agent, Session
from google.adk.sessions import VertexAiSessionService

# Configure
os.environ["GOOGLE_API_KEY"] = "your-key"

# Initialize
session_service = VertexAiSessionService()
agent = Agent(
    model="gemini-2.0-flash",
    instruction="Your instructions"
)
session = Session(service=session_service)

# Use
response = agent.invoke(session, "User input")
```

### Common Patterns
- **Stateful Conversation**: Use session state
- **Multi-Step Tasks**: Use workflow agents
- **Information Retrieval**: Use search grounding
- **File Generation**: Use artifacts
- **Long-Term Memory**: Use memory banks
- **Tool Integration**: Use built-in or wrapped tools
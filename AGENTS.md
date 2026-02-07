# LangGraph & LangChain Learning Workspace

This project is for studying the Udemy course: **"Learn to build real world AI Agents, multi agent workflows, and autonomous apps with LangGraph and LangChain"**.

## Project Structure

```
.
├── 1-Pydantic/           # Section 1: Pydantic fundamentals
├── AGENTS.md             # This file - learning context for Kimi
├── main.py               # Main entry point for experiments
├── pyproject.toml        # Python dependencies
└── uv.lock               # Locked dependencies
```

## Dependencies

Key packages (managed with uv):
- `langgraph>=1.0.7` - Graph-based agent workflows
- `langsmith>=0.6.9` - Observability and tracing
- `pydantic>=2.12.5` - Data validation and settings
- `pandas>=3.0.0` - Data manipulation

## Course Topics Covered

1. **Pydantic Fundamentals** - Data validation and schema definition
2. **LangChain Basics** - Chains, prompts, models
3. **LangGraph** - State machines, nodes, edges for multi-agent systems
4. **Multi-Agent Workflows** - Agent orchestration and communication
5. **Autonomous Apps** - Building self-directed AI applications

## How Kimi Can Help

When working in this project, Kimi should:

1. **Explain concepts** - Break down LangGraph/LangChain patterns clearly
2. **Review code** - Check implementations against course best practices
3. **Debug issues** - Help troubleshoot agent workflows and state management
4. **Suggest improvements** - Offer cleaner or more idiomatic approaches
5. **Create exercises** - Generate practice problems based on course topics

## Common Patterns

### LangGraph State Management
```python
from typing import TypedDict
from langgraph.graph import StateGraph

class AgentState(TypedDict):
    messages: list
    next_step: str
```

### Agent Node Pattern
```python
def agent_node(state: AgentState) -> AgentState:
    # Process state
    # Return updated state
    return {"messages": [...], "next_step": "..."}
```

### Conditional Edge Pattern
```python
from langgraph.graph import END

def should_continue(state: AgentState) -> str:
    if condition:
        return "next_node"
    return END
```

## Running Code

Use `uv run` to execute Python files with the project environment:

```bash
uv run main.py
uv run python script.py
uv run jupyter lab  # if notebooks are used
```

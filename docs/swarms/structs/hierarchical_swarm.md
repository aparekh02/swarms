# `HierarchicalSwarm`

The `HierarchicalSwarm` is a sophisticated multi-agent orchestration system that implements a hierarchical workflow pattern. It consists of a director agent that coordinates and distributes tasks to specialized worker agents, creating a structured approach to complex problem-solving.

## Overview

The Hierarchical Swarm follows a clear workflow pattern:

1. **Task Reception**: User provides a task to the swarm
2. **Planning**: Director creates a comprehensive plan and distributes orders to agents
3. **Execution**: Individual agents execute their assigned tasks (with optional real-time streaming)
4. **Feedback Loop**: Director evaluates results and issues new orders if needed (up to `max_loops`)
5. **Context Preservation**: All conversation history and context is maintained throughout the process

## Architecture

```mermaid
graph TD
    A[User Task] --> B[Director Agent]
    B --> C[Create Plan & Orders]
    C --> D[Distribute to Agents]
    D --> E[Agent 1]
    D --> F[Agent 2]
    D --> G[Agent N]
    E --> H[Execute Task]
    F --> H
    G --> H
    H --> I[Report Results]
    I --> J[Director Evaluation]
    J --> K{More Loops?}
    K -->|Yes| C
    K -->|No| L[Final Output]
```

## Key Features

| Feature                      | Description                                                                                   |
|------------------------------|-----------------------------------------------------------------------------------------------|
| **Hierarchical Coordination**| Director agent orchestrates all operations                                                    |
| **Specialized Agents**       | Each agent has specific expertise and responsibilities                                       |
| **Iterative Refinement**     | Multiple feedback loops for improved results                                                 |
| **Context Preservation**     | Full conversation history maintained                                                         |
| **Flexible Output Formats**  | Support for various output types (dict, str, list)                                           |
| **Comprehensive Logging**    | Detailed logging for debugging and monitoring                                                |
| **Live Streaming**           | Real-time streaming callbacks for monitoring agent outputs                                   |
| **Token-by-Token Updates**   | Watch text formation in real-time as agents generate responses                               |

## `HierarchicalSwarm` Constructor

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | `str` | `"HierarchicalAgentSwarm"` | The name of the swarm instance |
| `description` | `str` | `"Distributed task swarm"` | Brief description of the swarm's functionality |
| `director` | `Optional[Union[Agent, Callable, Any]]` | `None` | The director agent that orchestrates tasks |
| `agents` | `List[Union[Agent, Callable, Any]]` | `None` | List of worker agents in the swarm |
| `max_loops` | `int` | `1` | Maximum number of feedback loops between director and agents |
| `output_type` | `OutputType` | `"dict-all-except-first"` | Format for output (dict, str, list) |
| `feedback_director_model_name` | `str` | `"gpt-4o-mini"` | Model name for feedback director |
| `director_name` | `str` | `"Director"` | Name of the director agent |
| `director_model_name` | `str` | `"gpt-4o-mini"` | Model name for the director agent |
| `verbose` | `bool` | `False` | Enable detailed logging |
| `add_collaboration_prompt` | `bool` | `True` | Add collaboration prompts to agents |
| `planning_director_agent` | `Optional[Union[Agent, Callable, Any]]` | `None` | Optional planning agent for enhanced planning |

## Core Methods

### `run(task, img=None, streaming_callback=None, *args, **kwargs)`

Executes the hierarchical swarm for a specified number of feedback loops, processing the task through multiple iterations for refinement and improvement.

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `task` | `str` | **Required** | The initial task to be processed by the swarm |
| `img` | `str` | `None` | Optional image input for the agents |
| `streaming_callback` | `Callable[[str, str, bool], None]` | `None` | Optional callback for real-time streaming of agent outputs |
| `*args` | `Any` | - | Additional positional arguments |
| `**kwargs` | `Any` | - | Additional keyword arguments |

#### Returns

| Type | Description |
|------|-------------|
| `Any` | The formatted conversation history as output based on `output_type` |

#### Example

```python
from swarms import Agent
from swarms.structs.hiearchical_swarm import HierarchicalSwarm

# Create specialized agents
research_agent = Agent(
    agent_name="Research-Specialist",
    agent_description="Expert in market research and analysis",
    model_name="gpt-4.1",
)

financial_agent = Agent(
    agent_name="Financial-Analyst",
    agent_description="Specialist in financial analysis and valuation",
    model_name="gpt-4.1",
)

# Initialize the hierarchical swarm
swarm = HierarchicalSwarm(
    name="Financial-Analysis-Swarm",
    description="A hierarchical swarm for comprehensive financial analysis",
    agents=[research_agent, financial_agent],
    max_loops=2,
    verbose=True,
)

# Execute a complex task
task = "Analyze the market potential for Tesla (TSLA) stock"
result = swarm.run(task=task)
print(result)
```

#### Streaming Callback Example

```python
from swarms import Agent
from swarms.structs.hiearchical_swarm import HierarchicalSwarm

def streaming_callback(agent_name: str, chunk: str, is_final: bool):
    """Callback function for real-time streaming of agent outputs."""
    if not hasattr(streaming_callback, 'buffers'):
        streaming_callback.buffers = {}
        streaming_callback.paragraph_count = {}

    if agent_name not in streaming_callback.buffers:
        streaming_callback.buffers[agent_name] = ""
        streaming_callback.paragraph_count[agent_name] = 1
        print(f"\n🎬 {agent_name} starting...")

    if chunk.strip():
        tokens = chunk.replace('\n', ' \n ').split()
        for token in tokens:
            if token == '\n':
                if streaming_callback.buffers[agent_name].strip():
                    print(f"\n📄 {agent_name} - Paragraph {streaming_callback.paragraph_count[agent_name]} Complete:")
                    print(f"{streaming_callback.buffers[agent_name].strip()}")
                    streaming_callback.paragraph_count[agent_name] += 1
                    streaming_callback.buffers[agent_name] = ""
            else:
                streaming_callback.buffers[agent_name] += token + " "
                print(f"\r{agent_name} | {streaming_callback.buffers[agent_name].strip()}", end="", flush=True)

    if is_final:
        print(f"\n✅ {agent_name} completed!")

# Create agents
agents = [
    Agent(agent_name="Researcher", model_name="gpt-4o-mini"),
    Agent(agent_name="Analyst", model_name="gpt-4o-mini"),
]

# Initialize swarm
swarm = HierarchicalSwarm(
    name="Streaming-Analysis-Swarm",
    agents=agents,
    max_loops=1,
    verbose=True,
)

# Execute with streaming
task = "Analyze the impact of AI on the job market"
result = swarm.run(task=task, streaming_callback=streaming_callback)
```

#### Parameters (step method)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `task` | `str` | **Required** | The task to be executed in this step |
| `img` | `str` | `None` | Optional image input for the agents |
| `streaming_callback` | `Callable[[str, str, bool], None]` | `None` | Optional callback for real-time streaming of agent outputs |
| `*args` | `Any` | - | Additional positional arguments |
| `**kwargs` | `Any` | - | Additional keyword arguments |

#### Returns (step method)

| Type | Description |
|------|-------------|
| `str` | Feedback from the director based on agent outputs |

#### Example (step method)

```python
from swarms import Agent
from swarms.structs.hiearchical_swarm import HierarchicalSwarm

# Create development agents
frontend_agent = Agent(
    agent_name="Frontend-Developer",
    agent_description="Expert in React and modern web development",
    model_name="gpt-4.1",
)

backend_agent = Agent(
    agent_name="Backend-Developer",
    agent_description="Specialist in Node.js and API development",
    model_name="gpt-4.1",
)

# Initialize the swarm
swarm = HierarchicalSwarm(
    name="Development-Swarm",
    description="A hierarchical swarm for software development",
    agents=[frontend_agent, backend_agent],
    max_loops=1,
    verbose=True,
)

# Execute a single step
task = "Create a simple web app for file upload and download"
feedback = swarm.step(task=task)
print("Director Feedback:", feedback)
```

#### Parameters (batched_run method)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `tasks` | `List[str]` | **Required** | List of tasks to be processed |
| `img` | `str` | `None` | Optional image input for the agents |
| `streaming_callback` | `Callable[[str, str, bool], None]` | `None` | Optional callback for real-time streaming of agent outputs |
| `*args` | `Any` | - | Additional positional arguments |
| `**kwargs` | `Any` | - | Additional keyword arguments |

#### Returns (batched_run method)

| Type | Description |
|------|-------------|
| `List[Any]` | List of results for each task |

#### Example (batched_run method)

```python
from swarms import Agent
from swarms.structs.hiearchical_swarm import HierarchicalSwarm

# Create analysis agents
market_agent = Agent(
    agent_name="Market-Analyst",
    agent_description="Expert in market analysis and trends",
    model_name="gpt-4.1",
)

technical_agent = Agent(
    agent_name="Technical-Analyst",
    agent_description="Specialist in technical analysis and patterns",
    model_name="gpt-4.1",
)

# Initialize the swarm
swarm = HierarchicalSwarm(
    name="Analysis-Swarm",
    description="A hierarchical swarm for comprehensive analysis",
    agents=[market_agent, technical_agent],
    max_loops=2,
    verbose=True,
)

# Execute multiple tasks
tasks = [
    "Analyze Apple (AAPL) stock performance",
    "Evaluate Microsoft (MSFT) market position",
    "Assess Google (GOOGL) competitive landscape"
]

results = swarm.batched_run(tasks=tasks)
for i, result in enumerate(results):
    print(f"Task {i+1} Result:", result)
```

## Advanced Usage Examples

### Financial Analysis Swarm

```python
from swarms import Agent
from swarms.structs.hiearchical_swarm import HierarchicalSwarm

# Create specialized financial agents
market_research_agent = Agent(
    agent_name="Market-Research-Specialist",
    agent_description="Expert in market research, trend analysis, and competitive intelligence",
    system_prompt="""You are a senior market research specialist with expertise in:
    - Market trend analysis and forecasting
    - Competitive landscape assessment
    - Consumer behavior analysis
    - Industry report generation
    - Market opportunity identification
    - Risk assessment and mitigation strategies""",
    model_name="claude-3-sonnet-20240229",
)

financial_analyst_agent = Agent(
    agent_name="Financial-Analysis-Expert",
    agent_description="Specialist in financial statement analysis, valuation, and investment research",
    system_prompt="""You are a senior financial analyst with deep expertise in:
    - Financial statement analysis (income statement, balance sheet, cash flow)
    - Valuation methodologies (DCF, comparable company analysis, precedent transactions)
    - Investment research and due diligence
    - Financial modeling and forecasting
    - Risk assessment and portfolio analysis
    - ESG (Environmental, Social, Governance) analysis""",
    model_name="claude-3-sonnet-20240229",
)

# Initialize the hierarchical swarm
financial_analysis_swarm = HierarchicalSwarm(
    name="Financial-Analysis-Hierarchical-Swarm",
    description="A hierarchical swarm for comprehensive financial analysis with specialized agents",
    agents=[market_research_agent, financial_analyst_agent],
    max_loops=2,
    verbose=True,
)

# Execute financial analysis
task = "Conduct a comprehensive analysis of Tesla (TSLA) stock including market position, financial health, and investment potential"
result = financial_analysis_swarm.run(task=task)
print(result)
```

### Development Department Swarm

```python
from swarms import Agent
from swarms.structs.hiearchical_swarm import HierarchicalSwarm

# Create specialized development agents
frontend_developer_agent = Agent(
    agent_name="Frontend-Developer",
    agent_description="Senior frontend developer expert in modern web technologies and user experience",
    system_prompt="""You are a senior frontend developer with expertise in:
    - Modern JavaScript frameworks (React, Vue, Angular)
    - TypeScript and modern ES6+ features
    - CSS frameworks and responsive design
    - State management (Redux, Zustand, Context API)
    - Web performance optimization
    - Accessibility (WCAG) and SEO best practices""",
    model_name="claude-3-sonnet-20240229",
)

backend_developer_agent = Agent(
    agent_name="Backend-Developer",
    agent_description="Senior backend developer specializing in server-side development and API design",
    system_prompt="""You are a senior backend developer with expertise in:
    - Server-side programming languages (Python, Node.js, Java, Go)
    - Web frameworks (Django, Flask, Express, Spring Boot)
    - Database design and optimization (SQL, NoSQL)
    - API design and REST/GraphQL implementation
    - Authentication and authorization systems
    - Microservices architecture and containerization""",
    model_name="claude-3-sonnet-20240229",
)

# Initialize the development swarm
development_department_swarm = HierarchicalSwarm(
    name="Autonomous-Development-Department",
    description="A fully autonomous development department with specialized agents",
    agents=[frontend_developer_agent, backend_developer_agent],
    max_loops=3,
    verbose=True,
)

# Execute development project
task = "Create a simple web app that allows users to upload a file and then download it. The app should be built with React and Node.js."
result = development_department_swarm.run(task=task)
print(result)
```

## Output Types

The `HierarchicalSwarm` supports various output formats through the `output_type` parameter:

| Output Type | Description | Use Case |
|-------------|-------------|----------|
| `"dict-all-except-first"` | Returns all conversation history as a dictionary, excluding the first message | Default format for comprehensive analysis |
| `"dict"` | Returns conversation history as a dictionary | When you need structured data |
| `"str"` | Returns conversation history as a string | For simple text output |
| `"list"` | Returns conversation history as a list | For sequential processing |

## Streaming Callbacks

The `HierarchicalSwarm` supports real-time streaming of agent outputs through optional callback functions. This feature allows you to monitor the text generation process as it happens, token by token.

### Streaming Callback Function Signature

```python
def streaming_callback(agent_name: str, chunk: str, is_final: bool) -> None:
    """
    Callback function for real-time streaming of agent outputs.

    Args:
        agent_name (str): The name of the agent producing the output
        chunk (str): The chunk of text generated (empty if is_final=True)
        is_final (bool): True when the agent has completed its task
    """
    pass
```

### Streaming Callback Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `agent_name` | `str` | The name of the agent currently generating output |
| `chunk` | `str` | The text chunk generated by the agent |
| `is_final` | `bool` | Indicates if this is the final chunk (agent completed) |

### Live Paragraph Formation

The streaming callback can accumulate tokens to show live paragraph formation:

```python
def live_paragraph_callback(agent_name: str, chunk: str, is_final: bool):
    """Shows live paragraph formation as text is generated."""
    if not hasattr(live_paragraph_callback, 'buffers'):
        live_paragraph_callback.buffers = {}

    if agent_name not in live_paragraph_callback.buffers:
        live_paragraph_callback.buffers[agent_name] = ""
        print(f"\n🎬 {agent_name} starting...")

    if chunk.strip():
        tokens = chunk.replace('\n', ' \n ').split()
        for token in tokens:
            if token == '\n':
                if live_paragraph_callback.buffers[agent_name].strip():
                    print(f"\n📄 {agent_name} - Paragraph Complete:")
                    print(f"{live_paragraph_callback.buffers[agent_name].strip()}")
                    live_paragraph_callback.buffers[agent_name] = ""
            else:
                live_paragraph_callback.buffers[agent_name] += token + " "
                print(f"\r{agent_name} | {live_paragraph_callback.buffers[agent_name].strip()}", end="", flush=True)

    if is_final:
        print(f"\n✅ {agent_name} completed!")
```

### Streaming Use Cases

- **Real-time Monitoring**: Watch agents work simultaneously
- **Progress Tracking**: See text formation token by token
- **Live Debugging**: Monitor agent performance in real-time
- **User Experience**: Provide live feedback to users
- **Logging**: Capture detailed execution traces

### Streaming in Different Methods

Streaming callbacks work with all execution methods:

```python
# Single task with streaming
result = swarm.run(task=task, streaming_callback=my_callback)

# Single step with streaming
result = swarm.step(task=task, streaming_callback=my_callback)

# Batch processing with streaming
results = swarm.batched_run(tasks=tasks, streaming_callback=my_callback)
```

## Best Practices

| Best Practice                | Description                                                                                      |
|------------------------------|--------------------------------------------------------------------------------------------------|
| **Agent Specialization**     | Create agents with specific, well-defined expertise areas                                        |
| **Clear Task Descriptions**  | Provide detailed, actionable task descriptions                                                  |
| **Appropriate Loop Count**   | Set `max_loops` based on task complexity (1-3 for most tasks)                                   |
| **Verbose Logging**          | Enable verbose mode during development for debugging                                             |
| **Context Preservation**     | Leverage the built-in conversation history for continuity                                       |
| **Error Handling**           | Implement proper error handling for production use                                              |
| **Streaming Callbacks**      | Use streaming callbacks for real-time monitoring and user feedback                              |
| **Callback Performance**     | Keep streaming callbacks lightweight to avoid blocking the main execution thread                |

## Error Handling

The `HierarchicalSwarm` includes comprehensive error handling with detailed logging. Common issues and solutions:

- **No Agents**: Ensure at least one agent is provided
- **Invalid Director**: Verify the director agent is properly configured
- **Max Loops**: Set `max_loops` to a value greater than 0
- **Model Issues**: Check that all agents have valid model configurations

## Performance Considerations

- **Loop Optimization**: Balance between thoroughness and performance with `max_loops`
- **Agent Count**: More agents increase coordination overhead
- **Model Selection**: Choose appropriate models for your use case and budget
- **Verbose Mode**: Disable verbose logging in production for better performance

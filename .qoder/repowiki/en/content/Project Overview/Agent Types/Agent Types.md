# Agent Types

<cite>
**Referenced Files in This Document**   
- [base.py](file://app/agent/base.py)
- [manus.py](file://app/agent/manus.py)
- [data_analysis.py](file://app/agent/data_analysis.py)
- [swe.py](file://app/agent/swe.py)
- [react.py](file://app/agent/react.py)
- [sandbox_agent.py](file://app/agent/sandbox_agent.py)
- [toolcall.py](file://app/agent/toolcall.py)
- [SYSTEM_PROMPT](file://app/prompt/manus.py)
- [NEXT_STEP_PROMPT](file://app/prompt/manus.py)
- [visualization.py](file://app/prompt/visualization.py)
- [swe.py](file://app/prompt/swe.py)
- [python_execute.py](file://app/tool/python_execute.py)
- [data_visualization.py](file://app/tool/chart_visualization/data_visualization.py)
- [sb_shell_tool.py](file://app/tool/sandbox/sb_shell_tool.py)
- [sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py)
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py)
- [sb_vision_tool.py](file://app/tool/sandbox/sb_vision_tool.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Core Agent Architecture](#core-agent-architecture)
3. [Manus Agent](#manus-agent)
4. [DataAnalysis Agent](#dataanalysis-agent)
5. [SWE Agent](#swe-agent)
6. [ReAct Agent](#react-agent)
7. [Sandbox Agent](#sandbox-agent)
8. [Agent Selection and Use Cases](#agent-selection-and-use-cases)
9. [Integration Points](#integration-points)
10. [Extending and Customizing Agents](#extending-and-customizing-agents)
11. [Common Issues and Best Practices](#common-issues-and-best-practices)

## Introduction
OpenManus implements a modular agent system with specialized agent types designed for different task domains. Each agent type inherits from base classes and is specialized through prompt engineering and tool selection to optimize performance for specific use cases. This document details the architecture, implementation, and usage patterns for each agent type in the OpenManus framework.

## Core Agent Architecture

```mermaid
classDiagram
class BaseAgent {
+str name
+str description
+str system_prompt
+str next_step_prompt
+LLM llm
+Memory memory
+AgentState state
+int max_steps
+int current_step
+run(request) str
+step() str
+update_memory(role, content)
+is_stuck() bool
}
class ToolCallAgent {
+ToolCollection available_tools
+list[str] special_tool_names
+think() bool
+act() str
+step() str
}
class ReActAgent {
+think() bool
+act() str
+step() str
}
BaseAgent <|-- ToolCallAgent
BaseAgent <|-- ReActAgent
ToolCallAgent <|-- Manus
ToolCallAgent <|-- DataAnalysis
ToolCallAgent <|-- SWEAgent
ToolCallAgent <|-- SandboxManus
ReActAgent <|-- BrowserAgent
class Manus {
+MCPClients mcp_clients
+BrowserContextHelper browser_context_helper
+dict connected_servers
+create() Manus
+initialize_mcp_servers()
+connect_mcp_server()
+cleanup()
}
class DataAnalysis {
+int max_observe
+ToolCollection available_tools
}
class SWEAgent {
+int max_steps
+ToolCollection available_tools
}
class SandboxManus {
+dict sandbox_link
+initialize_sandbox_tools()
+delete_sandbox()
}
```

**Diagram sources**
- [base.py](file://app/agent/base.py#L1-L196)
- [toolcall.py](file://app/agent/toolcall.py)
- [manus.py](file://app/agent/manus.py#L1-L165)
- [data_analysis.py](file://app/agent/data_analysis.py#L1-L37)
- [swe.py](file://app/agent/swe.py#L1-L24)
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)
- [react.py](file://app/agent/react.py#L1-L38)

**Section sources**
- [base.py](file://app/agent/base.py#L1-L196)
- [toolcall.py](file://app/agent/toolcall.py)

## Manus Agent

The Manus agent serves as the general-purpose agent in OpenManus, designed to handle a wide variety of tasks through comprehensive tool access. It inherits from `ToolCallAgent` and specializes in multi-modal task execution by integrating local tools with MCP (Modular Cognitive Processing) servers.

The agent is initialized with a rich set of tools including Python execution, browser automation, file editing, human interaction, and task termination. It dynamically connects to configured MCP servers, incorporating their tools into its available tool collection. The Manus agent uses prompt engineering with a system prompt that establishes its role as an all-capable AI assistant and a next-step prompt that guides proactive tool selection.

During execution, the Manus agent monitors browser usage in recent messages and dynamically adjusts its next-step prompt context through the `BrowserContextHelper`. This allows for more informed decision-making when web browsing tasks are in progress.

```mermaid
sequenceDiagram
participant User
participant Manus
participant Tool
participant MCP
participant Sandbox
User->>Manus : Submit task request
Manus->>Manus : Initialize MCP connections
Manus->>Manus : Update memory with request
loop For each execution step
Manus->>Manus : Think (select appropriate tool)
Manus->>Tool : Execute selected tool
alt Tool is local
Tool-->>Manus : Return result
else Tool is MCP-based
Manus->>MCP : Forward tool request
MCP-->>Manus : Return MCP tool result
end
Manus->>Manus : Update memory with result
Manus->>Manus : Determine next action
end
Manus->>User : Return final results
```

**Diagram sources**
- [manus.py](file://app/agent/manus.py#L1-L165)
- [toolcall.py](file://app/agent/toolcall.py)
- [prompt/manus.py](file://app/prompt/manus.py#L1-L10)

**Section sources**
- [manus.py](file://app/agent/manus.py#L1-L165)
- [prompt/manus.py](file://app/prompt/manus.py#L1-L10)

## DataAnalysis Agent

The DataAnalysis agent specializes in data processing, visualization, and reporting tasks. It extends the `ToolCallAgent` class with a focused toolset optimized for data-centric workflows, including specialized Python execution, data visualization preparation, and chart generation capabilities.

This agent uses a domain-specific system prompt that establishes its role in data analysis tasks and emphasizes the importance of generating analysis conclusion reports. The next-step prompt guides the agent to break down problems and use tools step by step, with specific instructions to review and fix errors in observations.

The DataAnalysis agent is configured with a higher `max_observe` value (15000) compared to other agents, accommodating the larger data outputs typical in data analysis workflows. Its tool collection includes `NormalPythonExecute` for data processing, `VisualizationPrepare` for chart preparation, `DataVisualization` for rendering charts in various formats, and `Terminate` for task completion.

```mermaid
flowchart TD
Start([Data Analysis Task]) --> LoadData["Load Data from Source"]
LoadData --> PrepareData["Prepare and Clean Data"]
PrepareData --> AnalyzeData["Analyze Data Patterns"]
AnalyzeData --> Visualize["Visualize Results"]
Visualize --> GenerateReport["Generate Analysis Report"]
GenerateReport --> Review["Review Results and Insights"]
Review --> |Satisfactory| Terminate["Call Terminate Tool"]
Review --> |Needs Improvement| AdjustAnalysis["Adjust Analysis Approach"]
AdjustAnalysis --> AnalyzeData
Terminate --> End([Task Completed])
```

**Diagram sources**
- [data_analysis.py](file://app/agent/data_analysis.py#L1-L37)
- [prompt/visualization.py](file://app/prompt/visualization.py#L1-L10)
- [data_visualization.py](file://app/tool/chart_visualization/data_visualization.py#L1-L263)

**Section sources**
- [data_analysis.py](file://app/agent/data_analysis.py#L1-L37)
- [prompt/visualization.py](file://app/prompt/visualization.py#L1-L10)

## SWE Agent

The SWE (Software Engineering) agent is designed specifically for programming tasks and code development. It inherits from `ToolCallAgent` but implements a distinct paradigm focused on direct computer interaction for software engineering workflows.

This agent uses a specialized system prompt that establishes its role as an autonomous programmer working in a command-line environment with a file editor. The prompt emphasizes proper indentation in code and restricts the agent to issuing one tool call at a time, preventing the execution of multiple commands simultaneously.

The SWE agent's toolset is minimal but powerful, consisting of Bash execution for command-line operations, `StrReplaceEditor` for precise code modifications, and `Terminate` for task completion. This focused tool selection encourages the agent to solve programming tasks through careful, step-by-step code modifications rather than broad tool usage.

```mermaid
sequenceDiagram
participant Developer
participant SWEAgent
participant FileSystem
participant Terminal
Developer->>SWEAgent : Submit coding task
SWEAgent->>SWEAgent : Analyze task requirements
SWEAgent->>FileSystem : Read relevant files
SWEAgent->>SWEAgent : Plan code changes
SWEAgent->>FileSystem : Modify code with StrReplaceEditor
SWEAgent->>Terminal : Execute tests with Bash
SWEAgent->>SWEAgent : Evaluate test results
alt Tests pass
SWEAgent->>Developer : Report success and terminate
else Tests fail
SWEAgent->>SWEAgent : Diagnose issues
SWEAgent->>FileSystem : Fix code issues
SWEAgent->>Terminal : Re-run tests
end
```

**Diagram sources**
- [swe.py](file://app/agent/swe.py#L1-L24)
- [prompt/swe.py](file://app/prompt/swe.py#L1-L22)
- [bash.py](file://app/tool/bash.py)
- [str_replace_editor.py](file://app/tool/str_replace_editor.py)

**Section sources**
- [swe.py](file://app/agent/swe.py#L1-L24)
- [prompt/swe.py](file://app/prompt/swe.py#L1-L22)

## ReAct Agent

The ReAct (Reasoning and Acting) agent implements a cognitive architecture that separates reasoning from action execution. It inherits from the `BaseAgent` class and defines an abstract interface for the ReAct paradigm, which can be extended by specialized agents.

This agent type formalizes the think-act cycle by defining abstract methods for `think()` and `act()`, with a concrete `step()` method that orchestrates their execution. The `think()` method is responsible for processing the current state and deciding on the next action, while the `act()` method executes the decided actions.

The ReAct pattern enables more deliberate decision-making by ensuring that reasoning and action are distinct phases in the agent's workflow. This separation allows for more sophisticated planning and reduces the likelihood of impulsive or poorly considered actions.

```mermaid
flowchart TD
Start([Agent Execution]) --> Think["think() method\nProcess current state\nDecide next action"]
Think --> Decision{"Should act?"}
Decision --> |Yes| Act["act() method\nExecute decided action"]
Decision --> |No| NoAction["Return 'no action needed'"]
Act --> UpdateMemory["Update memory with results"]
NoAction --> UpdateMemory
UpdateMemory --> CheckTermination["Check if task complete\nor max steps reached"]
CheckTermination --> |Continue| Think
CheckTermination --> |Terminate| End([Execution Complete])
```

**Diagram sources**
- [react.py](file://app/agent/react.py#L1-L38)
- [base.py](file://app/agent/base.py#L1-L196)

**Section sources**
- [react.py](file://app/agent/react.py#L1-L38)

## Sandbox Agent

The Sandbox agent (SandboxManus) provides isolated execution capabilities through integration with Daytona sandbox environments. It inherits from `ToolCallAgent` like the general Manus agent but specializes in sandbox-based tool execution.

This agent is initialized with sandbox-specific tools that enable secure execution of potentially risky operations in an isolated environment. During creation, it establishes a sandbox instance and configures various sandbox tools for browser automation, file operations, shell commands, and vision capabilities.

The agent maintains references to sandbox links (VNC and website URLs) for monitoring and debugging purposes. It provides methods to initialize sandbox tools with password protection and to delete sandboxes when cleanup is required. The sandbox integration allows for safe execution of web browsing tasks, file system modifications, and command-line operations without risking the host environment.

```mermaid
sequenceDiagram
participant User
participant SandboxManus
participant Daytona
participant Sandbox
User->>SandboxManus : Submit task with sandbox requirements
SandboxManus->>Daytona : Create new sandbox instance
Daytona-->>SandboxManus : Return sandbox reference
SandboxManus->>SandboxManus : Initialize sandbox tools
SandboxManus->>SandboxManus : Add sandbox tools to available_tools
loop For each execution step
SandboxManus->>SandboxManus : Think (select sandbox tool)
alt Selected tool is sandbox-based
SandboxManus->>Sandbox : Execute sandbox tool
Sandbox-->>SandboxManus : Return tool result
else Selected tool is regular tool
SandboxManus->>Tool : Execute regular tool
Tool-->>SandboxManus : Return result
end
SandboxManus->>SandboxManus : Update memory with result
end
SandboxManus->>Daytona : Delete sandbox instance
SandboxManus->>User : Return final results
```

**Diagram sources**
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)
- [daytona/sandbox.py](file://app/daytona/sandbox.py)
- [sb_shell_tool.py](file://app/tool/sandbox/sb_shell_tool.py#L1-L419)
- [sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py#L1-L361)

**Section sources**
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)
- [daytona/README.md](file://app/daytona/README.md#L1-L57)

## Agent Selection and Use Cases

Choosing the appropriate agent type depends on the nature of the task at hand. The following guidelines help determine which agent to use for different scenarios:

```mermaid
flowchart TD
Start([Task Received]) --> TaskType{"Task Type?"}
TaskType --> |General purpose task\nMultiple tool types needed| Manus["Use Manus Agent\n• Broad tool access\n• MCP integration\n• General problem solving"]
TaskType --> |Data analysis\nVisualization\nReporting| DataAnalysis["Use DataAnalysis Agent\n• Specialized data tools\n• Chart visualization\n• Report generation"]
TaskType --> |Software development\nCode editing\nProgramming| SWE["Use SWE Agent\n• Code-focused tools\n• Precise file editing\n• Command-line access"]
TaskType --> |Reasoning-intensive task\nComplex planning\nMulti-step reasoning| ReAct["Use ReAct Agent\n• Explicit think-act cycle\n• Structured reasoning\n• Step-by-step planning"]
TaskType --> |Potentially risky operations\nWeb browsing\nIsolated execution| Sandbox["Use Sandbox Agent\n• Isolated environment\n• Secure execution\n• VNC monitoring"]
Manus --> End1([Execute with Manus])
DataAnalysis --> End2([Execute with DataAnalysis])
SWE --> End3([Execute with SWE])
ReAct --> End4([Execute with ReAct])
Sandbox --> End5([Execute with Sandbox])
```

**Diagram sources**
- [manus.py](file://app/agent/manus.py#L1-L165)
- [data_analysis.py](file://app/agent/data_analysis.py#L1-L37)
- [swe.py](file://app/agent/swe.py#L1-L24)
- [react.py](file://app/agent/react.py#L1-L38)
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)

**Section sources**
- [manus.py](file://app/agent/manus.py#L1-L165)
- [data_analysis.py](file://app/agent/data_analysis.py#L1-L37)
- [swe.py](file://app/agent/swe.py#L1-L24)
- [react.py](file://app/agent/react.py#L1-L38)
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)

## Integration Points

Agents in OpenManus integrate with various external systems and components through well-defined interfaces. These integration points enable the agents to extend their capabilities beyond basic functionality.

```mermaid
graph TB
subgraph "Agent Core"
BaseAgent[BaseAgent]
ToolCallAgent[ToolCallAgent]
ReActAgent[ReActAgent]
end
subgraph "Tools & Execution"
LocalTools[Local Tools]
MCP[MCP Servers]
Sandbox[Daytona Sandbox]
LLM[LLM Service]
end
subgraph "External Systems"
Web[Web Services]
Filesystem[File System]
CommandLine[Command Line]
Database[External Databases]
end
BaseAgent --> ToolCallAgent
BaseAgent --> ReActAgent
ToolCallAgent --> LocalTools
ToolCallAgent --> MCP
ToolCallAgent --> Sandbox
ReActAgent --> LocalTools
LocalTools --> Web
LocalTools --> Filesystem
LocalTools --> CommandLine
LocalTools --> Database
MCP --> ExternalMCP[External MCP Services]
Sandbox --> IsolatedEnvironment[Isolated Execution Environment]
BaseAgent --> LLM
```

**Diagram sources**
- [manus.py](file://app/agent/manus.py#L1-L165)
- [toolcall.py](file://app/agent/toolcall.py)
- [mcp.py](file://app/agent/mcp.py)
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)
- [llm.py](file://app/llm.py)

**Section sources**
- [manus.py](file://app/agent/manus.py#L1-L165)
- [toolcall.py](file://app/agent/toolcall.py)
- [mcp.py](file://app/agent/mcp.py)
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)

## Extending and Customizing Agents

Creating new agents or extending existing ones in OpenManus follows a consistent pattern based on inheritance and composition. Developers can create specialized agents by inheriting from base agent classes and customizing their behavior through prompt engineering and tool selection.

To create a new agent, extend either `ToolCallAgent` for tool-based workflows or `ReActAgent` for reasoning-act architectures. Customize the agent by setting appropriate prompts, configuring tool collections, and overriding methods as needed. The factory pattern used in agent creation (e.g., the `create` class method) allows for proper asynchronous initialization of agent components.

When extending agents, consider the principle of least privilege by including only the tools necessary for the agent's intended purpose. This improves security and focuses the agent's capabilities on its specific domain. Prompt engineering should align with the agent's toolset and intended use cases, guiding the LLM toward appropriate tool selection and task execution patterns.

```mermaid
classDiagram
class CustomAgent {
+str name
+str description
+str system_prompt
+str next_step_prompt
+ToolCollection available_tools
+create() CustomAgent
+initialize_custom_components()
+cleanup()
}
ToolCallAgent <|-- CustomAgent
CustomAgent --> CustomTool1
CustomAgent --> CustomTool2
CustomAgent --> ExternalService
class CustomTool1 {
+str name
+str description
+dict parameters
+execute() dict
}
class CustomTool2 {
+str name
+str description
+dict parameters
+execute() dict
}
class ExternalService {
+str api_endpoint
+str api_key
+call_api() dict
}
CustomAgent ..> CustomTool1 : uses
CustomAgent ..> CustomTool2 : uses
CustomTool1 ..> ExternalService : integrates with
CustomTool2 ..> ExternalService : integrates with
```

**Diagram sources**
- [base.py](file://app/agent/base.py#L1-L196)
- [toolcall.py](file://app/agent/toolcall.py)
- [manus.py](file://app/agent/manus.py#L1-L165)

**Section sources**
- [base.py](file://app/agent/base.py#L1-L196)
- [toolcall.py](file://app/agent/toolcall.py)

## Common Issues and Best Practices

When working with OpenManus agents, several common issues may arise, along with established best practices for optimal performance and reliability.

**Common Issues:**
- **Tool selection errors**: Agents may select inappropriate tools for the task. Mitigate by refining prompts and ensuring tool descriptions are clear.
- **State management**: Agents can become stuck in loops. The framework includes stuck state detection that adds prompts to change strategies when duplicate responses are detected.
- **Sandbox initialization failures**: Ensure Daytona API keys are properly configured and sandbox images are available.
- **MCP server connection issues**: Verify server configurations in the MCP settings and ensure network connectivity.
- **Resource cleanup**: Always ensure proper cleanup of sandboxes and MCP connections to prevent resource leaks.

**Best Practices:**
- **Prompt engineering**: Craft system and next-step prompts that clearly define the agent's role and guide appropriate tool selection.
- **Tool minimization**: Include only necessary tools for each agent type to focus capabilities and improve security.
- **Error handling**: Implement robust error handling in custom tools and ensure agents can recover from tool execution failures.
- **Monitoring**: Use the provided VNC and website links to monitor sandbox agent activities during execution.
- **Testing**: Test new agents and tools with simple tasks before deploying them for complex workflows.
- **Documentation**: Maintain clear documentation of agent capabilities and limitations to guide users in appropriate selection.

**Section sources**
- [base.py](file://app/agent/base.py#L1-L196)
- [manus.py](file://app/agent/manus.py#L1-L165)
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)
- [daytona/README.md](file://app/daytona/README.md#L1-L57)
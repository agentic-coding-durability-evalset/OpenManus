# Sandbox Agent

<cite>
**Referenced Files in This Document**   
- [sandbox_agent.py](file://app/agent/sandbox_agent.py)
- [sandbox.py](file://app/sandbox/core/sandbox.py)
- [manager.py](file://app/sandbox/core/manager.py)
- [terminal.py](file://app/sandbox/core/terminal.py)
- [client.py](file://app/sandbox/client.py)
- [config.py](file://app/config.py)
- [sandbox.py](file://app/daytona/sandbox.py)
- [tool_base.py](file://app/daytona/tool_base.py)
- [sb_shell_tool.py](file://app/tool/sandbox/sb_shell_tool.py)
- [sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py)
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py)
- [sb_vision_tool.py](file://app/tool/sandbox/sb_vision_tool.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Project Structure](#project-structure)
3. [Core Components](#core-components)
4. [Architecture Overview](#architecture-overview)
5. [Detailed Component Analysis](#detailed-component-analysis)
6. [Dependency Analysis](#dependency-analysis)
7. [Performance Considerations](#performance-considerations)
8. [Troubleshooting Guide](#troubleshooting-guide)
9. [Conclusion](#conclusion)

## Introduction
The Sandbox Agent is a specialized component designed to execute untrusted code within isolated Docker container environments. It provides a secure execution environment for evaluating third-party code, running potentially unsafe scripts, and testing software in clean, controlled conditions. The agent operates through a sophisticated sandbox system that manages container lifecycle, enforces resource limits, and handles command execution securely. This documentation details the agent's integration with core sandbox components, its communication mechanisms with the sandbox client, configuration options for security and performance, and practical use cases for isolated code execution.

## Project Structure
The project structure reveals a well-organized system with clear separation of concerns. The sandbox functionality is primarily contained within the `app/sandbox` directory, which houses the core execution logic, while agent-specific implementations reside in `app/agent`. The tool integrations that enable specific sandbox capabilities are located in `app/tool/sandbox`. Configuration management is centralized in the `config` directory, and comprehensive testing is provided in the `tests/sandbox` directory.

```mermaid
graph TD
subgraph "Agent Layer"
A[SandboxAgent] --> B[SandboxManus]
end
subgraph "Sandbox Core"
C[Sandbox] --> D[SandboxManager]
C --> E[Terminal]
C --> F[Client]
end
subgraph "Tool Integrations"
G[ShellTool] --> C
H[FilesTool] --> C
I[BrowserTool] --> C
J[VisionTool] --> C
end
subgraph "Configuration"
K[Config] --> L[SandboxSettings]
K --> M[DaytonaSettings]
end
B --> C
B --> G
B --> H
B --> I
B --> J
C --> K
```

**Diagram sources**
- [sandbox_agent.py](file://app/agent/sandbox_agent.py)
- [sandbox.py](file://app/sandbox/core/sandbox.py)
- [manager.py](file://app/sandbox/core/manager.py)
- [client.py](file://app/sandbox/client.py)

**Section sources**
- [app/agent/sandbox_agent.py](file://app/agent/sandbox_agent.py)
- [app/sandbox/core/sandbox.py](file://app/sandbox/core/sandbox.py)
- [app/sandbox/core/manager.py](file://app/sandbox/core/manager.py)
- [app/sandbox/core/terminal.py](file://app/sandbox/core/terminal.py)
- [app/sandbox/client.py](file://app/sandbox/client.py)

## Core Components
The Sandbox Agent system comprises several interconnected components that work together to provide secure code execution. At its core, the `SandboxManus` class serves as the primary agent implementation, coordinating the creation and management of sandbox environments. The `DockerSandbox` class provides the containerized execution environment with resource limits and command execution capabilities. The `SandboxManager` handles multiple sandbox instances, enforcing limits on concurrent sandboxes and automatically cleaning up idle environments. The `AsyncDockerizedTerminal` enables interactive command execution within containers, while various tool classes (`SandboxShellTool`, `SandboxFilesTool`, etc.) provide specific functionality for shell commands, file operations, browser automation, and vision capabilities.

**Section sources**
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)
- [sandbox.py](file://app/sandbox/core/sandbox.py#L1-L463)
- [manager.py](file://app/sandbox/core/manager.py#L1-L314)
- [terminal.py](file://app/sandbox/core/terminal.py#L1-L347)

## Architecture Overview
The Sandbox Agent architecture follows a layered approach with clear separation between the agent interface, sandbox management, and container execution. The agent interacts with the sandbox system through a client interface, which abstracts the underlying container management. The sandbox manager maintains a pool of available sandbox instances, enforcing resource limits and handling lifecycle management. Each sandbox instance runs in an isolated Docker container with restricted network access and controlled resource allocation. Tools are dynamically added to the agent's available tool collection, enabling specific capabilities like shell command execution, file manipulation, and browser automation.

```mermaid
graph TD
subgraph "Agent Layer"
Agent[SandboxManus Agent]
end
subgraph "Client Interface"
Client[LocalSandboxClient]
end
subgraph "Sandbox Management"
Manager[SandboxManager]
Manager --> |Creates| Sandbox[DockerSandbox]
end
subgraph "Container Execution"
Sandbox --> |Uses| Terminal[AsyncDockerizedTerminal]
Sandbox --> |Manages| Container[Docker Container]
Terminal --> |Executes| Shell[Shell Commands]
end
subgraph "Tool Integration"
ShellTool[SandboxShellTool] --> Client
FilesTool[SandboxFilesTool] --> Client
BrowserTool[SandboxBrowserTool] --> Client
VisionTool[SandboxVisionTool] --> Client
end
Agent --> ShellTool
Agent --> FilesTool
Agent --> BrowserTool
Agent --> VisionTool
ShellTool --> Client
FilesTool --> Client
BrowserTool --> Client
VisionTool --> Client
Client --> Manager
style Agent fill:#f9f,stroke:#333
style Client fill:#bbf,stroke:#333
style Manager fill:#f96,stroke:#333
style Sandbox fill:#9f9,stroke:#333
style Terminal fill:#99f,stroke:#333
style Container fill:#ff9,stroke:#333
```

**Diagram sources**
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)
- [client.py](file://app/sandbox/client.py#L1-L202)
- [manager.py](file://app/sandbox/core/manager.py#L1-L314)
- [sandbox.py](file://app/sandbox/core/sandbox.py#L1-L463)
- [terminal.py](file://app/sandbox/core/terminal.py#L1-L347)

## Detailed Component Analysis

### Sandbox Agent Implementation
The `SandboxManus` class extends `ToolCallAgent` to provide a versatile agent capable of executing tasks using sandboxed tools. It manages the initialization of sandbox tools, connection to MCP servers, and cleanup of resources. The agent creates a sandbox environment during initialization and adds various sandbox tools to its available tools collection, enabling shell commands, file operations, browser automation, and vision capabilities.

```mermaid
classDiagram
class SandboxManus {
+name : str
+description : str
+system_prompt : str
+next_step_prompt : str
+max_observe : int
+max_steps : int
+mcp_clients : MCPClients
+available_tools : ToolCollection
+special_tool_names : list[str]
+browser_context_helper : Optional[BrowserContextHelper]
+connected_servers : Dict[str, str]
+_initialized : bool
+sandbox_link : Optional[dict[str, dict[str, str]]]
+initialize_helper() SandboxManus
+create() SandboxManus
+initialize_sandbox_tools(password : str) None
+initialize_mcp_servers() None
+connect_mcp_server(server_url : str, server_id : str, use_stdio : bool, stdio_args : List[str]) None
+disconnect_mcp_server(server_id : str) None
+delete_sandbox(sandbox_id : str) None
+cleanup() None
+think() bool
}
class ToolCallAgent {
<<abstract>>
}
SandboxManus --|> ToolCallAgent
```

**Diagram sources**
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)

**Section sources**
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)

### Sandbox Core Components
The core sandbox components provide the foundation for isolated code execution. The `DockerSandbox` class manages a single container instance, handling creation, command execution, file operations, and cleanup. The `SandboxManager` oversees multiple sandbox instances, enforcing limits on concurrent sandboxes and automatically cleaning up idle environments. The `AsyncDockerizedTerminal` enables interactive command execution within containers through a persistent terminal session.

```mermaid
classDiagram
class DockerSandbox {
+config : SandboxSettings
+volume_bindings : Dict[str, str]
+client : docker.client
+container : Optional[Container]
+terminal : Optional[AsyncDockerizedTerminal]
+__init__(config : Optional[SandboxSettings], volume_bindings : Optional[Dict[str, str]]) None
+create() DockerSandbox
+_prepare_volume_bindings() Dict[str, Dict[str, str]]
+_ensure_host_dir(path : str) str
+run_command(cmd : str, timeout : Optional[int]) str
+read_file(path : str) str
+write_file(path : str, content : str) None
+_safe_resolve_path(path : str) str
+copy_from(src_path : str, dst_path : str) None
+copy_to(src_path : str, dst_path : str) None
+_create_tar_stream(name : str, content : bytes) io.BytesIO
+_read_from_tar(tar_stream) bytes
+cleanup() None
+__aenter__() DockerSandbox
+__aexit__(exc_type, exc_val, exc_tb) None
}
class SandboxManager {
+max_sandboxes : int
+idle_timeout : int
+cleanup_interval : int
+_sandboxes : Dict[str, DockerSandbox]
+_last_used : Dict[str, float]
+_locks : Dict[str, asyncio.Lock]
+_global_lock : asyncio.Lock
+_active_operations : Set[str]
+_cleanup_task : Optional[asyncio.Task]
+_is_shutting_down : bool
+__init__(max_sandboxes : int, idle_timeout : int, cleanup_interval : int) None
+ensure_image(image : str) bool
+sandbox_operation(sandbox_id : str) AsyncContextManager
+create_sandbox(config : Optional[SandboxSettings], volume_bindings : Optional[Dict[str, str]]) str
+get_sandbox(sandbox_id : str) DockerSandbox
+start_cleanup_task() None
+_cleanup_idle_sandboxes() None
+cleanup() None
+_safe_delete_sandbox(sandbox_id : str) None
+delete_sandbox(sandbox_id : str) None
+__aenter__() SandboxManager
+__aexit__(exc_type, exc_val, exc_tb) None
+get_stats() Dict
}
class AsyncDockerizedTerminal {
+client : docker.client
+container : Union[str, Container]
+working_dir : str
+env_vars : Optional[Dict[str, str]]
+default_timeout : int
+session : Optional[DockerSession]
+__init__(container : Union[str, Container], working_dir : str, env_vars : Optional[Dict[str, str]], default_timeout : int) None
+init() None
+_ensure_workdir() None
+_exec_simple(cmd : str) Tuple[int, str]
+run_command(cmd : str, timeout : Optional[int]) str
+close() None
+__aenter__() AsyncDockerizedTerminal
+__aexit__(exc_type, exc_val, exc_tb) None
}
class DockerSession {
+api : APIClient
+container_id : str
+exec_id : Optional[str]
+socket : Optional[socket.socket]
+__init__(container_id : str) None
+create(working_dir : str, env_vars : Dict[str, str]) None
+close() None
+_read_until_prompt() str
+execute(command : str, timeout : Optional[int]) str
+_sanitize_command(command : str) str
}
DockerSandbox --> SandboxManager : "managed by"
DockerSandbox --> AsyncDockerizedTerminal : "uses"
AsyncDockerizedTerminal --> DockerSession : "uses"
```

**Diagram sources**
- [sandbox.py](file://app/sandbox/core/sandbox.py#L1-L463)
- [manager.py](file://app/sandbox/core/manager.py#L1-L314)
- [terminal.py](file://app/sandbox/core/terminal.py#L1-L347)

**Section sources**
- [sandbox.py](file://app/sandbox/core/sandbox.py#L1-L463)
- [manager.py](file://app/sandbox/core/manager.py#L1-L314)
- [terminal.py](file://app/sandbox/core/terminal.py#L1-L347)

### Sandbox Tool Integration
The sandbox system provides several specialized tools that enable specific capabilities within the isolated environment. These tools follow a common base class pattern and integrate with the Daytona sandbox system to provide shell command execution, file operations, browser automation, and vision capabilities.

#### Shell Tool
The `SandboxShellTool` enables execution of shell commands within the sandbox environment using tmux sessions to maintain state between commands. It supports both blocking and non-blocking execution, allowing for long-running processes like servers or build operations.

```mermaid
classDiagram
class SandboxShellTool {
+name : str = "sandbox_shell"
+description : str
+parameters : dict
+__init__(sandbox : Optional[Sandbox], thread_id : Optional[str], **data) None
+_ensure_session(session_name : str) str
+_cleanup_session(session_name : str) None
+_execute_raw_command(command : str) Dict[str, Any]
+_execute_command(command : str, folder : Optional[str], session_name : Optional[str], blocking : bool, timeout : int) ToolResult
+_check_command_output(session_name : str, kill_session : bool) ToolResult
+_terminate_command(session_name : str) ToolResult
+_list_commands() ToolResult
+execute(action : str, command : str, folder : Optional[str], session_name : Optional[str], blocking : bool, timeout : int, kill_session : bool) ToolResult
+cleanup() None
}
class SandboxToolsBase {
+_urls_printed : ClassVar[bool]
+project_id : Optional[str]
+_sandbox : Optional[Sandbox]
+_sandbox_id : Optional[str]
+_sandbox_pass : Optional[str]
+workspace_path : str
+_sessions : dict[str, str]
+_ensure_sandbox() Sandbox
+sandbox() Sandbox
+sandbox_id() str
+clean_path(path : str) str
}
SandboxShellTool --|> SandboxToolsBase
```

**Diagram sources**
- [sb_shell_tool.py](file://app/tool/sandbox/sb_shell_tool.py#L1-L420)

#### Files Tool
The `SandboxFilesTool` provides comprehensive file operations within the sandbox environment, including creating, reading, updating, and deleting files. It ensures all operations are performed relative to the workspace directory for security.

```mermaid
classDiagram
class SandboxFilesTool {
+name : str = "sandbox_files"
+description : str
+parameters : dict
+SNIPPET_LINES : int
+__init__(sandbox : Optional[Sandbox], thread_id : Optional[str], **data) None
+clean_path(path : str) str
+_should_exclude_file(rel_path : str) bool
+_file_exists(path : str) bool
+get_workspace_state() dict
+execute(action : str, file_path : Optional[str], file_contents : Optional[str], old_str : Optional[str], new_str : Optional[str], permissions : Optional[str]) ToolResult
+_create_file(file_path : str, file_contents : str, permissions : str) ToolResult
+_str_replace(file_path : str, old_str : str, new_str : str) ToolResult
+_full_file_rewrite(file_path : str, file_contents : str, permissions : str) ToolResult
+_delete_file(file_path : str) ToolResult
+cleanup() None
+create_with_context(context : Context) SandboxFilesTool[Context]
}
SandboxFilesTool --|> SandboxToolsBase
```

**Diagram sources**
- [sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py#L1-L362)

#### Browser Tool
The `SandboxBrowserTool` enables browser automation within the sandbox environment, supporting navigation, interaction with elements, scrolling, tab management, and content extraction. It maintains state across calls, keeping the browser session alive until explicitly closed.

```mermaid
classDiagram
class SandboxBrowserTool {
+name : str = "sandbox_browser"
+description : str
+parameters : dict
+browser_message : Optional[ThreadMessage]
+__init__(sandbox : Optional[Sandbox], thread_id : Optional[str], **data) None
+_validate_base64_image(base64_string : str, max_size_mb : int) tuple[bool, str]
+_execute_browser_action(endpoint : str, params : dict, method : str) ToolResult
+execute(action : str, url : Optional[str], index : Optional[int], text : Optional[str], amount : Optional[int], page_id : Optional[int], keys : Optional[str], seconds : Optional[int], x : Optional[int], y : Optional[int], element_source : Optional[str], element_target : Optional[str]) ToolResult
+get_current_state(message : Optional[ThreadMessage]) ToolResult
+create_with_sandbox(sandbox : Sandbox) SandboxBrowserTool
}
SandboxBrowserTool --|> SandboxToolsBase
```

**Diagram sources**
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py#L1-L451)

#### Vision Tool
The `SandboxVisionTool` allows the agent to read image files within the sandbox, compressing them and converting to base64 for use in subsequent context. It supports common image formats and enforces size limits for performance and security.

```mermaid
classDiagram
class SandboxVisionTool {
+name : str = "sandbox_vision"
+description : str
+parameters : dict
+vision_message : Optional[ThreadMessage]
+__init__(sandbox : Optional[Sandbox], thread_id : Optional[str], **data) None
+compress_image(image_bytes : bytes, mime_type : str, file_path : str) tuple[bytes, str]
+execute(action : str, file_path : Optional[str]) ToolResult
}
SandboxVisionTool --|> SandboxToolsBase
```

**Diagram sources**
- [sb_vision_tool.py](file://app/tool/sandbox/sb_vision_tool.py#L1-L179)

**Section sources**
- [sb_shell_tool.py](file://app/tool/sandbox/sb_shell_tool.py#L1-L420)
- [sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py#L1-L362)
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py#L1-L451)
- [sb_vision_tool.py](file://app/tool/sandbox/sb_vision_tool.py#L1-L179)

## Dependency Analysis
The Sandbox Agent system has a well-defined dependency structure that ensures proper initialization and resource management. The agent depends on the sandbox core components for container management, which in turn depend on Docker SDK for Python. The tool integrations depend on the Daytona SDK for sandbox operations, while configuration management is handled through Pydantic models and TOML configuration files.

```mermaid
graph TD
A[SandboxManus] --> B[SandboxToolsBase]
B --> C[Daytona SDK]
A --> D[SandboxShellTool]
A --> E[SandboxFilesTool]
A --> F[SandboxBrowserTool]
A --> G[SandboxVisionTool]
D --> B
E --> B
F --> B
G --> B
A --> H[LocalSandboxClient]
H --> I[DockerSandbox]
I --> J[SandboxManager]
I --> K[AsyncDockerizedTerminal]
K --> L[DockerSession]
M[Config] --> N[SandboxSettings]
M --> O[DaytonaSettings]
A --> M
J --> M
P[Docker SDK] --> I
P --> K
Q[Pydantic] --> M
R[TOML] --> M
style A fill:#f9f,stroke:#333
style B fill:#ff9,stroke:#333
style C fill:#9f9,stroke:#333
style D fill:#ff9,stroke:#333
style E fill:#ff9,stroke:#333
style F fill:#ff9,stroke:#333
style G fill:#ff9,stroke:#333
style H fill:#bbf,stroke:#333
style I fill:#9f9,stroke:#333
style J fill:#f96,stroke:#333
style K fill:#99f,stroke:#333
style L fill:#99f,stroke:#333
style M fill:#f96,stroke:#333
style N fill:#f96,stroke:#333
style O fill:#f96,stroke:#333
style P fill:#9f9,stroke:#333
style Q fill:#f96,stroke:#333
style R fill:#f96,stroke:#333
```

**Diagram sources**
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)
- [tool_base.py](file://app/daytona/tool_base.py#L1-L139)
- [client.py](file://app/sandbox/client.py#L1-L202)
- [sandbox.py](file://app/sandbox/core/sandbox.py#L1-L463)
- [manager.py](file://app/sandbox/core/manager.py#L1-L314)
- [terminal.py](file://app/sandbox/core/terminal.py#L1-L347)
- [config.py](file://app/config.py#L1-L373)

**Section sources**
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L1-L223)
- [tool_base.py](file://app/daytona/tool_base.py#L1-L139)
- [client.py](file://app/sandbox/client.py#L1-L202)
- [sandbox.py](file://app/sandbox/core/sandbox.py#L1-L463)
- [manager.py](file://app/sandbox/core/manager.py#L1-L314)
- [terminal.py](file://app/sandbox/core/terminal.py#L1-L347)
- [config.py](file://app/config.py#L1-L373)

## Performance Considerations
The Sandbox Agent system is designed with performance in mind, balancing security requirements with execution efficiency. Container creation and startup times are optimized through the use of pre-pulled images and efficient initialization sequences. The sandbox manager implements automatic cleanup of idle sandboxes to conserve system resources. Command execution is handled asynchronously to prevent blocking operations, and file operations are optimized through direct Docker API calls rather than intermediate file transfers. The system also implements connection pooling and session reuse to minimize overhead for repeated operations. Resource limits are enforced at the container level to prevent any single sandbox from consuming excessive CPU or memory, ensuring fair resource allocation across multiple concurrent sandboxes.

## Troubleshooting Guide
When working with the Sandbox Agent, several common issues may arise. Container startup failures can occur due to missing Docker images, insufficient system resources, or network connectivity issues. These can typically be resolved by ensuring the required Docker image is available, verifying system resource availability, and checking network connectivity to the Docker daemon. Resource exhaustion issues may manifest as slow command execution or failed operations, which can be addressed by adjusting the sandbox configuration to reduce resource limits or by optimizing the code being executed. Network restrictions can prevent access to external services, which may require enabling network access in the sandbox configuration or using alternative approaches for external service integration. If encountering persistent issues, checking the system logs for detailed error messages can provide valuable diagnostic information.

**Section sources**
- [sandbox.py](file://app/sandbox/core/sandbox.py#L1-L463)
- [manager.py](file://app/sandbox/core/manager.py#L1-L314)
- [terminal.py](file://app/sandbox/core/terminal.py#L1-L347)
- [client.py](file://app/sandbox/client.py#L1-L202)

## Conclusion
The Sandbox Agent provides a robust and secure environment for executing untrusted code within isolated Docker containers. Its architecture combines a flexible agent interface with comprehensive sandbox management and specialized tool integrations to support a wide range of use cases, from evaluating third-party code to testing software in clean environments. The system's design emphasizes security through container isolation, resource limits, and network restrictions, while maintaining performance through efficient initialization and asynchronous operations. With its extensible tool framework and comprehensive configuration options, the Sandbox Agent offers a powerful solution for secure code execution in diverse scenarios.
# API Reference

<cite>
**Referenced Files in This Document**   
- [main.py](file://main.py)
- [run_mcp.py](file://run_mcp.py)
- [run_flow.py](file://run_flow.py)
- [sandbox_main.py](file://sandbox_main.py)
- [app/agent/manus.py](file://app/agent/manus.py)
- [app/config.py](file://app/config.py)
- [app/sandbox/core/manager.py](file://app/sandbox/core/manager.py)
- [app/sandbox/core/sandbox.py](file://app/sandbox/core/sandbox.py)
- [app/sandbox/client.py](file://app/sandbox/client.py)
- [app/tool/base.py](file://app/tool/base.py)
- [app/tool/tool_collection.py](file://app/tool/tool_collection.py)
- [app/tool/python_execute.py](file://app/tool/python_execute.py)
- [app/tool/file_operators.py](file://app/tool/file_operators.py)
- [app/tool/mcp.py](file://app/tool/mcp.py)
- [app/mcp/server.py](file://app/mcp/server.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Manus Agent API](#manus-agent-api)
3. [Tool Base Class Interface](#tool-base-class-interface)
4. [Sandbox API](#sandbox-api)
5. [Config Class](#config-class)
6. [MCP Server Endpoints](#mcp-server-endpoints)
7. [CLI Entry Points](#cli-entry-points)
8. [Authentication and Security](#authentication-and-security)
9. [Client Implementation Guidelines](#client-implementation-guidelines)
10. [Error Handling](#error-handling)

## Introduction
This document provides comprehensive API documentation for OpenManus, a versatile agent framework that supports both local and remote tool execution through the Model Context Protocol (MCP). The system enables task automation through intelligent agents that can execute code, browse the web, analyze data, and interact with various tools in sandboxed environments. The API exposes public interfaces for agent creation, tool management, sandbox operations, configuration access, and MCP server integration. The documentation covers method signatures, parameters, return values, exceptions, usage examples, and implementation guidelines for developers integrating with or extending the OpenManus framework.

## Manus Agent API
The Manus agent class serves as the primary interface for task execution in OpenManus. It inherits from ToolCallAgent and extends functionality to support MCP tools, browser context management, and initialization of external server connections. The agent orchestrates the execution of various tools to accomplish complex tasks through a ReAct-style loop of thinking and acting.

**Section sources**
- [app/agent/manus.py](file://app/agent/manus.py#L17-L164)

### create Method
Factory method to create and properly initialize a Manus instance with connections to configured MCP servers.

```python
@classmethod
async def create(cls, **kwargs) -> "Manus":
```

**Parameters**
- `**kwargs`: Keyword arguments passed to the class constructor

**Returns**
- `Manus`: Initialized Manus agent instance

**Exceptions**
- May propagate exceptions from `initialize_mcp_servers()`

**Usage Example**
```python
agent = await Manus.create()
```

**Section sources**
- [app/agent/manus.py](file://app/agent/manus.py#L59-L64)

### think Method
Process current state and decide next actions with appropriate context, including dynamic prompt adjustment when browser tools are in use.

```python
async def think(self) -> bool:
```

**Returns**
- `bool`: True if thinking produced actionable output, False otherwise

**Behavior**
- Automatically initializes MCP servers if not already initialized
- Modifies the next step prompt when browser interactions are detected
- Delegates to parent class implementation after context adjustment
- Restores the original prompt after thinking completes

**Section sources**
- [app/agent/manus.py](file://app/agent/manus.py#L139-L164)

### cleanup Method
Clean up Manus agent resources including browser context and MCP server connections.

```python
async def cleanup(self):
```

**Behavior**
- Cleans up browser context if initialized
- Disconnects from all MCP servers if the agent was initialized
- Resets initialization flag

**Section sources**
- [app/agent/manus.py](file://app/agent/manus.py#L130-L137)

### connect_mcp_server Method
Connect to an MCP server and add its tools to the agent's available tools collection.

```python
async def connect_mcp_server(
    self,
    server_url: str,
    server_id: str = "",
    use_stdio: bool = False,
    stdio_args: List[str] = None,
) -> None:
```

**Parameters**
- `server_url`: URL for SSE connection or command for stdio connection
- `server_id`: Optional identifier for the server connection
- `use_stdio`: Boolean indicating whether to use stdio transport
- `stdio_args`: Arguments for stdio command execution

**Behavior**
- Establishes connection via SSE or stdio based on parameters
- Tracks connected servers in connected_servers dictionary
- Adds only the new tools from this server to available_tools

**Section sources**
- [app/agent/manus.py](file://app/agent/manus.py#L90-L111)

### disconnect_mcp_server Method
Disconnect from an MCP server and remove its tools from the agent's available tools.

```python
async def disconnect_mcp_server(self, server_id: str = "") -> None:
```

**Parameters**
- `server_id`: Identifier of the server to disconnect from; if empty, disconnect from all servers

**Behavior**
- Disconnects from specified or all MCP servers
- Rebuilds available_tools collection without tools from disconnected servers
- Removes server from connected_servers tracking

**Section sources**
- [app/agent/manus.py](file://app/agent/manus.py#L113-L128)

## Tool Base Class Interface
The BaseTool class provides the foundation for all tools in OpenManus, implementing a standardized interface for tool execution, result handling, and function call formatting. Tools inherit from this class to ensure consistent behavior across the framework.

**Section sources**
- [app/tool/base.py](file://app/tool/base.py#L98-L181)

### ToolResult Class
Represents the result of a tool execution with support for output, error messages, base64-encoded images, and system messages.

```python
class ToolResult(BaseModel):
    output: Any = Field(default=None)
    error: Optional[str] = Field(default=None)
    base64_image: Optional[str] = Field(default=None)
    system: Optional[str] = Field(default=None)
```

**Methods**
- `__bool__()`: Returns True if any field has a value
- `__add__()`: Combines two ToolResult instances
- `__str__()`: String representation (error if present, otherwise output)
- `replace()`: Returns a new ToolResult with specified fields replaced

**Section sources**
- [app/tool/base.py](file://app/tool/base.py#L10-L85)

### BaseTool Abstract Class
Abstract base class for all tools that defines the standard interface and behavior.

```python
class BaseTool(ABC, BaseModel):
    name: str
    description: str
    parameters: Optional[dict] = None
```

**Abstract Methods**
- `execute(**kwargs) -> Any`: Abstract method that must be implemented by subclasses

**Concrete Methods**
- `__call__(**kwargs) -> Any`: Delegates to execute method
- `to_param() -> Dict`: Converts tool to OpenAI function calling format
- `success_response(data) -> ToolResult`: Creates a successful result
- `fail_response(msg) -> ToolResult`: Creates a failed result

**Section sources**
- [app/tool/base.py](file://app/tool/base.py#L98-L181)

### ToolCollection Class
Manages a collection of tools with methods for execution, addition, and parameter conversion.

```python
class ToolCollection:
    def __init__(self, *tools: BaseTool):
    def to_params(self) -> List[Dict[str, Any]]:
    async def execute(self, *, name: str, tool_input: Dict[str, Any] = None) -> ToolResult:
    def add_tool(self, tool: BaseTool):
    def add_tools(self, *tools: BaseTool):
```

**Section sources**
- [app/tool/tool_collection.py](file://app/tool/tool_collection.py#L11-L71)

## Sandbox API
The Sandbox API provides containerized execution environments for secure code execution with resource limits and file operations. It consists of DockerSandbox for individual sandbox instances, SandboxManager for managing multiple sandboxes, and LocalSandboxClient for high-level sandbox operations.

**Section sources**
- [app/sandbox/core/sandbox.py](file://app/sandbox/core/sandbox.py#L17-L461)
- [app/sandbox/core/manager.py](file://app/sandbox/core/manager.py#L13-L312)
- [app/sandbox/client.py](file://app/sandbox/client.py#L11-L201)

### DockerSandbox Class
Represents a single Docker containerized execution environment.

```python
class DockerSandbox:
    def __init__(
        self,
        config: Optional[SandboxSettings] = None,
        volume_bindings: Optional[Dict[str, str]] = None,
    ):
    async def create(self) -> "DockerSandbox":
    async def run_command(self, cmd: str, timeout: Optional[int] = None) -> str:
    async def read_file(self, path: str) -> str:
    async def write_file(self, path: str, content: str) -> None:
    async def copy_from(self, src_path: str, dst_path: str) -> None:
    async def copy_to(self, src_path: str, dst_path: str) -> None:
    async def cleanup(self) -> None:
```

**Section sources**
- [app/sandbox/core/sandbox.py](file://app/sandbox/core/sandbox.py#L17-L461)

### SandboxManager Class
Manages multiple DockerSandbox instances with automatic cleanup and concurrency control.

```python
class SandboxManager:
    def __init__(
        self,
        max_sandboxes: int = 100,
        idle_timeout: int = 3600,
        cleanup_interval: int = 300,
    ):
    async def create_sandbox(
        self,
        config: Optional[SandboxSettings] = None,
        volume_bindings: Optional[Dict[str, str]] = None,
    ) -> str:
    async def get_sandbox(self, sandbox_id: str) -> DockerSandbox:
    async def delete_sandbox(self, sandbox_id: str) -> None:
    async def cleanup(self) -> None:
    def get_stats(self) -> Dict:
```

**Section sources**
- [app/sandbox/core/manager.py](file://app/sandbox/core/manager.py#L13-L312)

### LocalSandboxClient Class
High-level client interface for sandbox operations.

```python
class LocalSandboxClient(BaseSandboxClient):
    async def create(
        self,
        config: Optional[SandboxSettings] = None,
        volume_bindings: Optional[Dict[str, str]] = None,
    ) -> None:
    async def run_command(self, command: str, timeout: Optional[int] = None) -> str:
    async def copy_from(self, container_path: str, local_path: str) -> None:
    async def copy_to(self, local_path: str, container_path: str) -> None:
    async def read_file(self, path: str) -> str:
    async def write_file(self, path: str, content: str) -> None:
    async def cleanup(self) -> None:
```

**Section sources**
- [app/sandbox/client.py](file://app/sandbox/client.py#L66-L201)

## Config Class
The Config class provides a singleton interface for accessing application settings from configuration files. It handles loading configuration from TOML files, managing default values, and providing type-safe access to various configuration sections.

**Section sources**
- [app/config.py](file://app/config.py#L196-L368)

### Configuration Properties
The Config class exposes several properties for accessing different configuration sections:

```python
@property
def llm(self) -> Dict[str, LLMSettings]:
    return self._config.llm

@property
def sandbox(self) -> SandboxSettings:
    return self._config.sandbox

@property
def browser_config(self) -> Optional[BrowserSettings]:
    return self._config.browser_config

@property
def search_config(self) -> Optional[SearchSettings]:
    return self._config.search_config

@property
def mcp_config(self) -> MCPSettings:
    return self._config.mcp_config

@property
def run_flow_config(self) -> RunflowSettings:
    return self._config.run_flow_config

@property
def workspace_root(self) -> Path:
    return WORKSPACE_ROOT

@property
def root_path(self) -> Path:
    return PROJECT_ROOT
```

**Section sources**
- [app/config.py](file://app/config.py#L331-L371)

### Configuration Loading
The Config class implements lazy loading of configuration with the following methods:

```python
def _get_config_path() -> Path:
def _load_config(self) -> dict:
def _load_initial_config(self):
```

The configuration is loaded from `config/config.toml` if it exists, otherwise from `config/config.example.toml`. The configuration includes settings for LLMs, sandbox, browser, search, MCP, run flow, and Daytona.

**Section sources**
- [app/config.py](file://app/config.py#L217-L230)

## MCP Server Endpoints
The MCP (Model Context Protocol) server implementation provides a standardized interface for tool access over different transport mechanisms. The server allows remote clients to discover and execute tools through a well-defined API.

**Section sources**
- [app/mcp/server.py](file://app/mcp/server.py#L28-L180)

### MCPServer Class
Main server class that manages tool registration and execution.

```python
class MCPServer:
    def __init__(self, name: str = "openmanus"):
    def register_tool(self, tool: BaseTool, method_name: Optional[str] = None) -> None:
    def register_all_tools(self) -> None:
    def run(self, transport: str = "stdio") -> None:
    async def cleanup(self) -> None:
```

**Section sources**
- [app/mcp/server.py](file://app/mcp/server.py#L28-L180)

### Tool Registration
The server supports dynamic tool registration with parameter validation and documentation:

```python
def register_tool(self, tool: BaseTool, method_name: Optional[str] = None) -> None:
```

When a tool is registered, the server:
- Creates an async wrapper method for execution
- Sets method metadata (name, docstring, signature)
- Stores parameter schema for programmatic access
- Registers the method with FastMCP server

**Section sources**
- [app/mcp/server.py](file://app/mcp/server.py#L58-L117)

### Transport Support
The server supports different transport mechanisms for client communication:

```python
def run(self, transport: str = "stdio") -> None:
```

Supported transports:
- `stdio`: Standard input/output for local process communication
- `sse`: Server-Sent Events for HTTP-based streaming

The server can be started with the desired transport method, with stdio as the default.

**Section sources**
- [app/mcp/server.py](file://app/mcp/server.py#L168-L178)

### Built-in Tools
The MCP server includes several built-in tools:
- `bash`: Execute shell commands
- `browser`: Browser automation
- `editor`: Text replacement operations
- `terminate`: Agent termination

These tools are automatically initialized when the server is created.

**Section sources**
- [app/mcp/server.py](file://app/mcp/server.py#L42-L56)

## CLI Entry Points
OpenManus provides several command-line interfaces for different use cases, each with specific arguments and usage patterns.

**Section sources**
- [main.py](file://main.py#L1-L37)
- [run_mcp.py](file://run_mcp.py#L1-L117)
- [run_flow.py](file://run_flow.py#L1-L53)
- [sandbox_main.py](file://sandbox_main.py#L1-L37)

### main.py
Primary entry point for running the Manus agent with a prompt.

```python
parser = argparse.ArgumentParser(description="Run Manus agent with a prompt")
parser.add_argument(
    "--prompt", type=str, required=False, help="Input prompt for the agent"
)
```

**Usage Patterns**
- Interactive mode: No arguments, prompts for input
- Direct execution: `--prompt "your prompt here"`

**Section sources**
- [main.py](file://main.py#L1-L37)

### run_mcp.py
Entry point for running the MCP agent with connection to MCP servers.

```python
parser = argparse.ArgumentParser(description="Run the MCP Agent")
parser.add_argument(
    "--connection",
    "-c",
    choices=["stdio", "sse"],
    default="stdio",
    help="Connection type: stdio or sse",
)
parser.add_argument(
    "--server-url",
    default="http://127.0.0.1:8000/sse",
    help="URL for SSE connection",
)
parser.add_argument(
    "--interactive", "-i", action="store_true", help="Run in interactive mode"
)
parser.add_argument("--prompt", "-p", help="Single prompt to execute and exit")
```

**Usage Patterns**
- Interactive mode: `python run_mcp.py -i`
- Single prompt: `python run_mcp.py -p "your prompt"`
- SSE connection: `python run_mcp.py -c sse --server-url http://localhost:8000/sse`

**Section sources**
- [run_mcp.py](file://run_mcp.py#L1-L117)

### run_flow.py
Entry point for running flow-based execution with planning and optional data analysis.

```python
parser = argparse.ArgumentParser(description="Run the MCP Agent")
parser.add_argument(
    "--connection",
    "-c",
    choices=["stdio", "sse"],
    default="stdio",
    help="Connection type: stdio or sse",
)
parser.add_argument(
    "--server-url",
    default="http://127.0.0.1:8000/sse",
    help="URL for SSE connection",
)
parser.add_argument(
    "--interactive", "-i", action="store_true", help="Run in interactive mode"
)
parser.add_argument("--prompt", "-p", help="Single prompt to execute and exit")
```

**Usage Patterns**
- Default execution: `python run_flow.py`
- With prompt: `python run_flow.py --prompt "your prompt"`

**Section sources**
- [run_flow.py](file://run_flow.py#L1-L53)

### sandbox_main.py
Entry point for running the SandboxManus agent with sandboxed execution.

```python
parser = argparse.ArgumentParser(description="Run Manus agent with a prompt")
parser.add_argument(
    "--prompt", type=str, required=False, help="Input prompt for the agent"
)
```

**Usage Patterns**
- Interactive mode: No arguments, prompts for input
- Direct execution: `--prompt "your prompt here"`

**Section sources**
- [sandbox_main.py](file://sandbox_main.py#L1-L37)

## Authentication and Security
OpenManus implements several security measures to protect system resources and ensure safe execution of agent tasks.

### Sandbox Security
The sandbox environment provides isolation for code execution with the following security features:
- Resource limits (CPU, memory)
- Optional network isolation
- Working directory restrictions
- Path traversal prevention in file operations
- Command execution timeouts

The SandboxSettings configuration allows controlling these security aspects through parameters like `network_enabled`, `memory_limit`, `cpu_limit`, and `timeout`.

**Section sources**
- [app/sandbox/core/sandbox.py](file://app/sandbox/core/sandbox.py#L31-L46)
- [app/config.py](file://app/config.py#L148-L158)

### Tool Execution Security
The PythonExecute tool implements security restrictions for code execution:
- Code runs in a separate process
- Timeout enforcement to prevent infinite loops
- Limited built-in functions
- Output captured through redirected stdout
- No access to return values (only printed output)

**Section sources**
- [app/tool/python_execute.py](file://app/tool/python_execute.py#L1-L76)

### Configuration Security
The configuration system protects sensitive data through:
- External configuration files (not hardcoded)
- Support for environment variables through configuration
- Secure handling of API keys in LLM settings
- Proxy configuration for network requests

**Section sources**
- [app/config.py](file://app/config.py#L81-L93)

## Client Implementation Guidelines
This section provides guidelines for implementing clients that interact with the OpenManus API.

### Agent Initialization
When creating a Manus agent, use the async create() factory method to ensure proper initialization:

```python
agent = await Manus.create()
```

This ensures that MCP servers are properly connected before the agent is used.

**Section sources**
- [app/agent/manus.py](file://app/agent/manus.py#L59-L64)

### Resource Management
Always clean up agent resources when finished to prevent resource leaks:

```python
try:
    await agent.run(prompt)
finally:
    await agent.cleanup()
```

This pattern ensures that browser contexts and MCP connections are properly closed even if an error occurs.

**Section sources**
- [main.py](file://main.py#L33-L36)

### Error Handling
Implement comprehensive error handling for all async operations, particularly for:
- Network operations (MCP server connections)
- File operations (sandbox file access)
- Code execution (PythonExecute tool)
- Configuration loading

Use try-except blocks to catch specific exceptions and provide meaningful error messages.

**Section sources**
- [app/agent/manus.py](file://app/agent/manus.py#L66-L88)

### Configuration Access
Access configuration through the singleton config instance:

```python
from app.config import config

# Access different configuration sections
llm_settings = config.llm
sandbox_settings = config.sandbox
mcp_settings = config.mcp_config
```

This ensures consistent configuration access throughout the application.

**Section sources**
- [app/config.py](file://app/config.py#L371-L371)

### Sandbox Usage
When using the sandbox API, follow these best practices:
- Use context managers when possible for automatic cleanup
- Handle file operations through the client interface
- Monitor resource usage through manager statistics
- Implement proper error handling for file and command operations

**Section sources**
- [app/sandbox/client.py](file://app/sandbox/client.py#L11-L201)

## Error Handling
OpenManus implements a comprehensive error handling system to manage exceptions and provide meaningful feedback.

### ToolResult Pattern
The framework uses the ToolResult class to standardize error reporting:

```python
class ToolResult(BaseModel):
    output: Any = Field(default=None)
    error: Optional[str] = Field(default=None)
    base64_image: Optional[str] = Field(default=None)
    system: Optional[str] = Field(default=None)
```

Tools should return ToolResult instances with either output (success) or error (failure), but not both.

**Section sources**
- [app/tool/base.py](file://app/tool/base.py#L10-L85)

### Exception Types
The system defines several exception types for different error conditions:
- `SandboxTimeoutError`: Command execution timeout in sandbox
- `ToolError`: General tool execution error
- `TokenLimitExceeded`: LLM token limit exceeded
- `RetryError`: Error that can be retried

**Section sources**
- [app/sandbox/core/exceptions.py](file://app/sandbox/core/exceptions.py#L1-L10)
- [app/exceptions.py](file://app/exceptions.py#L1-L10)

### Error Propagation
Errors are propagated through the system with appropriate context:
- Tool execution errors are caught and returned as ToolResult
- Network errors are caught and logged with connection details
- Configuration errors are raised during initialization
- Resource errors are handled with cleanup and retry logic

**Section sources**
- [app/agent/toolcall.py](file://app/agent/toolcall.py#L17-L249)

### Logging Strategy
The framework uses structured logging to capture error information:
- All errors are logged with context (tool name, parameters)
- Stack traces are captured for debugging
- Performance metrics are logged for slow operations
- User-facing messages are separated from technical details

**Section sources**
- [app/logger.py](file://app/logger.py#L1-L20)
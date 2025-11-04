# API Reference

<cite>
**Referenced Files in This Document**   
- [manus.py](file://app/agent/manus.py)
- [base.py](file://app/tool/base.py)
- [config.py](file://app/config.py)
- [server.py](file://app/mcp/server.py)
- [sandbox.py](file://app/sandbox/core/sandbox.py)
- [manager.py](file://app/sandbox/core/manager.py)
- [sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py)
- [sb_shell_tool.py](file://app/tool/sandbox/sb_shell_tool.py)
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py)
- [sb_vision_tool.py](file://app/tool/sandbox/sb_vision_tool.py)
</cite>

## Table of Contents
1. [Agent API](#agent-api)
2. [Tool API](#tool-api)
3. [Sandbox API](#sandbox-api)
4. [Configuration API](#configuration-api)
5. [MCP API](#mcp-api)

## Agent API

The Agent API provides the core interface for the Manus agent, a versatile general-purpose agent capable of solving various tasks using multiple tools, including MCP-based tools.

### Manus Class Interface

The `Manus` class inherits from `ToolCallAgent` and provides a comprehensive interface for agent operations.

**Properties:**
- `name`: The agent's name, defaults to "Manus"
- `description`: A description of the agent's capabilities
- `system_prompt`: The system prompt used by the agent, formatted with the workspace root directory
- `next_step_prompt`: The prompt used to determine the next step in the agent's workflow
- `max_observe`: Maximum number of observations (default: 10000)
- `max_steps`: Maximum number of steps the agent can take (default: 20)
- `mcp_clients`: MCP clients for remote tool access
- `available_tools`: Collection of available tools
- `special_tool_names`: List of special tool names, including "Terminate"
- `browser_context_helper`: Helper for browser context management
- `connected_servers`: Dictionary mapping server IDs to URLs/commands
- `_initialized`: Internal flag indicating initialization status

**Methods:**

#### create()
```python
@classmethod
async def create(cls, **kwargs) -> "Manus":
```
Factory method to create and properly initialize a Manus instance.

**Parameters:**
- `**kwargs`: Keyword arguments to pass to the constructor

**Returns:**
- An initialized Manus instance

**Exceptions:**
- May raise exceptions during initialization of MCP servers

**Usage Example:**
```python
agent = await Manus.create()
```

#### think()
```python
async def think(self) -> bool:
```
Process current state and decide next actions with appropriate context.

**Returns:**
- Boolean indicating whether the agent should continue thinking

**Behavior:**
- Initializes MCP servers if not already initialized
- Adjusts the next step prompt based on browser usage
- Calls the parent class's think method
- Restores the original prompt after execution

**Usage Example:**
```python
should_continue = await agent.think()
```

#### cleanup()
```python
async def cleanup(self):
```
Clean up Manus agent resources.

**Behavior:**
- Cleans up the browser context if available
- Disconnects from all MCP servers if the agent was initialized
- Resets the initialization flag

**Usage Example:**
```python
await agent.cleanup()
```

#### connect_mcp_server()
```python
async def connect_mcp_server(
    self,
    server_url: str,
    server_id: str = "",
    use_stdio: bool = False,
    stdio_args: List[str] = None,
) -> None:
```
Connect to an MCP server and add its tools.

**Parameters:**
- `server_url`: URL or command for the MCP server
- `server_id`: Optional server identifier
- `use_stdio`: Whether to use stdio transport
- `stdio_args`: Arguments for stdio command

**Behavior:**
- Connects to the MCP server using SSE or stdio
- Updates the connected servers dictionary
- Adds new tools from the server to the available tools collection

**Usage Example:**
```python
await agent.connect_mcp_server("http://localhost:8080", "my_server")
```

#### disconnect_mcp_server()
```python
async def disconnect_mcp_server(self, server_id: str = "") -> None:
```
Disconnect from an MCP server and remove its tools.

**Parameters:**
- `server_id`: Optional server identifier. If empty, disconnects from all servers

**Behavior:**
- Disconnects from the specified MCP server
- Removes the server from the connected servers dictionary
- Rebuilds the available tools collection without the disconnected server's tools

**Usage Example:**
```python
await agent.disconnect_mcp_server("my_server")
```

**Section sources**
- [manus.py](file://app/agent/manus.py#L17-L164)

## Tool API

The Tool API provides the base interface for all tools in the system, along with specific implementations for various capabilities.

### BaseTool Class

The `BaseTool` class serves as the foundation for all tools in the system, combining Pydantic model validation with standardized tool functionality.

```python
class BaseTool(ABC, BaseModel):
```

**Properties:**
- `name`: Tool name
- `description`: Tool description
- `parameters`: Optional dictionary defining the tool's parameters schema

**Methods:**

#### execute()
```python
@abstractmethod
async def execute(self, **kwargs) -> Any:
```
Abstract method that must be implemented by all tool subclasses to execute the tool's functionality.

**Parameters:**
- `**kwargs`: Tool-specific parameters

**Returns:**
- Tool execution result

#### to_param()
```python
def to_param(self) -> Dict:
```
Convert the tool to function call format compatible with OpenAI function calling.

**Returns:**
- Dictionary with tool metadata in OpenAI function calling format

#### success_response()
```python
def success_response(self, data: Union[Dict[str, Any], str]) -> ToolResult:
```
Create a successful tool result.

**Parameters:**
- `data`: Result data (dictionary or string)

**Returns:**
- ToolResult with success=True and formatted output

#### fail_response()
```python
def fail_response(self, msg: str) -> ToolResult:
```
Create a failed tool result.

**Parameters:**
- `msg`: Error message describing the failure

**Returns:**
- ToolResult with success=False and error message

#### __call__()
```python
async def __call__(self, **kwargs) -> Any:
```
Execute the tool with given parameters by calling the execute method.

**Parameters:**
- `**kwargs`: Parameters to pass to the execute method

**Returns:**
- Result of the execute method

**Section sources**
- [base.py](file://app/tool/base.py#L77-L172)

### ToolResult Class

The `ToolResult` class represents the result of a tool execution.

```python
class ToolResult(BaseModel):
```

**Properties:**
- `output`: Output data from the tool
- `error`: Optional error message
- `base64_image`: Optional base64-encoded image
- `system`: Optional system message

**Methods:**
- `__bool__()`: Returns True if any field has a value
- `__add__()`: Combines two ToolResult instances
- `__str__()`: String representation of the result
- `replace()`: Returns a new ToolResult with specified fields replaced

## Sandbox API

The Sandbox API provides container management, command execution, and file operations within a secure Docker environment.

### DockerSandbox Class

The `DockerSandbox` class provides a containerized execution environment with resource limits, file operations, and command execution capabilities.

```python
class DockerSandbox:
```

**Properties:**
- `config`: Sandbox configuration
- `volume_bindings`: Volume mapping configuration
- `client`: Docker client
- `container`: Docker container instance
- `terminal`: Container terminal interface

**Methods:**

#### create()
```python
async def create(self) -> "DockerSandbox":
```
Creates and starts the sandbox container.

**Returns:**
- Current sandbox instance

**Exceptions:**
- `docker.errors.APIError`: If Docker API call fails
- `RuntimeError`: If container creation or startup fails

**Usage Example:**
```python
sandbox = await DockerSandbox().create()
```

#### run_command()
```python
async def run_command(self, cmd: str, timeout: Optional[int] = None) -> str:
```
Runs a command in the sandbox.

**Parameters:**
- `cmd`: Command to execute
- `timeout`: Timeout in seconds

**Returns:**
- Command output as string

**Exceptions:**
- `RuntimeError`: If sandbox not initialized or command execution fails
- `TimeoutError`: If command execution times out

**Usage Example:**
```python
output = await sandbox.run_command("ls -la")
```

#### read_file()
```python
async def read_file(self, path: str) -> str:
```
Reads a file from the container.

**Parameters:**
- `path`: File path

**Returns:**
- File contents as string

**Exceptions:**
- `FileNotFoundError`: If file does not exist
- `RuntimeError`: If read operation fails

**Usage Example:**
```python
content = await sandbox.read_file("/workspace/data.txt")
```

#### write_file()
```python
async def write_file(self, path: str, content: str) -> None:
```
Writes content to a file in the container.

**Parameters:**
- `path`: Target path
- `content`: File content

**Exceptions:**
- `RuntimeError`: If write operation fails

**Usage Example:**
```python
await sandbox.write_file("/workspace/output.txt", "Hello World")
```

#### copy_from()
```python
async def copy_from(self, src_path: str, dst_path: str) -> None:
```
Copies a file from the container to the host.

**Parameters:**
- `src_path`: Source file path (container)
- `dst_path`: Destination path (host)

**Exceptions:**
- `FileNotFoundError`: If source file does not exist
- `RuntimeError`: If copy operation fails

**Usage Example:**
```python
await sandbox.copy_from("/workspace/report.pdf", "/local/reports/report.pdf")
```

#### copy_to()
```python
async def copy_to(self, src_path: str, dst_path: str) -> None:
```
Copies a file from the host to the container.

**Parameters:**
- `src_path`: Source file path (host)
- `dst_path`: Destination path (container)

**Exceptions:**
- `FileNotFoundError`: If source file does not exist
- `RuntimeError`: If copy operation fails

**Usage Example:**
```python
await sandbox.copy_to("/local/data.csv", "/workspace/data.csv")
```

#### cleanup()
```python
async def cleanup(self) -> None:
```
Cleans up sandbox resources, including stopping and removing the container.

**Usage Example:**
```python
await sandbox.cleanup()
```

**Section sources**
- [sandbox.py](file://app/sandbox/core/sandbox.py#L17-L461)

### SandboxManager Class

The `SandboxManager` class manages multiple DockerSandbox instances lifecycle including creation, monitoring, and cleanup.

```python
class SandboxManager:
```

**Properties:**
- `max_sandboxes`: Maximum allowed number of sandboxes
- `idle_timeout`: Sandbox idle timeout in seconds
- `cleanup_interval`: Cleanup check interval in seconds
- `_sandboxes`: Active sandbox instance mapping
- `_last_used`: Last used time record for sandboxes

**Methods:**

#### create_sandbox()
```python
async def create_sandbox(
    self,
    config: Optional[SandboxSettings] = None,
    volume_bindings: Optional[Dict[str, str]] = None,
) -> str:
```
Creates a new sandbox instance.

**Parameters:**
- `config`: Sandbox configuration
- `volume_bindings`: Volume mapping configuration

**Returns:**
- Sandbox ID

**Exceptions:**
- `RuntimeError`: If max sandbox count reached or creation fails

**Usage Example:**
```python
sandbox_id = await manager.create_sandbox()
```

#### get_sandbox()
```python
async def get_sandbox(self, sandbox_id: str) -> DockerSandbox:
```
Gets a sandbox instance by ID.

**Parameters:**
- `sandbox_id`: Sandbox ID

**Returns:**
- DockerSandbox instance

**Exceptions:**
- `KeyError`: If sandbox does not exist

**Usage Example:**
```python
sandbox = await manager.get_sandbox(sandbox_id)
```

#### delete_sandbox()
```python
async def delete_sandbox(self, sandbox_id: str) -> None:
```
Deletes specified sandbox.

**Parameters:**
- `sandbox_id`: Sandbox ID

**Usage Example:**
```python
await manager.delete_sandbox(sandbox_id)
```

#### cleanup()
```python
async def cleanup(self) -> None:
```
Cleans up all resources, including all sandboxes.

**Usage Example:**
```python
await manager.cleanup()
```

**Section sources**
- [manager.py](file://app/sandbox/core/manager.py#L17-L313)

## Configuration API

The Configuration API provides access to application settings and validation mechanisms through the Config class.

### Config Class

The `Config` class implements a singleton pattern to manage application configuration loaded from TOML files.

```python
class Config:
```

**Properties:**

#### llm
```python
@property
def llm(self) -> Dict[str, LLMSettings]:
```
Get the LLM configuration.

**Returns:**
- Dictionary of LLM settings

#### sandbox
```python
@property
def sandbox(self) -> SandboxSettings:
```
Get the sandbox configuration.

**Returns:**
- SandboxSettings instance

#### daytona
```python
@property
def daytona(self) -> DaytonaSettings:
```
Get the Daytona configuration.

**Returns:**
- DaytonaSettings instance

#### browser_config
```python
@property
def browser_config(self) -> Optional[BrowserSettings]:
```
Get the browser configuration.

**Returns:**
- Optional BrowserSettings instance

#### search_config
```python
@property
def search_config(self) -> Optional[SearchSettings]:
```
Get the search configuration.

**Returns:**
- Optional SearchSettings instance

#### mcp_config
```python
@property
def mcp_config(self) -> MCPSettings:
```
Get the MCP configuration.

**Returns:**
- MCPSettings instance

#### run_flow_config
```python
@property
def run_flow_config(self) -> RunflowSettings:
```
Get the Run Flow configuration.

**Returns:**
- RunflowSettings instance

#### workspace_root
```python
@property
def workspace_root(self) -> Path:
```
Get the workspace root directory.

**Returns:**
- Path to the workspace root

#### root_path
```python
@property
def root_path(self) -> Path:
```
Get the root path of the application.

**Returns:**
- Path to the application root

**Section sources**
- [config.py](file://app/config.py#L196-L368)

## MCP API

The MCP API provides server endpoints, SSE/stdio protocols, and tool registration interfaces for Model Context Protocol communication.

### MCPServer Class

The `MCPServer` class implements an MCP server with tool registration and management capabilities.

```python
class MCPServer:
```

**Properties:**
- `server`: FastMCP server instance
- `tools`: Dictionary mapping tool names to BaseTool instances

**Methods:**

#### register_tool()
```python
def register_tool(self, tool: BaseTool, method_name: Optional[str] = None) -> None:
```
Register a tool with parameter validation and documentation.

**Parameters:**
- `tool`: BaseTool instance to register
- `method_name`: Optional method name for registration

**Behavior:**
- Converts tool parameters to function format
- Creates an async wrapper function with proper metadata
- Builds docstring and signature from tool function metadata
- Registers the tool with the server

**Usage Example:**
```python
server.register_tool(PythonExecute())
```

#### register_all_tools()
```python
def register_all_tools(self) -> None:
```
Register all tools with the server.

**Behavior:**
- Iterates through all tools in the tools dictionary and registers each one

**Usage Example:**
```python
server.register_all_tools()
```

#### run()
```python
def run(self, transport: str = "stdio") -> None:
```
Run the MCP server.

**Parameters:**
- `transport`: Communication method (default: "stdio")

**Behavior:**
- Registers all tools
- Sets up cleanup function using atexit
- Starts the server with the specified transport

**Usage Example:**
```python
server.run(transport="stdio")
```

#### cleanup()
```python
async def cleanup(self) -> None:
```
Clean up server resources.

**Behavior:**
- Cleans up browser tool resources if present

**Usage Example:**
```python
await server.cleanup()
```

**Section sources**
- [server.py](file://app/mcp/server.py#L23-L159)

### Sandbox Tool Implementations

The MCP API includes several sandbox-based tool implementations that extend the functionality of the system.

#### SandboxFilesTool
The `SandboxFilesTool` provides file operations within a secure sandboxed environment.

**Actions:**
- `create_file`: Create a new file
- `str_replace`: Replace specific text in a file
- `full_file_rewrite`: Completely rewrite an existing file
- `delete_file`: Remove a file

**Usage Example:**
```python
tool = SandboxFilesTool(sandbox)
result = await tool.execute(action="create_file", file_path="test.txt", file_contents="Hello World")
```

#### SandboxShellTool
The `SandboxShellTool` executes shell commands in the workspace directory using tmux sessions.

**Actions:**
- `execute_command`: Execute a shell command
- `check_command_output`: Check output of a running command
- `terminate_command`: Terminate a running command
- `list_commands`: List active tmux sessions

**Usage Example:**
```python
tool = SandboxShellTool(sandbox)
result = await tool.execute(action="execute_command", command="ls -la", blocking=True)
```

#### SandboxBrowserTool
The `SandboxBrowserTool` provides browser automation capabilities within the sandbox.

**Actions:**
- `navigate_to`: Navigate to a URL
- `go_back`: Go back in browser history
- `click_element`: Click an element by index
- `input_text`: Input text into a form field
- `scroll_down/up`: Scroll the page
- `switch_tab`: Switch between browser tabs

**Usage Example:**
```python
tool = SandboxBrowserTool(sandbox)
result = await tool.execute(action="navigate_to", url="https://example.com")
```

#### SandboxVisionTool
The `SandboxVisionTool` allows reading image files within the sandbox.

**Actions:**
- `see_image`: Read and compress an image file

**Usage Example:**
```python
tool = SandboxVisionTool(sandbox)
result = await tool.execute(action="see_image", file_path="screenshot.png")
```

**Section sources**
- [sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py#L17-L361)
- [sb_shell_tool.py](file://app/tool/sandbox/sb_shell_tool.py#L17-L419)
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py#L17-L450)
- [sb_vision_tool.py](file://app/tool/sandbox/sb_vision_tool.py#L17-L178)
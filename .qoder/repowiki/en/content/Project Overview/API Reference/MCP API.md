# MCP API

<cite>
**Referenced Files in This Document**   
- [app/mcp/server.py](file://app/mcp/server.py)
- [app/tool/mcp.py](file://app/tool/mcp.py)
- [run_mcp_server.py](file://run_mcp_server.py)
- [app/config.py](file://app/config.py)
- [config/mcp.example.json](file://config/mcp.example.json)
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
This document provides comprehensive API documentation for the Model Context Protocol (MCP) implementation in OpenManus. It details the MCP server endpoints, including `/mcp/v1/server-info` and `/mcp/v1/subscribe` for Server-Sent Events (SSE) streaming, with request/response schemas, authentication methods, and error codes. The document explains the server implementation using FastAPI and SSE for real-time tool invocation, and covers the MCP client functionality in `app/tool/mcp.py` for connecting to remote tools. It includes examples of starting the MCP server via `run_mcp_server.py`, registering tools, and handling incoming requests. The document also addresses protocol versioning, compatibility considerations, and integration with the A2A protocol, providing client implementation guidelines and troubleshooting tips for connection issues and message serialization errors.

## Project Structure
The OpenManus project is organized into several key directories, each serving a specific purpose in the MCP implementation. The `app/mcp/` directory contains the server implementation, while `app/tool/` houses the client-side tools and utilities. Configuration files are located in the `config/` directory, and the root directory includes scripts for running the server and client.

```mermaid
graph TD
subgraph "Root Directory"
run_mcp_server_py[run_mcp_server.py]
run_mcp_py[run_mcp.py]
config_dir[config/]
app_dir[app/]
end
subgraph "App Directory"
mcp_dir[app/mcp/]
tool_dir[app/tool/]
agent_dir[app/agent/]
end
subgraph "MCP Directory"
server_py[server.py]
end
subgraph "Tool Directory"
mcp_py[app/tool/mcp.py]
end
subgraph "Config Directory"
mcp_example_json[mcp.example.json]
end
run_mcp_server_py --> server_py
server_py --> mcp_dir
mcp_dir --> app_dir
app_dir --> tool_dir
tool_dir --> mcp_py
config_dir --> mcp_example_json
```

**Diagram sources**
- [run_mcp_server.py](file://run_mcp_server.py)
- [app/mcp/server.py](file://app/mcp/server.py)
- [app/tool/mcp.py](file://app/tool/mcp.py)
- [config/mcp.example.json](file://config/mcp.example.json)

**Section sources**
- [run_mcp_server.py](file://run_mcp_server.py)
- [app/mcp/server.py](file://app/mcp/server.py)
- [app/tool/mcp.py](file://app/tool/mcp.py)
- [config/mcp.example.json](file://config/mcp.example.json)

## Core Components
The core components of the MCP implementation in OpenManus include the MCP server, the MCP client, and the configuration system. The MCP server is responsible for exposing tools via the Model Context Protocol, while the MCP client connects to these servers and manages the available tools. The configuration system allows for flexible setup of multiple MCP servers with different connection types.

**Section sources**
- [app/mcp/server.py](file://app/mcp/server.py)
- [app/tool/mcp.py](file://app/tool/mcp.py)
- [app/config.py](file://app/config.py)

## Architecture Overview
The MCP architecture in OpenManus is designed to facilitate real-time tool invocation through a server-client model. The server exposes endpoints for server information and SSE streaming, while the client connects to these endpoints to access and execute tools. The architecture supports both SSE and stdio transport methods, allowing for flexible deployment options.

```mermaid
graph TD
Client[Client Application]
MCPClient[MCP Client]
MCPServer[MCP Server]
Tools[Registered Tools]
Client --> MCPClient
MCPClient --> MCPServer
MCPServer --> Tools
subgraph "MCP Server"
MCPServer
Tools
end
subgraph "Client"
Client
MCPClient
end
```

**Diagram sources**
- [app/mcp/server.py](file://app/mcp/server.py)
- [app/tool/mcp.py](file://app/tool/mcp.py)

## Detailed Component Analysis
### MCP Server Analysis
The MCP server is implemented in `app/mcp/server.py` and is responsible for managing tool registration and handling incoming requests. It uses the FastMCP library to create a FastAPI-based server that exposes endpoints for server information and SSE streaming.

#### For Object-Oriented Components:
```mermaid
classDiagram
class MCPServer {
+server : FastMCP
+tools : Dict[str, BaseTool]
+__init__(name : str)
+register_tool(tool : BaseTool, method_name : Optional[str])
+register_all_tools()
+run(transport : str)
+cleanup()
+_build_docstring(tool_function : dict) str
+_build_signature(tool_function : dict) Signature
}
class FastMCP {
+tool()
+run(transport : str)
}
class BaseTool {
+name : str
+description : str
+parameters : dict
+execute(**kwargs) ToolResult
}
MCPServer --> FastMCP : "uses"
MCPServer --> BaseTool : "manages"
```

**Diagram sources**
- [app/mcp/server.py](file://app/mcp/server.py#L23-L159)

#### For API/Service Components:
```mermaid
sequenceDiagram
participant Client as "Client"
participant MCPClient as "MCP Client"
participant MCPServer as "MCP Server"
participant Tool as "Registered Tool"
Client->>MCPClient : connect_sse(server_url)
MCPClient->>MCPServer : initialize()
MCPServer->>MCPServer : list_tools()
MCPServer-->>MCPClient : ListToolsResult
MCPClient->>Client : Available tools
Client->>MCPClient : execute(tool_name, **kwargs)
MCPClient->>MCPServer : call_tool(tool_name, kwargs)
MCPServer->>Tool : execute(**kwargs)
Tool-->>MCPServer : ToolResult
MCPServer-->>MCPClient : ToolResult
MCPClient-->>Client : ToolResult
```

**Diagram sources**
- [app/mcp/server.py](file://app/mcp/server.py#L36-L75)
- [app/tool/mcp.py](file://app/tool/mcp.py#L49-L68)

### MCP Client Analysis
The MCP client is implemented in `app/tool/mcp.py` and is responsible for connecting to MCP servers and managing the available tools. It supports both SSE and stdio transport methods and provides a unified interface for tool execution.

#### For Object-Oriented Components:
```mermaid
classDiagram
class MCPClients {
+sessions : Dict[str, ClientSession]
+exit_stacks : Dict[str, AsyncExitStack]
+tool_map : Dict[str, MCPClientTool]
+tools : Tuple[MCPClientTool]
+connect_sse(server_url : str, server_id : str)
+connect_stdio(command : str, args : List[str], server_id : str)
+disconnect(server_id : str)
+_initialize_and_list_tools(server_id : str)
+_sanitize_tool_name(name : str) str
}
class MCPClientTool {
+session : Optional[ClientSession]
+server_id : str
+original_name : str
+execute(**kwargs) ToolResult
}
class ToolCollection {
+add_tool(tool : BaseTool)
+add_tools(*tools : BaseTool)
+get_tool(name : str) BaseTool
}
MCPClients --> MCPClientTool : "creates"
MCPClients --> ToolCollection : "extends"
```

**Diagram sources**
- [app/tool/mcp.py](file://app/tool/mcp.py#L36-L193)

#### For API/Service Components:
```mermaid
sequenceDiagram
participant Client as "Client"
participant MCPClients as "MCPClients"
participant MCPServer as "MCP Server"
Client->>MCPClients : connect_sse(server_url)
MCPClients->>MCPServer : sse_client(url)
MCPServer-->>MCPClients : streams
MCPClients->>MCPServer : ClientSession(*streams)
MCPServer-->>MCPClients : session
MCPClients->>MCPServer : initialize()
MCPServer-->>MCPClients : initialization response
MCPClients->>MCPServer : list_tools()
MCPServer-->>MCPClients : ListToolsResult
MCPClients->>Client : Tools registered
```

**Diagram sources**
- [app/tool/mcp.py](file://app/tool/mcp.py#L49-L68)

### Configuration Analysis
The MCP configuration is managed through the `MCPSettings` class in `app/config.py`, which loads server configurations from a JSON file. The configuration supports multiple servers with different connection types (SSE or stdio).

#### For Object-Oriented Components:
```mermaid
classDiagram
class MCPSettings {
+server_reference : str
+servers : Dict[str, MCPServerConfig]
+load_server_config() Dict[str, MCPServerConfig]
}
class MCPServerConfig {
+type : str
+url : Optional[str]
+command : Optional[str]
+args : List[str]
}
MCPSettings --> MCPServerConfig : "contains"
```

**Diagram sources**
- [app/config.py](file://app/config.py#L126-L159)

## Dependency Analysis
The MCP implementation in OpenManus has several key dependencies that enable its functionality. The server relies on the FastMCP library for creating the FastAPI-based server, while the client uses the mcp library for SSE and stdio connections. The configuration system depends on Pydantic for data validation and JSON parsing.

```mermaid
graph TD
MCPServer[app/mcp/server.py] --> FastMCP[FastMCP]
MCPServer --> BaseTool[BaseTool]
MCPServer --> Bash[Bash]
MCPServer --> BrowserUseTool[BrowserUseTool]
MCPServer --> StrReplaceEditor[StrReplaceEditor]
MCPServer --> Terminate[Terminate]
MCPClients[app/tool/mcp.py] --> ClientSession[ClientSession]
MCPClients --> sse_client[sse_client]
MCPClients --> stdio_client[stdio_client]
MCPClients --> ListToolsResult[ListToolsResult]
MCPClients --> TextContent[TextContent]
MCPClients --> BaseTool[BaseTool]
MCPClients --> ToolCollection[ToolCollection]
MCPSettings[app/config.py] --> MCPServerConfig[MCPServerConfig]
MCPSettings --> BaseModel[BaseModel]
MCPSettings --> Field[Field]
MCPSettings --> json[json]
FastMCP --> FastAPI[FastAPI]
ClientSession --> asyncio[asyncio]
sse_client --> aiohttp[aiohttp]
stdio_client --> asyncio[asyncio]
```

**Diagram sources**
- [app/mcp/server.py](file://app/mcp/server.py)
- [app/tool/mcp.py](file://app/tool/mcp.py)
- [app/config.py](file://app/config.py)

**Section sources**
- [app/mcp/server.py](file://app/mcp/server.py)
- [app/tool/mcp.py](file://app/tool/mcp.py)
- [app/config.py](file://app/config.py)

## Performance Considerations
The MCP implementation in OpenManus is designed with performance in mind. The use of SSE for real-time communication allows for efficient, low-latency tool invocation. The server is built on FastAPI, which is known for its high performance and scalability. The client uses asynchronous programming to handle multiple connections and tool executions concurrently.

The configuration system is optimized for quick loading and parsing of server configurations, minimizing startup time. The tool registration process is streamlined to reduce overhead, and the use of Pydantic for data validation ensures that input and output schemas are handled efficiently.

## Troubleshooting Guide
When working with the MCP implementation in OpenManus, several common issues may arise. This section provides guidance on troubleshooting connection issues, message serialization errors, and other common problems.

### Connection Issues
- **Ensure the MCP server is running**: Verify that the MCP server is started using `run_mcp_server.py` and is listening on the correct port.
- **Check the server URL**: Ensure that the server URL in the configuration file matches the actual server address.
- **Verify network connectivity**: Confirm that the client can reach the server over the network.

### Message Serialization Errors
- **Check JSON schema compliance**: Ensure that tool input and output schemas comply with the JSON schema defined in the tool registration.
- **Validate data types**: Make sure that the data types used in tool parameters match the expected types in the schema.
- **Handle special characters**: Be cautious with special characters in tool parameters, as they may need to be escaped or encoded.

### Configuration Issues
- **Validate the configuration file**: Ensure that the `mcp.json` file is correctly formatted and contains valid server configurations.
- **Check environment variables**: Verify that any required environment variables are set and accessible to the application.

**Section sources**
- [app/mcp/server.py](file://app/mcp/server.py)
- [app/tool/mcp.py](file://app/tool/mcp.py)
- [config/mcp.example.json](file://config/mcp.example.json)

## Conclusion
The MCP implementation in OpenManus provides a robust and flexible framework for real-time tool invocation through a server-client model. The use of FastAPI and SSE enables efficient, low-latency communication, while the configuration system allows for flexible deployment options. The client implementation provides a unified interface for accessing and executing tools, making it easy to integrate with various applications. By following the guidelines and best practices outlined in this document, developers can effectively use and extend the MCP implementation to meet their specific needs.
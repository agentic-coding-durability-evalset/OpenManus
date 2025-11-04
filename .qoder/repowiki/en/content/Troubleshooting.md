# Troubleshooting

<cite>
**Referenced Files in This Document**   
- [config.example.toml](file://config/config.example.toml)
- [mcp.example.json](file://config/mcp.example.json)
- [llm.py](file://app/llm.py)
- [sandbox.py](file://app/sandbox/core/sandbox.py)
- [manager.py](file://app/sandbox/core/manager.py)
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py)
- [python_execute.py](file://app/tool/python_execute.py)
- [file_operators.py](file://app/tool/file_operators.py)
- [mcp.py](file://app/tool/mcp.py)
- [manus.py](file://app/agent/manus.py)
- [logger.py](file://app/logger.py)
</cite>

## Table of Contents
1. [Configuration Errors](#configuration-errors)
2. [API Key Problems](#api-key-problems)
3. [Docker and Sandbox Failures](#docker-and-sandbox-failures)
4. [Tool Execution Timeouts](#tool-execution-timeouts)
5. [LLM Connectivity and Rate Limiting](#llm-connectivity-and-rate-limiting)
6. [Browser Automation Issues](#browser-automation-issues)
7. [MCP Connection and Protocol Errors](#mcp-connection-and-protocol-errors)
8. [Debugging Techniques](#debugging-techniques)
9. [Performance Troubleshooting](#performance-troubleshooting)

## Configuration Errors

This section addresses common configuration issues in OpenManus, particularly related to LLM settings, sandbox parameters, and MCP server configurations. Misconfigurations in the TOML or JSON configuration files can lead to initialization failures, unexpected behavior, or degraded performance.

The primary configuration file `config.example.toml` contains global LLM settings, optional vision model configurations, browser settings, search engine preferences, sandbox parameters, and MCP server references. Users should create a `config.toml` file based on this example, replacing placeholder values with actual credentials and settings. A common error occurs when users fail to rename `config.example.toml` to `config.toml`, resulting in the application using default or missing configuration values.

For MCP server configuration, the `mcp.example.json` file provides the correct structure for defining server connections. The configuration must specify the connection type ("sse" or "stdio") and corresponding URL or command. Missing or malformed JSON syntax, incorrect server IDs, or invalid connection types will prevent successful MCP server initialization.

**Section sources**
- [config.example.toml](file://config/config.example.toml)
- [mcp.example.json](file://config/mcp.example.json)

## API Key Problems

API key issues typically manifest as authentication failures when connecting to LLM providers such as Anthropic, OpenAI, or Azure. These problems are often indicated by error messages containing "Authentication failed" or "Invalid API key" in the logs.

The API key should be specified in the `[llm]` section of the configuration file under the `api_key` parameter. For AWS Bedrock, while an API key is required in the configuration, it is not actually used for authentication, which relies on AWS IAM credentials instead. Users should verify that their API key has the necessary permissions and has not expired. Rate limiting may also occur if the API key has reached its usage quota.

When troubleshooting API key issues, first verify that the key is correctly copied without extra whitespace. Then check that the `api_type` parameter matches the intended provider (e.g., "azure" for Azure OpenAI, "aws" for Amazon Bedrock). The application logs will provide specific error details that can help identify whether the issue is with authentication, authorization, or network connectivity.

**Section sources**
- [config.example.toml](file://config/config.example.toml)
- [llm.py](file://app/llm.py#L459-L496)

## Docker and Sandbox Failures

Sandbox failures are among the most common issues in OpenManus, typically related to Docker configuration, image availability, or resource constraints. The sandbox system uses Docker containers to provide isolated execution environments for running untrusted code.

Container startup failures often occur when the specified Docker image cannot be pulled from the registry. The `SandboxManager` in `manager.py` attempts to ensure the required image is available by pulling it if not found locally. Network issues, invalid image names, or authentication requirements for private registries can prevent successful image retrieval.

Resource limitations are another common cause of sandbox failures. The sandbox configuration allows setting memory limits (e.g., "1g") and CPU limits. If these limits are too restrictive for the intended operations, the container may fail to start or terminate unexpectedly. Users should verify that their Docker daemon has sufficient resources available and that the limits specified in the configuration are appropriate for their use case.

Network access configuration is also critical. By default, sandboxes may be created with network access disabled (`network_mode="none"`), which prevents internet connectivity. If network access is required for the task, the `network_enabled` flag must be set to `true` in the sandbox configuration.

**Section sources**
- [sandbox.py](file://app/sandbox/core/sandbox.py#L48-L87)
- [manager.py](file://app/sandbox/core/manager.py#L136-L174)
- [config.example.toml](file://config/config.example.toml)

## Tool Execution Timeouts

Tool execution timeouts occur when operations exceed their allocated time limits, particularly in the Python execution and command running tools. The `PythonExecute` tool has a default timeout of 5 seconds, which may be insufficient for complex computations or I/O operations.

When a timeout occurs, the process is terminated, and an error message indicating the timeout duration is returned. Users can adjust the timeout parameter when calling the tool, but should be cautious about setting excessively long timeouts that could impact system responsiveness.

For command execution in the sandbox environment, the default timeout is 120 seconds. This can be overridden by specifying a custom timeout value. The `run_command` method in `file_operators.py` handles the timeout logic by using `asyncio.wait_for` and properly cleaning up the process if it exceeds the time limit.

To resolve timeout issues, first determine whether the operation genuinely requires more time or if there is an underlying performance problem. For long-running operations, consider breaking them into smaller steps or implementing asynchronous processing. If the timeout persists even with extended durations, investigate potential blocking operations or resource contention.

**Section sources**
- [python_execute.py](file://app/tool/python_execute.py#L46-L74)
- [file_operators.py](file://app/tool/file_operators.py#L76-L112)

## LLM Connectivity and Rate Limiting

Connectivity issues with LLM providers can stem from network problems, incorrect endpoint URLs, or service outages. The application uses retry logic with exponential backoff (implemented via the `tenacity` library) to handle transient failures, attempting up to 6 times before giving up.

Rate limiting is a common issue when making frequent requests to LLM APIs. The application logs will indicate rate limit errors, typically with messages like "Rate limit exceeded." When this occurs, the system automatically applies exponential backoff between retry attempts. Users can mitigate rate limiting by reducing their request frequency, upgrading their API plan to allow higher limits, or implementing request batching.

The `ask` method in `llm.py` handles various OpenAI API errors, including `AuthenticationError`, `RateLimitError`, and general `APIError`. Each error type is logged appropriately, providing diagnostic information for troubleshooting. For persistent connectivity issues, verify that the `base_url` parameter in the configuration points to the correct API endpoint and that there are no firewall or proxy restrictions blocking the connection.

**Section sources**
- [llm.py](file://app/llm.py#L459-L496)
- [llm.py](file://app/llm.py#L590-L616)

## Browser Automation Issues

Browser automation problems typically involve the sandbox-based browser tool, which uses a headless browser instance running in a Docker container. Common issues include failure to navigate to URLs, problems with element interaction, and screenshot generation errors.

The `SandboxBrowserTool` validates base64-encoded screenshots to ensure they meet size and format requirements. Images exceeding 10MB in size or dimensions larger than 8192x8192 pixels will be rejected, and the image data will be removed from the response. This validation prevents memory issues and ensures compatibility with downstream processing.

Element interaction requires correct indexing, as the tool uses numeric indices to identify clickable elements on the page. If the page structure changes between requests, previously valid indices may become invalid. Users should always retrieve the current page state before attempting interactions to ensure they are using up-to-date element indices.

Network connectivity within the browser sandbox is controlled by the `network_enabled` configuration parameter. If browser navigation fails with connection errors, verify that network access is enabled in the sandbox settings. Additionally, ensure that the target websites are accessible from the Docker network and not blocked by any security policies.

**Section sources**
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py#L158-L182)
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py#L221-L231)

## MCP Connection and Protocol Errors

MCP (Model Context Protocol) connection issues can occur with both SSE (Server-Sent Events) and stdio transport types. The `MCPClients` class manages connections to multiple MCP servers and handles the initialization sequence, including session setup and tool discovery.

Connection failures are often due to incorrect server URLs, unreachable endpoints, or mismatched connection types. For SSE connections, verify that the server is running and accessible at the specified URL. For stdio connections, ensure that the command and arguments are correct and that the referenced server module exists.

Protocol errors may occur if there is a version mismatch between the client and server implementations of MCP. The initialization sequence involves exchanging capabilities and listing available tools. If this process fails, check that both client and server are using compatible versions of the MCP specification.

The application logs detailed information about MCP operations, including successful connections and any errors encountered. When troubleshooting, examine these logs to determine at which stage the connection is failing—whether during transport establishment, session initialization, or tool listing.

**Section sources**
- [mcp.py](file://app/tool/mcp.py#L36-L68)
- [manus.py](file://app/agent/manus.py#L66-L88)
- [mcp.example.json](file://config/mcp.example.json)

## Debugging Techniques

Effective debugging in OpenManus involves leveraging the logging system and analyzing execution traces. The application uses structured logging through `structlog`, with different output formats depending on the environment mode (local vs. production).

To enable verbose logging, set the `ENV_MODE` environment variable to "LOCAL", which activates the console renderer for more readable log output. The logger configuration in `logger.py` allows adjusting log levels for both console and file output, enabling detailed debugging information to be captured.

Execution traces can be analyzed by examining the sequence of tool calls and LLM interactions. Each tool execution is logged with relevant parameters and outcomes, allowing users to trace the agent's decision-making process. For complex issues, enabling debug-level logging will provide additional context about internal state changes and API interactions.

When investigating specific issues, search the logs for relevant error messages or warning indicators. The structured format of the logs makes it easier to filter and analyze large volumes of diagnostic information. Pay particular attention to stack traces and error codes, which can pinpoint the exact location and nature of failures.

**Section sources**
- [logger.py](file://app/logger.py#L0-L41)
- [utils/logger.py](file://app/utils/logger.py#L0-L31)

## Performance Troubleshooting

Performance issues in OpenManus typically manifest as slow agent responses or high resource utilization. These problems can stem from various sources, including inefficient tool usage, suboptimal LLM configurations, or resource-constrained execution environments.

One common performance bottleneck is excessive token usage in LLM interactions. The token counter in `llm.py` tracks both input and completion tokens, helping identify when conversations are approaching model limits. Long-running tasks that accumulate many messages in the context window can become increasingly slow and expensive.

To optimize performance, consider the following strategies:
- Limit the number of steps in agent workflows to prevent unnecessary iterations
- Use appropriate timeout values for tools to avoid hanging operations
- Monitor sandbox resource usage and adjust memory/CPU limits as needed
- Implement caching for frequently accessed data or repeated operations
- Optimize Python code executed in the sandbox for efficiency

For agents that use browser automation, minimize the number of page interactions and avoid unnecessary screenshot captures, as these operations can be resource-intensive. When processing large datasets, consider breaking operations into smaller batches rather than attempting to process everything in a single step.

**Section sources**
- [llm.py](file://app/llm.py#L185-L213)
- [python_execute.py](file://app/tool/python_execute.py#L0-L44)
- [file_operators.py](file://app/tool/file_operators.py#L138-L157)
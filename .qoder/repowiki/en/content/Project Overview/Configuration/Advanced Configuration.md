# Advanced Configuration

<cite>
**Referenced Files in This Document**   
- [config.example-daytona.toml](file://config/config.example-daytona.toml)
- [config.py](file://app/config.py)
- [sandbox.py](file://app/daytona/sandbox.py)
- [browser_use_tool.py](file://app/tool/browser_use_tool.py)
- [web_search.py](file://app/tool/web_search.py)
</cite>

## Table of Contents
1. [Daytona Integration Settings](#daytona-integration-settings)
2. [Browser Configuration Options](#browser-configuration-options)
3. [Search Engine Configuration](#search-engine-configuration)
4. [Runflow Settings for Multi-Agent Workflows](#runflow-settings-for-multi-agent-workflows)
5. [Configuration Merging and Source Hierarchy](#configuration-merging-and-source-hierarchy)
6. [Security Considerations and Best Practices](#security-considerations-and-best-practices)

## Daytona Integration Settings

The Daytona integration settings enable cloud-based workspace management and remote development environments through the Daytona platform. These settings are defined in the `DaytonaSettings` class within the configuration system and can be configured in the TOML configuration file.

The `daytona_api_key` setting is required for authentication with the Daytona API. This API key authenticates the application with the Daytona service, enabling access to cloud-based sandbox environments. Without a valid API key, the application cannot create or manage Daytona sandboxes.

The `daytona_server_url` specifies the endpoint for the Daytona API. By default, it points to "https://app.daytona.io/api", but can be customized for different deployment scenarios or regions. This setting allows the application to connect to different Daytona instances based on geographical location or specific deployment requirements.

The `sandbox_image_name` setting determines the Docker image used for the sandbox environment. The default value is "whitezxj/sandbox:0.1.0", which is specifically configured with all necessary tools and services for the application's functionality. Using a different image may result in missing tools or incompatible environments.

The `VNC_password` setting configures the password for VNC access to the sandbox environment, allowing users to visually interact with the remote development environment. If not specified, it defaults to "123456". This enables remote desktop access to the sandbox for debugging or monitoring purposes.

These settings work together to create fully managed cloud development environments that can be accessed remotely. When a sandbox is created through the `create_sandbox` function in `app/daytona/sandbox.py`, these configuration values are used to initialize the environment with the proper image, authentication, and access controls.

**Section sources**
- [config.py](file://app/config.py#L128-L137)
- [config.example-daytona.toml](file://config/config.example-daytona.toml#L97-L113)
- [sandbox.py](file://app/daytona/sandbox.py#L75-L105)

## Browser Configuration Options

Browser configuration options provide extensive control over browser behavior and integration within the application. These settings are defined in the `BrowserSettings` class and can be configured through the TOML configuration file.

The `headless` mode setting determines whether the browser runs in headless mode (without a visible UI). By default, this is set to `false`, allowing for visual inspection of browser activities. Setting this to `true` can improve performance and is typically used in automated environments where visual feedback is not required.

Proxy settings are configured through the nested `proxy` object, which includes `server`, `username`, and `password` fields. This allows the browser to route traffic through a proxy server, which can be useful for network isolation, security monitoring, or accessing resources behind corporate firewalls.

Chrome instance integration is supported through multiple configuration options. The `chrome_instance_path` setting allows connecting to an existing Chrome browser instance by specifying the path to the Chrome executable. This enables integration with a user's existing browser session, preserving cookies, extensions, and preferences.

Alternative connection methods include WebSocket (`wss_url`) and Chrome DevTools Protocol (`cdp_url`) endpoints, which allow connecting to remote or already-running browser instances. This is particularly useful for distributed environments where the browser runs on a different machine than the application.

Additional browser settings include `disable_security` (default: `true`), which disables certain browser security features to allow broader automation capabilities, and `extra_chromium_args` for passing custom command-line arguments to the browser process.

These browser settings are applied when initializing the browser instance in the `BrowserUseTool` class, where configuration values from the global config are mapped to the browser configuration object.

**Section sources**
- [config.py](file://app/config.py#L67-L88)
- [browser_use_tool.py](file://app/tool/browser_use_tool.py#L145-L174)
- [config.example-daytona.toml](file://config/config.example-daytona.toml#L77-L88)

## Search Engine Configuration

Search engine configuration allows for targeted web searches with language and country-specific results. The search settings are defined in the `SearchSettings` class and provide comprehensive control over search behavior.

The `engine` setting specifies the primary search engine to use, with "Google" as the default. Alternative options include "Baidu", "DuckDuckGo", and "Bing". This allows users to select the most appropriate search engine based on their needs, regional preferences, or content availability.

When the primary search engine fails, the system automatically falls back to alternative engines specified in the `fallback_engines` list. By default, this includes ["DuckDuckGo", "Baidu", "Bing"] in that order, providing redundancy and increasing the likelihood of successful search results.

Language targeting is controlled by the `lang` parameter, which defaults to "en" (English). Other supported values include "zh" (Chinese), "fr" (French), and others, allowing searches to be conducted in specific languages and return regionally appropriate results.

Country targeting is managed by the `country` parameter, defaulting to "us" (United States). This influences search results to be more relevant to specific geographical regions, such as "cn" (China) or "uk" (United Kingdom), affecting localized content, pricing, and availability.

The `retry_delay` setting (default: 60 seconds) determines how long to wait before retrying all search engines when they fail due to rate limits or temporary issues. The `max_retries` parameter (default: 3) limits the number of retry attempts, preventing infinite loops during prolonged service outages.

These search settings are utilized by the `WebSearch` tool, which implements a robust search strategy that respects the configured preferences while maintaining reliability through fallback mechanisms and retry logic.

**Section sources**
- [config.py](file://app/config.py#L38-L65)
- [web_search.py](file://app/tool/web_search.py#L200-L269)
- [config.example-daytona.toml](file://config/config.example-daytona.toml#L71-L95)

## Runflow Settings for Multi-Agent Workflows

Runflow settings enable the configuration of multi-agent workflows that coordinate different specialized agents to accomplish complex tasks. The primary setting is `use_data_analysis_agent`, a boolean flag that controls whether the data analysis agent is included in the workflow.

When `use_data_analysis_agent` is set to `true`, the system initializes and incorporates the `DataAnalysis` agent into the planning flow alongside the primary agent. This enables the workflow to handle data-intensive tasks such as data processing, analysis, and visualization using specialized tools.

The multi-agent workflow is orchestrated by the `PlanningFlow` class, which manages the execution of tasks across multiple agents. The flow determines which agent is best suited for each step based on the task requirements and agent capabilities.

The configuration system integrates with the `FlowFactory` to create appropriate flow instances based on the runflow settings. When the data analysis agent is enabled, it is added to the agents dictionary before creating the flow instance.

This multi-agent approach allows for specialization, where different agents can focus on their areas of expertise. For example, the primary agent might handle general planning and coordination, while the data analysis agent handles specific data processing tasks using its specialized toolset.

The runflow configuration enables complex deployment scenarios where different combinations of agents can be activated based on the specific requirements of the task at hand, providing flexibility and extensibility to the system.

**Section sources**
- [config.py](file://app/config.py#L89-L91)
- [run_flow.py](file://run_flow.py#L30-L35)
- [config.example-daytona.toml](file://config/config.example-daytona.toml#L111-L113)

## Configuration Merging and Source Hierarchy

The configuration system implements a sophisticated merging strategy that combines multiple configuration sources into a cohesive settings object. The `Config` class follows a singleton pattern to ensure consistent configuration access throughout the application.

Configuration sources are loaded in a specific hierarchy, with the system first looking for a `config.toml` file in the config directory. If this file doesn't exist, it falls back to `config.example.toml`. This allows users to maintain their custom configurations while preserving example templates.

The configuration merging process begins by loading the base LLM settings from the `[llm]` section, which provides default values for model, API endpoint, and other parameters. These defaults are then potentially overridden by specific LLM configurations defined in the file.

For each configuration domain (browser, search, sandbox, etc.), the system checks if the corresponding section exists in the configuration file. If present, it creates a settings object with the provided values; otherwise, it uses default values defined in the class definitions.

Browser configuration demonstrates a multi-layered merging approach, where proxy settings are handled as a nested object within the browser configuration. The system extracts proxy settings if a server is specified, then incorporates them into the main browser settings object.

The Daytona settings are particularly important as they are used to initialize the Daytona client configuration, which is then used to create and manage cloud sandboxes. These settings are passed directly to the `DaytonaConfig` constructor.

The final configuration object is stored in the singleton instance and made accessible through properties that provide type-safe access to different configuration domains, ensuring consistent and reliable configuration management throughout the application.

**Section sources**
- [config.py](file://app/config.py#L196-L368)
- [config.example-daytona.toml](file://config/config.example-daytona.toml#L0-L114)

## Security Considerations and Best Practices

Security considerations for configuration settings are critical, especially when dealing with sensitive information such as API keys and authentication credentials. The configuration system implements several best practices to ensure secure handling of sensitive data.

Sensitive settings like `daytona_api_key` should never be stored directly in configuration files that might be committed to version control. Instead, these values should be managed through environment variables or secure secret management systems.

The configuration system supports environment variable overrides, allowing sensitive values to be injected at runtime rather than being hardcoded in configuration files. This follows the principle of least privilege and reduces the risk of accidental exposure.

For the `VNC_password` setting, while a default value is provided for convenience, it should be changed in production environments to prevent unauthorized access to sandbox environments. Strong, randomly generated passwords are recommended.

API keys and other authentication tokens should be rotated regularly and have their access scoped to the minimum required permissions. The Daytona API key, for example, should only have permissions necessary for managing sandboxes, not broader account access.

Configuration files containing sensitive information should be protected with appropriate file system permissions, restricting access to only authorized users and processes.

When deploying in multi-user environments, consider implementing configuration profiles that separate sensitive settings from general application settings, allowing for safer sharing of configuration templates.

Environment variables provide a secure alternative to configuration files for sensitive settings. They can be set at deployment time and are less likely to be accidentally exposed through code repositories or logs.

Regular security audits of configuration files and settings are recommended to ensure compliance with security policies and to identify any potential exposure of sensitive information.

**Section sources**
- [config.py](file://app/config.py#L128-L137)
- [config.example-daytona.toml](file://config/config.example-daytona.toml#L97-L113)
- [sandbox.py](file://app/daytona/sandbox.py#L75-L105)
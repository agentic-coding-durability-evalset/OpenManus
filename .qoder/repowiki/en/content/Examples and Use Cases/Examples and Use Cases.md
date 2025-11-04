# Examples and Use Cases

<cite>
**Referenced Files in This Document**   
- [japan_travel_guide_instructions.txt](file://examples/use_case/japan-travel-plan/japan_travel_guide_instructions.txt)
- [browser.py](file://app/agent/browser.py)
- [browser_use_tool.py](file://app/tool/browser_use_tool.py)
- [data_analysis.py](file://app/agent/data_analysis.py)
- [data_visualization.py](file://app/tool/chart_visualization/data_visualization.py)
- [python_execute.py](file://app/tool/python_execute.py)
- [file_operators.py](file://app/tool/file_operators.py)
- [config.example.toml](file://config/config.example.toml)
- [chart_prepare.py](file://app/tool/chart_visualization/chart_prepare.py)
- [sandbox_agent.py](file://app/agent/sandbox_agent.py)
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py)
- [sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py)
- [sb_shell_tool.py](file://app/tool/sandbox/sb_shell_tool.py)
- [sb_vision_tool.py](file://app/tool/sandbox/sb_vision_tool.py)
- [web_search.py](file://app/tool/web_search.py)
- [google_search.py](file://app/tool/search/google_search.py)
</cite>

## Table of Contents
1. [Japan Travel Plan Example](#japan-travel-plan-example)
2. [Benchmark Examples](#benchmark-examples)
3. [Data Analysis with Visualization](#data-analysis-with-visualization)
4. [Code Generation and Debugging](#code-generation-and-debugging)
5. [Web Automation for Form Filling](#web-automation-for-form-filling)
6. [Research Tasks with Web Search](#research-tasks-with-web-search)
7. [Agent Customization](#agent-customization)
8. [Common Patterns and Templates](#common-patterns-and-templates)

## Japan Travel Plan Example

The Japan travel plan example demonstrates how OpenManus uses browser automation to research travel information and create a comprehensive guide. The agent follows the instructions specified in `japan_travel_guide_instructions.txt` to generate three versions of a travel handbook: detailed digital, print-friendly, and mobile-optimized.

The workflow begins with the BrowserAgent, which utilizes the BrowserUseTool to control a browser session. The agent navigates to relevant travel websites, extracts information about destinations, accommodations, transportation, and activities, then organizes this data into structured content. Using the file operations tools, it creates three HTML files with different formatting for various use cases.

The detailed digital version contains complete information including itineraries, hotel recommendations, and budget breakdowns. The print-friendly version condenses essential information with optimized formatting for physical printing. The mobile-optimized version features a touch-friendly interface with collapsible sections and dark mode support for on-the-go reference.

Throughout the process, the agent maintains state across browser sessions, follows links to gather comprehensive information, and uses the SandboxFilesTool to create and modify the handbook files. The final output provides travelers with multiple formats of the same core information, optimized for different situations during their trip.

**Section sources**
- [japan_travel_guide_instructions.txt](file://examples/use_case/japan-travel-plan/japan_travel_guide_instructions.txt)
- [browser.py](file://app/agent/browser.py)
- [browser_use_tool.py](file://app/tool/browser_use_tool.py)
- [sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py)

## Benchmark Examples

Benchmark examples in OpenManus provide a framework for measuring and evaluating agent performance across various tasks. These benchmarks assess capabilities in areas such as web navigation accuracy, information extraction precision, task completion time, and error handling.

The benchmark suite includes tests for browser automation, where agents are evaluated on their ability to navigate complex websites, fill forms correctly, and extract specific information from web pages. Data analysis benchmarks measure the accuracy of statistical calculations, visualization quality, and report comprehensiveness. Web search benchmarks evaluate the relevance of search results, content extraction quality, and the ability to synthesize information from multiple sources.

Performance metrics are collected through automated testing frameworks that track success rates, execution time, resource usage, and output quality. These benchmarks help identify areas for improvement in agent capabilities and provide quantitative measures for comparing different configurations or model versions.

**Section sources**
- [examples/benchmarks](file://examples/benchmarks)
- [browser.py](file://app/agent/browser.py)
- [web_search.py](file://app/tool/web_search.py)
- [data_analysis.py](file://app/agent/data_analysis.py)

## Data Analysis with Visualization

OpenManus provides robust capabilities for data analysis with visualization through its DataAnalysis agent and associated tools. This functionality enables users to process datasets, generate insights, and create visual representations of data patterns.

The data analysis workflow begins with the DataAnalysis agent, which combines multiple tools to process data. The VisualizationPrepare tool executes Python code to clean and transform raw data into structured formats suitable for visualization. This tool generates metadata that describes the visualizations to be created, including chart titles and data file paths.

The DataVisualization tool then takes this metadata and generates interactive charts in HTML format or static images in PNG format. It supports various chart types and can add analytical insights to visualizations. The agent can create multiple visualizations from a single dataset, each highlighting different aspects of the data.

For example, when analyzing travel data, the agent can generate charts showing destination popularity, price trends over time, or demographic breakdowns of travelers. The resulting visualizations are saved in the workspace and can be incorporated into reports or presentations.

**Section sources**
- [data_analysis.py](file://app/agent/data_analysis.py)
- [data_visualization.py](file://app/tool/chart_visualization/data_visualization.py)
- [chart_prepare.py](file://app/tool/chart_visualization/chart_prepare.py)
- [python_execute.py](file://app/tool/python_execute.py)

## Code Generation and Debugging

OpenManus supports code generation and debugging through its sandbox environment and code execution tools. The SandboxManus agent provides a secure environment for writing, testing, and debugging code without affecting the host system.

The agent uses the SandboxShellTool to execute commands in isolated environments, allowing for safe code compilation and testing. Developers can write code using the SandboxFilesTool to create and modify source files, then use the shell tool to run tests and check outputs. The PythonExecute tool enables execution of Python code with timeout and safety restrictions, preventing infinite loops or system resource exhaustion.

For debugging, the agent can analyze error messages, suggest fixes, and iteratively improve code. It can read existing code files, identify potential issues, and propose modifications. The sandbox environment maintains state between commands, allowing for complex development workflows that involve multiple files and dependencies.

This capability is particularly useful for generating code snippets, creating complete applications, or troubleshooting existing codebases. The agent can follow specifications to generate code, then test and refine it based on execution results.

**Section sources**
- [sandbox_agent.py](file://app/agent/sandbox_agent.py)
- [sb_shell_tool.py](file://app/tool/sandbox/sb_shell_tool.py)
- [sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py)
- [python_execute.py](file://app/tool/python_execute.py)

## Web Automation for Form Filling

Web automation for form filling is a core capability of OpenManus, enabled by the BrowserUseTool and its integration with browser automation libraries. This functionality allows agents to interact with web forms, fill in required fields, and submit information automatically.

The browser automation system can identify form elements on web pages, including text inputs, dropdown menus, checkboxes, and buttons. Using element indices provided in the browser state, the agent can target specific form fields for interaction. The input_text action allows the agent to enter text into form fields, while select_dropdown_option handles selection from dropdown menus.

For complex forms with conditional logic or multi-step processes, the agent can navigate through different pages, wait for dynamic content to load, and adapt its actions based on the current page state. It can handle forms that require file uploads, CAPTCHA challenges (when integrated with appropriate services), and forms with JavaScript validation.

This capability is useful for automating repetitive tasks such as booking reservations, submitting applications, or updating account information across multiple websites. The agent maintains session state, allowing it to complete multi-page forms and handle authentication processes.

**Section sources**
- [browser_use_tool.py](file://app/tool/browser_use_tool.py)
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py)
- [browser.py](file://app/agent/browser.py)

## Research Tasks with Web Search

OpenManus excels at research tasks through its integrated web search capabilities. The WebSearch tool provides access to multiple search engines, including Google, Baidu, Bing, and DuckDuckGo, allowing agents to gather information from diverse sources.

The research workflow begins with the agent formulating search queries based on the research objective. It uses the web_search tool to retrieve results, which are then analyzed to identify relevant information. The agent can follow links from search results to gather detailed content from specific web pages.

For comprehensive research, the agent can perform multiple searches with different queries, synthesize information from various sources, and organize findings into structured reports. It can extract key facts, identify conflicting information, and provide citations for sources.

The search system includes fallback mechanisms that automatically try alternative search engines if the primary engine fails or returns insufficient results. This ensures reliable information retrieval even when individual search services are temporarily unavailable.

**Section sources**
- [web_search.py](file://app/tool/web_search.py)
- [google_search.py](file://app/tool/search/google_search.py)
- [browser_use_tool.py](file://app/tool/browser_use_tool.py)

## Agent Customization

OpenManus allows extensive customization of agents for specific tasks through configuration files and tool composition. Users can modify agent behavior by adjusting parameters in configuration files such as config.example.toml, which controls LLM settings, browser options, and search engine preferences.

Agents can be customized by modifying their available tools collection. For example, a data analysis agent can be enhanced with additional visualization tools, while a web automation agent might include specialized form-filling capabilities. The ToolCollection system allows for flexible composition of tools based on task requirements.

Customization extends to prompt engineering, where system prompts and next-step prompts can be modified to guide agent behavior. The prompt templates in the app/prompt directory define how agents approach different tasks and can be adapted for specific domains or use cases.

Users can also create specialized agents by extending existing agent classes and overriding specific methods. This object-oriented approach allows for inheritance of core functionality while adding task-specific capabilities.

**Section sources**
- [config.example.toml](file://config/config.example.toml)
- [browser.py](file://app/agent/browser.py)
- [data_analysis.py](file://app/agent/data_analysis.py)
- [sandbox_agent.py](file://app/agent/sandbox_agent.py)

## Common Patterns and Templates

Successful implementations of OpenManus follow common patterns that optimize agent performance and reliability. These patterns include proper error handling, state management, and task decomposition.

One common pattern is the use of sandbox environments for potentially risky operations, ensuring system security while allowing extensive functionality. Another pattern involves breaking complex tasks into smaller steps, with each step verified before proceeding to the next.

Templates for agent configuration provide starting points for different use cases. The data analysis template includes tools for Python execution and visualization, while the web automation template focuses on browser control and form interaction. These templates can be adapted by modifying tool collections, adjusting LLM parameters, or changing prompt templates.

Best practices include setting appropriate timeouts for operations, implementing retry mechanisms for unreliable services, and maintaining comprehensive logging for debugging and auditing. These patterns and templates help users quickly implement effective solutions while avoiding common pitfalls.

**Section sources**
- [config.example.toml](file://config/config.example.toml)
- [app/agent](file://app/agent)
- [app/tool](file://app/tool)
- [app/prompt](file://app/prompt)
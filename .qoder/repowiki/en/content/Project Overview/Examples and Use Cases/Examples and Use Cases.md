# Examples and Use Cases

<cite>
**Referenced Files in This Document**   
- [run_flow.py](file://run_flow.py)
- [app/agent/data_analysis.py](file://app/agent/data_analysis.py)
- [app/tool/chart_visualization/python_execute.py](file://app/tool/chart_visualization/python_execute.py)
- [app/tool/chart_visualization/chart_prepare.py](file://app/tool/chart_visualization/chart_prepare.py)
- [app/tool/chart_visualization/data_visualization.py](file://app/tool/chart_visualization/data_visualization.py)
- [app/agent/browser.py](file://app/agent/browser.py)
- [app/tool/browser_use_tool.py](file://app/tool/browser_use_tool.py)
- [app/agent/sandbox_agent.py](file://app/agent/sandbox_agent.py)
- [app/flow/planning.py](file://app/flow/planning.py)
- [app/flow/flow_factory.py](file://app/flow/flow_factory.py)
- [examples/use_case/japan-travel-plan/japan_travel_handbook.html](file://examples/use_case/japan-travel-plan/japan_travel_handbook.html)
- [examples/use_case/japan-travel-plan/japan_travel_handbook_mobile.html](file://examples/use_case/japan-travel-plan/japan_travel_handbook_mobile.html)
- [examples/use_case/japan-travel-plan/japan_travel_handbook_print.html](file://examples/use_case/japan-travel-plan/japan_travel_handbook_print.html)
- [examples/use_case/japan-travel-plan/japan_travel_guide_instructions.txt](file://examples/use_case/japan-travel-plan/japan_travel_guide_instructions.txt)
</cite>

## Table of Contents
1. [Japan Travel Planning with HTML Output](#japan-travel-planning-with-html-output)
2. [Data Analysis Workflows with Chart Visualization](#data-analysis-workflows-with-chart-visualization)
3. [Web Automation with Browser Navigation](#web-automation-with-browser-navigation)
4. [Code Generation and Execution in Sandboxed Environments](#code-generation-and-execution-in-sandboxed-environments)
5. [Multi-Agent Coordination with Planning Flow](#multi-agent-coordination-with-planning-flow)
6. [Best Practices for Complex Task Structuring](#best-practices-for-complex-task-structuring)
7. [Common Pitfalls and Optimization Opportunities](#common-pitfalls-and-optimization-opportunities)

## Japan Travel Planning with HTML Output

The Japan travel planning example demonstrates OpenManus's capability to generate comprehensive travel itineraries with multiple output formats. When presented with a detailed travel request, the system creates a complete travel handbook in three different formats: a detailed digital version, a print-friendly version, and a mobile-optimized version.

The system processes the travel request by first analyzing the user's requirements including budget constraints ($2500-5000), travel dates (April 15-23), interests (historical sites, cultural experiences), and special occasions (a marriage proposal). It then generates a 7-day itinerary that includes daily activities, transportation details, accommodation recommendations, and budget breakdowns.

For the HTML output generation, OpenManus creates three distinct HTML files with tailored styling and functionality:
- The detailed digital version (japan_travel_handbook.html) contains comprehensive information with rich styling for desktop viewing
- The print-friendly version (japan_travel_handbook_print.html) features optimized formatting for physical printing with page breaks and printer-friendly styles
- The mobile-optimized version (japan_travel_handbook_mobile.html) includes touch-friendly interfaces, collapsible sections, and dark mode support for on-the-go reference

The generated handbook includes essential travel information such as emergency contacts, transportation details, budget tracking, cultural tips, and specific recommendations for the marriage proposal including location suggestions (Maruyama Park), timing recommendations, and backup plans for inclement weather.

**Section sources**
- [examples/use_case/japan-travel-plan/japan_travel_handbook.html](file://examples/use_case/japan-travel-plan/japan_travel_handbook.html)
- [examples/use_case/japan-travel-plan/japan_travel_handbook_mobile.html](file://examples/use_case/japan-travel-plan/japan_travel_handbook_mobile.html)
- [examples/use_case/japan-travel-plan/japan_travel_handbook_print.html](file://examples/use_case/japan-travel-plan/japan_travel_handbook_print.html)
- [examples/use_case/japan-travel-plan/japan_travel_guide_instructions.txt](file://examples/use_case/japan-travel-plan/japan_travel_guide_instructions.txt)

## Data Analysis Workflows with Chart Visualization

OpenManus provides robust data analysis capabilities through its DataAnalysis agent, which combines Python execution with advanced chart visualization tools. This workflow enables users to transform raw data into meaningful visual representations and insights.

The data analysis process follows a three-step approach using specialized tools:
1. **Data Preparation**: The VisualizationPrepare tool processes raw data, cleans datasets, and generates metadata for visualization
2. **Chart Generation**: The DataVisualization tool creates interactive charts in HTML format or static images in PNG format
3. **Insight Enhancement**: The same tool can add analytical insights to existing charts based on the data patterns

The system supports various chart types including bar charts, pie charts, line graphs, radar charts, and Sankey diagrams. For example, when analyzing sales data across different regions, the agent can generate comparative bar charts showing product performance. When examining market share data, it creates pie charts with percentage breakdowns. Time-series data is visualized as line graphs showing trends over time.

The workflow is designed to handle both simple and complex data analysis tasks. For multi-faceted datasets, the agent can generate multiple charts from a single dataset, each focusing on different aspects of the data. The system also supports comparative analysis across different datasets and can identify correlations and patterns that might not be immediately apparent.

**Section sources**
- [app/agent/data_analysis.py](file://app/agent/data_analysis.py)
- [app/tool/chart_visualization/python_execute.py](file://app/tool/chart_visualization/python_execute.py)
- [app/tool/chart_visualization/chart_prepare.py](file://app/tool/chart_visualization/chart_prepare.py)
- [app/tool/chart_visualization/data_visualization.py](file://app/tool/chart_visualization/data_visualization.py)

## Web Automation with Browser Navigation

The browser automation capabilities in OpenManus enable sophisticated web interaction through a dedicated BrowserAgent. This agent can navigate websites, extract content, fill forms, and interact with web elements to accomplish various tasks.

The browser automation system supports a comprehensive set of actions:
- **Navigation**: Go to specific URLs, go back, refresh pages, and perform web searches
- **Interaction**: Click elements, input text, select dropdown options, and send keyboard commands
- **Content Extraction**: Extract and analyze content from web pages based on specific goals
- **Tab Management**: Switch between tabs, open new tabs, and close tabs
- **Scrolling**: Scroll up/down by pixel amount or scroll to specific text

The agent maintains state across calls, keeping the browser session alive until explicitly closed. It uses element indices to reference interactive elements on the page, allowing for precise control over web interactions. When performing web searches, the system can automatically navigate to the most relevant search result and extract the required information.

This capability is particularly useful for tasks such as research, data collection from websites, monitoring web content, and automating repetitive web-based workflows. The agent can follow complex navigation paths, handle dynamic content loading, and adapt to different website structures.

**Section sources**
- [app/agent/browser.py](file://app/agent/browser.py)
- [app/tool/browser_use_tool.py](file://app/tool/browser_use_tool.py)

## Code Generation and Execution in Sandboxed Environments

OpenManus provides secure code execution capabilities through its sandboxed environment system. This allows for the safe generation and execution of code without compromising system security.

The sandbox environment is implemented through the SandboxManus agent, which creates isolated execution environments for running code. These sandboxes provide resource-limited, isolated containers where code can be executed safely. The system includes several key components:
- **SandboxShellTool**: Executes shell commands within the sandbox environment
- **SandboxFilesTool**: Manages file operations within the sandbox
- **SandboxBrowserTool**: Enables browser automation within the sandbox
- **SandboxVisionTool**: Provides vision capabilities within the sandbox

The code execution workflow follows these steps:
1. Code generation based on the user's requirements
2. Validation and syntax checking
3. Execution within the isolated sandbox environment
4. Capture of output and error messages
5. Return of results to the user

This approach ensures that potentially harmful code cannot affect the host system while still providing the full functionality needed for data analysis, automation, and other computational tasks. The sandbox environment can be configured with specific resource limits and access controls based on the requirements of the task.

**Section sources**
- [app/agent/sandbox_agent.py](file://app/agent/sandbox_agent.py)
- [app/sandbox/core/sandbox.py](file://app/sandbox/core/sandbox.py)
- [app/tool/sandbox/sb_shell_tool.py](file://app/tool/sandbox/sb_shell_tool.py)
- [app/tool/sandbox/sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py)

## Multi-Agent Coordination with Planning Flow

OpenManus supports complex task execution through its multi-agent coordination system, which uses a planning flow architecture to coordinate multiple specialized agents. This approach enables the system to handle sophisticated tasks that require different capabilities.

The planning flow system is implemented in the PlanningFlow class, which manages the execution of tasks using multiple agents. The workflow begins with the creation of an initial plan based on the user's request. This plan is then executed step by step, with each step potentially handled by a different agent based on its capabilities.

The system supports agent specialization, where different agents are responsible for different types of tasks:
- **Manus Agent**: General-purpose tasks and coordination
- **DataAnalysis Agent**: Data processing and visualization tasks
- **Browser Agent**: Web navigation and content extraction
- **Sandbox Agent**: Code execution in isolated environments

The FlowFactory creates the appropriate flow type (currently PlanningFlow) and initializes it with the required agents. The planning system tracks the progress of each step, marks completed steps, and provides status updates throughout the execution process. This coordinated approach allows for complex workflows where multiple agents work together to accomplish a common goal, with each agent contributing its specialized capabilities to the overall task.

**Section sources**
- [run_flow.py](file://run_flow.py)
- [app/flow/planning.py](file://app/flow/planning.py)
- [app/flow/flow_factory.py](file://app/flow/flow_factory.py)
- [app/agent/sandbox_agent.py](file://app/agent/sandbox_agent.py)
- [app/agent/data_analysis.py](file://app/agent/data_analysis.py)

## Best Practices for Complex Task Structuring

When structuring complex tasks in OpenManus, several best practices can enhance effectiveness and efficiency:

1. **Clear Task Decomposition**: Break down complex requests into smaller, manageable components. For example, a travel planning request should be decomposed into itinerary creation, budget planning, accommodation research, and activity scheduling.

2. **Appropriate Agent Selection**: Choose the right agent or combination of agents for the task. Use the DataAnalysis agent for data-intensive tasks, the Browser agent for web research, and the Sandbox agent for code execution.

3. **Structured Input Prompts**: Provide clear, detailed prompts that specify the desired output format, content requirements, and any constraints. Well-structured prompts lead to more accurate and useful results.

4. **Progressive Refinement**: Start with broad requests and progressively refine the output through follow-up queries. This iterative approach often yields better results than attempting to get everything perfect in a single request.

5. **Output Format Specification**: Clearly specify the desired output format, whether it's HTML, JSON, plain text, or a specific document structure. This ensures the output meets your integration or presentation needs.

6. **Error Handling Planning**: Anticipate potential issues and include contingency plans in your requests. For example, when planning travel, include backup options for outdoor activities in case of bad weather.

7. **Resource Awareness**: Be mindful of computational resources and execution time, especially for complex data analysis or extensive web scraping tasks.

## Common Pitfalls and Optimization Opportunities

Several common pitfalls can be avoided when using OpenManus, and various optimization opportunities exist:

**Common Pitfalls:**
- **Overly Broad Requests**: Vague or overly broad requests can lead to unfocused or incomplete results. Always provide specific details and constraints.
- **Ignoring Rate Limits**: Web automation tasks should respect website rate limits and terms of service to avoid being blocked.
- **Insufficient Error Handling**: Failing to plan for potential errors or exceptions can cause workflows to fail unexpectedly.
- **Resource Exhaustion**: Complex data analysis or long-running processes can consume significant resources. Monitor resource usage and set appropriate limits.
- **Security Oversights**: When executing code or accessing external systems, ensure proper security measures are in place.

**Optimization Opportunities:**
- **Parallel Processing**: Where possible, break tasks into independent components that can be processed in parallel.
- **Caching Results**: Store and reuse results from expensive operations to avoid redundant processing.
- **Incremental Processing**: For large datasets, process data in chunks rather than loading everything into memory at once.
- **Agent Specialization**: Leverage specialized agents for specific tasks rather than using general-purpose agents for everything.
- **Workflow Automation**: Identify repetitive patterns and create templates or scripts to automate them.
- **Performance Monitoring**: Track execution times and resource usage to identify bottlenecks and areas for improvement.

By understanding these pitfalls and optimization opportunities, users can maximize the effectiveness of OpenManus for their various use cases.
# Local Tools

<cite>
**Referenced Files in This Document**   
- [python_execute.py](file://app/tool/python_execute.py)
- [str_replace_editor.py](file://app/tool/str_replace_editor.py)
- [file_operators.py](file://app/tool/file_operators.py)
- [ask_human.py](file://app/tool/ask_human.py)
- [web_search.py](file://app/tool/web_search.py)
- [browser_use_tool.py](file://app/tool/browser_use_tool.py)
- [data_visualization.py](file://app/tool/chart_visualization/data_visualization.py)
- [sb_shell_tool.py](file://app/tool/sandbox/sb_shell_tool.py)
- [sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py)
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py)
- [sb_vision_tool.py](file://app/tool/sandbox/sb_vision_tool.py)
</cite>

## Table of Contents
1. [Python Execution](#python-execution)
2. [Browser Automation](#browser-automation)
3. [File Operations](#file-operations)
4. [Web Search](#web-search)
5. [Interactive Input](#interactive-input)
6. [Data Visualization](#data-visualization)
7. [Sandbox Tools](#sandbox-tools)

## Python Execution

The Python execution tools enable code execution within the agent's environment with safety measures and output capturing. Two variants are available: standard execution and sandboxed execution.

The `python_execute` tool allows executing Python code strings with a 5-second timeout by default. Only print outputs are visible, as function return values are not captured. The tool uses multiprocessing to isolate execution and prevent the agent from being blocked by long-running or infinite loops. Code execution occurs in a restricted environment with safety restrictions to prevent malicious operations.

For data analysis tasks, the `NormalPythonExecute` variant extends the base functionality with additional parameters for specifying code type (process, report, or others). This tool is specifically designed for in-depth data analysis and report generation, with requirements to save processed files and reports in the workspace directory.

**Section sources**
- [python_execute.py](file://app/tool/python_execute.py#L1-L75)
- [python_execute.py](file://app/tool/chart_visualization/python_execute.py#L1-L36)

## Browser Automation

Browser automation capabilities are provided through Playwright integration, enabling navigation, interaction, and content extraction from web pages. The `browser_use` tool supports various actions including navigating to URLs, clicking elements by index, inputting text, scrolling, and extracting content.

Key features include:
- Navigation: `go_to_url`, `go_back`, `web_search`
- Element interaction: `click_element`, `input_text`, `send_keys`
- Scrolling: `scroll_down`, `scroll_up`, `scroll_to_text`
- Tab management: `switch_tab`, `open_tab`, `close_tab`
- Content extraction: `extract_content`, `get_dropdown_options`

The tool maintains browser state across calls and provides element indexing to identify interactive components on the page. When executing actions, the tool returns detailed feedback including URLs, titles, and visual representations of the current page state.

**Section sources**
- [browser_use_tool.py](file://app/tool/browser_use_tool.py#L38-L75)

## File Operations

File operations are handled through two primary tools: `str_replace_editor` and `file_operators`. These tools provide comprehensive capabilities for viewing, creating, editing, and managing files in both local and sandboxed environments.

The `str_replace_editor` tool supports the following commands:
- `view`: Display file content with line numbers or list directory contents
- `create`: Create new files with specified content
- `str_replace`: Replace a unique string in a file with new content
- `insert`: Insert text at a specific line in a file
- `undo_edit`: Revert the last edit made to a file

Files are edited by specifying exact string matches to ensure precision. The tool maintains edit history to support undo operations and provides snippet previews of changes. Long outputs are truncated with appropriate notices to manage context length.

**Section sources**
- [str_replace_editor.py](file://app/tool/str_replace_editor.py#L1-L432)
- [file_operators.py](file://app/tool/file_operators.py#L1-L158)

## Web Search

Multi-engine web search functionality is provided through integration with Google, Bing, DuckDuckGo, and Baidu search engines. The `web_search` tool allows real-time information retrieval about any topic with automatic fallback to alternative engines if the primary search fails.

Search parameters include:
- `query`: The search query to submit (required)
- `num_results`: Number of results to return (default: 5)
- `lang`: Language code for results (default: en)
- `country`: Country code for results (default: us)
- `fetch_content`: Whether to fetch full content from result pages (default: false)

Each search engine implementation follows a consistent interface, returning results formatted with titles, URLs, and descriptions. The tool automatically parses search result pages and extracts relevant information, handling different HTML structures for each search engine.

**Section sources**
- [web_search.py](file://app/tool/web_search.py#L158-L198)
- [google_search.py](file://app/tool/search/google_search.py#L1-L33)
- [bing_search.py](file://app/tool/search/bing_search.py#L1-L144)
- [duckduckgo_search.py](file://app/tool/search/duckduckgo_search.py#L1-L57)
- [baidu_search.py](file://app/tool/search/baidu_search.py#L1-L54)

## Interactive Input

The `ask_human` tool enables interactive communication with users, allowing the agent to request clarification, confirmation, or additional information during task execution. This tool is essential for scenarios requiring human judgment, subjective evaluation, or access to information not available through automated means.

When invoked, the tool presents a question to the user and waits for input before continuing. The interaction follows a simple pattern:
1. Agent formulates a question using the `inquire` parameter
2. Question is presented to the user with clear formatting
3. User provides response through standard input
4. Response is returned to the agent for further processing

This capability bridges the gap between automated processing and human oversight, ensuring that the agent can handle ambiguous situations or tasks requiring human expertise.

**Section sources**
- [ask_human.py](file://app/tool/ask_human.py#L1-L21)

## Data Visualization

The chart visualization suite enables data analysis and visualization using VMind/VChart technology. This system consists of multiple components that work together to transform raw data into meaningful visual representations.

The `DataVisualization` tool processes JSON configuration files generated by the visualization preparation step to create charts in either PNG or HTML format. Key parameters include:
- `json_path`: Path to the JSON configuration file
- `output_type`: Rendering format (png for static images, html for interactive charts)
- `tool_type`: Operation type (visualization or insight)
- `language`: Interface language (en for English, zh for Chinese)

The visualization pipeline involves invoking Node.js subprocesses that use VMind to generate chart specifications and VChart for rendering. Charts are saved in the workspace directory with appropriate file extensions, and insights are provided in Markdown format when requested.

**Section sources**
- [data_visualization.py](file://app/tool/chart_visualization/data_visualization.py#L1-L263)

## Sandbox Tools

Sandbox-specific tools provide isolated execution environments for shell, file, browser, and vision operations. These tools enhance security by containing potentially risky operations within controlled environments while maintaining functionality.

### Shell Operations
The `sandbox_shell` tool enables command execution in a tmux session within the sandbox environment. It supports:
- `execute_command`: Run shell commands with optional blocking behavior
- `check_command_output`: Retrieve output from running commands
- `terminate_command`: Stop executing commands
- `list_commands`: List active tmux sessions

Commands are executed in the workspace directory and can be run non-blocking for long-running processes like servers or build operations.

### File Operations
The `sandbox_files` tool provides file management capabilities within the sandbox:
- `create_file`: Create new files with specified permissions
- `str_replace`: Replace text in existing files
- `full_file_rewrite`: Completely overwrite file contents
- `delete_file`: Remove files from the workspace

All operations are relative to the /workspace directory for security.

### Browser Operations
The `sandbox_browser` tool offers browser automation in a sandboxed environment with actions including navigation, element interaction, scrolling, and tab management. It maintains browser state across calls and provides screenshot capabilities.

### Vision Operations
The `sandbox_vision` tool allows reading image files from the sandbox, compressing them, and converting to base64 format for use in subsequent processing. It supports JPG, PNG, GIF, and WEBP formats with size limitations and automatic compression to optimize for context usage.

**Section sources**
- [sb_shell_tool.py](file://app/tool/sandbox/sb_shell_tool.py#L1-L419)
- [sb_files_tool.py](file://app/tool/sandbox/sb_files_tool.py#L1-L361)
- [sb_browser_tool.py](file://app/tool/sandbox/sb_browser_tool.py#L1-L450)
- [sb_vision_tool.py](file://app/tool/sandbox/sb_vision_tool.py#L1-L178)
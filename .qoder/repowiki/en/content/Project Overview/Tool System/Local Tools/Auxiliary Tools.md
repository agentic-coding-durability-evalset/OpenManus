# Auxiliary Tools

<cite>
**Referenced Files in This Document**   
- [bash.py](file://app/tool/bash.py)
- [computer_use_tool.py](file://app/tool/computer_use_tool.py)
- [planning.py](file://app/tool/planning.py)
- [str_replace_editor.py](file://app/tool/str_replace_editor.py)
- [terminate.py](file://app/tool/terminate.py)
- [file_operators.py](file://app/tool/file_operators.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Bash Tool](#bash-tool)
3. [Computer Use Tool](#computer-use-tool)
4. [Planning Tool](#planning-tool)
5. [Str Replace Editor](#str-replace-editor)
6. [Terminate Tool](#terminate-tool)
7. [Integration Patterns](#integration-patterns)
8. [Security and Performance Considerations](#security-and-performance-considerations)

## Introduction
OpenManus provides a suite of auxiliary tools designed to enhance agent capabilities through system integration, automation, and task management. These tools enable agents to execute shell commands, automate desktop interactions, decompose complex tasks, perform surgical file modifications, and gracefully terminate sessions. The tools are designed to work within both local and sandboxed environments, providing flexibility for different execution contexts while maintaining security and reliability.

## Bash Tool

The bash tool enables agents to execute shell commands within a terminal environment. It maintains a persistent bash session through the `_BashSession` class, which manages subprocess creation, command execution, and output capture. The tool uses a sentinel value `<<exit>>` to detect command completion and implements timeout protection (120 seconds by default) to prevent hanging processes.

The tool integrates with file operators through the command execution chain, allowing shell commands to interact with the filesystem. When executing commands, it captures both stdout and stderr outputs, providing comprehensive feedback to the agent. For long-running processes, the tool supports background execution with output redirection to files, enabling non-blocking operation.

The bash tool implements interactive capabilities by maintaining an open stdin connection, allowing agents to send additional input to running processes or interrupt them with `ctrl+c` commands. This enables complex workflows involving interactive applications or multi-step command sequences.

**Section sources**
- [bash.py](file://app/tool/bash.py#L0-L158)

## Computer Use Tool

The computer_use_tool provides desktop automation capabilities through integration with AI21 Labs' Computer Use API. It enables mouse control, keyboard input, screenshot capture, and other GUI interactions by sending HTTP requests to an automation service endpoint. The tool maintains state including current mouse position coordinates (mouse_x, mouse_y) and manages API communication through an aiohttp client session.

Key capabilities include:
- **Mouse operations**: Move, click, drag, scroll with configurable button and click count
- **Keyboard input**: Type text, press individual keys, or execute key combinations (hotkeys)
- **Screen capture**: Take screenshots and save them as timestamped PNG files
- **Timing control**: Pause execution for specified durations

The tool is initialized with a sandbox reference that provides the API base URL, enabling communication with the automation service. It uses the `create_with_sandbox` factory method to ensure proper initialization with sandbox context. Each action is implemented as a separate API request to the automation service, with appropriate error handling and response parsing.

**Section sources**
- [computer_use_tool.py](file://app/tool/computer_use_tool.py#L0-L487)

## Planning Tool

The planning tool enables LLM-driven task decomposition through hierarchical plan management. It allows agents to create, update, list, retrieve, and track progress on multi-step plans. The tool maintains an in-memory store of plans with their steps, statuses, and notes, enabling persistent state across interactions.

Key commands include:
- **create**: Initialize a new plan with ID, title, and steps
- **update**: Modify existing plan title or steps
- **list**: Display all available plans with progress indicators
- **get**: Retrieve details of a specific plan
- **set_active**: Designate a plan as the current active plan
- **mark_step**: Update status of individual steps (not_started, in_progress, completed, blocked)
- **delete**: Remove a plan from storage

The tool integrates with the planning flow system, where it works in conjunction with LLM prompting to generate initial plans and track execution progress. The planning flow uses the tool to create plans based on user requests, then iteratively executes steps while updating their status. This enables complex workflow orchestration with proper state tracking and progress monitoring.

**Section sources**
- [planning.py](file://app/tool/planning.py#L0-L363)

## Str Replace Editor

The str_replace_editor provides surgical file modification capabilities without requiring full file rewrites. It supports multiple commands for file manipulation:
- **view**: Display file content with line numbers or directory contents
- **create**: Create new files with specified content
- **str_replace**: Replace exact string matches in files
- **insert**: Insert text at specific line numbers
- **undo_edit**: Revert the last edit to a file

The tool ensures precision in modifications by requiring exact string matches for replacements and validating uniqueness of the target string. For `str_replace` operations, the `old_str` parameter must match exactly one occurrence in the file, with whitespace sensitivity. The tool provides context-aware feedback by showing snippets of modified sections with line numbers.

It maintains edit history through the `_file_history` dictionary, enabling the `undo_edit` functionality. The tool supports both local and sandboxed execution environments through the `_get_operator` method, which selects between `LocalFileOperator` and `SandboxFileOperator` based on configuration.

**Section sources**
- [str_replace_editor.py](file://app/tool/str_replace_editor.py#L0-L432)
- [file_operators.py](file://app/tool/file_operators.py#L0-L158)

## Terminate Tool

The terminate tool provides a mechanism for gracefully ending agent sessions. It accepts a status parameter indicating whether the interaction was successful or failed, and returns a completion message. This tool serves as the final step in agent workflows, signaling that all tasks have been completed or that further progress is impossible.

The tool is designed to be called when either:
1. All requested tasks have been successfully completed
2. The agent cannot proceed further with the current task
3. A terminal state has been reached in the workflow

By providing explicit session termination, the tool enables proper resource cleanup and workflow conclusion. It works in conjunction with other tools to ensure that agents do not continue processing after objectives have been met.

**Section sources**
- [terminate.py](file://app/tool/terminate.py#L0-L25)

## Integration Patterns

The auxiliary tools work together in complex agent workflows requiring multi-step coordination. Key integration patterns include:

**Command Execution Chain**: The bash tool integrates with file operators to enable shell commands that read, write, and manipulate files. This creates a powerful combination where agents can execute system commands while maintaining file state.

**Task Decomposition Workflow**: The planning tool works with LLM prompting to decompose complex tasks into manageable steps. The planning flow system uses the planning tool to create initial plans, then executes steps using appropriate agents while tracking progress through step status updates.

**Surgical Code Modification**: The str_replace_editor enables precise code changes without full file rewrites. This is particularly valuable in development workflows where agents need to modify specific sections of code while preserving overall structure and formatting.

**Multi-Tool Orchestration**: Agents combine multiple tools in sequence, such as using bash to execute commands, str_replace_editor to modify configuration files, and planning to track overall progress. This enables sophisticated automation scenarios that require coordination across different system interfaces.

**Section sources**
- [planning.py](file://app/tool/planning.py#L0-L363)
- [flow/planning.py](file://app/flow/planning.py#L0-L442)
- [prompt/planning.py](file://app/prompt/planning.py#L0-L27)

## Security and Performance Considerations

**Security Considerations for System Command Execution**:
The bash tool implements several security measures to prevent malicious command execution. While the current implementation relies on basic command sanitization, additional safeguards could be implemented, such as restricting access to sensitive directories or implementing command whitelists. The sandbox environment provides an additional security layer by isolating command execution from the host system.

**Performance Implications of External API Dependencies**:
The computer_use_tool introduces performance considerations due to its dependency on external API calls. Each action requires an HTTP request to the automation service, creating network latency. The tool implements proper timeout handling and error recovery, but frequent API calls can impact overall workflow performance. Caching strategies or batch operations could mitigate these effects in high-frequency scenarios.

**Integration Security**:
The str_replace_editor includes safeguards against ambiguous replacements by requiring exact string matches and validating uniqueness. This prevents unintended modifications when multiple instances of a string exist in a file. The tool also maintains edit history, enabling recovery from erroneous changes.

**Resource Management**:
All tools implement proper resource cleanup, with the computer_use_tool explicitly closing HTTP sessions and the bash tool terminating subprocesses. This prevents resource leaks during long-running agent operations.

**Section sources**
- [bash.py](file://app/tool/bash.py#L0-L158)
- [computer_use_tool.py](file://app/tool/computer_use_tool.py#L0-L487)
- [str_replace_editor.py](file://app/tool/str_replace_editor.py#L0-L432)
- [sandbox/core/terminal.py](file://app/sandbox/core/terminal.py#L0-L346)
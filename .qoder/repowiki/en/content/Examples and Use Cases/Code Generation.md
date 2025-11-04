# Code Generation

<cite>
**Referenced Files in This Document**   
- [swe.py](file://app/agent/swe.py)
- [python_execute.py](file://app/tool/python_execute.py)
- [sandbox.py](file://app/sandbox/core/sandbox.py)
- [file_operators.py](file://app/tool/file_operators.py)
- [config.py](file://app/config.py)
- [bash.py](file://app/tool/bash.py)
- [swe.py](file://app/prompt/swe.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [SWE Agent Architecture](#swe-agent-architecture)
3. [Code Execution Workflow](#code-execution-workflow)
4. [Sandbox Environment](#sandbox-environment)
5. [File Operations](#file-operations)
6. [Configuration Settings](#configuration-settings)
7. [Security and Resource Management](#security-and-resource-management)
8. [Practical Example: Sorting Algorithm Implementation](#practical-example-sorting-algorithm-implementation)
9. [Best Practices for Prompt Design](#best-practices-for-prompt-design)
10. [Conclusion](#conclusion)

## Introduction
The SWE (Software Engineer) Agent in OpenManus is an autonomous AI programmer designed to solve coding tasks through natural language interaction and direct computer interaction. This document details how the agent interprets requirements, generates code, executes it in a secure sandbox environment, and iteratively debugs based on test results or error feedback. The agent follows a structured workflow that includes receiving coding tasks, writing code, executing it via python_execute.py, analyzing errors, and refining solutions until completion.

**Section sources**
- [swe.py](file://app/agent/swe.py#L9-L23)

## SWE Agent Architecture
The SWEAgent class inherits from ToolCallAgent and implements the SWEAgent paradigm for executing code and natural conversations. It operates as an autonomous AI programmer that interacts directly with the computer to solve tasks. The agent has a maximum step limit of 20 and utilizes specific tools including Bash, StrReplaceEditor, and Terminate. The system prompt guides the agent's behavior in the command line interface with a special file editor that shows a limited number of lines at a time.

```mermaid
classDiagram
class SWEAgent {
+name : str = "swe"
+description : str
+system_prompt : str
+next_step_prompt : str
+available_tools : ToolCollection
+special_tool_names : List[str]
+max_steps : int = 20
}
class ToolCallAgent {
+name : str
+description : str
+system_prompt : str
+next_step_prompt : str
+available_tools : ToolCollection
+tool_choices : TOOL_CHOICE_TYPE
+special_tool_names : List[str]
+tool_calls : List[ToolCall]
+_current_base64_image : Optional[str]
+max_steps : int = 30
+max_observe : Optional[Union[int, bool]]
+think() bool
+act() str
+execute_tool(command : ToolCall) str
+_handle_special_tool(name : str, result : Any) void
+_should_finish_execution(**kwargs) bool
+_is_special_tool(name : str) bool
+cleanup() void
+run(request : Optional[str]) str
}
SWEAgent --|> ToolCallAgent
```

**Diagram sources **
- [swe.py](file://app/agent/swe.py#L9-L23)
- [toolcall.py](file://app/agent/toolcall.py#L17-L249)

**Section sources**
- [swe.py](file://app/agent/swe.py#L9-L23)
- [toolcall.py](file://app/agent/toolcall.py#L17-L249)

## Code Execution Workflow
The SWE Agent follows a systematic workflow for code generation and debugging. When receiving a coding task, the agent first interprets the natural language requirements and plans its approach. It then generates code using the available tools and executes it in a secure environment. The execution process involves calling the python_execute tool with the generated code, which runs in an isolated sandbox with timeout and safety restrictions. After execution, the agent analyzes the output or error messages and iteratively refines the solution until the task is completed successfully.

```mermaid
sequenceDiagram
participant User as "User"
participant SWEAgent as "SWE Agent"
participant PythonExecute as "PythonExecute Tool"
participant Sandbox as "Docker Sandbox"
User->>SWEAgent : Submit coding task
SWEAgent->>SWEAgent : Interpret requirements
SWEAgent->>SWEAgent : Generate code
SWEAgent->>PythonExecute : Execute code
PythonExecute->>Sandbox : Run code in sandbox
Sandbox-->>PythonExecute : Return execution result
PythonExecute-->>SWEAgent : Provide output/error
SWEAgent->>SWEAgent : Analyze results
alt Success
SWEAgent-->>User : Return successful solution
else Error
SWEAgent->>SWEAgent : Debug and refine code
SWEAgent->>PythonExecute : Execute revised code
PythonExecute->>Sandbox : Run revised code
Sandbox-->>PythonExecute : Return new result
PythonExecute-->>SWEAgent : Provide updated output
SWEAgent->>SWEAgent : Repeat until success
SWEAgent-->>User : Return final solution
end
```

**Diagram sources **
- [swe.py](file://app/agent/swe.py#L9-L23)
- [python_execute.py](file://app/tool/python_execute.py#L8-L74)
- [sandbox.py](file://app/sandbox/core/sandbox.py#L17-L461)

**Section sources**
- [swe.py](file://app/agent/swe.py#L9-L23)
- [python_execute.py](file://app/tool/python_execute.py#L8-L74)

## Sandbox Environment
The DockerSandbox class provides a containerized execution environment with resource limits, file operations, and command execution capabilities. It ensures secure code execution by isolating the process in a Docker container with configurable memory and CPU limits. The sandbox prevents path traversal attempts and restricts network access by default. Each sandbox instance is created with a unique container name and runs with specific resource constraints defined in the SandboxSettings configuration.

```mermaid
classDiagram
class DockerSandbox {
+config : SandboxSettings
+volume_bindings : Dict[str, str]
+client : docker.client
+container : Container
+terminal : AsyncDockerizedTerminal
+__init__(config : Optional[SandboxSettings], volume_bindings : Optional[Dict[str, str]]) void
+create() DockerSandbox
+_prepare_volume_bindings() Dict[str, Dict[str, str]]
+_ensure_host_dir(path : str) str
+run_command(cmd : str, timeout : Optional[int]) str
+read_file(path : str) str
+write_file(path : str, content : str) void
+_safe_resolve_path(path : str) str
+copy_from(src_path : str, dst_path : str) void
+copy_to(src_path : str, dst_path : str) void
+_create_tar_stream(name : str, content : bytes) io.BytesIO
+_read_from_tar(tar_stream) bytes
+cleanup() void
+__aenter__() DockerSandbox
+__aexit__(exc_type, exc_val, exc_tb) void
}
```

**Diagram sources **
- [sandbox.py](file://app/sandbox/core/sandbox.py#L17-L461)

**Section sources**
- [sandbox.py](file://app/sandbox/core/sandbox.py#L17-L461)

## File Operations
The file operations in OpenManus are handled through the FileOperator interface, which provides methods for reading, writing, and checking file existence in both local and sandbox environments. The LocalFileOperator implements these operations for the local filesystem, while the SandboxFileOperator handles operations within the sandbox environment. Both implementations ensure proper error handling and provide consistent interfaces for file manipulation across different execution contexts.

```mermaid
classDiagram
class FileOperator {
<<protocol>>
+read_file(path : PathLike) str
+write_file(path : PathLike, content : str) void
+is_directory(path : PathLike) bool
+exists(path : PathLike) bool
+run_command(cmd : str, timeout : Optional[float]) Tuple[int, str, str]
}
class LocalFileOperator {
+encoding : str = "utf-8"
+read_file(path : PathLike) str
+write_file(path : PathLike, content : str) void
+is_directory(path : PathLike) bool
+exists(path : PathLike) bool
+run_command(cmd : str, timeout : Optional[float]) Tuple[int, str, str]
}
class SandboxFileOperator {
+_ensure_sandbox_initialized() void
+read_file(path : PathLike) str
+write_file(path : PathLike, content : str) void
+is_directory(path : PathLike) bool
+exists(path : PathLike) bool
+run_command(cmd : str, timeout : Optional[float]) Tuple[int, str, str]
}
FileOperator <|.. LocalFileOperator
FileOperator <|.. SandboxFileOperator
```

**Diagram sources **
- [file_operators.py](file://app/tool/file_operators.py#L0-L158)

**Section sources**
- [file_operators.py](file://app/tool/file_operators.py#L0-L158)

## Configuration Settings
The SandboxSettings class defines the configuration for the execution sandbox, including whether to use the sandbox, the base Docker image, working directory, memory and CPU limits, command timeout, and network access permissions. These settings can be customized in the configuration file to meet specific security and performance requirements. The default configuration uses a Python 3.12-slim image with 512MB memory limit, 1.0 CPU limit, 300-second timeout, and disabled network access for enhanced security.

```mermaid
classDiagram
class SandboxSettings {
+use_sandbox : bool = False
+image : str = "python : 3.12-slim"
+work_dir : str = "/workspace"
+memory_limit : str = "512m"
+cpu_limit : float = 1.0
+timeout : int = 300
+network_enabled : bool = False
}
```

**Diagram sources **
- [config.py](file://app/config.py#L93-L104)

**Section sources**
- [config.py](file://app/config.py#L93-L104)

## Security and Resource Management
The SWE Agent implements multiple security and resource management mechanisms to ensure safe code execution. The PythonExecute tool uses multiprocessing with a timeout parameter to prevent infinite loops and limit execution time. The sandbox environment enforces memory and CPU limits through Docker configuration. Path traversal is prevented by the _safe_resolve_path method, which validates container paths. Network access is disabled by default in the sandbox configuration to prevent unauthorized external connections. Additionally, the agent monitors execution results and can terminate processes that exceed resource limits or exhibit problematic behavior.

**Section sources**
- [python_execute.py](file://app/tool/python_execute.py#L8-L74)
- [sandbox.py](file://app/sandbox/core/sandbox.py#L17-L461)
- [config.py](file://app/config.py#L93-L104)

## Practical Example: Sorting Algorithm Implementation
To demonstrate the SWE Agent's capabilities, consider implementing a sorting algorithm. The agent would receive a natural language request to "implement a quicksort algorithm in Python." It would generate the appropriate code, execute it in the sandbox environment, and verify the results. If errors occur, such as incorrect sorting logic or runtime exceptions, the agent would analyze the error messages, debug the code, and refine the implementation until it produces correct results. This iterative process showcases the agent's ability to interpret requirements, generate functional code, and systematically resolve issues through automated testing and debugging.

**Section sources**
- [swe.py](file://app/agent/swe.py#L9-L23)
- [python_execute.py](file://app/tool/python_execute.py#L8-L74)

## Best Practices for Prompt Design
Effective prompt design is crucial for maximizing code quality and debugging efficiency with the SWE Agent. Prompts should be clear, specific, and include all necessary context for the task. They should specify the desired programming language, required functionality, input/output formats, and any constraints or requirements. Including examples of expected behavior can help guide the agent toward correct solutions. For debugging tasks, providing detailed error messages and context about the failure can enable more targeted fixes. Well-crafted prompts reduce ambiguity and help the agent generate higher-quality code on the first attempt, minimizing the need for iterative refinement.

**Section sources**
- [swe.py](file://app/prompt/swe.py#L0-L22)

## Conclusion
The SWE Agent in OpenManus provides a robust framework for autonomous code generation and debugging through natural language interaction. By combining sophisticated agent architecture with secure sandbox execution and comprehensive file operations, it enables reliable and safe code development. The integration of configuration settings allows for customization of security and resource parameters to meet specific requirements. Through iterative refinement based on execution results, the agent can successfully complete complex programming tasks while maintaining high standards of code quality and system security.
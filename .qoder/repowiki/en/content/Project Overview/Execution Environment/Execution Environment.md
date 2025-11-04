# Execution Environment

<cite>
**Referenced Files in This Document**   
- [DockerSandbox](file://app/sandbox/core/sandbox.py)
- [SandboxManager](file://app/sandbox/core/manager.py)
- [AsyncDockerizedTerminal](file://app/sandbox/core/terminal.py)
- [SandboxSettings](file://app/config.py)
- [create_sandbox](file://app/daytona/sandbox.py)
- [LocalSandboxClient](file://app/sandbox/client.py)
</cite>

## Table of Contents
1. [Sandbox Architecture](#sandbox-architecture)
2. [Core Components](#core-components)
3. [Resource Limitations and Security Model](#resource-limitations-and-security-model)
4. [Daytona Integration](#daytona-integration)
5. [Configuration Options](#configuration-options)
6. [Common Issues and Troubleshooting](#common-issues-and-troubleshooting)
7. [Performance Optimization and Security Best Practices](#performance-optimization-and-security-best-practices)

## Sandbox Architecture

The OpenManus execution environment provides isolated code execution through Docker containerization, ensuring secure and controlled execution of untrusted code. The architecture is built around three core components: DockerSandbox, SandboxManager, and AsyncDockerizedTerminal, which work together to create, manage, and interact with isolated execution environments.

```mermaid
graph TD
subgraph "Execution Environment"
A[DockerSandbox] --> B[SandboxManager]
A --> C[AsyncDockerizedTerminal]
C --> D[Docker Container]
B --> A
E[Daytona] --> A
F[LocalSandboxClient] --> A
end
A -.->|Creates and manages| D
B -.->|Manages lifecycle of| A
C -.->|Provides terminal interface| A
E -.->|External sandbox provider| A
F -.->|Client interface| A
```

**Diagram sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py)
- [SandboxManager](file://app/sandbox/core/manager.py)
- [AsyncDockerizedTerminal](file://app/sandbox/core/terminal.py)

**Section sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py)
- [SandboxManager](file://app/sandbox/core/manager.py)

## Core Components

The execution environment consists of three primary components that handle different aspects of sandbox management and interaction.

### DockerSandbox Class

The DockerSandbox class represents an isolated execution environment within a Docker container. It provides the fundamental capabilities for containerized code execution, including command execution, file operations, and resource management. The sandbox is initialized with configuration parameters that define its behavior and resource constraints.

```mermaid
classDiagram
class DockerSandbox {
+config : SandboxSettings
+volume_bindings : Dict[str, str]
+client : docker.DockerClient
+container : Optional[Container]
+terminal : Optional[AsyncDockerizedTerminal]
+__init__(config : Optional[SandboxSettings], volume_bindings : Optional[Dict[str, str]])
+create() DockerSandbox
+run_command(cmd : str, timeout : Optional[int]) str
+read_file(path : str) str
+write_file(path : str, content : str) None
+copy_from(src_path : str, dst_path : str) None
+copy_to(src_path : str, dst_path : str) None
+cleanup() None
}
DockerSandbox --> SandboxSettings : "uses"
DockerSandbox --> AsyncDockerizedTerminal : "contains"
DockerSandbox --> Container : "manages"
```

**Diagram sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py#L0-L46)

**Section sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py)

### SandboxManager Class

The SandboxManager class handles the lifecycle management of multiple DockerSandbox instances. It provides centralized control over sandbox creation, monitoring, and cleanup, ensuring efficient resource utilization and preventing resource exhaustion.

```mermaid
classDiagram
class SandboxManager {
+max_sandboxes : int
+idle_timeout : int
+cleanup_interval : int
+_sandboxes : Dict[str, DockerSandbox]
+_last_used : Dict[str, float]
+_locks : Dict[str, asyncio.Lock]
+_global_lock : asyncio.Lock
+_active_operations : Set[str]
+_cleanup_task : Optional[asyncio.Task]
+_is_shutting_down : bool
+__init__(max_sandboxes : int, idle_timeout : int, cleanup_interval : int)
+ensure_image(image : str) bool
+sandbox_operation(sandbox_id : str) AsyncContextManager
+create_sandbox(config : Optional[SandboxSettings], volume_bindings : Optional[Dict[str, str]]) str
+get_sandbox(sandbox_id : str) DockerSandbox
+start_cleanup_task() None
+_cleanup_idle_sandboxes() None
+cleanup() None
+_safe_delete_sandbox(sandbox_id : str) None
+delete_sandbox(sandbox_id : str) None
+get_stats() Dict
}
SandboxManager --> DockerSandbox : "manages"
SandboxManager --> asyncio.Lock : "uses"
SandboxManager --> asyncio.Task : "uses"
```

**Diagram sources**
- [SandboxManager](file://app/sandbox/core/manager.py#L0-L46)

**Section sources**
- [SandboxManager](file://app/sandbox/core/manager.py)

### AsyncDockerizedTerminal Class

The AsyncDockerizedTerminal class provides an asynchronous interface for executing commands within a Docker container. It manages the terminal session, handles command execution, and ensures proper cleanup of resources.

```mermaid
classDiagram
class AsyncDockerizedTerminal {
+client : docker.DockerClient
+container : Container
+working_dir : str
+env_vars : Dict[str, str]
+default_timeout : int
+session : Optional[DockerSession]
+__init__(container : Union[str, Container], working_dir : str, env_vars : Optional[Dict[str, str]], default_timeout : int)
+init() None
+_ensure_workdir() None
+_exec_simple(cmd : str) Tuple[int, str]
+run_command(cmd : str, timeout : Optional[int]) str
+close() None
}
class DockerSession {
+api : APIClient
+container_id : str
+exec_id : Optional[str]
+socket : Optional[socket.socket]
+__init__(container_id : str)
+create(working_dir : str, env_vars : Dict[str, str]) None
+close() None
+_read_until_prompt() str
+execute(command : str, timeout : Optional[int]) str
+_sanitize_command(command : str) str
}
AsyncDockerizedTerminal --> DockerSession : "contains"
AsyncDockerizedTerminal --> Container : "interacts with"
DockerSession --> APIClient : "uses"
DockerSession --> socket.socket : "uses"
```

**Diagram sources**
- [AsyncDockerizedTerminal](file://app/sandbox/core/terminal.py#L0-L50)

**Section sources**
- [AsyncDockerizedTerminal](file://app/sandbox/core/terminal.py)

## Resource Limitations and Security Model

The execution environment implements comprehensive resource limitations and security measures to ensure safe and controlled code execution.

### Resource Limitations

The DockerSandbox class enforces resource limitations through Docker container configuration. These limitations are defined in the SandboxSettings configuration and applied during container creation.

```mermaid
flowchart TD
Start([Container Creation]) --> MemoryLimit["Set memory limit<br/>from config.memory_limit"]
MemoryLimit --> CPULimit["Set CPU limit<br/>from config.cpu_limit"]
CPULimit --> NetworkMode["Set network mode:<br/>'none' if network_disabled<br/>'bridge' otherwise"]
NetworkMode --> VolumeBindings["Configure volume bindings:<br/>- Working directory mapping<br/>- Custom volume mappings"]
VolumeBindings --> ContainerConfig["Create container with<br/>configured resource limits"]
ContainerConfig --> End([Container Ready])
```

**Diagram sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py#L48-L87)

**Section sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py)
- [SandboxSettings](file://app/config.py)

### Security Model

The execution environment implements multiple layers of security to prevent malicious activities and ensure isolation.

```mermaid
flowchart TD
subgraph "Security Measures"
A[Path Traversal Prevention] --> B["_safe_resolve_path method"]
B --> C["Rejects paths containing '..'"]
C --> D["Resolves relative paths within work_dir"]
E[Command Sanitization] --> F["_sanitize_command method"]
F --> G["Blocks dangerous commands:<br/>- rm -rf /<br/>- mkfs<br/>- dd if=/dev/zero<br/>- chmod -R 777 /"]
H[Network Isolation] --> I["network_mode='none' by default"]
I --> J["Bridge network only when enabled"]
K[File Operations] --> L["Uses tar streams for file transfer"]
L --> M["Validates file paths before operations"]
end
```

**Diagram sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py#L218-L259)
- [AsyncDockerizedTerminal](file://app/sandbox/core/terminal.py#L212-L258)

**Section sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py)
- [AsyncDockerizedTerminal](file://app/sandbox/core/terminal.py)

## Daytona Integration

The execution environment integrates with Daytona for workspace and session management, providing enhanced capabilities for remote sandbox operations.

### Daytona Sandbox Management

The Daytona integration allows for the creation and management of sandboxes through the Daytona platform, enabling persistent workspaces and advanced session management.

```mermaid
sequenceDiagram
participant Client
participant OpenManus
participant Daytona
Client->>OpenManus : create_sandbox(password, project_id)
OpenManus->>Daytona : Create sandbox with parameters
Daytona-->>OpenManus : Return sandbox instance
OpenManus->>Daytona : Start supervisord session
Daytona-->>OpenManus : Session started
OpenManus-->>Client : Sandbox created with ID
Client->>OpenManus : get_or_start_sandbox(sandbox_id)
OpenManus->>Daytona : Get sandbox by ID
Daytona-->>OpenManus : Return sandbox instance
alt Sandbox stopped/archived
OpenManus->>Daytona : Start sandbox
Daytona-->>OpenManus : Sandbox started
OpenManus->>Daytona : Start supervisord session
Daytona-->>OpenManus : Session started
end
OpenManus-->>Client : Sandbox ready
Client->>OpenManus : delete_sandbox(sandbox_id)
OpenManus->>Daytona : Delete sandbox
Daytona-->>OpenManus : Deletion confirmed
OpenManus-->>Client : Deletion successful
```

**Diagram sources**
- [create_sandbox](file://app/daytona/sandbox.py#L107-L164)
- [get_or_start_sandbox](file://app/daytona/sandbox.py#L52-L85)
- [delete_sandbox](file://app/daytona/sandbox.py#L147-L164)

**Section sources**
- [create_sandbox](file://app/daytona/sandbox.py)
- [get_or_start_sandbox](file://app/daytona/sandbox.py)
- [delete_sandbox](file://app/daytona/sandbox.py)

### Configuration and Environment Variables

The Daytona integration uses specific configuration parameters and environment variables to customize sandbox behavior.

```mermaid
flowchart TD
subgraph "Daytona Configuration"
A[CreateSandboxFromImageParams] --> B["image: sandbox_image_name"]
A --> C["public: true"]
A --> D["labels: {id: project_id}"]
A --> E["env_vars: CHROME_PERSISTENT_SESSION, RESOLUTION, VNC_PASSWORD, etc."]
A --> F["resources: CPU=2, Memory=4GB, Disk=5GB"]
A --> G["auto_stop_interval: 15 minutes"]
A --> H["auto_archive_interval: 24 hours"]
end
subgraph "Environment Variables"
I[CHROME_PERSISTENT_SESSION: "true"] --> J[Enables persistent Chrome session]
K[RESOLUTION: "1024x768x24"] --> L[Sets display resolution]
M[VNC_PASSWORD: password] --> N[Sets VNC access password]
O[ANONYMIZED_TELEMETRY: "false"] --> P[Disables telemetry]
end
```

**Diagram sources**
- [create_sandbox](file://app/daytona/sandbox.py#L107-L164)

**Section sources**
- [create_sandbox](file://app/daytona/sandbox.py)
- [DaytonaSettings](file://app/config.py)

## Configuration Options

The execution environment provides extensive configuration options for customizing sandbox behavior, resource limits, and storage settings.

### Container Image Configuration

The sandbox environment can be configured to use different container images based on the requirements of the execution environment.

```mermaid
flowchart TD
A[Sandbox Configuration] --> B["image: Container image name"]
B --> C{Default: python:3.12-slim}
C --> D[Custom image options]
D --> E[python:3.9-slim]
D --> F[python:3.11-slim]
D --> G[custom-base-image:tag]
D --> H[whitezxj/sandbox:0.1.0 for Daytona]
I[SandboxManager] --> J[ensure_image method]
J --> K[Checks if image exists locally]
K --> L{Image found?}
L --> |Yes| M[Uses existing image]
L --> |No| N[Pulls image from registry]
N --> O[Handles pull failures]
```

**Diagram sources**
- [SandboxSettings](file://app/config.py#L93-L110)
- [SandboxManager](file://app/sandbox/core/manager.py#L48-L94)

**Section sources**
- [SandboxSettings](file://app/config.py)
- [SandboxManager](file://app/sandbox/core/manager.py)

### Resource Limits Configuration

Resource limits can be configured to control the computational resources available to each sandbox instance.

```mermaid
flowchart TD
A[SandboxSettings] --> B["memory_limit: Memory allocation"]
B --> C{Default: 512m}
C --> D[Examples: 1g, 2g, 512m]
A --> E["cpu_limit: CPU allocation"]
E --> F{Default: 1.0}
F --> G[Examples: 0.5 (50% of one CPU), 2.0 (2 CPUs)]
A --> H["timeout: Command execution timeout"]
H --> I{Default: 300 seconds}
I --> J[Custom values: 60, 600, 1800]
A --> K["network_enabled: Network access"]
K --> L{Default: false}
L --> M[true: Bridge network]
L --> N[false: No network]
```

**Diagram sources**
- [SandboxSettings](file://app/config.py#L93-L110)
- [DockerSandbox](file://app/sandbox/core/sandbox.py#L48-L87)

**Section sources**
- [SandboxSettings](file://app/config.py)
- [DockerSandbox](file://app/sandbox/core/sandbox.py)

### Persistent Storage Configuration

The execution environment supports persistent storage through volume bindings, allowing data to persist beyond the lifecycle of individual sandbox instances.

```mermaid
flowchart TD
A[DockerSandbox] --> B["_prepare_volume_bindings method"]
B --> C["Creates volume bindings dictionary"]
C --> D["Host working directory"]
D --> E["Temporary directory with random suffix"]
E --> F["os.path.join(tempfile.gettempdir(), f'sandbox_{basename}_{random_hex}')"]
C --> G["Custom volume bindings"]
G --> H["Maps host_path to container_path"]
H --> I["Mode: rw (read-write)"]
A --> J["volume_bindings parameter"]
J --> K["Accepts dictionary of {host_path: container_path}"]
K --> L["Enables custom volume mappings"]
```

**Diagram sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py#L89-L127)

**Section sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py)

## Common Issues and Troubleshooting

This section addresses common issues that may occur when working with the execution environment and provides troubleshooting guidance.

### Container Startup Failures

Container startup failures can occur due to various reasons, including missing images, resource constraints, or configuration issues.

```mermaid
flowchart TD
A[Container Startup] --> B{Image available?}
B --> |No| C[Pull image from registry]
C --> D{Pull successful?}
D --> |No| E[Log error: Failed to pull image]
D --> |Yes| F[Proceed with container creation]
B --> |Yes| F
F --> G[Create container with host config]
G --> H{Creation successful?}
H --> |No| I[Log error: Container creation failed]
H --> |Yes| J[Start container]
J --> K{Start successful?}
K --> |No| L[Log error: Container startup failed]
K --> |Yes| M[Initialize terminal interface]
M --> N{Initialization successful?}
N --> |No| O[Cleanup resources]
N --> |Yes| P[Container ready]
```

**Diagram sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py#L48-L87)
- [SandboxManager](file://app/sandbox/core/manager.py#L136-L174)

**Section sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py)
- [SandboxManager](file://app/sandbox/core/manager.py)

### Resource Exhaustion

Resource exhaustion can occur when the system runs out of memory, CPU, or when the maximum number of sandboxes is reached.

```mermaid
flowchart TD
A[SandboxManager] --> B["max_sandboxes limit"]
B --> C{Number of sandboxes < max?}
C --> |No| D[Return error: Maximum sandboxes reached]
C --> |Yes| E[Proceed with sandbox creation]
E --> F[Monitor resource usage]
F --> G{Memory usage high?}
G --> |Yes| H[Log warning: High memory usage]
F --> I{CPU usage high?}
I --> |Yes| J[Log warning: High CPU usage]
K[Idle sandbox cleanup] --> L["idle_timeout parameter"]
L --> M{Sandbox idle > timeout?}
M --> |Yes| N[Clean up idle sandbox]
M --> |No| O[Keep sandbox active]
```

**Diagram sources**
- [SandboxManager](file://app/sandbox/core/manager.py#L96-L134)
- [SandboxManager](file://app/sandbox/core/manager.py#L210-L244)

**Section sources**
- [SandboxManager](file://app/sandbox/core/manager.py)

### Network Connectivity Problems

Network connectivity issues can arise due to the default network isolation settings or configuration problems.

```mermaid
flowchart TD
A[DockerSandbox] --> B["network_enabled setting"]
B --> C{True?}
C --> |No| D[Network mode: 'none']
C --> |Yes| E[Network mode: 'bridge']
D --> F[No network access]
E --> G[Network access enabled]
G --> H[Test network connectivity]
H --> I{Can reach external hosts?}
I --> |No| J[Check Docker network configuration]
I --> |Yes| K[Network working correctly]
L[Daytona integration] --> M[External network access]
M --> N[VNC connectivity]
N --> O{VNC password correct?}
O --> |No| P[Connection failed]
O --> |Yes| Q[Connection successful]
```

**Diagram sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py#L48-L87)
- [create_sandbox](file://app/daytona/sandbox.py#L107-L164)

**Section sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py)
- [create_sandbox](file://app/daytona/sandbox.py)

## Performance Optimization and Security Best Practices

This section provides recommendations for optimizing performance and ensuring security when deploying the execution environment in production.

### Performance Optimization Tips

Optimizing the execution environment can significantly improve performance and resource utilization.

```mermaid
flowchart TD
A[Performance Optimization] --> B["Pre-pull commonly used images"]
B --> C[Reduces container startup time]
A --> D["Set appropriate resource limits"]
D --> E[Avoid over-allocation of CPU and memory]
E --> F[Prevents resource contention]
A --> G["Configure optimal idle timeout"]
G --> H[Balances resource cleanup with reuse]
H --> I[Default: 3600 seconds]
A --> J["Use connection pooling"]
J --> K[Reuse Docker client connections]
K --> L[Reduces connection overhead]
A --> M["Monitor resource usage"]
M --> N[Identify performance bottlenecks]
N --> O[Adjust configuration accordingly]
```

**Diagram sources**
- [SandboxManager](file://app/sandbox/core/manager.py#L0-L46)
- [SandboxSettings](file://app/config.py#L93-L110)

**Section sources**
- [SandboxManager](file://app/sandbox/core/manager.py)
- [SandboxSettings](file://app/config.py)

### Security Best Practices

Implementing security best practices is crucial for protecting the system and ensuring safe code execution.

```mermaid
flowchart TD
A[Security Best Practices] --> B["Keep network access disabled by default"]
B --> C[Enable only when necessary]
A --> D["Use minimal base images"]
D --> E[Reduces attack surface]
E --> F[Example: python:3.12-slim]
A --> G["Regularly update base images"]
G --> H[Patches security vulnerabilities]
A --> I["Implement proper error handling"]
I --> J[Prevents information leakage]
J --> K[Log errors without exposing details]
A --> L["Use unique container names"]
L --> M[Generated with UUID]
M --> N[Prevents naming conflicts]
A --> O["Validate all file paths"]
O --> P[Prevents path traversal attacks]
P --> Q[Rejects paths with '..']
A --> R["Sanitize all commands"]
R --> S[Blocks dangerous operations]
S --> T[Checks for rm -rf /, mkfs, etc.]
```

**Diagram sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py#L218-L259)
- [AsyncDockerizedTerminal](file://app/sandbox/core/terminal.py#L212-L258)

**Section sources**
- [DockerSandbox](file://app/sandbox/core/sandbox.py)
- [AsyncDockerizedTerminal](file://app/sandbox/core/terminal.py)
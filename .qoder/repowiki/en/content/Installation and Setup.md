# Installation and Setup

<cite>
**Referenced Files in This Document**   
- [README.md](file://README.md)
- [config.example.toml](file://config/config.example.toml)
- [requirements.txt](file://requirements.txt)
- [setup.py](file://setup.py)
- [config.py](file://app/config.py)
</cite>

## Table of Contents
1. [Environment Prerequisites](#environment-prerequisites)
2. [Installation Methods](#installation-methods)
3. [Browser Automation Setup](#browser-automation-setup)
4. [Configuration Setup](#configuration-setup)
5. [LLM Provider Configuration Examples](#llm-provider-configuration-examples)
6. [Troubleshooting Common Issues](#troubleshooting-common-issues)

## Environment Prerequisites

Before installing OpenManus, ensure your system meets the following requirements:

- **Python 3.12**: OpenManus requires Python 3.12 specifically. The codebase includes version checks to warn users if an incompatible Python version is used.
- **Docker**: Required for sandboxing functionality to execute code in isolated environments. Docker enables containerized execution with resource limits and isolation.
- **Playwright**: Optional but recommended for browser automation capabilities. Playwright provides cross-browser automation support for Chromium, Firefox, and WebKit.

The project is designed to work across different operating systems, but Docker must be properly installed and running in the background to support sandboxed execution environments.

**Section sources**
- [README.md](file://README.md#L0-L195)
- [app/__init__.py](file://app/__init__.py#L0-L9)
- [app/config.py](file://app/config.py#L0-L372)

## Installation Methods

OpenManus provides two installation methods, with the `uv` package manager being the recommended approach for faster dependency resolution and better virtual environment management.

### Method 1: Using conda

For users who prefer conda as their package manager, follow these steps:

1. Create a new conda environment with Python 3.12:
```bash
conda create -n open_manus python=3.12
conda activate open_manus
```

2. Clone the OpenManus repository:
```bash
git clone https://github.com/FoundationAgents/OpenManus.git
cd OpenManus
```

3. Install the required dependencies from the requirements file:
```bash
pip install -r requirements.txt
```

This method creates an isolated environment using conda and installs all dependencies listed in the requirements.txt file via pip.

### Method 2: Using uv (Recommended)

The `uv` package manager is recommended for its speed and efficiency in dependency resolution:

1. Install the `uv` tool using the official installation script:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

2. Clone the repository and navigate into the project directory:
```bash
git clone https://github.com/FoundationAgents/OpenManus.git
cd OpenManus
```

3. Create a virtual environment with Python 3.12 and activate it:
```bash
uv venv --python 3.12
source .venv/bin/activate  # On Unix/macOS
# Or on Windows:
# .venv\Scripts\activate
```

4. Install dependencies using uv:
```bash
uv pip install -r requirements.txt
```

The uv method typically results in faster installation times and more reliable dependency resolution compared to traditional pip installations.

**Section sources**
- [README.md](file://README.md#L0-L195)
- [requirements.txt](file://requirements.txt#L0-L42)
- [setup.py](file://setup.py#L0-L49)

## Browser Automation Setup

Browser automation is an optional feature that can be enabled for enhanced web interaction capabilities:

1. After installing the main dependencies, install Playwright browser binaries:
```bash
playwright install
```

This command downloads the necessary browser executables (Chromium, Firefox, and WebKit) that enable automated browser interactions. The Playwright integration allows OpenManus to perform tasks such as web navigation, form filling, and content extraction from web pages.

The browser automation functionality is implemented through the `browser-use` package, which provides a high-level API for browser interactions. Configuration options for browser behavior (such as headless mode, proxy settings, and security features) can be specified in the configuration file.

**Section sources**
- [README.md](file://README.md#L0-L195)
- [requirements.txt](file://requirements.txt#L0-L42)
- [app/tool/browser_use_tool.py](file://app/tool/browser_use_tool.py#L145-L204)

## Configuration Setup

Proper configuration is essential for OpenManus to interact with LLM APIs and other services.

### Creating the Configuration File

1. Copy the example configuration file to create your own configuration:
```bash
cp config/config.example.toml config/config.toml
```

2. Edit the `config/config.toml` file to add your API keys and customize settings according to your needs.

The configuration system supports multiple LLM configurations and optional components. The main configuration sections include:

- **LLM settings**: Global language model configuration including model name, API endpoint, and authentication key
- **Vision model settings**: Separate configuration for vision-capable models when image processing is required
- **Browser settings**: Options for browser automation behavior
- **Search settings**: Configuration for web search functionality
- **Sandbox settings**: Parameters for the containerized execution environment
- **MCP settings**: Model Context Protocol server configurations
- **Runflow settings**: Options for multi-agent workflows

The configuration is loaded by the `Config` class in `app/config.py`, which implements a singleton pattern to ensure consistent access to configuration values throughout the application.

**Section sources**
- [README.md](file://README.md#L0-L195)
- [config.example.toml](file://config/config.example.toml#L0-L105)
- [app/config.py](file://app/config.py#L0-L372)

## LLM Provider Configuration Examples

OpenManus supports multiple LLM providers with specific configuration requirements for each.

### OpenAI Configuration

To configure OpenManus to use OpenAI models:

```toml
[llm]
model = "gpt-4o"
base_url = "https://api.openai.com/v1"
api_key = "sk-..."  # Replace with your actual API key
max_tokens = 4096
temperature = 0.0

[llm.vision]
model = "gpt-4o"
base_url = "https://api.openai.com/v1"
api_key = "sk-..."  # Replace with your actual API key
```

### Anthropic Configuration

For Anthropic's Claude models:

```toml
[llm]
model = "claude-3-7-sonnet-20250219"
base_url = "https://api.anthropic.com/v1/"
api_key = "YOUR_API_KEY"
max_tokens = 8192
temperature = 0.0

[llm.vision]
model = "claude-3-7-sonnet-20250219"
base_url = "https://api.anthropic.com/v1/"
api_key = "YOUR_API_KEY"
```

### Ollama Configuration

To use locally hosted models via Ollama:

```toml
[llm]
api_type = 'ollama'
model = "llama3.2"
base_url = "http://localhost:11434/v1"
api_key = "ollama"
max_tokens = 4096
temperature = 0.0

[llm.vision]
api_type = 'ollama'
model = "llama3.2-vision"
base_url = "http://localhost:11434/v1"
api_key = "ollama"
```

### Azure OpenAI Configuration

For Azure-hosted OpenAI services:

```toml
[llm]
api_type= 'azure'
model = "gpt-4o-mini"
base_url = "{YOUR_AZURE_ENDPOINT.rstrip('/')}/openai/deployments/{AZURE_DEPLOYMENT_ID}"
api_key = "YOUR_API_KEY"
max_tokens = 8096
temperature = 0.0
api_version="2024-08-01-preview"
```

### Google Gemini Configuration

To use Google's Gemini models:

```toml
[llm]
model = "gemini-2.0-flash"
base_url = "https://generativelanguage.googleapis.com/v1beta/openai/"
api_key = "YOUR_API_KEY"
temperature = 0.0
max_tokens = 8096
```

The configuration system allows for provider-specific settings through the `api_type` field, which determines how the LLM client will communicate with the service endpoint.

**Section sources**
- [config.example.toml](file://config/config.example.toml#L0-L105)
- [config.example-model-anthropic.toml](file://config/config.example-model-anthropic.toml#L0-L16)
- [config.example-model-azure.toml](file://config/config.example-model-azure.toml#L0-L18)
- [config.example-model-google.toml](file://config/config.example-model-google.toml#L0-L16)
- [config.example-model-ollama.toml](file://config/config.example-model-ollama.toml#L0-L17)
- [app/config.py](file://app/config.py#L0-L372)

## Troubleshooting Common Issues

Address common installation and setup problems with these solutions:

### Dependency Conflicts

If you encounter dependency conflicts during installation:

1. Use the recommended `uv` package manager instead of pip for better dependency resolution
2. Create a fresh virtual environment to avoid conflicts with existing packages
3. Ensure you're using Python 3.12 exactly, as specified in the requirements
4. Check that your `requirements.txt` file matches the version constraints in `setup.py`

### Missing Browser Binaries

If Playwright fails to launch browsers:

1. Ensure you've run `playwright install` after installing the Python package
2. Verify that the Playwright package version matches the browser binaries
3. If behind a corporate firewall, set appropriate proxy settings in the configuration file
4. On Linux systems, ensure all required system dependencies are installed (libglib2.0-0, libnss3, etc.)

### Docker Sandbox Issues

For problems with the sandboxed execution environment:

1. Ensure Docker is installed and running on your system
2. Verify that the user has permission to access the Docker daemon
3. Check that the `python:3.12-slim` image can be pulled from Docker Hub
4. Ensure sufficient system resources (memory and CPU) are available for container creation

### API Connection Problems

When experiencing issues connecting to LLM APIs:

1. Verify that API keys are correctly copied without leading/trailing spaces
2. Check that the `base_url` matches the provider's current API endpoint
3. Ensure network connectivity to the API endpoint (no firewall blocking)
4. For Azure OpenAI, verify that the deployment name matches exactly with what's configured in the Azure portal

### Configuration Loading Errors

If configuration values are not being applied:

1. Ensure the configuration file is named `config.toml` and located in the `config` directory
2. Verify that TOML syntax is correct (proper use of brackets, quotes, and indentation)
3. Check that environment variables aren't overriding configuration file values
4. Restart the application after making configuration changes

**Section sources**
- [README.md](file://README.md#L0-L195)
- [requirements.txt](file://requirements.txt#L0-L42)
- [app/config.py](file://app/config.py#L0-L372)
- [app/sandbox/core/sandbox.py](file://app/sandbox/core/sandbox.py#L0-L172)
- [app/sandbox/core/manager.py](file://app/sandbox/core/manager.py#L0-L94)
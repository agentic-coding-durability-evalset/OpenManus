# Installation and Setup

<cite>
**Referenced Files in This Document**   
- [README.md](file://README.md)
- [requirements.txt](file://requirements.txt)
- [config.example.toml](file://config/config.example.toml)
- [chart_visualization/README.md](file://app/tool/chart_visualization/README.md)
</cite>

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installation Methods](#installation-methods)
3. [Configuration Setup](#configuration-setup)
4. [Optional Components](#optional-components)
5. [Troubleshooting Guide](#troubleshooting-guide)
6. [Best Practices](#best-practices)

## Prerequisites

Before installing OpenManus, ensure your system meets the following requirements:

- **Python 3.12**: OpenManus is built and tested with Python 3.12. Using this specific version ensures compatibility with all dependencies and prevents potential runtime issues.
- **Git**: Required to clone the repository from GitHub.
- **Internet Connection**: Necessary for downloading dependencies, API communication, and optional component installations.

The system is designed to work across multiple platforms including Linux, macOS, and Windows, though some optional components may have platform-specific installation steps.

**Section sources**
- [README.md](file://README.md#L50-L55)

## Installation Methods

OpenManus provides two installation methods, with the uv package manager being the recommended approach for optimal performance and dependency resolution.

### Method 1: Using conda

For users familiar with conda environments, follow these steps:

1. Create and activate a dedicated conda environment:
```bash
conda create -n open_manus python=3.12
conda activate open_manus
```

2. Clone the repository and navigate into the project directory:
```bash
git clone https://github.com/FoundationAgents/OpenManus.git
cd OpenManus
```

3. Install all required Python dependencies:
```bash
pip install -r requirements.txt
```

This method leverages conda's environment management capabilities while using pip for Python package installation. The conda environment provides isolation, preventing conflicts with system-wide Python packages.

### Method 2: Using uv (Recommended)

The uv package manager offers faster installation and superior dependency resolution compared to traditional methods:

1. Install the uv tool:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

2. Clone the repository:
```bash
git clone https://github.com/FoundationAgents/OpenManus.git
cd OpenManus
```

3. Create and activate a virtual environment:
```bash
uv venv --python 3.12
source .venv/bin/activate  # On Unix/macOS
# On Windows: .venv\Scripts\activate
```

4. Install dependencies using uv:
```bash
uv pip install -r requirements.txt
```

The uv method is recommended because it provides significantly faster package resolution and installation times, better dependency conflict detection, and more reliable environment reproducibility across different systems.

**Section sources**
- [README.md](file://README.md#L57-L99)
- [requirements.txt](file://requirements.txt#L1-L43)

## Configuration Setup

Proper configuration is essential for OpenManus to interact with LLM services and other components.

1. Create a configuration file by copying the example:
```bash
cp config/config.example.toml config/config.toml
```

2. Edit the `config/config.toml` file to configure your LLM providers. The configuration supports multiple providers including OpenAI, Anthropic, Azure, and others:

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

For Anthropic:
```toml
[llm]
model = "claude-3-7-sonnet-20250219"
base_url = "https://api.anthropic.com/v1/"
api_key = "YOUR_API_KEY"
```

For Azure OpenAI:
```toml
[llm]
api_type = 'azure'
model = "YOUR_MODEL_NAME"
base_url = "{YOUR_AZURE_ENDPOINT.rstrip('/')}/openai/deployments/{AZURE_DEPLOYMENT_ID}"
api_key = "AZURE API KEY"
api_version = "AZURE API VERSION"
```

The configuration file uses TOML format and supports environment variable interpolation for enhanced security. Sensitive information like API keys should be stored securely and not committed to version control.

**Section sources**
- [config.example.toml](file://config/config.example.toml#L1-L40)
- [README.md](file://README.md#L101-L120)

## Optional Components

OpenManus supports several optional components that extend its functionality for specific use cases.

### Playwright for Browser Automation

For browser automation capabilities, install Playwright and its browser binaries:

```bash
playwright install
```

This command downloads the necessary browser binaries (Chromium, Firefox, and WebKit) required for web automation tasks. The installation can be customized to install only specific browsers if disk space is a concern.

### Chart Visualization Dependencies

To enable data analysis and visualization capabilities, additional dependencies are required:

#### Node.js Installation
The chart visualization tool requires Node.js (version 18 or higher):

**Mac/Linux:**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install node
nvm use 22
```

**Windows:**
Download and install nvm-windows from the official GitHub page, then:
```powershell
nvm install node
nvm use 22
```

#### NPM Dependencies
After Node.js installation, navigate to the chart visualization directory and install dependencies:
```bash
cd app/tool/chart_visualization
npm install
```

These components enable the DataAnalysis agent to process data, generate reports, and create visualizations using VMind and VChart libraries. The visualization_preparation tool converts data analysis results into JSON configuration files, which are then rendered by the data_visualization component.

**Section sources**
- [README.md](file://README.md#L122-L125)
- [chart_visualization/README.md](file://app/tool/chart_visualization/README.md#L6-L45)

## Troubleshooting Guide

This section addresses common issues encountered during installation and setup.

### Missing Dependencies
If you encounter import errors or missing module exceptions:
- Verify your virtual environment is activated
- Reinstall dependencies: `pip install -r requirements.txt` or `uv pip install -r requirements.txt`
- Check that Python 3.12 is being used: `python --version`
- Ensure all required system dependencies are installed (e.g., libgl1 for OpenCV)

### API Key Errors
When receiving authentication or authorization errors:
- Verify the API key is correctly copied without extra spaces
- Check that the base_url matches your provider's endpoint
- Ensure the API key has the necessary permissions
- For Azure, verify the api_version is correctly specified
- Consider using environment variables instead of hardcoding keys

### Docker Connectivity Issues
For sandboxed execution problems:
- Ensure Docker is running and accessible
- Verify your user has permissions to access the Docker daemon
- Check that sufficient system resources (memory, CPU) are available
- Verify network connectivity for container downloads

### Playwright Installation Failures
If Playwright fails to install browser binaries:
- Ensure you have sufficient disk space (several GB)
- Check network connectivity and firewall settings
- Try installing specific browsers: `playwright install chromium`
- Run with elevated permissions if encountering permission errors

**Section sources**
- [README.md](file://README.md#L50-L150)
- [requirements.txt](file://requirements.txt#L1-L43)

## Best Practices

To ensure a stable and secure OpenManus installation, follow these best practices:

### Environment Isolation
Always use isolated environments (conda or virtualenv) to prevent dependency conflicts with other Python projects. This ensures that OpenManus has access to the exact dependency versions it requires.

### Dependency Management
Use the recommended uv package manager for faster, more reliable dependency resolution. Regularly update dependencies while being cautious about breaking changes.

### Configuration Security
Never commit API keys or sensitive information to version control. Use environment variables or secure credential storage solutions. The configuration system supports variable interpolation for this purpose.

### Regular Updates
Keep both OpenManus and its dependencies up to date to benefit from bug fixes, security patches, and new features. Monitor the repository for updates and breaking changes.

### Testing Installation
After setup, verify the installation by running:
```bash
python main.py
```
And providing a simple test prompt to ensure all components are functioning correctly.

**Section sources**
- [README.md](file://README.md#L50-L150)
- [main.py](file://main.py#L1-L36)
# Human Interaction

<cite>
**Referenced Files in This Document**   
- [ask_human.py](file://app/tool/ask_human.py)
- [base.py](file://app/tool/base.py)
- [manus.py](file://app/agent/manus.py)
- [sandbox_agent.py](file://app/agent/sandbox_agent.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Core Implementation](#core-implementation)
3. [Parameter Schema and LLM Integration](#parameter-schema-and-llm-integration)
4. [Asynchronous Execution Model](#asynchronous-execution-model)
5. [Use Cases for Human Interaction](#use-cases-for-human-interaction)
6. [Integration with Agent Workflows](#integration-with-agent-workflows)
7. [Limitations and Constraints](#limitations-and-constraints)
8. [Best Practices for Inquiry Design](#best-practices-for-inquiry-design)

## Introduction
The AskHuman tool in OpenManus enables interactive communication between AI agents and human users, allowing agents to request clarification, approval, or additional information during task execution. This mechanism bridges the gap between autonomous operation and human oversight, ensuring that agents can handle ambiguous situations, ethical considerations, and complex decision points that require human judgment. The tool is integrated into various agent implementations, including Manus and SandboxManus, and serves as a critical component for maintaining safe and effective agent behavior.

## Core Implementation
The AskHuman tool provides a simple yet effective implementation for human interaction using Python's built-in `input()` function. When invoked, the tool pauses execution and waits for user input through standard input, creating a synchronous blocking operation. The implementation strips whitespace from the input using Python's `strip()` method to ensure clean data processing. The tool's execute method formats the inquiry with a clear "Bot:" prefix for the question and "You:" prompt for the user response, creating a conversational interface. This straightforward approach allows for immediate integration into command-line environments and debugging scenarios where direct human input is available.

**Section sources**
- [ask_human.py](file://app/tool/ask_human.py#L19-L20)

## Parameter Schema and LLM Integration
The AskHuman tool defines a structured parameter schema compatible with LLM function calling mechanisms. The schema specifies an object with a single required property "inquire" of type string, which contains the question to be asked to the human user. This schema follows the OpenAI function calling format, enabling seamless integration with LLM systems that support tool use. The parameters are defined as a JSON schema within the tool class, allowing the agent's planning system to understand the required inputs for invoking the tool. This structured approach ensures that the LLM can properly format tool calls with appropriate inquiry content, maintaining consistency across different agent implementations.

**Section sources**
- [ask_human.py](file://app/tool/ask_human.py#L8-L17)

## Asynchronous Execution Model
Despite using a blocking I/O operation (`input()`), the AskHuman tool is implemented as an asynchronous method to maintain compatibility with the agent's event loop architecture. The `execute` method is defined with the `async` keyword, allowing it to be awaited within the agent's asynchronous workflow. This design enables the tool to be integrated into the agent's tool execution pipeline alongside other asynchronous tools, even though the actual input operation is synchronous. The tool inherits from the BaseTool class, which provides the asynchronous calling interface, ensuring consistent behavior across all tools in the system. This hybrid approach allows the agent to manage the tool within its asynchronous framework while leveraging Python's simple input mechanism for human interaction.

**Section sources**
- [ask_human.py](file://app/tool/ask_human.py#L19-L20)
- [base.py](file://app/tool/base.py#L77-L172)

## Use Cases for Human Interaction
The AskHuman tool addresses several critical scenarios where human input is essential for proper agent operation. These include ambiguous instructions that require clarification, ethical considerations that need human approval before proceeding, and complex decision points that benefit from human judgment. The tool is particularly valuable when the agent encounters situations outside its training data or when safety-critical decisions must be made. By pausing execution to request human input, the agent can avoid making potentially harmful or incorrect decisions. This capability is especially important in applications involving sensitive data, financial transactions, or actions with real-world consequences.

## Integration with Agent Workflows
The AskHuman tool is integrated into multiple agent implementations within OpenManus, including the Manus and SandboxManus agents. It is included in the available_tools collection for these agents, making it accessible during task execution. The tool is registered alongside other core tools like PythonExecute, BrowserUseTool, and Terminate, allowing the agent to select it when appropriate. When the LLM determines that human input is needed, it generates a tool call with the "inquire" parameter containing the specific question. The agent's tool execution system then invokes the AskHuman tool, which pauses execution to collect user input before resuming the workflow with the human-provided information.

**Section sources**
- [manus.py](file://app/agent/manus.py#L33-L41)
- [sandbox_agent.py](file://app/agent/sandbox_agent.py#L36-L44)

## Limitations and Constraints
The AskHuman tool has several important limitations that affect its usability in different environments. The primary limitation is the blocking nature of the `input()` function, which halts all agent execution until human input is received. This creates a single-threaded interaction model that cannot handle multiple concurrent inquiries. The implementation lacks timeout handling, potentially causing the agent to wait indefinitely for user response. Additionally, the reliance on standard input limits deployment to environments where direct console access is available, making it unsuitable for web-based or headless applications without additional adaptation. These constraints mean the tool is best suited for development, debugging, and command-line applications rather than production systems requiring high availability.

## Best Practices for Inquiry Design
When using the AskHuman tool, crafting effective inquiry messages is crucial for obtaining useful human input. Inquiries should be clear, concise, and specific, providing sufficient context for the user to understand what information is needed. The question should directly address the decision point or ambiguity that requires human judgment. It's important to avoid vague or open-ended questions that may lead to unhelpful responses. When requesting approval for actions, the inquiry should clearly state the proposed action and its potential consequences. For clarification requests, the question should identify the specific aspect of the instructions that is ambiguous. Well-crafted inquiries improve the efficiency of human-agent collaboration and reduce the likelihood of miscommunication.
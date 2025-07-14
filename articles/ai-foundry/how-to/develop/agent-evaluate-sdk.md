---
title: Agent Evaluation with Azure AI Evaluation SDK
titleSuffix: Azure AI Foundry
description: This article provides instructions on how to evaluate an AI agent with the Azure AI Evaluation SDK.
manager: scottpolly
ms.service: azure-ai-foundry
ms.custom:
  - build-2025
  - references_regions
ms.topic: how-to
ms.date: 07/14/2025
ms.reviewer: changliu2
ms.author: lagayhar
author: lgayhardt
---
# Evaluate your AI agents locally with Azure AI Evaluation SDK (preview)

[!INCLUDE [feature-preview](../../includes/feature-preview.md)]

AI Agents are powerful productivity assistants to create workflows for business needs. However, they come with challenges for observability due to their complex interaction patterns. In this article, you learn how to run built-in evaluators locally on simple agent data or agent messages.

To build production-ready agentic applications and enable observability and transparency, developers need tools to assess not just the final output from an agent's workflows, but the quality and efficiency of the workflows themselves. For example, consider a typical agentic workflow:

:::image type="content" source="../../media/evaluations/agent-workflow-evaluation.gif" alt-text="Animation of the agent's workflow from user query to intent resolution to tool calls to final response." lightbox="../../media/evaluations/agent-workflow-evaluation.gif":::

An event like a user query "weather tomorrow" triggers an agentic workflow. It starts to execute multiple steps, such as reasoning through user intents, tool calling, and utilizing retrieval-augmented generation to produce a final response. In this process, evaluating each step of the workflow—along with the quality and safety of the final output—is crucial. Specifically, we formulate these evaluation aspects into the following evaluators for agents:

- [Intent resolution](https://aka.ms/intentresolution-sample): Measures whether the agent correctly identifies the user's intent.
-	[Tool call accuracy](https://aka.ms/toolcallaccuracy-sample): Measures whether the agent made the correct function tool calls to a user's request.
-	[Task adherence](https://aka.ms/taskadherence-sample): Measures whether the agent's final response adheres to its assigned tasks, according to its system message and prior steps.

You can also assess other quality and safety aspects of your agentic workflows, using our comprehensive suite of built-in evaluators. In general, agents emit agent messages. Transforming agent messages into the right evaluation data to use our evaluators can be a nontrivial task. If you build your agent using [Foundry Agent Service](../../../ai-services/agents/overview.md), you can [seamlessly evaluate it via our converter support](#evaluate-azure-ai-agents). If you build your agent outside of Foundry Agent Service, you can still use our evaluators as appropriate to your agentic workflow, by parsing your agent messages into the [required data formats](./evaluate-sdk.md#data-requirements-for-built-in-evaluators). See examples in [evaluating other agents](#evaluating-other-agents). 

## Getting started

First install the evaluators package from Azure AI evaluation SDK:

```python
pip install azure-ai-evaluation
```

## Evaluate Azure AI agents
If you use [Foundry Agent Service](../../../ai-services/agents/overview.md), you can seamlessly evaluate your agents via our converter support for Azure AI agent threads and runs. We support this list of evaluators for Azure AI agent messages from our converter: 

### Evaluators supported for evaluation data converter
- Quality: `IntentResolution`, `ToolCallAccuracy`, `TaskAdherence`, `Relevance`, `Coherence`, `Fluency`
- Safety: `CodeVulnerabilities`, `Violence`, `Self-harm`, `Sexual`, `HateUnfairness`, `IndirectAttack`, `ProtectedMaterials`.


> [!NOTE]
> `ToolCallAccuracyEvaluator` only supports Foundry Agent's Function Tool evaluation (user-defined python functions), but doesn't support other Tool evaluation. If an agent run invoked a tool other than Function Tool, it will output a "pass" and a reason that evaluating the invoked tool(s) is not supported. 


Here's an example to seamlessly build and evaluate an Azure AI agent. Separately from evaluation, Azure AI Foundry Agent Service requires `pip install azure-ai-projects azure-identity` and an Azure AI project connection string and the supported models.


### Create agent threads and runs


First define the custom tools you intend the agent to use (using a mock weather function as an example):

```python
from azure.ai.projects.models import FunctionTool, ToolSet
from typing import Set, Callable, Any

# Define some custom python function
def fetch_weather(location: str) -> str:
    """
    Fetches the weather information for the specified location.

    :param location (str): The location to fetch weather for.
    :return: Weather information as a JSON string.
    :rtype: str
    """
    # In a real-world scenario, you'd integrate with a weather API.
    # Here, we'll mock the response.
    mock_weather_data = {"Seattle": "Sunny, 25°C", "London": "Cloudy, 18°C", "Tokyo": "Rainy, 22°C"}
    weather = mock_weather_data.get(location, "Weather data not available for this location.")
    weather_json = json.dumps({"weather": weather})
    return weather_json

user_functions: Set[Callable[..., Any]] = {
    fetch_weather,
}

# Adding Tools to be used by Agent 
functions = FunctionTool(user_functions)

toolset = ToolSet()
toolset.add(functions)

AGENT_NAME = "Seattle Tourist Assistant"
```

If you are using [Azure AI Foundry (non-Hub) project](../create-projects.md?tabs=ai-foundry&pivots=fdp-project), create an agent with the toolset as follows:

> [!NOTE]
> If you are using a [Foundry Hub-based project](../create-projects.md?tabs=ai-foundry&pivots=hub-project) (which only supports lower versions of `azure-ai-projects<1.0.0b10 azure-ai-agents<1.0.0b10`), we strongly recommend migrating to [the latest Foundry Agent Service SDK python client library](../../agents/quickstart.md?pivots=programming-language-python-azure) with a [Foundry project set up for loggging batch evaluation results](../../how-to/develop/evaluate-sdk.md#prerequisite-set-up-steps-for-azure-ai-foundry-projects).

```python
import os
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from dotenv import load_dotenv

load_dotenv()

# Create an Azure AI Client from an endpoint, copied from your Azure AI Foundry project.
# You need to login to Azure subscription via Azure CLI and set the environment variables
# Azure AI Foundry project endpoint, example: AZURE_AI_PROJECT=https://your-account.services.ai.azure.com/api/projects/your-project
project_endpoint = os.environ["AZURE_AI_PROJECT"]  # Ensure the PROJECT_ENDPOINT environment variable is set

# Create an AIProjectClient instance
project_client = AIProjectClient(
    endpoint=project_endpoint,
    credential=DefaultAzureCredential(),  # Use Azure Default Credential for authentication
)


# Create an agent with the 
agent = project_client.agents.create_agent(
    model=os.environ["MODEL_DEPLOYMENT_NAME"],  # Model deployment name
    name="my-agent",  # Name of the agent
    instructions="You are a helpful agent",  # Instructions for the agent
    toolset=toolset
)
print(f"Created agent, ID: {agent.id}")

# Create a thread for communication
thread = project_client.agents.threads.create()
print(f"Created thread, ID: {thread.id}")

# Add a message to the thread
message = project_client.agents.messages.create(
    thread_id=thread.id,
    role="user",  # Role of the message sender
    content="What is the weather in Seattle today?",  # Message content
)
print(f"Created message, ID: {message['id']}")

# Create and process an agent run
run = project_client.agents.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
print(f"Run finished with status: {run.status}")

# Check if the run failed
if run.status == "failed":
    print(f"Run failed: {run.last_error}")

# Fetch and log all messages
messages = project_client.agents.messages.list(thread_id=thread.id)
for message in messages:
    print(f"Role: {message.role}, Content: {message.content}")
    
```




### Evaluate a single agent run

With agent runs created, you can easily use our converter to transform the Azure AI agent thread data into required evaluation data that the evaluators can understand. 
```python
import json, os
from azure.ai.evaluation import AIAgentConverter, IntentResolutionEvaluator

# Initialize the converter for Azure AI agents
converter = AIAgentConverter(project_client)

# Specify the thread and run id
thread_id = thread.id
run_id = run.id

converted_data = converter.convert(thread_id, run_id)
```

And that's it! `converted_data` will contain all inputs required for [these evaluators](#evaluators-supported-for-evaluation-data-converter). You don't need to read the input requirements for each evaluator and do any work to parse the inputs. All you need to do is select your evaluator and call the evaluator on this single run. We support AzureOpenAI or OpenAI [reasoning models](../../../ai-services/openai/how-to/reasoning.md) and non-reasoning models for the judge depending on the evaluators:

| Evaluators | Reasoning Models as Judge (ex: o-series models from Azure OpenAI / OpenAI) | Non-reasoning models as Judge (ex: gpt-4.1, gpt-4o, etc.) | To enable |
|------------|-----------------------------------------------------------------------------|-------------------------------------------------------------|-------|
| `Intent Resolution` / `Task Adherence` / `Tool Call Accuracy` / `Response Completeness`) | Supported | Supported | Set additional parameter `is_reasoning_model=True` in initializing evaluators |
| Other quality evaluators| Not Supported | Supported | -- |

For complex tasks that requires refined reasoning for the evaluation, we recommend a strong reasoning model like `o3-mini` or the o-series mini models released afterwards with a balance of reasoning performance and cost efficiency.

We set up a list of quality and safety evaluator in `quality_evaluators` and `safety_evaluators` and reference them in [evaluating multiples agent runs or a thread](#evaluate-multiple-agent-runs-or-threads).

```python
# specific to agentic workflows
from azure.ai.evaluation import IntentResolutionEvaluator, TaskAdherenceEvaluator, ToolCallAccuracyEvaluator 
# other quality as well as risk and safety metrics
from azure.ai.evaluation import RelevanceEvaluator, CoherenceEvaluator, CodeVulnerabilityEvaluator, ContentSafetyEvaluator, IndirectAttackEvaluator, FluencyEvaluator
from azure.identity import DefaultAzureCredential

import os
from dotenv import load_dotenv
load_dotenv()

model_config = {
    "azure_deployment": os.getenv("AZURE_DEPLOYMENT_NAME"),
    "api_key": os.getenv("AZURE_API_KEY"),
    "azure_endpoint": os.getenv("AZURE_ENDPOINT"),
    "api_version": os.getenv("AZURE_API_VERSION"),
}

reasoning_model_config = {
    "azure_deployment": "o3-mini",
    "api_key": os.getenv("AZURE_API_KEY"),
    "azure_endpoint": os.getenv("AZURE_ENDPOINT"),
    "api_version": os.getenv("AZURE_API_VERSION"),
}

# evaluators with reasoning model support
quality_evaluators = {evaluator.__name__: evaluator(model_config=reasoning_model_config, is_reasoning_model=True) for evaluator in [IntentResolutionEvaluator, TaskAdherenceEvaluator, ToolCallAccuracyEvaluator]}

# other evaluators do not support reasoning models 
quality_evaluators.update({ evaluator.__name__: evaluator(model_config=model_config) for evaluator in [CoherenceEvaluator, FluencyEvaluator, RelevanceEvaluator]})

## Using Azure AI Foundry (non-Hub) project endpoint, example: AZURE_AI_PROJECT=https://your-account.services.ai.azure.com/api/projects/your-project
azure_ai_project = os.environ.get("AZURE_AI_PROJECT")

safety_evaluators = {evaluator.__name__: evaluator(azure_ai_project=azure_ai_project, credential=DefaultAzureCredential()) for evaluator in[ContentSafetyEvaluator, IndirectAttackEvaluator, CodeVulnerabilityEvaluator]}

# reference the quality and safety evaluator list above
quality_and_safety_evaluators = {**quality_evaluators, **safety_evaluators}

for name, evaluator in quality_and_safety_evaluators.items():
    result = evaluator(**converted_data)
    print(name)
    print(json.dumps(result, indent=4)) 


```

#### Output format

The result of the AI-assisted quality evaluators for a query and response pair is a dictionary containing:

- `{metric_name}` provides a numerical score, on a likert scale (integer 1 to 5) or a float between 0-1.
- `{metric_name}_label` provides a binary label (if the metric outputs a binary score naturally).
- `{metric_name}_reason` explains why a certain score or label was given for each data point.

To further improve intelligibility, all evaluators accept a binary threshold (unless they output already binary outputs) and output two new keys. For the binarization threshold, a default is set and user can override it. The two new keys are:

- `{metric_name}_result` a "pass" or "fail" string based on a binarization threshold.
- `{metric_name}_threshold` a numerical binarization threshold set by default or by the user.
- `additional_details` contains debugging information about the quality of a single agent run. 

Example output for some evaluators: 

```
{
    "intent_resolution": 5.0, # likert scale: 1-5 integer 
    "intent_resolution_result": "pass", # pass because 5 > 3 the threshold
    "intent_resolution_threshold": 3,
    "intent_resolution_reason": "The assistant correctly understood the user's request to fetch the weather in Seattle. It used the appropriate tool to get the weather information and provided a clear and accurate response with the current weather conditions in Seattle. The response fully resolves the user's query with all necessary information."
}
{
    "task_adherence": 5.0, # likert scale: 1-5 integer 
    "task_adherence_result": "pass", # pass because 5 > 3 the threshold
    "task_adherence_threshold": 3,
    "task_adherence_reason": "The response accurately follows the instructions, fetches the correct weather information, and relays it back to the user without any errors or omissions."
}
{
    "tool_call_accuracy": 5,  # a score bewteen 1-5, higher is better
    "tool_call_accuracy_result": "pass", # pass because 1.0 > 0.8 the threshold
    "tool_call_accuracy_threshold": 3,
    "details": { ... } # helpful details for debugging the tool calls made by the agent
}
```


### Evaluate multiple agent runs or threads

To evaluate multiple agent runs or threads, we recommend using the batch `evaluate()` API for async evaluation. First, convert your agent thread data into a file via our converter support:

```python
import json
from azure.ai.evaluation import AIAgentConverter

# Initialize the converter
converter = AIAgentConverter(project_client)

# Specify a file path to save agent output (which is evaluation input data)
filename = os.path.join(os.getcwd(), "evaluation_input_data.jsonl")

evaluation_data = converter.prepare_evaluation_data(thread_ids=thread_id, filename=filename) 

print(f"Evaluation data saved to {filename}")
```

With the evaluation data prepared in one line of code, you can select the evaluators to assess the agent quality and submit a batch evaluation run. Here, we reference the same list of quality and safety evaluators in section [evaluate a single agent run](#evaluate-a-single-agent-run) `quality_and_safety_evaluators`:  

```python
import os
from dotenv import load_dotenv
load_dotenv()


# Batch evaluation API (local)
from azure.ai.evaluation import evaluate

response = evaluate(
    data=filename,
    evaluation_name="agent demo - batch run",
    evaluators=quality_and_safety_evaluators,
    # optionally, log your results to your Azure AI Foundry project for rich visualization 
    azure_ai_project=os.environ.get("AZURE_AI_PROJECT"),  # example: https://your-account.services.ai.azure.com/api/projects/your-project
)
# Inspect the average scores at a high-level
print(response["metrics"])
# Use the URL to inspect the results on the UI
print(f'AI Foundary URL: {response.get("studio_url")}')
```

Following the URI, you'll be redirected to Foundry to view your evaluation results in your Azure AI project and debug your application. Using reason fields and pass/fail, you are able to easily assess the quality and safety performance of your applications. You can run and compare multiple runs to test for regression or improvements.  

With Azure AI Evaluation SDK client library, you can seamlessly evaluate your Azure AI agents via our converter support, which enables observability and transparency into agentic workflows.


## Evaluating other agents

For agents outside of Azure AI Foundry Agent Service, you can still evaluate them by preparing the right data for the evaluators of your choice.

Agents typically emit messages to interact with a user or other agents. Our built-in evaluators can accept simple data types such as strings in `query`, `response`, `ground_truth` according to the [single-turn data input requirements](./evaluate-sdk.md#data-requirements-for-built-in-evaluators). However, to extract these simple data from agent messages can be a challenge, due to the complex interaction patterns of agents and framework differences. For example, as mentioned, a single user query can trigger a long list of agent messages, typically with multiple tool calls invoked.

As illustrated in the example, we enabled agent message support specifically for these built-in evaluators `IntentResolution`, `ToolCallAccuracy`, `TaskAdherence` to evaluate these aspects of agentic workflow. These evaluators take `tool_calls` or `tool_definitions` as parameters unique to agents.

| Evaluator       | `query`      | `response`      | `tool_calls`       | `tool_definitions`  | 
|----------------|---------------|---------------|---------------|---------------|
| `IntentResolutionEvaluator`   | Required: `Union[str, list[Message]]` | Required: `Union[str, list[Message]]`  | N/A | Optional: `list[ToolCall]`  |
| `ToolCallAccuracyEvaluator`   | Required: `Union[str, list[Message]]` | Optional: `Union[str, list[Message]]`  | Optional: `Union[dict, list[ToolCall]]` | Required: `list[ToolDefinition]`  |
| `TaskAdherenceEvaluator`         | Required: `Union[str, list[Message]]` | Required: `Union[str, list[Message]]`  | N/A | Optional: `list[ToolCall]`  |

- `Message`: `dict` openai-style message describing agent interactions with a user, where `query` must include a system message as the first message.
- `ToolCall`: `dict` specifying tool calls invoked during agent interactions with a user.
- `ToolDefinition`: `dict` describing the tools available to an agent.

For `ToolCallAccuracyEvaluator`, either `response` or  `tool_calls` must be provided. 

We'll demonstrate some examples of the two data formats: simple agent data, and agent messages. However, due to the unique requirements of these evaluators, we recommend referring to the [sample notebooks](#sample-notebooks) which illustrate the possible input paths for each evaluator.  

As with other [built-in AI-assisted quality evaluators](../../concepts/evaluation-evaluators/agent-evaluators.md), `IntentResolutionEvaluator` and `TaskAdherenceEvaluator` output a likert score (integer 1-5; higher score is better). `ToolCallAccuracyEvaluator` outputs the passing rate of all tool calls made (a float between 0-1) based on user query. To further improve intelligibility, all evaluators accept a binary threshold and output two new keys. For the binarization threshold, a default is set and user can override it. The two new keys are:

- `{metric_name}_result` a "pass" or "fail" string based on a binarization threshold.
- `{metric_name}_threshold` a numerical binarization threshold set by default or by the user.

### Simple agent data

In simple agent data format, `query` and `response` are simple python strings. For example:  

```python
import os
import json
from azure.ai.evaluation import AzureOpenAIModelConfiguration
from azure.identity import DefaultAzureCredential
from azure.ai.evaluation import IntentResolutionEvaluator, ResponseCompletenessEvaluator
  
model_config = AzureOpenAIModelConfiguration(
    azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
    api_key=os.environ["AZURE_OPENAI_API_KEY"],
    api_version=os.environ["AZURE_OPENAI_API_VERSION"],
    azure_deployment=os.environ["MODEL_DEPLOYMENT_NAME"],
)
 
intent_resolution_evaluator = IntentResolutionEvaluator(model_config)

# Evaluating query and response as strings
# A positive example. Intent is identified and understood and the response correctly resolves user intent
result = intent_resolution_evaluator(
    query="What are the opening hours of the Eiffel Tower?",
    response="Opening hours of the Eiffel Tower are 9:00 AM to 11:00 PM.",
)
print(json.dumps(result, indent=4))
 
```

Output (see [output format](#output-format) for details): 

```
{
    "intent_resolution": 5.0,
    "intent_resolution_result": "pass",
    "intent_resolution_threshold": 3,
    "intent_resolution_reason": "The response provides the opening hours of the Eiffel Tower, which directly addresses the user's query. The information is clear, accurate, and complete, fully resolving the user's intent.",
    "additional_details": {
        "conversation_has_intent": true,
        "agent_perceived_intent": "inquire about the opening hours of the Eiffel Tower",
        "actual_user_intent": "inquire about the opening hours of the Eiffel Tower",
        "correct_intent_detected": true,
        "intent_resolved": true
    }
}
```
Examples of `tool_calls` and `tool_definitions` for `ToolCallAccuracyEvaluator`: 

```python
import json 

query = "How is the weather in Seattle?"
tool_calls = [{
                    "type": "tool_call",
                    "tool_call_id": "call_CUdbkBfvVBla2YP3p24uhElJ",
                    "name": "fetch_weather",
                    "arguments": {
                        "location": "Seattle"
                    }
            },
            {
                    "type": "tool_call",
                    "tool_call_id": "call_CUdbkBfvVBla2YP3p24uhElJ",
                    "name": "fetch_weather",
                    "arguments": {
                        "location": "London"
                    }
            }]

tool_definitions = [{
                    "name": "fetch_weather",
                    "description": "Fetches the weather information for the specified location.",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "location": {
                                "type": "string",
                                "description": "The location to fetch weather for."
                            }
                        }
                    }
                }]
response = tool_call_accuracy(query=query, tool_calls=tool_calls, tool_definitions=tool_definitions)
print(json.dumps(response, indent=4))
```
Output (see [output format](#output-format) for details): 

```
{
    "tool_call_accuracy": 3,  # a score bewteen 1-5, higher is better
    "tool_call_accuracy_result": "fail",
    "tool_call_accuracy_threshold": 4,
    "details": { ... } # helpful details for debugging the tool calls made by the agent
}
```

### Agent messages

In agent message format, `query` and `response` are list of openai-style messages. Specifically, `query` carry the past agent-user interactions leading up to the last user query and requires the system message (of the agent) on top of the list; and `response` will carry the last message of the agent in response to the last user query. Example:

```python
import json

# user asked a question
query = [
    {
        "role": "system",
        "content": "You are a friendly and helpful customer service agent."
    },
    # past interactions omitted 
    # ...
    {
        "createdAt": "2025-03-14T06:14:20Z",
        "role": "user",
        "content": [
            {
                "type": "text",
                "text": "Hi, I need help with the last 2 orders on my account #888. Could you please update me on their status?"
            }
        ]
    }
]
# the agent emits multiple messages to fulfill the request
response = [
    {
        "createdAt": "2025-03-14T06:14:30Z",
        "run_id": "0",
        "role": "assistant",
        "content": [
            {
                "type": "text",
                "text": "Hello! Let me quickly look up your account details."
            }
        ]
    },
    {
        "createdAt": "2025-03-14T06:14:35Z",
        "run_id": "0",
        "role": "assistant",
        "content": [
            {
                "type": "tool_call",
                "tool_call_id": "tool_call_20250310_001",
                "name": "get_orders",
                "arguments": {
                    "account_number": "888"
                }
            }
        ]
    },
    # many more messages omitted 
    # ...
    # here is the agent's final response 
    {
        "createdAt": "2025-03-14T06:15:05Z",
        "run_id": "0",
        "role": "assistant",
        "content": [
            {
                "type": "text",
                "text": "The order with ID 123 has been shipped and is expected to be delivered on March 15, 2025. However, the order with ID 124 is delayed and should now arrive by March 20, 2025. Is there anything else I can help you with?"
            }
        ]
    }
]

# An example of tool definitions available to the agent 
tool_definitions = [
    {
        "name": "get_orders",
        "description": "Get the list of orders for a given account number.",
        "parameters": {
            "type": "object",
            "properties": {
                "account_number": {
                    "type": "string",
                    "description": "The account number to get the orders for."
                }
            }
        }
    },
    # other tool definitions omitted 
    # ...
]

result = intent_resolution_evaluator(
    query=query,
    response=response,
    # optionally provide the tool definitions
    tool_definitions=tool_definitions 
)
print(json.dumps(result, indent=4))

```

Output (see [output format](#output-format) for details): 

```
{
    "tool_call_accuracy": 2,  # a score bewteen 1-5, higher is better
    "tool_call_accuracy_result": "fail",
    "tool_call_accuracy_threshold": 3,
    "details": { ... } # helpful details for debugging the tool calls made by the agent
}
```
This evaluation schema helps you parse your agent data outside of Azure AI Foundry Agent Service, so that you can use our evaluators to support observability into your agentic workflows.   

## Sample notebooks

Now you're ready to try a sample for each of these evaluators:
- [Intent resolution](https://aka.ms/intentresolution-sample)
- [Tool call accuracy](https://aka.ms/toolcallaccuracy-sample)
- [Task adherence](https://aka.ms/taskadherence-sample)
- [Response Completeness](https://aka.ms/rescompleteness-sample)
- [End-to-end Azure AI agent evaluation](https://aka.ms/e2e-agent-eval-sample)

## Related content

- [Azure AI Evaluation Python SDK client reference documentation](https://aka.ms/azureaieval-python-ref)
- [Azure AI Evaluation SDK client Troubleshooting guide](https://aka.ms/azureaieval-tsg)
- [Learn more about the evaluation metrics](../../concepts/evaluation-metrics-built-in.md)
- [Evaluate your Generative AI applications remotely on the cloud](./cloud-evaluation.md)
- [Learn more about simulating test datasets for evaluation](./simulator-interaction-data.md)
- [View your evaluation results in Azure AI project](../../how-to/evaluate-results.md)
- [Get started building a chat app using the Azure AI Foundry SDK](../../quickstarts/get-started-code.md)
- [Get started with evaluation samples](https://aka.ms/aistudio/eval-samples)

# Getting Started with A2A

This tutorial will guide you through the basics of the A2A protocol and help you understand how to use it for agent communication.

## Objective

By the end of this tutorial, you will:
- Understand the basic concepts of A2A
- Know how to structure A2A messages
- Be able to send and receive simple messages between agents

## Prerequisites

- Basic understanding of APIs and web services
- Familiarity with JSON
- Python 3.7+ installed (for the example code)

## What is A2A?

A2A (Agent-to-Agent) is a protocol that enables standardized communication between AI agents. It provides a framework for agents to exchange information, make requests, and collaborate on tasks.

## Basic Concepts

### Agents

An agent is an autonomous AI system that can perform tasks and communicate with other agents. Each agent has:
- A unique identifier
- A set of capabilities (functions it can perform)
- The ability to send and receive messages

### Messages

Messages are the primary means of communication between agents. A basic A2A message includes:

```json
{
  "message_id": "msg_123456789",
  "sender": {
    "agent_id": "agent_abc",
    "name": "Weather Agent"
  },
  "recipient": {
    "agent_id": "agent_xyz",
    "name": "Planning Agent"
  },
  "intent": "provide_information",
  "content": {
    "temperature": 72,
    "conditions": "sunny",
    "location": "San Francisco"
  },
  "timestamp": "2023-06-15T14:30:00Z"
}
```

### Intents

Intents describe the purpose of a message. Common intents include:
- `request_information`: Ask for data
- `provide_information`: Share data
- `request_action`: Ask the recipient to do something
- `action_result`: Report the outcome of an action

## Example: Creating a Simple Agent

Here's a basic example of creating an agent in Python:

```python
import uuid
import json
import datetime

class SimpleAgent:
    def __init__(self, agent_id, name, capabilities):
        self.agent_id = agent_id
        self.name = name
        self.capabilities = capabilities
        self.messages = []
    
    def create_message(self, recipient, intent, content):
        message = {
            "message_id": f"msg_{uuid.uuid4().hex}",
            "sender": {
                "agent_id": self.agent_id,
                "name": self.name
            },
            "recipient": recipient,
            "intent": intent,
            "content": content,
            "timestamp": datetime.datetime.utcnow().isoformat() + "Z"
        }
        return message
    
    def send_message(self, recipient, intent, content):
        message = self.create_message(recipient, intent, content)
        # In a real implementation, this would send the message over a network
        print(f"Sending message: {json.dumps(message, indent=2)}")
        return message
    
    def receive_message(self, message):
        # In a real implementation, this would validate the message
        self.messages.append(message)
        print(f"Received message from {message['sender']['name']}")
        # Process the message based on intent
        if message["intent"] == "request_information":
            self.handle_information_request(message)
    
    def handle_information_request(self, message):
        # Example handler for information requests
        print(f"Processing information request about: {message['content']}")
        # Generate a response based on capabilities
        # ...

# Example usage
weather_agent = SimpleAgent(
    agent_id="agent_weather_1",
    name="Weather Service",
    capabilities=["provide_weather", "provide_forecast"]
)

planning_agent = SimpleAgent(
    agent_id="agent_planner_1",
    name="Trip Planner",
    capabilities=["plan_trip", "suggest_activities"]
)

# Planning agent requests weather information
request = planning_agent.send_message(
    recipient={
        "agent_id": weather_agent.agent_id,
        "name": weather_agent.name
    },
    intent="request_information",
    content={
        "type": "weather",
        "location": "San Francisco",
        "date": "2023-06-15"
    }
)

# Weather agent receives the request
weather_agent.receive_message(request)
```

## Next Steps

Now that you understand the basics of A2A, you can:
1. Explore the [A2A Architecture](../architecture/README.md) to learn more about the protocol
2. Try the [Creating Your First Agent](./first-agent.md) tutorial for a more comprehensive example
3. Check out the [Examples](../examples/README.md) for practical implementations

## Additional Resources

- [A2A Specification](https://github.com/google/A2A/tree/main/specification)
- [A2A GitHub Repository](https://github.com/google/A2A)

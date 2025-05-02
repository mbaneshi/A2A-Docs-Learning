# A2A Message Format

This document describes the standard message format used in the A2A protocol.

## Overview

A2A messages are JSON objects with a defined structure that enables standardized communication between agents. The message format is designed to be:

- **Flexible**: Accommodating various types of content
- **Extensible**: Allowing for future additions without breaking compatibility
- **Self-describing**: Including metadata that helps with message processing

## Message Structure

A standard A2A message has the following structure:

```json
{
  "message_id": "string",
  "conversation_id": "string",
  "sender": {
    "agent_id": "string",
    "name": "string",
    "version": "string"
  },
  "recipient": {
    "agent_id": "string",
    "name": "string"
  },
  "intent": "string",
  "content": {
    // Arbitrary JSON object
  },
  "metadata": {
    "timestamp": "string (ISO 8601)",
    "ttl": "number (seconds)",
    "priority": "string (high|medium|low)",
    "security": {
      "encryption": "string",
      "signature": "string"
    },
    "routing": {
      "path": ["string"],
      "max_hops": "number"
    }
  }
}
```

## Field Descriptions

### Required Fields

- **message_id**: A unique identifier for the message (UUID v4 recommended)
- **sender**: Information about the sending agent
  - **agent_id**: Unique identifier for the sending agent
  - **name**: Human-readable name of the sending agent
- **recipient**: Information about the receiving agent
  - **agent_id**: Unique identifier for the receiving agent
  - **name**: Human-readable name of the receiving agent
- **intent**: The purpose of the message (e.g., "request_information", "provide_information")
- **content**: The actual payload of the message (structure depends on the intent)

### Optional Fields

- **conversation_id**: Identifier for a sequence of related messages
- **sender.version**: Version of the sending agent
- **metadata**: Additional information about the message
  - **timestamp**: When the message was created (ISO 8601 format)
  - **ttl**: Time-to-live in seconds
  - **priority**: Message priority (high, medium, low)
  - **security**: Security-related information
    - **encryption**: Encryption method used
    - **signature**: Digital signature for verification
  - **routing**: Information for message routing
    - **path**: List of agents the message has passed through
    - **max_hops**: Maximum number of routing hops allowed

## Common Intents

A2A defines several standard intents:

| Intent | Description | Expected Response |
|--------|-------------|-------------------|
| `request_information` | Request data from another agent | `provide_information` |
| `provide_information` | Share data with another agent | None (or acknowledgment) |
| `request_action` | Ask another agent to perform an action | `action_result` |
| `action_result` | Report the outcome of an action | None (or acknowledgment) |
| `error` | Report an error condition | Depends on the error |
| `acknowledge` | Acknowledge receipt of a message | None |
| `capability_query` | Ask about an agent's capabilities | `capability_response` |
| `capability_response` | Describe an agent's capabilities | None |

## Content Structure

The structure of the `content` field depends on the `intent`:

### request_information

```json
{
  "query_type": "string",
  "parameters": {
    // Query parameters
  },
  "response_format": "string (optional)"
}
```

### provide_information

```json
{
  "information_type": "string",
  "data": {
    // The requested information
  },
  "source": "string (optional)",
  "confidence": "number (optional, 0-1)"
}
```

### request_action

```json
{
  "action": "string",
  "parameters": {
    // Action parameters
  },
  "callback": {
    // Optional callback information
  }
}
```

### action_result

```json
{
  "action": "string",
  "status": "string (success|failure|partial)",
  "result": {
    // Result data
  },
  "error": {
    // Error information (if status is failure)
  }
}
```

## Examples

### Weather Information Request

```json
{
  "message_id": "msg_1234567890abcdef",
  "conversation_id": "conv_abcdef1234567890",
  "sender": {
    "agent_id": "agent_planner_1",
    "name": "Trip Planner",
    "version": "1.0.0"
  },
  "recipient": {
    "agent_id": "agent_weather_1",
    "name": "Weather Service"
  },
  "intent": "request_information",
  "content": {
    "query_type": "weather",
    "parameters": {
      "location": "San Francisco",
      "date": "2023-06-15",
      "metrics": ["temperature", "precipitation", "wind"]
    },
    "response_format": "json"
  },
  "metadata": {
    "timestamp": "2023-06-14T18:30:00Z",
    "priority": "medium"
  }
}
```

### Weather Information Response

```json
{
  "message_id": "msg_abcdef1234567890",
  "conversation_id": "conv_abcdef1234567890",
  "sender": {
    "agent_id": "agent_weather_1",
    "name": "Weather Service",
    "version": "2.1.0"
  },
  "recipient": {
    "agent_id": "agent_planner_1",
    "name": "Trip Planner"
  },
  "intent": "provide_information",
  "content": {
    "information_type": "weather",
    "data": {
      "location": "San Francisco",
      "date": "2023-06-15",
      "temperature": {
        "high": 72,
        "low": 58,
        "unit": "F"
      },
      "precipitation": {
        "chance": 0.1,
        "type": "none"
      },
      "wind": {
        "speed": 8,
        "direction": "W",
        "unit": "mph"
      }
    },
    "source": "National Weather Service",
    "confidence": 0.95
  },
  "metadata": {
    "timestamp": "2023-06-14T18:30:05Z",
    "priority": "medium"
  }
}
```

## Validation

A2A provides JSON Schema definitions for validating messages. These schemas can be found in the [A2A GitHub repository](https://github.com/google/A2A/tree/main/specification/json).

## Extensions

The A2A message format can be extended with additional fields as needed. Extensions should follow these guidelines:

1. Use namespaced field names to avoid conflicts (e.g., `x-mycompany-field`)
2. Place extensions in the `metadata` object when possible
3. Document extensions clearly
4. Make extensions optional to maintain compatibility

## Further Reading

- [A2A Authentication](./authentication.md)
- [A2A Error Handling](./error-handling.md)
- [Implementation Examples](../examples/README.md)

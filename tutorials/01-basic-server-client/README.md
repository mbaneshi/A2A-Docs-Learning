# Tutorial 1: Building a Basic A2A Server and Client

This tutorial will guide you through creating a minimal A2A server and client implementation. By the end, you'll have a working A2A system that can exchange simple text messages.

## Objective

In this tutorial, you will:
- Understand the core components of the A2A protocol
- Implement a basic A2A server that can handle task requests
- Create a client that can send tasks and process responses
- Test the communication between client and server

## Prerequisites

- Python 3.12+ or Node.js 18+ (depending on which implementation you choose)
- Basic understanding of HTTP and REST APIs
- Familiarity with async programming concepts
- Development environment with required dependencies installed

## Concepts Covered

- **A2A Protocol Basics**: Understanding the core protocol structure
- **Agent Card**: Creating and exposing agent metadata
- **Task Management**: Handling task creation and state transitions
- **JSON-RPC**: Working with the JSON-RPC 2.0 message format
- **Request/Response Flow**: Understanding the basic communication pattern

## Implementation

This tutorial provides implementations in both Python and JavaScript. Choose the one that best fits your needs:

- [Python Implementation](./python/README.md)
- [JavaScript Implementation](./javascript/README.md)

Both implementations cover the same concepts and result in functionally equivalent systems.

## System Architecture

The system we'll build consists of two main components:

1. **A2A Server**:
   - Exposes an HTTP endpoint for receiving A2A requests
   - Publishes an Agent Card at a well-known URL
   - Processes tasks and manages their state
   - Generates responses based on task input

2. **A2A Client**:
   - Discovers the server's capabilities via its Agent Card
   - Creates and sends tasks to the server
   - Processes responses from the server
   - Manages the client-side state of tasks

Here's a high-level diagram of the system:

```
┌─────────────┐                 ┌─────────────┐
│             │  1. Agent Card  │             │
│             │◄────────────────┤             │
│   Client    │                 │   Server    │
│             │  2. Task Send   │             │
│             ├────────────────►│             │
│             │  3. Response    │             │
│             │◄────────────────┤             │
└─────────────┘                 └─────────────┘
```

## Communication Flow

1. **Discovery**: Client fetches the Agent Card from the server's well-known URL
2. **Task Creation**: Client creates a new task with a unique ID and initial message
3. **Task Submission**: Client sends the task to the server via a `tasks/send` request
4. **Processing**: Server processes the task and updates its state
5. **Response**: Server sends back the updated task with a response message
6. **Completion**: Task reaches a terminal state (completed, failed, canceled)

## Testing

After implementing both the server and client, you'll test them by:
1. Starting the server
2. Running the client to send a simple text message
3. Verifying that the server processes the message correctly
4. Checking that the client receives and displays the response

## Common Issues

- **CORS Issues**: If testing in a browser, you may encounter CORS errors
- **JSON-RPC Format**: Ensure your requests follow the JSON-RPC 2.0 specification
- **Task IDs**: Make sure each task has a unique ID
- **Content Types**: Set the correct Content-Type headers for requests and responses

## Extensions

Once you have the basic implementation working, you can extend it by:
- Adding support for multiple messages in a conversation
- Implementing error handling for various failure scenarios
- Adding authentication to secure the communication
- Supporting different types of content beyond simple text

## Next Steps

After completing this tutorial, proceed to [Tutorial 2: Implementing Streaming Responses with SSE](../02-streaming-responses/README.md) to learn how to implement real-time updates using Server-Sent Events.

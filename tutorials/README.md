# A2A Protocol Tutorials

This directory contains step-by-step tutorials for implementing and using the A2A protocol in various scenarios. These tutorials are designed to provide hands-on experience with A2A, from basic concepts to advanced implementations.

## Available Tutorials

### Basic Tutorials

1. [Building a Basic A2A Server and Client](./01-basic-server-client/README.md)
   - Create a minimal A2A server
   - Implement a basic A2A client
   - Send and receive simple text messages
   - Understand the core request/response flow

2. [Implementing Streaming Responses with SSE](./02-streaming-responses/README.md)
   - Set up Server-Sent Events (SSE)
   - Implement streaming task updates
   - Handle streaming on the client side
   - Manage connection lifecycle

3. [Exchanging Files and Structured Data](./03-files-and-structured-data/README.md)
   - Send and receive file attachments
   - Work with structured data using DataPart
   - Handle different content types
   - Implement file processing workflows

4. [Setting Up Push Notifications](./04-push-notifications/README.md)
   - Configure push notification endpoints
   - Implement webhook receivers
   - Secure notification channels
   - Handle offline updates

5. [Integrating with Agent Frameworks](./05-framework-integration/README.md)
   - Connect A2A with Google ADK
   - Integrate with LangGraph
   - Use A2A with CrewAI
   - Implement adapters for different frameworks

## Tutorial Structure

Each tutorial follows a consistent structure:

1. **Objective**: What you'll learn and build
2. **Prerequisites**: Required knowledge and setup
3. **Concepts**: Key concepts and components covered
4. **Implementation**: Step-by-step instructions with code
5. **Testing**: How to test and verify your implementation
6. **Common Issues**: Troubleshooting and pitfalls to avoid
7. **Extensions**: Ways to extend or modify the implementation
8. **Next Steps**: Where to go from here

## Getting Started

To get the most out of these tutorials:

1. Start with the first tutorial and work through them in order
2. Complete all the exercises in each tutorial
3. Experiment with modifying the code to understand how it works
4. Refer to the [A2A Protocol Specification](https://github.com/google/A2A/tree/main/specification) for detailed reference

## Requirements

- Python 3.12+ or Node.js 18+ (depending on the tutorial)
- Basic understanding of HTTP and REST APIs
- Familiarity with async programming concepts
- Development environment with required dependencies installed

## Support

If you encounter issues while following these tutorials:

1. Check the "Common Issues" section in each tutorial
2. Refer to the [A2A GitHub repository](https://github.com/google/A2A) for the latest updates
3. Check the [GitHub Discussions](https://github.com/google/A2A/discussions) for community help

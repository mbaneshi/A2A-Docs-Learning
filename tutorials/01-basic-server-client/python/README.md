# Building a Basic A2A Server and Client in Python

This guide walks you through implementing a minimal A2A server and client in Python.

## Setup

1. Create a new directory for your project:
   ```bash
   mkdir a2a-basic
   cd a2a-basic
   ```

2. Create a virtual environment and activate it:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. Install the required dependencies:
   ```bash
   pip install starlette uvicorn pydantic httpx
   ```

4. Create the following file structure:
   ```
   a2a-basic/
   ├── server.py
   ├── client.py
   └── types.py
   ```

## Step 1: Define Common Types

First, let's define the common data types used by both the server and client. Create a file named `types.py`:

```python
from enum import Enum
from typing import Any, Dict, List, Optional, Union
from pydantic import BaseModel, Field


class TaskState(str, Enum):
    SUBMITTED = "submitted"
    WORKING = "working"
    INPUT_REQUIRED = "input-required"
    COMPLETED = "completed"
    FAILED = "failed"
    CANCELED = "canceled"


class TextPart(BaseModel):
    type: str = "text"
    text: str
    metadata: Dict[str, Any] = Field(default_factory=dict)


class Message(BaseModel):
    role: str  # "user" or "agent"
    parts: List[TextPart]
    metadata: Dict[str, Any] = Field(default_factory=dict)


class TaskStatus(BaseModel):
    state: TaskState
    message: Optional[Message] = None
    timestamp: Optional[str] = None


class Task(BaseModel):
    id: str
    sessionId: Optional[str] = None
    status: TaskStatus
    history: List[Message] = Field(default_factory=list)
    artifacts: List[Any] = Field(default_factory=list)
    metadata: Dict[str, Any] = Field(default_factory=dict)


class TaskSendParams(BaseModel):
    id: str
    sessionId: Optional[str] = None
    message: Message
    historyLength: Optional[int] = None
    metadata: Dict[str, Any] = Field(default_factory=dict)


class TaskIdParams(BaseModel):
    id: str


class TaskQueryParams(TaskIdParams):
    historyLength: Optional[int] = None


class JSONRPCRequest(BaseModel):
    jsonrpc: str = "2.0"
    id: Optional[Union[str, int]] = None
    method: str


class SendTaskRequest(JSONRPCRequest):
    method: str = "tasks/send"
    params: TaskSendParams


class GetTaskRequest(JSONRPCRequest):
    method: str = "tasks/get"
    params: TaskQueryParams


class JSONRPCError(BaseModel):
    code: int
    message: str
    data: Optional[Any] = None


class JSONRPCResponse(BaseModel):
    jsonrpc: str = "2.0"
    id: Optional[Union[str, int]] = None
    result: Optional[Any] = None
    error: Optional[JSONRPCError] = None


class SendTaskResponse(JSONRPCResponse):
    result: Optional[Task] = None


class GetTaskResponse(JSONRPCResponse):
    result: Optional[Task] = None


class AgentSkill(BaseModel):
    id: str
    name: str
    description: Optional[str] = None
    tags: List[str] = Field(default_factory=list)
    examples: Optional[List[str]] = None
    inputModes: Optional[List[str]] = None
    outputModes: Optional[List[str]] = None


class AgentCapabilities(BaseModel):
    streaming: bool = False
    pushNotifications: bool = False
    stateTransitionHistory: bool = False


class AgentAuthentication(BaseModel):
    schemes: List[str]
    credentials: Optional[str] = None


class AgentCard(BaseModel):
    name: str
    description: Optional[str] = None
    url: str
    version: str
    capabilities: AgentCapabilities
    authentication: Optional[AgentAuthentication] = None
    defaultInputModes: List[str] = Field(default=["text/plain"])
    defaultOutputModes: List[str] = Field(default=["text/plain"])
    skills: List[AgentSkill]
```

## Step 2: Implement the Server

Now, let's create a basic A2A server. Create a file named `server.py`:

```python
from starlette.applications import Starlette
from starlette.responses import JSONResponse
from starlette.requests import Request
import uvicorn
import json
import uuid
from datetime import datetime
import asyncio
from typing import Dict

from types import (
    AgentCard,
    AgentCapabilities,
    AgentSkill,
    AgentAuthentication,
    Task,
    TaskState,
    TaskStatus,
    Message,
    TextPart,
    SendTaskRequest,
    GetTaskRequest,
    SendTaskResponse,
    GetTaskResponse,
    JSONRPCError,
)


class SimpleA2AServer:
    def __init__(self, host="0.0.0.0", port=5000):
        self.host = host
        self.port = port
        self.tasks: Dict[str, Task] = {}
        
        # Create the Starlette app
        self.app = Starlette()
        
        # Add routes
        self.app.add_route("/.well-known/agent.json", self.get_agent_card, methods=["GET"])
        self.app.add_route("/", self.process_request, methods=["POST"])
    
    def get_agent_card(self, request: Request) -> JSONResponse:
        """Return the agent card describing this agent's capabilities."""
        agent_card = AgentCard(
            name="Simple Echo Agent",
            description="A simple agent that echoes back messages",
            url=f"http://{self.host}:{self.port}/",
            version="1.0.0",
            capabilities=AgentCapabilities(
                streaming=False,
                pushNotifications=False,
                stateTransitionHistory=False,
            ),
            authentication=AgentAuthentication(
                schemes=["none"]
            ),
            skills=[
                AgentSkill(
                    id="echo",
                    name="Echo",
                    description="Echoes back the message sent to it",
                    tags=["echo", "test"],
                    examples=["Hello, world!"]
                )
            ]
        )
        return JSONResponse(agent_card.model_dump(exclude_none=True))
    
    async def process_request(self, request: Request) -> JSONResponse:
        """Process incoming A2A requests."""
        try:
            # Parse the request body
            body = await request.json()
            
            # Determine the request type based on the method
            method = body.get("method")
            
            if method == "tasks/send":
                # Handle task send request
                req = SendTaskRequest.model_validate(body)
                return await self.handle_send_task(req)
            elif method == "tasks/get":
                # Handle task get request
                req = GetTaskRequest.model_validate(body)
                return await self.handle_get_task(req)
            else:
                # Unknown method
                error = JSONRPCError(
                    code=-32601,
                    message=f"Method not found: {method}"
                )
                return JSONResponse({"jsonrpc": "2.0", "id": body.get("id"), "error": error.model_dump()})
        
        except json.JSONDecodeError:
            # Invalid JSON
            error = JSONRPCError(
                code=-32700,
                message="Parse error: Invalid JSON"
            )
            return JSONResponse({"jsonrpc": "2.0", "error": error.model_dump()}, status_code=400)
        
        except Exception as e:
            # Internal error
            error = JSONRPCError(
                code=-32603,
                message=f"Internal error: {str(e)}"
            )
            return JSONResponse({"jsonrpc": "2.0", "id": body.get("id", None), "error": error.model_dump()}, status_code=500)
    
    async def handle_send_task(self, request: SendTaskRequest) -> JSONResponse:
        """Handle a tasks/send request."""
        task_id = request.params.id
        message = request.params.message
        
        # Check if the task already exists
        if task_id in self.tasks:
            # Existing task - update it
            task = self.tasks[task_id]
            task.history.append(message)
        else:
            # New task - create it
            task = Task(
                id=task_id,
                sessionId=request.params.sessionId or str(uuid.uuid4()),
                status=TaskStatus(state=TaskState.SUBMITTED),
                history=[message]
            )
            self.tasks[task_id] = task
        
        # Process the task (in a real implementation, this might be more complex)
        await self.process_task(task)
        
        # Create the response
        response = SendTaskResponse(
            id=request.id,
            result=task
        )
        
        return JSONResponse(response.model_dump(exclude_none=True))
    
    async def handle_get_task(self, request: GetTaskRequest) -> JSONResponse:
        """Handle a tasks/get request."""
        task_id = request.params.id
        
        # Check if the task exists
        if task_id not in self.tasks:
            error = JSONRPCError(
                code=-32000,
                message=f"Task not found: {task_id}"
            )
            return JSONResponse({"jsonrpc": "2.0", "id": request.id, "error": error.model_dump()})
        
        task = self.tasks[task_id]
        
        # Apply history length limit if specified
        if request.params.historyLength is not None:
            # Create a copy of the task with limited history
            task_copy = task.model_copy()
            history_length = request.params.historyLength
            if history_length > 0:
                task_copy.history = task.history[-history_length:]
            else:
                task_copy.history = []
            task = task_copy
        
        # Create the response
        response = GetTaskResponse(
            id=request.id,
            result=task
        )
        
        return JSONResponse(response.model_dump(exclude_none=True))
    
    async def process_task(self, task: Task) -> None:
        """Process a task and generate a response."""
        # Update task status to working
        task.status = TaskStatus(
            state=TaskState.WORKING,
            timestamp=datetime.utcnow().isoformat() + "Z"
        )
        
        # Simulate some processing time
        await asyncio.sleep(1)
        
        # Get the last message (from the user)
        last_message = task.history[-1]
        
        # Create a response message
        response_text = f"Echo: {' '.join([part.text for part in last_message.parts if part.type == 'text'])}"
        response_message = Message(
            role="agent",
            parts=[TextPart(type="text", text=response_text)]
        )
        
        # Add the response to the task history
        task.history.append(response_message)
        
        # Update task status to completed
        task.status = TaskStatus(
            state=TaskState.COMPLETED,
            message=response_message,
            timestamp=datetime.utcnow().isoformat() + "Z"
        )
    
    def start(self):
        """Start the server."""
        uvicorn.run(self.app, host=self.host, port=self.port)


if __name__ == "__main__":
    server = SimpleA2AServer()
    server.start()
```

## Step 3: Implement the Client

Now, let's create a basic A2A client. Create a file named `client.py`:

```python
import httpx
import json
import uuid
import asyncio
from typing import Dict, Any, Optional

from types import (
    AgentCard,
    Task,
    Message,
    TextPart,
    TaskSendParams,
    SendTaskRequest,
    SendTaskResponse,
    GetTaskRequest,
    GetTaskResponse,
    TaskQueryParams,
)


class SimpleA2AClient:
    def __init__(self, server_url: str):
        self.server_url = server_url
        self.agent_card: Optional[AgentCard] = None
    
    async def fetch_agent_card(self) -> AgentCard:
        """Fetch the agent card from the server."""
        async with httpx.AsyncClient() as client:
            response = await client.get(f"{self.server_url}/.well-known/agent.json")
            response.raise_for_status()
            card_data = response.json()
            self.agent_card = AgentCard.model_validate(card_data)
            return self.agent_card
    
    async def send_task(self, message_text: str, task_id: Optional[str] = None) -> Task:
        """Send a task to the server."""
        # Generate a task ID if not provided
        if task_id is None:
            task_id = str(uuid.uuid4())
        
        # Create the message
        message = Message(
            role="user",
            parts=[TextPart(type="text", text=message_text)]
        )
        
        # Create the task parameters
        params = TaskSendParams(
            id=task_id,
            message=message
        )
        
        # Create the request
        request = SendTaskRequest(
            id=str(uuid.uuid4()),
            params=params
        )
        
        # Send the request to the server
        async with httpx.AsyncClient() as client:
            response = await client.post(
                self.server_url,
                json=request.model_dump(exclude_none=True),
                headers={"Content-Type": "application/json"}
            )
            response.raise_for_status()
            response_data = response.json()
            
            # Parse the response
            task_response = SendTaskResponse.model_validate(response_data)
            
            if task_response.error:
                raise Exception(f"Error from server: {task_response.error.message}")
            
            return task_response.result
    
    async def get_task(self, task_id: str, history_length: Optional[int] = None) -> Task:
        """Get a task from the server."""
        # Create the request parameters
        params = TaskQueryParams(
            id=task_id,
            historyLength=history_length
        )
        
        # Create the request
        request = GetTaskRequest(
            id=str(uuid.uuid4()),
            params=params
        )
        
        # Send the request to the server
        async with httpx.AsyncClient() as client:
            response = await client.post(
                self.server_url,
                json=request.model_dump(exclude_none=True),
                headers={"Content-Type": "application/json"}
            )
            response.raise_for_status()
            response_data = response.json()
            
            # Parse the response
            task_response = GetTaskResponse.model_validate(response_data)
            
            if task_response.error:
                raise Exception(f"Error from server: {task_response.error.message}")
            
            return task_response.result


async def main():
    # Create a client
    client = SimpleA2AClient("http://localhost:5000")
    
    # Fetch the agent card
    try:
        card = await client.fetch_agent_card()
        print(f"Connected to agent: {card.name}")
        print(f"Description: {card.description}")
        print(f"Skills: {', '.join(skill.name for skill in card.skills)}")
        print()
    except Exception as e:
        print(f"Failed to fetch agent card: {e}")
        return
    
    # Send a task
    try:
        message = input("Enter a message to send: ")
        task = await client.send_task(message)
        
        print(f"\nTask ID: {task.id}")
        print(f"Status: {task.status.state}")
        
        # Print the conversation
        print("\nConversation:")
        for msg in task.history:
            role = "You" if msg.role == "user" else "Agent"
            for part in msg.parts:
                if part.type == "text":
                    print(f"{role}: {part.text}")
        
    except Exception as e:
        print(f"Error: {e}")


if __name__ == "__main__":
    asyncio.run(main())
```

## Step 4: Run the Server and Client

1. Start the server in one terminal:
   ```bash
   python server.py
   ```

2. In another terminal, run the client:
   ```bash
   python client.py
   ```

3. When prompted, enter a message to send to the server. The client will display the response from the server.

## Understanding the Code

### Server Implementation

The server implementation consists of several key components:

1. **Agent Card**: The server exposes an Agent Card at `/.well-known/agent.json` that describes its capabilities.

2. **Request Handling**: The server processes JSON-RPC requests and routes them to the appropriate handler based on the method.

3. **Task Management**: The server maintains a dictionary of tasks, each with its own state and history.

4. **Task Processing**: When a task is received, the server processes it and generates a response.

### Client Implementation

The client implementation includes:

1. **Agent Discovery**: The client fetches the Agent Card from the server to discover its capabilities.

2. **Task Creation**: The client creates tasks with unique IDs and sends them to the server.

3. **Response Processing**: The client processes the responses from the server and displays them to the user.

## Next Steps

This is a minimal implementation of an A2A server and client. You can extend it in several ways:

1. Add support for more message types beyond simple text
2. Implement error handling for various failure scenarios
3. Add authentication to secure the communication
4. Support multiple messages in a conversation

Once you're comfortable with this basic implementation, proceed to the next tutorial to learn about implementing streaming responses with Server-Sent Events.

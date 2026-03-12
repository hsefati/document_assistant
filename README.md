# Document Assistant - Intelligent Document Processing System

A multi-agent document processing system built with LangChain, LangGraph, and OpenAI that intelligently answers questions, summarizes documents, and performs calculations on financial and healthcare documents.

## Quick Start

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Create .env file
echo "OPENAI_API_KEY=sk-your-key-here" > .env

# 3. Run
python main.py
```

Then ask questions:
```
Enter Message: What's the total in invoice INV-001?
Enter Message: Calculate sum of all invoices
Enter Message: Summarize all contracts
```

## Features

✅ **Multi-Agent System** - Specialized agents for Q&A, summarization, calculations  
✅ **Intent Classification** - Auto-detects task type (qa/summarization/calculation)  
✅ **Structured Outputs** - Type-safe Pydantic schemas for all responses  
✅ **Conversation Memory** - Maintains context across multiple turns  
✅ **Session Persistence** - Resume conversations from previous sessions  
✅ **Tool Integration** - Document search, calculator, and content retrieval  
✅ **Audit Trail** - Complete logging of all operations  

## Architecture

### System Flow

```
User Input
    ↓
Intent Classifier → Determines: qa / summarization / calculation
    ↓
Specialized Agent → Routes to appropriate agent
    ├→ Q&A Agent (search + extract)
    ├→ Summarization Agent (create summaries)
    └→ Calculation Agent (math operations)
    ↓
Memory Update → Compresses conversation
    ↓
Structured Response → Validated output
    ↓
Session Persistence → Saved to disk
```

### Core Components

| Component | Purpose |
|-----------|---------|
| **Intent Classifier** | Determines task type with confidence score |
| **Q&A Agent** | Answers questions about documents |
| **Summarization Agent** | Creates summaries and extracts key points |
| **Calculation Agent** | Performs mathematical operations |
| **Retriever** | Searches and retrieves document content |
| **Memory System** | Maintains conversation history and context |

## Installation

### Requirements
- Python 3.9+
- OpenAI API key

### Setup

```bash
# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Create .env file
echo "OPENAI_API_KEY=sk-your-key" > .env
```

## Usage

### Interactive Mode

```bash
python main.py
```

**Commands:**
- `/help` - Show help
- `/docs` - List documents
- `/quit` - Exit

**Example Queries:**
```
What's the total in invoice INV-001?
Calculate the sum of all invoices
Summarize all contracts
Find documents with amounts over $50,000
```

### Programmatic Usage

```python
from document_assistant.src.assistant import DocumentAssistant
import os
from dotenv import load_dotenv

load_dotenv()

# Initialize
assistant = DocumentAssistant(openai_api_key=os.getenv("OPENAI_API_KEY"))

# Start session
session_id = assistant.start_session("user_1")

# Process message
response = assistant.process_message("What documents do we have?")

# Access response
print(f"Intent: {response['intent'].intent_type}")
print(f"Answer: {response['current_response']['answer']}")
print(f"Confidence: {response['current_response']['confidence']}")
print(f"Tools Used: {response['tools_used']}")
```

### Resume Previous Session

```python
# Load existing session
session_id = "7a8b9c0d-1e2f-3a4b-5c6d-7e8f9a0b1c2d"
assistant.start_session("user_1", session_id=session_id)

# Continue conversation - full context preserved
response = assistant.process_message("What else?")
```

## How It Works

### Intent Classification

The system determines what you're asking for:

```
User: "Calculate the sum of all invoices"
      ↓
Classifier: CALCULATION task (confidence: 0.98)
Reasoning: "User explicitly asks to 'calculate'"
```

**Intent Types:**
- `qa` - Questions about documents
- `summarization` - Requests for summaries
- `calculation` - Math operations
- `unknown` - Can't determine

### Multi-Agent Processing

Each agent type handles its domain:

**Q&A Example:**
```
User: "What's in invoice INV-001?"
→ Search for INV-001
→ Extract relevant information
→ Return structured answer with sources and confidence
```

**Calculation Example:**
```
User: "Sum all Q1 invoices"
→ Find all Q1 invoices
→ Use calculator tool
→ Return result with breakdown
```

### State & Memory

Two-level memory system for efficiency:

1. **Conversation History** - Complete record (persisted)
2. **Conversation Summary** - Compressed version (in-memory)

Example:
```
Turn 1: User asks about invoices → Full history + summary saved
Turn 2: User asks total → System uses summary from Turn 1
Turn 3: User compares → System has full context from both turns
```

### Structured Outputs

All responses are validated Pydantic schemas:

**AnswerResponse** (Q&A)
```python
{
    "question": str,
    "answer": str,
    "sources": List[str],      # Document IDs
    "confidence": float,        # 0.0-1.0
    "timestamp": datetime
}
```

**CalculationResponse** (Math)
```python
{
    "expression": str,
    "result": float,
    "explanation": str,         # Step-by-step
    "units": str,              # E.g., "USD"
    "timestamp": datetime
}
```

**SummarizationResponse** (Summaries)
```python
{
    "summary": str,
    "key_points": List[str],
    "document_ids": List[str],
    "timestamp": datetime
}
```

**UserIntent** (Classification)
```python
{
    "intent_type": str,         # qa/summarization/calculation/unknown
    "confidence": float,
    "reasoning": str
}
```

## Available Tools

Agents can use these tools:

| Tool | Purpose |
|------|---------|
| `document_search` | Find documents by keyword/type |
| `document_read` | Get full document content |
| `calculator` | Evaluate math expressions |
| `list_documents` | List all documents |

## Example Conversations

### Example 1: Simple Question
```
User: What's the total amount in invoice INV-001?

Process:
1. Intent: qa (confidence: 0.95)
2. Search for INV-001
3. Extract total: $20,000

Memory:
- summary: "User asked about INV-001: $20,000"
- active_documents: ["INV-001"]
```

### Example 2: Calculation
```
User: Calculate sum of all invoices

Process:
1. Intent: calculation (confidence: 0.98)
2. Find all invoices: INV-001 ($20k), INV-002 ($69.3k), INV-003 ($214.5k)
3. Calculate: 20000 + 69300 + 214500 = $303,800

Memory:
- summary: "Calculated all invoices: $303,800"
- active_documents: ["INV-001", "INV-002", "INV-003"]
```

### Example 3: Multi-Turn
```
Turn 1: "List all invoices"
→ Response: Lists 3 invoices

Turn 2: "What's the total?" (uses context from Turn 1)
→ Response: $303,800

Turn 3: "How does that compare to contracts?" (uses both previous turns)
→ Response: "Invoices are 1.69x higher than contracts"
```

## Project Structure

```
document_assistant/
├── README.md                  # This file
├── main.py                    # Entry point
├── requirements.txt
├── .env                       # API keys (create this)
│
├── document_assistant/src/
│   ├── schemas.py            # Pydantic models
│   ├── retrieval.py          # Document storage/search
│   ├── tools.py              # Tool definitions
│   ├── prompts.py            # LLM prompts
│   ├── agent.py              # LangGraph workflow
│   └── assistant.py          # Main API
│
├── sessions/                 # Persisted conversations
└── logs/                     # Tool usage logs
```

## Implementation Details

### Design Decisions

1. **LangGraph** - Multi-agent orchestration with state management
2. **Pydantic Schemas** - Type-safe validated outputs
3. **Two-Level Memory** - Full history + compressed summary
4. **JSON Persistence** - Simple, inspectable session storage
5. **Tool Logging** - Complete audit trail

### Key Concepts

**State Structure:**
```python
{
    "user_input": str,
    "messages": List[BaseMessage],       # Accumulates
    "intent": UserIntent,
    "next_step": str,                    # Routing
    "conversation_summary": str,         # Compressed
    "active_documents": List[str],
    "current_response": Dict,
    "tools_used": List[str],
    "actions_taken": List[str],          # Execution trace
    "session_id": str,
    "user_id": str
}
```

**Message Reducer:**
- Uses LangGraph's `add_messages` to automatically accumulate messages
- Prevents duplicates and maintains proper ordering
- Enables efficient state updates

**Intent Classification:**
- Dedicated graph node with UserIntent schema
- Returns confidence score and reasoning
- Enables transparent routing decisions

**Structured Output Pipeline:**
- Schema defined as Pydantic model
- Bound to LLM with `with_structured_output()`
- Output validated at runtime
- Errors caught and logged


## Development

### Add Custom Document
```python
from document_assistant.src.retrieval import Document

doc = Document(
    doc_id="CUSTOM-001",
    title="My Doc",
    content="...",
    doc_type="invoice",
    metadata={"total": 5000}
)
assistant.retriever.add_document(doc)
```

### View Logs
```bash
ls logs/
cat logs/tool_usage_*.json | python -m json.tool
```

## Summary

The Document Assistant combines:
- **LangGraph** for multi-agent orchestration
- **Pydantic** for type-safe responses
- **LangChain** for LLM integration
- **Persistent sessions** for resumable conversations
- **Structured memory** for efficiency
- **Complete logging** for auditability

Get started: `python main.py`



## Agent Architecture

The LangGraph agent follows this workflow:

![](./docs/langgraph_agent_architecture.png)
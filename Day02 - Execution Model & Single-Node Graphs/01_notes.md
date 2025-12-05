# Day 2 — Execution Model & Single-Node Graphs

## Learning Objectives

| # | Learning Objective | Description |
|---|--------------------|-------------|
| 1 | Understand LangGraph execution model | Learn how graphs compile, execute, and pass state step-by-step |
| 2 | Build single-node graphs | Create and run your first working LangGraph with one node |
| 3 | Debug basic graphs | Learn to trace execution, inspect state, and troubleshoot issues |

---

## Note 1: LangGraph Execution Model (Step-by-Step)

### 1. Introduction

Before you build complex graphs, you need to understand how LangGraph actually runs your workflows. The execution model is the engine that makes everything work—it's what takes your graph definition and turns it into a running application.

Understanding execution helps you:
- Debug why your graph isn't working
- Optimize performance
- Understand state flow
- Build more reliable workflows

**Why it matters:**
- You'll know exactly what happens when you call `app.invoke()`
- You can trace problems to specific execution steps
- You'll understand how state flows between nodes
- You can predict graph behavior before running it

### 2. Deep-Dive Explanation

#### The Execution Lifecycle

When you create and run a LangGraph, here's what happens:

**Step 1: Define Your Graph**
```python
graph = StateGraph(State)
graph.add_node("my_node", my_function)
graph.set_entry_point("my_node")
```

At this point, you've just defined the structure. Nothing has run yet.

**Step 2: Compile the Graph**
```python
app = graph.compile()
```

Compilation does several things:
- Validates your graph structure (checks for cycles, missing nodes, etc.)
- Builds an internal execution plan
- Sets up state management
- Prepares for execution

Think of compilation like compiling code—it checks for errors and prepares for runtime.

**Step 3: Invoke the Graph**
```python
result = app.invoke({"input": "hello"})
```

Now the magic happens. Here's the execution flow:

```
1. Start with initial state: {"input": "hello"}
2. Find entry point node
3. Execute entry point node:
   - Pass state to node function
   - Node processes state
   - Node returns state updates
4. Merge updates into state
5. Check edges from current node
6. Determine next node(s)
7. Repeat steps 3-6 until reaching end
8. Return final state
```

#### State Flow During Execution

State flows through your graph like this:

```
Initial State: {"input": "hello"}
    |
    v
[Node 1 receives state]
    |
    v
Node 1 processes: reads state["input"], does work
    |
    v
Node 1 returns: {"output": "HELLO"}
    |
    v
State merged: {"input": "hello", "output": "HELLO"}
    |
    v
[Next node receives updated state]
```

**Key points:**
- Each node receives the **full current state**
- Nodes return **updates** (not the full state)
- LangGraph **merges** updates into state automatically
- State accumulates as it flows through the graph

#### Execution Modes

LangGraph supports different execution modes:

**1. Synchronous (`invoke`)**
```python
result = app.invoke({"input": "hello"})
# Blocks until complete, returns final state
```

**2. Streaming (`stream`)**
```python
for chunk in app.stream({"input": "hello"}):
    print(chunk)
# Yields state updates as each node completes
```

**3. Async (`ainvoke`, `astream`)**
```python
result = await app.ainvoke({"input": "hello"})
# Non-blocking, for async applications
```

#### Edge Evaluation

When a node completes, LangGraph evaluates edges:

**Regular Edge:**
```python
graph.add_edge("node1", "node2")
# Always goes to node2, no evaluation needed
```

**Conditional Edge:**
```python
def route(state):
    if state["value"] > 10:
        return "node_a"
    else:
        return "node_b"

graph.add_conditional_edges("node1", route)
# Evaluates route function, then goes to returned node
```

#### Execution Visualization

Here's what happens internally:

```
[Compile]
    |
    v
Graph Structure Validated
    |
    v
Execution Plan Created
    |
    v
[Invoke]
    |
    v
State: {"input": "hello"}
    |
    v
Execute: entry_node
    |
    v
State: {"input": "hello", "step1": "done"}
    |
    v
Evaluate edges → next_node
    |
    v
Execute: next_node
    |
    v
State: {"input": "hello", "step1": "done", "step2": "done"}
    |
    v
No more edges → END
    |
    v
Return final state
```

### 3. Examples

#### Example 1: Tracing Simple Execution

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class State(TypedDict):
    value: int
    result: str

def add_one(state: State) -> State:
    new_value = state["value"] + 1
    print(f"Node executing: value was {state['value']}, now {new_value}")
    return {"value": new_value, "result": f"Added 1: {new_value}"}

graph = StateGraph(State)
graph.add_node("add", add_one)
graph.set_entry_point("add")
graph.add_edge("add", END)

app = graph.compile()

# Execution trace:
# 1. Initial state: {"value": 5}
# 2. Execute "add" node
# 3. Node prints: "Node executing: value was 5, now 6"
# 4. Node returns: {"value": 6, "result": "Added 1: 6"}
# 5. State merged: {"value": 6, "result": "Added 1: 6"}
# 6. Edge to END → execution stops
# 7. Return: {"value": 6, "result": "Added 1: 6"}

result = app.invoke({"value": 5})
print(result)  # {"value": 6, "result": "Added 1: 6"}
```

#### Example 2: Streaming Execution

```python
# Same graph as above

for chunk in app.stream({"value": 5}):
    print(f"Chunk: {chunk}")

# Output:
# Chunk: {"add": {"value": 6, "result": "Added 1: 6"}}
# Each chunk shows which node executed and what state it produced
```

#### Example 3: State Accumulation

```python
def step1(state: State) -> State:
    return {"step1_done": True}

def step2(state: State) -> State:
    # Can read step1_done from state!
    if state.get("step1_done"):
        return {"step2_done": True, "message": "Step 1 was done!"}
    return {"step2_done": False}

graph = StateGraph(State)
graph.add_node("step1", step1)
graph.add_node("step2", step2)
graph.add_edge("step1", "step2")
graph.set_entry_point("step1")

app = graph.compile()
result = app.invoke({})

# Execution:
# 1. Start: {}
# 2. step1 executes: returns {"step1_done": True}
# 3. State: {"step1_done": True}
# 4. step2 executes: reads state["step1_done"], returns {"step2_done": True, "message": "Step 1 was done!"}
# 5. Final state: {"step1_done": True, "step2_done": True, "message": "Step 1 was done!"}
```

### 4. Mini Summary (Key Takeaways for this Note)

- **Compilation** validates and prepares your graph for execution
- **Invocation** runs the graph step-by-step, executing nodes and following edges
- **State flows** through nodes, accumulating updates as it goes
- **Streaming** lets you see execution progress in real-time
- Understanding execution helps you debug and optimize your graphs

---

## Note 2: Building a Single-Node Graph (One Input → One Output)

### 1. Introduction

The best way to learn LangGraph is by building. Let's start with the simplest possible graph: one node that takes input and produces output. This might seem trivial, but it teaches you the fundamentals that every complex graph builds on.

A single-node graph is perfect for:
- Learning the basic syntax
- Understanding state flow
- Testing your setup
- Building confidence before complex graphs

**Why it matters:**
- Every graph starts with understanding single nodes
- You'll use this pattern as building blocks in larger graphs
- It's the foundation for everything else you'll learn

### 2. Deep-Dive Explanation

#### The Simplest Graph Structure

A single-node graph has:
1. A state definition
2. One node function
3. An entry point
4. An exit point (END)

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

# 1. Define state
class MyState(TypedDict):
    input: str
    output: str

# 2. Define node
def my_node(state: MyState) -> MyState:
    # Process input
    result = state["input"].upper()
    # Return update
    return {"output": result}

# 3. Build graph
graph = StateGraph(MyState)
graph.add_node("process", my_node)
graph.set_entry_point("process")
graph.add_edge("process", END)

# 4. Compile and run
app = graph.compile()
result = app.invoke({"input": "hello"})
# Result: {"input": "hello", "output": "HELLO"}
```

#### Breaking It Down

**Step 1: State Definition**
```python
class MyState(TypedDict):
    input: str
    output: str
```
This tells LangGraph what data structure flows through your graph. TypedDict ensures type safety.

**Step 2: Node Function**
```python
def my_node(state: MyState) -> MyState:
    # Your logic here
    return {"output": "result"}
```
- Takes state as input
- Does work
- Returns state updates (as a dict)

**Step 3: Graph Construction**
```python
graph = StateGraph(MyState)  # Create graph with state type
graph.add_node("process", my_node)  # Add node with name and function
graph.set_entry_point("process")  # Where execution starts
graph.add_edge("process", END)  # How to exit
```

**Step 4: Compilation and Execution**
```python
app = graph.compile()  # Prepare for execution
result = app.invoke({"input": "hello"})  # Run it!
```

#### Common Patterns

**Pattern 1: Simple Transformation**
```python
def transform(state):
    return {"output": state["input"].upper()}
```

**Pattern 2: LLM Call**
```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI()

def call_llm(state):
    response = llm.invoke(state["input"])
    return {"output": response.content}
```

**Pattern 3: Tool Call**
```python
def use_tool(state):
    result = some_tool(state["input"])
    return {"output": result}
```

### 3. Examples

#### Example 1: Basic Text Transformation

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class TextState(TypedDict):
    text: str
    reversed_text: str

def reverse_text(state: TextState) -> TextState:
    reversed = state["text"][::-1]
    return {"reversed_text": reversed}

graph = StateGraph(TextState)
graph.add_node("reverse", reverse_text)
graph.set_entry_point("reverse")
graph.add_edge("reverse", END)

app = graph.compile()
result = app.invoke({"text": "hello"})
print(result["reversed_text"])  # "olleh"
```

#### Example 2: Single LLM Node

```python
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI
from typing import TypedDict

class QAState(TypedDict):
    question: str
    answer: str

llm = ChatOpenAI(model="gpt-3.5-turbo")

def answer_question(state: QAState) -> QAState:
    prompt = f"Answer this question: {state['question']}"
    response = llm.invoke(prompt)
    return {"answer": response.content}

graph = StateGraph(QAState)
graph.add_node("answer", answer_question)
graph.set_entry_point("answer")
graph.add_edge("answer", END)

app = graph.compile()
result = app.invoke({"question": "What is Python?"})
print(result["answer"])
```

#### Example 3: Single Tool Node

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict
import requests

class SearchState(TypedDict):
    query: str
    result: str

def web_search(state: SearchState) -> SearchState:
    # Simplified search (in real app, use proper search API)
    query = state["query"]
    # ... search logic ...
    result = f"Search results for: {query}"
    return {"result": result}

graph = StateGraph(SearchState)
graph.add_node("search", web_search)
graph.set_entry_point("search")
graph.add_edge("search", END)

app = graph.compile()
result = app.invoke({"query": "LangGraph tutorial"})
print(result["result"])
```

#### Example 4: Node with Logging

```python
def process_with_logging(state: MyState) -> MyState:
    print(f"Processing: {state['input']}")
    result = state["input"].upper()
    print(f"Result: {result}")
    return {"output": result}

# Same graph setup...
# Now you can see what's happening during execution
```

### 4. Mini Summary (Key Takeaways for this Note)

- **Single-node graphs** are the foundation—master these first
- **State definition** tells LangGraph what data flows through
- **Node functions** take state, do work, return updates
- **Graph construction** involves: create graph, add node, set entry, add edge to END
- **Compile and invoke** to run your graph
- Start simple, then build complexity

---

## Note 3: Running and Debugging a Basic Graph

### 1. Introduction

You've built your first graph—great! Now you need to know how to run it, see what's happening, and fix problems when they occur. Debugging is a crucial skill for building reliable LangGraph applications.

This note covers:
- Running graphs in different ways
- Inspecting state at each step
- Common errors and how to fix them
- Best practices for debugging

**Why it matters:**
- Real graphs will have bugs—you need to find and fix them
- Understanding execution flow helps you optimize
- Good debugging practices save hours of frustration
- Production apps need observability

### 2. Deep-Dive Explanation

#### Running Your Graph

**Basic Invocation:**
```python
result = app.invoke({"input": "hello"})
# Returns final state
```

**Streaming (see progress):**
```python
for chunk in app.stream({"input": "hello"}):
    print(chunk)
# Shows state after each node
```

**With intermediate steps:**
```python
# See what happens at each step
for event in app.stream({"input": "hello"}, stream_mode="values"):
    print(f"State: {event}")
```

#### Inspecting State

**Print state in nodes:**
```python
def my_node(state):
    print(f"Current state: {state}")  # Debug print
    # ... process ...
    return {"output": "result"}
```

**Use logging:**
```python
import logging
logging.basicConfig(level=logging.DEBUG)

def my_node(state):
    logging.debug(f"Processing state: {state}")
    # ... process ...
```

**Inspect final result:**
```python
result = app.invoke({"input": "hello"})
print(f"Final state keys: {result.keys()}")
print(f"Output: {result.get('output')}")
```

#### Common Errors and Fixes

**Error 1: Missing State Field**
```python
# Error: KeyError: 'input'
# Cause: You're accessing state["input"] but it wasn't in initial state

# Fix: Make sure initial state has all required fields
result = app.invoke({"input": "hello"})  # Include "input"
```

**Error 2: Wrong State Type**
```python
# Error: Type mismatch
# Cause: State definition doesn't match what you're using

# Fix: Update TypedDict to match your actual state
class State(TypedDict):
    input: str  # Make sure this matches
    output: str
```

**Error 3: Node Not Found**
```python
# Error: Node 'my_node' not found
# Cause: You referenced a node that doesn't exist

# Fix: Check node names match exactly
graph.add_node("my_node", my_function)  # Name must match
graph.add_edge("my_node", END)  # Same name here
```

**Error 4: No Entry Point**
```python
# Error: No entry point set
# Cause: Forgot to set_entry_point()

# Fix: Always set an entry point
graph.set_entry_point("first_node")
```

**Error 5: Missing Edge**
```python
# Error: Node has no outgoing edges
# Cause: Node doesn't know where to go next

# Fix: Add an edge (to another node or END)
graph.add_edge("my_node", END)
```

#### Debugging Strategies

**Strategy 1: Add Print Statements**
```python
def my_node(state):
    print("=" * 50)
    print(f"Node: my_node")
    print(f"Input state: {state}")
    result = process(state)
    print(f"Output: {result}")
    print("=" * 50)
    return result
```

**Strategy 2: Use Streaming**
```python
# See execution step-by-step
for chunk in app.stream({"input": "hello"}):
    node_name = list(chunk.keys())[0]
    state = chunk[node_name]
    print(f"{node_name}: {state}")
```

**Strategy 3: Validate State**
```python
def my_node(state):
    # Validate input
    assert "input" in state, "Missing 'input' in state"
    assert isinstance(state["input"], str), "Input must be string"
    
    # Process...
    return {"output": "result"}
```

**Strategy 4: Test Nodes Individually**
```python
# Test node function directly
test_state = {"input": "hello"}
result = my_node(test_state)
print(result)  # Verify it works before adding to graph
```

#### Visualization

**Print graph structure:**
```python
# See your graph structure
print(graph.get_graph().draw_ascii())
```

**Use LangSmith (if configured):**
```python
# LangSmith automatically tracks execution
# View in LangSmith dashboard for detailed traces
```

### 3. Examples

#### Example 1: Debugging with Prints

```python
def process_node(state):
    print(f"[DEBUG] Entering process_node")
    print(f"[DEBUG] State keys: {list(state.keys())}")
    print(f"[DEBUG] Input value: {state.get('input', 'NOT FOUND')}")
    
    try:
        result = state["input"].upper()
        print(f"[DEBUG] Result: {result}")
        return {"output": result}
    except Exception as e:
        print(f"[ERROR] Exception: {e}")
        raise

# Run and see debug output
result = app.invoke({"input": "hello"})
```

#### Example 2: Streaming for Debugging

```python
# See what happens at each step
print("Starting execution...")
for event in app.stream({"input": "hello"}, stream_mode="values"):
    print(f"Step completed. State: {event}")
print("Execution complete.")
```

#### Example 3: Error Handling in Nodes

```python
def safe_node(state):
    try:
        # Your processing
        result = risky_operation(state["input"])
        return {"output": result}
    except KeyError as e:
        print(f"Missing key in state: {e}")
        return {"output": "ERROR: Missing input", "error": str(e)}
    except Exception as e:
        print(f"Unexpected error: {e}")
        return {"output": "ERROR", "error": str(e)}
```

#### Example 4: Validating Graph Structure

```python
# Before compiling, check your graph
print("Nodes:", graph.nodes)
print("Edges:", graph.edges)

# After compiling, you can inspect
app = graph.compile()
print("Graph compiled successfully")
print("Entry point:", app.get_graph().first)
```

### 4. Mini Summary (Key Takeaways for this Note)

- **Use streaming** to see execution progress in real-time
- **Add print/logging statements** in nodes to trace execution
- **Validate state** at the start of nodes to catch errors early
- **Test nodes individually** before adding to graph
- **Common errors** include missing state fields, wrong types, missing edges
- **Debugging is iterative**—add instrumentation, run, inspect, fix, repeat

---

## Day Summary / Key Takeaways

Excellent work! You've completed Day 2. Here's what you've learned:

- **Execution model**: LangGraph compiles your graph, then executes nodes step-by-step, passing state between them
- **Single-node graphs**: The foundation of all LangGraph workflows—one node that processes input and produces output
- **State flow**: State flows through nodes, accumulating updates as it goes
- **Debugging**: Use streaming, print statements, and validation to understand and fix your graphs
- **Running graphs**: You can invoke synchronously, stream for progress, or use async methods

**What you can do now:**
- ✅ Build and run a single-node LangGraph
- ✅ Understand how state flows through your graph
- ✅ Debug basic issues using streaming and logging
- ✅ Trace execution step-by-step

**Next up**: In Day 3, you'll connect multiple nodes together to build sequential and parallel workflows!


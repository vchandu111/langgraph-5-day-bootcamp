# Day 5 — Persistence & Reducers in LangGraph

## Learning Objectives

| # | Learning Objective | Description |
|---|--------------------|-------------|
| 1 | Understand persistence and checkpointing | Learn why persistence matters and how to enable it in LangGraph |
| 2 | Master reducers | Understand how reducers combine state across nodes and branches |
| 3 | Build reliable workflows | Use persistence and reducers for debugging and production reliability |

---

## Note 1: Persistence and Checkpointing in LangGraph

### 1. Introduction

In production applications, workflows can be long-running, fail partway through, or need to be resumed. Persistence lets you save the state of your graph at each step (checkpoints), so you can resume from where you left off if something goes wrong.

Persistence is crucial for:
- Recovering from failures
- Debugging long workflows
- Resuming interrupted processes
- Observing execution history

**Why it matters:**
- Production apps need reliability
- Long workflows can't afford to restart from scratch
- Debugging is much easier with execution history
- Essential for building robust AI applications

### 2. Deep-Dive Explanation

#### What is Persistence?

Persistence means saving the state of your graph at each step (checkpoint) so you can:
- Resume from a checkpoint if execution fails
- Replay execution to see what happened
- Debug by inspecting intermediate states
- Continue a workflow after interruption

**Without persistence:**
```
[Start] → [Node 1] → [Node 2] → [Node 3] → [FAILURE]
                                    ↑
                              Lost all progress!
```

**With persistence:**
```
[Start] → [Checkpoint 1] → [Node 2] → [Checkpoint 2] → [Node 3] → [FAILURE]
                                                              ↑
                                                    Resume from Checkpoint 2
```

#### How Checkpointing Works

**Checkpoints** are snapshots of your graph's state at specific points (typically after each node execution).

**Checkpoint contains:**
- Current state values
- Which nodes have executed
- Execution metadata (timestamps, etc.)
- Graph configuration

**When checkpoints are created:**
- After each node completes (by default)
- At specific points you define
- Before/after critical operations

#### Enabling Persistence

**Step 1: Choose a Checkpoint Backend**

LangGraph supports different backends:
- **Memory**: In-memory storage (for testing)
- **File System**: Save to files
- **Database**: Save to SQLite, PostgreSQL, etc.
- **Cloud**: Save to cloud storage

**Step 2: Configure Checkpointer**

```python
from langgraph.checkpoint.memory import MemorySaver

# Simple in-memory checkpointer (for testing)
checkpointer = MemorySaver()

# Or file-based (for production)
from langgraph.checkpoint.sqlite import SqliteSaver
checkpointer = SqliteSaver.from_conn_string("checkpoints.db")
```

**Step 3: Compile with Checkpointer**

```python
app = graph.compile(checkpointer=checkpointer)
```

**Step 4: Use Thread IDs**

When invoking, use a thread ID to identify the execution:

```python
config = {"configurable": {"thread_id": "user-123"}}
result = app.invoke({"input": "hello"}, config=config)
```

#### Resuming from Checkpoints

**Resume execution:**
```python
# If execution failed, resume from last checkpoint
result = app.invoke(
    {"input": "continue"},
    config={"configurable": {"thread_id": "user-123"}}
)
```

**Get checkpoint history:**
```python
# See all checkpoints for a thread
checkpoints = checkpointer.list({"configurable": {"thread_id": "user-123"}})
for checkpoint in checkpoints:
    print(f"Checkpoint: {checkpoint}")
```

#### Benefits of Persistence

1. **Reliability**: Resume after failures
2. **Debugging**: Inspect execution history
3. **Observability**: See what happened at each step
4. **Long Workflows**: Handle workflows that take hours/days
5. **User Sessions**: Maintain state across multiple interactions

### 3. Examples

#### Example 1: Basic Persistence Setup

```python
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict

class State(TypedDict):
    value: int
    result: str

def process(state: State) -> State:
    return {"result": f"Processed {state['value']}"}

graph = StateGraph(State)
graph.add_node("process", process)
graph.add_edge("process", END)
graph.set_entry_point("process")

# Enable persistence
checkpointer = MemorySaver()
app = graph.compile(checkpointer=checkpointer)

# Run with thread ID
config = {"configurable": {"thread_id": "test-1"}}
result = app.invoke({"value": 5}, config=config)
print(result)  # {"value": 5, "result": "Processed 5"}

# Resume (state is preserved)
result2 = app.invoke({"value": 10}, config=config)
# Can access previous state if needed
```

#### Example 2: File-Based Persistence

```python
from langgraph.checkpoint.sqlite import SqliteSaver

# Create SQLite checkpointer
checkpointer = SqliteSaver.from_conn_string("my_checkpoints.db")

app = graph.compile(checkpointer=checkpointer)

# Use thread ID for each user/session
user_config = {"configurable": {"thread_id": "user-123"}}
result = app.invoke({"input": "hello"}, config=user_config)

# Later, resume from checkpoint
result = app.invoke({"input": "continue"}, config=user_config)
```

#### Example 3: Inspecting Checkpoints

```python
# List all checkpoints for a thread
checkpoints = checkpointer.list({"configurable": {"thread_id": "user-123"}})

for checkpoint in checkpoints:
    print(f"Checkpoint ID: {checkpoint['checkpoint_id']}")
    print(f"State: {checkpoint['channel_values']}")
    print(f"Metadata: {checkpoint['metadata']}")
    print("---")

# Get specific checkpoint
latest = checkpointer.get({"configurable": {"thread_id": "user-123"}})
print(f"Latest state: {latest['channel_values']}")
```

#### Example 4: Resuming After Failure

```python
try:
    result = app.invoke({"input": "process"}, config=config)
except Exception as e:
    print(f"Error: {e}")
    # Execution failed, but checkpoint was saved
    
    # Resume from last checkpoint
    # Get the last successful state
    checkpoint = checkpointer.get(config)
    last_state = checkpoint['channel_values']
    
    # Continue from where we left off
    result = app.invoke(
        {"input": "retry"},
        config=config  # Same thread ID resumes from checkpoint
    )
```

### 4. Mini Summary (Key Takeaways for this Note)

- **Persistence** saves graph state at checkpoints for reliability
- **Checkpointing** creates snapshots after each node execution
- **Enable persistence** by adding a checkpointer when compiling
- **Use thread IDs** to identify different execution sessions
- **Resume from checkpoints** if execution fails or is interrupted
- **Inspect checkpoints** to debug and observe execution history
- Essential for production applications and long-running workflows

---

## Note 2: Reducers — Combining State Across Nodes/Branches

### 1. Introduction

When multiple nodes update state, or when parallel branches merge, you need to decide how to combine their updates. Reducers are functions that define how state updates are merged.

Reducers are especially important for:
- Parallel execution (multiple nodes updating state)
- Iterative workflows (accumulating results)
- Branch merging (combining parallel results)
- State conflicts (resolving updates)

**Why it matters:**
- Parallel nodes need to merge their outputs
- Iterative workflows accumulate state
- You need control over how state combines
- Essential for complex workflows

### 2. Deep-Dive Explanation

#### What are Reducers?

**Reducers** are functions that define how state updates are merged when:
- Multiple nodes update the same state field
- Parallel branches complete and merge
- Iterative workflows accumulate results

**Without explicit reducers**, LangGraph uses default merging (usually last write wins or list append).

**With reducers**, you control exactly how state combines.

#### Default State Merging

By default, LangGraph merges state updates:

```python
# Node 1 returns: {"result": "A"}
# Node 2 returns: {"result": "B"}
# Merged state: {"result": "B"}  # Last write wins
```

For lists/arrays:
```python
# Node 1 returns: {"items": ["A"]}
# Node 2 returns: {"items": ["B"]}
# Merged state: {"items": ["A", "B"]}  # Appended
```

#### Defining Custom Reducers

**Reducer function signature:**
```python
def my_reducer(current: Any, update: Any) -> Any:
    """
    current: Current value in state
    update: New value from node
    returns: Combined value
    """
    # Your combination logic
    return combined_value
```

**Example: Sum Reducer**
```python
def sum_reducer(current: int, update: int) -> int:
    return current + update
```

**Example: List Append Reducer**
```python
def append_reducer(current: list, update: list) -> list:
    return current + update
```

**Example: Dictionary Merge Reducer**
```python
def merge_dict_reducer(current: dict, update: dict) -> dict:
    return {**current, **update}  # Merge dictionaries
```

#### Using Reducers in State Definition

**Method 1: Using Annotated with Reducer**

```python
from typing import Annotated
from langgraph.graph import reduce

class State(TypedDict):
    total: Annotated[int, reduce(sum_reducer)]
    items: Annotated[list, reduce(append_reducer)]
    metadata: Annotated[dict, reduce(merge_dict_reducer)]
```

**Method 2: Using Reducer Classes**

```python
from langgraph.graph import add_messages

class State(TypedDict):
    messages: Annotated[list, add_messages]  # Built-in message reducer
    count: Annotated[int, reduce(sum_reducer)]
```

#### Common Reducer Patterns

**Pattern 1: Accumulation**
```python
def accumulate_reducer(current: list, update: Any) -> list:
    return current + [update]

# Use: Accumulate results from multiple nodes
```

**Pattern 2: Maximum/Minimum**
```python
def max_reducer(current: float, update: float) -> float:
    return max(current, update)

# Use: Keep the best score
```

**Pattern 3: Union (for sets)**
```python
def union_reducer(current: set, update: set) -> set:
    return current | update

# Use: Combine unique items
```

**Pattern 4: Custom Logic**
```python
def smart_merge_reducer(current: dict, update: dict) -> dict:
    # Merge, but prefer update values
    merged = {**current, **update}
    # Add special handling
    if "priority" in update:
        merged["priority_items"] = update.get("items", [])
    return merged
```

#### Reducers for Parallel Execution

When parallel nodes complete, their state updates are merged using reducers:

```python
# Parallel Node 1 returns: {"results": ["A"]}
# Parallel Node 2 returns: {"results": ["B"]}
# Parallel Node 3 returns: {"results": ["C"]}

# With append reducer:
# Final state: {"results": ["A", "B", "C"]}
```

#### Reducers for Iterative Workflows

In loops, reducers accumulate results:

```python
# Iteration 1: {"attempts": [1]}
# Iteration 2: {"attempts": [2]}
# Iteration 3: {"attempts": [3]}

# With append reducer:
# Final: {"attempts": [1, 2, 3]}
```

### 3. Examples

#### Example 1: Simple Sum Reducer

```python
from langgraph.graph import StateGraph, END, reduce
from typing import TypedDict, Annotated

def sum_reducer(current: int, update: int) -> int:
    return current + update

class State(TypedDict):
    total: Annotated[int, reduce(sum_reducer)]
    value: int

def add_value(state: State) -> State:
    return {"total": state["value"]}  # Adds to existing total

graph = StateGraph(State)
graph.add_node("add", add_value)
graph.add_edge("add", END)
graph.set_entry_point("add")

app = graph.compile()

# Initial state with total = 0
result = app.invoke({"total": 0, "value": 5})
# Result: {"total": 5, "value": 5}

# If called again with different value
result = app.invoke({"total": 5, "value": 3}, config={"configurable": {"thread_id": "same"}})
# Result: {"total": 8, "value": 3}  # Accumulated!
```

#### Example 2: List Accumulation Reducer

```python
def append_reducer(current: list, update: list) -> list:
    return current + update

class State(TypedDict):
    results: Annotated[list, reduce(append_reducer)]
    new_item: str

def add_result(state: State) -> State:
    return {"results": [state["new_item"]]}

# Multiple nodes can add to results
def add_result_1(state: State) -> State:
    return {"results": ["Result 1"]}

def add_result_2(state: State) -> State:
    return {"results": ["Result 2"]}

graph = StateGraph(State)
graph.add_node("add1", add_result_1)
graph.add_node("add2", add_result_2)
graph.add_edge("add1", "add2")
graph.set_entry_point("add1")

app = graph.compile()
result = app.invoke({"results": []})
# Result: {"results": ["Result 1", "Result 2"]}
```

#### Example 3: Parallel Results Reducer

```python
def merge_results_reducer(current: dict, update: dict) -> dict:
    # Merge dictionaries, combine lists
    merged = {**current}
    for key, value in update.items():
        if key in merged and isinstance(merged[key], list) and isinstance(value, list):
            merged[key] = merged[key] + value
        else:
            merged[key] = value
    return merged

class ParallelState(TypedDict):
    all_results: Annotated[dict, reduce(merge_results_reducer)]
    query: str

def parallel_task1(state: ParallelState) -> ParallelState:
    return {"all_results": {"source1": ["result1"]}}

def parallel_task2(state: ParallelState) -> ParallelState:
    return {"all_results": {"source2": ["result2"]}}

# When both complete, results are merged
# Final: {"all_results": {"source1": ["result1"], "source2": ["result2"]}}
```

#### Example 4: Message Reducer (Built-in)

```python
from langgraph.graph import add_messages
from langchain_core.messages import HumanMessage, AIMessage

class ChatState(TypedDict):
    messages: Annotated[list, add_messages]  # Built-in reducer for messages

def respond(state: ChatState) -> ChatState:
    # Add AI response
    return {"messages": [AIMessage(content="Hello!")]}

# Messages are automatically appended, not replaced
```

### 4. Mini Summary (Key Takeaways for this Note)

- **Reducers** define how state updates are merged
- **Use `Annotated` with `reduce()`** to specify reducers in state definition
- **Common patterns**: sum, append, merge, max/min, union
- **Essential for parallel execution** to combine results from multiple nodes
- **Important for iterative workflows** to accumulate results
- **Built-in reducers** like `add_messages` for common use cases
- Reducers give you control over state combination logic

---

## Note 3: Using Persistence + Reducers for Reliability & Debugging

### 1. Introduction

Combining persistence and reducers creates powerful, production-ready workflows. Persistence ensures you don't lose progress, and reducers ensure state combines correctly. Together, they enable reliable, debuggable, and observable applications.

This note shows you how to use both features together to build robust LangGraph applications.

**Why it matters:**
- Production apps need both reliability and correct state management
- Debugging is much easier with checkpoints and proper state merging
- Long workflows require persistence and accumulation
- Real-world applications combine these features

### 2. Deep-Dive Explanation

#### Combining Persistence and Reducers

**Persistence** saves your progress, **reducers** ensure state merges correctly. Use both for production apps:

```python
# State with reducers
class State(TypedDict):
    results: Annotated[list, reduce(append_reducer)]
    attempts: Annotated[int, reduce(sum_reducer)]
    metadata: Annotated[dict, reduce(merge_dict_reducer)]

# Graph with persistence
checkpointer = SqliteSaver.from_conn_string("app.db")
app = graph.compile(checkpointer=checkpointer)

# Run with thread ID
config = {"configurable": {"thread_id": "user-123"}}
result = app.invoke({"results": [], "attempts": 0}, config=config)
```

#### Debugging with Checkpoints

**Inspect execution history:**
```python
# Get all checkpoints for debugging
checkpoints = checkpointer.list(config)

for i, checkpoint in enumerate(checkpoints):
    state = checkpoint['channel_values']
    print(f"Step {i}: {state}")
    print(f"  Results: {state.get('results', [])}")
    print(f"  Attempts: {state.get('attempts', 0)}")
```

**Replay execution:**
```python
# See how state evolved
for checkpoint in checkpoints:
    state = checkpoint['channel_values']
    # Analyze state at each step
    # Identify where issues occurred
```

#### Reliable Parallel Execution

**With reducers, parallel nodes merge correctly:**
```python
class ParallelState(TypedDict):
    all_data: Annotated[dict, reduce(merge_dict_reducer)]

# Parallel nodes update state
# Reducer ensures proper merging
# Persistence saves intermediate states
```

#### Reliable Iterative Workflows

**With reducers, loops accumulate correctly:**
```python
class IterativeState(TypedDict):
    attempts: Annotated[list, reduce(append_reducer)]
    total_score: Annotated[float, reduce(sum_reducer)]

# Each iteration adds to attempts
# Reducer accumulates correctly
# Persistence saves progress
```

#### Error Recovery Pattern

**Resume after failure:**
```python
try:
    result = app.invoke(input_data, config=config)
except Exception as e:
    # Get last checkpoint
    checkpoint = checkpointer.get(config)
    last_state = checkpoint['channel_values']
    
    # Analyze what went wrong
    print(f"Failed at state: {last_state}")
    print(f"Error: {e}")
    
    # Resume or retry
    result = app.invoke(
        {"input": "retry", **last_state},
        config=config
    )
```

#### Observability Pattern

**Track execution:**
```python
def log_checkpoint(checkpointer, config):
    checkpoint = checkpointer.get(config)
    state = checkpoint['channel_values']
    
    print(f"Current state:")
    print(f"  Results count: {len(state.get('results', []))}")
    print(f"  Attempts: {state.get('attempts', 0)}")
    print(f"  Last update: {checkpoint['metadata'].get('timestamp')}")

# Call during execution
log_checkpoint(checkpointer, config)
```

### 3. Examples

#### Example 1: Persistent Parallel Workflow with Reducers

```python
from langgraph.graph import StateGraph, END, reduce
from langgraph.checkpoint.sqlite import SqliteSaver
from typing import TypedDict, Annotated

def append_reducer(current: list, update: list) -> list:
    return current + update

class ParallelState(TypedDict):
    results: Annotated[list, reduce(append_reducer)]
    query: str

def parallel_task1(state: ParallelState) -> ParallelState:
    return {"results": ["Result from task 1"]}

def parallel_task2(state: ParallelState) -> ParallelState:
    return {"results": ["Result from task 2"]}

def combine(state: ParallelState) -> ParallelState:
    # All results are already merged by reducer
    combined = "\n".join(state["results"])
    return {"final": combined}

graph = StateGraph(ParallelState)
graph.add_node("task1", parallel_task1)
graph.add_node("task2", parallel_task2)
graph.add_node("combine", combine)

def route_to_parallel(state):
    return ["task1", "task2"]

graph.add_conditional_edges("start", route_to_parallel)
graph.add_edge("task1", "combine")
graph.add_edge("task2", "combine")
graph.add_edge("combine", END)

# Enable persistence
checkpointer = SqliteSaver.from_conn_string("parallel.db")
app = graph.compile(checkpointer=checkpointer)

# Run with persistence
config = {"configurable": {"thread_id": "user-1"}}
result = app.invoke({"results": [], "query": "test"}, config=config)

# Checkpoints saved, can resume if needed
```

#### Example 2: Iterative Workflow with Persistence

```python
class IterativeState(TypedDict):
    attempts: Annotated[list, reduce(append_reducer)]
    current_result: str
    iteration: int

def attempt(state: IterativeState) -> IterativeState:
    iteration = state.get("iteration", 0) + 1
    result = f"Attempt {iteration}"
    return {
        "attempts": [result],
        "current_result": result,
        "iteration": iteration
    }

def should_continue(state: IterativeState) -> str:
    if state.get("iteration", 0) >= 3:
        return "end"
    return "continue"

graph = StateGraph(IterativeState)
graph.add_node("attempt", attempt)
graph.add_conditional_edges("attempt", should_continue, {
    "continue": "attempt",
    "end": END
})
graph.set_entry_point("attempt")

# Persistence saves each iteration
checkpointer = SqliteSaver.from_conn_string("iterative.db")
app = graph.compile(checkpointer=checkpointer)

config = {"configurable": {"thread_id": "iter-1"}}
result = app.invoke({"attempts": [], "iteration": 0}, config=config)

# Can inspect all attempts
checkpoints = checkpointer.list(config)
for cp in checkpoints:
    print(f"Iteration {cp['channel_values']['iteration']}: {cp['channel_values']['current_result']}")
```

#### Example 3: Debugging Failed Execution

```python
# Execution failed, but checkpoints were saved
try:
    result = app.invoke(input_data, config=config)
except Exception as e:
    print(f"Error: {e}")
    
    # Get execution history
    checkpoints = checkpointer.list(config)
    
    print("Execution history:")
    for i, cp in enumerate(checkpoints):
        state = cp['channel_values']
        print(f"  Step {i}: {state}")
        
        # Identify problematic state
        if "error" in state:
            print(f"    Error found: {state['error']}")
    
    # Get last successful checkpoint
    last = checkpointer.get(config)
    print(f"Last successful state: {last['channel_values']}")
    
    # Resume from last checkpoint
    result = app.invoke(
        {"input": "retry", **last['channel_values']},
        config=config
    )
```

### 4. Mini Summary (Key Takeaways for this Note)

- **Combine persistence and reducers** for production-ready workflows
- **Persistence** saves progress for reliability and debugging
- **Reducers** ensure state merges correctly in parallel and iterative workflows
- **Debugging** is easier with checkpoint inspection
- **Error recovery** is possible by resuming from checkpoints
- **Observability** comes from inspecting execution history
- Essential for building reliable, debuggable production applications

---

## Day Summary / Key Takeaways

Congratulations! You've completed the full 5-day LangGraph bootcamp! Here's what you've mastered:

- **Persistence and checkpointing**: Save graph state for reliability, debugging, and resuming workflows
- **Reducers**: Control how state updates merge across nodes, branches, and iterations
- **Combined power**: Using persistence + reducers for production-ready, debuggable applications
- **Reliability**: Build workflows that can recover from failures
- **Observability**: Inspect execution history and debug issues
- **Production readiness**: Essential features for real-world applications

**What you can do now:**
- ✅ Enable persistence in your graphs
- ✅ Define custom reducers for state merging
- ✅ Debug workflows using checkpoints
- ✅ Resume workflows after failures
- ✅ Build reliable, production-ready LangGraph applications

**You've completed the full bootcamp!** You now have a solid foundation in LangGraph and can build sophisticated, reliable AI workflows. Keep practicing, experiment with different patterns, and build amazing applications! 🎉


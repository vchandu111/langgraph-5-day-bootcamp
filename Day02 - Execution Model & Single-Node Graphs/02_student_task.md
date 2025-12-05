# Day 2 — Student Assignment

## Instructions

Today's tasks focus on building and running your first actual LangGraph code! These exercises map to the three Notes you just read:

- **Tasks 1-2** relate to **Note 1: LangGraph Execution Model**
- **Tasks 3-4** relate to **Note 2: Building Single-Node Graphs**
- **Tasks 5-6** relate to **Note 3: Running and Debugging**

You'll write real code this time, so make sure you have:
- Python 3.10+ installed
- LangGraph and LangChain installed (`pip install langgraph langchain openai`)
- An OpenAI API key (or another LLM provider)

Let's build some graphs! 🚀

---

## Tasks

### Task 1: Trace Execution Manually

**Objective**: Understand execution flow by tracing it yourself.

**What to do:**
1. Look at this graph code:
```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class State(TypedDict):
    number: int
    doubled: int
    result: str

def double_number(state: State) -> State:
    doubled = state["number"] * 2
    return {"doubled": doubled, "result": f"Double of {state['number']} is {doubled}"}

graph = StateGraph(State)
graph.add_node("double", double_number)
graph.set_entry_point("double")
graph.add_edge("double", END)

app = graph.compile()
result = app.invoke({"number": 5})
```

2. **Manually trace** what happens during execution. Write down:
   - Initial state
   - What happens in the "double" node
   - State after the node
   - Final result

**Your trace:**
```
Step 1: Initial state: 
Step 2: Node "double" receives:
Step 3: Node "double" processes:
Step 4: Node "double" returns:
Step 5: State after merge:
Step 6: Final result:
```

---

### Task 2: Add Debugging to See Execution

**Objective**: Use streaming and print statements to observe execution.

**What to do:**
1. Take the graph from Task 1
2. Add print statements inside the `double_number` function to show:
   - What state it receives
   - What it's processing
   - What it returns
3. Use `app.stream()` instead of `app.invoke()` to see execution step-by-step
4. Run it and observe the output

**Your code:**
```python
# Modify the function to add prints
def double_number(state: State) -> State:
    # Add your print statements here
    pass

# Use streaming
# Your streaming code here
```

**What you should see:**
- Print output from inside the node
- Streaming output showing state updates
- Final state

---

### Task 3: Build a Single-Node LLM Graph

**Objective**: Create your first working LangGraph that calls an LLM.

**What to do:**
1. Create a new Python file `task3_llm_graph.py`
2. Build a single-node graph that:
   - Takes a question as input
   - Calls an LLM to answer it
   - Returns the answer
3. Use `ChatOpenAI` from `langchain_openai`
4. Make sure to set up your API key (use environment variables)

**Requirements:**
- State should have `question` and `answer` fields
- Node should call the LLM with the question
- Graph should compile and run successfully

**Your code structure:**
```python
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI
from typing import TypedDict

# Define state
# Define node function
# Build graph
# Compile and run
```

**Test it with:** `{"question": "What is LangGraph?"}`

---

### Task 4: Modify Graph to Log Input and Output

**Objective**: Add logging to understand what's happening.

**What to do:**
1. Take your graph from Task 3
2. Modify the node to:
   - Log the input question
   - Log the LLM response
   - Log the final answer
3. Use Python's `logging` module or simple `print()` statements
4. Run it and verify you can see all the logs

**Your modified node:**
```python
def answer_question(state: State) -> State:
    # Add logging here
    # Log input
    # Call LLM
    # Log response
    # Return answer
    pass
```

**Expected output format:**
```
[LOG] Input question: What is LangGraph?
[LOG] LLM response: LangGraph is...
[LOG] Final answer: LangGraph is...
```

---

### Task 5: Handle Errors Gracefully

**Objective**: Make your graph robust by handling potential errors.

**What to do:**
1. Take your graph from Task 3 or 4
2. Add error handling to the node:
   - Handle missing "question" in state
   - Handle LLM API errors
   - Return a helpful error message instead of crashing
3. Test with:
   - Valid input: `{"question": "Hello"}`
   - Missing field: `{}`
   - (Optional) Invalid API key to test API errors

**Your error-handling code:**
```python
def answer_question(state: State) -> State:
    try:
        # Check for required fields
        # Call LLM
        # Return answer
    except KeyError as e:
        # Handle missing field
    except Exception as e:
        # Handle other errors
    pass
```

**Expected behavior:**
- Valid input: Returns answer normally
- Missing field: Returns error message in state, doesn't crash
- API error: Returns error message, doesn't crash

---

### Task 6: Create a Simple Tool Node

**Objective**: Build a single-node graph that uses a tool (not an LLM).

**What to do:**
1. Create a new file `task6_tool_graph.py`
2. Build a single-node graph with a simple "calculator" tool:
   - Takes a math expression as input (e.g., "2 + 2")
   - Evaluates it safely (use `eval()` carefully or a math parser)
   - Returns the result
3. State should have `expression` and `result` fields

**Requirements:**
- Use a simple calculation (don't use LLM, use Python code)
- Handle invalid expressions gracefully
- Log the calculation process

**Your code:**
```python
# Simple calculator node
def calculate(state: State) -> State:
    # Parse expression
    # Calculate result
    # Return result
    pass
```

**Test cases:**
- `{"expression": "2 + 2"}` → should return `{"expression": "2 + 2", "result": 4}`
- `{"expression": "10 * 5"}` → should return `{"expression": "10 * 5", "result": 50}`
- `{"expression": "invalid"}` → should handle error gracefully

---

## Expected Output (What You Should See)

After completing these tasks, you should have:

1. **Task 1**: A written trace showing you understand execution flow
   - You can explain step-by-step what happens when a graph runs
   - You understand how state flows through nodes

2. **Task 2**: Working code with debugging output
   - You can see execution progress using streaming
   - Print statements show what's happening inside nodes

3. **Task 3**: A working single-node LLM graph
   - Graph compiles without errors
   - Graph runs and returns an answer
   - You can invoke it with a question and get a response

4. **Task 4**: Enhanced graph with logging
   - You can see input, processing, and output clearly
   - Logs help you understand execution flow

5. **Task 5**: Robust graph with error handling
   - Graph doesn't crash on invalid input
   - Error messages are helpful
   - You can handle edge cases gracefully

6. **Task 6**: A tool-based graph
   - Graph uses a tool (calculator) instead of LLM
   - Tool processes input and returns result
   - Error handling for invalid inputs

**Code quality check:**
- ✅ All graphs compile without errors
- ✅ All graphs run and produce expected output
- ✅ Error handling is in place
- ✅ Code is readable and well-commented

> ❗ **Remember**: Focus on getting your graphs to work. Don't worry about making them perfect—the goal is to understand execution and build confidence with single-node graphs.


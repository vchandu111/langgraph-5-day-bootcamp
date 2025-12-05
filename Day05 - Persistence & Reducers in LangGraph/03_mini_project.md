# Day 5 — Mini Project

## Project Overview

**Project Name**: Persistent Research Session with Reducer

In this final project, you'll build a complete workflow that combines everything you've learned: sequential steps, parallel execution, persistence, and reducers. This project demonstrates production-ready patterns for building reliable LangGraph applications.

**What the project does:**
- Takes a research topic as input
- Runs multiple parallel searches (simulated)
- Uses reducers to merge search results
- Persists state at each step
- Allows resuming if interrupted
- Combines all results into a final summary
- Provides checkpoint inspection for debugging

**Why this workflow is useful:**
- Demonstrates all major LangGraph concepts
- Shows production-ready patterns
- Combines persistence and reducers
- Real-world use case (research aggregation)
- Reliable and debuggable

---

## Project Requirements

Your Persistent Research Session must:

1. **Sequential Setup**: Initialize research session
2. **Parallel Searches**: Run 3+ parallel "search" nodes
3. **Reducer Merging**: Use reducers to combine parallel search results
4. **Persistence**: Enable checkpointing for reliability
5. **Combination**: Merge all results into final summary
6. **Checkpoint Inspection**: Provide code to inspect execution history
7. **Resume Capability**: Can resume from checkpoints if interrupted

**Constraints:**
- Must use at least 3 parallel search nodes
- Must use reducers for merging parallel results
- Must enable persistence (SQLite or Memory)
- Must track search results in state
- Must provide checkpoint inspection code

**Optional enhancements:**
- Add error handling and recovery
- Implement actual web search (using APIs)
- Add quality scoring for search results
- Implement result deduplication
- Add progress tracking

---

## Step-by-Step Instructions

### Step 1: Set Up Your Project

1. Create a new file: `day05_project.py`

2. Import required modules:
```python
from langgraph.graph import StateGraph, END, reduce
from langgraph.checkpoint.sqlite import SqliteSaver
# Or for testing: from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict, Annotated
from dotenv import load_dotenv
import os

load_dotenv()
```

---

### Step 2: Define State with Reducers

Create a state structure that uses reducers for merging:

```python
def append_reducer(current: list, update: list) -> list:
    """Merges lists by appending."""
    return current + update

def merge_dict_reducer(current: dict, update: dict) -> dict:
    """Merges dictionaries."""
    return {**current, **update}

class ResearchState(TypedDict):
    topic: str
    search_results: Annotated[list, reduce(append_reducer)]  # Accumulate results
    sources: Annotated[dict, reduce(merge_dict_reducer)]  # Merge source info
    summary: str
    status: str
```

---

### Step 3: Create Sequential Setup Node

Initialize the research session:

```python
def initialize_research(state: ResearchState) -> ResearchState:
    """
    Initializes the research session.
    """
    print(f"[INIT] Starting research on: {state['topic']}")
    
    return {
        "status": "initialized",
        "search_results": [],  # Will be populated by parallel searches
        "sources": {}  # Will be populated by parallel searches
    }
```

---

### Step 4: Create Parallel Search Nodes

Create multiple search nodes that run in parallel:

```python
def search_source1(state: ResearchState) -> ResearchState:
    """
    Simulates search from source 1 (e.g., academic papers).
    """
    print(f"[SEARCH] Source 1 searching: {state['topic']}")
    
    topic = state["topic"]
    # Simulated search results
    results = [
        f"Source 1 result 1 about {topic}",
        f"Source 1 result 2 about {topic}",
    ]
    source_info = {"source1": {"count": len(results), "type": "academic"}}
    
    return {
        "search_results": results,
        "sources": source_info
    }

def search_source2(state: ResearchState) -> ResearchState:
    """
    Simulates search from source 2 (e.g., web articles).
    """
    print(f"[SEARCH] Source 2 searching: {state['topic']}")
    
    topic = state["topic"]
    results = [
        f"Source 2 result 1 about {topic}",
        f"Source 2 result 2 about {topic}",
        f"Source 2 result 3 about {topic}",
    ]
    source_info = {"source2": {"count": len(results), "type": "web"}}
    
    return {
        "search_results": results,
        "sources": source_info
    }

def search_source3(state: ResearchState) -> ResearchState:
    """
    Simulates search from source 3 (e.g., news articles).
    """
    print(f"[SEARCH] Source 3 searching: {state['topic']}")
    
    topic = state["topic"]
    results = [
        f"Source 3 result 1 about {topic}",
    ]
    source_info = {"source3": {"count": len(results), "type": "news"}}
    
    return {
        "search_results": results,
        "sources": source_info
    }
```

---

### Step 5: Create Combination Node

Combine all search results:

```python
def combine_results(state: ResearchState) -> ResearchState:
    """
    Combines all search results into a summary.
    """
    print("[COMBINE] Merging search results...")
    
    results = state.get("search_results", [])
    sources = state.get("sources", {})
    
    # Create summary
    summary = f"""
Research Summary for: {state['topic']}

Total Results: {len(results)}
Sources: {len(sources)}

Results:
{chr(10).join(f"- {r}" for r in results)}

Source Information:
{chr(10).join(f"- {k}: {v}" for k, v in sources.items())}
"""
    
    print(f"[COMBINE] Combined {len(results)} results from {len(sources)} sources")
    
    return {
        "summary": summary.strip(),
        "status": "completed"
    }
```

---

### Step 6: Build Your Graph

Connect all nodes:

```python
# Create graph
graph = StateGraph(ResearchState)

# Add nodes
graph.add_node("initialize", initialize_research)
graph.add_node("search1", search_source1)
graph.add_node("search2", search_source2)
graph.add_node("search3", search_source3)
graph.add_node("combine", combine_results)

# Set entry point
graph.set_entry_point("initialize")

# Sequential: initialize first
graph.add_edge("initialize", "search1")  # This will route to parallel

# Route to all parallel searches
def route_to_searches(state: ResearchState) -> list:
    return ["search1", "search2", "search3"]

graph.add_conditional_edges("initialize", route_to_searches)

# All searches feed into combine
graph.add_edge("search1", "combine")
graph.add_edge("search2", "combine")
graph.add_edge("search3", "combine")

# Combine exits
graph.add_edge("combine", END)
```

**Note**: The routing above might need adjustment. A simpler approach:

```python
graph.set_entry_point("initialize")
graph.add_edge("initialize", "search1")

# Use a pass-through node or route function
def route_to_parallel(state):
    return ["search2", "search3"]

graph.add_conditional_edges("initialize", route_to_parallel, {
    "search2": "search2",
    "search3": "search3"
})

# Or simpler: all searches can start from initialize
# Then all feed into combine
```

**Simplified approach:**
```python
graph.set_entry_point("initialize")

# After initialize, run all three searches in parallel
def start_searches(state):
    return ["search1", "search2", "search3"]

graph.add_conditional_edges("initialize", start_searches)

# All searches complete, then combine
graph.add_edge("search1", "combine")
graph.add_edge("search2", "combine")  
graph.add_edge("search3", "combine")
graph.add_edge("combine", END)
```

---

### Step 7: Enable Persistence

Add checkpointing:

```python
# For testing, use MemorySaver
from langgraph.checkpoint.memory import MemorySaver
checkpointer = MemorySaver()

# For production, use SQLite
# from langgraph.checkpoint.sqlite import SqliteSaver
# checkpointer = SqliteSaver.from_conn_string("research.db")

# Compile with persistence
app = graph.compile(checkpointer=checkpointer)
```

---

### Step 8: Run and Inspect

Run your workflow and inspect checkpoints:

```python
# Run with thread ID
config = {"configurable": {"thread_id": "research-1"}}

result = app.invoke({
    "topic": "Artificial Intelligence",
    "search_results": [],
    "sources": {},
    "summary": "",
    "status": ""
}, config=config)

print("\n" + "="*50)
print("FINAL RESULT:")
print("="*50)
print(result["summary"])

# Inspect checkpoints
print("\n" + "="*50)
print("CHECKPOINT HISTORY:")
print("="*50)

checkpoints = checkpointer.list(config)
for i, checkpoint in enumerate(checkpoints):
    state = checkpoint['channel_values']
    print(f"\nCheckpoint {i}:")
    print(f"  Status: {state.get('status', 'N/A')}")
    print(f"  Results count: {len(state.get('search_results', []))}")
    print(f"  Sources: {list(state.get('sources', {}).keys())}")
```

---

### Step 9: Add Resume Functionality

Add code to resume from checkpoints:

```python
def resume_research(checkpointer, thread_id, topic):
    """
    Resumes research from last checkpoint.
    """
    config = {"configurable": {"thread_id": thread_id}}
    
    # Get last checkpoint
    checkpoint = checkpointer.get(config)
    if checkpoint:
        last_state = checkpoint['channel_values']
        print(f"Resuming from checkpoint. Current status: {last_state.get('status')}")
        
        # Continue research
        result = app.invoke({
            "topic": topic,
            **last_state  # Continue from last state
        }, config=config)
        
        return result
    else:
        # No checkpoint, start fresh
        return app.invoke({
            "topic": topic,
            "search_results": [],
            "sources": {},
            "summary": "",
            "status": ""
        }, config=config)

# Test resume
result = resume_research(checkpointer, "research-1", "Machine Learning")
```

---

### Step 10: Add Debugging Utilities

Create functions to inspect and debug:

```python
def inspect_execution(checkpointer, thread_id):
    """
    Inspects execution history for debugging.
    """
    config = {"configurable": {"thread_id": thread_id}}
    checkpoints = checkpointer.list(config)
    
    print(f"\nExecution History for thread: {thread_id}")
    print("="*50)
    
    for i, checkpoint in enumerate(checkpoints):
        state = checkpoint['channel_values']
        metadata = checkpoint.get('metadata', {})
        
        print(f"\nStep {i}:")
        print(f"  Topic: {state.get('topic', 'N/A')}")
        print(f"  Status: {state.get('status', 'N/A')}")
        print(f"  Search Results: {len(state.get('search_results', []))} items")
        print(f"  Sources: {list(state.get('sources', {}).keys())}")
        if metadata:
            print(f"  Metadata: {metadata}")

# Use it
inspect_execution(checkpointer, "research-1")
```

---

## Example Interaction / Output

**Input Example:**
```python
config = {"configurable": {"thread_id": "research-1"}}
result = app.invoke({
    "topic": "Quantum Computing",
    "search_results": [],
    "sources": {},
    "summary": "",
    "status": ""
}, config=config)
```

**Expected Behavior:**

1. **Initialize**: Research session starts
2. **Parallel Searches**: Three searches run simultaneously
3. **Reducer Merging**: Results are merged using reducers
4. **Combination**: All results combined into summary
5. **Persistence**: Checkpoints saved at each step

**Console Output:**
```
[INIT] Starting research on: Quantum Computing
[SEARCH] Source 1 searching: Quantum Computing
[SEARCH] Source 2 searching: Quantum Computing
[SEARCH] Source 3 searching: Quantum Computing
[COMBINE] Merging search results...
[COMBINE] Combined 6 results from 3 sources
```

**Final Result:**
```
Research Summary for: Quantum Computing

Total Results: 6
Sources: 3

Results:
- Source 1 result 1 about Quantum Computing
- Source 1 result 2 about Quantum Computing
- Source 2 result 1 about Quantum Computing
- Source 2 result 2 about Quantum Computing
- Source 2 result 3 about Quantum Computing
- Source 3 result 1 about Quantum Computing

Source Information:
- source1: {'count': 2, 'type': 'academic'}
- source2: {'count': 3, 'type': 'web'}
- source3: {'count': 1, 'type': 'news'}
```

**Checkpoint Inspection:**
```
Checkpoint 0:
  Status: initialized
  Results count: 0
  Sources: []

Checkpoint 1:
  Status: initialized
  Results count: 2
  Sources: ['source1']

Checkpoint 2:
  Status: initialized
  Results count: 5
  Sources: ['source1', 'source2']

Checkpoint 3:
  Status: initialized
  Results count: 6
  Sources: ['source1', 'source2', 'source3']

Checkpoint 4:
  Status: completed
  Results count: 6
  Sources: ['source1', 'source2', 'source3']
```

---

## Bonus Challenges (Optional)

1. **Real Web Search**: Replace simulated searches with actual API calls (Google Search API, etc.)

2. **Result Deduplication**: Add a node that removes duplicate results before combining

3. **Quality Scoring**: Score each search result and filter low-quality ones

4. **Incremental Updates**: Allow adding new searches to existing research session

5. **Export Functionality**: Export research summary to file (JSON, Markdown, etc.)

6. **Error Recovery**: Handle search failures gracefully and continue with available results

7. **Progress Tracking**: Add progress percentage based on completed searches

---

## Tips & Hints

- **Start with MemorySaver** for testing (easier setup)
- **Test reducers separately** to ensure they work correctly
- **Use consistent thread IDs** to maintain state
- **Inspect checkpoints frequently** to understand state evolution
- **Verify parallel execution** by checking timestamps or logs
- **Test resume functionality** by interrupting and resuming

**Common Issues:**

- **Reducer not working**: Make sure you're using `Annotated[type, reduce(func)]` correctly
- **Checkpoints not saving**: Verify checkpointer is passed to `compile()`
- **State not merging**: Check that parallel nodes return updates in correct format
- **Resume not working**: Ensure you're using the same thread ID

---

## Submission Checklist

Before completing the bootcamp, make sure you have:

- [ ] Working research workflow with parallel searches
- [ ] Reducers defined and used correctly
- [ ] Persistence enabled (checkpointing works)
- [ ] Parallel execution (3+ searches run simultaneously)
- [ ] Results combine correctly using reducers
- [ ] Checkpoint inspection code
- [ ] Resume functionality (can resume from checkpoint)
- [ ] Error handling
- [ ] Logging that shows execution flow
- [ ] Graph compiles and runs without errors
- [ ] Tested with multiple topics
- [ ] (Optional) Bonus features implemented

**File to submit:**
- `day05_project.py` - Your complete Persistent Research Session

**What to demonstrate:**
- Parallel searches execute simultaneously
- Reducers merge results correctly
- Checkpoints are saved and can be inspected
- Can resume from checkpoints
- Final summary combines all results
- Execution history is visible

---

## 🎉 Congratulations!

You've completed the **LangGraph 5-Day Beginner Bootcamp**! 

You now know how to:
- ✅ Build single-node and multi-node graphs
- ✅ Create sequential and parallel workflows
- ✅ Implement conditional routing and loops
- ✅ Use persistence for reliability
- ✅ Apply reducers for state management
- ✅ Debug and observe your workflows
- ✅ Build production-ready AI applications

Keep practicing, experiment with different patterns, and build amazing LangGraph applications! 🚀


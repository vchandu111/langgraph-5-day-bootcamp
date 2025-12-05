# Day 3 — Sequential & Parallel Workflows

## Learning Objectives

| # | Learning Objective | Description |
|---|--------------------|-------------|
| 1 | Build sequential workflows | Connect multiple nodes in sequence, passing state along the chain |
| 2 | Create parallel workflows | Execute multiple nodes simultaneously using fan-out patterns |
| 3 | Combine parallel results | Merge outputs from parallel branches using fan-in patterns |

---

## Note 1: Sequential Workflows in LangGraph

### 1. Introduction

Most real-world AI workflows aren't single steps—they're sequences of operations. You might need to: summarize a document, then extract keywords, then generate questions. Each step depends on the previous one.

Sequential workflows in LangGraph are straightforward: connect nodes with edges, and state flows from one to the next. This is the foundation for building complex, multi-step AI applications.

**Why it matters:**
- Real GenAI apps almost always have multiple steps
- Sequential workflows are the most common pattern
- Understanding this unlocks building sophisticated applications
- It's the natural next step after single-node graphs

### 2. Deep-Dive Explanation

#### What is a Sequential Workflow?

A sequential workflow executes nodes one after another, in order:

```
[Node A] → [Node B] → [Node C] → [Node D]
```

Each node:
1. Receives state from the previous node
2. Processes it
3. Updates state
4. Passes state to the next node

#### Building Sequential Graphs

**Step 1: Define Multiple Nodes**

```python
def step1(state: State) -> State:
    # Process input
    return {"step1_result": "done"}

def step2(state: State) -> State:
    # Can use step1_result from state
    result = state.get("step1_result")
    return {"step2_result": f"Processed {result}"}

def step3(state: State) -> State:
    # Can use results from both previous steps
    return {"final_result": "complete"}
```

**Step 2: Connect Nodes with Edges**

```python
graph = StateGraph(State)
graph.add_node("step1", step1)
graph.add_node("step2", step2)
graph.add_node("step3", step3)

# Connect in sequence
graph.add_edge("step1", "step2")
graph.add_edge("step2", "step3")
graph.add_edge("step3", END)

graph.set_entry_point("step1")
```

**Visual Flow:**
```
Start → [step1] → [step2] → [step3] → End
```

#### State Accumulation

As state flows through sequential nodes, it accumulates:

```
Initial: {"input": "hello"}
    |
    v
[step1] adds: {"step1_result": "processed"}
State: {"input": "hello", "step1_result": "processed"}
    |
    v
[step2] adds: {"step2_result": "done"}
State: {"input": "hello", "step1_result": "processed", "step2_result": "done"}
    |
    v
[step3] adds: {"final_result": "complete"}
Final: {"input": "hello", "step1_result": "processed", "step2_result": "done", "final_result": "complete"}
```

Each node can read from the accumulated state, not just the previous node's output.

#### Common Sequential Patterns

**Pattern 1: Pipeline Processing**
```
Input → Preprocess → Process → Postprocess → Output
```

**Pattern 2: Multi-Step Analysis**
```
Text → Summarize → Extract Keywords → Generate Questions → Format
```

**Pattern 3: Validation Chain**
```
Input → Validate → Transform → Verify → Output
```

### 3. Examples

#### Example 1: Simple Three-Step Pipeline

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class PipelineState(TypedDict):
    text: str
    cleaned: str
    processed: str
    final: str

def clean_text(state: PipelineState) -> PipelineState:
    cleaned = state["text"].strip().lower()
    return {"cleaned": cleaned}

def process_text(state: PipelineState) -> PipelineState:
    # Can read from "cleaned" that step1 added
    processed = state["cleaned"].upper()
    return {"processed": processed}

def format_output(state: PipelineState) -> PipelineState:
    # Can read from both previous steps
    final = f"Original: {state['text']}, Processed: {state['processed']}"
    return {"final": final}

graph = StateGraph(PipelineState)
graph.add_node("clean", clean_text)
graph.add_node("process", process_text)
graph.add_node("format", format_output)

graph.add_edge("clean", "process")
graph.add_edge("process", "format")
graph.add_edge("format", END)

graph.set_entry_point("clean")

app = graph.compile()
result = app.invoke({"text": "  Hello World  "})
# Result: {"text": "  Hello World  ", "cleaned": "hello world", 
#          "processed": "HELLO WORLD", "final": "Original:   Hello World  , Processed: HELLO WORLD"}
```

#### Example 2: Summarize Then Analyze

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI()

class AnalysisState(TypedDict):
    article: str
    summary: str
    insights: str

def summarize(state: AnalysisState) -> AnalysisState:
    prompt = f"Summarize this article: {state['article']}"
    response = llm.invoke(prompt)
    return {"summary": response.content}

def analyze(state: AnalysisState) -> AnalysisState:
    # Uses summary from previous step
    prompt = f"Based on this summary, provide 3 key insights: {state['summary']}"
    response = llm.invoke(prompt)
    return {"insights": response.content}

graph = StateGraph(AnalysisState)
graph.add_node("summarize", summarize)
graph.add_node("analyze", analyze)
graph.add_edge("summarize", "analyze")
graph.add_edge("analyze", END)
graph.set_entry_point("summarize")

app = graph.compile()
result = app.invoke({"article": "Long article text here..."})
```

#### Example 3: Multi-Step with Intermediate Validation

```python
def validate_input(state: State) -> State:
    if not state.get("input"):
        return {"error": "Input required", "valid": False}
    return {"valid": True}

def process(state: State) -> State:
    if not state.get("valid"):
        return {"error": "Skipping process - input invalid"}
    # Process only if valid
    return {"processed": state["input"].upper()}

def format(state: State) -> State:
    if state.get("error"):
        return {"output": f"Error: {state['error']}"}
    return {"output": f"Result: {state['processed']}"}

graph = StateGraph(State)
graph.add_node("validate", validate_input)
graph.add_node("process", process)
graph.add_node("format", format)
graph.add_edge("validate", "process")
graph.add_edge("process", "format")
graph.add_edge("format", END)
graph.set_entry_point("validate")
```

### 4. Mini Summary (Key Takeaways for this Note)

- **Sequential workflows** connect nodes in order using edges
- **State accumulates** as it flows through nodes—each node can read all previous state
- **Edges define flow**—use `add_edge()` to connect nodes sequentially
- **Common pattern**: Input → Process → Transform → Output
- Sequential workflows are the foundation for most LangGraph applications

---

## Note 2: Parallel Workflows in LangGraph

### 1. Introduction

Sometimes you want to do multiple things at the same time. Maybe you want to: search the web, query a database, and call an API—all in parallel to speed things up. Or you want to generate multiple answer candidates and pick the best one.

Parallel workflows let you execute multiple nodes simultaneously, then combine their results. This is powerful for performance and for exploring multiple paths.

**Why it matters:**
- Parallel execution is faster than sequential
- You can explore multiple solutions simultaneously
- Real apps often need to gather information from multiple sources
- It's a key pattern for building efficient AI workflows

### 2. Deep-Dive Explanation

#### What is a Parallel Workflow?

A parallel workflow executes multiple nodes at the same time (fan-out), then combines results (fan-in):

```
        [Node A]
           |
    +-----+-----+
    |     |     |
    v     v     v
[Node B] [Node C] [Node D]
    |     |     |
    +-----+-----+
           |
        [Node E]
```

**Fan-out**: One node branches to multiple nodes
**Fan-in**: Multiple nodes combine into one

#### Fan-Out: Branching to Multiple Nodes

To execute nodes in parallel, you need to route from one node to multiple nodes simultaneously.

**Method 1: Using a list in conditional edges**

```python
def route_to_parallel(state):
    # Return list of nodes to execute in parallel
    return ["node1", "node2", "node3"]

graph.add_conditional_edges(
    "start",
    route_to_parallel
)
```

**Method 2: Using `add_edge` with multiple targets**

In newer LangGraph versions, you can add edges to multiple nodes:

```python
# This executes node1, node2, node3 in parallel
graph.add_edge("start", "node1")
graph.add_edge("start", "node2")
graph.add_edge("start", "node3")
```

**Method 3: Explicit parallel mapping**

```python
graph.add_node("start", start_node)
graph.add_node("parallel1", parallel_node1)
graph.add_node("parallel2", parallel_node2)

# Route to both
def route(state):
    return ["parallel1", "parallel2"]

graph.add_conditional_edges("start", route)
```

#### Fan-In: Combining Parallel Results

After parallel nodes execute, you need to combine their results. This is where **reducers** come in (we'll cover reducers in detail in Day 5).

For now, a simple approach is to have a node that waits for all parallel nodes to complete, then combines their outputs:

```python
def combine_results(state: State) -> State:
    # Read results from all parallel nodes
    result1 = state.get("parallel1_result")
    result2 = state.get("parallel2_result")
    result3 = state.get("parallel3_result")
    
    combined = f"{result1} + {result2} + {result3}"
    return {"combined": combined}

# After parallel nodes, route to combiner
graph.add_node("combine", combine_results)
graph.add_edge("parallel1", "combine")
graph.add_edge("parallel2", "combine")
graph.add_edge("parallel3", "combine")
```

**Note**: LangGraph automatically handles state merging from parallel nodes. All parallel node outputs are merged into state before the next node receives it.

#### State in Parallel Execution

When nodes run in parallel:
1. Each parallel node receives the **same input state**
2. Each node processes independently
3. All outputs are **merged** into state
4. The next node receives the **combined state**

```
Input State: {"input": "hello"}
    |
    +---> [Node1] → {"result1": "A"}
    |
    +---> [Node2] → {"result2": "B"}
    |
    +---> [Node3] → {"result3": "C"}

Merged State: {"input": "hello", "result1": "A", "result2": "B", "result3": "C"}
```

#### Common Parallel Patterns

**Pattern 1: Gather Information**
```
Query → [Search Web] ┐
       [Query DB]    ├→ Combine Results
       [Call API]   ┘
```

**Pattern 2: Generate Candidates**
```
Prompt → [Generate A] ┐
        [Generate B]  ├→ Select Best
        [Generate C]  ┘
```

**Pattern 3: Multi-Model Comparison**
```
Input → [Model 1] ┐
       [Model 2]  ├→ Compare
       [Model 3]  ┘
```

### 3. Examples

#### Example 1: Simple Parallel Execution

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class ParallelState(TypedDict):
    input: str
    result1: str
    result2: str
    result3: str

def parallel_task1(state: ParallelState) -> ParallelState:
    return {"result1": f"Task1: {state['input']}"}

def parallel_task2(state: ParallelState) -> ParallelState:
    return {"result2": f"Task2: {state['input']}"}

def parallel_task3(state: ParallelState) -> ParallelState:
    return {"result3": f"Task3: {state['input']}"}

def combine(state: ParallelState) -> ParallelState:
    combined = f"{state['result1']} | {state['result2']} | {state['result3']}"
    return {"combined": combined}

graph = StateGraph(ParallelState)
graph.add_node("start", lambda s: s)  # Pass-through
graph.add_node("task1", parallel_task1)
graph.add_node("task2", parallel_task2)
graph.add_node("task3", parallel_task3)
graph.add_node("combine", combine)

# Route to all three in parallel
def route_to_all(state):
    return ["task1", "task2", "task3"]

graph.add_conditional_edges("start", route_to_all)

# All three feed into combine
graph.add_edge("task1", "combine")
graph.add_edge("task2", "combine")
graph.add_edge("task3", "combine")
graph.add_edge("combine", END)

graph.set_entry_point("start")

app = graph.compile()
result = app.invoke({"input": "test"})
# All three tasks run in parallel, then combine
```

#### Example 2: Parallel LLM Calls

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI()

class LLMParallelState(TypedDict):
    question: str
    answer1: str
    answer2: str
    answer3: str

def expert1(state: LLMParallelState) -> LLMParallelState:
    prompt = f"As a coding expert, answer: {state['question']}"
    response = llm.invoke(prompt)
    return {"answer1": response.content}

def expert2(state: LLMParallelState) -> LLMParallelState:
    prompt = f"As a writing expert, answer: {state['question']}"
    response = llm.invoke(prompt)
    return {"answer2": response.content}

def expert3(state: LLMParallelState) -> LLMParallelState:
    prompt = f"As a general assistant, answer: {state['question']}"
    response = llm.invoke(prompt)
    return {"answer3": response.content}

def select_best(state: LLMParallelState) -> LLMParallelState:
    # Simple selection logic (in real app, use more sophisticated method)
    answers = [state["answer1"], state["answer2"], state["answer3"]]
    best = max(answers, key=len)  # Pick longest (simple heuristic)
    return {"best_answer": best}

graph = StateGraph(LLMParallelState)
graph.add_node("start", lambda s: s)
graph.add_node("expert1", expert1)
graph.add_node("expert2", expert2)
graph.add_node("expert3", expert3)
graph.add_node("select", select_best)

def route_to_experts(state):
    return ["expert1", "expert2", "expert3"]

graph.add_conditional_edges("start", route_to_experts)
graph.add_edge("expert1", "select")
graph.add_edge("expert2", "select")
graph.add_edge("expert3", "select")
graph.add_edge("select", END)
graph.set_entry_point("start")

app = graph.compile()
result = app.invoke({"question": "How do I learn Python?"})
# Three experts answer in parallel, then best is selected
```

#### Example 3: Parallel Tool Calls

```python
def search_web(state: State) -> State:
    # Simulated web search
    return {"web_result": "Web search results..."}

def query_db(state: State) -> State:
    # Simulated DB query
    return {"db_result": "Database results..."}

def call_api(state: State) -> State:
    # Simulated API call
    return {"api_result": "API results..."}

def merge_info(state: State) -> State:
    merged = f"""
    Web: {state['web_result']}
    DB: {state['db_result']}
    API: {state['api_result']}
    """
    return {"merged_info": merged}

# Graph setup similar to above...
# All three tools run in parallel, then merge
```

### 4. Mini Summary (Key Takeaways for this Note)

- **Parallel workflows** execute multiple nodes simultaneously for speed
- **Fan-out** branches from one node to multiple nodes
- **Fan-in** combines results from parallel nodes
- **State merging** happens automatically—all parallel outputs merge into state
- Use parallel execution when nodes are independent and can run simultaneously
- Common use cases: gathering info from multiple sources, generating candidates, comparing models

---

## Note 3: Combining Multiple Branches into One Result (Fan-In)

### 1. Introduction

After running nodes in parallel, you almost always need to combine their results. This "fan-in" pattern is crucial for building practical parallel workflows.

Fan-in can be simple (just concatenate results) or complex (intelligent merging, voting, ranking). Understanding fan-in patterns helps you build sophisticated workflows that leverage parallel execution effectively.

**Why it matters:**
- Parallel execution is only useful if you can combine results meaningfully
- Different combination strategies suit different use cases
- Fan-in patterns appear in most real-world parallel workflows
- It's the bridge between parallel execution and final output

### 2. Deep-Dive Explanation

#### What is Fan-In?

Fan-in is the process of combining outputs from multiple parallel nodes into a single result:

```
[Node A] ┐
[Node B] ├→ [Combine] → Final Result
[Node C] ┘
```

The combine node receives state that includes outputs from all parallel nodes.

#### Simple Combination Strategies

**Strategy 1: Concatenation**
```python
def combine(state: State) -> State:
    result1 = state.get("parallel1_result", "")
    result2 = state.get("parallel2_result", "")
    result3 = state.get("parallel3_result", "")
    
    combined = f"{result1}\n{result2}\n{result3}"
    return {"combined": combined}
```

**Strategy 2: List Aggregation**
```python
def combine(state: State) -> State:
    results = [
        state.get("result1"),
        state.get("result2"),
        state.get("result3")
    ]
    return {"all_results": results}
```

**Strategy 3: Dictionary Merge**
```python
def combine(state: State) -> State:
    # Extract all parallel results
    parallel_results = {
        "source1": state.get("result1"),
        "source2": state.get("result2"),
        "source3": state.get("result3")
    }
    return {"sources": parallel_results}
```

#### Advanced Combination Strategies

**Strategy 1: Voting/Consensus**
```python
def vote_best(state: State) -> State:
    answers = [
        state.get("answer1"),
        state.get("answer2"),
        state.get("answer3")
    ]
    # Simple voting: most common answer
    best = max(set(answers), key=answers.count)
    return {"consensus": best}
```

**Strategy 2: Ranking/Selection**
```python
def rank_answers(state: State) -> State:
    answers = [
        {"source": "expert1", "text": state.get("answer1")},
        {"source": "expert2", "text": state.get("answer2")},
        {"source": "expert3", "text": state.get("answer3")}
    ]
    # Rank by some criteria (length, keywords, etc.)
    ranked = sorted(answers, key=lambda x: len(x["text"]), reverse=True)
    return {"ranked": ranked, "best": ranked[0]}
```

**Strategy 3: LLM-Based Synthesis**
```python
def synthesize(state: State) -> State:
    # Use LLM to combine multiple answers intelligently
    prompt = f"""
    Combine these answers into one comprehensive response:
    1. {state.get('answer1')}
    2. {state.get('answer2')}
    3. {state.get('answer3')}
    """
    response = llm.invoke(prompt)
    return {"synthesized": response.content}
```

#### Handling Partial Results

Sometimes parallel nodes might fail or return empty results. Your combine function should handle this:

```python
def combine_safely(state: State) -> State:
    results = []
    for key in ["result1", "result2", "result3"]:
        value = state.get(key)
        if value:  # Only include non-empty results
            results.append(value)
    
    if not results:
        return {"combined": "No results available", "error": True}
    
    combined = "\n".join(results)
    return {"combined": combined, "error": False}
```

#### State Merging in LangGraph

When parallel nodes complete, LangGraph automatically merges their state updates:

```
Parallel Node 1 returns: {"result1": "A"}
Parallel Node 2 returns: {"result2": "B"}
Parallel Node 3 returns: {"result3": "C"}

Merged State: {"input": "hello", "result1": "A", "result2": "B", "result3": "C"}
```

The combine node receives this merged state and can access all parallel results.

### 3. Examples

#### Example 1: Simple Text Combination

```python
def combine_texts(state: State) -> State:
    text1 = state.get("summary", "")
    text2 = state.get("keywords", "")
    text3 = state.get("questions", "")
    
    combined = f"""
    Summary: {text1}
    
    Keywords: {text2}
    
    Questions: {text3}
    """
    return {"final_output": combined.strip()}

# After parallel nodes that generate summary, keywords, questions
graph.add_node("combine", combine_texts)
graph.add_edge("summary_node", "combine")
graph.add_edge("keywords_node", "combine")
graph.add_edge("questions_node", "combine")
```

#### Example 2: Select Best Answer

```python
def select_best(state: State) -> State:
    answers = [
        {"expert": "coding", "answer": state.get("coding_answer")},
        {"expert": "writing", "answer": state.get("writing_answer")},
        {"expert": "general", "answer": state.get("general_answer")}
    ]
    
    # Simple heuristic: longest answer (in real app, use better logic)
    best = max(answers, key=lambda x: len(x["answer"] or ""))
    
    return {
        "selected_answer": best["answer"],
        "selected_expert": best["expert"],
        "all_answers": answers
    }
```

#### Example 3: LLM Synthesis

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI()

def synthesize_answers(state: State) -> State:
    prompt = f"""
    You are synthesizing multiple expert opinions. Create a comprehensive answer.
    
    Coding Expert: {state.get('coding_answer', 'N/A')}
    Writing Expert: {state.get('writing_answer', 'N/A')}
    General Expert: {state.get('general_answer', 'N/A')}
    
    Combine the best insights from each into one coherent answer.
    """
    
    response = llm.invoke(prompt)
    return {"synthesized_answer": response.content}
```

#### Example 4: Aggregating Search Results

```python
def aggregate_search(state: State) -> State:
    web_results = state.get("web_results", [])
    db_results = state.get("db_results", [])
    api_results = state.get("api_results", [])
    
    # Combine all results
    all_results = {
        "web": web_results,
        "database": db_results,
        "api": api_results
    }
    
    # Create summary
    total = len(web_results) + len(db_results) + len(api_results)
    summary = f"Found {total} total results from 3 sources"
    
    return {
        "all_results": all_results,
        "summary": summary,
        "total_count": total
    }
```

### 4. Mini Summary (Key Takeaways for this Note)

- **Fan-in** combines results from parallel nodes into a single output
- **Simple strategies**: concatenation, lists, dictionaries
- **Advanced strategies**: voting, ranking, LLM synthesis
- **Handle edge cases**: partial results, failures, empty outputs
- **State merging** is automatic—combine nodes receive all parallel outputs
- Choose combination strategy based on your use case

---

## Day Summary / Key Takeaways

Fantastic! You've completed Day 3. Here's what you've mastered:

- **Sequential workflows**: Connect multiple nodes in order, with state flowing and accumulating through each step
- **Parallel workflows**: Execute multiple nodes simultaneously for speed and efficiency
- **Fan-out**: Branch from one node to multiple parallel nodes
- **Fan-in**: Combine results from parallel nodes using various strategies
- **State management**: Understand how state accumulates in sequences and merges in parallel execution

**What you can do now:**
- ✅ Build multi-step sequential workflows
- ✅ Create parallel execution patterns
- ✅ Combine parallel results effectively
- ✅ Design workflows that mix sequential and parallel patterns

**Next up**: In Day 4, you'll learn conditional routing and iterative workflows (loops and retries)!


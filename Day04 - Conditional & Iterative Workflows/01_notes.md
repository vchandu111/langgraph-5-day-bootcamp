# Day 4 — Conditional & Iterative Workflows

## Learning Objectives

| # | Learning Objective | Description |
|---|--------------------|-------------|
| 1 | Implement conditional routing | Route to different nodes based on state or conditions (if-else in graphs) |
| 2 | Build iterative workflows | Create loops, retries, and refinement patterns |
| 3 | Apply GenAI patterns | Use reflection, critique, and revision patterns in your workflows |

---

## Note 1: Conditional Routing in LangGraph (If-Else in Graphs)

### 1. Introduction

Real AI workflows rarely follow a fixed path. Sometimes you need to route to different nodes based on what happened earlier, or what the user asked, or what the model decided. This is conditional routing—the graph equivalent of if-else statements.

Conditional routing lets you build intelligent workflows that adapt to different situations, making your graphs dynamic and responsive.

**Why it matters:**
- Most real-world workflows need decision-making
- You can route to different experts, tools, or processes
- Makes graphs adaptable and intelligent
- Essential for building production-ready applications

### 2. Deep-Dive Explanation

#### What is Conditional Routing?

Conditional routing lets you choose which node to execute next based on the current state:

```
[Node A]
    |
    +--[if condition1]--> [Node B]
    |
    +--[if condition2]--> [Node C]
    |
    +--[else]--> [Node D]
```

The routing decision is made dynamically based on state values.

#### How Conditional Edges Work

**Step 1: Define a Routing Function**

```python
def route_decision(state: State) -> str:
    """
    Returns the name of the next node to execute.
    """
    if state["value"] > 10:
        return "node_b"
    elif state["value"] < 0:
        return "node_c"
    else:
        return "node_d"
```

**Step 2: Add Conditional Edge**

```python
graph.add_conditional_edges(
    "source_node",      # Node that routes from
    route_decision,     # Function that decides where to go
    {
        "node_b": "node_b",  # Map return values to node names
        "node_c": "node_c",
        "node_d": "node_d"
    }
)
```

**Simplified version** (if function returns node names directly):
```python
graph.add_conditional_edges("source_node", route_decision)
```

#### Routing Based on Different Criteria

**1. Route Based on State Values**
```python
def route_by_value(state):
    value = state.get("category")
    if value == "code":
        return "code_expert"
    elif value == "writing":
        return "writing_expert"
    else:
        return "general_assistant"
```

**2. Route Based on LLM Decision**
```python
def route_by_llm(state):
    # Use LLM to classify intent
    prompt = f"Classify this question: {state['question']}\nOptions: code, writing, general"
    response = llm.invoke(prompt)
    
    if "code" in response.content.lower():
        return "code_expert"
    elif "writing" in response.content.lower():
        return "writing_expert"
    else:
        return "general_assistant"
```

**3. Route Based on Conditions**
```python
def route_by_condition(state):
    if state.get("needs_tool"):
        return "use_tool"
    elif state.get("needs_search"):
        return "search"
    else:
        return "direct_answer"
```

#### Multiple Conditional Edges

You can have multiple conditional edges in your graph:

```python
# First routing decision
graph.add_conditional_edges("classify", route_intent)

# Later, another routing decision
graph.add_conditional_edges("process", route_quality)
```

#### Combining Regular and Conditional Edges

You can mix regular edges (always go to next node) with conditional edges:

```python
# Always go from A to B
graph.add_edge("node_a", "node_b")

# Then conditionally route from B
graph.add_conditional_edges("node_b", route_decision)
```

### 3. Examples

#### Example 1: Simple Value-Based Routing

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict

class RoutingState(TypedDict):
    number: int
    result: str

def process_number(state: RoutingState) -> RoutingState:
    number = state["number"]
    if number > 0:
        return {"result": "positive"}
    elif number < 0:
        return {"result": "negative"}
    else:
        return {"result": "zero"}

def handle_positive(state: RoutingState) -> RoutingState:
    return {"result": f"Handled positive: {state['number']}"}

def handle_negative(state: RoutingState) -> RoutingState:
    return {"result": f"Handled negative: {state['number']}"}

def handle_zero(state: RoutingState) -> RoutingState:
    return {"result": "Handled zero"}

def route_decision(state: RoutingState) -> str:
    result = state.get("result", "")
    if result == "positive":
        return "positive_handler"
    elif result == "negative":
        return "negative_handler"
    else:
        return "zero_handler"

graph = StateGraph(RoutingState)
graph.add_node("process", process_number)
graph.add_node("positive_handler", handle_positive)
graph.add_node("negative_handler", handle_negative)
graph.add_node("zero_handler", handle_zero)

graph.set_entry_point("process")
graph.add_conditional_edges("process", route_decision)
graph.add_edge("positive_handler", END)
graph.add_edge("negative_handler", END)
graph.add_edge("zero_handler", END)

app = graph.compile()
result = app.invoke({"number": 5})  # Routes to positive_handler
```

#### Example 2: LLM-Based Intent Classification

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI()

class IntentState(TypedDict):
    question: str
    intent: str
    answer: str

def classify_intent(state: IntentState) -> IntentState:
    prompt = f"""
    Classify this question into one category: code, writing, or general.
    Question: {state['question']}
    Respond with only one word: code, writing, or general.
    """
    response = llm.invoke(prompt)
    intent = response.content.strip().lower()
    return {"intent": intent}

def code_expert(state: IntentState) -> IntentState:
    prompt = f"As a coding expert, answer: {state['question']}"
    response = llm.invoke(prompt)
    return {"answer": response.content}

def writing_expert(state: IntentState) -> IntentState:
    prompt = f"As a writing expert, answer: {state['question']}"
    response = llm.invoke(prompt)
    return {"answer": response.content}

def general_assistant(state: IntentState) -> IntentState:
    prompt = f"Answer this question: {state['question']}"
    response = llm.invoke(prompt)
    return {"answer": response.content}

def route_by_intent(state: IntentState) -> str:
    intent = state.get("intent", "general")
    if "code" in intent:
        return "code_expert"
    elif "writing" in intent:
        return "writing_expert"
    else:
        return "general_assistant"

graph = StateGraph(IntentState)
graph.add_node("classify", classify_intent)
graph.add_node("code_expert", code_expert)
graph.add_node("writing_expert", writing_expert)
graph.add_node("general_assistant", general_assistant)

graph.set_entry_point("classify")
graph.add_conditional_edges("classify", route_by_intent)
graph.add_edge("code_expert", END)
graph.add_edge("writing_expert", END)
graph.add_edge("general_assistant", END)

app = graph.compile()
result = app.invoke({"question": "How do I reverse a list in Python?"})
# Routes to code_expert
```

#### Example 3: Tool-Based Routing

```python
def check_if_tool_needed(state: State) -> State:
    question = state["question"]
    # Simple heuristic: if question contains "calculate" or "search"
    if "calculate" in question.lower():
        return {"needs_tool": True, "tool_type": "calculator"}
    elif "search" in question.lower():
        return {"needs_tool": True, "tool_type": "search"}
    else:
        return {"needs_tool": False}

def route_to_tool(state: State) -> str:
    if not state.get("needs_tool"):
        return "direct_answer"
    
    tool_type = state.get("tool_type")
    if tool_type == "calculator":
        return "use_calculator"
    else:
        return "use_search"

# Graph setup...
graph.add_conditional_edges("check_tool", route_to_tool)
```

### 4. Mini Summary (Key Takeaways for this Note)

- **Conditional routing** lets you choose the next node based on state
- **Routing functions** return the name of the next node to execute
- **Use `add_conditional_edges()`** to add conditional routing
- **Route based on**: state values, LLM decisions, conditions, or any logic
- **Mix regular and conditional edges** in the same graph
- Conditional routing makes graphs dynamic and intelligent

---

## Note 2: Iterative Workflows (Loops, Retries, Refinement)

### 1. Introduction

Sometimes you need to do something multiple times: retry a failed operation, refine an answer until it's good enough, or iterate on a solution. Iterative workflows let you create loops in your graphs.

Loops are powerful for:
- Retrying failed operations
- Refining outputs until they meet quality standards
- Iterative problem-solving
- Building self-improving systems

**Why it matters:**
- Real workflows often need retries and refinement
- Quality control through iteration
- Self-correcting systems
- Essential for reliable production applications

### 2. Deep-Dive Explanation

#### What are Iterative Workflows?

Iterative workflows execute nodes multiple times, typically with a condition that determines when to stop:

```
[Generate] → [Check Quality] → [Good?] → [Return]
                    |                        ↑
                    |                        |
                    +--[Needs Work?] → [Refine] ─┘
```

The loop continues until a condition is met (quality is good, max iterations reached, etc.).

#### Creating Loops with Conditional Edges

**Basic Loop Pattern:**

```python
def should_continue(state: State) -> str:
    """
    Returns "continue" to loop, or "end" to exit.
    """
    if state.get("iteration_count", 0) >= 3:
        return "end"  # Max iterations reached
    
    if state.get("quality") == "good":
        return "end"  # Quality is good, stop
    
    return "continue"  # Keep looping

graph.add_node("generate", generate_answer)
graph.add_node("check", check_quality)
graph.add_node("refine", refine_answer)

# Create loop: generate → check → (continue → refine → generate) or (end)
graph.add_edge("generate", "check")
graph.add_conditional_edges("check", should_continue, {
    "continue": "refine",
    "end": END
})
graph.add_edge("refine", "generate")  # Loop back
```

#### Loop Control Mechanisms

**1. Maximum Iterations**
```python
def should_continue(state: State) -> str:
    iteration = state.get("iteration_count", 0)
    if iteration >= 5:  # Max 5 iterations
        return "end"
    return "continue"
```

**2. Quality Threshold**
```python
def should_continue(state: State) -> str:
    quality_score = state.get("quality_score", 0)
    if quality_score >= 0.8:  # Good enough
        return "end"
    return "continue"
```

**3. Convergence Check**
```python
def should_continue(state: State) -> str:
    current = state.get("current_result")
    previous = state.get("previous_result")
    
    if current == previous:  # No change, converged
        return "end"
    return "continue"
```

**4. Combination of Conditions**
```python
def should_continue(state: State) -> str:
    iteration = state.get("iteration_count", 0)
    quality = state.get("quality_score", 0)
    
    if iteration >= 5 or quality >= 0.8:
        return "end"
    return "continue"
```

#### Tracking Iterations

Always track iteration count to prevent infinite loops:

```python
def generate_with_count(state: State) -> State:
    count = state.get("iteration_count", 0)
    # ... generate ...
    return {
        "result": generated_result,
        "iteration_count": count + 1
    }
```

#### Common Loop Patterns

**Pattern 1: Retry Until Success**
```python
def retry_operation(state: State) -> State:
    try:
        result = risky_operation()
        return {"success": True, "result": result}
    except Exception as e:
        return {"success": False, "error": str(e)}

def should_retry(state: State) -> str:
    if state.get("success"):
        return "end"
    if state.get("retry_count", 0) >= 3:
        return "end"  # Give up after 3 retries
    return "retry"
```

**Pattern 2: Refine Until Good**
```python
def check_quality(state: State) -> State:
    quality = evaluate_quality(state["answer"])
    return {"quality_score": quality, "quality": "good" if quality > 0.8 else "poor"}

def should_refine(state: State) -> str:
    if state.get("quality") == "good":
        return "end"
    return "refine"
```

**Pattern 3: Iterative Improvement**
```python
def improve(state: State) -> State:
    current = state.get("solution")
    improved = apply_improvement(current)
    return {"solution": improved, "improvement_count": state.get("improvement_count", 0) + 1}
```

### 3. Examples

#### Example 1: Simple Retry Loop

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict
import random

class RetryState(TypedDict):
    attempt: int
    result: str
    success: bool

def try_operation(state: RetryState) -> RetryState:
    attempt = state.get("attempt", 0) + 1
    # Simulate sometimes failing operation
    success = random.random() > 0.5  # 50% success rate
    
    if success:
        return {"attempt": attempt, "success": True, "result": "Operation succeeded!"}
    else:
        return {"attempt": attempt, "success": False, "result": "Operation failed, retrying..."}

def should_retry(state: RetryState) -> str:
    if state.get("success"):
        return "end"
    if state.get("attempt", 0) >= 3:
        return "end"  # Max 3 attempts
    return "retry"

def finalize(state: RetryState) -> RetryState:
    return {"final_result": state.get("result")}

graph = StateGraph(RetryState)
graph.add_node("try", try_operation)
graph.add_node("finalize", finalize)

graph.set_entry_point("try")
graph.add_conditional_edges("try", should_retry, {
    "retry": "try",  # Loop back
    "end": "finalize"
})
graph.add_edge("finalize", END)

app = graph.compile()
result = app.invoke({"attempt": 0, "success": False})
```

#### Example 2: Refine Until Good Quality

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI()

class RefinementState(TypedDict):
    question: str
    answer: str
    quality_score: float
    iteration: int

def generate_answer(state: RefinementState) -> RefinementState:
    prompt = f"Answer this question: {state['question']}"
    response = llm.invoke(prompt)
    return {"answer": response.content, "iteration": state.get("iteration", 0) + 1}

def check_quality(state: RefinementState) -> RefinementState:
    answer = state.get("answer", "")
    # Simple quality check: length and completeness
    quality = min(len(answer) / 100, 1.0)  # Normalize to 0-1
    return {"quality_score": quality}

def refine_answer(state: RefinementState) -> RefinementState:
    prompt = f"""
    Improve this answer to be more comprehensive and detailed:
    Question: {state['question']}
    Current answer: {state['answer']}
    """
    response = llm.invoke(prompt)
    return {"answer": response.content}

def should_refine(state: RefinementState) -> str:
    quality = state.get("quality_score", 0)
    iteration = state.get("iteration", 0)
    
    if quality >= 0.7 or iteration >= 3:
        return "end"
    return "refine"

graph = StateGraph(RefinementState)
graph.add_node("generate", generate_answer)
graph.add_node("check", check_quality)
graph.add_node("refine", refine_answer)

graph.set_entry_point("generate")
graph.add_edge("generate", "check")
graph.add_conditional_edges("check", should_refine, {
    "refine": "refine",
    "end": END
})
graph.add_edge("refine", "generate")  # Loop back

app = graph.compile()
result = app.invoke({"question": "What is machine learning?", "iteration": 0})
```

#### Example 3: Iterative Problem Solving

```python
def solve_step(state: State) -> State:
    current_solution = state.get("solution", "")
    # Apply one step of problem solving
    improved = apply_one_step(current_solution)
    return {"solution": improved, "step_count": state.get("step_count", 0) + 1}

def check_convergence(state: State) -> State:
    current = state.get("solution")
    previous = state.get("previous_solution", "")
    converged = (current == previous)
    return {"converged": converged, "previous_solution": current}

def should_continue(state: State) -> str:
    if state.get("converged"):
        return "end"
    if state.get("step_count", 0) >= 10:
        return "end"
    return "continue"

# Graph: solve → check → (continue → solve) or (end)
```

### 4. Mini Summary (Key Takeaways for this Note)

- **Iterative workflows** create loops using conditional edges that route back to previous nodes
- **Loop control** uses conditions like max iterations, quality thresholds, or convergence
- **Always track iterations** to prevent infinite loops
- **Common patterns**: retry until success, refine until good, iterative improvement
- **Loops are powerful** for building self-correcting and quality-controlled systems

---

## Note 3: Typical GenAI Patterns: Reflection, Critique, and Revision

### 1. Introduction

Some of the most powerful GenAI patterns involve the model reflecting on its own output, critiquing it, and revising it. This creates self-improving systems that can produce higher quality results.

These patterns are:
- **Reflection**: The model examines its own output
- **Critique**: The model identifies issues or areas for improvement
- **Revision**: The model improves the output based on critique

**Why it matters:**
- Produces higher quality outputs
- Self-correcting systems
- Reduces need for human intervention
- Common in production GenAI applications

### 2. Deep-Dive Explanation

#### The Reflection-Critique-Revision Pattern

This is a three-step iterative pattern:

```
[Generate] → [Reflect] → [Critique] → [Good?] → [Return]
                              |                      ↑
                              |                      |
                              +--[Needs Work?] → [Revise] → [Generate] ─┘
```

**Step 1: Generate**
- Create initial output

**Step 2: Reflect**
- Model examines its own output
- Identifies strengths and weaknesses

**Step 3: Critique**
- Model provides specific feedback
- Lists issues or improvements needed

**Step 4: Decide**
- Check if quality is good enough
- If yes, return; if no, revise

**Step 5: Revise** (if needed)
- Model improves output based on critique
- Loop back to generate/reflect

#### Implementing Reflection

**Reflection Node:**
```python
def reflect(state: State) -> State:
    prompt = f"""
    Reflect on this answer. What are its strengths and weaknesses?
    
    Question: {state['question']}
    Answer: {state['answer']}
    
    Provide honest reflection.
    """
    response = llm.invoke(prompt)
    return {"reflection": response.content}
```

#### Implementing Critique

**Critique Node:**
```python
def critique(state: State) -> State:
    prompt = f"""
    Critique this answer. What specific improvements are needed?
    
    Question: {state['question']}
    Answer: {state['answer']}
    Reflection: {state['reflection']}
    
    Provide specific, actionable critique.
    """
    response = llm.invoke(prompt)
    return {"critique": response.content}
```

#### Implementing Revision

**Revision Node:**
```python
def revise(state: State) -> State:
    prompt = f"""
    Improve this answer based on the critique.
    
    Question: {state['question']}
    Original Answer: {state['answer']}
    Critique: {state['critique']}
    
    Provide an improved answer.
    """
    response = llm.invoke(prompt)
    return {"answer": response.content, "revision_count": state.get("revision_count", 0) + 1}
```

#### Quality Check

**Quality Check Node:**
```python
def check_quality(state: State) -> State:
    # Use LLM to evaluate quality
    prompt = f"""
    Rate the quality of this answer (1-10):
    
    Question: {state['question']}
    Answer: {state['answer']}
    Critique: {state.get('critique', 'N/A')}
    
    Respond with just a number 1-10.
    """
    response = llm.invoke(prompt)
    try:
        score = int(response.content.strip())
    except:
        score = 5  # Default
    
    return {"quality_score": score, "quality_good": score >= 7}
```

#### Complete Pattern Implementation

```python
def should_continue(state: State) -> str:
    if state.get("quality_good"):
        return "end"
    if state.get("revision_count", 0) >= 3:
        return "end"  # Max 3 revisions
    return "revise"

graph.add_node("generate", generate_answer)
graph.add_node("reflect", reflect)
graph.add_node("critique", critique)
graph.add_node("check_quality", check_quality)
graph.add_node("revise", revise)

graph.set_entry_point("generate")
graph.add_edge("generate", "reflect")
graph.add_edge("reflect", "critique")
graph.add_edge("critique", "check_quality")
graph.add_conditional_edges("check_quality", should_continue, {
    "revise": "revise",
    "end": END
})
graph.add_edge("revise", "generate")  # Loop back
```

### 3. Examples

#### Example 1: Simple Reflection Pattern

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI()

class ReflectionState(TypedDict):
    question: str
    answer: str
    reflection: str
    revised_answer: str
    iteration: int

def generate_answer(state: ReflectionState) -> ReflectionState:
    prompt = f"Answer: {state['question']}"
    response = llm.invoke(prompt)
    return {"answer": response.content, "iteration": state.get("iteration", 0) + 1}

def reflect_on_answer(state: ReflectionState) -> ReflectionState:
    prompt = f"""
    Reflect on this answer. Is it complete? Accurate? Well-structured?
    
    Question: {state['question']}
    Answer: {state['answer']}
    """
    response = llm.invoke(prompt)
    return {"reflection": response.content}

def revise_answer(state: ReflectionState) -> ReflectionState:
    prompt = f"""
    Improve this answer based on the reflection:
    
    Question: {state['question']}
    Original: {state['answer']}
    Reflection: {state['reflection']}
    
    Provide an improved answer.
    """
    response = llm.invoke(prompt)
    return {"revised_answer": response.content}

def should_revise(state: ReflectionState) -> str:
    iteration = state.get("iteration", 0)
    reflection = state.get("reflection", "").lower()
    
    # Simple heuristic: if reflection mentions issues, revise
    needs_work = any(word in reflection for word in ["incomplete", "unclear", "missing", "improve"])
    
    if not needs_work or iteration >= 2:
        return "end"
    return "revise"

graph = StateGraph(ReflectionState)
graph.add_node("generate", generate_answer)
graph.add_node("reflect", reflect_on_answer)
graph.add_node("revise", revise_answer)

graph.set_entry_point("generate")
graph.add_edge("generate", "reflect")
graph.add_conditional_edges("reflect", should_revise, {
    "revise": "revise",
    "end": END
})
graph.add_edge("revise", "generate")  # Loop back

app = graph.compile()
result = app.invoke({"question": "Explain quantum computing", "iteration": 0})
```

#### Example 2: Full Critique-Revision Loop

```python
def critique_answer(state: State) -> State:
    prompt = f"""
    Provide specific, actionable critique of this answer:
    
    Question: {state['question']}
    Answer: {state['answer']}
    
    List specific issues and improvements needed.
    """
    response = llm.invoke(prompt)
    return {"critique": response.content}

def revise_based_on_critique(state: State) -> State:
    prompt = f"""
    Revise this answer addressing the critique:
    
    Question: {state['question']}
    Original Answer: {state['answer']}
    Critique: {state['critique']}
    
    Provide the revised answer.
    """
    response = llm.invoke(prompt)
    return {"answer": response.content, "revision_count": state.get("revision_count", 0) + 1}

def evaluate_improvement(state: State) -> State:
    # Check if answer improved
    prompt = f"""
    Has this answer improved? Rate quality 1-10.
    
    Question: {state['question']}
    Answer: {state['answer']}
    Previous Critique: {state['critique']}
    
    Respond with just a number.
    """
    response = llm.invoke(prompt)
    try:
        score = int(response.content.strip())
    except:
        score = 5
    
    improved = score >= 7
    return {"quality_score": score, "improved": improved}

def should_continue_revising(state: State) -> str:
    if state.get("improved"):
        return "end"
    if state.get("revision_count", 0) >= 3:
        return "end"
    return "continue"

# Graph: generate → critique → revise → evaluate → (continue → critique) or (end)
```

### 4. Mini Summary (Key Takeaways for this Note)

- **Reflection pattern**: Model examines its own output for strengths/weaknesses
- **Critique pattern**: Model provides specific, actionable feedback
- **Revision pattern**: Model improves output based on critique
- **Iterative improvement**: Loop until quality is acceptable
- **Self-correcting systems**: These patterns create AI that improves itself
- **Common in production**: Many real GenAI apps use these patterns

---

## Day Summary / Key Takeaways

Outstanding work! You've completed Day 4. Here's what you've learned:

- **Conditional routing**: Route to different nodes based on state, conditions, or LLM decisions
- **Iterative workflows**: Create loops for retries, refinement, and iterative improvement
- **Reflection patterns**: Model examines and critiques its own output
- **Revision loops**: Improve outputs iteratively until quality is acceptable
- **Dynamic graphs**: Build workflows that adapt and improve

**What you can do now:**
- ✅ Build graphs with conditional routing
- ✅ Create loops for retries and refinement
- ✅ Implement reflection-critique-revision patterns
- ✅ Design self-improving AI workflows

**Next up**: In Day 5, you'll learn about persistence and reducers for building reliable, production-ready workflows!


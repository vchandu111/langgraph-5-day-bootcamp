# Day 1 — Mini Project

## Project Overview

**Project Name**: Graph Blueprint for a Question-Answering Agent

In this project, you'll design (not code yet!) a LangGraph-based architecture for a smart question-answering agent. This agent will be able to handle different types of questions, use tools when needed, and provide well-formatted answers.

**Why this workflow is useful:**
- Real QA systems need to route questions to the right handler
- Some questions need tools (web search, calculator), others don't
- Production QA systems need to handle edge cases and provide good UX
- This is a perfect use case for LangGraph's conditional routing and state management

**What you'll design:**
- A graph structure with nodes and edges
- State that flows through the graph
- Conditional routing logic
- A clear execution flow

---

## Project Requirements

Your QA agent must:

1. **Accept user questions** as input
2. **Classify the question type** (factual, calculation, code-related, general)
3. **Route conditionally** based on question type:
   - Factual questions → Use web search tool
   - Calculation questions → Use calculator tool
   - Code questions → Route to code expert node
   - General questions → Answer directly with LLM
4. **Generate an answer** using the appropriate method
5. **Format the response** consistently
6. **Handle errors gracefully** (if a tool fails, fall back to direct LLM answer)

**Constraints:**
- Must have at least 4 nodes
- Must use conditional routing (not just linear flow)
- Must define what state flows through the graph
- Design should be clear enough that someone else could implement it

---

## Step-by-Step Instructions

### Step 1: Define Your Nodes

List all the nodes (functions) your graph will need. Each node should do one specific thing.

**Example nodes:**
- `receive_question` - Accepts user input
- `classify_question` - Determines question type
- `web_search` - Searches the web for factual answers
- `calculate` - Performs calculations
- `code_expert` - Handles code-related questions
- `direct_answer` - Answers general questions with LLM
- `format_response` - Formats the final answer
- `error_handler` - Handles failures

**Your turn**: List your nodes (you can use the examples above or create your own).

---

### Step 2: Design Your State

Define what data needs to flow through your graph.

**Think about:**
- What does each node need to read?
- What does each node need to write?
- What information needs to persist across nodes?

**Example state structure:**
```python
class QAState(TypedDict):
    user_question: str          # The original question
    question_type: str           # Classified type (factual, calculation, etc.)
    tool_result: Optional[str]   # Result from tool (if used)
    answer: str                  # The generated answer
    formatted_response: str      # Final formatted output
    error: Optional[str]          # Error message if something fails
```

**Your turn**: Design your state structure. Explain why you included each field.

---

### Step 3: Map Out Your Graph

Draw or describe your graph structure showing:
- All nodes
- All edges (regular and conditional)
- Entry point
- Exit point

**Example structure:**
```
[receive_question]
    |
    v
[classify_question]
    |
    +--[factual]--> [web_search] --> [generate_answer]
    |
    +--[calculation]--> [calculate] --> [generate_answer]
    |
    +--[code]--> [code_expert] --> [generate_answer]
    |
    +--[general]--> [direct_answer]
    |
    v
[format_response]
    |
    v
[End]
```

**Your turn**: Create your own graph diagram. Make sure it shows:
- All nodes from Step 1
- How conditional routing works
- Where errors are handled (if you included error handling)

---

### Step 4: Define Conditional Routing Logic

Describe the logic for your conditional edges. When does the graph route to each node?

**Example routing logic:**
```python
def route_question(state):
    question_type = state["question_type"]
    
    if question_type == "factual":
        return "web_search"
    elif question_type == "calculation":
        return "calculate"
    elif question_type == "code":
        return "code_expert"
    else:
        return "direct_answer"
```

**Your turn**: Write out (in plain English or pseudocode) how your routing decisions work.

---

### Step 5: Describe Node Behaviors

For each node, describe what it does in 1-2 sentences.

**Example:**
- `classify_question`: Takes the user question, uses an LLM to classify it into one of four types (factual, calculation, code, general), returns updated state with question_type.

- `web_search`: Takes the question, calls a web search tool, stores the search results in state.tool_result.

**Your turn**: Describe what each of your nodes does.

---

### Step 6: Identify Potential Issues

Think about what could go wrong and how your graph handles it.

**Questions to consider:**
- What if classification fails?
- What if a tool call times out?
- What if the LLM generates an error?
- How do you handle invalid inputs?

**Your turn**: List 2-3 potential issues and how your graph design handles them (or how it should handle them).

---

## Example Interaction / Output

**Input Example:**
- "What is the capital of France?"
- "Calculate 15 * 23 + 7"
- "How do I reverse a list in Python?"
- "Tell me a joke"

**Expected Behavior:**

For "What is the capital of France?":
1. Question is received
2. Classified as "factual"
3. Routed to web_search node
4. Web search tool finds information
5. Answer is generated from search results
6. Response is formatted and returned

For "Calculate 15 * 23 + 7":
1. Question is received
2. Classified as "calculation"
3. Routed to calculate node
4. Calculator tool computes the result
5. Answer is generated
6. Response is formatted and returned

**Output Example (Format):**

```
Question: What is the capital of France?
Type: factual
Answer: The capital of France is Paris.
Source: Web search
```

or

```
Question: Calculate 15 * 23 + 7
Type: calculation
Answer: 352
Method: Calculator tool
```

**Your turn**: Describe what the output format would look like for your design. Include examples for at least 2 different question types.

---

## Bonus Challenges (Optional)

If you finish early or want to extend your design:

1. **Add a feedback loop**: Design a node that asks the user "Was this answer helpful?" and routes to improvement if needed.

2. **Add parallel processing**: Design a node that generates multiple answer candidates in parallel, then selects the best one.

3. **Add conversation history**: Modify your state to include previous questions and answers, and design nodes that can reference this history.

4. **Add quality checking**: Design a node that checks answer quality and routes to refinement if the answer is poor.

---

## Tips & Hints

- **Start simple**: Don't try to handle every edge case initially. Get the basic flow working first.
- **Think in terms of nodes**: Each node should do one thing well. If a node is doing too much, split it.
- **State is your friend**: Put everything you might need later in state. It's easier to ignore unused fields than to add missing ones.
- **Draw it out**: Visualizing your graph helps catch issues before you code.
- **Consider the user experience**: Think about what the user sees at each step. Is the flow logical?
- **Error handling matters**: In real apps, things fail. Design for failure from the start.

---

## Submission Checklist

Before moving to Day 2, make sure you have:

- [ ] Listed all nodes your graph needs
- [ ] Designed a state structure with all necessary fields
- [ ] Created a graph diagram showing nodes and edges
- [ ] Defined conditional routing logic
- [ ] Described what each node does
- [ ] Identified potential issues and solutions
- [ ] Provided example interactions and outputs

**Remember**: This is a design exercise. You don't need to write any code yet—that comes in Day 2! Focus on thinking through the architecture clearly.


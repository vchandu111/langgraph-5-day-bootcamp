# Day 1 — Student Assignment

## Instructions

Welcome to your first set of LangGraph tasks! These exercises are designed to reinforce what you learned in today's notes. Each task maps to one of the three Notes you just read:

- **Tasks 1-2** relate to **Note 1: Why LangGraph?**
- **Task 3** relates to **Note 2: LangGraph vs Classic Chains/Agents**
- **Tasks 4-5** relate to **Note 3: Core Building Blocks**

Take your time with each task. The goal isn't to write perfect code yet—it's to build intuition about when and why to use LangGraph.

---

## Tasks

### Task 1: Describe a LangGraph Use Case

**Objective**: Think about a real-world scenario where LangGraph would be better than a simple linear chain.

**What to do:**
1. Think of a GenAI application you've built or want to build (e.g., customer support bot, content generator, code assistant)
2. Describe a workflow in that app that would benefit from LangGraph
3. Explain why a linear chain would be awkward or insufficient

**Example format:**
```
Application: Customer Support Bot

Workflow: 
1. User asks a question
2. Classify the intent (technical, billing, general)
3. Route to appropriate expert
4. If answer quality is low, refine it
5. Format and return response

Why LangGraph:
- Needs conditional routing (step 3)
- Needs iterative refinement (step 4)
- Would be messy with nested if-statements in a chain
```

**Your turn**: Write your own example (different from the one above).

---

### Task 2: Draw a Text-Based Graph

**Objective**: Practice thinking in graphs by designing a simple GenAI assistant workflow.

**What to do:**
Design a text-based graph (using ASCII art or text) for a simple GenAI assistant that:
- Takes a user question
- Checks if it needs a tool (like web search)
- If yes, calls the tool, then answers
- If no, answers directly
- Formats the final response

**Example format:**
```
[Start]
  |
  v
[Receive Question]
  |
  v
[Check if Tool Needed]
  |
  +--[Yes]--> [Call Tool] --> [Generate Answer]
  |
  +--[No]--> [Generate Answer]
  |
  v
[Format Response]
  |
  v
[End]
```

**Your turn**: Draw your own graph for a different assistant (e.g., code reviewer, writing coach, research assistant).

---

### Task 3: Compare Approaches

**Objective**: Understand when to use LangGraph vs traditional chains.

**What to do:**
For each scenario below, decide whether you'd use:
- A) Traditional LangChain Chain
- B) LangChain Agent
- C) LangGraph

And explain why in 1-2 sentences.

**Scenarios:**

1. **Simple Q&A**: User asks a question, LLM answers directly
   - Your choice: _____
   - Why: _____

2. **Multi-step with routing**: Classify user intent, then route to one of three specialized assistants
   - Your choice: _____
   - Why: _____

3. **Dynamic tool selection**: Let the model decide which tools to use based on the question
   - Your choice: _____
   - Why: _____

4. **Iterative refinement**: Generate an answer, check quality, refine if needed (up to 3 times)
   - Your choice: _____
   - Why: _____

5. **Parallel processing**: Summarize a document, extract keywords, and generate questions—all at the same time
   - Your choice: _____
   - Why: _____

---

### Task 4: Identify Nodes and Edges

**Objective**: Practice identifying the building blocks of a graph.

**What to do:**
Look at this workflow description and identify:
- All the **nodes** (what work needs to be done)
- All the **edges** (how data flows)
- What **state** might be needed (what data flows through)

**Workflow:**
"A research assistant that takes a research topic, searches the web for information, summarizes the findings, extracts key insights, and formats everything into a report."

**Your answer:**

**Nodes:**
1. _____
2. _____
3. _____
4. _____
5. _____

**Edges:**
- _____ → _____
- _____ → _____
- etc.

**State (what data flows through):**
- _____
- _____
- _____

---

### Task 5: Design State Structure

**Objective**: Practice thinking about what data needs to flow through a graph.

**What to do:**
Design a state structure (as a Python TypedDict or just a list of keys) for a "Smart Email Assistant" that:
- Receives an email
- Classifies it (urgent, normal, spam)
- If urgent: sends a notification
- Generates a draft response
- Stores the conversation history

**Your state structure:**
```python
class EmailAssistantState(TypedDict):
    # Add your state fields here
    pass
```

Or as a simple list:
- State fields: _____, _____, _____

**Explain your choices**: Why did you include each field? What data does each node need?

---

## Expected Output (What You Should See)

After completing these tasks, you should have:

1. **A clear understanding** of when LangGraph is the right tool
   - You can identify workflows that need conditional routing, loops, or parallel execution
   - You understand the trade-offs between chains, agents, and graphs

2. **Graph thinking skills**
   - You can visualize workflows as nodes and edges
   - You can break down a problem into discrete steps (nodes) and connections (edges)

3. **State design intuition**
   - You can identify what data needs to flow through a graph
   - You understand that state carries information between nodes

4. **Written descriptions** (not code yet)
   - Task 1: A use case description
   - Task 2: A text-based graph diagram
   - Task 3: Choices and explanations for 5 scenarios
   - Task 4: Lists of nodes, edges, and state fields
   - Task 5: A state structure design

> ❗ **Remember**: These are conceptual tasks. You don't need to write any LangGraph code yet—that comes in Day 2! Focus on understanding the "why" and "what" before the "how."


# Day 2 — Mini Project

## Project Overview

**Project Name**: Single-Node Answering Graph

In this project, you'll build a complete, working LangGraph application with a single node that answers questions. This might seem simple, but it's the foundation for everything you'll build later.

**What the project does:**
- Takes a user question as input
- Uses an LLM to generate an answer
- Formats the response nicely
- Handles errors gracefully
- Provides logging for debugging

**Why this workflow is useful:**
- This is the building block for more complex QA systems
- You'll use this pattern in larger graphs
- It teaches you the fundamentals of state, nodes, and execution
- Real applications often start with simple single-node workflows

---

## Project Requirements

Your single-node answering graph must:

1. **Accept a question** as input in the state
2. **Call an LLM** to generate an answer
3. **Return the answer** in a structured format
4. **Handle errors** gracefully (missing input, API errors, etc.)
5. **Include logging** to show what's happening
6. **Format the output** nicely (not just raw LLM response)

**Constraints:**
- Must use exactly one node (this is a single-node graph project!)
- Must use LangGraph (not just a plain LangChain chain)
- Must handle at least 2 types of errors
- Must include logging/output that shows execution

**Optional enhancements:**
- Add a system prompt to guide the LLM
- Format the answer with markdown or structure
- Add a timestamp to the response
- Validate the question before processing

---

## Step-by-Step Instructions

### Step 1: Set Up Your Environment

1. Create a new Python file: `day02_project.py`

2. Install required packages (if not already installed):
```bash
pip install langgraph langchain openai python-dotenv
```

3. Set up your API key:
   - Create a `.env` file in your project directory
   - Add: `OPENAI_API_KEY=your_key_here`
   - Or set it as an environment variable

4. Import necessary modules:
```python
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI
from typing import TypedDict
from dotenv import load_dotenv
import os

load_dotenv()
```

---

### Step 2: Define Your State

Create a TypedDict that defines what data flows through your graph.

**Your state should include:**
- `question`: The user's question (string)
- `answer`: The generated answer (string)
- (Optional) `error`: Error message if something goes wrong (string, optional)

**Example structure:**
```python
class QAState(TypedDict):
    question: str
    answer: str
    # Add other fields as needed
```

---

### Step 3: Create the LLM Instance

Set up your LLM. You can use OpenAI or another provider.

```python
llm = ChatOpenAI(
    model="gpt-3.5-turbo",  # or "gpt-4" for better quality
    temperature=0.7
)
```

---

### Step 4: Define Your Node Function

Create a function that:
1. Takes state as input
2. Extracts the question
3. Calls the LLM
4. Extracts the answer
5. Returns updated state

**Include error handling:**
- Check if question exists
- Handle LLM API errors
- Return helpful error messages

**Include logging:**
- Log the input question
- Log when LLM is called
- Log the response

**Example structure:**
```python
def answer_question(state: QAState) -> QAState:
    # Your implementation here
    pass
```

---

### Step 5: Build Your Graph

1. Create the graph with your state type
2. Add your node
3. Set the entry point
4. Add an edge to END

```python
graph = StateGraph(QAState)
graph.add_node("answer", answer_question)
graph.set_entry_point("answer")
graph.add_edge("answer", END)
```

---

### Step 6: Compile and Test

1. Compile your graph:
```python
app = graph.compile()
```

2. Test with a simple question:
```python
result = app.invoke({"question": "What is Python?"})
print(result["answer"])
```

3. Test error handling:
```python
# Test with missing question
result = app.invoke({})
# Should handle gracefully
```

---

### Step 7: Add Streaming (Optional but Recommended)

Try using streaming to see execution in real-time:

```python
for chunk in app.stream({"question": "What is LangGraph?"}):
    print(chunk)
```

---

### Step 8: Format Your Output

Make the output user-friendly. You could:
- Add a header/title
- Format with markdown
- Include metadata (timestamp, model used, etc.)

```python
def format_output(state: QAState) -> str:
    # Format the answer nicely
    formatted = f"""
Question: {state['question']}

Answer: {state['answer']}
"""
    return formatted
```

---

## Example Interaction / Output

**Input Example:**
```python
result = app.invoke({"question": "What is LangGraph?"})
```

**Expected Behavior:**
1. Graph receives state with question
2. Node logs: "Processing question: What is LangGraph?"
3. Node calls LLM
4. Node logs: "LLM response received"
5. Node extracts answer and updates state
6. Graph returns final state

**Output Example (Format):**

**Console Output (from logging):**
```
[INFO] Processing question: What is LangGraph?
[INFO] Calling LLM...
[INFO] LLM response received
[INFO] Answer extracted successfully
```

**Final State:**
```python
{
    "question": "What is LangGraph?",
    "answer": "LangGraph is a library for building stateful, multi-actor applications with LLMs. It extends LangChain with graph-based workflows that allow for more complex control flow, including loops, conditionals, and parallel execution."
}
```

**Formatted Output (if you added formatting):**
```
Question: What is LangGraph?

Answer: LangGraph is a library for building stateful, multi-actor 
applications with LLMs. It extends LangChain with graph-based 
workflows that allow for more complex control flow, including loops, 
conditionals, and parallel execution.
```

**Error Handling Example:**
```python
result = app.invoke({})  # Missing question
# Should return:
{
    "question": "",
    "answer": "",
    "error": "Error: Missing 'question' in input state"
}
```

---

## Bonus Challenges (Optional)

If you finish early or want to extend your project:

1. **Add a system prompt**: Configure the LLM with a system message to guide its responses (e.g., "You are a helpful assistant that provides concise answers.")

2. **Add response validation**: Check if the answer is too short, too long, or contains certain keywords, and handle accordingly.

3. **Add multiple question types**: Detect if the question is about code, general knowledge, or math, and adjust the prompt accordingly.

4. **Add conversation history**: Modify state to include previous questions/answers and make the LLM aware of context.

5. **Add streaming response**: Instead of waiting for the full answer, stream tokens as they're generated.

6. **Add retry logic**: If the LLM call fails, retry up to 3 times before giving up.

---

## Tips & Hints

- **Start simple**: Get a basic version working first, then add features
- **Test frequently**: After each step, test to make sure it works
- **Use logging liberally**: It's your best friend for debugging
- **Handle errors early**: Add error handling from the start, not as an afterthought
- **Read error messages**: LangGraph errors are usually helpful—read them carefully
- **Check your API key**: Many issues are just missing or invalid API keys
- **Use streaming for debugging**: `app.stream()` shows you exactly what's happening
- **Validate state**: Check that required fields exist before using them

**Common Issues:**

- **"KeyError: 'question'"**: Make sure you're passing the question in the initial state
- **"Node not found"**: Check that node names match exactly in `add_node()` and `add_edge()`
- **"No entry point"**: Don't forget `set_entry_point()`
- **API errors**: Check your API key and network connection
- **Type errors**: Make sure your TypedDict matches what you're actually using

---

## Submission Checklist

Before moving to Day 3, make sure you have:

- [ ] A working single-node graph that answers questions
- [ ] Error handling for at least 2 error types
- [ ] Logging that shows execution flow
- [ ] Code that compiles and runs without errors
- [ ] Tested with multiple questions
- [ ] Tested error cases
- [ ] Code is readable and commented
- [ ] (Optional) Formatted output
- [ ] (Optional) Bonus features

**File to submit:**
- `day02_project.py` - Your complete working graph

**What to demonstrate:**
- Graph compiles successfully
- Graph answers questions correctly
- Error handling works
- Logging shows execution

Great job! You've built your first working LangGraph. In Day 3, you'll connect multiple nodes together! 🎉


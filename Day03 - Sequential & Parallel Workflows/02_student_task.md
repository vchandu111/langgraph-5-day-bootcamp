# Day 3 — Student Assignment

## Instructions

Today you'll practice building sequential and parallel workflows! These tasks map to the three Notes you just read:

- **Tasks 1-2** relate to **Note 1: Sequential Workflows**
- **Tasks 3-4** relate to **Note 2: Parallel Workflows**
- **Tasks 5-6** relate to **Note 3: Fan-In Patterns**

You'll be building real graphs with multiple nodes, so make sure you understand the concepts from the notes. Start simple and build up complexity.

---

## Tasks

### Task 1: Create a Simple Sequential Graph

**Objective**: Build a 3-node sequential workflow.

**What to do:**
1. Create a file `task1_sequential.py`
2. Build a graph with three nodes:
   - `clean`: Removes whitespace and converts to lowercase
   - `process`: Converts text to uppercase
   - `format`: Wraps result in a formatted string
3. Connect them sequentially: clean → process → format
4. Test with: `{"text": "  Hello World  "}`

**Requirements:**
- Each node should read from previous node's output
- State should accumulate through all three nodes
- Final output should show all intermediate steps

**Expected flow:**
```
Input: "  Hello World  "
  ↓
[clean] → "hello world"
  ↓
[process] → "HELLO WORLD"
  ↓
[format] → "Result: HELLO WORLD (from: hello world)"
```

---

### Task 2: Build "Summarize → Extract Keywords" Workflow

**Objective**: Create a sequential workflow with LLM calls.

**What to do:**
1. Create `task2_summarize_keywords.py`
2. Build a sequential graph:
   - `summarize`: Takes article text, uses LLM to create summary
   - `extract_keywords`: Takes the summary, uses LLM to extract 5 keywords
3. The second node should use the summary from the first node
4. Test with a sample article (you can use a short paragraph)

**Requirements:**
- Use `ChatOpenAI` for both nodes
- Second node should reference the summary in its prompt
- Include error handling
- Log each step

**State structure:**
```python
class ArticleState(TypedDict):
    article: str
    summary: str
    keywords: str
```

**Expected behavior:**
- First node generates summary
- Second node reads summary and extracts keywords
- Final state contains both summary and keywords

---

### Task 3: Create a Parallel Graph with Two LLM Prompts

**Objective**: Execute two LLM calls in parallel.

**What to do:**
1. Create `task3_parallel_llm.py`
2. Build a graph that:
   - Takes a question as input
   - Routes to two parallel nodes:
     - `answer_concise`: Answers in 1-2 sentences
     - `answer_detailed`: Answers in a paragraph
   - Both nodes run in parallel
   - A combine node merges both answers

**Requirements:**
- Both LLM nodes should run simultaneously
- Combine node should show both answers
- Use conditional edges or multiple edges to route to parallel nodes
- Test with: `{"question": "What is machine learning?"}`

**Expected output:**
```python
{
    "question": "...",
    "concise_answer": "...",
    "detailed_answer": "...",
    "combined": "Concise: ...\nDetailed: ..."
}
```

---

### Task 4: Parallel Tool Execution

**Objective**: Run multiple tools in parallel.

**What to do:**
1. Create `task4_parallel_tools.py`
2. Build a graph with three parallel "tool" nodes:
   - `tool1`: Simulates a web search (returns mock results)
   - `tool2`: Simulates a database query (returns mock results)
   - `tool3`: Simulates an API call (returns mock results)
3. All three should run in parallel
4. A combine node aggregates all results

**Requirements:**
- Each tool node should simulate some work (use `time.sleep(0.1)` to simulate delay)
- All tools should receive the same input query
- Combine node should merge all three results
- Log when each tool starts and completes

**Mock tool example:**
```python
def web_search(state):
    time.sleep(0.1)  # Simulate network delay
    return {"web_result": f"Web results for: {state['query']}"}
```

**Expected behavior:**
- All three tools start at the same time
- They complete (approximately) at the same time
- Combine node receives all three results
- Final output shows aggregated information

---

### Task 5: Implement Fan-In with Simple Concatenation

**Objective**: Practice combining parallel results.

**What to do:**
1. Take your parallel graph from Task 3 or 4
2. Modify the combine node to use different combination strategies:
   - **Strategy A**: Simple concatenation (join with newlines)
   - **Strategy B**: List aggregation (put all results in a list)
   - **Strategy C**: Dictionary format (organize by source)

**Requirements:**
- Implement all three strategies (you can create three different combine functions)
- Test each one
- Compare the outputs

**Example outputs:**

**Strategy A (Concatenation):**
```python
{"combined": "Result1\nResult2\nResult3"}
```

**Strategy B (List):**
```python
{"combined": ["Result1", "Result2", "Result3"]}
```

**Strategy C (Dictionary):**
```python
{"combined": {"source1": "Result1", "source2": "Result2", "source3": "Result3"}}
```

---

### Task 6: Build "Summarize & Analyze" Workflow (Sequential + Parallel)

**Objective**: Combine sequential and parallel patterns.

**What to do:**
1. Create `task6_hybrid.py`
2. Build a workflow that:
   - **Step 1 (Sequential)**: Summarize an article
   - **Step 2 (Parallel)**: After summarization, run three analyses in parallel:
     - Extract keywords
     - Generate questions
     - Identify main topics
   - **Step 3 (Sequential)**: Combine all results into a final report

**Requirements:**
- First step is sequential (summarize)
- Second step is parallel (three analyses)
- Third step combines parallel results
- Use LLM for summarize and the three analyses
- Final combine node creates a structured report

**State structure:**
```python
class HybridState(TypedDict):
    article: str
    summary: str
    keywords: str
    questions: str
    topics: str
    final_report: str
```

**Expected flow:**
```
[article] → [summarize] → [extract_keywords] ┐
                          [generate_questions] ├→ [combine] → [final_report]
                          [identify_topics]   ┘
```

**Test with a sample article and verify:**
- Summary is generated first
- Three analyses run in parallel (after summary)
- Final report combines everything

---

## Expected Output (What You Should See)

After completing these tasks, you should have:

1. **Task 1**: A working 3-node sequential graph
   - Text flows through clean → process → format
   - State accumulates correctly
   - You can see intermediate values in final state

2. **Task 2**: A sequential LLM workflow
   - Summary is generated first
   - Keywords extraction uses the summary
   - Both results are in final state

3. **Task 3**: A parallel LLM graph
   - Two LLM calls execute simultaneously
   - Both answers are generated
   - Combine node merges them

4. **Task 4**: Parallel tool execution
   - Three tools run at the same time
   - All complete around the same time (not sequentially)
   - Results are aggregated

5. **Task 5**: Multiple fan-in strategies
   - You understand different ways to combine results
   - You can choose the right strategy for different use cases

6. **Task 6**: Hybrid sequential + parallel workflow
   - Sequential step runs first
   - Parallel steps execute after
   - Final combination creates comprehensive output

**Code quality check:**
- ✅ All graphs compile and run
- ✅ Sequential workflows pass state correctly
- ✅ Parallel workflows execute simultaneously
- ✅ Fan-in nodes combine results properly
- ✅ Error handling is in place
- ✅ Code is readable and well-commented

> ❗ **Tips**: 
> - Start with Task 1 to get comfortable with sequential flows
> - Use `app.stream()` to see parallel execution in action
> - Add print statements to verify nodes run in parallel (check timestamps)
> - Test with simple inputs first, then add complexity


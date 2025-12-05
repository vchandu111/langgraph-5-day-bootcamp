# Day 4 — Student Assignment

## Instructions

Today you'll practice building conditional and iterative workflows! These tasks map to the three Notes you just read:

- **Tasks 1-2** relate to **Note 1: Conditional Routing**
- **Tasks 3-4** relate to **Note 2: Iterative Workflows**
- **Tasks 5-6** relate to **Note 3: Reflection, Critique, and Revision**

You'll be building graphs with routing logic and loops, so make sure you understand conditional edges and how to create cycles in graphs.

---

## Tasks

### Task 1: Create a Graph with Conditional Routing

**Objective**: Build a graph that routes to different nodes based on input.

**What to do:**
1. Create `task1_conditional_routing.py`
2. Build a graph that:
   - Takes a number as input
   - Routes to different handlers based on value:
     - If number > 10 → "large_handler"
     - If number < 0 → "negative_handler"
     - Otherwise → "small_handler"
3. Each handler should process the number differently
4. Test with: `{"number": 15}`, `{"number": -5}`, `{"number": 5}`

**Requirements:**
- Use conditional edges for routing
- Each handler node should do something different
- Final state should show which handler was used

**Expected behavior:**
- Input 15 → routes to large_handler
- Input -5 → routes to negative_handler
- Input 5 → routes to small_handler

---

### Task 2: Build an LLM-Based Router

**Objective**: Use an LLM to classify intent and route accordingly.

**What to do:**
1. Create `task2_llm_router.py`
2. Build a graph that:
   - Takes a user question
   - Uses LLM to classify intent (code, writing, or general)
   - Routes to appropriate expert node based on classification
   - Each expert answers the question
3. Test with different question types

**Requirements:**
- Classification node uses LLM
- Routing function reads classification from state
- Three expert nodes (code, writing, general)
- Each expert provides specialized answer

**State structure:**
```python
class RouterState(TypedDict):
    question: str
    intent: str
    answer: str
```

**Expected flow:**
```
[question] → [classify] → [code_expert] or [writing_expert] or [general_expert] → [answer]
```

---

### Task 3: Implement a Simple Retry Loop

**Objective**: Create a graph that retries an operation until success.

**What to do:**
1. Create `task3_retry_loop.py`
2. Build a graph that:
   - Attempts an operation (can be simulated with random success/failure)
   - Checks if operation succeeded
   - If failed and retries < 3, retry
   - If succeeded or max retries, end
3. Track attempt count in state

**Requirements:**
- Operation node that sometimes fails
- Check node that evaluates success
- Conditional routing: retry or end
- Loop back to operation node on retry
- Track attempt count

**State structure:**
```python
class RetryState(TypedDict):
    attempt: int
    success: bool
    result: str
```

**Expected behavior:**
- Operation may fail initially
- Graph retries up to 3 times
- Stops when successful or max retries reached
- Final state shows attempt count and result

---

### Task 4: Build a "Refine Until Good" Loop

**Objective**: Create an iterative refinement workflow.

**What to do:**
1. Create `task4_refine_loop.py`
2. Build a graph that:
   - Generates an initial answer
   - Checks answer quality (simple heuristic: length, keywords, etc.)
   - If quality is low and iterations < 3, refine the answer
   - If quality is good or max iterations, return answer
3. Use LLM for generation and refinement

**Requirements:**
- Generate node creates initial answer
- Quality check node evaluates answer
- Refine node improves answer
- Loop: generate → check → (refine → generate) or (end)
- Track iteration count

**State structure:**
```python
class RefinementState(TypedDict):
    question: str
    answer: str
    quality_score: float
    iteration: int
```

**Expected behavior:**
- Initial answer may be short or incomplete
- Quality check identifies issues
- Refinement improves answer
- Loop continues until quality is acceptable or max iterations

---

### Task 5: Implement Reflection Pattern

**Objective**: Build a graph where the model reflects on its output.

**What to do:**
1. Create `task5_reflection.py`
2. Build a graph that:
   - Generates an answer to a question
   - Reflects on the answer (model examines its own output)
   - Decides if answer is good enough
   - If not, revises based on reflection
   - If yes, returns answer
3. Use LLM for all steps

**Requirements:**
- Generate node creates answer
- Reflect node examines answer
- Decision node evaluates if revision needed
- Revise node improves answer (if needed)
- Loop: generate → reflect → (revise → generate) or (end)

**State structure:**
```python
class ReflectionState(TypedDict):
    question: str
    answer: str
    reflection: str
    needs_revision: bool
    iteration: int
```

**Expected behavior:**
- Model generates answer
- Model reflects on answer quality
- If reflection indicates issues, model revises
- Process repeats until reflection is positive or max iterations

---

### Task 6: Full Critique-Revision Workflow

**Objective**: Implement complete reflection-critique-revision pattern.

**What to do:**
1. Create `task6_critique_revision.py`
2. Build a graph that:
   - Generates an answer
   - Reflects on the answer
   - Critiques the answer (specific issues)
   - Evaluates if critique indicates need for revision
   - Revises answer based on critique
   - Loops until quality is good or max revisions
3. Use LLM for all steps

**Requirements:**
- Generate → Reflect → Critique → Evaluate → (Revise → Generate) or (End)
- Critique should be specific and actionable
- Revision should address critique points
- Track revision count
- Max 3 revisions

**State structure:**
```python
class CritiqueState(TypedDict):
    question: str
    answer: str
    reflection: str
    critique: str
    quality_score: int
    revision_count: int
```

**Expected behavior:**
- Complete cycle: generate, reflect, critique, evaluate, (optionally) revise
- Critique provides specific feedback
- Revision addresses critique
- Process improves answer iteratively
- Stops when quality is acceptable

---

## Expected Output (What You Should See)

After completing these tasks, you should have:

1. **Task 1**: A working conditional routing graph
   - Routes correctly based on input values
   - Different handlers execute based on conditions
   - Final state shows which path was taken

2. **Task 2**: An LLM-based router
   - Classification works correctly
   - Routing goes to appropriate expert
   - Expert provides specialized answer

3. **Task 3**: A retry loop
   - Operation retries on failure
   - Stops after success or max retries
   - Attempt count is tracked

4. **Task 4**: A refinement loop
   - Answer quality improves with iterations
   - Loop stops when quality is good
   - Iteration count is tracked

5. **Task 5**: Reflection pattern
   - Model reflects on its output
   - Revision happens when needed
   - Final answer is improved

6. **Task 6**: Complete critique-revision workflow
   - Full cycle: generate → reflect → critique → evaluate → revise
   - Critique is specific and helpful
   - Revision addresses critique
   - Answer quality improves iteratively

**Code quality check:**
- ✅ All graphs compile and run
- ✅ Conditional routing works correctly
- ✅ Loops have proper exit conditions
- ✅ Iteration counts are tracked
- ✅ No infinite loops
- ✅ Error handling is in place
- ✅ Code is readable and well-commented

> ❗ **Tips**: 
> - Always add max iteration limits to prevent infinite loops
> - Test conditional routing with different inputs
> - Use logging to see which path your graph takes
> - Start with simple conditions, then add complexity
> - Verify loops actually iterate (check iteration count in final state)


# Day 5 — Student Assignment

## Instructions

Today you'll practice using persistence and reducers! These tasks map to the three Notes you just read:

- **Tasks 1-2** relate to **Note 1: Persistence and Checkpointing**
- **Tasks 3-4** relate to **Note 2: Reducers**
- **Tasks 5-6** relate to **Note 3: Using Persistence + Reducers Together**

You'll be working with checkpoints, state merging, and building reliable workflows. Make sure you understand how persistence and reducers work before starting.

---

## Tasks

### Task 1: Enable Persistence in an Existing Graph

**Objective**: Add persistence to a graph you've built before.

**What to do:**
1. Take any graph from previous days (e.g., Day 2's single-node graph, Day 3's sequential graph)
2. Create `task1_persistence.py`
3. Enable persistence using `MemorySaver` (for simplicity)
4. Run the graph with a thread ID
5. Inspect the checkpoints to see saved state

**Requirements:**
- Import `MemorySaver` from `langgraph.checkpoint.memory`
- Compile graph with checkpointer
- Use thread ID in config when invoking
- List and inspect checkpoints after execution

**Expected behavior:**
- Graph runs normally
- Checkpoints are saved after each node
- You can inspect checkpoint history
- State is preserved in checkpoints

---

### Task 2: Resume from Checkpoint

**Objective**: Practice resuming execution from a saved checkpoint.

**What to do:**
1. Create `task2_resume.py`
2. Build a simple 3-node sequential graph
3. Enable persistence
4. Run the graph partway (stop after 2 nodes)
5. Resume execution from the checkpoint
6. Verify it continues from where it left off

**Requirements:**
- Graph should have at least 3 nodes
- Use thread ID consistently
- Manually stop/resume (or simulate failure)
- Show that state is preserved

**Hint**: You can "stop" by catching an exception or by designing the graph to pause at a certain point.

**Expected behavior:**
- First execution runs 2 nodes, saves checkpoint
- Resume execution continues from node 3
- Final state includes results from all nodes

---

### Task 3: Implement a Simple Reducer

**Objective**: Create a custom reducer for state merging.

**What to do:**
1. Create `task3_reducer.py`
2. Build a graph where multiple nodes update the same state field
3. Define a custom reducer (e.g., sum, append, merge)
4. Use `Annotated` with `reduce()` in your state definition
5. Verify that state merges correctly

**Requirements:**
- Define a custom reducer function
- Use it in state definition with `Annotated[type, reduce(reducer_func)]`
- Multiple nodes should update the same field
- Verify merging works as expected

**Example reducer ideas:**
- Sum reducer for numbers
- Append reducer for lists
- Merge reducer for dictionaries
- Max/min reducer for scores

**Expected behavior:**
- Multiple nodes update the same state field
- Reducer combines updates correctly
- Final state shows merged result

---

### Task 4: Reducer for Parallel Execution

**Objective**: Use reducers to combine results from parallel nodes.

**What to do:**
1. Create `task4_parallel_reducer.py`
2. Build a graph with 3 parallel nodes
3. Each parallel node adds results to a shared list/dict
4. Use a reducer to merge parallel results
5. A combine node processes the merged results

**Requirements:**
- At least 3 parallel nodes
- Each node updates a shared state field
- Reducer merges parallel updates
- Combine node receives merged state

**State structure example:**
```python
class ParallelState(TypedDict):
    all_results: Annotated[list, reduce(append_reducer)]
    query: str
```

**Expected behavior:**
- Parallel nodes execute simultaneously
- Each adds to shared state field
- Reducer merges all updates
- Combine node receives all results

---

### Task 5: Persistent Iterative Workflow with Reducer

**Objective**: Combine persistence and reducers in an iterative workflow.

**What to do:**
1. Create `task5_persistent_iterative.py`
2. Build an iterative workflow (loop) that:
   - Tracks attempts in a list (using reducer)
   - Accumulates a score (using reducer)
   - Has persistence enabled
3. Run the workflow
4. Inspect checkpoints to see how state accumulates

**Requirements:**
- Iterative loop (generate → check → refine or end)
- Reducer for accumulating attempts
- Reducer for accumulating scores
- Persistence enabled
- Inspect checkpoints after execution

**State structure:**
```python
class IterativeState(TypedDict):
    attempts: Annotated[list, reduce(append_reducer)]
    total_score: Annotated[float, reduce(sum_reducer)]
    current_result: str
    iteration: int
```

**Expected behavior:**
- Loop runs multiple iterations
- Attempts accumulate in list
- Score accumulates
- Checkpoints show state evolution
- Can resume from any checkpoint

---

### Task 6: Debug a Workflow Using Checkpoints

**Objective**: Practice debugging by inspecting checkpoints.

**What to do:**
1. Create `task6_debug_checkpoints.py`
2. Build a workflow that might have issues (intentionally or not)
3. Enable persistence
4. Run the workflow
5. Inspect checkpoints to:
   - See state at each step
   - Identify where issues occurred
   - Understand execution flow
6. Fix any issues you find

**Requirements:**
- Workflow with multiple nodes
- Persistence enabled
- Checkpoint inspection code
- Analysis of execution history
- (Optional) Fix and re-run

**What to inspect:**
- State values at each checkpoint
- Which nodes executed
- How state changed between steps
- Any error states

**Expected output:**
- List of all checkpoints
- State at each checkpoint
- Analysis of execution flow
- Identification of any issues

---

## Expected Output (What You Should See)

After completing these tasks, you should have:

1. **Task 1**: A graph with persistence enabled
   - Checkpoints are saved
   - You can list and inspect checkpoints
   - State is preserved in checkpoints

2. **Task 2**: Ability to resume from checkpoints
   - Execution can be paused/resumed
   - State is preserved between runs
   - Workflow continues correctly

3. **Task 3**: Custom reducer implementation
   - Reducer function defined
   - Used in state definition
   - State merges correctly

4. **Task 4**: Parallel execution with reducer
   - Parallel nodes update shared state
   - Reducer merges parallel updates
   - Combine node receives all results

5. **Task 5**: Persistent iterative workflow
   - Loop runs with persistence
   - Reducers accumulate state correctly
   - Checkpoints show state evolution
   - Can inspect execution history

6. **Task 6**: Debugging with checkpoints
   - Can inspect all checkpoints
   - Understand execution flow
   - Identify issues in state/execution
   - (Optional) Fixed issues

**Code quality check:**
- ✅ Persistence is enabled correctly
- ✅ Thread IDs are used consistently
- ✅ Reducers are defined and used properly
- ✅ Checkpoints can be inspected
- ✅ State merging works as expected
- ✅ Code is readable and well-commented

> ❗ **Tips**: 
> - Start with `MemorySaver` for simplicity (no file setup needed)
> - Use consistent thread IDs to maintain state across invocations
> - Test reducers with simple cases first
> - Inspect checkpoints to understand state evolution
> - Reducers are especially important for parallel and iterative workflows


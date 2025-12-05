# Day 4 — Mini Project

## Project Overview

**Project Name**: Smart Router Assistant

In this project, you'll build an intelligent assistant that routes questions to different expert nodes based on intent, and includes a quality improvement loop. This combines conditional routing with iterative refinement—two powerful patterns you've learned today.

**What the project does:**
- Takes a user question
- Classifies the intent (code, writing, or general)
- Routes to the appropriate expert
- Expert generates an answer
- Checks answer quality
- If quality is low, routes to an "Improve Answer" node
- Loops until quality is acceptable or max iterations reached

**Why this workflow is useful:**
- Demonstrates conditional routing in a practical scenario
- Shows iterative improvement for quality control
- Combines multiple patterns (routing + loops)
- Real-world pattern for production QA systems
- Self-improving system that ensures quality

---

## Project Requirements

Your Smart Router Assistant must:

1. **Classify user intent** using an LLM (code, writing, or general)
2. **Route conditionally** to one of three expert nodes:
   - Code questions → Dev Expert
   - Writing questions → Writing Coach
   - General questions → General Assistant
3. **Generate answer** from the appropriate expert
4. **Check answer quality** (length, completeness, relevance)
5. **Improve answer** if quality is low (up to 2 improvements)
6. **Return final answer** when quality is acceptable

**Constraints:**
- Must use conditional routing (not just if-statements in code)
- Must have an iterative improvement loop
- Max 3 total iterations (initial + 2 improvements)
- All LLM calls should use the same model
- Include error handling

**Optional enhancements:**
- Add logging to show routing decisions
- Track which expert was used
- Show quality scores in output
- Add a "fallback" expert if classification fails

---

## Step-by-Step Instructions

### Step 1: Set Up Your Project

1. Create a new file: `day04_project.py`

2. Import required modules:
```python
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI
from typing import TypedDict
from dotenv import load_dotenv
import os

load_dotenv()
```

3. Initialize LLM:
```python
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0.7)
```

---

### Step 2: Define Your State

Create a TypedDict that captures all necessary data:

```python
class RouterAssistantState(TypedDict):
    question: str
    intent: str
    answer: str
    quality_score: float
    needs_improvement: bool
    improvement_count: int
    expert_used: str
    final_answer: str
```

---

### Step 3: Create the Classification Node

This node classifies user intent:

```python
def classify_intent(state: RouterAssistantState) -> RouterAssistantState:
    """
    Classifies the user's question intent.
    """
    print("[CLASSIFY] Analyzing question intent...")
    
    try:
        question = state.get("question", "")
        if not question:
            return {"intent": "general", "expert_used": "general"}
        
        prompt = f"""
        Classify this question into exactly one category: code, writing, or general.
        
        Question: {question}
        
        Respond with only one word: code, writing, or general.
        """
        
        response = llm.invoke(prompt)
        intent = response.content.strip().lower()
        
        # Normalize intent
        if "code" in intent or "programming" in intent:
            intent = "code"
        elif "writing" in intent or "write" in intent:
            intent = "writing"
        else:
            intent = "general"
        
        print(f"[CLASSIFY] Intent classified as: {intent}")
        return {"intent": intent}
        
    except Exception as e:
        print(f"[ERROR] Classification failed: {e}")
        return {"intent": "general", "expert_used": "general"}
```

---

### Step 4: Create Routing Function

This function decides which expert to use:

```python
def route_to_expert(state: RouterAssistantState) -> str:
    """
    Routes to the appropriate expert based on intent.
    """
    intent = state.get("intent", "general")
    
    if intent == "code":
        return "dev_expert"
    elif intent == "writing":
        return "writing_coach"
    else:
        return "general_assistant"
```

---

### Step 5: Create Expert Nodes

**Dev Expert:**
```python
def dev_expert(state: RouterAssistantState) -> RouterAssistantState:
    """
    Handles code-related questions.
    """
    print("[EXPERT] Dev Expert processing...")
    
    try:
        question = state.get("question", "")
        prompt = f"""
        As a coding expert, provide a clear, helpful answer with code examples if relevant.
        
        Question: {question}
        """
        
        response = llm.invoke(prompt)
        answer = response.content
        
        print(f"[EXPERT] Dev Expert answer generated ({len(answer)} chars)")
        return {
            "answer": answer,
            "expert_used": "dev_expert"
        }
        
    except Exception as e:
        print(f"[ERROR] Dev Expert failed: {e}")
        return {"answer": "Error generating answer", "expert_used": "dev_expert"}
```

**Writing Coach:**
```python
def writing_coach(state: RouterAssistantState) -> RouterAssistantState:
    """
    Handles writing-related questions.
    """
    print("[EXPERT] Writing Coach processing...")
    
    try:
        question = state.get("question", "")
        prompt = f"""
        As a writing coach, provide helpful writing advice and examples.
        
        Question: {question}
        """
        
        response = llm.invoke(prompt)
        answer = response.content
        
        print(f"[EXPERT] Writing Coach answer generated ({len(answer)} chars)")
        return {
            "answer": answer,
            "expert_used": "writing_coach"
        }
        
    except Exception as e:
        print(f"[ERROR] Writing Coach failed: {e}")
        return {"answer": "Error generating answer", "expert_used": "writing_coach"}
```

**General Assistant:**
```python
def general_assistant(state: RouterAssistantState) -> RouterAssistantState:
    """
    Handles general questions.
    """
    print("[EXPERT] General Assistant processing...")
    
    try:
        question = state.get("question", "")
        prompt = f"""
        Answer this question helpfully and clearly.
        
        Question: {question}
        """
        
        response = llm.invoke(prompt)
        answer = response.content
        
        print(f"[EXPERT] General Assistant answer generated ({len(answer)} chars)")
        return {
            "answer": answer,
            "expert_used": "general_assistant"
        }
        
    except Exception as e:
        print(f"[ERROR] General Assistant failed: {e}")
        return {"answer": "Error generating answer", "expert_used": "general_assistant"}
```

---

### Step 6: Create Quality Check Node

This evaluates answer quality:

```python
def check_quality(state: RouterAssistantState) -> RouterAssistantState:
    """
    Checks if the answer quality is acceptable.
    """
    print("[QUALITY] Checking answer quality...")
    
    try:
        answer = state.get("answer", "")
        question = state.get("question", "")
        
        # Simple quality metrics
        length_score = min(len(answer) / 200, 1.0)  # Prefer longer answers
        has_content = 1.0 if len(answer.strip()) > 50 else 0.0
        relevance = 1.0 if question.lower() in answer.lower() or len(answer) > 100 else 0.5
        
        quality = (length_score * 0.4 + has_content * 0.3 + relevance * 0.3)
        
        needs_improvement = quality < 0.6
        
        print(f"[QUALITY] Quality score: {quality:.2f}, Needs improvement: {needs_improvement}")
        
        return {
            "quality_score": quality,
            "needs_improvement": needs_improvement
        }
        
    except Exception as e:
        print(f"[ERROR] Quality check failed: {e}")
        return {"quality_score": 0.5, "needs_improvement": True}
```

---

### Step 7: Create Improvement Node

This improves the answer if quality is low:

```python
def improve_answer(state: RouterAssistantState) -> RouterAssistantState:
    """
    Improves the answer based on quality feedback.
    """
    print("[IMPROVE] Improving answer...")
    
    try:
        question = state.get("question", "")
        current_answer = state.get("answer", "")
        quality_score = state.get("quality_score", 0)
        
        prompt = f"""
        Improve this answer to be more comprehensive, detailed, and helpful.
        Current quality issues: Answer may be too short or incomplete.
        
        Original Question: {question}
        Current Answer: {current_answer}
        
        Provide an improved, more detailed answer.
        """
        
        response = llm.invoke(prompt)
        improved_answer = response.content
        
        improvement_count = state.get("improvement_count", 0) + 1
        
        print(f"[IMPROVE] Answer improved (iteration {improvement_count})")
        
        return {
            "answer": improved_answer,
            "improvement_count": improvement_count
        }
        
    except Exception as e:
        print(f"[ERROR] Improvement failed: {e}")
        return {"improvement_count": state.get("improvement_count", 0) + 1}
```

---

### Step 8: Create Decision Function

This decides whether to improve or finish:

```python
def should_improve(state: RouterAssistantState) -> str:
    """
    Decides whether to improve the answer or finish.
    """
    needs_improvement = state.get("needs_improvement", False)
    improvement_count = state.get("improvement_count", 0)
    
    if not needs_improvement:
        return "finish"
    
    if improvement_count >= 2:  # Max 2 improvements
        return "finish"
    
    return "improve"
```

---

### Step 9: Create Finalize Node

This prepares the final output:

```python
def finalize_answer(state: RouterAssistantState) -> RouterAssistantState:
    """
    Prepares the final answer for return.
    """
    print("[FINALIZE] Preparing final answer...")
    
    final_answer = state.get("answer", "")
    expert = state.get("expert_used", "unknown")
    quality = state.get("quality_score", 0)
    improvements = state.get("improvement_count", 0)
    
    print(f"[FINALIZE] Expert: {expert}, Quality: {quality:.2f}, Improvements: {improvements}")
    
    return {"final_answer": final_answer}
```

---

### Step 10: Build Your Graph

Connect all nodes:

```python
# Create graph
graph = StateGraph(RouterAssistantState)

# Add nodes
graph.add_node("classify", classify_intent)
graph.add_node("dev_expert", dev_expert)
graph.add_node("writing_coach", writing_coach)
graph.add_node("general_assistant", general_assistant)
graph.add_node("check_quality", check_quality)
graph.add_node("improve", improve_answer)
graph.add_node("finalize", finalize_answer)

# Set entry point
graph.set_entry_point("classify")

# Route to experts
graph.add_conditional_edges("classify", route_to_expert)

# All experts check quality
graph.add_edge("dev_expert", "check_quality")
graph.add_edge("writing_coach", "check_quality")
graph.add_edge("general_assistant", "check_quality")

# Quality check routes to improve or finish
graph.add_conditional_edges("check_quality", should_improve, {
    "improve": "improve",
    "finish": "finalize"
})

# Improvement loops back to quality check
graph.add_edge("improve", "check_quality")

# Finalize exits
graph.add_edge("finalize", END)

# Compile
app = graph.compile()
```

---

### Step 11: Test Your Workflow

Test with different question types:

```python
# Test code question
result1 = app.invoke({
    "question": "How do I reverse a list in Python?",
    "improvement_count": 0
})
print("\n" + "="*50)
print("CODE QUESTION RESULT:")
print("="*50)
print(f"Expert: {result1['expert_used']}")
print(f"Quality: {result1['quality_score']:.2f}")
print(f"Improvements: {result1['improvement_count']}")
print(f"Answer: {result1['final_answer']}")

# Test writing question
result2 = app.invoke({
    "question": "How do I write a compelling introduction?",
    "improvement_count": 0
})
print("\n" + "="*50)
print("WRITING QUESTION RESULT:")
print("="*50)
print(f"Expert: {result2['expert_used']}")
print(f"Answer: {result2['final_answer']}")

# Test general question
result3 = app.invoke({
    "question": "What is machine learning?",
    "improvement_count": 0
})
print("\n" + "="*50)
print("GENERAL QUESTION RESULT:")
print("="*50)
print(f"Expert: {result3['expert_used']}")
print(f"Answer: {result3['final_answer']}")
```

---

## Example Interaction / Output

**Input Example:**
```python
result = app.invoke({
    "question": "How do I use list comprehensions in Python?",
    "improvement_count": 0
})
```

**Expected Behavior:**

1. **Classification**: Question classified as "code"
2. **Routing**: Routes to Dev Expert
3. **Answer Generation**: Dev Expert generates answer
4. **Quality Check**: Evaluates answer quality
5. **Decision**: If quality low, routes to Improve
6. **Improvement**: Answer is improved (if needed)
7. **Loop**: Quality checked again
8. **Finalization**: Final answer prepared

**Console Output:**
```
[CLASSIFY] Analyzing question intent...
[CLASSIFY] Intent classified as: code
[EXPERT] Dev Expert processing...
[EXPERT] Dev Expert answer generated (450 chars)
[QUALITY] Checking answer quality...
[QUALITY] Quality score: 0.65, Needs improvement: False
[FINALIZE] Preparing final answer...
[FINALIZE] Expert: dev_expert, Quality: 0.65, Improvements: 0
```

**Output Example:**
```python
{
    "question": "How do I use list comprehensions in Python?",
    "intent": "code",
    "expert_used": "dev_expert",
    "quality_score": 0.65,
    "improvement_count": 0,
    "final_answer": "[Detailed answer about list comprehensions...]"
}
```

**If Quality is Low:**
```
[QUALITY] Quality score: 0.45, Needs improvement: True
[IMPROVE] Improving answer...
[IMPROVE] Answer improved (iteration 1)
[QUALITY] Checking answer quality...
[QUALITY] Quality score: 0.72, Needs improvement: False
[FINALIZE] Preparing final answer...
```

---

## Bonus Challenges (Optional)

1. **Add Expert Selection Feedback**: Show why a particular expert was chosen.

2. **Improve Quality Metrics**: Use an LLM to evaluate quality instead of simple heuristics.

3. **Add Fallback Expert**: If classification fails or is unclear, route to a general expert.

4. **Track Improvement History**: Store all versions of the answer to show how it improved.

5. **Add User Feedback Loop**: Allow user to rate answer quality and trigger another improvement.

6. **Multi-Expert Consultation**: Route to multiple experts and combine their answers.

---

## Tips & Hints

- **Test routing**: Verify each expert is called for the right question types
- **Check loop limits**: Make sure improvement loop doesn't run forever
- **Quality metrics**: Adjust quality thresholds based on your needs
- **Logging is key**: Use print statements to trace execution
- **Handle edge cases**: What if classification fails? What if answer is empty?
- **Start simple**: Get basic routing working, then add improvement loop

**Common Issues:**

- **Infinite loops**: Make sure improvement_count is tracked and limits are enforced
- **Routing not working**: Check that conditional edges return correct node names
- **Quality always low**: Adjust quality metrics to be more realistic
- **State not updating**: Verify nodes return state updates correctly

---

## Submission Checklist

Before moving to Day 5, make sure you have:

- [ ] Working classification node
- [ ] Conditional routing to three experts
- [ ] Three expert nodes that generate answers
- [ ] Quality check node
- [ ] Improvement node
- [ ] Iterative loop that improves answers
- [ ] Loop exit conditions (max iterations, quality threshold)
- [ ] Finalization node
- [ ] Error handling throughout
- [ ] Logging that shows execution flow
- [ ] Graph compiles and runs without errors
- [ ] Tested with different question types
- [ ] (Optional) Bonus features implemented

**File to submit:**
- `day04_project.py` - Your complete Smart Router Assistant

**What to demonstrate:**
- Classification works correctly
- Routing goes to appropriate expert
- Quality check evaluates answers
- Improvement loop works when quality is low
- Loop exits correctly (no infinite loops)
- Final answer is well-formatted

Excellent work! You've built a sophisticated routing and improvement system. In Day 5, you'll learn about persistence and reducers for production-ready workflows! 🎉


# Day 3 — Mini Project

## Project Overview

**Project Name**: Summarize & Analyze Workflow

In this project, you'll build a complete workflow that combines sequential and parallel execution patterns. The workflow will summarize an article, then run multiple analyses in parallel, and finally combine everything into a structured report.

**What the project does:**
- Takes an article or long text as input
- Summarizes it using an LLM (sequential step)
- Runs three parallel analyses on the summary:
  - Extracts key keywords
  - Generates discussion questions
  - Identifies main topics
- Combines all results into a final structured report

**Why this workflow is useful:**
- This pattern appears in many real-world GenAI applications
- Demonstrates both sequential and parallel execution
- Shows how to combine multiple analyses effectively
- Useful for content analysis, research, and document processing

---

## Project Requirements

Your workflow must:

1. **Sequential Step**: Summarize the input article first
2. **Parallel Steps**: After summarization, run three analyses simultaneously:
   - Extract 5-10 keywords
   - Generate 3-5 discussion questions
   - Identify 3-5 main topics
3. **Combination Step**: Merge all results into a structured final report
4. **Error Handling**: Handle missing input, LLM errors, etc.
5. **Logging**: Show execution progress (which step is running)

**Constraints:**
- Must use at least one sequential pattern (summarize first)
- Must use parallel execution for the three analyses
- Must combine parallel results into final output
- All LLM calls should use the same model (for consistency)

**Optional enhancements:**
- Add a validation step before summarization
- Format the final report nicely (markdown, structured text)
- Add metadata (processing time, model used)
- Allow customization (number of keywords, questions, etc.)

---

## Step-by-Step Instructions

### Step 1: Set Up Your Project

1. Create a new file: `day03_project.py`

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

Create a TypedDict that captures all the data flowing through your workflow.

**Required fields:**
- `article`: Original article text
- `summary`: Generated summary
- `keywords`: Extracted keywords
- `questions`: Generated questions
- `topics`: Identified topics
- `final_report`: Combined final output

**Optional fields:**
- `error`: Error messages
- `processing_time`: Time taken

```python
class ArticleAnalysisState(TypedDict):
    article: str
    summary: str
    keywords: str
    questions: str
    topics: str
    final_report: str
    # Add optional fields as needed
```

---

### Step 3: Create the Summarize Node (Sequential)

This is the first step that runs before parallel execution.

```python
def summarize_article(state: ArticleAnalysisState) -> ArticleAnalysisState:
    """
    Summarizes the input article.
    This runs first, before parallel analyses.
    """
    print("[STEP 1] Summarizing article...")
    
    try:
        article = state.get("article", "")
        if not article:
            return {"error": "No article provided"}
        
        prompt = f"""
        Summarize the following article in 2-3 paragraphs:
        
        {article}
        """
        
        response = llm.invoke(prompt)
        summary = response.content
        
        print(f"[STEP 1] Summary generated ({len(summary)} characters)")
        return {"summary": summary}
        
    except Exception as e:
        print(f"[ERROR] Summarization failed: {e}")
        return {"error": f"Summarization error: {str(e)}"}
```

---

### Step 4: Create Three Parallel Analysis Nodes

These will run simultaneously after summarization.

**Node 1: Extract Keywords**
```python
def extract_keywords(state: ArticleAnalysisState) -> ArticleAnalysisState:
    """
    Extracts key keywords from the summary.
    Runs in parallel with other analyses.
    """
    print("[PARALLEL] Extracting keywords...")
    
    try:
        summary = state.get("summary", "")
        if not summary:
            return {"keywords": "N/A - No summary available"}
        
        prompt = f"""
        Extract 5-10 key keywords from this summary. 
        Return them as a comma-separated list:
        
        {summary}
        """
        
        response = llm.invoke(prompt)
        keywords = response.content.strip()
        
        print(f"[PARALLEL] Keywords extracted: {keywords[:50]}...")
        return {"keywords": keywords}
        
    except Exception as e:
        print(f"[ERROR] Keyword extraction failed: {e}")
        return {"keywords": f"Error: {str(e)}"}
```

**Node 2: Generate Questions**
```python
def generate_questions(state: ArticleAnalysisState) -> ArticleAnalysisState:
    """
    Generates discussion questions from the summary.
    Runs in parallel with other analyses.
    """
    print("[PARALLEL] Generating questions...")
    
    try:
        summary = state.get("summary", "")
        if not summary:
            return {"questions": "N/A - No summary available"}
        
        prompt = f"""
        Generate 3-5 thoughtful discussion questions based on this summary.
        Format each question on a new line with a number:
        
        {summary}
        """
        
        response = llm.invoke(prompt)
        questions = response.content.strip()
        
        print(f"[PARALLEL] Questions generated")
        return {"questions": questions}
        
    except Exception as e:
        print(f"[ERROR] Question generation failed: {e}")
        return {"questions": f"Error: {str(e)}"}
```

**Node 3: Identify Topics**
```python
def identify_topics(state: ArticleAnalysisState) -> ArticleAnalysisState:
    """
    Identifies main topics from the summary.
    Runs in parallel with other analyses.
    """
    print("[PARALLEL] Identifying topics...")
    
    try:
        summary = state.get("summary", "")
        if not summary:
            return {"topics": "N/A - No summary available"}
        
        prompt = f"""
        Identify 3-5 main topics from this summary.
        Return them as a comma-separated list:
        
        {summary}
        """
        
        response = llm.invoke(prompt)
        topics = response.content.strip()
        
        print(f"[PARALLEL] Topics identified: {topics[:50]}...")
        return {"topics": topics}
        
    except Exception as e:
        print(f"[ERROR] Topic identification failed: {e}")
        return {"topics": f"Error: {str(e)}"}
```

---

### Step 5: Create the Combine Node (Fan-In)

This node combines all parallel results into a final report.

```python
def create_final_report(state: ArticleAnalysisState) -> ArticleAnalysisState:
    """
    Combines all analyses into a structured final report.
    This runs after all parallel nodes complete.
    """
    print("[FINAL] Creating final report...")
    
    try:
        summary = state.get("summary", "N/A")
        keywords = state.get("keywords", "N/A")
        questions = state.get("questions", "N/A")
        topics = state.get("topics", "N/A")
        
        report = f"""
# Article Analysis Report

## Summary
{summary}

## Key Keywords
{keywords}

## Discussion Questions
{questions}

## Main Topics
{topics}
"""
        
        print("[FINAL] Report created successfully")
        return {"final_report": report.strip()}
        
    except Exception as e:
        print(f"[ERROR] Report creation failed: {e}")
        return {"final_report": f"Error creating report: {str(e)}"}
```

---

### Step 6: Build Your Graph

Now connect all nodes in the correct order.

```python
# Create graph
graph = StateGraph(ArticleAnalysisState)

# Add nodes
graph.add_node("summarize", summarize_article)
graph.add_node("extract_keywords", extract_keywords)
graph.add_node("generate_questions", generate_questions)
graph.add_node("identify_topics", identify_topics)
graph.add_node("create_report", create_final_report)

# Sequential: Start with summarize
graph.set_entry_point("summarize")

# After summarize, route to all three parallel analyses
def route_to_analyses(state):
    return ["extract_keywords", "generate_questions", "identify_topics"]

graph.add_conditional_edges("summarize", route_to_analyses)

# All three parallel nodes feed into combine
graph.add_edge("extract_keywords", "create_report")
graph.add_edge("generate_questions", "create_report")
graph.add_edge("identify_topics", "create_report")

# Final node exits
graph.add_edge("create_report", END)

# Compile
app = graph.compile()
```

---

### Step 7: Test Your Workflow

Test with a sample article:

```python
sample_article = """
Artificial Intelligence (AI) has transformed numerous industries, 
from healthcare to finance. Machine learning algorithms can now 
diagnose diseases, predict market trends, and even drive cars. 
However, this rapid advancement raises important ethical questions 
about privacy, bias, and job displacement. As AI becomes more 
pervasive, society must grapple with how to harness its benefits 
while mitigating its risks.
"""

result = app.invoke({"article": sample_article})
print("\n" + "="*50)
print("FINAL REPORT:")
print("="*50)
print(result["final_report"])
```

---

### Step 8: Add Streaming (Optional)

See execution in real-time:

```python
print("Executing workflow with streaming...")
for chunk in app.stream({"article": sample_article}):
    node_name = list(chunk.keys())[0]
    print(f"\nCompleted: {node_name}")
```

---

## Example Interaction / Output

**Input Example:**
```python
article = """
[Your article text here - can be any length]
"""

result = app.invoke({"article": article})
```

**Expected Behavior:**

1. **Sequential Step**: Summarize runs first
   - Logs: `[STEP 1] Summarizing article...`
   - Generates summary

2. **Parallel Steps**: Three analyses run simultaneously
   - Logs: `[PARALLEL] Extracting keywords...`
   - Logs: `[PARALLEL] Generating questions...`
   - Logs: `[PARALLEL] Identifying topics...`
   - All three complete around the same time

3. **Combination Step**: Final report is created
   - Logs: `[FINAL] Creating final report...`
   - Combines all results

**Output Example (Format):**

```
# Article Analysis Report

## Summary
[2-3 paragraph summary of the article]

## Key Keywords
keyword1, keyword2, keyword3, keyword4, keyword5

## Discussion Questions
1. [Question 1]
2. [Question 2]
3. [Question 3]

## Main Topics
topic1, topic2, topic3, topic4
```

**Console Output:**
```
[STEP 1] Summarizing article...
[STEP 1] Summary generated (450 characters)
[PARALLEL] Extracting keywords...
[PARALLEL] Generating questions...
[PARALLEL] Identifying topics...
[PARALLEL] Keywords extracted: AI, machine learning, ethics...
[PARALLEL] Questions generated
[PARALLEL] Topics identified: Artificial Intelligence, Ethics...
[FINAL] Creating final report...
[FINAL] Report created successfully
```

---

## Bonus Challenges (Optional)

1. **Add Validation**: Create a node that validates the article (length, content, etc.) before summarization.

2. **Improve Combination**: Use an LLM to intelligently synthesize the parallel results instead of simple concatenation.

3. **Add Metrics**: Track processing time for each step and include in the final report.

4. **Customizable Analysis**: Allow users to specify how many keywords/questions/topics to generate.

5. **Error Recovery**: If one parallel analysis fails, still generate a report with the successful analyses.

6. **Multiple Articles**: Extend to process multiple articles and compare them.

---

## Tips & Hints

- **Start simple**: Get the sequential summarize step working first
- **Test parallel execution**: Use `time.time()` to verify nodes run simultaneously
- **Handle missing summary**: Parallel nodes should handle cases where summary might be missing
- **Use streaming**: `app.stream()` helps you see the execution order
- **Check state merging**: Verify that all parallel results appear in state before the combine node
- **Add delays for testing**: Use `time.sleep()` in parallel nodes to see they run simultaneously

**Common Issues:**

- **Parallel nodes not running**: Make sure you're using conditional edges or multiple edges correctly
- **Missing state in combine**: Check that parallel nodes return state updates correctly
- **Sequential before parallel**: Ensure summarize runs first (set as entry point, route from it)
- **State conflicts**: Make sure parallel nodes update different state keys

---

## Submission Checklist

Before moving to Day 4, make sure you have:

- [ ] Working sequential summarize step
- [ ] Three parallel analysis nodes that run simultaneously
- [ ] Combine node that merges all results
- [ ] Error handling for all nodes
- [ ] Logging that shows execution progress
- [ ] Graph compiles and runs without errors
- [ ] Final report is well-formatted and complete
- [ ] Tested with multiple article inputs
- [ ] (Optional) Bonus features implemented

**File to submit:**
- `day03_project.py` - Your complete workflow

**What to demonstrate:**
- Sequential step runs first
- Parallel steps execute simultaneously (check logs/timing)
- All results are combined correctly
- Final report is comprehensive and well-formatted

Excellent work! You've built a sophisticated workflow combining sequential and parallel patterns. In Day 4, you'll learn conditional routing and loops! 🎉


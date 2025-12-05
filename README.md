# LangGraph — 5-Day Beginner Roadmap

> A fast, structured LangGraph foundation designed specifically for GenAI developers who already know Python and want to build reliable AI workflows.

---

## How to Use This Repository

This bootcamp is designed to be completed **day by day** (Day 1 → Day 5). Each day builds on the previous one, so make sure you understand the concepts before moving forward.

### Daily Workflow

Within each day folder, follow this order:

1. **Start with `01_notes.md`**
   - Read each topic-wise Note section carefully
   - Take your time to understand the concepts
   - Try running any code examples provided

2. **Then attempt `02_student_task.md`**
   - Complete the tasks that map to each Note/topic
   - These are designed to reinforce what you learned
   - Don't skip ahead—each task builds understanding

3. **Finally, work on `03_mini_project.md`**
   - This is your hands-on project for the day
   - Apply everything you've learned
   - Experiment and iterate!

**Pro tip:** Practice examples as you go; don't just read. The best way to learn LangGraph is by building graphs yourself.

---

## Technical Requirements

### Python Version
- **Python 3.10+** (required for modern LangGraph features)

### Required Libraries

Install the following packages:

```bash
pip install langgraph langchain openai python-dotenv
```

Or if you prefer using a requirements file:

```txt
langgraph>=0.2.0
langchain>=0.1.0
openai>=1.0.0
python-dotenv>=1.0.0
```

### Environment Setup

1. Create a `.env` file in the root directory
2. Add your API keys:

```env
OPENAI_API_KEY=your_api_key_here
```

3. Load environment variables in your Python scripts:

```python
from dotenv import load_dotenv
load_dotenv()
```

### Running Your Graphs

Simply run your Python files:

```bash
python day02_project.py
```

Make sure your API keys are set up before running any code that uses LLMs.

---

## Roadmap Overview

| Day | Folder Name                                      | Focus Topics                                      |
|-----|--------------------------------------------------|---------------------------------------------------|
| 1   | Why LangGraph & Core Concepts           | Why LangGraph, graph mindset, core concepts       |
| 2   | Execution Model & Single-Node Graphs    | Execution model, basic single-node graphs         |
| 3   | Sequential & Parallel Workflows         | Sequential flows, parallel branches, fan-in       |
| 4   | Conditional & Iterative Workflows       | Routing, loops, retries, reflection patterns      |
| 5   | Persistence & Reducers in LangGraph     | Persistence, checkpointing, reducers, reliability |

---

## What You'll Learn

By the end of this 5-day bootcamp, you'll be able to:

- ✅ Understand why LangGraph is powerful for building AI workflows
- ✅ Build and execute single-node and multi-node graphs
- ✅ Create sequential and parallel workflows
- ✅ Implement conditional routing and iterative loops
- ✅ Use persistence and reducers for reliable, production-ready workflows
- ✅ Debug and observe your LangGraph applications

---

## Credits

Created by **Chandra Sekhar**.

---

## Getting Started

Ready to begin? Navigate to **Day01 - Why LangGraph & Core Concepts** and open `01_notes.md`. Let's build some amazing AI workflows! 🚀


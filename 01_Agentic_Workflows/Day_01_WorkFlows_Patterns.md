# Day 3: Building Effective Agents — Architectures, Workflows & Production Patterns

![Anthropic graphic](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2Fd3083d3f40bb2b6f477901cc9a240738d3dd1371-2401x1000.png&w=3840&q=75)

## 📌 Overview / TL;DR
The most successful production LLM systems prioritize simple, composable patterns over heavy, opaque frameworks. This guide breaks down the core architectural distinction between prescriptive **workflows** and autonomous **agents**, outlines 5 fundamental workflow patterns, and defines production rules for building reliable agentic systems.

---

## 📐 Architectural Foundation: Workflows vs. Agents

*   **Workflows:** Systems where LLMs and tools are orchestrated through **predefined code paths**. They offer high predictability and consistency for well-defined tasks.
*   **Agents:** Systems where LLMs **dynamically direct** their own processes, execution loops, and tool usage. They offer maximum flexibility for open-ended, non-deterministic tasks.
*   **The Augmented LLM (Base Building Block):** A foundational LLM enhanced with retrieval (RAG), tools (e.g., via Model Context Protocol / MCP), and memory. Every workflow or agent is built by composing augmented LLMs.

---

## ⚙️ The 5 Fundamental Workflow Patterns

### 1. Prompt Chaining
![Anthropic graphic](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F7418719e3dab222dccb379b8879e1dc08ad34c78-2401x1000.png&w=3840&q=75)
*   **Mechanism:** Decomposes a task into a linear sequence of LLM calls, where each step processes the output of the previous one. Can include programmatic "gates" (validation checks) between steps.
*   **When to Use:** Tasks that can be cleanly broken into fixed subtasks.
*   **Trade-off:** Sacrifices latency to achieve significantly higher accuracy per subtask.
*   **Examples:** Generating a document outline -> validating the outline -> writing the draft; or generating copy -> translating copy.

### 2. Routing
![Anthropic graphic](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F5c0c0e9fe4def0b584c04d37849941da55e5e71c-2401x1000.png&w=3840&q=75)
*   **Mechanism:** Classifies an input and directs it to a specialized downstream prompt, tool, or specialized model.
*   **When to Use:** Complex tasks with distinct input categories that require specialized handling.
*   **Examples:** 
    *   Routing customer support queries (refunds vs. technical support vs. general inquiries).
    *   Cost/Performance optimization: Routing easy queries to smaller models (e.g., Claude Haiku) and complex queries to frontier models (e.g., Claude Sonnet).

### 3. Parallelization
![Anthropic graphic](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F406bb032ca007fd1624f261af717d70e6ca86286-2401x1000.png&w=3840&q=75)
LLMs work simultaneously on a task and their outputs are aggregated programmatically.
*   **Sectioning (Subtasks):** Breaking a task into independent parallel subtasks (e.g., running a primary LLM task while simultaneously running a separate guardrail instance to screen for policy violations).
*   **Voting (Consensus):** Running the same prompt multiple times (or across different prompts) to gather diverse outputs or reach a consensus threshold (e.g., scanning code for security vulnerabilities).

### 4. Orchestrator-Workers
![Anthropic graphic](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F8985fc683fae4780fb34eab1365ab78c7e51bc8e-2401x1000.png&w=3840&q=75)
*   **Mechanism:** A central "Orchestrator" LLM dynamically analyzes an input, breaks it down into unpredictable subtasks, delegates them to "Worker" LLMs, and synthesizes the results.
*   **Key Distinction from Parallelization:** Subtasks are **not predefined**; they are dynamically generated based on the specific input.
*   **Examples:** Multi-file code refactoring (where the number of files changed depends on the bug), complex multi-source research tasks.

### 5. Evaluator-Optimizer
![Anthropic graphic](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F14f51e6406ccb29e695da48b17017e899a6119c7-2401x1000.png&w=3840&q=75)
*   **Mechanism:** A feedback loop where a "Generator" LLM produces a response and an "Evaluator" LLM provides critiques and refinement instructions until a threshold is met.
*   **When to Use:** Tasks with clear evaluation criteria where iterative refinement yields measurable quality gains.
*   **Examples:** Nuanced literary translation, deep iterative web search/synthesis.

---

## 🤖 Autonomous Agents

Agents operate in open-ended loops for non-deterministic problems where steps cannot be hardcoded.

*   **Operating Cycle:** Command / Discussion -> Dynamic Planning -> Execution via Tools -> Environment Feedback ("Ground Truth") -> Re-evaluation / Iteration.
*   **Key Design Requirements:**
    *   **Environmental Ground Truth:** Agents must receive explicit execution results (tool outputs, code execution errors) at each step to calibrate progress.
    *   **Stopping Conditions:** Must implement strict iteration limits or safety checkpoints to prevent runaway loops and uncontrolled cost.
    *   **Human-in-the-Loop:** Pause execution at critical decision points for user verification.
*   **Trade-off:** Higher autonomy increases task capacity but introduces higher latency, higher costs, and compounding error risk.

---

## ⚠️ Production Engineering Principles

1.  **Start Simple (KISS Principle):** Always begin with the simplest possible approach (single LLM calls with optimized prompts or RAG). Only add multi-step complexity when it demonstrably improves task performance.
2.  **Beware the "Framework Trap":** Thick SDK abstractions and GUI builders hide underlying prompts, tool calls, and raw responses, making system debugging exceptionally difficult.
3.  **Build Direct First:** Start by using direct LLM APIs to maintain visibility and control. If using a framework, ensure you thoroughly understand the underlying code paths.
4.  **Prioritize Transparency:** Explicitly log and display the agent's planning and reasoning steps.
5.  **Invest in Agent-Computer Interfaces (ACI):** The reliability of an agent heavily depends on how well its tools and schemas are documented and tested. Treat tool prompt engineering as a core priority.
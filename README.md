# Autonomous Diagnostic and Agentic Management System (ADAMS)

## 1. Overview
ADAMS is a state-driven, multi-agent orchestration framework designed for autonomous software engineering. The system operates on a closed-loop, file-based design where a central Manager Agent analyzes the project state and dynamically delegates tasks to specialized sub-agents until all issues are resolved. 

Instead of relying on complex, heavy AI reasoning at every step, ADAMS utilizes structured Markdown files (`prompt.md` and `prompt_done.md`) to maintain state, pass context, and ensure a strict, self-correcting execution loop. The workflow is entirely reactive to concrete file states, reducing token overhead and maximizing practical reliability for real-world developer workflows.

## 2. The Core Execution Loop
The system runs in a continuous loop that only breaks when the project reaches a zero-error state. 

### Phase 1: Diagnostics and Task Generation (The Manager)
* **Initialization:** The Manager Agent wakes up and reads the existing project documentation (MD files, readmes, etc.) to understand the context.
* **Prompt Generation:** The Manager acts as the diagnostic brain. It generates a master execution plan saved strictly as `prompt.md`.
* **Strict Architecture:** The `prompt.md` file acts as the single source of truth for the cycle. It is structured with system parameters and AI directions clearly defined at the top, followed by a definitive task allocation table at the bottom. This table isolates and assigns specific tasks to concurrent models (e.g., assigning API endpoint logic to a backend agent and UI fixes to a UX agent).

### Phase 2: Parallel Execution (Specialized Agents)
* The system reads the bottom table of `prompt.md` and spins up only the specific agents required for this cycle.
* These specialized models run concurrently, focusing solely on their isolated sections of the codebase (e.g., two frontend pages, one database schema).
* They operate strictly within the boundaries of the `prompt.md` directives, keeping their cognitive load light and focused on implementation.

### Phase 3: Verification and Handoff (The Validator)
* Once the specialized agents complete their tasks, a final, dedicated Validator/Tester Agent is triggered.
* **Validation:** This agent tests all recent changes, checks for regressions, and compiles the results.
* **State Reporting:** It generates a new file named `prompt_done.md`. The instructions for this agent are highly specific: it must aggressively extract any remaining errors, skipped tasks, or unfixed issues, isolating them in a dedicated `Problems Not Solved` section within the file.
* **Active Triggering (No Polling):** Instead of the Manager wasting resources polling the system to see if the work is done, the Validator Agent executes a direct terminal command/prompt. This command actively wakes up the Manager, pointing it directly to the newly generated `prompt_done.md` file with the context needed to start the next evaluation.

### Phase 4: Evaluation and Iteration
* The Manager reads `prompt_done.md`.
* **If errors exist:** The Manager digests the `Problems Not Solved` section and instantly generates a *new* `prompt.md` file, launching a fresh cycle dedicated solely to resolving the remaining technical debt.
* **If zero errors exist:** The loop breaks. 

### Phase 5: Developer Handoff
* When no problems remain, the Manager Agent performs its final task: generating a developer-friendly summary file.
* This file outlines exactly what was changed in the project, the specific problems that were solved during the loops, and strategic suggestions for the next features or architectural improvements the developer should focus on.
* The Manager then safely halts execution.

## 3. Core System Components

### A. The Agents
1. **The Manager Agent:** The central orchestrator. Requires deep context but does no coding. Its sole job is diagnosing state, writing `prompt.md`, and writing the final summary.
2. **Specialized Workers:** Lightweight, highly focused implementation models (Backend, Frontend, UX, Database). They require minimal context beyond their specific assignment in the `prompt.md` table.
3. **The Validator Agent:** The final checkpoint. A testing-focused model that runs the code, validates logic, and strictly documents failures.

### B. The State Files
1. **`prompt.md` (The Blueprint):** Contains the top-level AI directions and the bottom table assigning specific tasks to specific agents.
2. **`prompt_done.md` (The Reality Check):** Contains the raw results of the execution phase, dominated by the critical `Problems Not Solved` section.
3. **`final_summary.md` (The Handoff):** The human-readable conclusion of the successful loop, detailing changes, resolutions, and future suggestions.

## 4. Advantages of this Architecture
* **High Efficiency & Low Cost:** By separating the "thinking" (Manager) from the "doing" (Workers), you avoid running heavy, expensive models for simple codebase changes.
* **Self-Correcting:** The strict requirement for the Validator to extract "unsolved problems" guarantees that edge cases and failed implementations are automatically caught and fed back into the next loop.
* **Event-Driven Handoff:** Having the Validator actively trigger the Manager via a direct command eliminates idle polling, making the system highly responsive.
* **Developer-Centric:** The system operates exactly how a human engineering team does—plan, build, QA, revise, and report—leaving the developer with a clean, summarized output rather than a chaotic log history.

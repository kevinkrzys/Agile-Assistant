# Agile Assistant — Agent Orchestration

This repository contains a small orchestration of specialized agents defined in
[agile_assistant/agent.py](agile_assistant/agent.py#L1-L400). The file declares
four main agents and an orchestrator that enforces a strict, approval-gated
workflow for converting business requirements into user stories and test
cases.

## Overview

`agile_assistant/agent.py` implements:

- `requirements_agent` — analyzes and clarifies business requirements.
- `user_story_agent` — converts approved requirements into persona-specific
  user stories with functional acceptance criteria.
- `test_case_agent` — generates human-readable functional test cases mapped to
  user stories.
- `root_agent` — the orchestration agent that coordinates the three
  specialized agents and enforces Product Manager (PM) approval gates.

Read the implementation and detailed instructions in
[agile_assistant/agent.py](agile_assistant/agent.py#L1-L400).

## Agent Roles and Responsibilities

- **Requirements Agent (`requirements_agent`)**
  - Purpose: Extracts, normalizes, and validates business requirements from
    PRDs/BRDs/project briefs.
  - Tasks: List requirements, classify issues (missing, ambiguous,
    conflicting, out-of-scope, assumptions), and generate clarifying
    questions for the Product Manager.
  - Rules: Must not assume or invent missing details, propose solutions, or
    move forward without PM sign-off.

- **User Story Agent (`user_story_agent`)**
  - Purpose: Translate approved requirements into implementation-ready user
    stories for engineers.
  - Tasks: Produce one or more persona-focused stories per requirement, each
    with a clear title and Given/When/Then acceptance criteria.
  - Rules: Use only approved requirements; do not add technical or
    non-functional details; stop and request PM clarification if unclear.

- **Test Case Agent (`test_case_agent`)**
  - Purpose: Generate functional test cases (happy and negative paths) that
    map back to user stories.
  - Tasks: Create traceable, labeled test cases (`Must-have` vs
    `Nice-to-have`), and highlight coverage gaps where acceptance criteria are
    incomplete.
  - Rules: Avoid non-functional testing and technical automation specifics.

- **Root Orchestration Agent (`root_agent`)**
  - Purpose: Single entry point that routes documents through the pipeline
    (requirements → PM approval → stories → PM approval → tests).
  - Tasks: Invoke specialized agents in the correct sequence, enforce
    explicit PM approval before each transition, and preserve traceability.
  - Rules: Do not perform specialized work itself or bypass approval gates.

## Workflow / Flow Between Agents

1. Input (PRD/BRD/project brief) is submitted to the `root_agent`.
2. `root_agent` invokes `requirements_agent` to analyze and produce:
   - Extracted requirements
   - Identified issues and clarifying questions
3. The `root_agent` pauses and presents the requirements output to the PM.
   - The PM must answer clarifying questions and explicitly approve the
     corrected/confirmed requirements.
4. Once approved, `root_agent` invokes `user_story_agent` with only approved
   requirements to produce persona-based user stories and Given/When/Then
   acceptance criteria.
5. The `root_agent` again requires PM approval for proceeding to test
   generation. If `user_story_agent` flags unclear requirements, workflow
   returns to the PM for clarification.
6. After approval, `root_agent` invokes `test_case_agent` to create
   traceable functional test cases for each user story.
7. The final output is a labeled set of artifacts: Requirements Analysis,
   User Stories, and Test Cases, with explicit workflow state markers such as
   "Awaiting PM Approval".

## Design Principles & Strict Rules

- Separation of concerns: each agent has a single responsibility (analyze,
  write stories, or generate tests).
- Approval gates: the pipeline requires explicit PM sign-off before advancing
  between stages to avoid propagating incorrect assumptions.
- No invention or inference: agents must not invent missing requirements or
  propose implementation details.
- Traceability: outputs include links/labels to connect requirements → stories
  → tests for auditing and review.

## Value Delivered by This Implementation

- Improves clarity and reduces rework by forcing requirement validation early.
- Enforces a quality gate (PM approval) so downstream artifacts align with
  business intent.
- Creates consistent, repeatable outputs (standardized user story and test
  formats) that are easy to review and import into tooling like Jira.
- Enables specialization: each agent can be tuned independently (prompt
  rules, model variant) to improve accuracy for its task.
- Enhances auditability and traceability: you can trace each test case back
  to the original requirement and PM decisions.

## Notes

- Models used in the current definitions are Gemini-family variants. Adjust
  model parameters in `agile_assistant/agent.py` as needed for cost/latency/QA
  trade-offs.
- The `root_agent` intentionally does not implement analysis, story writing,
  or test creation itself — it orchestrates and enforces workflow discipline.
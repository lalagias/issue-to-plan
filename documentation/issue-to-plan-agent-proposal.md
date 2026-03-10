# Issue-to-Plan Agent (MVP Proposal)

## One-line pitch

Turn a product request or bug report into a repository-grounded implementation plan that an engineer (or a second agent) can execute.

## Why this should exist

Teams often know *what* they want built but lose time figuring out *where* to change code safely.  
This agent fills that gap by converting raw issue text into an implementation-ready engineering brief.

## MVP scope (simple and realistic)

### Inputs

- A chat prompt describing a feature, bug, refactor, or enhancement
- Optional pasted issue text (for example from Linear or GitHub)

### What the agent does

1. Normalize the request into a structured problem statement
2. Inspect repository structure and relevant files
3. Produce a concrete implementation plan

### Output

A structured planning brief (Markdown) designed for:

- human engineers to implement directly, or
- a follow-up implementation agent to consume.

## Explicit non-goals for MVP

To keep version 1 reliable, this agent does **not**:

- write production code
- create branches or pull requests
- perform automatic ticket updates
- require Linear integration

## Core dependencies

- GitHub repository access (read-only is enough for MVP)
- Guild repository-aware tooling (file discovery + code inspection)
- Guild workspace/session context to persist planning output

## Future dependencies (later phases)

- Linear ingestion (native issue fetch)
- Agent-to-agent handoff protocol (formal plan payload)
- PR/branch automation (for downstream implementation agent)

## Suggested v1 interaction flow

1. **Intake**
   - Classify request type: `feature | bug | enhancement | refactor`
   - Extract goals, constraints, and missing info
2. **Repository grounding**
   - Identify likely files/modules
   - Find related patterns and boundaries
3. **Plan generation**
   - Ordered implementation steps
   - Files likely touched
   - Data/API/UI considerations

## Recommended output format

Use a stable structure so humans and future agents can parse it consistently:

1. **Problem Summary**
2. **Repository Findings**
3. **Implementation Plan (ordered)**
4. **Open Questions**

## Quality bar (how to evaluate)

1. **Repo relevance**  
   Are the suggested files/modules actually the right ones?
2. **Actionability**  
   Can an engineer execute this without re-planning from scratch?
3. **Handoff quality**  
   Can another agent consume it with minimal interpretation?
4. **Real-world usefulness**  
   On real issues, does it save engineering time and reduce ambiguity?

## Minimal success criteria for launch

- Produces consistent, structured plans across multiple issue types
- Correctly identifies likely implementation areas in the repository
- Produces a concrete ordered plan that an engineer can execute directly

## Nice-to-have next step after MVP

Add a second "Plan-to-Implementation" agent that consumes this brief and proposes or writes code in a controlled workflow.


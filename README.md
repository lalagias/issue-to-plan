# Issue-to-Plan Agent

Turn any product request, bug report, or engineering issue into a **repository-grounded implementation plan** — ready for an engineer or a follow-up agent to execute.

## What it does

Most teams know *what* they want built but lose time figuring out *where* to change code safely. This agent bridges that gap.

Give it an issue description and a GitHub repository, and it will:

1. **Understand the request** — classifies the issue (feature, bug, enhancement, refactor), restates the goal, and asks clarifying questions if anything is ambiguous.
2. **Inspect the repository** — browses the file tree, reads relevant source files, and identifies architectural patterns and boundaries.
3. **Produce a structured plan** — outputs an actionable implementation brief with specific files, ordered steps, and open questions.

## Output format

Every plan follows a consistent structure:

- **Problem Summary** — what needs to be built or fixed, and why.
- **Repository Findings** — relevant files, modules, patterns, and dependencies discovered in the codebase.
- **Implementation Plan** — ordered, concrete steps naming specific files and describing what to change.
- **Open Questions** — assumptions made, decisions needing human input, or gaps in the codebase.

## Example usage

> *"Add a dark mode toggle to the settings page"* + `owner/repo`

The agent will inspect the repo, find the settings page component, identify the existing theming approach, and produce a step-by-step plan for adding the toggle — grounded in the actual code.

## Built with

- [Guild AI](https://guild.ai) agent SDK
- GitHub integration for repository inspection (read-only)

## Non-goals (v1)

This agent deliberately does **not**:

- Write or commit code
- Create branches or pull requests
- Update tickets or project management tools

It focuses on one thing — producing high-quality, actionable plans.

## License

MIT

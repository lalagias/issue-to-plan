---
name: guild-agent-dev
description: Guild Agent SDK development patterns and best practices. Use when writing agent code, importing SDK modules, using task.tools, creating LLM agents, or troubleshooting agent TypeScript errors.
---

# Guild Agent Development

Build agents for Guild using the CLI. **Always use the Guild CLI for agent operations — never use raw git commands.**

## When to Use This

Activate when user:

- Mentions "guild agent" commands
- Wants to create, save, or publish an agent
- Is working in an agent directory
- Mentions agent development workflow
- Asks about agent versioning or publishing
- Wants to build a new agent

## SDK Reference

### Imports

The SDK core comes from `@guildai/agents-sdk`. Service tools are in separate `@guildai-services/*` packages.

```typescript
// Agent factories
import { agent, llmAgent } from '@guildai/agents-sdk';

// Types
import type {
  Task,
  AgentResult,
  TypedToolResult,
  TypedToolError,
} from '@guildai/agents-sdk';

// Result helpers (for self-managed state agents)
import { ask, output, callTools } from '@guildai/agents-sdk';

// Platform tools (from SDK)
import { guildTools, userInterfaceTools } from '@guildai/agents-sdk';

// Service tools (from separate packages — NOT from SDK)
import { gitHubTools } from '@guildai-services/guildai~github';
import { slackTools } from '@guildai-services/guildai~slack';
import { jiraTools } from '@guildai-services/guildai~jira';
import { bitbucketTools } from '@guildai-services/guildai~bitbucket';
import { azureDevOpsTools } from '@guildai-services/guildai~azure-devops';

// Utilities
import { pick, progressLogNotifyEvent } from '@guildai/agents-sdk';

// Advanced (for compiled agents with LLM tool loops)
import { delegatedCallsOf, asToolResultContent } from '@guildai/agents-sdk';

// Zod (provided by runtime, do NOT add to dependencies)
import { z } from 'zod';
```

### Service Packages Table

| Service        | Package                                  | Export               | Tool Name Prefix |
| -------------- | ---------------------------------------- | -------------------- | ---------------- |
| GitHub         | `@guildai-services/guildai~github`       | `gitHubTools`        | `github_`        |
| Slack          | `@guildai-services/guildai~slack`        | `slackTools`         | `slack_`         |
| Jira           | `@guildai-services/guildai~jira`         | `jiraTools`          | `jira_`          |
| Bitbucket      | `@guildai-services/guildai~bitbucket`    | `bitbucketTools`     | `bitbucket_`     |
| Azure DevOps   | `@guildai-services/guildai~azure-devops` | `azureDevOpsTools`   | `azure_devops_`  |
| Guild          | `@guildai/agents-sdk`                    | `guildTools`         | `guild_`         |
| User Interface | `@guildai/agents-sdk`                    | `userInterfaceTools` | `ui_`            |

### Tool Access via `task.tools.*`

All tool calls go through `task.tools.<toolName>(args)`.

```typescript
// GitHub
const pr = await task.tools.github_pulls_get({ owner, repo, pull_number: 123 });
const results = await task.tools.github_search_issues_and_pull_requests({
  q: 'is:pr is:open repo:owner/name',
});

// Slack
await task.tools.slack_chat_post_message({ channel: 'C1234567890', text: 'Hello!' });

// Jira
const issues = await task.tools.jira_search_and_reconsile_issues_using_jql({
  jql: 'project = MYPROJ AND status = Open',
});

// User interface
const response = await task.tools.ui_prompt({ type: 'text', text: 'What repo?' });
await task.tools.ui_notify(progressLogNotifyEvent('Processing...'));

// Guild
const me = await task.tools.guild_get_me({});
await task.tools.guild_credentials_request({ service: 'GITHUB' });
```

### Task Properties

| Property         | Description                                                                          |
| ---------------- | ------------------------------------------------------------------------------------ |
| `task.sessionId` | Session ID for correlating operations                                                |
| `task.console`   | Debug logging (`task.console.debug(...)`, `.info(...)`, `.warn(...)`, `.error(...)`) |
| `task.tools`     | Primary API for calling all tools                                                    |
| `task.guild`     | Guild service (available when **all** `guildTools` included — see warning below)     |

## Agent Patterns

Three patterns, ordered by simplicity:

### 1. LLM Agent (`llmAgent()`) — Simplest

For conversational/prompt-driven agents where the LLM IS the logic.

```typescript
import { guildTools, llmAgent, pick } from '@guildai/agents-sdk';
import { gitHubTools } from '@guildai-services/guildai~github';

export default llmAgent({
  description: 'Helps users with GitHub questions',
  tools: {
    ...pick(gitHubTools, ['github_issues_list_for_repo', 'github_issues_get']),
    ...guildTools, // ALWAYS spread fully — never pick() from guildTools
  },
  systemPrompt: `
    You are a helpful assistant that answers questions about GitHub repositories.
    Use the GitHub tools to look up information when asked.
  `,
  mode: 'multi-turn', // "one-shot" (default) or "multi-turn"
});
```

### 2. Automatic State Agent (`run()`) — Recommended for Code-First

Runtime manages state via continuations. Requires `"use agent"` directive.

```typescript
'use agent';

import {
  type Task,
  agent,
  guildTools,
  pick,
  userInterfaceTools,
} from '@guildai/agents-sdk';
import { gitHubTools } from '@guildai-services/guildai~github';
import { z } from 'zod';

const inputSchema = z.object({
  type: z.literal('text'),
  text: z.string().describe('Repository in owner/repo format'),
});

const outputSchema = z.object({
  type: z.literal('text'),
  text: z.string().describe('Summary of open PRs'),
});

const tools = {
  ...pick(gitHubTools, ['github_search_issues_and_pull_requests']),
  ...guildTools,
  ...userInterfaceTools,
};

type Tools = typeof tools;

async function run(input: z.infer<typeof inputSchema>, task: Task<Tools>) {
  const repo = input.text.trim();

  const results = await task.tools.github_search_issues_and_pull_requests({
    q: `is:pr is:open repo:${repo}`,
    per_page: 20,
  });

  if (!results.items?.length) {
    return { type: 'text', text: `No open PRs found in ${repo}` };
  }

  const summary = results.items
    .map((pr) => `- #${pr.number}: ${pr.title} (by ${pr.user?.login})`)
    .join('\n');

  return { type: 'text', text: `## Open PRs in ${repo}\n\n${summary}` };
}

export default agent({
  description: 'Lists open PRs in a GitHub repository',
  inputSchema,
  outputSchema,
  tools,
  run,
});
```

### 3. Self-Managed State Agent (`start()`/`onToolResults()`)

For explicit state control. Uses `ask()`, `output()`, `callTools()` helpers.

```typescript
import {
  agent,
  ask,
  output,
  callTools,
  userInterfaceTools,
  type Task,
  type AgentResult,
  type TypedToolResult,
  type TypedToolError,
} from '@guildai/agents-sdk';
import { z } from 'zod';

const stateSchema = z.object({ count: z.number() });
const tools = { ...userInterfaceTools };
type Tools = typeof tools;
type State = z.infer<typeof stateSchema>;

async function start(input, task: Task<Tools, State>): Promise<AgentResult<Output, Tools>> {
  await task.save({ count: 1 });
  return ask(`Got: ${input.text}`);
}

async function onToolResults(
  results: Array<TypedToolResult<Tools> | TypedToolError<Tools>>,
  task: Task<Tools, State>
): Promise<AgentResult<Output, Tools>> {
  const state = await task.restore();
  const result = results[0];
  if (result.type === 'tool-result' && result.output.text === 'done') {
    return output({ type: 'text', text: `Final count: ${state!.count}` });
  }
  await task.save({ count: state!.count + 1 });
  return ask(`Count: ${state!.count + 1}`);
}

export default agent({
  description: 'Tracks conversation state explicitly',
  stateSchema,
  tools,
  start,
  onToolResults,
});
```

## Anti-Hallucination Guide

### NEVER `pick()` from `guildTools`

**Always spread `guildTools` fully. NEVER use `pick(guildTools, [...])`.**

```typescript
// ❌ WRONG — task.guild will be undefined
const tools = {
  ...pick(guildTools, ['guild_get_me', 'guild_create_agent']),
};

// ✅ CORRECT — always spread fully
const tools = {
  ...guildTools,
  ...userInterfaceTools,
};
```

### DO NOT USE (Common Mistakes)

```typescript
// ❌ WRONG: identifier is deprecated
export default agent({ identifier: "my-agent", ... })

// ❌ WRONG: service tools are NOT in @guildai/agents-sdk
import { gitHubTools } from "@guildai/agents-sdk"

// ❌ WRONG: these direct service accessors don't exist
const pr = await task.github.search_issues(...)
await task.slack.post_message(...)

// ❌ WRONG: parameter name
github_search_issues_and_pull_requests({ query: "..." })  // Use { q: "..." }

// ❌ WRONG: task.ui_prompt() is not a method on task
await task.ui_prompt("What repo?")

// ❌ WRONG: missing "use agent" directive on coded agents
import { agent } from "@guildai/agents-sdk"
export default agent({ run: async (input, task) => { ... } })
```

### CORRECT Patterns

```typescript
// ✅ No identifier needed
export default agent({ description: "My agent", ... })

// ✅ Service tools from @guildai-services/* packages
import { gitHubTools } from "@guildai-services/guildai~github"

// ✅ Platform tools from @guildai/agents-sdk
import { guildTools, userInterfaceTools } from "@guildai/agents-sdk"

// ✅ Use task.tools.* for all tool calls
const pr = await task.tools.github_pulls_get({ owner, repo, pull_number })

// ✅ "use agent" directive for coded agents
"use agent"
import { agent } from "@guildai/agents-sdk"
export default agent({ run: async (input, task) => { ... } })
```

## package.json

```json
{
  "name": "guild-agent-{name}",
  "version": "1.0.0",
  "type": "module",
  "dependencies": {
    "slackify-markdown": "^4.5.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

**Important:**

- Do NOT add `@guildai/agents-sdk`, `@guildai-services/*`, or `zod` to dependencies
- DO add third-party packages your agent uses to `dependencies`

## File Structure

```
my-agent/
├── .git/              # Git repo (remote is Guild server)
├── .gitignore         # Includes guild.json
├── agent.ts           # Your agent code (at project root, NOT in src/)
├── package.json       # Dependencies
├── tsconfig.json      # TypeScript config
└── guild.json         # Agent ID (gitignored, local only)
```

## CLI Commands

```bash
guild agent create <name>                          # Create new agent
guild agent create <name> --template LLM           # Create with template
guild agent init                                   # Initialize local workspace
guild agent save --message "description"           # Save changes
guild agent save --message "v1.0" --wait --publish # Save + validate + publish
guild agent pull                                   # Pull remote changes
guild agent test                                   # Interactive test
guild agent chat "Hello"                           # Test with input
guild agent versions [agent-id]                    # Version history
guild agent clone <agent-id>                       # Clone existing agent
guild agent code [agent-id]                        # View agent source
guild agent grep <pattern>                         # Search agent code
```

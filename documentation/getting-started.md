# Building Agents with the [Guild CLI](https://www.npmjs.com/package/@guildai/cli)

## Prerequisites

Before you begin, make sure you have:

- **Node.js 25+** and **npm** — [nodejs.org](https://nodejs.org)  
- **A Guild account** — Guild.ai is in closed beta. Request an invitation at [hello@guild.ai](mailto:hello@guild.ai)

## Install the CLI

```shell
npm install -g @guildai/cli
```

Then authenticate with Guild:

```shell
guild auth login
```

This opens your browser to complete sign-in at [app.guild.ai](https://app.guild.ai). It also configures your local npm registry so the CLI can install Guild packages.

Verify everything is working:

```shell
guild auth status
# ✓ Authenticated
```

If you run into issues, see [Troubleshooting](#troubleshooting) below.

### Select your default workspace:

### 

```shell
guild workspace select
```

### 

If you have one workspace, it's selected automatically. If you have several, pick one from the list. This tells the CLI where to run agent tests and chat sessions.

### Set up coding assistant skills (optional)

If you use Claude Code or another coding assistant, install Guild CLI skills into your project:

```shell
guild setup
```

This deposits SDK reference and CLI workflow docs into `.claude/skills/` so your coding assistant understands Guild agent development patterns.

## Hello World

Let's create a simple agent that greets users.

### 1\. Create the agent

```shell
mkdir hello-agent && cd hello-agent
guild agent init --name hello-agent --template LLM
```

This creates the agent in the Guild backend, initializes a local git repo, and pulls starter files:

```
hello-agent/
├── agent.ts          # Your agent code (edit this)
├── package.json      # Dependencies (runtime packages are pre-configured)
├── tsconfig.json     # TypeScript config
├── guild.json        # Local config (managed by the CLI, don't edit)
└── .gitignore
```

### 2\. Edit the agent

Open `agent.ts` and replace the contents:

```ts
import { llmAgent, guildTools, userInterfaceTools } from '@guildai/agents-sdk';

export default llmAgent({
  description: 'A friendly greeting agent',
  tools: { ...guildTools, ...userInterfaceTools },
  systemPrompt: `You are a friendly assistant. Greet users warmly and answer their questions.`,
  mode: 'multi-turn',
});
```

### 3\. Test it

```shell
guild agent test --ephemeral
```

This opens an interactive chat session. Type a message and press Enter:

```
You: Hello!
Agent: Hello! Welcome — I'm happy to help. What can I do for you today?
```

Press `Ctrl+C` to exit.

### 4\. Save and publish

```shell
guild agent save --message "First version" --wait --publish
```

- `--wait` blocks until validation passes  
- `--publish` makes the agent available in the Guild catalog

Check status:

```shell
guild agent get
guild agent versions
```

That's it — your first agent is live.

## Installing Skills

If you use [Claude Code](https://docs.anthropic.com/en/docs/claude-code), install Guild CLI skills into your project:

```shell
guild setup
```

This deposits SDK reference and CLI workflow docs into `.claude/skills/` so Claude Code understands Guild agent development patterns. Support for other coding assistants is coming soon.

## Create an Agent

If you skipped the Hello World, here's the quick version:

```shell
mkdir my-agent && cd my-agent
guild agent init
```

The CLI will prompt for a name and template. Or skip the prompts:

```shell
guild agent init --name my-agent --template LLM
```

### Templates

| Template | Use when |
| :---- | :---- |
| `LLM` | The LLM is the logic. You write a prompt and pick tools. Start here. |
| `AUTO_MANAGED_STATE` | You write procedural TypeScript that calls tools inline. |
| `BLANK` | You want full control over the agent lifecycle. |

## Write Your Agent

Edit `agent.ts`. The template gives you a working starting point — modify it to fit your use case.

### LLM Agent

Define a system prompt and the tools the LLM can use. No procedural code needed.

```ts
import { llmAgent, guildTools, userInterfaceTools } from '@guildai/agents-sdk';
import { gitHubTools } from '@guildai-services/guildai~github';

export default llmAgent({
  description: 'Answers questions about GitHub repositories',
  tools: { ...gitHubTools, ...guildTools, ...userInterfaceTools },
  systemPrompt: `You help users understand their GitHub repositories.
Use the GitHub tools to look up real data when asked.`,
  mode: 'multi-turn',
});
```

`mode: "multi-turn"` keeps the conversation going after each response. Use `"one-shot"` (default) when the agent should respond once and finish.

### Code-First Agent

Write TypeScript that calls tools directly. The runtime manages state between tool calls via continuations. Requires the `"use agent"` directive at the top of the file.

```ts
'use agent';

import { agent, guildTools, userInterfaceTools, type Task } from '@guildai/agents-sdk';
import { gitHubTools } from '@guildai-services/guildai~github';
import { z } from 'zod';

const tools = { ...gitHubTools, ...guildTools, ...userInterfaceTools };
type Tools = typeof tools;

const inputSchema = z.object({
  type: z.literal('text'),
  text: z.string().describe('Repository in owner/repo format'),
});

const outputSchema = z.object({
  type: z.literal('text'),
  text: z.string(),
});

async function run(input: z.infer<typeof inputSchema>, task: Task<Tools>) {
  const prs = await task.tools.github_search_issues_and_pull_requests({
    q: `is:pr is:open repo:${input.text}`,
    per_page: 10,
  });

  if (!prs.items?.length) {
    return { type: 'text' as const, text: 'No open PRs.' };
  }

  const summary = prs.items.map((pr) => `- #${pr.number}: ${pr.title}`).join('\n');

  return { type: 'text' as const, text: summary };
}

export default agent({
  description: 'Lists open PRs in a GitHub repo',
  inputSchema,
  outputSchema,
  tools,
  run,
});
```

### Tools Overview

Every agent has access to three categories of tools:

- **Service tools** (e.g., `gitHubTools`, `slackTools`) — call third-party APIs. When a user first triggers a service tool, Guild prompts them to connect their account via OAuth.  
- **`guildTools`** — query the Guild platform (look up agents, workspaces, sessions, triggers).  
- **`userInterfaceTools`** — interact with the user during a session (ask questions, send progress updates).

All tool calls go through `task.tools`:

```ts
// Service tools
const pr = await task.tools.github_pulls_get({ owner, repo, pull_number: 42 });
await task.tools.slack_chat_post_message({ channel: 'C123', text: 'PR merged' });

// User interface tools
const answer = await task.tools.ui_prompt({ type: 'text', text: 'Which repo?' });
```

## Available Services

Import service tools from their packages. These are provided by the runtime — don't add them to `package.json`.

| Service | Import |
| :---- | :---- |
| GitHub | `import { gitHubTools } from "@guildai-services/guildai~github"` |
| Slack | `import { slackTools } from "@guildai-services/guildai~slack"` |
| Jira | `import { jiraTools } from "@guildai-services/guildai~jira"` |
| Bitbucket | `import { bitbucketTools } from "@guildai-services/guildai~bitbucket"` |
| Azure DevOps | `import { azureDevOpsTools } from "@guildai-services/guildai~azure-devops"` |

## Available Models

Guild agents can use the following LLM providers and models:

| Provider | Models |
| :---- | :---- |
| Anthropic | Claude Sonnet 4, Claude Haiku 3.5 |
| OpenAI | GPT-4o, GPT-4o mini |
| Google | Gemini 2.0 Flash |

Model selection is handled by the runtime. LLM agents use the platform's default model unless configured otherwise.

## Development Loop

The typical workflow is: pull → edit → test → save.

### 1\. Pull latest changes

If others are working on the same agent, pull their changes first:

```shell
guild agent pull
```

### 2\. Test locally

```shell
guild agent test              # Interactive chat session
guild agent chat "Hello"      # Send a single message
```

`guild agent test` opens an interactive session where you can chat with your agent. Changes to `agent.ts` take effect on the next save — you don't need to restart.

### 3\. Save your work

```shell
guild agent save --message "Add Slack notifications"
```

This commits your code and creates a new version in the Guild backend. Versions start as drafts.

### 4\. Publish

```shell
guild agent save --message "Ready to ship" --wait --publish
```

`--wait` blocks until validation passes. `--publish` makes the agent available in the catalog. You can also publish separately:

```shell
guild agent publish
```

### Check status

```shell
guild agent get                # Agent info and current version
guild agent versions           # Version history
guild agent code               # View source of latest version
```

## Key Rules

- Agent code lives at `agent.ts` in the project root.  
- Don't add `@guildai/agents-sdk`, `zod`, or `@guildai-services/*` to `package.json`. The runtime provides them. Only add third-party packages you actually use.  
- Always call tools through `task.tools.<name>(args)`. Never access services directly.  
- Always use `guild agent save` to commit and `guild agent pull` to sync. Don't use raw git commands.  
- Don't edit `guild.json` — it's managed by the CLI.

## Other Commands

```shell
guild agent clone owner/agent-name    # Clone an existing agent to work on locally
guild agent init --fork owner/name    # Fork an agent as a starting point
guild agent pull                      # Pull remote changes into local directory
guild agent unpublish                 # Remove from catalog
guild agent revalidate                # Re-run validation on latest version
```

## Diagnostics

Run `guild doctor` to check your setup:

```shell
guild doctor
```

```
Checking Guild CLI setup...

  ✓ Authentication       Logged in
  ✓ Server               Connected to https://app.guild.ai/api (125ms)
  ✓ Global config        ~/.guild/config.json
  ✓ Default workspace    my-workspace
  - Local config         Not in an agent directory
  ✓ Git                  Installed

5 passed, 0 failed, 1 skipped
```

## Troubleshooting

### "Connection refused" or "Cannot connect to server"

1. Check your internet connection  
2. Run `guild doctor` to see which check is failing

### "Workspace not found" or wrong workspace

Your default workspace may not match the target server. Override with an environment variable:

```shell
GUILD_WORKSPACE_ID=<workspace-id> guild chat "hello"
```

Or set a new default:

```shell
guild workspace select
```

### "Not authenticated" from Guild CLI

Run `guild auth login` to re-authenticate:

```shell
guild auth login
guild auth status
```

### "No agent ID provided and not in an agent directory"

You need to either run the command from inside an agent directory (one with a `guild.json` file), or pass the agent ID explicitly:

```shell
# Option 1: cd into the agent directory
cd my-agent
guild agent get

# Option 2: pass the agent ID
guild agent get <agent-id>
```

### "No changes to commit" after a failed save

If a previous `guild agent save` committed locally but failed to push (e.g., network error), just run save again. It detects the unpushed commits and resumes:

```shell
guild agent save --message "Retry"
```

### Validation failures

After saving, if validation fails:

```shell
# Check the latest version for errors
guild agent versions --limit 1

# Fix the issue and save again
guild agent save --message "Fix validation error" --wait
```

### Agent test not responding

If `guild agent test` hangs or produces no output:

1. Check your agent code compiles: look for TypeScript errors in `agent.ts`  
2. Make sure you've saved at least once: `guild agent save --message "initial"`  
3. Try a single message instead: `guild agent chat "hello"`

## Next Steps

- Browse agents at [app.guild.ai/agents](https://app.guild.ai/agents)  
- Fork an existing agent: `guild agent init --fork owner/agent-name`  
- Clone one to study: `guild agent clone owner/agent-name`


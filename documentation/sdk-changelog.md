# Changelog

## 0.3.10

- Removed IAP (Identity-Aware Proxy) from production (`app.guild.ai`)  
- Published as `@guildai/cli` on public npm registry  
- Automated npm publish via GitHub Actions on version bump  
- Auth failures show beta invitation guidance  
- `ensureAuthenticated()` validates token against the server before rendering UI  
- Updated getting-started guide for public npm install

## 0.3.9

- `GUILD_WORKSPACE_ID` and `GUILD_OWNER_ID` environment variable overrides  
- `guild workspace select` writes to `guild.json` when run from an agent directory  
- Clear error message when workspace is not found during chat session creation

## 0.3.8

- `guild agent owners` command to list accounts that can own agents  
- `--owner` flag on `guild agent init` and `guild agent fork` to select owner  
- `guild config set default_owner <id>` to set a default owner for agent creation

## 0.3.7

- `guild agent save` recovers from a previous save that committed locally but failed to push

## 0.3.6

- `guild setup` command to install Claude Code skills into a project  
- Show build errors inline on `agent test`, `agent chat`, `agent save`, and `agent publish`

## 0.3.5

- `guild agent pull` command to sync remote changes into local agent directory  
- `guild agent grep` command to search agent source code  
- `guild agent search` command to find agents in the catalog  
- Session resume hints and `--resume` flag for `guild agent chat`  
- `Ctrl+D` and `exit`/`quit` to close chat sessions  
- Pagination for list commands  
- Light background terminal support  
- `agent chat` creates ephemeral version automatically  
- `agent test` now uses the same interactive chat UI as `agent chat`  
- Service tool names updated to match OpenAPI specs (e.g., `github_pulls_get`)  
- Removed built-in demo agents  
- Fixed git tracking after save push

## 0.3.1

- Clean up npm registry config on `guild auth logout`

## 0.3.0

- Default backend changed from shared.guildai.dev to app.guild.ai (production)  
- `guild config` command group (`list`, `get`, `set`, `path`) for persistent CLI configuration  
- `guild doctor` command for diagnosing setup issues (auth, server, config, workspace, git)  
- Persistent `--debug`, `--json`, `--quiet` flags via `guild config set`  
- Fixed splash scrollback and flicker on iTerm2 (two-phase clear strategy replaces Ink's clearTerminal)  
- Splash animation restored to 30fps  
- Fixed null agent crash in task panel  
- Clarified `guild chat` vs `guild agent chat` help text  
- `--public`/`--private` flags for `guild agent update`  
- `guild auth token` cleanup

## 0.2.1

- Splash screen UX and error message consistency  
- Auto-append `/api` to `GUILDCORE_URL` when missing  
- Show workspaces from all orgs with owner display  
- Session management commands (`guild session list`, `delete`)  
- Trigger management commands (`guild workspace triggers`)  
- `--ephemeral` flag for `guild agent test`  
- Reuse existing version in `agent test`  
- Handle agent object in event matching  
- CLI sends `Accept: application/json` header  
- Accurate error messages for keychain failures  
- Wait for validation then publish  
- Prevent git hangs in non-TTY environments  
- `guild auth token` command  
- Unified `guild agent chat` with shared ChatApp  
- Full-name identifier support (`owner/agent-name`)  
- Production IAP support for app.guild.ai  
- Workspace agent subcommands  
- `guild workspace current` command  
- Agent install request prompts in chat  
- Chat task panel and spinner UX improvements  
- Improved publish error handling  
- TypeScript and ESLint checks for test files  
- Better E2E test helpers with structured error messages

## 0.2.0

- Chat UI overhaul: Ink (React for CLIs) replaces blessed  
- Task panel showing hierarchical agent/tool activity  
- Animated splash screen with Lottie logo and orange particle background  
- Braille canvas spinner system  
- Event-driven messaging system (replaces polling)  
- Agent-builder meta agent for creating agents via conversation  
- Agent revalidate command  
- Fake LLM provider for fast E2E tests  
- Switched default backend to shared.guildai.dev with IAP support  
- `--json` flag implies quiet mode  
- Agent validation on version creation  
- VSCode-friendly auth UX (clickable URLs, friendly labels)  
- Agent chat commands and JSON input/output modes  
- E2E test infrastructure with fast/slow split  
- Clear invalid auth token on 401  
- ESLint rule to prevent debug console misuse

## 0.1.0

- Initial CLI release  
- OAuth device flow authentication (`guild auth login`, `status`, `logout`)  
- Interactive chat with Guild assistant (`guild chat`)  
- One-shot mode (`guild chat --once`)  
- Chat progress spinners and completion detection  
- Agent development workflow (`guild agent init`, `save`, `test`, `publish`, `fork`, `tags`)  
- Workspace commands (`guild workspace list`, `select`, `create`)  
- Context management commands (`guild context`)  
- Splash screen with ASCII braille logo  
- Events API integration (replaces messages API)  
- Centralized baseUrl configuration  
- Configurable backend via `GUILDCORE_URL` / `GUILDCORE_PORT`


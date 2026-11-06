# Network Solutions for Claude Code

A Claude Code plugin for Network Solutions. It does two things:

- **Brand building:** turns a business idea into a name, matching domain options, logo concepts, and website layouts, then lets you refine and save those choices before continuing in Network Solutions.
- **Hosting:** plans, deploys, diagnoses, and rolls back Node.js and FastAPI apps on your Network Solutions hosting, straight from your repository.

## Features

Brand building:

- Suggest business names from an idea you describe in plain language
- Search for matching domains, check a specific domain, and show registration pricing
- Generate logo options and refine a selected logo
- Generate website layouts, then adjust the color palette, font pairing, imagery, and hero copy
- Save a chosen domain, logo, or website layout to a draft, then continue in Network Solutions when you're ready

Hosting:

- Detect your app's runtime and recommend a hosting plan (read-only)
- Deploy your repository to Network Solutions hosting
- Investigate a failing deployment and recover it
- Roll back to a previous known-good release
- Answer how-to questions about Network Solutions hosting

Domain purchases, hosting purchases, and brand checkout happen on the Network Solutions website. Deploys and rollbacks run from Claude Code, and each one waits for your explicit approval. Before a deploy, Claude shows you any Dockerfile it wrote, plus the target and application name.

## Prerequisites

- [Claude Code](https://code.claude.com/docs/en/plugins) installed
- For deploys, diagnosis, and rollbacks: Node.js 22.12 or later and the Network Solutions CLI, installed with `npm i -g @network-solutions/cli`. Claude asks before installing it for you.

## MCP server

The plugin connects to the public Network Solutions MCP server, `network-solutions`, set in [`.mcp.json`](.mcp.json). It provides brand building, domain search, and hosting plans without sign-in. There is no API key or environment variable to set.

Deploys, diagnosis, and rollbacks use the Network Solutions CLI, which handles your account sign-in with `netsol login`.

## Installation

Clone this repository, then start Claude Code with the plugin directory:

```sh
git clone https://github.com/netsolaicoe/netsol-claude-brand-builder.git
claude --plugin-dir ./netsol-claude-brand-builder
```

Claude Code loads the plugin for that session. To check the plugin structure after making changes, run `claude plugin validate ./netsol-claude-brand-builder` from the directory containing the clone, or `claude plugin validate .` from the repository root.

## Usage

In the Claude Code session started above, try prompts such as:

- “I’m starting a neighborhood bakery. Help me find a name.”
- “Create a logo and website for Cedar & Crumb, a neighborhood bakery.”
- “Check whether cedarandcrumb.com is available.”
- “Deploy this repository to Network Solutions.”
- “Why is my Network Solutions deployment failing?”

Reply in plain language to choose or refine the options Claude shows. Saving a logo or website layout makes the Continue to Network Solutions action available.

## Repository contents

| Path | Purpose |
| --- | --- |
| [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json) | Plugin metadata and component paths |
| [`skills/network-solutions-business-toolkit/SKILL.md`](skills/network-solutions-business-toolkit/SKILL.md) | Conversation guidance and supported brand workflows |
| [`skills/hosting-help/SKILL.md`](skills/hosting-help/SKILL.md) | Read-only answers about Network Solutions hosting |
| [`skills/app-detect-and-plan/SKILL.md`](skills/app-detect-and-plan/SKILL.md) | Detect an app runtime and produce a hosting plan |
| [`skills/deploy-to-network-solutions/SKILL.md`](skills/deploy-to-network-solutions/SKILL.md) | Deploy a repository to Network Solutions |
| [`skills/diagnose-and-recover-deployment/SKILL.md`](skills/diagnose-and-recover-deployment/SKILL.md) | Investigate a deployment and recover it with consent |
| [`skills/rollback-deployment/SKILL.md`](skills/rollback-deployment/SKILL.md) | Restore a prior known-good release |
| [`.mcp.json`](.mcp.json) | Network Solutions MCP server connection |

## License

Apache 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE) for copyright and trademark information.

# Network Solutions Brand Builder for Claude Code

A Claude Code plugin for developing a business brand with Network Solutions. It turns a business idea into a name, matching domain options, logo concepts, and website layouts, then lets you refine and save those choices before continuing in Network Solutions.

The plugin contains a [Brand Creator skill](skills/network-solutions-business-toolkit/SKILL.md), hosting skills for planning, deploying, diagnosing, and rolling back an application, and a connection to the Network Solutions MCP gateway. Domain availability and pricing come from the gateway. Previews and saved choices stay drafts until you continue in Network Solutions.

## Features

- Suggest business names from an idea you describe in plain language
- Search for matching domains, check a specific domain, and show registration pricing
- Generate logo options and refine a selected logo
- Generate website layouts, then adjust the color palette, font pairing, imagery, and hero copy
- Save a chosen domain, logo, or website layout to a draft
- Offer Continue to Network Solutions after a logo or website layout is saved

Purchasing a domain or publishing a site happens in Network Solutions, after you leave this chat.

## Prerequisites

- [Claude Code](https://code.claude.com/docs/en/plugins) installed
- Access to the Network Solutions MCP gateway

The connection is already set in [`.mcp.json`](.mcp.json). It uses the public gateway URL and a static client header, so there is no API key or other environment variable to set. If the gateway asks you to sign in, follow the prompt from Claude Code or your Network Solutions administrator.

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
| [`.mcp.json`](.mcp.json) | Network Solutions MCP gateway connection |

## License

Apache 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE) for copyright and trademark information.

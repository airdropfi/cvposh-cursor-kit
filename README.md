# CVPosh for Cursor and Grok Build

Work on the resumes you built at [CVPosh](https://cvposh.com) without copy and paste, from Cursor or Grok Build. Everything is read-only: your agent can read your CVs, score them for applicant tracking systems (ATS) and compare them with a job posting, but it cannot change, delete or share them.

## At a glance

- **What it is:** CVPosh is a resume and CV builder with an MCP server built for AI agents: your agent reads your saved CVs, scores them for ATS compatibility and compares them with a job posting, and you review and edit the result.
- **Who uses it:** an AI assistant or agent (ChatGPT, Claude, Claude Code, Cursor, Gemini CLI, other MCP clients) working for a job seeker or career coach. The human tags the work, approves the connection and reviews the output.
- **What it does not do:** every tool is read-only, so an agent cannot create, edit, delete, export or share a CV. The ATS score is a fixed checklist, not a simulation of any employer's system, and it does not predict interviews or hiring.
- **Price:** the connector needs a free CVPosh account and is not tied to a paid plan. CVPosh Pro is $5 per month or $9 once and covers DOCX export, more PDF exports, all 25 templates and AI job tailoring in the editor.
- **Data and control:** the ATS and keyword checks run on CVPosh servers with fixed rules, call no AI model and store nothing. You approve the connection on a consent page, see it under Settings > API keys, and can delete it at any time; connections expire after 90 days. Profile photos are never sent.
- **MCP:** `https://cvposh.com/api/mcp` (Streamable HTTP). Official MCP Registry name: `com.cvposh/cvposh`. Server card: https://cvposh.com/.well-known/mcp/server-card.json
- **Limits per connection:** 120 requests a minute, 100 tool calls a day.

## Install in 60 seconds

1. In CVPosh, open Settings > API keys and create a key (Cursor connects with the key, not OAuth).
2. Put it in the `CVPOSH_API_KEY` environment variable and add the server to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "cvposh": {
      "url": "https://cvposh.com/api/mcp",
      "headers": { "Authorization": "Bearer ${env:CVPOSH_API_KEY}" }
    }
  }
}
```

Open Customize, then MCPs: `cvposh` should list four tools.

## Prompts to copy

You assign the work and review the result:

- "Assign to CVPosh: list my saved CVs, run the ATS check on the newest one and show me the issues in priority order. Change nothing; I will make the edits and review each one."
- "Use CVPosh to compare my CV with the job posting below. Show matched and missing keywords and quote where each comes from, so I can approve what to add. [paste posting]"
- "Read my CV with CVPosh and draft three stronger bullets for my most recent job. Show each next to the original; I approve before anything goes into my CV."

More prompts, the agent quickstart, error handling and the human-oversight controls: https://cvposh.com/mcp (markdown: https://cvposh.com/mcp.md)

## Sign-in: API key (not OAuth) in Cursor

CVPosh's OAuth sign-in only accepts the callbacks of Claude, ChatGPT and local loopback clients (`http://localhost` / `127.0.0.1`). Cursor's own callback (`cursor://`) is not on that list, so **in Cursor you connect with a CVPosh API key** sent as an `Authorization: Bearer` header.

1. Sign in at https://cvposh.com, open Settings > API keys and create a key. Treat it like a password: never commit it or paste it into a public file. Delete the key there to disconnect.
2. Put the key in an environment variable (for example `CVPOSH_API_KEY`) that Cursor can see, or replace the placeholder in the config below. Cursor expands `${env:NAME}` in `mcp.json`.

## Install in Cursor

Add this to `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project):

```json
{
  "mcpServers": {
    "cvposh": {
      "url": "https://cvposh.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ${env:CVPOSH_API_KEY}"
      }
    }
  }
}
```

Or use the deeplink, which opens Cursor and asks you to confirm adding the `cvposh` server with a placeholder key; replace `YOUR_API_KEY` with your key in Cursor's MCP settings afterwards:

[![Add CVPosh to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/link/mcp/install?name=cvposh&config=eyJ1cmwiOiJodHRwczovL2N2cG9zaC5jb20vYXBpL21jcCIsImhlYWRlcnMiOnsiQXV0aG9yaXphdGlvbiI6IkJlYXJlciBZT1VSX0FQSV9LRVkifX0)

App link: `cursor://anysphere.cursor-deeplink/mcp/install?name=cvposh&config=eyJ1cmwiOiJodHRwczovL2N2cG9zaC5jb20vYXBpL21jcCIsImhlYWRlcnMiOnsiQXV0aG9yaXphdGlvbiI6IkJlYXJlciBZT1VSX0FQSV9LRVkifX0`

Then open Customize, then MCPs: `cvposh` should list four tools. Optional: copy `rules/cvposh.mdc` to `.cursor/rules/cvposh.mdc` in your project (it is set to "Agent Decides"), and the folders in `skills/` to `~/.cursor/skills/`.

Cursor asks you to approve MCP tool calls by default. Leave that on.

## Install in Grok Build

```bash
grok mcp add --transport http cvposh https://cvposh.com/api/mcp --header "Authorization: Bearer YOUR_API_KEY"
```

Grok Build also loads `.cursor/mcp.json`. We have not tested Grok Build with this server.

## What it contains

| Component | Name | What it does |
|---|---|---|
| MCP server | `cvposh` (`https://cvposh.com/api/mcp`) | Four read-only tools: `list_cvs`, `get_cv`, `ats_check`, `analyze_keywords`. |
| Rule | `rules/cvposh.mdc` | Read-only, personal data, honest tailoring, scores only from tool output. |
| Skill | `review-my-cv` | Read a CV, run the ATS check, get prioritised edits to apply yourself. |
| Skill | `tailor-cv-to-job` | Compare a CV with a job posting: matched and missing keywords, honest rewording suggestions. |

## Tools

| Tool | What it does |
|---|---|
| `list_cvs` | List your saved CVs: id, title, template, last updated. No CV content. |
| `get_cv` | Read one CV as structured data and plain text. Profile photos are never returned. |
| `ats_check` | Score a saved CV (`cvId`) or pasted `resumeText` for ATS compatibility, 0-100, with issues and suggestions. |
| `analyze_keywords` | Compare a saved CV or `resumeText` with a `jobDescription`: matched keywords, missing keywords, match score. |

The ATS and keyword checks are fixed rules run on CVPosh's server; they do not call an AI model and store nothing.

## Use

Try the prompts in [examples/prompts.md](examples/prompts.md).

## What it will and will not do

- It reads. It never edits, deletes or shares a CV. It suggests wording; you apply it in the CVPosh editor.
- It only reads **your own** CVs, as the signed-in key owner. A resume is personal data: only connect an agent you trust with it.
- It does not invent experience: missing keywords you cannot honestly claim are named as gaps.
- Nobody can promise a CV will pass an ATS or lead to an interview.

## Data and privacy

The MCP server receives what the tools need: CV ids, and any resume or job text you ask your agent to send for checking, and returns your own CV content when you ask to read one. This repository stores nothing and runs no code on your machine; it is a server address, Markdown instructions and a logo. See the [privacy policy](https://cvposh.com/privacy) and [terms](https://cvposh.com/terms).

## Layout

```text
.cursor-plugin/plugin.json   Cursor Plugin manifest (also read by Grok Build)
mcp.json                     MCP server for Cursor (API key header)
.mcp.json                    the same server for Grok Build and other loaders
rules/cvposh.mdc             rule
skills/                      two skills (SKILL.md)
assets/logo.png              1024x1024 logo
examples/prompts.md          example prompts
```

## Support

support@cvposh.com or https://cvposh.com.

## Links

- Site: https://cvposh.com
- MCP page (setup for every client, quickstart, oversight): https://cvposh.com/mcp
- Pricing: https://cvposh.com/pricing
- Official MCP Registry: https://registry.modelcontextprotocol.io/v0/servers?search=cvposh
- Other clients: https://github.com/airdropfi/cvposh-claude-plugin, https://github.com/airdropfi/cvposh-gemini-extension, https://github.com/airdropfi/cvposh-cursor-kit
- Privacy: https://cvposh.com/privacy#ai-assistants

## License

MIT for the files in this repository (see `LICENSE`). The CVPosh service is governed by its terms.

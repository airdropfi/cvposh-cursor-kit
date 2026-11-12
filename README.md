# CVPosh for Cursor and Grok Build

Work on the resumes you built at [CVPosh](https://cvposh.com) without copy and paste, from Cursor or Grok Build. Everything is read-only: your agent can read your CVs, score them for applicant tracking systems (ATS) and compare them with a job posting, but it cannot change, delete or share them.

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

## License

MIT for the files in this repository (see `LICENSE`). The CVPosh service is governed by its terms.

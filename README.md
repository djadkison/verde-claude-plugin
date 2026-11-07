# Verde

[Verde](https://www.getverde.ai) is shared memory for a team and its AI. A
team keeps its knowledge in vaults, vaults hold buckets, and buckets hold
documents. Every document has a visibility level: public, company, or
private. This plugin connects Claude to that memory so it can look up what
your team has already decided and write down what it decides next.

## What the plugin adds

- **The Verde connector**: Verde's hosted MCP server at
  `https://www.getverde.ai/api/mcp`. Its tools search and read documents,
  their history and comments; create, update, supersede, archive and publish
  documents; organize buckets and their typed properties; and read and add
  files in the team's file library.
- **The `verde-memory` skill**: how to use those tools well. It covers when
  to search Verde before answering a question about your team, how to save one
  fact per document in the right bucket without creating duplicates, how to
  keep documents current without losing their history, and the confirmation
  step before anything goes on the public web.

## Getting started

You need a Verde account; you can create one at
[getverde.ai](https://www.getverde.ai). In Claude Code you can also install
the plugin straight from this repository:

```
/plugin install verde --marketplace djadkison/verde-claude-plugin
```

The first time Claude uses a Verde
tool, you sign in to Verde in your browser and approve the connection. The
server uses OAuth 2.1 with PKCE and registers the client dynamically, so this
plugin contains no API keys and asks you for none.

On the approval screen you choose what the connection can reach (every team
you belong to, or a narrower set) and whether Automatic Memory is on.

## What Claude can and cannot do

- Claude acts with your Verde permissions, never more. It reaches only the
  vaults and visibility levels your account already has.
- Writing a public document always leaves a draft. Taking it live is a
  separate call that the server refuses unless it carries an explicit
  confirmation, and the skill tells Claude to show you the exact text and get
  your approval first.
- With **Automatic Memory** on, Claude may save clearly durable, non-sensitive
  information without asking each time. Those saves are always your own
  private drafts, Claude tells you what it saved, and nothing is shared or
  published. You can turn it off per connection under MCP Connections in Verde.
- Edits keep the earlier text as a revision, and archived documents can be
  restored.

To disconnect, revoke the connection under MCP Connections in Verde, or
uninstall the plugin.

## Privacy and support

- Privacy policy: <https://www.getverde.ai/privacy>
- Terms of service: <https://www.getverde.ai/terms>
- Support: <https://www.getverde.ai/support> or hello@getverde.ai

## License

The configuration and text in this plugin are released under the MIT License
(see `LICENSE`). The Verde name and logo are trademarks of Enthusiast, LLC and
are not licensed for reuse.

# LLM SkillHub for Cursor

Gives Cursor ready-made agent skills from [LLM SkillHub](https://llmskillhub.com):
a growing catalogue of SKILL.md packages and the evals that test them, where
every public version is scanned for leaked credentials and prompt injection
and read by a person before it's listed.

## What it adds

- **MCP server** `llmskillhub` (`https://api.llmskillhub.com/mcp`, no key
  needed): search the catalogue by describing a task, read any skill's
  files, browse categories, and get the exact install command. Every tool is
  read-only.
- **Skill `find-skills`**: when you ask for something a published skill
  already does, Cursor searches, reads the skill, tells you what it needs
  access to, and installs it into your project or follows it for the task,
  once you agree.
- **Skill `skill-evals`**: finds evals (reviewed test suites), attaches them
  to your own skills, runs them and reads the scores, and helps you write and
  publish an eval.
- **Skill `publish-skill`**: packages your own instructions as a skill and
  publishes it with the `llmsh` CLI.

## Usage

Ask in chat, for example:

```
Find a skill on LLM SkillHub for reviewing pull requests and show me what it does.
```

To keep a skill in your project, install it for Cursor:

```bash
npm install -g llmskillhub            # the llmsh CLI, once
llmsh install owner/skill@version --for cursor
```

This puts the skill in `.cursor/skills/` in your project (or `~/.cursor/skills/`
outside one). Cursor loads it natively and uses it when a task matches the
skill's description, or when you type `/skill-name` in chat.

## Configuration

None. The MCP server is public and read-only, with no account, sign-in or API
key. Installing skills and publishing use the `llmsh` CLI; publishing needs an
account (`llmsh login`).

## Data handling

| What | Sent to | Kept |
|---|---|---|
| Catalogue searches and file reads | `api.llmskillhub.com`, through the MCP server | The search text is in the request body and isn't logged or stored. Each request's IP address and user agent are in the server's request log, deleted after 30 days |
| Skills and evals you choose to publish (`llmsh publish`) | `api.llmskillhub.com`, through the `llmsh` CLI | Until you remove them |
| Installing the CLI (`npm install -g llmskillhub`) | `registry.npmjs.org` | npm's own policy |

Nothing is installed or published without you asking. See the
[privacy policy](https://llmskillhub.com/privacy) and
[terms](https://llmskillhub.com/terms).

## License

MIT

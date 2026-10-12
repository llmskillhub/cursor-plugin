---
name: publish-skill
description: >
  Turn reusable agent instructions into a skill package and publish it to
  LLM SkillHub (llmskillhub.com) for human review: folder layout, SKILL.md
  frontmatter the validator accepts, writing the body for an agent, then
  validate and publish with the llmsh CLI. Use when the user wants to share,
  package, publish or update a skill, or asks why llmsh validate rejected one.
---

# Publish a skill to LLM SkillHub

## When to reach for this

The user has instructions worth reusing (a workflow, a review checklist, a
tool integration) and wants others to be able to install them. Or they have a
skill folder already and want to publish it, release a new version, or fix a
validation error.

## Process

1. **Look first.** Check whether the user already has a skill folder
   (a directory with a `SKILL.md`). If they do, read it before changing
   anything. Check the CLI: `llmsh --version`. If it's missing, the user
   installs it with `npm install -g llmskillhub`, then signs in with
   `llmsh login`.

2. **Lay out the folder.** The folder name is the skill's name. Only
   `SKILL.md` is required, and only these four subdirectories are recognised:

   ```
   my-skill/
   ├── SKILL.md      # required: manifest and agent instructions
   ├── scripts/      # helper scripts the skill runs
   ├── references/   # reference docs and data
   ├── assets/       # templates, images, static files
   └── examples/     # usage examples
   ```

   `llmsh init my-skill` scaffolds this and asks for what the validator needs.

3. **Write the frontmatter.** `name` and `description` are required.

   ```yaml
   ---
   name: my-skill
   description: >
     What it does and when to use it, including phrases users would search
     for. 40 to 1024 characters, no angle brackets.
   license: MIT
   compatibility: "Requires Python 3.11+"
   metadata:
     llmskillhub:
       version: 0.1.0
       categories: [coding]
       keywords: [refactoring, python]
       repository: https://github.com/you/my-skill
       authors:
         - name: Your Name
           handle: your-hub-handle
       capabilities:
         network: false
         filesystem: read      # none | read | read-write
         shell: false
         secrets: []           # named secrets the skill needs
   ---
   ```

   Rules the validator enforces:
   - `name`: lowercase letters, digits and single hyphens, at most 64
     characters, identical to the folder name.
   - `authors`: a list of objects with `name` (and optional `handle`,
     `email`), not a list of strings. `authors: ["Your Name"]` is rejected
     with `cannot unmarshal !!str into skillpkg.Author`.
   - `categories`: use the catalogue's built-in ones: documents, data,
     coding, devops, web, writing, design, research, business,
     communication, security, agents. Any other value creates a new
     category and draws a warning.
   - `capabilities`: always declare them. They're shown on the listing page.

4. **Write the body for an agent, not a human reader.** Use three sections:
   - `## When to reach for this`: the situation in the user's words. The
     agent decides from this whether the skill applies.
   - `## Process`: numbered steps in order. Inspect first, then do the work,
     then verify with an observable result ("prints `OK`", "exit code 0").
     Give exact commands, paths and expected output.
   - `## What not to do`: known failure modes. Usually worth more than the
     happy path.

5. **Validate locally:** `llmsh validate path/to/my-skill`. Fix every `✗`
   error; `!` warnings are allowed but read them. Success ends with
   `no problems found`.

6. **Dry-run on the server:** `llmsh publish --dry-run path/to/my-skill`.
   It reports description and completeness scores and stores nothing. If the
   folder has uncommitted git changes, `llmsh` refuses; commit them, or pass
   `--allow-dirty` if that's intended.

7. **Publish:** `llmsh publish path/to/my-skill`. Expected output ends with
   `submitted · digest … · pending`. A person reviews every public version
   before it's listed. Check progress with `llmsh status my-skill`.

8. **Release an update:** bump `metadata.llmskillhub.version`, then validate
   and publish again. The previous version stays live until the new one is
   approved. `llmsh diff path/to/my-skill` shows what changed against what's
   published.

## Limits

| Constraint | Limit |
|---|---|
| Compressed archive | 5 MiB |
| Uncompressed total | 25 MiB |
| Files | 500 |
| Single file | 4 MiB |
| `SKILL.md` | 256 KiB |
| Path depth | 12 levels |

## What not to do

- **Don't embed secrets or credentials.** The review scan rejects them.
- **Don't use angle brackets in `description`.**
- **Don't write vague steps.** "Set up the environment" is useless to an
  agent; "run `pip install -r requirements.txt` in the project root" works.
- **Don't let `name` and the folder name differ.**
- **Don't publish without being asked.** Publishing submits the user's work
  for public listing; confirm with them first, and use
  `llmsh publish --private` if they only want it in their own space.

# Hashnode GQL Agent Skill: publish blog posts with AI agents

The official [agent skill](https://skills.sh) for the **Hashnode GraphQL API**
(`https://gql-beta.hashnode.com`). It turns Hashnode into an agent-based
blogging platform. AI coding agents such as **Claude Code, Cursor, GitHub
Copilot, and any agent that supports skills** learn how to read and write your
Hashnode blog correctly: publish and update posts, manage drafts, upload
images, and paginate feeds, with the auth, Pro-plan, and rate-limit rules
built in.

## Add the skill to Claude Code (or any agent)

```bash
npx skills add Hashnode/gql-skill
```

This installs the `gql-api` skill. The agent loads it automatically whenever
you ask it to work with your Hashnode blog or the Hashnode API. No manual
setup is needed beyond exporting your token.

## What your agent can do with it

- **Publish a blog post with AI**: draft in your editor, then "publish this
  to my Hashnode blog" (`publishPost`, `createDraft` then `publishDraft`).
- **Automate blog publishing**: cross-post, backdate, schedule via drafts,
  set SEO/OG metadata, add posts to a series, publish as co-authors.
- **Upload images**: two-step presigned S3 flow for covers and OG images.
- **Read anything public**: posts, feeds, users, tags, and publications,
  with correct cursor pagination.
- **Respect the rules**: the skill teaches the agent Hashnode's Pro-plan
  gating, page-size caps, query-depth limits, and token hygiene, so requests
  don't fail in loops.

## What's inside

```
skills/gql-api/
├── SKILL.md                    # endpoint, auth, Pro-gating rules, agent rules
└── references/
    ├── schema.graphql          # full introspection schema (SDL), canonical type reference
    ├── queries.md              # all queries: args, returns, auth/Pro flags
    ├── mutations.md            # all mutations: inputs, payloads, auth/Pro flags
    ├── auth-and-roles.md       # PAT setup, public vs auth, roles, contributor flow
    ├── errors-and-limits.md    # error codes, page caps, depth, payload/image limits
    └── recipes.md              # publish a post, paginate, upload an image
```

`schema.graphql` is the full SDL from `gql-beta` introspection (every type, field,
argument, and input) and is the canonical reference for exact names. The curated
Markdown files layer on the auth and Pro-gating behavior the schema can't express.

## FAQ

### How do I get a Hashnode API key (Personal Access Token)?

Generate a PAT in the Hashnode dashboard (**Account Settings → Developer /
API tokens**) and export it in your shell:

```bash
export HASHNODE_PAT=...
```

Treat it like a password: it grants full write access to your publications.
Don't paste it into chats, scripts, or commits. Authenticated requests send it
by shell interpolation only: `Authorization: Bearer $HASHNODE_PAT`.

### Is the Hashnode API free?

Public reads (posts, feeds, users, tags) are free and need no token. Write
mutations and publication-scoped reads require the target publication to have
an active [**Hashnode Pro**](https://hashnode.com/pro) plan. See the
[changelog announcement](https://hashnode.com/changelog/2026-05-13-graphql-api-paid-access)
and the full gating rules in [skills/gql-api/SKILL.md](skills/gql-api/SKILL.md).

### Agent skill vs. MCP server: which is this?

This is a **skill**: a set of instructions and references the agent loads into
context, so it can call the GraphQL API directly with plain HTTP. There is no
extra server process, and it works in any skills-compatible agent.

### Which agents does it work with?

Any agent that supports the [skills format](https://skills.sh): Claude Code,
Cursor, GitHub Copilot, OpenCode, and others.

## Using the API directly

1. Get a PAT (see FAQ above) and `export HASHNODE_PAT=...`.
2. Send it on authenticated requests via shell interpolation:
   `Authorization: Bearer $HASHNODE_PAT`.
3. POST GraphQL to `https://gql-beta.hashnode.com/`.

## Maintenance

This repo is the public distribution surface for the skill. The source of truth
lives alongside the API in the private `Hashnode/gql` repo
(`skills/gql-api/`), generated from the GraphQL schema so the docs don't drift.
Changes are published here.

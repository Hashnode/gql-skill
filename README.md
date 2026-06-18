# Hashnode gql skill

An [agent skill](https://skills.sh) for the Hashnode GraphQL API
(`https://gql-beta.hashnode.com`). It teaches AI coding agents — Claude Code,
Cursor, Copilot, and others — how to read and write Hashnode content correctly:
the endpoint, Personal Access Token auth, which operations need a Pro plan, the
limits that cause requests to fail, and a full query/mutation reference.

## Install

```bash
npx skills add Hashnode/gql-skill
```

This installs the `gql-api` skill into your agent. The agent loads it
automatically when you ask it to work with the Hashnode gql API.

## What's inside

```
skills/gql-api/
├── SKILL.md                    # endpoint, auth, Pro-gating rules, agent rules
└── references/
    ├── schema.graphql          # full introspection schema (SDL) — canonical type reference
    ├── queries.md              # all queries — args, returns, auth/Pro flags
    ├── mutations.md            # all mutations — inputs, payloads, auth/Pro flags
    ├── auth-and-roles.md       # PAT setup, public vs auth, roles, contributor flow
    ├── errors-and-limits.md    # error codes, page caps, depth, payload/image limits
    └── recipes.md              # publish a post, paginate, upload an image
```

`schema.graphql` is the full SDL from `gql-beta` introspection (every type, field,
argument, and input) and is the canonical reference for exact names. The curated
Markdown files layer on the auth and Pro-gating behavior the schema can't express.

## Using the API

1. Get a Personal Access Token from the Hashnode dashboard
   (Account Settings → Developer / API tokens).
2. Send it on authenticated requests: `Authorization: Bearer YOUR_PAT`.
3. POST GraphQL to `https://gql-beta.hashnode.com/`.

Write mutations and publication-scoped reads require the target publication to
have an active Pro plan. See `skills/gql-api/SKILL.md` for the full rules.

## Maintenance

This repo is the public distribution surface for the skill. The source of truth
lives alongside the API in the private `Hashnode/gql` repo
(`skills/gql-api/`), generated from the GraphQL schema so the docs don't drift.
Changes are published here.

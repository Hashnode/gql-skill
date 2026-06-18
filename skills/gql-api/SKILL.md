---
name: gql-api
description: >-
  Query and mutate content on the Hashnode GraphQL API (posts, drafts,
  publications, users, tags, series, comments). Use when a user wants to read
  or write data through the Hashnode gql endpoint, needs a reference for a
  specific query or mutation, is wiring up a Personal Access Token, or hits a
  "Publication does not have an active Pro plan" / FORBIDDEN error and needs to
  know an operation is Pro-gated.
---

# Hashnode GraphQL API

A GraphQL API (Apollo Server) for publishing and managing content on Hashnode:
posts, drafts, publications, users, tags, series, and comments. This skill tells
an agent how to talk to it correctly — endpoint, auth, which operations need a
Pro plan, the limits that cause requests to fail, and where to find the full
field-level reference.

## Endpoint

| Environment | URL | Path |
|-------------|-----|------|
| Production  | `https://gql-beta.hashnode.com` | `/` (POST GraphQL here) |
| Local dev   | `http://localhost:8080` | `/graphql` |

Standard GraphQL over HTTP `POST` with a JSON body (`{ "query": "...", "variables": {...} }`).
Introspection is enabled, so the schema is browsable from any GraphQL IDE.

## Authentication

Auth uses a **Personal Access Token (PAT)**. Send it as a bearer token:

```
Authorization: Bearer YOUR_PAT
```

Get a PAT from the Hashnode dashboard (Account Settings → Developer / API tokens).

- **Public reads** need no token: `post`, `feed`, `user`, `tag`, `publication`,
  `documentationProject`, `checkCustomDomainAvailability`, `checkSubdomainAvailability`.
- **Authenticated** operations need a valid PAT: `me`, `draft`, `scheduledPost`,
  and **every mutation**. A missing/invalid token returns `UNAUTHENTICATED`.

See [references/auth-and-roles.md](references/auth-and-roles.md) for the role model
(OWNER / EDITOR / CONTRIBUTOR) and the contributor review workflow.

## Pro plan gating

Some operations require the **target publication** to have an active Pro plan.
This is keyed to the publication, not the calling user. When the publication is
not Pro, the API returns:

```json
{
  "errors": [{
    "message": "Publication does not have an active Pro plan. Upgrade in your dashboard to access this via the API.",
    "extensions": { "code": "FORBIDDEN" }
  }]
}
```

**When you see this, tell the user the publication needs to upgrade to Pro in the
Hashnode dashboard. Do not retry the request — it will keep failing until the
publication is on Pro.**

Pro-gated operations:

- **All write mutations:** `publishPost`, `updatePost`, `createDraft`,
  `updateDraft`, `publishDraft`, `submitDraftForReview`, `rejectDraftSubmission`,
  `deleteDraft`.
- **Publication-scoped reads:** `publication`, `draft`, `scheduledPost`,
  `searchPostsOfPublication`, `topCommenters`. (`draft` and `scheduledPost` also
  require a PAT and authorization; the owning publication must be Pro.)

Public feed/post/user/tag reads are **not** Pro-gated.

## Reference

- [references/schema.graphql](references/schema.graphql) — full SDL (introspection schema) from `gql-beta`: the canonical source for every type, field, argument, and input. Use this when you need exact field names or types; use the curated files below for auth/Pro behavior the schema can't express.
- [references/queries.md](references/queries.md) — all 13 queries: arguments, return types, auth/Pro notes.
- [references/mutations.md](references/mutations.md) — all 9 mutations: inputs, payloads, auth/Pro notes.
- [references/auth-and-roles.md](references/auth-and-roles.md) — PAT setup, public vs. authenticated, roles, contributor review flow.
- [references/errors-and-limits.md](references/errors-and-limits.md) — error codes, page-size caps, query depth, payload/image limits.
- [references/recipes.md](references/recipes.md) — end-to-end examples: publish a post, paginate a feed, upload an image.

## Rules for the agent

1. Always send `Authorization: Bearer <PAT>` for `me`, `draft`, `scheduledPost`,
   and any mutation. Without it you get `UNAUTHENTICATED`.
2. On `FORBIDDEN` + the Pro-plan message, stop and tell the user to upgrade the
   publication to Pro. Don't retry.
3. Respect page-size caps: most connections cap `first` at **100**, drafts at
   **50**. Asking for more is silently clamped.
4. Keep query depth at or below **10** — deeper queries are rejected.
5. Tag inputs use **slug** (e.g. `javascript`), max **15** per post/draft.
6. Pagination is cursor-based: read `pageInfo.endCursor` / `pageInfo.hasNextPage`,
   pass `endCursor` as the next `after`.
7. Mutations are never cached; queries may be cached for up to ~25s.

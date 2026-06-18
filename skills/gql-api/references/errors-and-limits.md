# Errors & limits

## Error codes

Errors arrive in the standard GraphQL shape with a code in `extensions.code`:

```json
{
  "errors": [{
    "message": "Post not found",
    "extensions": { "code": "NOT_FOUND" },
    "path": ["post"]
  }]
}
```

| Code | Meaning | Agent action |
|------|---------|--------------|
| `UNAUTHENTICATED` | Missing/invalid PAT on an authenticated operation. | Add a valid `Authorization: Bearer` header. |
| `FORBIDDEN` | Insufficient permission **or** publication not on Pro. | If the message mentions the Pro plan, tell the user to upgrade — don't retry. Otherwise the user lacks the required role. |
| `NOT_FOUND` | Resource missing **or** intentionally hidden. | Don't assume auth fixes it: an inaccessible draft returns `NOT_FOUND` by design, not `FORBIDDEN`. |
| `BAD_USER_INPUT` | Invalid arguments/input. | Fix the variables. |
| `INTERNAL_SERVER_ERROR` | Server error (details are not exposed). | Retry later; report if persistent. |

Only these codes are returned with real messages; anything else is collapsed to
`INTERNAL_SERVER_ERROR` with a generic message.

## The Pro-plan error (most common gotcha)

```
message: "Publication does not have an active Pro plan. Upgrade in your dashboard to access this via the API."
code:    FORBIDDEN
```

Triggered by all write mutations and by `publication` / `searchPostsOfPublication`
/ `topCommenters` when the target publication is not on Pro. Resolution is to
upgrade the publication to Pro in the dashboard — retrying the same call will not
help.

## Limits

| Limit | Value |
|-------|-------|
| Max query depth | **10** (deeper queries rejected) |
| Page size — most connections (`first`) | **100** max |
| Page size — draft connections | **50** max |
| Request body size | **100 KB** |
| Image upload size | **8 MB**, `image/*` only, **no SVG** |
| Tags per post/draft | **15** max |

Page sizes above the cap are clamped, not errored — don't rely on getting more
than the cap in one page; paginate instead.

## Rate limits

The published targets are 20,000 queries/min and 500 mutations/min, but rate
limiting is **not currently enforced** by the server. Treat these as advisory and
don't build logic that depends on hitting (or being blocked at) them.

## Caching

Query responses may be cached for up to ~25 seconds. Mutations are never cached.
A read issued immediately after a write may briefly return stale data.

## Pagination

All list fields are cursor-based connections:

```graphql
feed(first: 10, after: $cursor) {
  edges { node { id } cursor }
  pageInfo { hasNextPage endCursor }
}
```

Loop: pass `pageInfo.endCursor` as the next `after`; stop when
`pageInfo.hasNextPage` is `false`.

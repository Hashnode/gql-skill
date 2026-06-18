# Authentication & roles

## Personal Access Token (PAT)

The API authenticates with a Personal Access Token. Generate one in the Hashnode
dashboard (Account Settings → Developer / API tokens) and send it on every
authenticated request:

```
Authorization: Bearer YOUR_PAT
```

The `Bearer ` prefix is optional and case-insensitive. A deactivated user's token
is treated as unauthenticated.

## Public vs. authenticated operations

| Operation | Needs PAT? |
|-----------|-----------|
| `post`, `feed`, `user`, `tag`, `publication`, `documentationProject` | No |
| `checkCustomDomainAvailability`, `checkSubdomainAvailability` | No |
| `searchPostsOfPublication`, `topCommenters` | No token, but Pro publication required |
| `me` | Yes |
| `draft`, `scheduledPost` | Yes, and the owning publication must be Pro |
| All mutations | Yes |

A missing or invalid token on an authenticated operation returns `UNAUTHENTICATED`.

> Note: `publication`, `draft`, `scheduledPost`, `searchPostsOfPublication`, and
> `topCommenters` are all publication-scoped reads, so the publication they target
> must be on the Pro plan or they return `FORBIDDEN`. See the Pro gating section in
> SKILL.md.

## Roles (team publications)

`UserPublicationRole`:

| Role | Capabilities |
|------|--------------|
| `OWNER` | Creator of the publication; can do everything. |
| `EDITOR` | Customize the blog, approve/reject posts, manage members. |
| `CONTRIBUTOR` | Join and contribute an article. Cannot publish directly. |

## Contributor review workflow

Contributors **cannot publish or directly create posts**. Their path is:

1. `createDraft` / `updateDraft` to write the draft.
2. `submitDraftForReview` to place it in the editors' review queue.
3. An owner/editor/author publishes it, or `rejectDraftSubmission` sends it back.

Contributors also can't list other authors' drafts. When generating calls for a
contributor, route writes through the review flow rather than `publishPost` /
`publishDraft`.

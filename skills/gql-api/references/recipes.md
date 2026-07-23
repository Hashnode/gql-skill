# Recipes

End-to-end flows that combine multiple calls. All examples assume a Pro
publication and an `Authorization: Bearer $HASHNODE_PAT` header, with the token
supplied via the `HASHNODE_PAT` environment variable (never inlined literally —
see [auth-and-roles.md](auth-and-roles.md)).

## Publish a post

Direct publish (owner/editor/author):

```graphql
mutation PublishPost($input: PublishPostInput!) {
  publishPost(input: $input) {
    post { id slug url title }
  }
}
```

Variables:

```json
{
  "input": {
    "publicationId": "PUBLICATION_OBJECT_ID",
    "title": "Shipping a GraphQL API",
    "contentMarkdown": "# Intro\n\nBody in **markdown**.",
    "tags": [{ "slug": "graphql" }, { "slug": "api" }],
    "enableToc": true
  }
}
```

If you get `FORBIDDEN` with the Pro-plan message, the publication isn't on Pro —
tell the user to upgrade.

## Draft, then publish

```graphql
mutation CreateDraft($input: CreateDraftInput!) {
  createDraft(input: $input) { draft { id } }
}

mutation PublishDraft($input: PublishDraftInput!) {
  publishDraft(input: $input) { post { id slug url } }
}
```

1. `createDraft` with `{ publicationId, title, contentMarkdown }` → returns `draft.id`.
2. `publishDraft` with `{ draftId: "<draft.id>" }` → returns the published post.
   The draft is soft-deleted on success.

## Contributor flow (submit for review)

Contributors can't publish. Submit instead:

```graphql
mutation Submit($input: SubmitDraftForReviewInput!) {
  submitDraftForReview(input: $input) { draft { id isSubmittedForReview } }
}
```

An owner/editor/author then publishes or calls `rejectDraftSubmission`.

## Upload an image (two steps)

`createImageUploadURL` returns a presigned S3 POST. You must then upload the file
to S3 yourself — the GraphQL call does not accept the bytes.

Step 1 — get the presigned POST:

```graphql
mutation ($input: CreateImageUploadInput!) {
  createImageUploadURL(input: $input) {
    presignedPost { url fields }
  }
}
```

with `{ "input": { "contentType": "image/png" } }`.

Step 2 — POST the file to `presignedPost.url` as `multipart/form-data`, including
every key/value from `presignedPost.fields` first, then the `file` field last:

```bash
curl -X POST "<presignedPost.url>" \
  -F "key=<fields.key>" \
  # ...all other entries from presignedPost.fields... \
  -F "file=@./cover.png"
```

The resulting object URL (built from the upload URL + key) is what you pass as
`coverImage` / `ogImage` in `publishPost` or as `coverImageOptions.coverImageURL`
in `createDraft`. Constraints: `image/*` only, no SVG, 8 MB max.

## Paginate a feed

```graphql
query Feed($after: String) {
  feed(first: 20, after: $after) {
    edges { node { id title url } cursor }
    pageInfo { hasNextPage endCursor }
  }
}
```

Call repeatedly, passing the previous `pageInfo.endCursor` as `after`, until
`hasNextPage` is `false`. Don't request `first` above 100 — it's clamped.

## Search within a publication (Pro)

```graphql
query Search($filter: SearchPostsOfPublicationFilter!) {
  searchPostsOfPublication(first: 10, sortBy: DATE_PUBLISHED_DESC, filter: $filter) {
    edges { node { id title publishedAt } }
    pageInfo { hasNextPage endCursor }
  }
}
```

```json
{
  "filter": {
    "publicationId": "PUBLICATION_OBJECT_ID",
    "query": "graphql",
    "time": { "relative": { "relative": "LAST_N_DAYS", "n": 30 } }
  }
}
```

## Link to a post's discussion or a comment on hashnode.com

The discussion URL needs the post's `slug` AND `id`; a comment permalink adds
the comment's `id`. Three ways to get them, in order of preference:

**1. You just published the post.** The `publishPost` / `publishDraft` payload
already returns `post { id slug url }`. Build the link from that; no extra
query needed.

**2. You have the post id.** The `post` query is public and not Pro-gated:

```graphql
query PostLinkParts($id: ID!) {
  post(id: $id) {
    id
    slug
    url
    comments(first: 20) {
      edges { node { id author { username } } }
    }
  }
}
```

**3. You only have the host and slug.** Use `publication(host:) { post(slug:) }`
with the same fields. Note the `publication` query is Pro-gated: it fails with
`FORBIDDEN` unless the target publication has an active Pro plan.

Then build:

- Discussion: `https://hashnode.com/posts/<slug>/<post.id>`
- Comment: `https://hashnode.com/posts/<slug>/<post.id>/comment/<comment.id>`
- Post on the author's blog: the post's `url` field, as returned.

Example with `slug: sealed-with-a-kyss-inside-an-android-banking-rat`,
`id: 6a603fb103e2cb323e7851f6`:

```
https://hashnode.com/posts/sealed-with-a-kyss-inside-an-android-banking-rat/6a603fb103e2cb323e7851f6
```

Legacy schemes (`/discussions/post/<id>`, id-only, slug-only) 404. Never emit
them, and never browse hashnode.com to reverse-engineer URL formats.

# Mutations

All 9 mutations. **Every mutation requires a valid PAT** (`UNAUTHENTICATED`
without one). All write mutations are **Pro-gated**: the target publication must
have an active Pro plan, or the call returns `FORBIDDEN` with the message
"Publication does not have an active Pro plan. Upgrade in your dashboard to
access this via the API." `createImageUploadURL` requires auth but is not
Pro-gated.

| Mutation | Input | Payload | Access |
|----------|-------|---------|--------|
| `publishPost` | `PublishPostInput!` | `PublishPostPayload!` (`post`) | Auth + Pro |
| `updatePost` | `UpdatePostInput!` | `UpdatePostPayload!` (`post`) | Auth + Pro |
| `createDraft` | `CreateDraftInput!` | `CreateDraftPayload!` (`draft`) | Auth + Pro |
| `updateDraft` | `UpdateDraftInput!` | `UpdateDraftPayload!` (`draft`) | Auth + Pro |
| `publishDraft` | `PublishDraftInput!` | `PublishDraftPayload!` (`post`) | Auth + Pro |
| `submitDraftForReview` | `SubmitDraftForReviewInput!` | `SubmitDraftForReviewPayload!` (`draft`) | Auth + Pro |
| `rejectDraftSubmission` | `RejectDraftSubmissionInput!` | `RejectDraftSubmissionPayload!` (`draft`) | Auth + Pro |
| `deleteDraft` | `DeleteDraftInput!` | `DeleteDraftPayload!` (`draft`) | Auth + Pro |
| `createImageUploadURL` | `CreateImageUploadInput!` | `CreateImageUploadPayload!` (`presignedPost`) | Auth |

## publishPost

Publish a new post directly to a publication.

```graphql
mutation ($input: PublishPostInput!) {
  publishPost(input: $input) { post { id slug url } }
}
```

`PublishPostInput`:
- `publicationId: ObjectId!` — target publication (must be Pro).
- `title: String!`
- `contentMarkdown: String!` — body in markdown.
- `subtitle, coverImage, slug` — optional. Slug auto-generated from title if omitted.
- `tags: [PublishPostTagInput!]` — each `{ slug: String!, name: String }`, max **15**. Unknown tags are created.
- `originalArticleURL` — canonical URL for republished articles.
- `metaTitle, metaDescription, ogImage` — SEO/OG overrides.
- `disableComments, isDelisted, enableToc` — booleans.
- `publishAs: ObjectId` — publish on behalf of another member (team publications only).
- `coAuthors: [ObjectId!]` — max 4, must be publication members.
- `seriesId: ObjectId` — add to a series.
- `publishedAt: DateTime` — backdate.

## updatePost

Update an existing post. `UpdatePostInput` mirrors `PublishPostInput` but keyed
by `id: ID!`; all content fields optional.

```graphql
mutation ($input: UpdatePostInput!) {
  updatePost(input: $input) { post { id slug url } }
}
```

## createDraft

Create a draft without publishing. `CreateDraftInput`:
- `publicationId: ObjectId!` (must be Pro).
- `title, subtitle, contentMarkdown, slug` — all optional.
- `tags` — `[PublishPostTagInput!]`, max 15.
- `seriesId, disableComments, originalArticleURL, publishedAt`.
- `settings: CreateDraftSettingsInput` — `{ enableTableOfContent, delist, activateNewsletter, slugOverridden }`.
- `metaTags: MetaTagsInput` — `{ title, description, image }`.
- `coverImageOptions: CoverImageOptionsInput` — `{ coverImageURL, coverImageAttribution, coverImagePhotographer, isCoverAttributionHidden, stickCoverToBottom }`.
- `publishAs: ObjectId`, `coAuthors: [ObjectId!]` (max 4, team publications).

```graphql
mutation ($input: CreateDraftInput!) {
  createDraft(input: $input) { draft { id title } }
}
```

## updateDraft

Update an existing draft. `UpdateDraftInput` mirrors `CreateDraftInput` but keyed
by `draftId: ID!`.

## publishDraft

Publish an existing draft as a post. The draft is soft-deleted once the post is
created.

```graphql
mutation ($input: PublishDraftInput!) {
  publishDraft(input: $input) { post { id slug url } }
}
```

`PublishDraftInput`: `{ draftId: ID! }`.

## submitDraftForReview

On a team publication, submit a draft to the editor review queue. Used by
**contributors**, who cannot publish directly (see auth-and-roles.md).

`SubmitDraftForReviewInput`: `{ draftId: ID! }`. Returns the updated `draft`.

## rejectDraftSubmission

Reject a draft previously submitted for review. Only publication owners, admins,
and authors can reject.

`RejectDraftSubmissionInput`: `{ draftId: ID! }`. Returns the updated `draft`.

## deleteDraft

Soft-delete a draft (sets it inactive). The draft's author can delete their own;
owners/admins/authors can delete any draft in the publication.

`DeleteDraftInput`: `{ draftId: ID! }`. Returns the (soft-deleted) `draft`.

## createImageUploadURL

Returns a presigned S3 POST for uploading an image. **Two-step flow** — see
[recipes.md](recipes.md#upload-an-image). Auth required; not Pro-gated.

`CreateImageUploadInput`: `{ contentType: String! }` — must start with `image/`
(e.g. `image/png`). SVG is rejected; max image size is **8 MB**.

```graphql
mutation ($input: CreateImageUploadInput!) {
  createImageUploadURL(input: $input) {
    presignedPost { url fields }
  }
}
```

`presignedPost.fields` is a `JSONObject` of form fields to include in the upload
POST body alongside the file.

# Queries

All 13 root queries. Auth column: **Public** = no token needed, **Auth** = valid
PAT required, **Pro** = the target publication must have an active Pro plan
(returns `FORBIDDEN` otherwise).

| Query | Args | Returns | Access |
|-------|------|---------|--------|
| `me` | — | `MyUser` | Auth |
| `post` | `id: ID!` | `Post` | Public |
| `publication` | `id: ObjectId, host: String` | `Publication` | Pro |
| `user` | `username: String!` | `User` | Public |
| `tag` | `slug: String!` | `Tag` | Public |
| `feed` | `first: Int!, after: String, filter: FeedFilter` | `FeedPostConnection!` | Public |
| `draft` | `id: ObjectId!` | `Draft` | Auth + Pro |
| `scheduledPost` | `id: ObjectId!` | `ScheduledPost` | Auth + Pro |
| `checkCustomDomainAvailability` | `input: CheckCustomDomainAvailabilityInput!` | `CheckCustomDomainAvailabilityResult!` | Public |
| `checkSubdomainAvailability` | `subdomain: String!` | `CheckSubdomainAvailabilityResult!` | Public |
| `searchPostsOfPublication` | `first: Int!, after: String, sortBy: PostSortBy, filter: SearchPostsOfPublicationFilter!` | `SearchPostConnection!` | Pro |
| `topCommenters` | `publicationId: ObjectId!, limit: Int` | `[User!]!` | Pro |
| `documentationProject` | `id: ID, host: String` | `DocumentationProject` | Public |

## me

Current authenticated user. Requires a PAT; returns `UNAUTHENTICATED` without one.

```graphql
query { me { id username name email } }
```

## post

Single post by id.

```graphql
query ($id: ID!) {
  post(id: $id) {
    id title subtitle slug url brief readTimeInMinutes
    content { markdown html }
    author { username name }
    tags { name slug }
    publishedAt
  }
}
```

## publication (Pro)

Publication by `id` or `host`. The publication must be on Pro, or the call
returns `FORBIDDEN`. Exposes nested connections for posts, drafts, series,
members, static pages, etc. (all cursor-paginated — see errors-and-limits.md).

```graphql
query ($host: String!) {
  publication(host: $host) {
    id title url isTeam followersCount
    seo { title description }
    posts(first: 10) { edges { node { id title } } pageInfo { hasNextPage endCursor } }
  }
}
```

`seo` returns only values explicitly set in the publication's SEO settings,
and `null` otherwise, with no fallback to the publication name or about text.
`descriptionSEO` is deprecated in favor of `seo.description`.

## user

Public profile by username, with nested `posts`, `publications`, `followers`,
`follows` connections.

```graphql
query ($username: String!) {
  user(username: $username) {
    id username name tagline followersCount followingsCount
    socialMediaLinks { twitter github website }
  }
}
```

## tag

Tag by slug, with a `posts` connection. Always reference tags by **slug**.

```graphql
query ($slug: String!) {
  tag(slug: $slug) { id name slug followersCount postsCount }
}
```

## feed

Paginated global feed. `first` is required. `FeedFilter` supports `tags`,
`excludeTags`, `publications`, `excludePublications`.

```graphql
query ($first: Int!, $after: String, $filter: FeedFilter) {
  feed(first: $first, after: $after, filter: $filter) {
    edges { node { id title brief url author { username } publishedAt } cursor }
    pageInfo { hasNextPage endCursor }
  }
}
```

## draft (Auth + Pro)

A single draft by id. Requires auth, that you are authorized for the draft (its
author, or an admin/author member or owner of its publication), and that the
draft's owning publication is on Pro. For privacy, a draft you cannot access
returns `NOT_FOUND` (not `FORBIDDEN`) — do not treat that as "retry with auth."
An authorized caller on a non-Pro publication gets `FORBIDDEN` with the Pro-plan
message.

## scheduledPost (Auth + Pro)

A scheduled post by id, including its underlying `draft`. Requires auth, that you
own the scheduled post (author or scheduler), and that the scheduled post's owning
publication is on Pro. A non-Pro publication returns `FORBIDDEN` with the Pro-plan
message.

## checkCustomDomainAvailability / checkSubdomainAvailability

Availability checks for blog setup. Both return `{ available, message }`.

```graphql
query ($input: CheckCustomDomainAvailabilityInput!) {
  checkCustomDomainAvailability(input: $input) { available message }
}
query { checkSubdomainAvailability(subdomain: "my-blog") { available message } }
```

## searchPostsOfPublication (Pro)

Full-text/filtered search scoped to one publication. `filter.publicationId` is
required; the publication must be Pro. Supports author/tag/time filters and
`sortBy` (`DATE_PUBLISHED_ASC` | `DATE_PUBLISHED_DESC`).

```graphql
query ($filter: SearchPostsOfPublicationFilter!) {
  searchPostsOfPublication(first: 10, filter: $filter) {
    edges { node { id title } }
    pageInfo { hasNextPage endCursor }
  }
}
```

`SearchPostsOfPublicationFilter`: `query`, `publicationId!`, `deletedOnly`,
`authorIds`, `tagIds`, `requiredTagsIds`, `time` (`TimeFilter` with `absolute`
`{from,to}` or `relative` `{relative: TimePeriod!, n: Int!}`).
`TimePeriod`: `LAST_N_HOURS | LAST_N_DAYS | LAST_N_WEEKS | LAST_N_MONTHS | LAST_N_YEARS`.

## topCommenters (Pro)

Top commenters for a publication. `publicationId` required; publication must be
Pro. Returns `[User!]!`.

## documentationProject

A docs project by `id` or `host`, with a `guides` connection.

```graphql
query ($host: String!) {
  documentationProject(host: $host) {
    id title slug
    guides(first: 10) { edges { node { id title } } }
  }
}
```

## Custom scalars

- `DateTime` — ISO 8601 string.
- `ObjectId` — MongoDB ObjectId as a string.
- `JSONObject` — arbitrary JSON object.

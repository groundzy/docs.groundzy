# Explore / Community Backend Design

Design for Firestore collections and rules to back the Explore drawer (feed, species, challenges, leaderboard). Aligns with [Visibility & Permission Model v2](visibility-permission-model-v2.md) Phase 4.

**Status**: Design specification. Implementation phased per plan.

---

## 1. Overview

The Explore drawer surfaces:

- **Feed**: Social posts tied to trees (photos, captions, reactions)
- **Species**: Trending / discovery (aggregated from tree data)
- **Challenges**: Gamification (contests, progress, rewards)
- **Leaderboard**: User rankings (trees documented, points)

All content must respect tree visibility: only **public** trees appear in community surfaces.

---

## 2. Collections

### 2.1 tree_posts (Community Feed)

Per [visibility-permission-model-v2](visibility-permission-model-v2.md), community content is separate from the canonical tree record.

**Option A: Subcollection per tree**

```
tree_posts/{treeId}/posts/{postId}
```

- `treeId` = parent tree (must be public)
- `postId` = auto-generated
- Fields: `authorId`, `caption`, `mediaUrls[]`, `createdAt`, `likeCount`, `commentCount`, `hashtags[]`, `aiConfidence?`, `status` (active|hidden|reported)

**Option B: Flat feed collection with tree reference**

```
community_posts/{postId}
```

- Fields: `treeId`, `authorId`, `caption`, `mediaUrls[]`, `createdAt`, `likeCount`, `commentCount`, `hashtags[]`, `aiConfidence?`, `status`
- Index: `createdAt` DESC for feed queries
- Eligibility: Tree must exist in `tree_public_summaries` with `visibilityLevel == 'public'`

**Recommendation**: Option A for locality (tree-scoped), with a **feed index** or Cloud Function to aggregate into a queryable feed for home timeline.

### 2.2 Post reactions and comments

```
tree_posts/{treeId}/posts/{postId}/likes/{userId}
tree_posts/{treeId}/posts/{postId}/comments/{commentId}
```

- Likes: document exists = liked
- Comments: `authorId`, `text`, `createdAt`

### 2.3 challenges

```
challenges/{challengeId}
```

- Fields: `title`, `description`, `type` (e.g. blossom, biggest_tree, fall_colors), `startAt`, `endAt`, `reward`, `criteria` (e.g. `{ type: "post_count", min: 5 }`), `status` (draft|active|ended)
- Admin-only write; authenticated read when `status == 'active'`

### 2.4 challenge_progress

```
challenge_progress/{userId}_{challengeId}
```

- Document ID: composite for single doc per user per challenge
- Fields: `userId`, `challengeId`, `progress`, `completedAt?`, `updatedAt`
- Write: user can update own progress (with validation against challenge criteria)

### 2.5 leaderboard_snapshots (or computed)

**Option A: Precomputed snapshots**

```
leaderboard_snapshots/{periodId}
```

- `periodId` = e.g. `2025-Q1` or `weekly-2025-W12`
- Fields: `entries[]` = `{ userId, username, treeCount, rank, avatarUrl? }`
- Updated by scheduled Cloud Function

**Option B: Query-time aggregation**

- Query `trees` (or `tree_public_summaries`) grouped by owner, count, order by count DESC
- Requires composite index; may be expensive at scale

**Recommendation**: Option A for performance; Option B for MVP if user base is small.

---

## 3. Security Rules (Draft)

### tree_posts

```
match /tree_posts/{treeId}/posts/{postId} {
  // Read: authenticated; tree must be public (check tree_public_summaries)
  allow read: if isAuthenticated() && treeIsPublic(treeId);
  // Create: authenticated; user is author; tree is public
  allow create: if isAuthenticated() && request.resource.data.authorId == request.auth.uid && treeIsPublic(treeId);
  // Update/delete: author only
  allow update, delete: if isAuthenticated() && resource.data.authorId == request.auth.uid;
}
```

Helper `treeIsPublic(treeId)` would need to read `tree_public_summaries/{treeId}` or `trees/{treeId}` — consider denormalizing `visibilityLevel` into post for rule simplicity.

### challenges

```
match /challenges/{challengeId} {
  allow read: if isAuthenticated() && resource.data.status == 'active';
  allow write: if isGlobalAdmin();
}
```

### challenge_progress

```
match /challenge_progress/{docId} {
  allow read: if isAuthenticated();
  allow create, update: if isAuthenticated() && request.resource.data.userId == request.auth.uid;
  allow delete: if false;
}
```

### leaderboard_snapshots

```
match /leaderboard_snapshots/{periodId} {
  allow read: if isAuthenticated();
  allow write: if isGlobalAdmin();
}
```

---

## 4. Moderation

Before wide launch:

- **Report post**: `community_reports/{reportId}` — `postRef`, `reporterId`, `reason`, `createdAt`
- **Admin tooling**: List reports, hide/remove post, warn/block user
- **Block user**: `user_blocks/{blockerId}/blocked/{blockedId}` — hide blocked users' content from blocker's feed

---

## 5. Indexes

- `community_posts` (if Option B): `createdAt` DESC
- `tree_posts/{treeId}/posts`: `createdAt` DESC
- `challenge_progress`: `challengeId`, `progress` DESC (for leaderboard within challenge)
- `leaderboard_snapshots`: by `periodId`

---

## 6. Migration Path

1. **Phase A** (current): Skeleton loading + empty states; no Firestore.
2. **Phase B**: Add `tree_posts` (or `community_posts`), challenges, challenge_progress; wire Explore drawer to real data.
3. **Phase C**: Leaderboard snapshots, moderation collections, report/block UI.

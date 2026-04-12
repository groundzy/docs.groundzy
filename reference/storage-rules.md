# Firebase Storage Rules Reference

Storage rules are in `firebase/storage.rules`. Deploy with `npx firebase deploy --only storage`.

## Paths

### Tree Photos

`organizations/{organizationId}/trees/{treeId}/photos/{photoId}`

- **Read**: Authenticated + user in organization
- **Write**: Authenticated + in org + `contentType` matches `image/*` + size &lt; 10MB
- **Delete**: Authenticated + in org

### User Gallery (My Photos)

`users/{userId}/gallery/{photoId}`

- **Read**: Authenticated + owner
- **Write**: Authenticated + owner + `image/*` + size &lt; 10MB
- **Delete**: Authenticated + owner

### Profile Avatar

`users/{userId}/profile/avatar`

- **Read**: Any authenticated user (for avatars, team display)
- **Write**: Owner + `image/(jpeg|png|webp)` + size &lt; 3MB
- **Delete**: Owner

### Social Assets

`social-assets/{path}`

- **Read**: Public (for Instagram publishing)
- **Write**: Global admin only (`admins/{uid}` exists)

### Default

All other paths: deny read and write.

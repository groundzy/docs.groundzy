# Firebase Configuration

Firebase project configuration.

## Files

| File | Purpose | Doc |
|------|---------|-----|
| firebase.json | Firebase project config | Root config |
| firestore.rules | Firestore security rules | [reference/firestore-collections.md](../reference/firestore-collections.md) |
| firestore.indexes.json | Firestore composite indexes | [reference/firestore-indexes.md](../reference/firestore-indexes.md) |
| storage.rules | Storage security rules | [reference/storage-rules.md](../reference/storage-rules.md) |
| storage.cors.json | Storage CORS | CORS for uploads |

## Deploy

```bash
npx firebase deploy --only firestore
npx firebase deploy --only firestore:indexes
npx firebase deploy --only storage
```

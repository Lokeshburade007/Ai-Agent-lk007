# MongoDB Debugging Playbook

## Query returns nothing — but you can see the doc in Compass

1. **`_id` type mismatch.** The doc's `_id` is `ObjectId`, you're searching with a string:
```js
await Model.findOne({ _id: req.params.id });        // ❌ string
await Model.findById(req.params.id);                // ✅ Mongoose auto-converts
await Model.findOne({ _id: new mongoose.Types.ObjectId(id) });  // ✅ explicit
```

2. **Case sensitivity.** `{ email: 'User@Example.com' }` doesn't match `user@example.com`. Use case-insensitive regex or a unique normalized index.

3. **Wrong database/collection.** You're connected to `mydb_dev`, doc is in `mydb_prod`. Print `mongoose.connection.name`.

4. **Schema filtering.** Mongoose strips fields not in the schema. If your query references a non-schema field, it's removed before being sent.

5. **`select(...)` excluded what you needed.** `.select('-password')` then accessing `.password` later returns `undefined`.

## Slow query — N+1, missing index, full collection scan

1. Run with `.explain('executionStats')`:
```js
await Model.find({ status: 'active' }).explain('executionStats');
```
Look at `executionStats.totalDocsExamined` vs `nReturned`. If examined ≫ returned, you're missing an index.

2. Add the index:
```js
collection.createIndex({ status: 1 });
// or in Mongoose schema:
schema.index({ status: 1, createdAt: -1 });
```

3. **ESR rule** for compound indexes: Equality, Sort, Range. Put equality fields first, then sort fields, then range fields.

## N+1 query pattern

```js
const users = await User.find();
for (const u of users) {
  u.posts = await Post.find({ userId: u._id });  // 💥 N queries
}
```

Fix with `$lookup` aggregation or a single `Post.find({ userId: { $in: ids } })`:
```js
const userIds = users.map(u => u._id);
const posts = await Post.find({ userId: { $in: userIds } });
const byUser = groupBy(posts, p => p.userId.toString());
users.forEach(u => u.posts = byUser[u._id.toString()] ?? []);
```

## Connection pool exhausted / slow under load

- Mongoose default pool is 5–10. Bump for high-concurrency:
```js
mongoose.connect(uri, { maxPoolSize: 50 });
```
- Long-running queries hold a connection — add `maxTimeMS` to bound them.
- Unclosed cursors: when iterating large sets, `await cursor.close()` when done.

## Write conflict / "WriteConflict" under load

Two transactions tried to modify the same doc. Mongo retries up to a limit, then throws.

Fix:
- Reduce contention (smaller batches, narrower updates).
- Use optimistic concurrency: `updateOne({ _id, version: X }, { $set: { ..., version: X+1 }})`.
- For counters, use `$inc` instead of read-modify-write.

## ValidationError on save

Mongoose schema validation failed. The error message lists which fields. Common causes:
- Required field missing.
- Enum value not in allowed list.
- Custom validator returned false.

If you're inserting raw data (e.g., from a migration), bypass validation deliberately:
```js
await Model.collection.insertOne(rawDoc);  // skips Mongoose layer
```

## "MongoServerError: E11000 duplicate key"

Unique index violation. The doc you're inserting has a value that already exists for an indexed field. Either:
- Check existence first (`findOne` then `create`).
- Use `findOneAndUpdate({...}, { $setOnInsert: {...} }, { upsert: true })` for "create if not exists".

## `ObjectId` comparison returns false

```js
user._id === post.userId        // false even if same ID
user._id.equals(post.userId)    // ✅
user._id.toString() === post.userId.toString()   // ✅
```

## Aggregation pipeline returns wrong shape

- `$lookup` always returns an array (even for single match). Use `$unwind` after.
- `$project` excluded a field you still wanted — explicit project lists all keys.
- `$match` after `$lookup` is slow; do `$match` first to reduce dataset.

## Mongoose populate returns null / empty

- The referenced doc doesn't exist (orphan reference).
- The `ref` in the schema doesn't match the registered model name (case-sensitive).
- The `path` you're populating isn't an `ObjectId` field — populate only works on ref fields.

## Backup / restore

```bash
mongodump --uri="mongodb://localhost:27017/mydb" --out=./backup
mongorestore --uri="mongodb://localhost:27017/mydb_restored" ./backup/mydb
```

Always test restore in a non-prod environment before relying on the backup.

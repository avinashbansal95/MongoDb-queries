# MongoDB: Sorting, Pagination, and Query Performance

🚀 **Awesome!** You've nailed array queries — one of the trickiest but most powerful parts of MongoDB.

Now, let's move to the next key topic:

**➡️ Intermediate → Advanced: Sorting, Pagination, and Query Performance**

This is essential for real-world apps (e.g., dashboards, search results, APIs).

## What We'll Cover

- `.sort()` with single & multiple fields
- `.skip()` and `.limit()` for pagination
- Performance pitfalls
- How to make them fast with indexes
- Best practices for scalable apps

📁 **File:** `queries/intermediate/03_sort_limit.js`

Let's use your existing `learnDB.courses` collection.

## 1. Sort by a Single Field

```javascript
// Sort courses by price (ascending)
db.courses.find().sort({ price: 1 });

// Sort by price (descending)
db.courses.find().sort({ price: -1 });

// Sort by title (alphabetical)
db.courses.find().sort({ title: 1 });
```

**Key:**
- 🔼 `1` = Ascending (A→Z, low→high)
- 🔽 `-1` = Descending (Z→A, high→low)

## 2. Sort by Multiple Fields

```javascript
// Sort by category (asc), then price (desc)
db.courses.find().sort({
  category: 1,
  price: -1
});
```

**👉 Result:**
- All "Database" courses together, then "DevOps", etc.
- Within each category, higher-priced courses first

## 3. Pagination: skip() and limit()

```javascript
// Get first 2 courses
db.courses.find().limit(2);

// Skip first 2, get next 2 (page 2, if page size = 2)
db.courses.find().skip(2).limit(2);

// Page 3: skip 4, limit 2
db.courses.find().skip(4).limit(2);
```

**📌 Formula:**
```javascript
skip( (page - 1) * limit )
```
*Example: Page 3, 2 items per page → skip(4)*

## 4. Combine Sort + Pagination (Most Common Pattern)

```javascript
// Get top 2 most expensive courses
db.courses.find()
  .sort({ price: -1 })
  .limit(2);

// Page 2 of courses sorted by price (descending)
db.courses.find()
  .sort({ price: -1 })
  .skip(2)
  .limit(2);
```

✅ This is how APIs like `/courses?sort=price&order=desc&page=2&limit=2` are implemented.

## 5. Performance Problem: Deep Pagination

```javascript
// ❌ SLOW on large datasets
db.courses.find()
  .sort({ price: -1 })
  .skip(1000)
  .limit(10);
```

**🔻 Why slow?**
- MongoDB must scan the first 1000 matched documents
- Even if you only want 10
- Gets worse as page number increases

## 6. Solution: Use Keyset Pagination (aka Seek Method)

Instead of `skip()`, use a condition based on the last seen value.

**Step 1: Get first page**
```javascript
// First page: top 5 by price
db.courses.find()
  .sort({ price: -1 })
  .limit(5);
```
*Suppose the last course on this page has price = 95.*

**Step 2: Next page (get next 5)**
```javascript
// Get next 5 courses with price < 95
db.courses.find({ price: { $lt: 95 } })
  .sort({ price: -1 })
  .limit(5);
```

**Benefits:**
- ✅ No `skip()` → much faster
- ✅ Works like "Load More" in feeds
- ✅ Used by Twitter, Facebook, etc.

**🔑 Requirement:** Sorting field (price) must be unique or combined with `_id`

If not unique:
```javascript
db.courses.find({
  $or: [
    { price: { $lt: 95 } },
    { price: 95, _id: { $lt: ObjectId("...") } }
  ]
})
.sort({ price: -1, _id: -1 })
.limit(5);
```

## 7. Count Total Matching Documents

```javascript
// How many active courses?
db.courses.count({ isActive: true });

// Or use modern syntax
db.courses.countDocuments({ isActive: true });

// For exact count (slower on large collections)
db.courses.estimatedDocumentCount();
```

**📌 Use:**
- `countDocuments()` → accurate, supports query
- `estimatedDocumentCount()` → fast, no query
- Avoid deprecated `.count()` without options

## 8. Sort with Embedded Fields (Using Dot Notation)

```javascript
// Sort by first enrolled student's progress
db.courses.find().sort({ "studentsEnrolled.0.progress": -1 });

// Sort by number of reviews (trick: use aggregation for this)
// But we can sort by array size only if we store it or use aggregation
```

⚠️ **Note:** You cannot directly sort by "reviews.length" unless you store it as a field.

## 9. Performance: Use Indexes!

Without indexes, `sort()`, `skip()`, `limit()` can be very slow.

**Create Index for Sort + Pagination**
```javascript
// Speed up sorting by price
db.courses.createIndex({ price: -1 });

// For multi-field sort
db.courses.createIndex({ category: 1, price: -1 });

// For keyset pagination
db.courses.createIndex({ price: -1, _id: -1 });
```

**Results:**
- ✅ With index: Pagination is fast, even at page 1000
- ❌ Without: Deep `skip()` becomes slow

## 🧪 Practice Queries

Add these to `03_sort_limit.js`:

```javascript
// 1. Find all active courses, sorted by price (high to low)
db.courses.find({ isActive: true }).sort({ price: -1 });

// 2. Get the 2 cheapest courses
db.courses.find().sort({ price: 1 }).limit(2);

// 3. Get courses in "Database" or "DevOps", sorted by title
db.courses.find({
  category: { $in: ["Database", "DevOps"] }
})
.sort({ title: 1 });

// 4. Paginate: Page 2 (limit 2)
db.courses.find()
.sort({ price: -1 })
.skip(2)
.limit(2);

// 5. Keyset pagination: After price=95, get next 2
db.courses.find({ price: { $lt: 95 } })
.sort({ price: -1 })
.limit(2);

// 6. Count how many courses have at least one student
db.courses.countDocuments({ "studentsEnrolled.0": { $exists: true } });
```

## ✅ Summary: Key Takeaways

| Concept | Usage | Notes |
|---------|--------|-------|
| `.sort({ field: 1 })` | Always use with index | 1=asc, -1=desc |
| `.skip(n).limit(m)` | OK for small offsets | Avoid deep pagination |
| **Keyset Pagination** | Use `$lt`, `$gt` on sort field + `_id` | Much faster for large datasets |
| `countDocuments()` | Use for accurate counts with filters | Preferred over deprecated `.count()` |
| **Indexes** | Essential for performance | Create compound indexes for multi-field sorts |

---

**Next Topic:** Advanced Aggregation Pipeline 🚀
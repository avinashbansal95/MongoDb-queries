# MongoDB: Unique Sorting Fields and Keyset Pagination Explained

🔑 **"The sorting field must be unique, or combined with _id"** — when using keyset pagination (also called seek method).

This is a critical concept for efficient and correct pagination in MongoDB (and databases in general).

## What We'll Cover

- What "unique" means
- Why non-unique sort fields break pagination
- How combining with `_id` fixes it
- Real examples with detailed explanations

## 🎯 Goal: Keyset Pagination (Seek Method)

Instead of:
```javascript
.skip(1000).limit(10)  // ❌ Slow
```

We use:
```javascript
.find({ price: { $lt: lastSeenPrice } })  // ✅ Fast
.limit(10)
```

But this only works correctly if we can uniquely order the results.

## 🔍 What Does "Unique" Mean?

A field is **unique** if no two documents have the same value for that field.

### Example: Unique Field
```json
{ "_id": 1, "orderId": "A100" }
{ "_id": 2, "orderId": "A101" }
```
✅ `orderId` is unique — each value appears only once.

### Example: Non-Unique Field
```json
{ "_id": 1, "price": 95 }
{ "_id": 2, "price": 95 }
```
❌ `price` is not unique — two courses have `price: 95`

## 🚨 Problem: Sorting by Non-Unique Field

Let's say you have these courses (sorted by `price: -1`):

| ID | Course | Price |
|----|--------|-------|
| C1 | React Course | 120 |
| C2 | MongoDB Course | 95 |
| C3 | DevOps Course | 95 |
| C4 | Algorithms | 95 |
| C5 | UI Design | 80 |

### Step 1: Get First Page
```javascript
db.courses.find().sort({ price: -1 }).limit(2)
```

**👉 Returns:**
- C1 (120)
- C2 (95)

You note: `last price seen = 95`

### Step 2: Get Next Page
```javascript
db.courses.find({ price: { $lt: 95 } }).sort({ price: -1 }).limit(2)
```

**👉 Returns:**
- C5 (80)

**But wait — C3 and C4 (both price=95) are missing!**

### ❌ Why?

Because `$lt: 95` means **strictly less than 95**, so all `price=95` docs are excluded — even though they weren't fully consumed.

👉 **You skipped C3 and C4 — data loss!**

## ✅ Solution 1: Make Sort Field Unique

If `price` were unique, this wouldn't happen.

But in real life, prices are rarely unique — many products have the same price.

So we need another way.

## ✅ Solution 2: Combine Sort Field with `_id`

Since `_id` is always unique, we can use it to break ties.

### Step 1: Sort by price, then by _id

```javascript
db.courses.find().sort({ price: -1, _id: 1 })
```

Now the order is **deterministic**:

| ID | Course | Price | Sort Order |
|----|--------|-------|------------|
| C1 | React | 120 | (120, C1) → first |
| C2 | Mongo | 95 | (95, C2) → second |
| C3 | DevOps | 95 | (95, C3) → third |
| C4 | Algo | 95 | (95, C4) → fourth |
| C5 | UI | 80 | (80, C5) → fifth |

Even though price is the same for C2, C3, C4 — the `_id` ensures a **fixed, predictable order**.

### Step 2: Keyset Pagination with Compound Condition

After fetching up to C2 (`price=95, _id=C2`), get the next page:

```javascript
db.courses.find({
  $or: [
    { price: { $lt: 95 } },           // Any course cheaper
    { price: 95, _id: { $gt: "C2" } } // Or same price but after C2
  ]
})
.sort({ price: -1, _id: 1 })
.limit(2)
```

**👉 Returns:**
- C3 (price=95, _id > C2)
- C4

### Next page:
```javascript
{ price: { $lt: 95 } OR (price=95 AND _id > "C4") }
```

**👉 Returns:** C5

### Results:
- ✅ No duplicates
- ✅ No skipped documents
- ✅ Fast (uses index)
- ✅ Works at scale

## 🧠 Why This Works

The compound condition has two parts:

1. **`{ price: { $lt: 95 } }`** - Gets all cheaper courses
2. **`{ price: 95, _id: { $gt: "C2" } }`** - Gets remaining courses at same price, but **after** the last seen one

Together, they mean:
> **"Get everything that comes after (95, C2) in the sorted order"**

## 🏗️ Index for Performance

Create a compound index:

```javascript
db.courses.createIndex({ price: -1, _id: 1 })
```

Now the query:
- ✅ Uses the index
- ✅ Avoids sorting in memory
- ✅ Skips don't scan irrelevant docs
- ✅ Scales to millions of records

## 🆚 Summary: Unique vs Non-Unique Sort Field

| Field Type | Examples | Works for Keyset? | Reason |
|------------|----------|-------------------|--------|
| **✅ Unique** | `orderId`, `_id` | ✅ Yes | Every value is different — no ambiguity |
| **❌ Non-Unique** | `price`, `category` | ❌ No (alone) | Multiple docs have same value — can skip or repeat |
| **✅ Non-Unique + `_id`** | `{ price: -1, _id: 1 }` | ✅ Yes | `_id` breaks ties, ensures total order |

## 📌 Real-World Example

Imagine Twitter feed sorted by likes:

- Many tweets have `likes: 1000`
- If you paginate with only `likes < 1000`, you might skip some 1000-like tweets
- But if you sort by `(likes, _id)`, you can safely resume from the last seen `_id`

## ✅ Final Answer

**🔹 "Unique" means:** no two documents have the same value for that field.

**🔹 When a sort field (like price) is not unique,** sorting alone doesn't give a deterministic order — MongoDB may return same-priced docs in different order across queries.

**🔹 To fix this, combine the sort field with `_id`:**

```javascript
.sort({ price: -1, _id: 1 })
```

and use:

```javascript
$or: [
  { price: { $lt: lastPrice } },
  { price: lastPrice, _id: { $gt: lastId } }
]
```

This ensures:
- ✅ No skipped documents
- ✅ No duplicates
- ✅ Fast, index-backed pagination
- ✅ Scalable to large datasets

---

**🚀 You're now thinking like a backend engineer building real-world APIs!**

Ready to move to Aggregation Pipeline? We'll start with `$match`, `$group`, and build up to `$lookup` and analytics.

Just say: **"Let's do aggregation!"** and I'll set up the dataset and queries.
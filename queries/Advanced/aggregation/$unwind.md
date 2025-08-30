# $unwind Stage Solutions

## Question 15: Unwind product tags and count frequency of each tag

```javascript
db.products.aggregate([
  {
    $unwind: "$tags"
  },
  {
    $group: {
      _id: "$tags",
      frequency: { $sum: 1 }
    }
  },
  {
    $sort: {
      frequency: -1
    }
  }
])
```

## Question 16: Unwind tags and find which category has the most "computers" tagged products

```javascript
db.products.aggregate([
  {
    $unwind: "$tags"
  },
  {
    $match: {
      tags: "computers"
    }
  },
  {
    $group: {
      _id: "$category",
      productCount: { $sum: 1 }
    }
  },
  {
    $sort: {
      productCount: -1
    }
  },
  {
    $limit: 1
  }
])
```

## Question 17: Create a report showing each tag with associated product count and average price

```javascript
db.products.aggregate([
  {
    $unwind: "$tags"
  },
  {
    $group: {
      _id: "$tags",
      productCount: { $sum: 1 },
      averagePrice: { $avg: "$price" },
      totalProducts: { $addToSet: "$name" }  // Optional: see which products
    }
  },
  {
    $project: {
      tag: "$_id",
      productCount: 1,
      averagePrice: { $round: ["$averagePrice", 2] },
      totalProducts: 1,
      _id: 0
    }
  },
  {
    $sort: {
      productCount: -1
    }
  }
])
```

## Expected Results

### Question 15: Tag frequency:
- "computers": 2 times (Laptop, Tablet)
- "work": 1 time
- "mobile": 1 time
- "communication": 1 time
- "kitchen": 1 time
- "coffee": 1 time
- etc.

### Question 16: Category with most "computers" tags:
- "Electronics": 2 products (Laptop, Tablet)

### Question 17: Tag report:
- "computers": 2 products, avg price $699
- "work": 1 product, avg price $999
- "mobile": 1 product, avg price $699
- etc.

## How $unwind Works

### Before $unwind:

```javascript
{ _id: 1, name: "Laptop", tags: ["computers", "work"] }
{ _id: 2, name: "Smartphone", tags: ["mobile", "communication"] }
```

### After $unwind: "$tags":

```javascript
{ _id: 1, name: "Laptop", tags: "computers" }
{ _id: 1, name: "Laptop", tags: "work" }
{ _id: 2, name: "Smartphone", tags: "mobile" }
{ _id: 2, name: "Smartphone", tags: "communication" }
```

## Key Points

- **Duplicates documents** for each array element
- **Flattens arrays** into separate documents
- **Preserves other fields** from original document
- **Essential** for analyzing array data

## $unwind Options

```javascript
// Basic unwind
{ $unwind: "$tags" }

// With options (preserve empty/null arrays)
{ 
  $unwind: {
    path: "$tags",
    preserveNullAndEmptyArrays: true
  }
}

// With index (adds array position)
{
  $unwind: {
    path: "$tags", 
    includeArrayIndex: "tagIndex"
  }
}
```
# MongoDB Advanced Aggregation Stages

## $bucket Stage Solutions

### Question 26: Group products into price buckets: $0-$100, $100-$500, $500+

```javascript
db.products.aggregate([
  {
    $bucket: {
      groupBy: "$price",
      boundaries: [0, 100, 500, 1000],
      default: "1000+",
      output: {
        count: { $sum: 1 },
        products: { $push: "$name" },
        averagePrice: { $avg: "$price" },
        totalStock: { $sum: "$stock" },
        minPrice: { $min: "$price" },
        maxPrice: { $max: "$price" }
      }
    }
  },
  {
    $addFields: {
      priceRange: {
        min: "$minPrice",
        max: "$maxPrice"
      }
    }
  },
  {
    $project: {
      count: 1,
      products: 1,
      averagePrice: 1,
      totalStock: 1,
      priceRange: 1
      // minPrice and maxPrice are excluded
    }
  }
])
```

## $bucketAuto Stage Solutions

### Question 27: Auto-bucket customers by totalSpent into 3 buckets

```javascript
db.customers.aggregate([
  {
    $bucketAuto: {
      groupBy: "$totalSpent",
      buckets: 3,
      output: {
        count: { $sum: 1 },
        customers: { $push: "$name" },
        averageSpent: { $avg: "$totalSpent" },
        ageRange: {
          min: { $min: "$age" },
          max: { $max: "$age" }
        },
        spendingRange: {
          min: { $min: "$totalSpent" },
          max: { $max: "$totalSpent" }
        }
      }
    }
  }
])
```

## $sample Stage Solutions

### Question 28: Get 3 random products from the database

```javascript
db.products.aggregate([
  {
    $sample: {
      size: 3
    }
  }
])
```

### More practical example - random products with specific fields:

```javascript
db.products.aggregate([
  {
    $sample: {
      size: 3
    }
  },
  {
    $project: {
      name: 1,
      category: 1,
      price: 1,
      rating: 1
    }
  }
])
```

## $replaceRoot Stage Solutions

### Question 29: Flatten nested shipping address fields in orders

```javascript
db.orders.aggregate([
  {
    $replaceRoot: {
      newRoot: {
        $mergeObjects: [
          "$$ROOT",
          {
            city: "$shippingAddress.city",
            state: "$shippingAddress.state",
            zipCode: "$shippingAddress.zipCode"
          }
        ]
      }
    }
  },
  {
    $project: {
      shippingAddress: 0  // Remove original nested field
    }
  }
])
```

### Alternative approach using $addFields + $unset:

```javascript
db.orders.aggregate([
  {
    $addFields: {
      city: "$shippingAddress.city",
      state: "$shippingAddress.state", 
      zipCode: "$shippingAddress.zipCode"
    }
  },
  {
    $unset: "shippingAddress"
  }
])
```

## Key Learning Points

### $bucket vs $bucketAuto

#### $bucket:
- **Manual boundaries**: You define exact boundary values
- **Predictable ranges**: Know exactly what ranges you'll get
- **Custom default**: Handle outliers with a default bucket
- **Use when**: You have specific business requirements for ranges

#### $bucketAuto:
- **Automatic boundaries**: MongoDB determines optimal boundaries
- **Even distribution**: Tries to distribute documents evenly
- **Specify count**: You just say how many buckets you want
- **Use when**: You want even distribution without caring about exact ranges

### $sample Use Cases:
- **Testing**: Get random sample data for development
- **Analytics**: Statistical sampling for large datasets
- **Features**: "Random products", "Discover new items"
- **Performance**: Sample large collections for quick analysis

### $replaceRoot Applications:
- **Flattening**: Move nested fields to root level
- **Restructuring**: Completely change document structure
- **Promoting**: Elevate embedded documents to main document
- **Data transformation**: Reshape documents for different uses

## Comparison Examples

### $bucket Example Output:
```javascript
[
  {
    "_id": 0,        // $0-$100 range
    "count": 1,
    "products": ["Book"],
    "averagePrice": 29
  },
  {
    "_id": 100,      // $100-$500 range
    "count": 4,
    "products": ["Headphones", "Coffee Maker", "Running Shoes", "Tablet"],
    "averagePrice": 218.75
  },
  {
    "_id": 500,      // $500-$1000 range
    "count": 2,
    "products": ["Smartphone", "Laptop"],
    "averagePrice": 849
  }
]
```

### $bucketAuto Example Output:
```javascript
[
  {
    "_id": { "min": 87, "max": 528 },    // Low spenders
    "count": 2,
    "customers": ["Alice Brown", "Bob Johnson"]
  },
  {
    "_id": { "min": 1198, "max": 1647 }, // Medium spenders  
    "count": 2,
    "customers": ["John Doe", "Jane Smith"]
  },
  {
    "_id": { "min": 1998, "max": 1998 }, // High spenders
    "count": 1,
    "customers": ["Charlie Wilson"]
  }
]
```

## Performance Tips

- **$bucket**: More efficient than manual grouping with conditions
- **$bucketAuto**: Good for unknown data distributions
- **$sample**: Very efficient for large collections
- **$replaceRoot**: Use when you need significant document restructuring

## Real-World Applications

### Business Analytics:
- Price tier analysis
- Customer segmentation
- Performance bucketing
- Market research sampling

### Data Processing:
- Document flattening for APIs
- Structure normalization
- Random data selection
- ETL transformations
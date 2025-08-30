# MongoDB Complex Aggregation Operations

## Array Operations

### Question 30: Find products that have both "computers" and "work" tags

```javascript
db.products.aggregate([
  {
    $match: {
      tags: { $all: ["computers", "work"] }
    }
  }
])
```

#### Alternative using aggregation expressions:

```javascript
db.products.aggregate([
  {
    $match: {
      $expr: {
        $and: [
          { $in: ["computers", "$tags"] },
          { $in: ["work", "$tags"] }
        ]
      }
    }
  }
])
```

### Question 31: Calculate average helpful votes for products with multiple reviews

```javascript
db.reviews.aggregate([
  {
    $group: {
      _id: "$productId",
      reviewCount: { $sum: 1 },
      averageHelpful: { $avg: "$helpful" },
      totalHelpful: { $sum: "$helpful" }
    }
  },
  {
    $match: {
      reviewCount: { $gt: 1 }  // Products with multiple reviews
    }
  },
  {
    $lookup: {
      from: "products",
      localField: "_id",
      foreignField: "_id",
      as: "product"
    }
  },
  {
    $unwind: "$product"
  },
  {
    $project: {
      productName: "$product.name",
      reviewCount: 1,
      averageHelpful: { $round: ["$averageHelpful", 2] },
      totalHelpful: 1
    }
  },
  {
    $sort: { averageHelpful: -1 }
  }
])
```

## Conditional Operations

### Question 32: Categorize orders as "high-value" (>$500) or "low-value" (≤$500)

```javascript
db.orders.aggregate([
  {
    $addFields: {
      valueCategory: {
        $cond: {
          if: { $gt: ["$totalAmount", 500] },
          then: "high-value",
          else: "low-value"
        }
      }
    }
  }
])
```

#### More detailed categorization:

```javascript
db.orders.aggregate([
  {
    $addFields: {
      valueCategory: {
        $switch: {
          branches: [
            { case: { $lte: ["$totalAmount", 200] }, then: "low-value" },
            { case: { $lte: ["$totalAmount", 500] }, then: "medium-value" },
            { case: { $lte: ["$totalAmount", 1000] }, then: "high-value" }
          ],
          default: "premium-value"
        }
      }
    }
  },
  {
    $group: {
      _id: "$valueCategory",
      count: { $sum: 1 },
      averageAmount: { $avg: "$totalAmount" },
      totalRevenue: { $sum: "$totalAmount" }
    }
  }
])
```

### Question 33: Create a customer status based on multiple conditions (age, totalSpent, loyaltyPoints)

```javascript
db.customers.aggregate([
  {
    $addFields: {
      customerStatus: {
        $switch: {
          branches: [
            {
              case: {
                $and: [
                  { $gte: ["$totalSpent", 1500] },
                  { $gte: ["$loyaltyPoints", 200] },
                  { $gte: ["$age", 25] }
                ]
              },
              then: "VIP"
            },
            {
              case: {
                $and: [
                  { $gte: ["$totalSpent", 500] },
                  { $gte: ["$loyaltyPoints", 100] }
                ]
              },
              then: "Premium"
            },
            {
              case: {
                $or: [
                  { $lt: ["$age", 25] },
                  { $lt: ["$totalSpent", 200] }
                ]
              },
              then: "New"
            }
          ],
          default: "Standard"
        }
      },
      ageGroup: {
        $switch: {
          branches: [
            { case: { $lt: ["$age", 30] }, then: "Young" },
            { case: { $lte: ["$age", 50] }, then: "Middle" }
          ],
          default: "Senior"
        }
      }
    }
  },
  {
    $group: {
      _id: {
        status: "$customerStatus",
        ageGroup: "$ageGroup"
      },
      count: { $sum: 1 },
      averageSpent: { $avg: "$totalSpent" },
      averageLoyaltyPoints: { $avg: "$loyaltyPoints" }
    }
  }
])
```

## Date Operations

### Question 34: Find orders placed in the last 30 days from June 15, 2023

```javascript
db.orders.aggregate([
  {
    $addFields: {
      referenceDate: new Date("2023-06-15")
    }
  },
  {
    $addFields: {
      daysDifference: {
        $divide: [
          { $subtract: ["$referenceDate", "$orderDate"] },
          1000 * 60 * 60 * 24  // Convert milliseconds to days
        ]
      }
    }
  },
  {
    $match: {
      daysDifference: { $lte: 30, $gte: 0 }
    }
  },
  {
    $project: {
      _id: 1,
      customerId: 1,
      totalAmount: 1,
      orderDate: 1,
      daysDifference: { $round: ["$daysDifference", 0] }
    }
  }
])
```

#### Alternative using $dateSubtract (MongoDB 5.0+):

```javascript
db.orders.aggregate([
  {
    $match: {
      orderDate: {
        $gte: {
          $dateSubtract: {
            startDate: new Date("2023-06-15"),
            unit: "day",
            amount: 30
          }
        },
        $lte: new Date("2023-06-15")
      }
    }
  }
])
```

### Question 35: Calculate days between order date and review date for each review

```javascript
db.reviews.aggregate([
  {
    $lookup: {
      from: "orders",
      let: { prodId: "$productId", custId: "$customerId" },
      pipeline: [
        {
          $match: {
            $expr: {
              $and: [
                { $eq: ["$productId", "$$prodId"] },
                { $eq: ["$customerId", "$$custId"] }
              ]
            }
          }
        }
      ],
      as: "order"
    }
  },
  {
    $unwind: "$order"
  },
  {
    $addFields: {
      daysBetween: {
        $divide: [
          { $subtract: ["$reviewDate", "$order.orderDate"] },
          1000 * 60 * 60 * 24
        ]
      }
    }
  },
  {
    $lookup: {
      from: "products",
      localField: "productId",
      foreignField: "_id",
      as: "product"
    }
  },
  {
    $unwind: "$product"
  },
  {
    $project: {
      customerReview: {
        customerId: "$customerId",
        productName: "$product.name",
        rating: "$rating",
        comment: "$comment",
        orderDate: "$order.orderDate",
        reviewDate: "$reviewDate",
        daysToReview: { $round: ["$daysBetween", 0] }
      }
    }
  },
  {
    $sort: { "customerReview.daysToReview": 1 }
  }
])
```

## Key Learning Points

### Array Query Operators

#### $all vs $in:
- `$all`: Document must contain ALL specified values
- `$in`: Document must contain ANY of the specified values

```javascript
// Must have BOTH "computers" AND "work" tags
{ tags: { $all: ["computers", "work"] } }

// Must have EITHER "computers" OR "work" tags  
{ tags: { $in: ["computers", "work"] } }
```

### Complex Conditional Logic

#### Multi-level Switch Statements:
```javascript
$switch: {
  branches: [
    { case: { $and: [condition1, condition2] }, then: "result1" },
    { case: { $or: [condition3, condition4] }, then: "result2" }
  ],
  default: "default_result"
}
```

#### Nested Conditions:
```javascript
$cond: {
  if: { condition1 },
  then: {
    $cond: {
      if: { condition2 },
      then: "nested_result",
      else: "alternative"
    }
  },
  else: "default"
}
```

### Date Calculations

#### Manual Date Arithmetic:
```javascript
// Days difference
$divide: [
  { $subtract: [date1, date2] },
  1000 * 60 * 60 * 24  // ms to days conversion
]

// Hours difference
$divide: [
  { $subtract: [date1, date2] },
  1000 * 60 * 60  // ms to hours conversion
]
```

#### MongoDB 5.0+ Date Functions:
- `$dateAdd`: Add time to a date
- `$dateSubtract`: Subtract time from a date
- `$dateDiff`: Calculate difference between dates

## Performance Considerations

### Array Operations:
- Index array fields for better `$all` and `$in` performance
- Consider array size impact on document storage

### Complex Conditions:
- Simple conditions perform better than nested logic
- Consider pre-calculating complex fields when possible

### Date Operations:
- Index date fields for range queries
- Use appropriate date types (Date vs String)
- Consider time zone implications

## Real-World Applications

### Business Intelligence:
- Customer segmentation based on multiple criteria
- Product performance analysis with complex conditions
- Time-based analytics and reporting

### E-commerce:
- Advanced product filtering
- Customer lifecycle analysis
- Order pattern identification

### Analytics:
- Multi-dimensional data analysis
- Cohort analysis with date calculations
- Performance metrics with conditional logic
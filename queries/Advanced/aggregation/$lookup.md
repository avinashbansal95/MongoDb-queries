# $lookup Stage Solutions

## Question 21: Join orders with customer information

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customerInfo"
    }
  },
  {
    $unwind: "$customerInfo"
  }
])
```

## Question 22: Join orders with both customer and product information

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customerInfo"
    }
  },
  {
    $lookup: {
      from: "products",
      localField: "productId",
      foreignField: "_id",
      as: "productInfo"
    }
  },
  {
    $unwind: "$customerInfo"
  },
  {
    $unwind: "$productInfo"
  }
])
```

## Question 23: Get customer details with their order history and total order count

```javascript
db.customers.aggregate([
  {
    $lookup: {
      from: "orders",
      localField: "_id",
      foreignField: "customerId",
      as: "orders"
    }
  },
  {
    $addFields: {
      totalOrders: { $size: "$orders" },
      totalOrderValue: { $sum: "$orders.totalAmount" }
    }
  },
  {
    $project: {
      name: 1,
      email: 1,
      city: 1,
      totalSpent: 1,
      totalOrders: 1,
      totalOrderValue: 1,
      orders: {
        $map: {
          input: "$orders",
          as: "order",
          in: {
            orderId: "$$order._id",
            productId: "$$order.productId",
            quantity: "$$order.quantity",
            amount: "$$order.totalAmount",
            orderDate: "$$order.orderDate",
            status: "$$order.status"
          }
        }
      }
    }
  }
])
```

## Question 24: Create a complete order report with customer, product, and review information

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customer"
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
    $lookup: {
      from: "reviews",
      localField: "productId",
      foreignField: "productId",
      as: "reviews"
    }
  },
  {
    $unwind: "$customer"
  },
  {
    $unwind: "$product"
  },
  {
    $addFields: {
      reviewCount: { $size: "$reviews" },
      averageRating: { $avg: "$reviews.rating" },
      customerReviews: {
        $filter: {
          input: "$reviews",
          as: "review",
          cond: { $eq: ["$$review.customerId", "$customerId"] }
        }
      }
    }
  },
  {
    $project: {
      orderId: "$_id",
      orderDate: 1,
      quantity: 1,
      totalAmount: 1,
      status: 1,
      customer: {
        name: "$customer.name",
        email: "$customer.email",
        city: "$customer.city"
      },
      product: {
        name: "$product.name",
        category: "$product.category",
        brand: "$product.brand",
        price: "$product.price"
      },
      productReviews: {
        totalReviews: "$reviewCount",
        averageRating: { $round: ["$averageRating", 2] }
      },
      customerReviewForThisProduct: { $arrayElemAt: ["$customerReviews", 0] }
    }
  }
])
```

## Expected Results

### Question 21: 
Orders with customer names, emails, cities, etc.

### Question 22: 
Complete order details with customer names and product information

### Question 23: 
Each customer with their order history array and counts:

```javascript
{
  name: "John Doe",
  email: "john@email.com", 
  totalOrders: 2,
  totalOrderValue: 1198,
  orders: [
    { orderId: 101, productId: 1, amount: 999, ... },
    { orderId: 103, productId: 3, amount: 199, ... }
  ]
}
```

### Question 24: 
Complete business report with all related information

## Key Learning Points

### $lookup Syntax:

```javascript
{
  $lookup: {
    from: "collectionToJoin",           // Target collection
    localField: "fieldInCurrentDoc",    // Field from current collection
    foreignField: "fieldInTargetDoc",   // Field from target collection  
    as: "resultArrayName"               // Name for joined data array
  }
}
```

### $lookup Patterns:

#### 1. Basic Join (1:1 relationship):
```javascript
{ $lookup: { from: "users", localField: "userId", foreignField: "_id", as: "user" } },
{ $unwind: "$user" }  // Convert array to object
```

#### 2. Multiple Joins:
```javascript
{ $lookup: { ... } },  // First join
{ $lookup: { ... } },  // Second join  
{ $unwind: "$table1" },
{ $unwind: "$table2" }
```

#### 3. One-to-Many (keep array):
```javascript
{ $lookup: { from: "orders", localField: "_id", foreignField: "customerId", as: "orders" } }
// Don't unwind - keep orders as array
```

## Advanced $lookup Features

### Pipeline-based $lookup (MongoDB 3.6+):

```javascript
{
  $lookup: {
    from: "orders",
    let: { customerId: "$_id" },
    pipeline: [
      { $match: { $expr: { $eq: ["$customerId", "$$customerId"] } } },
      { $match: { status: "completed" } },  // Additional filtering
      { $sort: { orderDate: -1 } }
    ],
    as: "completedOrders"
  }
}
```

## Common $lookup Issues

### 1. Data Types Must Match:
```javascript
// This WON'T work if one is string and other is number
localField: "customerId",    // "1001" (string)
foreignField: "_id"          // 1001 (number)
```

### 2. Always Results in Array:
```javascript
// $lookup always creates an array, even for single matches
"customerInfo": [{ name: "John", email: "john@email.com" }]

// Use $unwind to convert to object
"customerInfo": { name: "John", email: "john@email.com" }
```

## Performance Tips

- Index foreign fields for better performance
- `$match` before `$lookup` to reduce documents to join
- Use pipeline in `$lookup` for complex join conditions
- Consider embedding vs joining based on query patterns

## Real-World Applications

- **Order management**: Join orders with customer and product details
- **Analytics**: Combine transaction data with user profiles
- **Reporting**: Create comprehensive business reports
- **APIs**: Denormalize data for frontend consumption


# $group Stage Solutions

## Question 11: Count total number of products in each category

```javascript
db.products.aggregate([
  {
    $group: {
      _id: "$category",
      productCount: { $sum: 1 }
    }
  }
])
```

## Question 12: Find average price and total stock for each brand

```javascript
db.products.aggregate([
  {
    $group: {
      _id: "$brand",
      averagePrice: { $avg: "$price" },
      totalStock: { $sum: "$stock" }
    }
  }
])
```

## Question 13: Calculate total revenue, average order value, and order count by customer city

```javascript
db.orders.aggregate([
  {
    $group: {
      _id: "$shippingAddress.city",
      totalRevenue: { $sum: "$totalAmount" },
      averageOrderValue: { $avg: "$totalAmount" },
      orderCount: { $sum: 1 }
    }
  }
])
```

## Question 14: Group orders by month and find monthly revenue with order count

```javascript
db.orders.aggregate([
  {
    $group: {
      _id: {
        year: { $year: "$orderDate" },
        month: { $month: "$orderDate" }
      },
      monthlyRevenue: { $sum: "$totalAmount" },
      orderCount: { $sum: 1 }
    }
  },
  {
    $sort: {
      "_id.year": 1,
      "_id.month": 1
    }
  }
])
```

### Alternative with cleaner date formatting:

```javascript
db.orders.aggregate([
  {
    $group: {
      _id: { $dateToString: { format: "%Y-%m", date: "$orderDate" } },
      monthlyRevenue: { $sum: "$totalAmount" },
      orderCount: { $sum: 1 }
    }
  },
  {
    $sort: { _id: 1 }
  }
])
```

## Expected Results

### Question 11: Product count by category:
- Electronics: 4 products
- Sports: 1 product
- Appliances: 1 product
- Education: 1 product
- Furniture: 1 product

### Question 12: Brand statistics:
- TechCorp: avg $699, total stock 90
- PhoneCo: avg $699, total stock 100
- BrewMaster: avg $199, total stock 30
- etc.

### Question 13: City-wise order statistics:
- New York: total $1198, avg $599, count 2
- Los Angeles: total $1647, avg $823.5, count 2
- Chicago: total $528, avg $264, count 2
- etc.

### Question 14: Monthly revenue:
- 2023-06: total $5459, count 8 orders

## Key Learning Points for $group

### $group Fundamentals:
- `_id` field defines what to group by (like GROUP BY in SQL)
- `_id: null` groups all documents together
- Field references use `"$fieldName"` syntax

### Accumulator Operators:
- `$sum: 1` - counts documents (like COUNT(*))
- `$sum: "$field"` - sums field values
- `$avg: "$field"` - calculates average
- `$max: "$field"` - finds maximum value
- `$min: "$field"` - finds minimum value
- `$first: "$field"` - first value in group
- `$last: "$field"` - last value in group

### Grouping Techniques:
- Single field: `_id: "$category"`
- Nested field: `_id: "$shippingAddress.city"`
- Multiple fields: `_id: { field1: "$field1", field2: "$field2" }`
- Date extraction: `_id: { $year: "$dateField" }`

## Common Patterns

```javascript
// Count documents
{ $sum: 1 }

// Sum a field
{ $sum: "$amount" }

// Average calculation
{ $avg: "$price" }

// Group by date parts
_id: { 
  year: { $year: "$date" },
  month: { $month: "$date" }
}
```
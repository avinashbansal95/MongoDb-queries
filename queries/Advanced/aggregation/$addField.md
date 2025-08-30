# $addFields Stage Solutions

## Question 18: Add a field indicating if product is "expensive" (price > $500)

```javascript
db.products.aggregate([
  {
    $addFields: {
      isExpensive: { $gt: ["$price", 500] }
    }
  }
])
```

### Alternative with text labels:

```javascript
db.products.aggregate([
  {
    $addFields: {
      priceCategory: {
        $cond: {
          if: { $gt: ["$price", 500] },
          then: "Expensive",
          else: "Affordable"
        }
      }
    }
  }
])
```

## Question 19: Calculate profit margin assuming 30% cost ratio (profit = price * 0.3)

```javascript
db.products.aggregate([
  {
    $addFields: {
      profitMargin: { $multiply: ["$price", 0.3] },
      costPrice: { $multiply: ["$price", 0.7] },
      profitPercentage: 30
    }
  }
])
```

### More detailed version:

```javascript
db.products.aggregate([
  {
    $addFields: {
      costPrice: { $multiply: ["$price", 0.7] },
      profitMargin: { $multiply: ["$price", 0.3] },
      profitPercentage: {
        $multiply: [
          { $divide: [{ $multiply: ["$price", 0.3] }, "$price"] },
          100
        ]
      }
    }
  }
])
```

## Question 20: Add customer tier based on totalSpent (Bronze: <$500, Silver: $500-$1500, Gold: >$1500)

```javascript
db.customers.aggregate([
  {
    $addFields: {
      customerTier: {
        $switch: {
          branches: [
            { 
              case: { $lt: ["$totalSpent", 500] }, 
              then: "Bronze" 
            },
            { 
              case: { 
                $and: [
                  { $gte: ["$totalSpent", 500] }, 
                  { $lte: ["$totalSpent", 1500] }
                ]
              }, 
              then: "Silver" 
            }
          ],
          default: "Gold"
        }
      }
    }
  }
])
```

### Alternative using nested $cond:

```javascript
db.customers.aggregate([
  {
    $addFields: {
      customerTier: {
        $cond: {
          if: { $lt: ["$totalSpent", 500] },
          then: "Bronze",
          else: {
            $cond: {
              if: { $lte: ["$totalSpent", 1500] },
              then: "Silver",
              else: "Gold"
            }
          }
        }
      }
    }
  }
])
```

## Expected Results

### Question 18: Products with isExpensive flag:
- Laptop ($999): isExpensive: true
- Smartphone ($699): isExpensive: true
- Tablet ($399): isExpensive: false
- Headphones ($249): isExpensive: false
- etc.

### Question 19: Products with profit calculations:
- Laptop: profitMargin: $299.7, costPrice: $699.3
- Smartphone: profitMargin: $209.7, costPrice: $489.3
- etc.

### Question 20: Customers with tiers:
- John Doe ($1198): "Silver"
- Jane Smith ($1647): "Gold"
- Bob Johnson ($528): "Silver"
- Alice Brown ($87): "Bronze"
- Charlie Wilson ($1998): "Gold"

## Key Learning Points

### $addFields vs $project:

```javascript
// $addFields - ADDS new fields, KEEPS existing fields
{ $addFields: { newField: "value" } }

// $project - SELECTS specific fields, EXCLUDES others  
{ $project: { name: 1, newField: "value" } }
```

### Conditional Operators:

#### Simple True/False:
```javascript
isExpensive: { $gt: ["$price", 500] }  // Returns boolean
```

#### If-Then-Else:
```javascript
{
  $cond: {
    if: { condition },
    then: "value if true",
    else: "value if false"
  }
}
```

#### Multiple Conditions (Switch):
```javascript
{
  $switch: {
    branches: [
      { case: { condition1 }, then: "result1" },
      { case: { condition2 }, then: "result2" }
    ],
    default: "default result"
  }
}
```

### Mathematical Operations:
- `$multiply: [field, number]` - Multiplication
- `$divide: [field, number]` - Division
- `$add: [field, number]` - Addition
- `$subtract: [field, number]` - Subtraction
- `$round: [field, decimals]` - Rounding

### Comparison Operators in Expressions:
- `$gt: [field, value]` - Greater than
- `$gte: [field, value]` - Greater than or equal
- `$lt: [field, value]` - Less than
- `$lte: [field, value]` - Less than or equal
- `$eq: [field, value]` - Equal to

## Real-World Applications

### Business Logic:
```javascript
// Customer segmentation
customerSegment: { $switch: { ... } }

// Pricing tiers  
priceTier: { $cond: { ... } }

// Performance indicators
performanceRating: { $switch: { ... } }
```

### Calculated Fields:
```javascript
// Financial calculations
totalWithTax: { $multiply: ["$price", 1.08] }

// Derived metrics
profitMargin: { $subtract: ["$revenue", "$cost"] }
```

## Performance Tips

- `$addFields` is more efficient than `$project` when you need most fields
- Use early in pipeline when calculations are needed for later stages
- Complex calculations can impact performance - consider pre-calculating when possible

---

## Congratulations! You've completed Phase 1 (Questions 1-20)! 🎉

You've now mastered the fundamental aggregation stages:

✅ `$match` - Filtering documents  
✅ `$project` - Field selection and transformation  
✅ `$sort` - Ordering results  
✅ `$limit` - Limiting results  
✅ `$skip` - Pagination  
✅ `$group` - Aggregating data  
✅ `$unwind` - Array processing  
✅ `$addFields` - Adding computed fields
# $project Stage Solutions

## Question 4: Show only product name and price for all products

```javascript
db.products.aggregate([
  {
    $project: {
      name: 1,
      price: 1
    }
  }
])
```

## Question 5: Create a new field "discountedPrice" (10% discount) and show name, original price, and discounted price

```javascript
db.products.aggregate([
  {
    $project: {
      name: 1,
      price: 1,
      discountedPrice: { $multiply: ["$price", 0.9] }
    }
  }
])
```

## Question 6: Extract year from order date and create age categories for customers (young: <30, middle: 30-50, senior: >50)

### Customer Age Categories

```javascript
db.customers.aggregate([
  {
    $project: {
      name: 1,
      age: 1,
      ageCategory: {
        $switch: {
          branches: [
            { case: { $lt: ["$age", 30] }, then: "young" },
            { case: { $and: [{ $gte: ["$age", 30] }, { $lte: ["$age", 50] }] }, then: "middle" }
          ],
          default: "senior"
        }
      }
    }
  }
])
```

### Alternative solution using $cond (nested conditionals):

```javascript
db.customers.aggregate([
  {
    $project: {
      name: 1,
      age: 1,
      ageCategory: {
        $cond: {
          if: { $lt: ["$age", 30] },
          then: "young",
          else: {
            $cond: {
              if: { $lte: ["$age", 50] },
              then: "middle",
              else: "senior"
            }
          }
        }
      }
    }
  }
])
```

### For the orders date extraction version:

```javascript
db.orders.aggregate([
  {
    $project: {
      _id: 1,
      customerId: 1,
      orderYear: { $year: "$orderDate" },
      orderMonth: { $month: "$orderDate" },
      orderDay: { $dayOfMonth: "$orderDate" }
    }
  }
])
```

## Key Learning Points

### Question 4:
- `1` means include the field, `0` means exclude
- `_id` is included by default (use `_id: 0` to exclude it)
- Results show only name and price fields

### Question 5:
- `$multiply` operator for mathematical operations
- Field references use `"$fieldName"` syntax
- Creates calculated fields alongside existing ones
- discountedPrice = price × 0.9 (90% of original price)

### Question 6:
- `$switch` is cleaner for multiple conditions (like switch statement)
- `$cond` is for simple if-then-else (can be nested)
- Date extraction functions: `$year`, `$month`, `$dayOfMonth`, `$dayOfWeek`
- Comparison operators work inside expressions: `$lt`, `$gte`, `$and`

## Common $project Operators

### Math:
- `$add`, `$subtract`, `$multiply`, `$divide`, `$mod`

### Comparison:
- `$eq`, `$gt`, `$lt`, `$gte`, `$lte`

### Logical:
- `$and`, `$or`, `$not`

### Conditional:
- `$cond`, `$switch`, `$ifNull`

### Date:
- `$year`, `$month`, `$day`, `$hour`, `$dateToString`

### String:
- `$concat`, `$substr`, `$toLower`, `$toUpper`
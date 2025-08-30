# $sort Stage Solutions

## Question 7: Sort products by price in descending order

```javascript
db.products.aggregate([
  {
    $sort: {
      price: -1
    }
  }
])
```

## Question 8: Sort customers by totalSpent (desc) and then by age (asc)

```javascript
db.customers.aggregate([
  {
    $sort: {
      totalSpent: -1,
      age: 1
    }
  }
])
```

# $limit & $skip Stage Solutions

## Question 9: Get the top 5 most expensive products

```javascript
db.products.aggregate([
  {
    $sort: {
      price: -1
    }
  },
  {
    $limit: 5
  }
])
```

## Question 10: Implement pagination - skip first 3 products and get next 2 products sorted by price

```javascript
db.products.aggregate([
  {
    $sort: {
      price: -1
    }
  },
  {
    $skip: 3
  },
  {
    $limit: 2
  }
])
```

## Expected Results

### Question 7: Products ordered by price (highest to lowest):
1. Laptop - $999
2. Smartphone - $699
3. Tablet - $399
4. Desk Chair - $299
5. Headphones - $249
6. Coffee Maker - $199
7. Running Shoes - $129
8. Book - $29

### Question 8: Customers ordered by totalSpent (desc), then age (asc):
1. Charlie Wilson - $1998, age 52
2. Jane Smith - $1647, age 34
3. John Doe - $1198, age 28
4. Bob Johnson - $528, age 45
5. Alice Brown - $87, age 29

### Question 9:
Top 5 most expensive products (Laptop, Smartphone, Tablet, Desk Chair, Headphones)

### Question 10:
Products 4-5 in price ranking (Desk Chair $299, Headphones $249)

## Key Learning Points

### $sort:
- `-1` = descending order (highest to lowest)
- `1` = ascending order (lowest to highest)
- Multiple sort fields: first field has priority, second field breaks ties
- Order of fields in sort object matters!

### $limit:
- Limits the number of documents returned
- Always comes AFTER $sort to get "top N" results
- Think of it as "LIMIT" in SQL

### $skip:
- Skips the specified number of documents
- Used with $limit for pagination
- Order matters: $sort → $skip → $limit

## Pipeline Order for Pagination

```javascript
// Page 1: skip 0, limit 5
// Page 2: skip 5, limit 5  
// Page 3: skip 10, limit 5
// Formula: skip = (page - 1) * pageSize
```

## Performance Tips

- Always use indexes on sort fields for better performance
- $sort should come early in pipeline when possible
- $match before $sort reduces data to sort
- Avoid sorting large datasets without indexes
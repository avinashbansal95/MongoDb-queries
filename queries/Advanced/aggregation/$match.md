# \$match Stage Solutions

### Question 1: Find all products in the "Electronics" category

``` javascript
db.products.aggregate([
  {
    $match: {
      category: "Electronics"
    }
  }
])
```

### Question 2: Find products with price between \$200 and \$800 and rating above 4.0

``` javascript
db.products.aggregate([
  {
    $match: {
      price: { $gte: 200, $lte: 800 },
      rating: { $gt: 4.0 }
    }
  }
])
```

### Question 3: Find orders placed in June 2023 with status "completed" or "shipped"

``` javascript
db.orders.aggregate([
  {
    $match: {
      orderDate: {
        $gte: new Date("2023-06-01"),
        $lt: new Date("2023-07-01")
      },
      status: { $in: ["completed", "shipped"] }
    }
  }
])
```

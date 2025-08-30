# $facet Stage Solutions

## Question 25: Create multiple analytics views: product stats by category, price ranges, and top brands

```javascript
db.products.aggregate([
  {
    $facet: {
      // Analytics View 1: Product stats by category
      categoryStats: [
        {
          $group: {
            _id: "$category",
            productCount: { $sum: 1 },
            averagePrice: { $avg: "$price" },
            totalStock: { $sum: "$stock" },
            averageRating: { $avg: "$rating" },
            minPrice: { $min: "$price" },
            maxPrice: { $max: "$price" }
          }
        },
        {
          $sort: { productCount: -1 }
        }
      ],
      
      // Analytics View 2: Price range distribution
      priceRanges: [
        {
          $bucket: {
            groupBy: "$price",
            boundaries: [0, 100, 300, 600, 1000],
            default: "1000+",
            output: {
              count: { $sum: 1 },
              products: { $push: "$name" },
              averageRating: { $avg: "$rating" },
              totalStock: { $sum: "$stock" }
            }
          }
        }
      ],
      
      // Analytics View 3: Top brands analysis
      topBrands: [
        {
          $group: {
            _id: "$brand",
            productCount: { $sum: 1 },
            averagePrice: { $avg: "$price" },
            totalInventoryValue: { 
              $sum: { $multiply: ["$price", "$stock"] } 
            },
            averageRating: { $avg: "$rating" },
            categories: { $addToSet: "$category" }
          }
        },
        {
          $addFields: {
            categoryCount: { $size: "$categories" }
          }
        },
        {
          $sort: { totalInventoryValue: -1 }
        },
        {
          $limit: 5
        }
      ],
      
      // Analytics View 4: Overall summary
      overallSummary: [
        {
          $group: {
            _id: null,
            totalProducts: { $sum: 1 },
            averagePrice: { $avg: "$price" },
            totalInventoryValue: { 
              $sum: { $multiply: ["$price", "$stock"] } 
            },
            averageRating: { $avg: "$rating" },
            uniqueCategories: { $addToSet: "$category" },
            uniqueBrands: { $addToSet: "$brand" },
            priceRange: {
              $push: {
                min: { $min: "$price" },
                max: { $max: "$price" }
              }
            }
          }
        },
        {
          $project: {
            _id: 0,
            totalProducts: 1,
            averagePrice: { $round: ["$averagePrice", 2] },
            totalInventoryValue: { $round: ["$totalInventoryValue", 2] },
            averageRating: { $round: ["$averageRating", 2] },
            uniqueCategoriesCount: { $size: "$uniqueCategories" },
            uniqueBrandsCount: { $size: "$uniqueBrands" },
            categories: "$uniqueCategories",
            brands: "$uniqueBrands"
          }
        }
      ]
    }
  }
])
```

## Expected Output Structure

```javascript
{
  "categoryStats": [
    {
      "_id": "Electronics",
      "productCount": 4,
      "averagePrice": 586.75,
      "totalStock": 250,
      "averageRating": 4.35,
      "minPrice": 249,
      "maxPrice": 999
    },
    // ... other categories
  ],
  
  "priceRanges": [
    {
      "_id": 0,           // $0-$100 range
      "count": 1,
      "products": ["Book"],
      "averageRating": 4.3,
      "totalStock": 200
    },
    {
      "_id": 100,         // $100-$300 range  
      "count": 3,
      "products": ["Running Shoes", "Coffee Maker", "Headphones"],
      "averageRating": 4.43,
      "totalStock": 165
    }
    // ... other ranges
  ],
  
  "topBrands": [
    {
      "_id": "TechCorp",
      "productCount": 2,
      "averagePrice": 699,
      "totalInventoryValue": 62910,
      "averageRating": 4.3,
      "categories": ["Electronics"],
      "categoryCount": 1
    }
    // ... other brands
  ],
  
  "overallSummary": [
    {
      "totalProducts": 8,
      "averagePrice": 350.25,
      "totalInventoryValue": 143455,
      "averageRating": 4.35,
      "uniqueCategoriesCount": 5,
      "uniqueBrandsCount": 8,
      "categories": ["Electronics", "Sports", "Appliances", "Education", "Furniture"],
      "brands": ["TechCorp", "PhoneCo", "BrewMaster", ...]
    }
  ]
}
```

## Key Learning Points

### What is $facet?

- **Parallel Processing**: Runs multiple aggregation pipelines on the same input documents
- **Multiple Views**: Creates different analytical perspectives in one query
- **Dashboard Ready**: Perfect for creating comprehensive reports/dashboards

### $facet Syntax:

```javascript
{
  $facet: {
    "pipeline1Name": [ /* aggregation stages */ ],
    "pipeline2Name": [ /* aggregation stages */ ],
    "pipeline3Name": [ /* aggregation stages */ ]
  }
}
```

### $facet Benefits:

#### 1. Performance:
- Processes documents only once
- Reduces database round trips
- Efficient for analytics dashboards

#### 2. Convenience:
- Multiple reports in single query
- Consistent data snapshot
- Simplified application logic

## Real-World $facet Use Cases

### E-commerce Dashboard:
```javascript
{
  $facet: {
    salesByCategory: [...],
    topCustomers: [...], 
    monthlyTrends: [...],
    inventoryStatus: [...],
    performanceMetrics: [...]
  }
}
```

### User Analytics:
```javascript
{
  $facet: {
    userDemographics: [...],
    activityPatterns: [...],
    engagementMetrics: [...],
    conversionFunnels: [...]
  }
}
```

### Financial Reports:
```javascript
{
  $facet: {
    revenueByRegion: [...],
    profitMargins: [...],
    expenseBreakdown: [...],
    growthTrends: [...]
  }
}
```

## $facet Limitations

- Each sub-pipeline is independent (can't reference results from other facets)
- Memory intensive for large datasets
- Results are always arrays (even single documents)

## $facet vs Multiple Queries

### Multiple Queries:
```javascript
// 4 separate database calls
db.products.aggregate([...])  // Category stats
db.products.aggregate([...])  // Price ranges  
db.products.aggregate([...])  // Top brands
db.products.aggregate([...])  // Summary
```

### $facet (Better):
```javascript
// 1 database call with multiple results
db.products.aggregate([
  { $facet: { /* all 4 analyses */ } }
])
```

## Advanced $facet Pattern

```javascript
db.collection.aggregate([
  // Pre-processing stages (apply to all facets)
  { $match: { status: "active" } },
  { $addFields: { /* common calculations */ } },
  
  // Multiple analytical views
  {
    $facet: {
      view1: [...],
      view2: [...],
      view3: [...]
    }
  },
  
  // Post-processing (reshape results if needed)
  {
    $project: {
      dashboard: {
        categories: "$view1",
        trends: "$view2", 
        summary: "$view3"
      }
    }
  }
])
```

## $facet is Perfect For:

- Business Intelligence dashboards
- Analytics APIs
- Comprehensive reports
- A/B testing analysis
- Multi-dimensional data exploration
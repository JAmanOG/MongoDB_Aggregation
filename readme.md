### MongoDB Aggregation and Pipeline: A Comprehensive Guide (Important TOpics)

MongoDB Aggregation is a powerful framework for performing data processing and transformation on your collections. Aggregation operations process data records and return computed results. In MongoDB, aggregations work using the **aggregation pipeline**, where data is passed through multiple stages to achieve the desired result. Each stage of the pipeline applies an operation to the data and passes the result to the next stage.

### **1. What is the Aggregation Pipeline?**

The **aggregation pipeline** is a framework in MongoDB where you can pass documents through several stages, each of which transforms the documents. The stages are processed in sequence, and the output of one stage becomes the input for the next stage.

#### Basic Syntax:
```js
db.collection.aggregate([
   { <stage1> },
   { <stage2> },
   { <stage3> },
   ...
])
```

### **2. Stages in the Aggregation Pipeline**

Each stage in the pipeline performs a specific operation. MongoDB provides several stages to process and transform data. Let’s explore the most commonly used stages.

#### **2.1. $match** - Filtering Documents (similar to a SQL `WHERE` clause)

The `$match` stage filters documents by a condition, only passing the matching documents to the next stage.

**Syntax:**
```js
db.collection.aggregate([
   { $match: { <query> } }
])
```

**Example:**
```js
db.orders.aggregate([
   { $match: { status: "shipped" } }
])
```

In this example, only documents where the `status` is `"shipped"` will be passed to the next stage.

#### **2.2. $group** - Grouping Documents

The `$group` stage groups documents by a specified field and performs aggregation operations (like counting, summing, averaging, etc.).

**Syntax:**
```js
db.collection.aggregate([
   {
      $group: {
         _id: <expression>, // Grouping key
         <field1>: { <accumulator>: <expression> },
         <field2>: { <accumulator>: <expression> },
         ...
      }
   }
])
```

**Example:**
```js
db.sales.aggregate([
   {
      $group: {
         _id: "$item", // Group by 'item'
         totalSales: { $sum: "$quantity" } // Sum the 'quantity' field
      }
   }
])
```
This groups the documents by the `item` field and calculates the total sales for each item.

#### **2.3. $project** - Shaping the Output

The `$project` stage reshapes each document, allowing you to include or exclude specific fields, add computed fields, or rename fields.

**Syntax:**
```js
db.collection.aggregate([
   { $project: { <field1>: 1, <field2>: 0, ... } }
])
```

**Example:**
```js
db.sales.aggregate([
   { $project: { item: 1, quantity: 1, totalPrice: { $multiply: ["$quantity", "$price"] } } }
])
```
This example includes the `item` and `quantity` fields and creates a new `totalPrice` field by multiplying the `quantity` and `price` fields.

#### **2.4. $sort** - Sorting Documents

The `$sort` stage sorts the documents based on the specified field in ascending (`1`) or descending (`-1`) order.

**Syntax:**
```js
db.collection.aggregate([
   { $sort: { <field>: 1 or -1 } }
])
```

**Example:**
```js
db.sales.aggregate([
   { $sort: { totalSales: -1 } } // Sort by 'totalSales' in descending order
])
```

#### **2.5. $limit** - Limiting the Number of Documents

The `$limit` stage limits the number of documents passed to the next stage.

**Syntax:**
```js
db.collection.aggregate([
   { $limit: <number> }
])
```

**Example:**
```js
db.sales.aggregate([
   { $limit: 5 } // Limits the result to 5 documents
])
```

#### **2.6. $skip** - Skipping Documents

The `$skip` stage skips a specified number of documents.

**Syntax:**
```js
db.collection.aggregate([
   { $skip: <number> }
])
```

**Example:**
```js
db.sales.aggregate([
   { $skip: 10 } // Skips the first 10 documents
])
```

#### **2.7. $lookup** - Joining Collections

The `$lookup` stage allows you to perform a left outer join to another collection.

**Syntax:**
```js
db.collection.aggregate([
   {
      $lookup: {
         from: <collection>,
         localField: <field>,
         foreignField: <field>,
         as: <output array field>
      }
   }
])
```

**Example:**
```js
db.orders.aggregate([
   {
      $lookup: {
         from: "customers", // Join with 'customers' collection
         localField: "customerId",
         foreignField: "_id",
         as: "customerDetails"
      }
   }
])
```
This performs a join between the `orders` collection and the `customers` collection, adding the customer details to the orders.

#### **2.8. $unwind** - Deconstructing Arrays

The `$unwind` stage is used to deconstruct an array field, outputting a document for each element in the array.

**Syntax:**
```js
db.collection.aggregate([
   { $unwind: "$arrayField" }
])
```

**Example:**
```js
db.orders.aggregate([
   { $unwind: "$items" } // Unwinds the 'items' array
])
```
This will create a new document for each item in the `items` array.

---

### **3. Combining Multiple Stages**

You can chain multiple stages together to form more complex aggregations.

**Example:**
```js
db.sales.aggregate([
   { $match: { item: "apple" } },             // Filter to 'apple' sales
   { $group: { _id: "$store", totalSales: { $sum: "$quantity" } } }, // Group by 'store'
   { $sort: { totalSales: -1 } },             // Sort by total sales in descending order
   { $limit: 3 }                              // Limit to top 3 stores
])
```

### **4. Accumulator Operators**

Accumulators are used within `$group` to perform calculations. Some common accumulators are:

- `$sum`: Adds values together.
- `$avg`: Calculates the average.
- `$min`: Returns the minimum value.
- `$max`: Returns the maximum value.
- `$first`: Returns the first value from the grouped documents.
- `$last`: Returns the last value from the grouped documents.

**Example:**
```js
db.sales.aggregate([
   {
      $group: {
         _id: "$item",
         totalQuantity: { $sum: "$quantity" },
         averagePrice: { $avg: "$price" }
      }
   }
])
```

---

### **5. Advanced Topics**

#### **5.1. $facet** - Multi-faceted Aggregations

The `$facet` stage allows you to run multiple aggregations on the same set of input documents in parallel and return the results in a single output.

**Example:**
```js
db.sales.aggregate([
   {
      $facet: {
         "categoryA": [ { $match: { category: "A" } }, { $group: { _id: null, total: { $sum: "$quantity" } } } ],
         "categoryB": [ { $match: { category: "B" } }, { $group: { _id: null, total: { $sum: "$quantity" } } } ]
      }
   }
])
```

#### **5.2. $bucket** and $bucketAuto - Bucketing Documents

The `$bucket` stage categorizes documents into groups (buckets) based on a specified field.

**Example:**
```js
db.sales.aggregate([
   {
      $bucket: {
         groupBy: "$price", 
         boundaries: [ 0, 10, 20, 30 ], 
         default: "Other",
         output: {
            "count": { $sum: 1 },
            "averageQuantity": { $avg: "$quantity" }
         }
      }
   }
])
```

---

### **6. Aggregation Pipeline Optimization**

MongoDB optimizes the pipeline to improve performance. You can help this process by placing `$match` and `$sort` stages early in the pipeline to reduce the number of documents processed.

---

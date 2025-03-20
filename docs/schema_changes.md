# Schema Evolution and Feature Implementation: MongoDB vs. PostgreSQL

When upgrading an e-commerce application from version 1 to version 2, several new features might be introduced, such as tracking sellers, customer reviews with sentiment analysis, and similar product recommendations. Below, we compare how MongoDB and PostgreSQL handle these changes.

---

## 1. Feature: Tracking Sellers in Orders

### **MongoDB Approach**
MongoDB's flexible schema allows you to introduce a `"seller"` field without downtime.

#### **Updating Existing Documents**
```javascript
db.products.updateMany(
    {'_id': {$in: ['id0', 'id1', 'id2']}},
    { $set: {seller: "Unknown Seller"}}
)
```

#### **Advantages**
- **No Downtime** – No need for predefined schema modifications.
- **Application Flexibility** – Update application logic to include `"seller"` in new orders.
- **Efficient Queries** – Easily group and filter documents by seller.

---

### **PostgreSQL Approach**
Schema modifications in PostgreSQL require structured changes.

#### **Steps to Implement**
1. **Alter Table** – Add a new column:
   ```sql
   ALTER TABLE products ADD COLUMN seller VARCHAR(255) DEFAULT 'Unknown Seller';
   ```
2. **Data Migration** – Populate existing rows.
3. **Application Changes** – Ensure new inserts include the `"seller"` field.
4. **Schema Constraints** – Add foreign keys if needed.

> **Consideration:** Schema migrations can introduce downtime and require careful planning.

---

## 2. Feature: Customer Reviews with Sentiment Analysis

### **MongoDB Approach**
MongoDB applies the **subset pattern** to store only the **last 15 reviews** per product, keeping queries efficient while maintaining recent records. In addition, MongoDB’s **Atlas Vector Search** can be leveraged to perform sentiment analysis by storing sentiment vectors within the review documents.

#### **Storing the Last 15 Reviews with Sentiment Vectors**
```javascript
db.products.updateOne(
    { _id: productId },
    {
        $push: {
            reviews: {
                $each: [{
                    reviewer_id: userId,
                    rating: 5,
                    comment: "Great product!",
                    comment_embedding: [0.8, 0.1, 0.05]  // Example sentiment vector
                }]
            }
        }
    }
)
```

#### **Example: Performing a Vector Search Query for Sentiment Analysis**
```javascript
db.products.aggregate([
    {
        $vectorSearch: {
            query: [0.75, 0.12, 0.08],  // Embedded query
            path: "reviews.comment_embedding",
            numCandidates: 50,
            limit: 5,
            index: "sentimentVectorIndex"
        }
    }
])
```

#### **Advantages**
- **Efficient Storage** – Only recent reviews are stored (subset pattern), reducing data bloat.
- **Integrated Vector Search** – Directly query embedded sentiment vectors for products with similar customer feedback.
- **Real-time Filtering** – Leverage MongoDB’s native vector search capabilities for fast, scalable sentiment analysis.

---

### **PostgreSQL Approach**
For PostgreSQL, a high-level approach for sentiment analysis involves:
- **Storing Sentiment Vectors** in a separate table or as part of a JSONB column.
- **Using the PGVector Extension** to index and perform vector similarity searches.
- **Querying with Vector Similarity Functions** to find products with comparable sentiment, though this requires additional setup and manual index tuning.

> **Consideration:** PostgreSQL's vector search with PGVector is powerful but demands more manual schema modifications and tuning compared to MongoDB’s integrated solution.

---

## 3. Feature: Similar Products Using Vector Search

### **MongoDB Approach**
MongoDB 7.1 introduces **$vectorSearch** for efficient similarity searches using **Atlas Vector Search**.

#### **Finding Similar Products**
```javascript
db.products.aggregate([
    {
        $vectorSearch: {
            query: productVector,
            path: "feature_vector",
            numCandidates: 50,
            limit: 5,
            index: "productVectorIndex"
        }
    }
])
```

#### **Advantages of Atlas Vector Search**
- **No Schema Overhaul** – Vectors are stored directly within product documents.
- **Seamless Scaling** – Managed indexing and automatic scaling in MongoDB Atlas.
- **Integrated Search** – Combine vector search with text search, filters, and aggregations in one query.
- **No External Dependencies** – No need for extra extensions or manual tuning.

---

### **PostgreSQL Approach**
PostgreSQL supports vector search using the **PGVector** extension.

#### **High-Level Steps**
- **Install PGVector** to enable vector operations.
- **Store Vectors** in the appropriate schema.
- **Create an Index** (e.g., using `ivfflat` or `hnsw`) for fast retrieval.
- **Query Using Distance-Based Similarity** to find related products.

> **Consideration:** While PGVector is a robust solution, it requires more manual configuration and tuning compared to MongoDB's integrated Atlas Vector Search.

---

## **Conclusion**
| Feature                     | MongoDB Advantages                                      | PostgreSQL Challenges                                 |
|-----------------------------|--------------------------------------------------------|------------------------------------------------------|
| **Tracking Sellers**        | No schema changes, instant updates                     | Requires structured schema changes, migrations       |
| **Customer Reviews**        | **Subset pattern (last 15 reviews)**, integrated vector search for sentiment analysis  | Requires separate storage, additional setup for vector search |
| **Similar Products Search** | **Atlas Vector Search** (scalable, integrated)         | Requires PGVector, manual tuning, schema changes      |

MongoDB’s **schema flexibility**, **Atlas Vector Search**, and **native support for modern features** enable seamless upgrades. In contrast, PostgreSQL requires **structured schema changes**, **manual index tuning**, and **additional extensions** to achieve similar functionality.

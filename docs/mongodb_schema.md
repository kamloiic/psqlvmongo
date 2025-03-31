# MongoDB Schema Solution

This document outlines a possible MongoDB schema design for the discussed application. In alignment with the specifications documented in [e-commerce webpage description](/docs/ecommerce.md), we have  applied different MongoDB design patterns to address the distinct requirements of each component within our e-commerce application.

It is important to note that this represents one possible approach among many valid solutions. Database schema design inherently involves trade-offs, and the optimal solution depends on each specific use case requirements, traffic patterns, and development priorities. This design serves as an illustrative example rather than a universal solution.

![MongoDB_Schema](/docs/pics/mongodb_schema.png)

## Products Collection

The Products collection is designed to efficiently store and retrieve product data while optimizing for the most common access patterns. For this collection, we have implemented the Subset Pattern for reviews:

- We store only the 15 most recent reviews within each product document in the `last_15_reviews` array
- The complete set of reviews remains in the dedicated Reviews collection
- We include essential review information (rating, text, user details) in the subset to avoid additional queries

When implementing this pattern, we maintain the subset with an update strategy:
- When a new review is added, we push it to the product's `last_15_reviews` array
- If the array exceeds 15 items, we remove the oldest review from the array
- This maintains a sliding window of the most recent reviews

### Subset Pattern Implementation

This pattern provides several key advantages:
- **Performance optimization**: Displays recent reviews on product pages without additional queries
- **Document size management**: Prevents exceeding MongoDB's 16MB document limit by limiting the number of embedded reviews
- **Optimized for common use cases**: Most users only view the most recent reviews
- **Reduced read latency**: No need for joins or lookups for the most common scenarios
- **Balanced approach**: Complete review history is still available when needed

For more information, see MongoDB's documentation on the [Subset Pattern](https://www.mongodb.com/blog/post/building-with-patterns-the-subset-pattern).

## Reviews Collection

The Reviews collection is designed to store comprehensive review data for all products in our e-commerce application. This collection serves as the source of truth for all reviews in the system.

This collection is optimized for:
- **Complete review history**: Maintains a comprehensive archive of all reviews ever written
- **Flexibility in querying**: Supports queries across multiple dimensions (by product, by user, by rating)
- **Analytics capabilities**: Enables in-depth analysis of review patterns and trends
- **Source of truth**: Acts as the authoritative source from which the subset in Products is derived

This collection design complements the Subset Pattern used in the Products collection. The Reviews collection provides complete historical data when needed, while the Products collection embeds just the most recent reviews for performance optimization. This separation of concerns allows us to balance comprehensive data storage with efficient query performance across the application.

## Users Collection

The Users collection manages user profiles and their associated information for our e-commerce platform.

For the Users collection, we've embedded the `payment_methods` and `address` arrays directly within each user document. This approach represents a fundamental MongoDB design approach: data that is accesed together should be stored together.

This approach offers several advantages:

- **Atomic operations**: All user data can be updated in a single operation
- **Data locality**: Related user information is stored together, improving read performance
- **Simplified queries**: Common operations like "get user profile with all addresses" require only one query
- **Reduced complexity**: No need to manage relationships across multiple collections for user-specific data

This straightforward embedding works well for the Users collection because:

1. The number of addresses and payment methods per user is typically small
2. This data is almost always accessed together with the user profile
3. The embedded documents have a clear ownership relationship with the parent document


## Carts Collection

The Carts collection manages active shopping carts in our e-commerce application. This collection is designed to optimize for frequent cart operations and quick checkout processes.

We embed the `cart_items` array directly within each cart document. This approach:

- **Reduces query complexity** by retrieving all cart items in a single operation
- **Ensures atomic updates** so cart modifications are all-or-nothing
- **Improves performance** for the common operation of viewing the full cart


### Extended Reference Pattern Implementation

We implement the Extended Reference Pattern in our cart schema in two ways:

For product information in the cart items:
- **Eliminates the need for joins** with the Products collection when displaying the cart
- **Improves read performance** for cart display operations

For user information:
- **Creates a clean reference** to the Users collection via the user_id
- **Maintains data integrity** by keeping a single source of truth for user data

Together, these patterns create a cart schema that efficiently supports the core shopping experience, enabling fast cart retrieval and modification while maintaining connections to the broader data model.

For more information, see MongoDB's documentation on the [Extended Reference](https://www.mongodb.com/blog/post/building-with-patterns-the-extended-reference-pattern) pattern.


## Orders Collection

The Orders collection captures completed transactions in our e-commerce platform. This collection is designed to provide comprehensive order history while optimizing for frequent reads and occasional writes.

Similar to the Carts collection, we embed the complete `order_items` array within each order document. This approach:

- **Creates a complete historical record** of what was purchased and at what price
- **Preserves order information** even if products are later modified or removed
- **Enables fast order retrieval** without joins or lookups

### Extended Reference Pattern

We implement the Extended Reference Pattern in multiple areas of the Orders collection:

1. **Order items** include extended product information rather than just IDs
2. **Payment method** information includes essential details for reference
3. **Addresses** contain complete shipping information

This pattern:
- **Reduces dependency on other collections** for order display and processing
- **Creates a self-contained record** for each transaction
- **Improves read performance** for order history and details pages
- **Preserves historical state** regardless of future changes to referenced entities
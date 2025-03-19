## E-commerce Webpage Description

The application includes several key interfaces that reflect the typical e-commerce user experience. These  interface descriptions, that include wireframe representations and MQL example queries, illustrate the required data and functionality and should be your starting point when designing the data schema in MongoDB. Remember, when designing a data model in MongoDB you should first start with the requirements, i.e., understand access patterns, identify relationships and define performance requirements, and then, move on to designing the schema that best supports those requirements.

### Homepage

![Homepage Wireframe](/docs/pics/ecommerce_homepage.png)

The homepage serves as the main landing page for customers and represents the primary product discovery interface. Key features include:

- **Product Grid**: Six promoted products displayed that allows for easy scanning of multiple offerings at once.

- **Product Information**: Each product card displays essential information needed for quick decision-making:
  - Product name
  - Product price
  - Product image

- **Add to Cart Functionality**: Each product includes a dedicated "Add to Cart" button that enables customers to make immediate purchase decisions without navigating to product detail pages. To see how this functionality is implemented at the database level, refer to the [Add Item To The Cart](#add-item-to-the-cart) section, which details the MongoDB query operations.

From a database perspective, this interface primarily interacts with the product catalog, requiring efficient querying of product details, inventory status, and pricing information. The MongoDB schema will need to support rapid retrieval of these product listings. For example, this query serves the primary purpose of retrieving the latest 6 published products with their names and prices, enabling quick display of recent additions to the catalog for users browsing the storefront.

```javascript
db.products.find({ 
    promoted: true,
    is_active: true, 
    stock_quantity:{ 
        $gt: 1
    } 
})
.project({
    product_name: 1,
    unitary_price: 1,
    picture_link: 1
})
.sort({ 
    published: -1 
})
.limit(6);
```

### Product Page

![Product Page Wireframe](/docs/pics/product_page.png)

The product page provides detailed information about individual products, facilitating informed purchase decisions. Key features of this interface include:

- **Product Image**: A section displaying a high-resolution picture of the product.

- **Product Information**: This section delivers detailed product data to assist customers in evaluating their options:
  - **Product Name**: 
  - **Price**: Provides the product unitary price
  - **Description**: A text area providing in-depth information about the product’s features, specifications, and benefits.

- **Add to Cart Functionality**: An easily accessible button allows users to quickly add the product to their cart. To see how this functionality is implemented at the database level, refer to the [Add Item To The Cart](#add-item-to-the-cart) section, which details the MongoDB query operations.

- **Customer Reviews**: A section displaying the first 15 reviews. Additional reviews become visible as the user scrolls down, offering insights into customer satisfaction and experiences.

From a database perspective, this interface requires fast access to specific product details, including descriptions, pricing, availability, and the last 15 reviews. The MongoDB schema should support efficient retrieval and display of this information. Here's an example query that retrieves detailed information for a particular product, including its reviews:
```javascript
db.products.find({ 
      product_id: "real_product_id",
      is_active: true,
  }).
  project({
      product_name: 1,
      unitary_price: 1,
      description: 1,
      picture_link: 1
      last_15_reviews:1,
      stock_quantity:1,
      weight:1,
      dimensions:1
    });
```

### Review Cart and Confirm Order Page

![Review Cart and Confirm Order Wireframe](/docs/pics/cart.png)

The cart page allows customers to review their own cart, where they can see their selected items before completing the purchase. This checkout interface includes several key components:

- **Product List**: Displays all items added to the cart in a scrollable list format, showing:
  - Product pictures: Visual representations of each item
  - Product name
  - Product description
  - Product price: Individual pricing for each item
  - Quantity: Number of items chosen
  - Quantity controls: "+1/-1" buttons allowing customers to adjust quantities directly from the cart

- **Selection Controls**: Checkboxes next to each product enable users to:
  - Select/deselect specific items for purchase
  - Control which items are included in the final transaction

- **Order Processing**:
  - "Place order" button: This button completes the purchase process and add a new order.

From a database perspective, this interface interacts mainly with the carts collection. The MongoDB schema will need to support efficient retrieval of a specific costumer cart, containing product details and management of cart state. Here's an example query that supports retrieval of a user's cart with product details:

```javascript
db.carts.find({
  user_id:"real_user_id",
  cart_id:"real_cart_id"
}).
project({
  "cart_items.product_name":1,
  "cart_items.description":1,
  "cart_items.quantity":1,
  "cart_item.unitary_price":1,
  "cart_item.picture_link":1,
}). sort("cart_items.added_at":-1)
```
The total amount can be compute running an aggregation pipeline. An example query is provided below:
```javascript
db.carts.aggregate([
  {
    $match: {
      cart_id:"real_cart_id",
			user_id:"real_user_id"
    }
  },
  {
    $addFields: {
      filtered_items: {
        $filter: {
          input: "$cart_items",
          as: "item",
          cond: { $in: ["$$item.product_name", ["selected_product_1", "selected_product_2"]] }
        }
      }
    }
  },
  {
    $project: {
      filtered_items: 1,
      totalAmount: {
        $sum: {
          $map: {
            input: "$filtered_items",
            as: "item",
            in: {
              $multiply: ["$$item.quantity", "$$item.unitary_price"]
            }
          }
        }
      }
    }
  }
])
```
#### Add Item to the Cart

As discussed in the Homepage and Product page sections, items can be added to the cart from multiple interfaces within the application. Both the Homepage's product and the Product Page include 'Add to Cart' buttons that trigger the cart update process. When a customer clicks these buttons, the application needs to handle adding new products, or updating existing product quantities. From a database perspective, this is how an example query would look like:

```javascript
[{
  $match: {
  	email:"user_email"
  }
},
   {
      $set: {
        cart_item: { 
          $ifNull: ["$cart_item", []] 
        },
        total_price: { 
          $ifNull: ["$total_price", 0] 
        }
      }
    },
    {
      $set: {
        productExists: {
          $in: ["product_name", { 
            $map: { 
              input: "$cart_item", 
              as: "item", 
              in: "$$item.product_name" 
            } 
          }]
        }
      }
    },
    {
      $set: {
        cart_item: {
          $cond: {
            if: "$productExists",
            then: {
              $map: {
                input: "$cart_item",
                as: "item",
                in: {
                  $cond: {
                    if: { $eq: ["$$item.product_name", "product_name"] },
                    then: {
                      product_name: "$$item.product_name",
                      description: "$$item.description",
                      quantity: { $add: ["$$item.quantity", 1] },
                      price: "$$item.price",
                      added_at: "$$item.added_at"
                    },
                    else: "$$item"
                  }
                }
              }
            },
            else: {
              $concatArrays: [
                "$cart_item",
                [{
                  product_name: "product_name",
                  description: "description",
                  quantity: 1,
                  price: 149.99,
                  added_at: new Date()
                }]
              ]
            }
          }
        },
        total_price: {
          $add: ["$total_price", price]
        },
        last_modified: new Date()
      }
    },
    {
      $unset: ["productExists"]
    }
  ]
```

#### Remove Item from the Cart

On the other hand deleting items from the cart requires a specialized database operation that can identify the specific cart item, remove it from the cart_item array, and update all related cart information in a single atomic transaction. This ensures that customers always see an accurate representation of their cart contents and total price, even after removing items. Find a query example

```javascript
[
    {
      $match: {
        email:"email",
        "cart_item.product_name": "product_name"  
      }
    },
    {
      $set: {
        itemToRemove: {
          $filter: {
            input: "$cart_item",
            as: "item",
            cond: { $eq: ["$$item.product_name", "product_name"] }
          }
        }
      }
    },
    {
      $set: {
        priceToDeduct: {
          $multiply: [
            { $arrayElemAt: ["$itemToRemove.price", 0] },
            { $arrayElemAt: ["$itemToRemove.quantity", 0] }
          ]
        }
      }
    },
    {
      $set: {
        cart_item: {
          $filter: {
            input: "$cart_item",
            as: "item",
            cond: { $ne: ["$$item.product_name", "product_name"] }
          }
        },
        
        total_price: { $subtract: ["$total_price", "$priceToDeduct"] },
        
        last_modified: new Date(),
        
        status: {
          $cond: {
            if: { 
              $eq: [
                { 
                  $size: {
                    $filter: {
                      input: "$cart_item",
                      as: "item",
                      cond: { $ne: ["$$item.product_name", "product_name"] }
                    }
                  }
                }, 
                0
              ]
            },
            then: "empty",
            else: "active"
          }
        }
      }
    },
    {
      $unset: ["itemToRemove", "priceToDeduct"]
    }
  ]
  ```

 #### Order Processing:
  "Place order" button: It displays a call-to-action that completes the purchase process. When clicked, this button initiates the creation of a new order record in the orders collection, which then becomes visible in the Orders page. This action effectively transforms cart items into an order with a unique tracking number and timestamp. To see how this works from a database perspective navigate to [orders page](#orders-page).



### Orders Page

![Orders Page Wireframe](/docs/pics/orders_page.png)

The Orders page provides customers with a comprehensive view of their order history, enabling them to track and manage previous purchases. Key features of this interface include:

- **Order List**: Displays all orders made by the customer in a scrollable list format, showing:
  - Order picture: Visual picture representing the product
  - Order creation date: When the order was placed
  - Order amount: Total cost of the purchase
  - Tracking number: Unique identifier for shipping and order management
  - Name: Name of the product
  - Description: Brief summary of the product

- **Order Details**: Each order entry includes a "View Details" button that allows customers to access more comprehensive information about their specific purchase, including detailed item listings, shipping status, and delivery information. This will open a separate tab named Order Details (link)

From a database perspective, this interface primarily interacts with the orders collection. The MongoDB schema will need to support efficient retrieval of order history for a specific customer, including basic order details with the option to access more comprehensive information upon request. Here's an example query that retrieves a user's order history:

```javascript
db.orders.find({ 
    email: "customer_email"
})
.project({
    created_at: 1,
    "items.product_name": 1,
    "items.description":1,
    "items.quantity":1,
    tracking_number: 1,
})
.sort({ 
    created_at: -1 
});
```

When considering how orders items are added to an order from a database perspective, an example query, where one document is created for each item in the cart, could be as follows:
```javascript
db.orders.insert({ 
     email: "customer@example.com",
  phone: "+1234567890",
  items: [
    {
      product_id: ObjectId("product1id"),
      product_name: "Wireless Headphones",
      quantity: 1,
      unit_price: 149.99,
      total_item_price: 149.99
    },
    {
      product_id: ObjectId("product2id"),
      product_name: "Smart Watch",
      quantity: 2,
      unit_price: 199.99,
      total_item_price: 399.98
    }
  ],
  status: "placed",
  amount: 549.97,
  shipping_fee: 15.00,
  tax_amount: 45.37,
  addresses: {
    shipping: {
      street: "123 Main St",
      city: "Anytown",
      state: "NY",
      zipcode: "10001",
      country: "USA"
    },
    billing: {
      street: "123 Main St",
      city: "Anytown",
      state: "NY",
      zipcode: "10001",
      country: "USA"
    }
  },
  tracking_number: "TRK" + new Date().getTime().toString().substring(7),
  created_at: new Date(),
  updated_at: new Date(),
  payment_method: "credit_card"
});
```

### Order Details Page

![Order Details Wireframe](/docs/pics/order_details_page.png)

The Order Details page provides customers with a comprehensive view of a specific order information after clicking the "View Details" button from the [orders page](#orders-page). This interface displays all relevant information about a selected order, including:

- **Order Identification**:
  - Tracking number
  - Order status: Indicates the current state of the order (processing, shipped, delivered, etc.)

- **Address Information**:
  - Billing address: Complete billing address used for the order
  - Shipping address: Delivery destination for the purchased item

- **Payment Information**:
  - Payment method: The method used to complete the transaction (credit card, PayPal, etc.)
  - Price breakdown: Clear itemization of costs including:
    - Subtotal: Base price of all items
    - VAT/Tax: Applied taxes
    - Shipping fee: Cost for delivery
    - Total amount: Final payment amount

- **Product Details**:
  - Product picture: Visual representation of the purchased item
  - Product name: Clear identification of the item purchased
  - Product description: Additional details about the purchased product

From a database perspective, this interface retrieves detailed information from a specific order document in the orders collection. Here's an example query that supports the retrieval of comprehensive order details:

```javascript
db.orders.find({ 
    email: "user_email",
    tracking_number: "tracking_number_value"
})
.project({
    tracking_number: 1,
    status: 1,
    "addresses.shipping_address": 1,
    "adresses.billing_address":1
    payment_method: 1,
    "item.product_name":1,
    "item.description":1,
    "item.quantity":1,
    "item.unitary_price:":1,
    "item.picture_link":1,
    tax_amount:1,
    shipping_fee: 1,
});

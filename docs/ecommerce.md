## E-commerce Frontend Wireframes

The application includes several key interfaces that reflect the typical e-commerce user experience. These wireframes illustrate how the database schema supports  user interactions.

### Homepage

![Homepage Wireframe](/docs/pics/ecommerce_homepage.png)

The homepage serves as the main landing page for customers and represents the primary product discovery interface. Key features include:

- **Product Grid**: Six featured products displayed in a responsive grid layout, allowing for easy scanning of multiple offerings at once. The grid automatically adapts to different screen sizes to ensure an optimal viewing experience.

- **Product Information**: Each product card displays essential information needed for quick decision-making:
  - Product name: Clear, readable text that identifies the item
  - Product price: Prominently displayed to facilitate purchase decisions
  - Product image: High-visibility area (shown as placeholders in the wireframe) to showcase the product visually

- **Add to Cart Functionality**: Each product includes a dedicated "Add to Cart" button that enables customers to make immediate purchase decisions without navigating to product detail pages.

- **Navigation Elements**:
  - Search bar: Positioned at the top for product discovery through keyword searches
  - User profile access: Icon in the top-right providing account management functions
  - Shopping cart: "View cart" button giving immediate access to review selected products
  - Scrollable interface: Vertical scrolling to browse additional products beyond the initially visible offerings


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
    name: 1,
    price: 1,
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

- **Product Images**: A prominent section displaying high-resolution images of the product. Users can browse through multiple images using clickable thumbnails, enhancing their visual understanding of the item.

- **Product Information**: This section delivers detailed product data to assist customers in evaluating their options:
  - **Product Name and Price**: Displayed near the top, making it easy to recognize and assess the product financially.
  - **Description**: A text area providing in-depth information about the product’s features, specifications, and benefits.

- **Add to Cart Functionality**: An easily accessible button allows users to quickly add the product to their cart, encouraging efficient purchase processes.

- **Customer Reviews**: A section devoted to user-generated content displaying the first 15 reviews. Additional reviews become visible as the user scrolls down, offering insights into customer satisfaction and experiences.

- **Navigation Elements**:
  - **Search Bar**: Positioned for straightforward keyword-based search capabilities, designed to streamline product discovery.
  - **Profile Access**: Easily recognized icon enabling users to enter account management and profile settings.
  - **View Cart**: Quick access to the shopping cart is provided to review selected items.
  - **Scrollable Interface**: Allows browsing through detailed product information without leaving the page.

From a database perspective, this interface requires fast access to specific product details, including descriptions, pricing, availability, and reviews. The MongoDB schema should support efficient retrieval and display of this information. Here's an example query that retrieves detailed information for a particular product, including its reviews:
```javascript
db.products.find({ 
      _id: ObjectId("productid"),
      is_active: true,
  }).
  project({
      name: 1,
      price: 1,
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
  - Product thumbnails: Visual representations of each item
  - Product name and brief description: Clear identification of items being purchased
  - Product price: Individual pricing for each item
  - Quantity controls: "+1/-1" buttons allowing customers to adjust quantities directly from the cart

- **Selection Controls**: Checkboxes next to each product enable users to:
  - Select/deselect specific items for purchase
  - Potentially save items for later or remove them from the order
  - Control which items are included in the final transaction

- **Order Processing**:
  - "Place order" button: Prominently displayed call-to-action that completes the purchase process
  - Scrollable interface: Allows reviewing all cart items regardless of order size

From a database perspective, this interface interacts mainly with the carts collection. The MongoDB schema will need to support efficient retrieval of a specific costumer cart, containing product details and management of cart state. Here's an example query that supports retrieval of a user's cart with product details:

```javascript
db.carts.find({
  email:"user_email"
}).
project({
  email:0,
  cart_items:1,
  total_price:1
}). sort("cart_items.added_at":-1)
```

#### Add Item to the Cart

As discussed in the Homepage and Product page sections, items can be added to the cart from multiple interfaces within the application. Both the Homepage's product grid and the Product Page include 'Add to Cart' buttons that trigger the cart update process. When a customer clicks these buttons, the application needs to handle various scenarios: creating a new cart, adding new products, or updating existing product quantitie

```javascript
[{
  $match: {
  	email:"user_email"
  }
},
   {
      $set: {
        // First, set defaults for a new document if needed
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
        // Check if the product exists in the cart
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
        // Update the cart based on whether product exists
        cart_item: {
          $cond: {
            if: "$productExists",
            then: {
              // Product exists - update its quantity
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
              // Product doesn't exist - add it to cart
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
        
        // Always update the total price
        total_price: {
          $add: ["$total_price", price]
        },
        
        // Always update last_modified
        last_modified: new Date()
      }
    },
    {
      // Remove the temporary field
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
        // Find the item to be removed and calculate its total price
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
        // Calculate the price to deduct based on price × quantity
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
        // Remove the item from the cart
        cart_item: {
          $filter: {
            input: "$cart_item",
            as: "item",
            cond: { $ne: ["$$item.product_name", "product_name"] }
          }
        },
        
        // Update the total price
        total_price: { $subtract: ["$total_price", "$priceToDeduct"] },
        
        // Update last_modified timestamp
        last_modified: new Date(),
        
        // Set status to "empty" if cart becomes empty, otherwise keep it "active"
        status: {
          $cond: {
            if: { 
              $eq: [
                { 
                  $size: {
                    $filter: {
                      input: "$cart_item",
                      as: "item",
                      cond: { $ne: ["$$item.product_name", "Wireless Headphones"] }
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
      // Remove temporary fields
      $unset: ["itemToRemove", "priceToDeduct"]
    }
  ]
  ```
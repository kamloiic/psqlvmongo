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
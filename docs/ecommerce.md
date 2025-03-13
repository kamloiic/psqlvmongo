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
    _id: 0,
    name: 1,
    price: 1,
    picture_link: 1
})
.sort({ 
    published: -1 
})
.limit(6);
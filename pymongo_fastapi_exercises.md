# PyMongo FastAPI Exercises - Easy to Hard

Complete these 15 exercises to master PyMongo with FastAPI. Each exercise builds on previous concepts and increases in complexity.

## Setup Instructions

```bash
# Install dependencies
pip install fastapi pymongo uvicorn python-dotenv

# MongoDB connection (adjust as needed)
MONGODB_URL=mongodb://localhost:27017/
DATABASE_NAME=ecommerce_db
```

## Prerequisites

Before starting the exercises, make sure you have:
1. MongoDB installed and running locally (or access to MongoDB Atlas)
2. Seeded your database with the provided JSON files:
   - `users.json` → `users` collection
   - `products.json` → `products` collection
   - `orders.json` → `orders` collection

You can import them using `mongoimport` or insert them via PyMongo/MongoDB Compass.

## Working with ObjectId

Since MongoDB auto-generates ObjectIds for documents, you'll need to handle conversions between ObjectId and strings:

```python
from bson.objectid import ObjectId

# Converting string to ObjectId for querying
try:
    object_id = ObjectId(user_id_string)
    user = collection.find_one({"_id": object_id})
except Exception:
    # Invalid ObjectId format
    raise HTTPException(status_code=400, detail="Invalid ID format")

# Converting ObjectId to string for JSON responses
if user:
    user["_id"] = str(user["_id"])  # Convert ObjectId to string
    return user
```

**Helper Function for Converting ObjectIds:**
```python
def serialize_doc(doc):
    """Convert ObjectId to string in a document"""
    if doc and "_id" in doc:
        doc["_id"] = str(doc["_id"])
    return doc

def serialize_docs(docs):
    """Convert ObjectId to string in multiple documents"""
    return [serialize_doc(doc) for doc in docs]
```

---

## Exercise 1: Basic Connection and Find All (Easy)

**Objective**: Set up FastAPI with MongoDB and create an endpoint to retrieve all users.

**Requirements**:
- Create a FastAPI app
- Connect to MongoDB
- Create a GET endpoint `/users` that returns all users
- Convert ObjectId to string for JSON serialization

**Hints**:
- Use `collection.find()` to get all documents
- Remember to handle `_id` serialization (ObjectId → string)
- Iterate through cursor and convert each document

**Expected Output**:
```json
[
  {
    "_id": "507f1f77bcf86cd799439011",
    "name": "Alice Johnson",
    "email": "alice.johnson@email.com",
    ...
  },
  ...
]
```

---

## Exercise 2: Find One by ID (Easy)

**Objective**: Create an endpoint to retrieve a single user by their ID.

**Requirements**:
- Create a GET endpoint `/users/{user_id}`
- Return the user if found, otherwise return 404
- Handle the case where the user doesn't exist
- Convert string ID to ObjectId for querying

**Hints**:
- Import ObjectId: `from bson.objectid import ObjectId`
- Use `collection.find_one({"_id": ObjectId(user_id)})`
- Wrap ObjectId conversion in try-except to handle invalid IDs
- Use FastAPI's `HTTPException` for 404 responses
- Convert ObjectId back to string in response

**Test Cases**:
- GET `/users/507f1f77bcf86cd799439011` → Returns user (use actual ObjectId from your database)
- GET `/users/invalid_id` → Returns 400 (Invalid ID format)
- GET `/users/507f1f77bcf86cd799439099` → Returns 404 (Valid format but doesn't exist)

---

## Exercise 3: Create New Document (Easy-Medium)

**Objective**: Create an endpoint to add a new user to the database.

**Requirements**:
- Create a POST endpoint `/users`
- Accept user data with validation (use Pydantic models)
- MongoDB will auto-generate ObjectId
- Return the created user with status code 201

**Hints**:
- Use Pydantic `BaseModel` for request validation
- Use `collection.insert_one()` to add the document
- Access the generated ObjectId from `result.inserted_id`
- Set default values for fields like `joined_date`, `total_orders`, `total_spent`
- Convert ObjectId to string in response

**Request Body Example**:
```json
{
  "name": "John Doe",
  "email": "john.doe@email.com",
  "age": 30,
  "city": "Portland",
  "country": "USA"
}
```

---

## Exercise 4: Query with Filters (Medium)

**Objective**: Create an endpoint to search users with multiple filter options.

**Requirements**:
- Create a GET endpoint `/users/search`
- Support query parameters: `status`, `min_age`, `max_age`, `city`
- Return filtered results

**Hints**:
- Use query operators like `$gte`, `$lte`
- Build the filter dictionary dynamically based on provided parameters

**Test Cases**:
- GET `/users/search?status=active` → All active users
- GET `/users/search?min_age=30&city=Chicago` → Users 30+ in Chicago
- GET `/users/search?status=active&max_age=35` → Active users under 35

---

## Exercise 5: Update Document (Medium)

**Objective**: Create an endpoint to update user information.

**Requirements**:
- Create a PUT/PATCH endpoint `/users/{user_id}`
- Update only the provided fields (partial update)
- Return the updated user
- Return 404 if user doesn't exist
- Handle invalid ObjectId format

**Hints**:
- Convert string ID to ObjectId for querying
- Use `$set` operator with `update_one()`
- Use Pydantic model with all optional fields
- Check `matched_count` to verify the user exists
- Retrieve and return the updated document

**Request Body Example**:
```json
{
  "age": 29,
  "city": "Brooklyn"
}
```

---

## Exercise 6: Pagination (Medium)

**Objective**: Implement pagination for the products list.

**Requirements**:
- Create a GET endpoint `/products`
- Accept query parameters: `page` (default 1), `page_size` (default 10)
- Return paginated results with metadata

**Hints**:
- Use `skip()` and `limit()`
- Calculate: `skip = (page - 1) * page_size`
- Include total count in response

**Expected Response**:
```json
{
  "data": [...],
  "page": 1,
  "page_size": 10,
  "total": 10,
  "total_pages": 1
}
```

---

## Exercise 7: Array Operations (Medium)

**Objective**: Create endpoints to manage user tags.

**Requirements**:
- Create a POST endpoint `/users/{user_id}/tags/add` to add a tag
- Create a DELETE endpoint `/users/{user_id}/tags/remove` to remove a tag
- Ensure no duplicate tags when adding
- Handle invalid ObjectId format

**Hints**:
- Convert string ID to ObjectId
- Use `$addToSet` for adding (prevents duplicates)
- Use `$pull` for removing
- Return the updated user
- Check if user exists before updating

**Test Cases**:
- POST `/users/507f1f77bcf86cd799439011/tags/add` with body `{"tag": "favorite"}` → Adds tag
- DELETE `/users/507f1f77bcf86cd799439011/tags/remove` with body `{"tag": "favorite"}` → Removes tag

---

## Exercise 8: Aggregation - Basic Statistics (Medium-Hard)

**Objective**: Create an endpoint that returns product statistics by category.

**Requirements**:
- Create a GET endpoint `/products/stats`
- Group products by category
- Return: count, average price, min price, max price per category

**Hints**:
- Use aggregation pipeline with `$group`
- Use `$avg`, `$min`, `$max`, `$sum` operators

**Expected Response**:
```json
[
  {
    "_id": "Electronics",
    "count": 5,
    "avg_price": 477.99,
    "min_price": 29.99,
    "max_price": 1299.99
  },
  ...
]
```

---

## Exercise 9: Complex Query with Sorting (Medium-Hard)

**Objective**: Create an advanced product search endpoint.

**Requirements**:
- Create a GET endpoint `/products/advanced-search`
- Support parameters: `category`, `min_price`, `max_price`, `in_stock`, `min_rating`, `sort_by`, `sort_order`
- Apply sorting (price, rating, name)
- Return only active products

**Hints**:
- Build filter dynamically
- For `in_stock`, check `stock > 0`
- Use `sort()` with 1 (ascending) or -1 (descending)

**Test Cases**:
- GET `/products/advanced-search?category=Electronics&sort_by=price&sort_order=desc`
- GET `/products/advanced-search?min_price=50&max_price=200&in_stock=true`

---

## Exercise 10: Increment and Decrement (Medium-Hard)

**Objective**: Create endpoints to manage product stock.

**Requirements**:
- Create a POST endpoint `/products/{product_id}/stock/add` to increase stock
- Create a POST endpoint `/products/{product_id}/stock/remove` to decrease stock
- Prevent negative stock values
- Return error if attempting to remove more than available
- Handle invalid ObjectId format

**Hints**:
- Convert string ID to ObjectId
- Use `$inc` operator
- Check current stock before decrementing
- Use transactions for safety (optional but recommended)
- Validate quantity is positive

**Request Body Example**:
```json
{
  "quantity": 5
}
```

---

## Exercise 11: Lookup/Join Operations (Hard)

**Objective**: Create an endpoint that returns orders with complete user and product information.

**Requirements**:
- Create a GET endpoint `/orders/{order_id}/details`
- Join with users collection to get user details
- Join with products collection to get current product details for each item
- Return enriched order information

**Hints**:
- Use aggregation with `$lookup`
- May need multiple lookups or `$unwind` for array items
- Consider using multiple aggregation stages

**Expected Response Structure**:
```json
{
  "order_id": "order_001",
  "user": {
    "name": "Alice Johnson",
    "email": "alice.johnson@email.com"
  },
  "items": [
    {
      "product_name": "Wireless Mouse",
      "quantity": 2,
      "price": 29.99,
      "current_stock": 150,
      "current_price": 29.99
    }
  ],
  "total_amount": 99.97
}
```

---

## Exercise 12: Upsert and Complex Updates (Hard)

**Objective**: Create an endpoint to process an order which updates user statistics.

**Requirements**:
- Create a POST endpoint `/orders/process`
- Create the order document
- Increment user's `total_orders` by 1
- Increment user's `total_spent` by order amount
- Decrease product stock for each item in the order
- Use transactions to ensure atomicity

**Hints**:
- Use `with session.start_transaction():`
- Multiple update operations must succeed or all fail
- Validate stock availability before processing

**Request Body Example**:
```json
{
  "user_id": "user_001",
  "items": [
    {"product_id": "prod_002", "quantity": 2},
    {"product_id": "prod_005", "quantity": 1}
  ],
  "shipping_address": {...},
  "payment_method": "credit_card"
}
```

---

## Exercise 13: Text Search and Indexes (Hard)

**Objective**: Implement product search functionality with text indexes.

**Requirements**:
- Create a text index on product `name` and `tags`
- Create a GET endpoint `/products/search/text?q={query}`
- Return products matching the search term
- Sort by relevance score

**Hints**:
- Use `collection.create_index([("name", "text"), ("tags", "text")])`
- Use `$text` and `$search` operators
- Access text score with `{"score": {"$meta": "textScore"}}`

**Test Cases**:
- GET `/products/search/text?q=laptop` → Returns laptops
- GET `/products/search/text?q=gaming` → Returns gaming products

---

## Exercise 14: Advanced Aggregation - Sales Report (Hard)

**Objective**: Create a comprehensive sales report endpoint.

**Requirements**:
- Create a GET endpoint `/reports/sales`
- Accept date range parameters: `start_date`, `end_date`
- Return:
  - Total revenue
  - Number of orders
  - Top 5 best-selling products
  - Revenue by status
  - Average order value

**Hints**:
- Use multiple aggregation pipelines or stages
- `$match` for date filtering
- `$unwind` for order items
- `$group` for aggregations
- `$sort` and `$limit` for top products

**Expected Response**:
```json
{
  "date_range": {
    "start": "2024-01-01",
    "end": "2024-02-03"
  },
  "total_revenue": 4254.85,
  "total_orders": 8,
  "average_order_value": 531.86,
  "revenue_by_status": [
    {"status": "delivered", "revenue": 3200.00},
    {"status": "shipped", "revenue": 1299.99},
    ...
  ],
  "top_products": [
    {
      "product_id": "prod_001",
      "product_name": "Laptop Pro 15",
      "quantity_sold": 2,
      "revenue": 2599.98
    },
    ...
  ]
}
```

---

## Exercise 15: Bulk Operations and Performance (Hard)

**Objective**: Create an endpoint for bulk product updates with optimized performance.

**Requirements**:
- Create a POST endpoint `/products/bulk-update`
- Accept an array of product updates
- Use bulk operations for efficiency
- Support different operation types: update, upsert, delete
- Return summary of operations performed

**Hints**:
- Use `bulk_write()` with operations like `UpdateOne`, `DeleteOne`
- Process multiple operations in a single database round trip
- Return counts of successful operations

**Request Body Example**:
```json
{
  "operations": [
    {
      "type": "update",
      "product_id": "prod_001",
      "data": {"price": 1199.99}
    },
    {
      "type": "update",
      "product_id": "prod_002",
      "data": {"stock": 200}
    },
    {
      "type": "delete",
      "product_id": "prod_010"
    }
  ]
}
```

**Expected Response**:
```json
{
  "acknowledged": true,
  "matched_count": 2,
  "modified_count": 2,
  "deleted_count": 1,
  "upserted_count": 0
}
```

---

## Bonus Challenges

Once you've completed all 15 exercises, try these additional challenges:

1. **Rate Limiting**: Add rate limiting to prevent abuse
2. **Caching**: Implement Redis caching for frequently accessed data
3. **Full-Text Search**: Enhance text search with fuzzy matching
4. **Data Validation**: Add comprehensive validation rules
5. **API Documentation**: Generate automatic API docs with examples
6. **Testing**: Write unit tests for all endpoints
7. **Authentication**: Add JWT authentication and user permissions
8. **WebSocket**: Create a real-time order tracking feature
9. **Export**: Create endpoints to export data as CSV/Excel
10. **Analytics Dashboard**: Build complex analytics queries

---

## Tips for Success

- Always handle errors gracefully with appropriate status codes
- Validate input data using Pydantic models
- Use meaningful variable names
- Add proper logging
- Test edge cases (empty results, invalid IDs, etc.)
- Consider indexing frequently queried fields
- Use connection pooling (PyMongo handles this by default)
- Close connections properly or use context managers
- Comment your complex queries
- Refer to the PyMongo cheatsheet for syntax help

## Evaluation Criteria

- **Correctness**: Does it work as specified?
- **Error Handling**: Are edge cases handled?
- **Code Quality**: Is the code clean and readable?
- **Performance**: Are efficient MongoDB operations used?
- **API Design**: Are endpoints RESTful and well-structured?

Good luck with your exercises! 🚀

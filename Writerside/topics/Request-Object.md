# Request Object
![](requestobj.png)

The Request object (`req`) in Express represents the HTTP request and provides information about the request made to the server. It contains properties and methods to handle data such as query parameters, route parameters, headers, the request body, and much more.

---

### **a. What is Response Object?**

While the Request object represents the incoming HTTP request, the **Response object (`res`)** represents the HTTP response that an Express app sends when it gets an incoming request. It is used to send data back to the client (e.g., JSON, HTML, status codes, etc.).

Example:
```javascript
app.get('/example', (req, res) => {
  res.status(200).json({ message: 'Hello, world!' });
});
```

### **b. Request Object Methods**

The Request object provides several methods and properties for handling client requests. Below are some important ones:

- **`req.query`**: Access query parameters (e.g., `/users?name=John`).
- **`req.params`**: Access route parameters (e.g., `/users/:id`).
- **`req.body`**: Access the request body, commonly used in POST and PUT requests.
- **`req.headers`**: Access headers sent by the client.
- **`req.method`**: Get the HTTP method used in the request (GET, POST, etc.).
- **`req.url`**: The full URL requested.

---

### **i. Query Parameters (`req.query`)**

Query parameters are typically used to pass optional data to the server in the URL, usually for filtering, sorting, limiting, or pagination. They are appended to the URL after a `?` symbol.

#### Accessing Query Parameters:
```javascript
app.get('/users', (req, res) => {
  const { name, age } = req.query;
  res.json({ name, age });
});
```

#### Example URL:
```
/users?name=John&age=25
```
The above URL would populate `req.query` as:
```json
{
  "name": "John",
  "age": "25"
}
```

---

### **ii. Operations which Require Query Params**

#### a. Filtering
Filtering allows clients to retrieve specific data based on certain conditions. You can use query parameters to filter results.

Example:
```javascript
app.get('/products', (req, res) => {
  const { category } = req.query;
  // Logic to filter products based on category
  res.json({ message: `Filtered products by category: ${category}` });
});
```

#### b. Sorting
Sorting allows clients to receive data in a specific order (e.g., ascending or descending based on a field).

Example:
```javascript
app.get('/products', (req, res) => {
  const { sortBy, order } = req.query;
  // Logic to sort products based on sortBy field and order ('asc' or 'desc')
  res.json({ message: `Sorted by ${sortBy} in ${order} order` });
});
```

Example URL:
```
/products?sortBy=price&order=asc
```

#### c. Limiting
Limiting allows you to control how many results are returned from a query. This is especially useful for large datasets.

Example:
```javascript
app.get('/products', (req, res) => {
  const { limit } = req.query;
  // Logic to limit the number of products returned
  res.json({ message: `Limited to ${limit} products` });
});
```

Example URL:
```
/products?limit=10
```

#### d. Pagination
Pagination allows you to break large datasets into smaller chunks and fetch data page by page.

Example:
```javascript
app.get('/products', (req, res) => {
  const { page, pageSize } = req.query;
  const offset = (parseInt(page as string) - 1) * parseInt(pageSize as string);
  // Logic to return paginated results
  res.json({ message: `Page: ${page}, Page Size: ${pageSize}`, offset });
});
```

Example URL:
```
/products?page=2&pageSize=10
```

---

### **ii. Route Parameters (`req.params`)**

Route parameters are named URL segments that act as placeholders for data. They are defined in the route path with a colon (`:`) and can be accessed using `req.params`.

#### Example:
```javascript
app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  // Logic to fetch user by id
  res.json({ message: `User ID: ${id}` });
});
```

Example URL:
```
/users/123
```

In this case, `req.params` would contain:
```json
{
  "id": "123"
}
```

Route parameters are often used when you want to reference a specific resource (like a user, product, or article) by its unique identifier.

---

#### i. Operations which Require Route Params

##### a. Fetching a Single Resource
You can use route parameters to fetch a specific resource based on a unique identifier (e.g., fetching a single user by their ID).

Example:
```javascript
app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  // Fetch user by id
  res.json({ message: `Fetching user with ID: ${id}` });
});
```

##### b. Updating a Specific Resource
Route parameters can also be used for updating a specific resource.

Example:
```javascript
app.put('/users/:id', (req, res) => {
  const { id } = req.params;
  const { name, email } = req.body;
  // Update the user in the database
  res.json({ message: `User with ID ${id} updated`, name, email });
});
```

##### c. Deleting a Specific Resource
You can use route parameters to delete a specific resource based on its unique ID.

Example:
```javascript
app.delete('/users/:id', (req, res) => {
  const { id } = req.params;
  // Delete the user from the database
  res.status(204).json({ message: `User with ID ${id} deleted` });
});
```

---

### Summary

- **Query Parameters (`req.query`)**: Used for passing optional data like filtering, sorting, limiting, and pagination via URL queries (e.g., `/users?name=John`).
- **Route Parameters (`req.params`)**: Used for passing dynamic data directly in the URL path (e.g., `/users/:id`), commonly used for operations like fetching, updating, or deleting a resource.

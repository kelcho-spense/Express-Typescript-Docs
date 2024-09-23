# App Object (Express Object)

The `app` object in Express represents the core of your Express application. It is an instance of the `express()` function, and it provides methods to define routes, middleware, and configure various settings for your web application or API.

### Basic Express Application Setup
Before diving into the specific methods, here’s a simple example of how to initialize the `app` object.

```javascript
import express from 'express';

const app = express();
const port = 3000;

app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
```

The `app` object provides several HTTP methods that map to HTTP request verbs (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`, etc.). Additionally, `app.use()` allows you to mount middleware and handle requests before they reach the route handler.

---

### **a. `app.get()`**

`app.get()` handles HTTP **GET** requests to a specific route. This is typically used to retrieve or read data from the server.

```javascript
app.get('/users', (req, res) => {
  res.json([{ id: 1, name: 'John Doe' }]);
});
```

#### Parameters:
- **Path (string)**: The URL or endpoint you want to handle (e.g., `/users`).
- **Callback Function (middleware)**: A function that receives the `Request` and `Response` objects. It contains logic to process the request and return the response.

#### Example with Route Parameters:
```javascript
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.json({ id: userId, name: 'John Doe' });
});
```

#### Example with Query Parameters:
```javascript
app.get('/search', (req, res) => {
  const query = req.query.q; // Example: /search?q=express
  res.json({ message: `Searching for ${query}` });
});
```

---

### **b. `app.post()`**

`app.post()` handles HTTP **POST** requests. It is commonly used to send data to the server, typically for creating new resources (e.g., user registration, creating a blog post).

```javascript
app.post('/users', (req, res) => {
  const { name, email } = req.body; // Assuming body-parser middleware is used
  res.status(201).json({ id: 1, name, email });
});
```

#### Parameters:
- **Path (string)**: The URL or endpoint you want to handle (e.g., `/users`).
- **Callback Function**: Receives the `Request` and `Response` objects. The `req.body` contains data sent from the client (usually in JSON format).

#### Example: Posting Form Data
```javascript
app.use(express.json()); // Parse incoming JSON data
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded data

app.post('/users', (req, res) => {
  const { username, password } = req.body;
  res.status(201).json({ message: 'User created', username });
});
```

---

### **c. `app.put()`**

`app.put()` handles HTTP **PUT** requests. This is used to update a resource completely (replacing it with the new data). The client typically sends the full resource, and the server updates the record.

```javascript
app.put('/users/:id', (req, res) => {
  const userId = req.params.id;
  const { name, email } = req.body;
  // Update the user in the database here
  res.json({ message: `User ${userId} updated`, name, email });
});
```

#### Parameters:
- **Path (string)**: The URL or endpoint, often with a dynamic parameter (e.g., `/users/:id`).
- **Callback Function**: `req.body` contains the updated data sent by the client.

#### Example: Full Update
```javascript
app.put('/articles/:id', (req, res) => {
  const articleId = req.params.id;
  const { title, content } = req.body;
  // Replace the old article with the new one
  res.json({ message: `Article ${articleId} updated`, title, content });
});
```

---

### **d. `app.delete()`**

`app.delete()` handles HTTP **DELETE** requests, used to delete a resource on the server.

```javascript
app.delete('/users/:id', (req, res) => {
  const userId = req.params.id;
  // Remove the user from the database here
  res.status(204).json({ message: `User ${userId} deleted` });
});
```

#### Parameters:
- **Path (string)**: The URL or endpoint, often with a dynamic parameter (e.g., `/users/:id`).
- **Callback Function**: The request usually only contains the ID of the resource to delete.

#### Example: Delete Request with Confirmation
```javascript
app.delete('/posts/:id', (req, res) => {
  const postId = req.params.id;
  // Logic to delete post by postId
  res.status(200).json({ message: `Post ${postId} deleted` });
});
```

---

### **e. `app.patch()`**

`app.patch()` handles HTTP **PATCH** requests, used to update part of a resource (as opposed to `PUT`, which replaces the entire resource).

```javascript
app.patch('/users/:id', (req, res) => {
  const userId = req.params.id;
  const { email } = req.body; // Assume we're only updating the email
  // Update the user's email in the database here
  res.json({ message: `User ${userId} email updated`, email });
});
```

#### Parameters:
- **Path (string)**: The URL or endpoint, often with a dynamic parameter (e.g., `/users/:id`).
- **Callback Function**: Receives the partial update data in `req.body`.

#### Example: Partial Update
```javascript
app.patch('/profiles/:id', (req, res) => {
  const profileId = req.params.id;
  const updates = req.body; // Partial updates like { age: 30 }
  // Apply updates to the profile in the database
  res.json({ message: `Profile ${profileId} updated`, updates });
});
```

---

### **f. `app.use()`**

`app.use()` mounts middleware functions, which are functions that execute during the request-response cycle. Middleware can modify the `Request` and `Response` objects or terminate the request-response cycle.

#### Middleware Example: Global Middleware

```javascript
app.use((req, res, next) => {
  console.log(`${req.method} request made to ${req.url}`);
  next(); // Pass control to the next middleware or route handler
});
```

#### Middleware Example: Applying to a Specific Route
You can apply middleware to specific routes or route groups by passing a path as the first argument to `app.use()`.

```javascript
const authMiddleware = (req, res, next) => {
  const token = req.headers['authorization'];
  if (!token) return res.status(401).json({ message: 'Unauthorized' });
  // Verify token here (e.g., JWT verification)
  next();
};

app.use('/api/protected', authMiddleware, (req, res) => {
  res.json({ message: 'You have access to the protected route' });
});
```

#### Example: Using Built-in Middleware
Express provides built-in middleware to handle common tasks like parsing JSON and serving static files.

```javascript
// Built-in middleware to serve static files (HTML, CSS, images)
app.use(express.static('public'));

// Built-in middleware to parse incoming JSON data
app.use(express.json());

// Built-in middleware to parse URL-encoded data (from HTML forms)
app.use(express.urlencoded({ extended: true }));
```

#### Example: Third-Party Middleware
You can also use third-party middleware, such as `morgan` for logging or `cors` for Cross-Origin Resource Sharing.

```javascript
import cors from 'cors';
import morgan from 'morgan';

// Use morgan for logging requests
app.use(morgan('dev'));

// Enable CORS for all routes
app.use(cors());
```

---

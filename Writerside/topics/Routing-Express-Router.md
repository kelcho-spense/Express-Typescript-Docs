# Routing (Express.Router())

Express provides a powerful and flexible way to define routing for your web application or API. The `express.Router` is a modular, mountable route handler that allows you to organize your routes into separate files and group them logically.

Routing refers to how an application responds to client requests for a particular endpoint (URL path and HTTP method). Express routes can be simple, complex, or even organized into separate files using `express.Router()`.

---

### **Basic Routing**

Basic routing involves defining routes for HTTP methods (e.g., GET, POST, PUT, DELETE) using simple callback functions. This is typically used for handling a single endpoint.

#### Example:
```javascript
import express, { Request, Response } from 'express';

const app = express();

app.get('/home', (req: Request, res: Response) => {
  res.send('Welcome to the Home Page!');
});

app.post('/submit', (req: Request, res: Response) => {
  res.send('Form submitted!');
});

app.listen(3000, () => {
  console.log('Server is running on http://localhost:3000');
});
```

In this example:
- `app.get('/home', ...)`: Handles HTTP GET requests to `/home`.
- `app.post('/submit', ...)`: Handles HTTP POST requests to `/submit`.

---

### **Complex Routing with `express.Router`**

As your application grows, managing all your routes in a single file becomes impractical. To address this, Express provides the `Router()` function to create modular and mountable route handlers. Each router can handle its own set of routes and be exported from a file for use in the main application.

#### Example: Using `express.Router` for Complex Routing

```javascript
import express, { Request, Response } from 'express';

const router = express.Router();

// Route to get all users
router.get('/users', (req: Request, res: Response) => {
  res.json([{ id: 1, name: 'John Doe' }, { id: 2, name: 'Jane Doe' }]);
});

// Route to get a user by ID
router.get('/users/:id', (req: Request, res: Response) => {
  const { id } = req.params;
  res.json({ id, name: 'John Doe' });
});

// Route to create a new user
router.post('/users', (req: Request, res: Response) => {
  const { name } = req.body;
  res.status(201).json({ id: 3, name });
});

export default router;
```

#### In the main application:

```javascript
import express from 'express';
import userRoutes from './routes/userRoutes'; // Importing router

const app = express();
app.use(express.json());

// Mount the user routes under /api
app.use('/api', userRoutes);

app.listen(3000, () => {
  console.log('Server is running on http://localhost:3000');
});
```

In this example:
- We define several routes using `express.Router()`.
- We then mount the router to the main application using `app.use('/api', userRoutes)`.
- The API now has routes like `/api/users` and `/api/users/:id`.

---

### **Various Ways to Structure Routes**

There are different ways to structure routes depending on the complexity and size of your application. Let’s explore some common ways to organize routes in Express.

#### 1. **Flat Structure (Basic Applications)**

For small applications, a simple flat structure is often sufficient:

```
src/
  ├── app.ts        # Main application file
  └── routes/
      └── index.ts  # All routes defined in one file
```

```javascript
// src/routes/index.ts
import express from 'express';

const router = express.Router();

router.get('/', (req, res) => res.send('Hello World'));
router.get('/about', (req, res) => res.send('About Page'));

export default router;

// src/app.ts
import express from 'express';
import routes from './routes/index';

const app = express();
app.use('/', routes);

app.listen(3000, () => console.log('Server running'));
```

This is simple and ideal for applications with just a few routes.

#### 2. **Modular Structure (Medium Applications)**

For larger applications, you might organize routes by resource (e.g., users, posts, products).

```
src/
  ├── app.ts
  └── routes/
      ├── userRoutes.ts
      ├── postRoutes.ts
      └── index.ts
```

```javascript
// src/routes/userRoutes.ts
import express from 'express';
const router = express.Router();

router.get('/', (req, res) => res.send('Get all users'));
router.post('/', (req, res) => res.send('Create user'));

export default router;

// src/routes/postRoutes.ts
import express from 'express';
const router = express.Router();

router.get('/', (req, res) => res.send('Get all posts'));
router.post('/', (req, res) => res.send('Create post'));

export default router;

// src/routes/index.ts
import express from 'express';
import userRoutes from './userRoutes';
import postRoutes from './postRoutes';

const router = express.Router();

router.use('/users', userRoutes);
router.use('/posts', postRoutes);

export default router;

// src/app.ts
import express from 'express';
import routes from './routes';

const app = express();
app.use('/api', routes);

app.listen(3000, () => console.log('Server running'));
```

In this case:
- Routes are separated by resource (`users`, `posts`).
- The `routes/index.ts` file imports and consolidates the other route files.

#### 3. **Advanced Structure (Large Applications)**

For large applications, you may want to include controllers, services, and middleware, each in separate directories to promote separation of concerns.

```
src/
  ├── app.ts
  ├── controllers/
  │   ├── userController.ts
  │   └── postController.ts
  ├── routes/
  │   ├── userRoutes.ts
  │   └── postRoutes.ts
  └── services/
      ├── userService.ts
      └── postService.ts
```

**Example**:

```javascript
// src/controllers/userController.ts
import { Request, Response } from 'express';
import { getAllUsers } from '../services/userService';

export const getUsers = (req: Request, res: Response) => {
  const users = getAllUsers();
  res.json(users);
};

// src/routes/userRoutes.ts
import express from 'express';
import { getUsers } from '../controllers/userController';

const router = express.Router();
router.get('/', getUsers);

export default router;

// src/app.ts
import express from 'express';
import userRoutes from './routes/userRoutes';

const app = express();
app.use('/api/users', userRoutes);

app.listen(3000, () => console.log('Server running'));
```

This structure promotes clean code and separation of concerns. Each layer (controllers, services, routes) has its responsibility, which makes it easier to manage and scale.

---

### **Best Practices for Structuring Routes**

1. **Organize by Feature or Resource**:
    - Group related routes by resource (e.g., `/users`, `/posts`) rather than HTTP method (e.g., `/getUser`, `/createUser`). This promotes RESTful practices and cleaner route structures.

2. **Use `express.Router` for Modular Code**:
    - Define routes in separate files/modules using `express.Router()`. This keeps your code modular, easier to maintain, and allows for cleaner imports in the main application.

3. **Use Route Controllers for Logic**:
    - Avoid placing complex logic directly in route definitions. Instead, delegate business logic to controller functions, which handle processing and return the response.

4. **Use Middleware for Reusable Logic**:
    - Implement middleware functions for common functionality like authentication, logging, or request validation. Apply middleware either globally or to specific routes.

5. **Separate Concerns**:
    - Keep route definitions, controllers, services, and business logic in separate directories to maintain clear boundaries between different parts of the application.

6. **Keep the Router DRY (Don’t Repeat Yourself)**:
    - Avoid repeating common route prefixes. For example, mount resource routes under `/api` once instead of specifying `/api` for every route.

7. **Use Named Route Parameters for Dynamic Data**:
    - Use route parameters like `/users/:id` for accessing specific resources instead of relying on query parameters.

8. **Handle Errors in Routes**:
    - Ensure that routes handle errors gracefully by catching exceptions and returning meaningful error responses with appropriate status codes.

Example of error handling:
```javascript
router.get('/:id', async (req: Request, res: Response, next: NextFunction) => {
  try {
    const user = await getUserById(req.params.id);
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    next(error);
  }
});
```

---

### Summary

- **Basic Routing**: Define simple routes using HTTP methods like `GET`, `POST`, `PUT`, etc.
- **Complex Routing**: Use `express.Router()` to organize routes in a modular way for larger applications.
- **Structure**: Depending on the application's size, routes can be organized into flat, modular, or advanced structures with separation of concerns (controllers, services, middleware).
- **Best Practices**: Group routes by resource, delegate logic to controllers, use middleware for reusable logic, and handle errors properly.

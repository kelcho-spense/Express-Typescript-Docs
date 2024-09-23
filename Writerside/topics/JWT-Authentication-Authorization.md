# JWT (Authentication &amp; Authorization)
![](auth_route.png)

JWT (JSON Web Token) is a compact and self-contained way of transmitting information between parties as a JSON object. It is often used for securing APIs by authenticating users via tokens rather than sessions.

JWT consists of three parts:
1. **Header**: Contains information about the algorithm used (usually HS256).
2. **Payload**: Contains the claims (user data, permissions, etc.).
3. **Signature**: A unique string created by encoding the header and payload with a secret key.

In a typical JWT authentication setup, a client sends credentials (like email and password), the server validates them, and if successful, the server generates a JWT token, which the client can use to authenticate itself on subsequent requests.

---

### **a. Token Creation**

To implement JWT in Express, you'll need the `jsonwebtoken` library to create and verify tokens.

#### **Installation**

```bash
npm install jsonwebtoken bcryptjs
```

Here, we'll also use `bcryptjs` for hashing passwords.

#### **Example Setup: Generating a JWT Token**

1. **Create a User Authentication Flow:**

- When the user logs in or signs up, the server generates a JWT token and sends it to the client. The client stores the token (usually in localStorage or cookies) and includes it in the `Authorization` header of subsequent requests.

#### **Token Creation Example**

```javascript
import express from 'express';
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';

const app = express();
const SECRET_KEY = 'your-secret-key'; // Ideally store this in environment variables

app.use(express.json());

// Mock user database
const users: { id: number, email: string, password: string }[] = [];

// Create a JWT token for a user
const createToken = (user: { id: number; email: string }) => {
  return jwt.sign({ id: user.id, email: user.email }, SECRET_KEY, { expiresIn: '1h' });
};

// Example user registration
app.post('/register', async (req, res) => {
  const { email, password } = req.body;

  // Hash the password before storing it
  const hashedPassword = await bcrypt.hash(password, 10);
  const newUser = { id: users.length + 1, email, password: hashedPassword };
  users.push(newUser);

  const token = createToken(newUser);
  res.status(201).json({ message: 'User registered successfully', token });
});

// Example user login
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  const user = users.find(u => u.email === email);

  if (!user || !(await bcrypt.compare(password, user.password))) {
    return res.status(401).json({ message: 'Invalid email or password' });
  }

  const token = createToken(user);
  res.json({ message: 'Login successful', token });
});

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

In this setup:
- **Password Hashing**: User passwords are hashed using `bcryptjs` before being stored in the database (in this example, a simple in-memory array).
- **JWT Token**: Upon successful login or registration, a JWT token is generated using `jwt.sign()`, with the user's ID and email as the payload. The token expires in 1 hour.

#### Example Request for Token Creation (Login):

```bash
curl -X POST http://localhost:3000/login \
  -H "Content-Type: application/json" \
  -d '{"email": "john@example.com", "password": "secret"}'
```

---

### **b. User Management API (Authorization & Authentication)**

JWT Authentication involves two key processes:
1. **Authentication**: Verifying the user's identity (logging in).
2. **Authorization**: Ensuring that a user has the correct permissions to access specific resources.

#### **1. Authentication: Protecting Routes with JWT**

Once the token is created, the client sends the token in the `Authorization` header for protected routes. The server will verify the token using `jwt.verify()` to ensure that the token is valid.

#### Example of Protecting a Route:

```javascript
// Middleware to verify the JWT token
const authenticateToken = (req: express.Request, res: express.Response, next: express.NextFunction) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1]; // Bearer token

  if (!token) return res.status(401).json({ message: 'No token provided' });

  jwt.verify(token, SECRET_KEY, (err, user) => {
    if (err) return res.status(403).json({ message: 'Invalid or expired token' });

    req.user = user; // Attach user info to the request object
    next(); // Proceed to the next middleware or route handler
  });
};

// Protected route
app.get('/protected', authenticateToken, (req, res) => {
  res.json({ message: `Welcome, user with email ${req.user.email}`, user: req.user });
});
```

In this example:
- **`authenticateToken` Middleware**: Verifies the token sent by the client. If the token is valid, it adds the user data to the request object and allows access to the protected route.
- **Protected Route**: Only accessible if the client sends a valid JWT token.

#### Example Request to a Protected Route:

```bash
curl -X GET http://localhost:3000/protected \
  -H "Authorization: Bearer <your-jwt-token>"
```

If the token is valid, the server will return the protected data. Otherwise, it will respond with a `401 Unauthorized` or `403 Forbidden` status.

---

### **Authorization: Role-based Access Control (RBAC)**

You can add an additional layer of authorization by assigning roles to users (e.g., "admin", "user") and verifying their permissions when accessing certain routes.

#### Example: Role-based Authorization

1. **Adding Roles to JWT Token**:
   When generating the JWT token, include the user's role in the payload.

```javascript
const createToken = (user: { id: number; email: string; role: string }) => {
  return jwt.sign({ id: user.id, email: user.email, role: user.role }, SECRET_KEY, { expiresIn: '1h' });
};
```

2. **Role-based Middleware**:
   Create middleware to check if the authenticated user has the correct role to access a resource.

```javascript
const authorizeRole = (role: string) => {
  return (req: express.Request, res: express.Response, next: express.NextFunction) => {
    if (req.user.role !== role) {
      return res.status(403).json({ message: 'Access forbidden: Insufficient permissions' });
    }
    next();
  };
};

// Example protected route only accessible by admins
app.get('/admin', authenticateToken, authorizeRole('admin'), (req, res) => {
  res.json({ message: 'Welcome Admin!' });
});
```

In this setup:
- **`authorizeRole` Middleware**: Checks if the authenticated user has the required role to access the route.

#### Example User Login with Role:

```javascript
// Mock user data with roles
const users: { id: number, email: string, password: string, role: string }[] = [
  { id: 1, email: 'admin@example.com', password: 'hashedpassword', role: 'admin' },
  { id: 2, email: 'user@example.com', password: 'hashedpassword', role: 'user' },
];
```

---

### **Handling Token Expiration and Refresh Tokens**

JWT tokens are often set to expire after a short time (e.g., 1 hour) to reduce security risks. You can issue a new token by implementing a **refresh token** strategy.

1. **Access Token**: Short-lived token used for API requests.
2. **Refresh Token**: Longer-lived token stored securely on the client side, used to request a new access token when it expires.

#### Example of Refresh Token Flow:

```javascript
const refreshTokens: string[] = []; // In-memory store for refresh tokens

// Generate a new refresh token
const createRefreshToken = (user: { id: number; email: string }) => {
  const refreshToken = jwt.sign({ id: user.id, email: user.email }, SECRET_KEY, { expiresIn: '7d' });
  refreshTokens.push(refreshToken); // Store refresh token
  return refreshToken;
};

// Route to generate new access token using refresh token
app.post('/token', (req, res) => {
  const { token } = req.body;
  if (!token || !refreshTokens.includes(token)) {
    return res.status(403).json({ message: 'Invalid refresh token' });
  }

  jwt.verify(token, SECRET_KEY, (err, user) => {
    if (err) return res.status(403).json({ message: 'Invalid refresh token' });

    const accessToken = createToken(user);
    res.json({ accessToken });
  });
});
```

In this setup:
- **Refresh Token Store**: Refresh tokens are stored in memory (for simplicity), but you should store them securely (e.g., in a database or Redis).
- **Token Generation**: When the access token expires, the client sends the refresh token to the server to get a new access token.

---

### **Complete User Management API Example**

```javascript
import express from 'express';
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';

const app = express();
const SECRET_KEY = 'your-secret-key';
const users:

 { id: number, email: string, password: string, role: string }[] = [];

app.use(express.json());

const createToken = (user: { id: number; email: string; role: string }) => {
  return jwt.sign({ id: user.id, email: user.email, role: user.role }, SECRET_KEY, { expiresIn: '1h' });
};

const authenticateToken = (req: express.Request, res: express.Response, next: express.NextFunction) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  if (!token) return res.status(401).json({ message: 'No token provided' });

  jwt.verify(token, SECRET_KEY, (err, user) => {
    if (err) return res.status(403).json({ message: 'Invalid or expired token' });
    req.user = user;
    next();
  });
};

// Register
app.post('/register', async (req, res) => {
  const { email, password, role } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);
  const newUser = { id: users.length + 1, email, password: hashedPassword, role };
  users.push(newUser);
  const token = createToken(newUser);
  res.status(201).json({ message: 'User registered', token });
});

// Login
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  const user = users.find(u => u.email === email);
  if (!user || !(await bcrypt.compare(password, user.password))) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }
  const token = createToken(user);
  res.json({ message: 'Login successful', token });
});

// Protected route
app.get('/profile', authenticateToken, (req, res) => {
  res.json({ message: `Welcome, user with email ${req.user.email}`, user: req.user });
});

app.listen(3000, () => console.log('Server running on http://localhost:3000'));
```

---

### Summary

- **Token Creation**: JWT tokens are generated using `jsonwebtoken`. They contain user data and expire after a set time.
- **Authentication**: The token is sent in the `Authorization` header for protected routes and verified using middleware.
- **Authorization**: Role-based access control (RBAC) can be implemented to restrict certain routes based on user roles.
- **Refresh Tokens**: Used to refresh access tokens when they expire, allowing for secure and scalable authentication.

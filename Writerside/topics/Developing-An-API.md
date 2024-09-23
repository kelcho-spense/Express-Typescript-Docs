# Developing An API
Here's a step-by-step guide on how to create the API you shared from scratch using Node.js, TypeScript, Express, and various other tools and libraries that are shown in your code:

## Project Overview
The two images depict how a typical JWT (JSON Web Token) based authentication flow and Express.js middleware architecture can work in an application built using TypeScript and Express.js.
### Express.js Application Architecture

![](app flow.png)

This diagram showcases how different parts of the application interact when a request is made.

1. **HTTP Endpoint**: This represents an HTTP request sent to the server. It could be a request to a specific route like `/api/users`.
2. **Middleware**: This is the middleware layer, which typically handles things like authentication, validation, logging, or error handling. In this case, it can be processing JWT tokens, such as validating tokens before requests are forwarded.
3. **Controller**: The controller is responsible for handling the business logic associated with the request. It might invoke specific services or methods to handle the logic of a request.
4. **Service**: The service layer contains the actual business logic and operations that the controller may require. It handles more specific tasks like interacting with external services or applying core business rules.
5. **Database**: The final component is the database, which the service interacts with to fetch, store, or modify data.

In this Express.js setup, the middleware is critical for ensuring valid requests (like JWT token verification) before the controller logic is executed.

### How this Works Together in Express.js with JWT and TypeScript:
- **Express.js**: Manages the routes, handling incoming HTTP requests, and applying middleware for validation.
- **JWT**: Used to secure the routes and manage user sessions, validating tokens and issuing new ones if needed.
- **TypeScript**: Adds type safety and structure, making it easier to develop large-scale applications while ensuring the right types are enforced at compile time.

### JWT Authentication Flow
![](token flow.png)

1. **Authorized Request**: A request comes in with an access token.
2. **Is the Access Token Valid?**:
   - The system first checks if the provided access token is valid (properly signed, etc.).
   - If **No**, the system returns an unauthorized error.
   - If **Yes**, it moves to the next step.
3. **Has the Access Token Expired?**:
   - The next check is whether the token has expired.
   - If **No**, the request is processed, and it moves to the route handler.
   - If **Yes**, it checks the next condition.
4. **Is a Valid Refresh Token Included?**:
   - If the token is expired, the system checks whether a valid refresh token is included in the request.
   - If **Yes**, a new access token is issued, and the request is processed.
   - If **No**, the system returns an unauthorized error.


### **1. Setting Up the Project**

1. **Initialize the Node.js Project:**

   Open your terminal in the root directory where you want to create your project, then run:

   ```bash
   mkdir user-management-api
   cd user-management-api
   npm init -y
   ```

2. **Install Required Dependencies:**

   Install the dependencies listed in the `package.json` file from your project. These include `express`, `typescript`, `zod`, `bcrypt`, `jsonwebtoken`, `swagger-ui-express`, and others.

   Run:

   ```bash
   npm install express bcrypt cors dotenv helmet jsonwebtoken morgan multer pino pino-pretty prom-client swagger-jsdoc swagger-ui-express uuid zod
   ```

3. **Install Development Dependencies:**

   Install TypeScript and other development dependencies for type definitions and TypeScript compilation.

   ```bash
   npm install -D @types/bcrypt @types/cors @types/express @types/jsonwebtoken @types/morgan @types/mssql @types/multer @types/node @types/swagger-jsdoc @types/swagger-ui-express tsx typescript
   ```

4. **Set Up TypeScript Configuration:**

   Create a `tsconfig.json` file and configure it like this:

   ```json
   {
     "compilerOptions": {
       "target": "ES2022",
       "module": "commonjs",
       "rootDir": "./src",
       "outDir": "./dist",
       "esModuleInterop": true,
       "forceConsistentCasingInFileNames": true,
       "strict": true,
       "skipLibCheck": true
     },
     "exclude": ["node_modules", "dist"],
     "include": ["src/**/*.ts"]
   }
   ```

### **2. Project Structure**

Your project should have a folder structure similar to the one you provided:

```
project-root
│
├── src
│   ├── config
│   ├── controllers
│   ├── middleware
│   ├── models
│   ├── routes
│   ├── schemas
│   ├── services
│   ├── types
│   ├── app.ts
│   ├── server.ts
├── .env
├── .gitignore
├── package.json
├── tsconfig.json
└── README.md
```

---

### **3. Environment Variables**

Set up environment variables for sensitive data such as database credentials and JWT secrets. Create a `.env` file with the following content:

```bash
PORT=5000
DB_USER=your_db_user
DB_PASSWORD=your_db_password
DB_DATABASE=your_database
DB_SERVER=your_db_server
JWT_SECRET=your_jwt_secret
```

---

### **4. Setting Up Express**

In `src/app.ts`, set up the basic Express application:

```javascript
// src/app.ts
import express, { Request, Response } from 'express';
import helmet from 'helmet';
import cors from 'cors';
import rateLimiter from './middleware/rateLimiter';
import userRoutes from './routes/userRoutes';
import productRoutes from './routes/productRoutes';
import setupSwagger from './config/swagger';
import { metricsMiddleware, metricsRoute } from './middleware/metrics';
import morgan from 'morgan';
import { config } from './config';

const app = express();

// Security middleware
app.use(helmet());

// Enable CORS
app.use(cors());

// Body parsing middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Alternatively, using morgan
app.use(morgan('combined'));

// Rate limiting
app.use(rateLimiter);

// Metrics middleware
app.use(metricsMiddleware);

// Routes
app.use('/api', userRoutes);
app.use('/api/products', productRoutes);

// Metrics endpoint
app.get('/metrics', metricsRoute);

// Swagger documentation
setupSwagger(app, config.port);

// Health check
app.get('/health', (req: Request, res: Response) => {
  res.status(200).json({ status: 'UP' });
});

export default app;
```

---

### **5. User Routes and Controllers**

In `src/routes/userRoutes.ts`, define your user-related routes:

```javascript
import express from 'express';
import {
  register,
  login,
  getUserSessions,
  logout,
  logoutFromAllSessions,
  getAllUsersController
} from '../controllers/userController';
import { requireUser } from '../middleware/requireUser';
import { deserializeUser } from '../middleware/deserializeUser';
import { refreshToken } from '../middleware/refreshToken';
import zodValidate from '../middleware/zodValidate';
import { registerSchema, loginSchema, logoutSchema, userIdSchema } from '../schemas/userSchemas';

const userRoutes = express.Router();

userRoutes.use(deserializeUser);

/**
 * @openapi
 * tags:
 *   name: Users
 *   description: User management and authentication
 */

/**
 * @openapi
 * /users/register:
 *   post:
 *     summary: Register a new user
 *     tags: [Users]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/RegisterRequest'
 *     responses:
 *       201:
 *         description: User registered successfully
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/User'
 *       400:
 *         description: Bad request
 */
userRoutes.post('/users/register', zodValidate(registerSchema), register);

/**
 * @openapi
 * /users/login:
 *   post:
 *     summary: Login a user
 *     tags: [Users]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/LoginRequest'
 *     responses:
 *       200:
 *         description: User logged in successfully
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/LoginResponse'
 *       401:
 *         description: Unauthorized
 */
userRoutes.post('/users/login', zodValidate(loginSchema), login);

/**
 * @openapi
 * /users/sessions:
 *   get:
 *     summary: Get user sessions
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     responses:
 *       200:
 *         description: List of user sessions
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/Session'
 *       401:
 *         description: Unauthorized
 */
userRoutes.get('/users/sessions', requireUser, getUserSessions);

/**
 * @openapi
 * /users/logout:
 *   post:
 *     summary: Logout from current session
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     responses:
 *       200:
 *         description: Logged out successfully
 *       401:
 *         description: Unauthorized
 */
userRoutes.post('/users/logout', zodValidate(logoutSchema), requireUser, logout);

/**
 * @openapi
 * /users/logout-all:
 *   post:
 *     summary: Logout from all sessions
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     responses:
 *       200:
 *         description: Logged out from all sessions successfully
 *       401:
 *         description: Unauthorized
 */
userRoutes.post('/users/logout-all', zodValidate(userIdSchema), deserializeUser, requireUser, logoutFromAllSessions);

/**
 * @openapi
 * /users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     responses:
 *       200:
 *         description: List of all users
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/User'
 *       401:
 *         description: Unauthorized
 */
userRoutes.get('/users', requireUser, getAllUsersController);

/**
 * @openapi
 * /users/refresh-token:
 *   post:
 *     summary: Refresh access token
 *     tags: [Users]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/RefreshTokenRequest'
 *     responses:
 *       200:
 *         description: Tokens refreshed successfully
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/RefreshTokenResponse'
 *       401:
 *         description: Unauthorized
 */
userRoutes.post('/users/refresh-token', refreshToken);

export default userRoutes;

```

---

### **6. User Controller**

In `src/controllers/userController.ts`, implement the controller logic for each endpoint:

```javascript
// src/controllers/userController.ts
import { Request, Response } from 'express';
import { deleteAllSessions, deleteSession, getAllUsers, getSessions, loginUser, registerUser } from '../services/userService';
import { AuthRequest } from "../middleware/deserializeUser";
import { RegisterRequest } from '../models/User';

// User Registration Endpoint
export const register = async (req: Request, res: Response) => {
    const newUser: RegisterRequest = req.body;
    const profileImage = req.file?.path;

    try {
        const user = await registerUser(newUser.username, newUser.email, newUser.password, profileImage);
        res.status(201).json({ message: 'User registered successfully.', user: { id: user.id, username: user.username, email: user.email } });
    } catch (error: any) {
        res.status(500).json({ message: 'Registration failed.', error: error.message });
    }
};

// User Login Endpoint
export const login = async (req: Request, res: Response) => {
    const { email, password } = req.body;

    try {
        const { accessToken, refreshToken, user } = await loginUser(email, password);
        res.status(200).json({ message: 'Login successful', accessToken, refreshToken, user });
    } catch (error: any) {
        res.status(401).json({ message: 'Login failed', error: error.message });
    }
};

// Get all Sessions : getting all active sessions for the current user:
export const getUserSessions = async (req: AuthRequest, res: Response) => {
    if (!req.user) {
        return res.status(401).json({ message: 'Unauthorized' });
    }

    const userId = req.user.id;  // Ensure to attach the user in the deserialization process
    console.log(userId)
    try {
        const sessions = await getSessions(userId);
        res.status(200).json(sessions);
    } catch (error: any) {
        res.status(500).json({ message: 'Could not retrieve sessions', error: error.message });
    }
};

// Delete Session (logout from a single device)
export const logout = async (req: Request, res: Response) => {
    const { refreshToken } = req.body; // The refresh token to invalidate

    try {
        await deleteSession(refreshToken);
        res.status(200).json({ message: 'Logged out successfully' });
    } catch (error) {
        res.status(500).json({ message: 'Failed to log out', error });
    }
};

// Delete All Session (logout from all devices)
export const logoutFromAllSessions = async (req: AuthRequest, res: Response) => {
    const userId = (req as any).user.id;

    try {
        await deleteAllSessions(userId);
        res.status(200).json({ message: 'Logged out from all devices' });
    } catch (error) {
        res.status(500).json({ message: 'Failed to log out from all devices', error });
    }
};

// Get All Users Endpoint
export const getAllUsersController = async (req: Request, res: Response) => {
    try {
        const users = await getAllUsers();
        res.status(200).json({ users });
    } catch (error) {
        res.status(500).json({ message: 'Failed to retrieve users.', error });
    }
};

```

---

### **7. User Service**

In `src/services/userService.ts`, create the actual logic for handling user data, sessions, and authentication:

```javascript
// src/services/userService.ts
import db from '../config/db';
import { User, LoginResponse } from '../models/User';
import bcrypt from 'bcrypt';
import { v4 as uuidv4 } from 'uuid';
import jwt from 'jsonwebtoken';
import { config } from '../config';


//register user
export const registerUser = async (username: string, email: string, password: string, profileImage?: string): Promise<User> => {
    const hashedPassword = await bcrypt.hash(password, 10);
    const user: User = {
        id: uuidv4(),
        username,
        email,
        password: hashedPassword,
        profileImage,
        createdAt: new Date(),
        updatedAt: new Date(),
    };

    await db.poolConnect;
    const request = db.pool.request();
    await request
        .input('id', db.sql.VarChar, user.id)
        .input('username', db.sql.VarChar, user.username)
        .input('email', db.sql.VarChar, user.email)
        .input('password', db.sql.VarChar, user.password)
        .input('profileImage', db.sql.VarChar, user.profileImage || null)
        .input('createdAt', db.sql.DateTime, user.createdAt)
        .input('updatedAt', db.sql.DateTime, user.updatedAt)
        .query(`
            INSERT INTO Users (id, username, email, password, profileImage, createdAt, updatedAt)
            VALUES (@id, @username, @email, @password, @profileImage, @createdAt, @updatedAt)
        `);
    return user;
};

// login user
export const loginUser = async (email: string, password: string): Promise<LoginResponse> => {
    const user = await getUserByEmail(email);
   
    if (!user) throw new Error('User not Found')

    const isPasswordValid = await bcrypt.compare(password, user.password);

    if (!isPasswordValid) throw new Error('Invalid credentials');

    const accessToken = jwt.sign({ id: user.id, email: user.email }, config.jwtSecret, { expiresIn: '15m' });
    const refreshToken = jwt.sign({ id: user.id, email: user.email }, config.jwtSecret, { expiresIn: '7d' });

    await storeRefreshToken(user.id, refreshToken);

    return {
        accessToken,
        refreshToken,
        user: {
            id: user.id,
            username: user.username,
            email: user.email,
        },
    };
};

// get user via email
export const getUserByEmail = async (email: string): Promise<User> => {
    await db.poolConnect;
    const request = db.pool.request();
    const result = await request
        .input('email', db.sql.VarChar, email)
        .query('SELECT * FROM users WHERE email = @email')

    return result.recordset[0];
}
// read all users
export const getAllUsers = async (): Promise<Partial<User>[]> => {
    await db.poolConnect;
    const request = db.pool.request();
    const result = await request.query('SELECT id, username, email, profileImage, createdAt FROM Users');

    const users: Partial<User>[] = result.recordset;

    return users;
};

//update user 
export const updateUser = async (id: string, updatedFields: Partial<User>): Promise<User> => {
    const { username, email, profileImage, password } = updatedFields;

    await db.poolConnect;
    const request = db.pool.request();

    if (password) {
        updatedFields.password = await bcrypt.hash(password, 10);
    }

    await request
        .input('id', db.sql.VarChar, id)
        .input('username', db.sql.VarChar, username || null)
        .input('email', db.sql.VarChar, email || null)
        .input('profileImage', db.sql.VarChar, profileImage || null)
        .input('password', db.sql.VarChar, updatedFields.password || null)
        .input('updatedAt', db.sql.DateTime, new Date())
        .query(`
        UPDATE Users
        SET 
          username = ISNULL(@username, username),
          email = ISNULL(@email, email),
          password = ISNULL(@password, password),
          profileImage = ISNULL(@profileImage, profileImage),
          updatedAt = @updatedAt
        WHERE id = @id
      `);

    const updatedUser = await request
        .input('id', db.sql.VarChar, id)
        .query('SELECT * FROM Users WHERE id = @id');

    return updatedUser.recordset[0];
};

// delete user 
export const deleteUser = async (id: string): Promise<void> => {
    await db.poolConnect;
    const request = db.pool.request();

    await request.input('id', db.sql.VarChar, id).query('DELETE FROM Users WHERE id = @id');
};

// active sessions
export const getSessions = async (userId: string): Promise<{ refreshToken: string, createdAt: Date, updatedAt: Date }[]> => {
    await db.poolConnect;
    const request = db.pool.request();

    const result = await request
        .input('userId', db.sql.VarChar, userId)
        .query('SELECT refreshToken, createdAt, updatedAt FROM Sessions WHERE userId = @userId');

    return result.recordset;
};

// Refresh Token
export const refreshAccessToken = async (refreshToken: string): Promise<string> => {
    try {
        // Verify the refresh token
        const decoded = jwt.verify(refreshToken, config.jwtSecret) as { id: string, email: string };
        const { id, email } = decoded;

        await db.poolConnect;
        const request = db.pool.request();

        // Check if the refresh token exists in the sessions table
        const result = await request
            .input('refreshToken', db.sql.VarChar, refreshToken)
            .query('SELECT * FROM Sessions WHERE refreshToken = @refreshToken');

        if (!result.recordset.length) {
            throw new Error('Invalid refresh token');
        }

        // If valid, issue a new access token
        const newAccessToken = jwt.sign({ id, email }, config.jwtSecret, { expiresIn: '15m' });

        return newAccessToken;
    } catch (error: any) {
        throw new Error('Could not refresh access token: ' + error.message);
    }
};

// delete session(logout)
export const deleteSession = async (refreshToken: string): Promise<void> => {
    await db.poolConnect;
    const request = db.pool.request();

    // Remove the refresh token from the Sessions table
    await request
        .input('refreshToken', db.sql.VarChar, refreshToken)
        .query('DELETE FROM Sessions WHERE refreshToken = @refreshToken');
};

/** Delete All Sessions (Logout from all devices)
 * This is a useful service to log a user out from all sessions (all devices).
 * */
export const deleteAllSessions = async (userId: string): Promise<void> => {
    await db.poolConnect;
    const request = db.pool.request();

    // Remove all sessions for the user
    await request
        .input('userId', db.sql.VarChar, userId)
        .query('DELETE FROM Sessions WHERE userId = @userId');
};

// Helper to store refresh token
const storeRefreshToken = async (userId: string, refreshToken: string) => {
    await db.poolConnect;
    const request = db.pool.request();
    await request
        .input('userId', db.sql.VarChar, userId)
        .input('refreshToken', db.sql.VarChar, refreshToken)
        .query(`
        INSERT INTO Sessions (userId, refreshToken, createdAt, updatedAt)
        VALUES (@userId, @refreshToken, GETDATE(), GETDATE())
    `);
};

```

---

### **8. Swagger Documentation**

You can generate Swagger API documentation using `swagger-jsdoc`. Here's an example `swagger.ts` configuration:

```javascript
import { Express, Request, Response } from "express";
import swaggerJsdoc from "swagger-jsdoc";
import swaggerUi from "swagger-ui-express";
import logger from "../middleware/logger";

const options: swaggerJsdoc.Options = {
    definition: {
        openapi: "3.0.0",
        info: {
            title: "User Management API Docs",
            version: "1.0",
            description: "API documentation for the User Management system.",
            license: {
                name: 'MIT License',
                url: "https://github.com/kelcho-spense/Express-Typescript-Course/blob/main/LICENSE"
            }
        },
        servers: [
            {
                url: "http://localhost:8000/api", // Set your base URL here
                description: "Local server"
            }
        ],
        components: {
            securitySchemes: {
                bearerAuth: {
                    type: "http",
                    scheme: "bearer",
                    bearerFormat: "JWT",
                },
            },
        },
        security: [
            {
                bearerAuth: [],
            },
        ],
    },
    apis: ["./src/routes/*.ts", "./src/models/*.ts"],
};

const swaggerSpec = swaggerJsdoc(options);

function swaggerDocs(app: Express, port: number) {
    // Swagger page
    app.use("/docs", swaggerUi.serve, swaggerUi.setup(swaggerSpec));

    // Docs in JSON format
    app.get("/docs.json", (req: Request, res: Response) => {
        res.setHeader("Content-Type", "application/json");
        res.send(swaggerSpec);
    });

    logger.info(`Docs available at http://localhost:${port}/docs`);
}

export default swaggerDocs;

```
### **9. middlewares**

- `deserializeUser.ts`
```javascript
// src/middleware/deserializeUser.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import { config } from '../config';
import { User } from '../models/User';
import { refreshAccessToken } from '../services/userService';

interface AuthRequest extends Request {
    user?: User;
}

export const deserializeUser = async (req: AuthRequest, res: Response, next: NextFunction) => {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) return res.status(401).json({ message: 'Unauthorized' });

    try {
        const decoded = jwt.verify(token, config.jwtSecret) as User;
        req.user = decoded;
        next();
    } catch (error: any) {
        // Detect token expiration
        if (error.name === 'TokenExpiredError') {
            const refreshToken = req.headers['x-refresh-token'];

            if (!refreshToken) {
                return res.status(401).json({ message: 'Token expired and no refresh token provided' });
            }

            // Attempt to refresh the token
            try {
                const newAccessToken = await refreshAccessToken(refreshToken as string);
                res.setHeader('x-access-token', newAccessToken);

                // Reattempt the original request after issuing the new token
                const decoded = jwt.verify(newAccessToken, config.jwtSecret) as User;
                req.user = decoded;
                next();
            } catch (refreshError) {
                return res.status(403).json({ message: 'Could not refresh token' });
            }
        } else {
            return res.status(401).json({ message: 'Invalid token' });
        }
    }
};

export type { AuthRequest };

```
- `logger.ts`
```javascript
import pino from 'pino'

const logger = pino({
    base: {
        pid: false
    },
    transport: {
        target: 'pino-pretty',
        options: {
            colorize: true
        }
    }
})

export default logger;

/**
 * Pino is a fast, lightweight, and highly configurable logger built on
 * V8's built-in JSON.stringify() function. It's a low-overhead alternative
 * to other popular loggers like Morgan and Winston.
 *
 * The "base" option removes the process ID from the log output, which is
 * not necessary for our use case.
 *
 * The "transport" option is used to customize the log output. In this case,
 * we're using the "pino-pretty" transport, which formats the log output in
 * a human-readable format. The "colorize" option is set to true, which adds
 * color to the log output.
 */
```

- `metrics.ts`
```javascript
// src/middleware/metrics.ts
import { Request, Response, NextFunction } from 'express';
import client from 'prom-client';

const collectDefaultMetrics = client.collectDefaultMetrics;
collectDefaultMetrics();

const httpRequestDurationMicroseconds = new client.Histogram({
    name: 'http_request_duration_ms',
    help: 'Duration of HTTP requests in ms',
    labelNames: ['method', 'route', 'status_code'],
    buckets: [50, 100, 300, 500, 1000, 3000],
});

const metricsMiddleware = (req: Request, res: Response, next: NextFunction) => {
    const start = Date.now();

    res.on('finish', () => {
        const duration = Date.now() - start;
        httpRequestDurationMicroseconds
            .labels(req.method, req.route ? req.route.path : req.path, res.statusCode.toString())
            .observe(duration);
    });

    next();
};

const metricsRoute = async (req: Request, res: Response) => {
    res.set('Content-Type', client.register.contentType);
    res.end(await client.register.metrics());
};

export { metricsMiddleware, metricsRoute };
```

- `rateLimiter.ts`
```javascript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    standardHeaders: 'draft-7', // draft-6: `RateLimit-*` headers; draft-7: combined `RateLimit` header
    legacyHeaders: false, // Disable the `X-RateLimit-*` headers.
    message: 'Too many requests from this IP, please try again later.',
});
export default limiter;

/**
 * The `limiter` is a middleware that limits the number of requests
 * that can be made from a single IP address within a given time
 * window (15 minutes in this case). If the limit is exceeded, the
 * middleware will return a 429 status code and the message
 * "Too many requests from this IP, please try again later.".
 *
 * The `standardHeaders` option is set to `'draft-7'` which means
 * the middleware will use the combined `RateLimit` header. The
 * `legacyHeaders` option is set to `false` which means the
 * middleware will not use the `X-RateLimit-*` headers.
 *
 * The `windowMs` option is set to `15 * 60 * 1000` which means
 * the middleware will count the number of requests made within
 * the last 15 minutes. The `max` option is set to `100` which
 * means the middleware will allow up to 100 requests from a
 * single IP address within the given time window.
 *
 * The `message` option is set to the string "Too many requests
 * from this IP, please try again later." which is the message
 * that will be returned when the limit is exceeded.
 */

```

- `refreshToken.ts`
```javascript
import { config } from '../config';
import { Request, Response } from 'express';
import jwt from 'jsonwebtoken';

export const refreshToken = async (req: Request, res: Response) => {
    const { refreshToken } = req.body;
    if (!refreshToken) return res.status(403).json({ message: 'Token expired, login required' });

    try {
        const decoded = jwt.verify(refreshToken, config.jwtSecret) as { id: string; email: string };
        const newAccessToken = jwt.sign({ id: decoded.id, email: decoded.email }, config.jwtSecret, { expiresIn: '15m' });
        res.setHeader('Authorization', `Bearer ${newAccessToken}`);
        return res.json({ accessToken: newAccessToken });
    } catch (error) {
        return res.status(403).json({ message: 'Invalid refresh token' });
    }
};

```

- `requireUser.ts`
```javascript
import { Response, NextFunction } from 'express';
import { AuthRequest } from './deserializeUser'

export const requireUser = (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!req.user) return res.status(401).json({ message: 'Unauthorized: No user found' });
    next();
};
```

- `zodValidate.ts`
```javascript
import { ZodSchema } from 'zod';
import { Request, Response, NextFunction } from 'express';

const zodValidate = (schema: ZodSchema<any>) => (req: Request, res: Response, next: NextFunction) => {
    try {
        schema.parse({
            body: req.body,
            query: req.query,
            params: req.params,
        });
        next();
    } catch (error: any) {
        res.status(400).json({
            error: error.errors,
        });
    }
};

export default zodValidate;

/**This is a middleware function named validate that takes a Zod schema as an argument.
 * It validates incoming HTTP requests against the provided schema. If the request is valid,
 * it calls the next middleware function (next()).
 * If the request is invalid, it returns a 400 error response with the validation errors. 
 *  */
```


### **10. Running the Project**

Once everything is set up:

1. Run TypeScript compilation:

   ```bash
   npx tsc
   ```

2. Run the development server using `tsx`:

   ```bash
   npm run dev
   ```

You should now be able to test your API on `http://localhost:5000` and access Swagger documentation at `http://localhost:5000/docs`.

---


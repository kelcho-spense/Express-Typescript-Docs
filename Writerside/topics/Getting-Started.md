# Getting Started

### a. **Create an Express App**

1. **Initialize a Node project:**
   ```bash
   npm init -y
   ```

2. **Install Express and TypeScript:**
   ```bash
   npm install express
   npm install --save-dev typescript @types/express tsx
   ```

3. **Set up TypeScript:**
   Create a `tsconfig.json`: and add
```json
{
   "compilerOptions": {
      "target": "ESNext",   // Target ES6 or later to support modern JavaScript features
      "module": "Node16",    // Use node modules version 16
      "rootDir": "./src",       // Source files location
      "outDir": "./dist",          // Output directory for compiled files
      "esModuleInterop": true,        // Enables support for `import` in CommonJS
      "strict": true,                     // Enables strict type-checking options
      "skipLibCheck": true,             // Skips type checking of declaration files
      "moduleResolution": "Node16",        // For resolving node modules
      "resolveJsonModule": true,              // Enable importing `.json` files
      "allowSyntheticDefaultImports": true,    // Allow default imports from modules
      "forceConsistentCasingInFileNames": true   // Ensure consistent casing in file names
   },
   "include": ["src/**/*.ts"],          // Include all TypeScript files in the `src` folder
   "exclude": ["node_modules"]          // Exclude `node_modules`
}
```

4. **Create a basic Express server:**
   In `src/server.ts`:
   ```javascript
   import express, { Request, Response } from 'express';

   const app = express();
   const port = 8000;

   app.use(express.json()); // Middleware to parse JSON requests

   app.get('/', (req: Request, res: Response) => {
     res.send('Hello, Express with TypeScript!');
   });

   app.listen(port, () => {
     console.log(`Server is running on http://localhost:${port}`);
   });
   ```

5. **Run the server:**
   Add this to `package.json` scripts:
   ```json
   "scripts": {
     "dev": "tsx watch src/app.ts",
      "start": "node dist/index.js"
   }
   ```
6. Test the app, install the REST Client vs code extension. create `app.http` and add
```cURL
### HTTP/1.1

GET http://localhost:8000/ 
```
   Start the server :
```bash
   npm start
```
---

### b. **Frameworks Built on Express**
Several frameworks extend Express to provide additional features and structure:

- **NestJS**: A framework built with TypeScript, offering a modular architecture similar to Angular. It enhances Express with dependency injection, strong typing, and structured patterns for scalable applications.

- **Loopback**: Extends Express to build APIs rapidly, with built-in connectors for databases, auto-generated REST APIs, and strong TypeScript support.

- **Sails.js**: Another Express-based framework, which provides a MVC architecture and supports real-time features out of the box.

---

### c. **Proper Express Folder Structure for REST API**

A well-structured folder organization is essential for scalability and maintainability. Here is an ideal folder structure:

```
src/
  ├── controllers/        # Handles the business logic
  ├── routes/             # Defines routes for each resource
  ├── models/             # Database models or data definitions
  ├── services/           # Handles data processing and calls to external APIs
  ├── middleware/         # Custom middleware logic (auth, logging, etc.)
  ├── config/             # Environment variables, configuration settings
  ├── utils/              # Helper functions and utility classes
  ├── types/              # Custom TypeScript types/interfaces
  ├── app.ts              # Express application initialization
  └── server.ts           # Server startup file
```

### Example Breakdown:

#### **i. Model, Routes, Controllers**

1. **Models**: Defines the structure of your database schema or data interfaces.

```javascript
// src/models/User.ts
export interface User {
  id: string;
  name: string;
  email: string;
}
```

2. **Controllers**: Implements the core business logic, processing requests and sending appropriate responses.

```javascript
// src/controllers/userController.ts
import { Request, Response } from 'express';

export const getUsers = (req: Request, res: Response): void => {
  const users = [{ id: '1', name: 'John Doe', email: 'john@example.com' }];
  res.json(users);
};
```

3. **Routes**: Maps routes to corresponding controllers.

```javascript
// src/routes/userRoutes.ts
import express from 'express';
import { getUsers } from '../controllers/userController';

const router = express.Router();

router.get('/users', getUsers);

export default router;
```

4. **Middleware**: Add custom logic between request and response.

```javascript
// src/middleware/authMiddleware.ts
import { Request, Response, NextFunction } from 'express';

export const authMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const authToken = req.headers['authorization'];

  if (authToken) {
    // Do some token verification here
    next(); // Proceed to the next middleware/controller
  } else {
    res.status(401).json({ message: 'Unauthorized' });
  }
};
```

5. **Server and App Initialization:**
```javascript
// src/app.ts
import express from 'express';
import userRoutes from './routes/userRoutes';

const app = express();
app.use(express.json());

app.use('/api', userRoutes);

export default app;

// src/server.ts
import app from './app';

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

---

#### **ii. Model, Routes, Controllers, Services**

1. **Services**: Contains logic for database operations, external API calls, or complex data manipulation. Services abstract the business logic and keep the controller lean.

```javascript
// src/services/userService.ts
import { User } from '../models/User';

export const getAllUsers = async (): Promise<User[]> => {
  // Simulate fetching data from a database
  return [{ id: '1', name: 'John Doe', email: 'john@example.com' }];
};
```

2. **Controller using a Service**:

```javascript
// src/controllers/userController.ts
import { Request, Response } from 'express';
import { getAllUsers } from '../services/userService';

export const getUsers = async (req: Request, res: Response) => {
  try {
    const users = await getAllUsers();
    res.status(200).json(users);
  } catch (error) {
    res.status(500).json({ message: 'Internal Server Error' });
  }
};
```
---


# Schema Validation (Zod)

**Zod** is a TypeScript-first schema declaration and validation library. It provides a simple and declarative way to define and validate data schemas, which is especially useful when working with user input, API requests, and complex data models in Express.js applications.

Compared to other validation libraries (like Joi or Yup), Zod is tightly integrated with TypeScript, providing full type inference and static type safety, which can help prevent runtime errors and improve developer experience.

#### **Why Zod?**
- **TypeScript-first**: Zod automatically infers TypeScript types from your schema, reducing redundancy.
- **Declarative**: You define schemas once and use them for both validation and type-checking.
- **Simple API**: Zod provides a clean and simple interface for creating schemas and validating data.

---

### **Installation**

To use Zod, first install it via npm:

```bash
npm install zod
```

---

### **Basic Usage of Zod**

Zod allows you to define schemas for various data types and validate data against those schemas. Here’s an example of how to create a simple schema and validate an object:

#### Example of Basic Validation:

```javascript
import { z } from 'zod';

const userSchema = z.object({
  id: z.number(),
  name: z.string().min(3, 'Name must be at least 3 characters long'),
  email: z.string().email(),
  age: z.number().positive().int().optional(), // Optional field
});

const userData = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
};

try {
  userSchema.parse(userData); // Validation successful
  console.log('User data is valid');
} catch (err) {
  console.log(err.errors); // Print validation errors
}
```

In this example:
- **`z.number()`**: Defines a number type.
- **`z.string()`**: Defines a string type, with additional `.min()` for length validation and `.email()` for email format validation.
- **`z.number().positive()`**: Ensures the number is positive.
- **`.optional()`**: Specifies that the field is optional.

---

### **Zod with Express**

You can integrate Zod with Express for request validation, ensuring incoming data is valid before processing it in your application. Below are examples of how to use Zod for validating incoming data in routes.

#### Example: Validating Request Body in an Express Route

```javascript
import express from 'express';
import { z } from 'zod';

const app = express();
app.use(express.json());

const userSchema = z.object({
  name: z.string().min(3),
  email: z.string().email(),
  age: z.number().int().positive(),
});

app.post('/users', (req, res) => {
  try {
    // Validate request body against userSchema
    const userData = userSchema.parse(req.body);
    res.status(200).json({ message: 'User created successfully', data: userData });
  } catch (err) {
    res.status(400).json({ message: 'Validation failed', errors: err.errors });
  }
});

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

In this example:
- **`userSchema.parse(req.body)`**: Validates the incoming request body against the defined schema. If validation passes, the user data is processed; if it fails, an error is returned.
- **Error Handling**: If Zod’s `parse()` method throws an error, it will contain an array of validation errors, which can be sent back to the client.

#### Example Request:
```bash
curl -X POST http://localhost:3000/users -H "Content-Type: application/json" -d '{"name": "Jo", "email": "invalidemail", "age": -5}'
```

This would return:
```json
{
  "message": "Validation failed",
  "errors": [
    { "message": "String must contain at least 3 character(s)", "path": ["name"] },
    { "message": "Invalid email", "path": ["email"] },
    { "message": "Number must be greater than 0", "path": ["age"] }
  ]
}
```

---

### **Complex Zod Schemas**

Zod allows you to build more complex schemas that include nested objects, arrays, enums, and more.

#### Nested Objects:

You can define schemas for nested objects and validate them.

```javascript
const addressSchema = z.object({
  street: z.string(),
  city: z.string(),
  zipCode: z.string().regex(/^\d{5}$/, 'Invalid zip code format'),
});

const userSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  address: addressSchema, // Nested object
});

const userData = {
  id: 1,
  name: 'Jane Doe',
  email: 'jane@example.com',
  address: {
    street: '123 Main St',
    city: 'New York',
    zipCode: '10001',
  },
};

userSchema.parse(userData); // Validation passes
```

#### Arrays:

Zod allows you to validate arrays and ensure that every element in the array conforms to a schema.

```javascript
const postSchema = z.object({
  id: z.number(),
  title: z.string(),
});

const blogSchema = z.object({
  name: z.string(),
  posts: z.array(postSchema), // Array of posts
});

const blogData = {
  name: 'My Blog',
  posts: [
    { id: 1, title: 'First Post' },
    { id: 2, title: 'Second Post' },
  ],
};

blogSchema.parse(blogData); // Validation passes
```

#### Enums:

You can define a set of valid values using Zod’s `z.enum()` method.

```javascript
const userTypeSchema = z.enum(['admin', 'user', 'guest']);

const userData = {
  id: 1,
  type: 'admin',
};

userTypeSchema.parse(userData.type); // Validation passes
```

---

### **Asynchronous Validation**

In some cases, validation might need to be asynchronous (e.g., validating data from an external API or checking if a record exists in a database). Zod allows you to write async validations with `refine`.

#### Example: Async Validation with `refine`

```javascript
const userSchema = z.object({
  email: z.string().email(),
}).refine(async (data) => {
  // Simulate checking if email exists in the database
  const emailExists = await fakeDatabaseCheck(data.email);
  return !emailExists;
}, {
  message: 'Email is already in use',
  path: ['email'],
});

async function fakeDatabaseCheck(email: string): Promise<boolean> {
  const existingEmails = ['existing@example.com'];
  return existingEmails.includes(email);
}

userSchema.parseAsync({ email: 'new@example.com' })
  .then(() => console.log('Valid email'))
  .catch(err => console.log('Validation failed', err.errors));
```

In this example:
- **`refine()`**: Adds a custom validation rule. Here, it checks if the email exists in a simulated database.
- **`parseAsync()`**: Used for asynchronous schema validation.

---

### **Custom Error Messages**

You can customize error messages to provide more meaningful feedback to the user.

```javascript
const userSchema = z.object({
  name: z.string().min(3, { message: 'Name must be at least 3 characters long' }),
  email: z.string().email({ message: 'Invalid email format' }),
});

try {
  userSchema.parse({ name: 'Jo', email: 'invalidemail' });
} catch (err) {
  console.log(err.errors);
}
```

Output:
```json
[
  { "message": "Name must be at least 3 characters long", "path": ["name"] },
  { "message": "Invalid email format", "path": ["email"] }
]
```

---

### **Integration with Express Middleware**

You can create reusable validation middleware using Zod for Express. This middleware can be used to validate request bodies, query parameters, or route parameters.

#### Example: Zod Validation Middleware

```javascript
import { Request, Response, NextFunction } from 'express';
import { z } from 'zod';

const validate = (schema: z.ZodSchema) => (req: Request, res: Response, next: NextFunction) => {
  try {
    schema.parse(req.body);
    next();
  } catch (err) {
    res.status(400).json({ errors: err.errors });
  }
};

const userSchema = z.object({
  name: z.string().min(3),
  email: z.string().email(),
});

app.post('/users', validate(userSchema), (req, res) => {
  res.status(200).json({ message: 'User created successfully' });
});
```

In this example:
- **`validate`**: Middleware that validates the request body against the provided schema and returns errors if validation fails.

---

### Summary

- **Zod** is a TypeScript-first validation library that provides type-safe schemas for data validation.
- **Basic Usage**: Zod schemas can be defined to validate simple types (strings, numbers) and more complex structures (nested objects, arrays, enums).
- **Express Integration**: Zod can be integrated into Express routes to validate incoming request data. You can create reusable validation middleware for request bodies, query parameters, or route parameters.
- **Asynchronous Validation

**: Use `refine()` for custom async validation logic (e.g., checking database entries).
- **Custom Error Messages**: You can provide more meaningful error messages using Zod’s error message options.
- **Middleware**: Zod can be wrapped into a middleware for cleaner and reusable validation handling in Express.

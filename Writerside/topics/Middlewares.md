# Middlewares

Express middlewares are functions that execute during the request-response cycle. They can perform various tasks such as parsing request bodies, logging, handling errors, rate-limiting, and more. Middlewares can be built-in, custom, or external libraries that you plug into your application.

#### Middleware Types:
- **Built-in**: Included in the Express framework.
- **External**: Additional middleware provided by third-party libraries.

---

### **a. Inbuilt Middlewares**

Express has several built-in middlewares that handle common tasks like parsing request bodies. Two of the most commonly used built-in middlewares are `express.json()` and `express.urlencoded()`.

#### **i. `express.json()`**

The `express.json()` middleware is used to parse incoming JSON payloads. It makes the body of the request available in `req.body`. This is commonly used in APIs where JSON data is submitted in POST or PUT requests.

```javascript
import express from 'express';
const app = express();

app.use(express.json()); // Parses JSON requests

app.post('/data', (req, res) => {
  console.log(req.body); // Access the JSON data sent in the request
  res.send('Data received');
});

app.listen(3000, () => {
  console.log('Server is running on http://localhost:3000');
});
```

#### **ii. `express.urlencoded({ extended: true })`**

This middleware is used to parse URL-encoded payloads, which are usually submitted by HTML forms. It makes the form data available in `req.body`.

- **`extended: true`**: Allows for richer objects and arrays to be encoded.
- **`extended: false`**: Uses the classic encoding of the query-string library.

```javascript
app.use(express.urlencoded({ extended: true })); // Parses URL-encoded requests

app.post('/submit', (req, res) => {
  console.log(req.body); // Access the form data submitted
  res.send('Form submitted');
});
```

---

### **b. External Middlewares**

Express has a large ecosystem of third-party middlewares. Here are some useful external middlewares you can integrate into your Express application:

---

### **i. `express-rate-limit`**

The `express-rate-limit` middleware helps you limit the number of requests a client can make to your API in a given time window. This is useful for preventing abuse (e.g., DDoS attacks) by limiting the frequency of requests.

#### Installation:

```bash
npm install express-rate-limit
```

#### Example Usage:

```javascript
import express from 'express';
import rateLimit from 'express-rate-limit';

const app = express();

// Rate limit middleware
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests, please try again later.'
});

app.use('/api', limiter); // Apply the rate limit to all API routes

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

---

### **ii. `pino` (with `pino-pretty`)**

`Pino` is a fast and lightweight logging library for Node.js. It's a great alternative to more verbose logging systems like `winston`. When combined with `pino-pretty`, the logs are formatted in a more human-readable way for development.

#### Installation:

```bash
npm install pino pino-pretty
```

#### Example Usage:

```javascript
import express from 'express';
import pino from 'pino';
import pinoHttp from 'pino-http';

const logger = pino({ prettyPrint: true });
const app = express();

app.use(pinoHttp({ logger })); // Integrate Pino with Express

app.get('/', (req, res) => {
  req.log.info('Request received'); // Example log
  res.send('Hello, Pino logger!');
});

app.listen(3000, () => {
  logger.info('Server running on http://localhost:3000');
});
```

With `pino-pretty`, the logs are displayed in a cleaner, human-friendly format.

---

### **iii. `prom-client`**

The `prom-client` middleware helps integrate **Prometheus** metrics into your Express application. This allows you to track various performance metrics like response times, request counts, and more.

#### Installation:

```bash
npm install prom-client
```

#### Example Usage:

```javascript
import express from 'express';
import client from 'prom-client';

const app = express();

// Create metrics for response time
const restResponseTimeHistogram = new client.Histogram({
  name: 'rest_response_time_duration_seconds',
  help: 'REST API response time in seconds',
  labelNames: ['method', 'route', 'status_code'],
});

const databaseResponseTimeHistogram = new client.Histogram({
  name: 'database_response_time_duration_seconds',
  help: 'Database response time in seconds',
  labelNames: ['operation'],
});

// Middleware to start timer for response time
app.use((req, res, next) => {
  const end = restResponseTimeHistogram.startTimer();
  res.on('finish', () => {
    end({ method: req.method, route: req.route?.path, status_code: res.statusCode });
  });
  next();
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

#### Key Metrics:
1. **`restResponseTimeHistogram`**: Tracks the REST API's response time.
2. **`databaseResponseTimeHistogram`**: Tracks response times for database queries (you would call this in your database service).

---

### **iv. `morgan`**

`morgan` is a popular HTTP request logger middleware. It provides detailed logs about requests made to your server, which can be very useful for debugging and monitoring.

#### Installation:

```bash
npm install morgan
```

#### Example Usage:

##### 1. Basic Logging to Console:

```javascript
import express from 'express';
import morgan from 'morgan';

const app = express();

// Use morgan for logging HTTP requests
app.use(morgan('dev')); // 'dev' is a pre-defined format

app.get('/', (req, res) => {
  res.send('Hello, Morgan Logger!');
});

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

##### 2. Logging with `pino` (using `pino-pretty`):

You can replace `morgan` with `pino` for logging if you need high performance logging.

```javascript
import express from 'express';
import pino from 'pino';
import pinoHttp from 'pino-http';

const logger = pino({ prettyPrint: true });

const app = express();
app.use(pinoHttp({ logger }));

app.get('/', (req, res) => {
  req.log.info('Logging request'); // Log using Pino
  res.send('Hello, Pino and Express!');
});

app.listen(3000, () => {
  logger.info('Server is running');
});
```

##### 3. Writing Logs to a File:

You can also configure `morgan` to log requests to a file instead of the console.

```javascript
import express from 'express';
import morgan from 'morgan';
import fs from 'fs';
import path from 'path';

const app = express();

// Create a write stream for logging into a file
const accessLogStream = fs.createWriteStream(path.join(__dirname, 'access.log'), { flags: 'a' });

// Use morgan for logging into a file
app.use(morgan('combined', { stream: accessLogStream }));

app.get('/', (req, res) => {
  res.send('Logging requests to a file');
});

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

---

### **v. Swagger (`swagger-ui-express` & `swagger-jsdoc`)**

Swagger is a set of tools to help document and visualize REST APIs. You can use `swagger-ui-express` to serve a Swagger UI and `swagger-jsdoc` to generate the OpenAPI specification (Swagger docs) from comments in your code.

#### Installation:

```bash
npm install swagger-ui-express swagger-jsdoc
```

#### Example Setup:

```javascript
import express from 'express';
import swaggerUi from 'swagger-ui-express';
import swaggerJsdoc from 'swagger-jsdoc';

const app = express();

const swaggerOptions = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'Express API Documentation',
      version: '1.0.0',
      description: 'A sample API'
    },
  },
  apis: ['./src/routes/*.ts'], // Path to your API route files
};

const swaggerDocs = swaggerJsdoc(swaggerOptions);

// Serve Swagger UI
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocs));

// Example API route
app.get('/api', (req, res) => {
  res.send('API works!');
});

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

#### Adding Swagger Documentation to Routes:

You can add Swagger documentation directly in your routes using comments.

```javascript
/**
 * @swagger
 * /api:
 *   get:
 *     description: Get API status
 *    

 responses:
 *       200:
 *         description: Success
 */
app.get('/api', (req, res) => {
  res.send('API works!');
});
```

In this setup:
- **`swagger-ui-express`**: Provides a UI to view and interact with the API documentation.
- **`swagger-jsdoc`**: Parses comments in your code and generates an OpenAPI specification.

---

### Summary

- **Built-in Middlewares**:
    - `express.json()`: Parses incoming JSON requests.
    - `express.urlencoded({ extended: true })`: Parses URL-encoded data (from HTML forms).

- **External Middlewares**:
    - **`express-rate-limit`**: Prevents abuse by limiting request rates.
    - **`pino`**: High-performance logging library (with `pino-pretty` for human-readable logs).
    - **`prom-client`**: Provides Prometheus metrics to monitor API performance.
    - **`morgan`**: Simple HTTP request logging (to the console or a file).
    - **`swagger-ui-express` & `swagger-jsdoc`**: Automatically generate and serve API documentation via Swagger.

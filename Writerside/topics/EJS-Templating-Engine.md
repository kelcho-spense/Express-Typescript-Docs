# EJS Templating Engine

This guide provides a comprehensive introduction to using the EJS (Embedded JavaScript) templating engine within an Express.js application written in TypeScript. We'll explore the basic syntax of EJS, demonstrate its use in email templating, and cover other common applications such as rendering dynamic web pages.


## Introduction to Templating Engines and EJS

### What is a Templating Engine?

A templating engine allows you to generate dynamic HTML (or other text-based formats) by embedding code within templates. This enables the creation of dynamic content based on data, user input, or other variables. Templating engines are essential for rendering views in web applications, generating emails, and more.

### Why EJS?

[EJS](https://ejs.co/) is a simple and flexible templating engine for Node.js. It uses plain JavaScript syntax within templates, making it easy to learn and integrate, especially if you're already familiar with JavaScript. EJS is versatile and can be used for rendering web pages, generating emails, and other text-based outputs.

**Key Features of EJS:**

- **Simplicity:** Easy to learn with straightforward syntax.
- **Flexibility:** Can be used for various purposes beyond web pages, such as email templates.
- **Integration:** Seamlessly integrates with Express.js and TypeScript.
- **Extensibility:** Supports partials and includes for reusable components.

## Basic Syntax of EJS

Understanding the basic syntax of EJS is crucial before integrating it into your projects. EJS templates are essentially HTML files with embedded JavaScript code.

### Variables

To embed variables within your template, use `<%= %>` for escaping and `<%- %>` for unescaped content.

```html
<!-- Escaped Output -->
<p>Hello, <%= username %>!</p>

<!-- Unescaped Output (Use with caution) -->
<p>Content: <%- content %></p>
```

### Control Flow

EJS supports standard JavaScript control flow statements like `if`, `for`, `each`, etc.

```html
<!-- If Statement -->
<% if (user.isAdmin) { %>
  <p>Welcome, admin!</p>
<% } else { %>
  <p>Welcome, user!</p>
<% } %>

<!-- For Loop -->
<ul>
  <% for(let i = 0; i < items.length; i++) { %>
    <li><%= items[i] %></li>
  <% } %>
</ul>

<!-- ForEach Loop -->
<ul>
  <% items.forEach(function(item) { %>
    <li><%= item %></li>
  <% }); %>
</ul>
```

### Includes

EJS allows you to include other EJS files within a template, promoting reusability.

```html
<!-- Including a Header Partial -->
<%- include('partials/header') %>

<!-- Including a Footer Partial -->
<%- include('partials/footer') %>
```

### Comments

Comments in EJS templates are not rendered in the output.

```html
<%# This is a comment and won't be rendered %>
```

## Hands-On Implementation

Let's implement EJS in an Express.js application using TypeScript. We'll create an API endpoint that sends templated emails using EJS.

### Prerequisites

- **Node.js** (v14 or later)
- **npm** or **yarn**
- Basic knowledge of **TypeScript** and **Express.js**

### 1. Initialize the Project

Create a new directory for your project and initialize it with npm:

```bash
mkdir express-ejs-email-ts
cd express-ejs-email-ts
npm init -y
```

### 2. Install Dependencies

Install the necessary packages:

```bash
npm install express nodemailer ejs dotenv
npm install --save-dev typescript @types/express @types/node ts-node-dev
```

**Dependencies:**

- **express**: Web framework for Node.js
- **nodemailer**: Module to send emails
- **ejs**: Templating engine
- **dotenv**: Loads environment variables from a `.env` file

**Dev Dependencies:**

- **typescript**, **ts-node-dev**: For TypeScript support
- **@types/\***: Type definitions for TypeScript

### 3. Configure TypeScript

Initialize a TypeScript configuration:

```bash
npx tsc --init
```

Modify the `tsconfig.json` as needed. A basic configuration might look like this:

```json
{
  "compilerOptions": {
    "target": "ES6",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  }
}
```

### 4. Set Up Project Structure

Create the following structure:

```
express-ejs-email-ts/
├── src/
│   ├── templates/
│   │   └── welcomeEmail.ejs
│   ├── index.ts
│   └── email.ts
├── .env
├── package.json
└── tsconfig.json
```

### 5. Create EJS Templates

Create an EJS template for the email. For example, `src/templates/welcomeEmail.ejs`:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Welcome Email</title>
</head>
<body>
  <h1>Welcome, <%= username %>!</h1>
  <p>Thank you for signing up. We're excited to have you on board.</p>
  
  <% if (isAdmin) { %>
    <p>You have administrative privileges.</p>
  <% } else { %>
    <p>You are a regular user.</p>
  <% } %>
  
  <h2>Your Interests:</h2>
  <ul>
    <% interests.forEach(function(interest) { %>
      <li><%= interest %></li>
    <% }); %>
  </ul>
  
  <p>Best regards,<br/>The Team</p>
</body>
</html>
```

**Explanation:**

- **Variables:** `<%= username %>` injects the `username` variable.
- **Control Flow:** The `if` statement displays different content based on the `isAdmin` flag.
- **Loop:** Iterates over the `interests` array to list user interests.

### 6. Configure Express to Use EJS

While EJS is primarily used for rendering HTML pages, we'll leverage it for email templating by rendering EJS templates to HTML strings.

Create `src/email.ts` to handle email sending logic with EJS templating:

```javascript
// src/email.ts
import nodemailer from 'nodemailer';
import dotenv from 'dotenv';
import ejs from 'ejs';
import path from 'path';

dotenv.config();

interface EmailOptions {
  to: string;
  subject: string;
  template: string; // Path to the EJS template
  data: any;        // Data to inject into the template
}

export const sendTemplatedEmail = async (options: EmailOptions) => {
  // Create a transporter
  const transporter = nodemailer.createTransport({
    service: process.env.EMAIL_SERVICE, // e.g., 'Gmail'
    auth: {
      user: process.env.EMAIL_USER,
      pass: process.env.EMAIL_PASS,
    },
  });

  // Resolve the template path
  const templatePath = path.join(__dirname, 'templates', options.template);

  try {
    // Render the EJS template to HTML
    const html = await ejs.renderFile(templatePath, options.data);

    // Define mail options
    const mailOptions = {
      from: process.env.EMAIL_USER,
      to: options.to,
      subject: options.subject,
      html: html,
    };

    // Send the email
    const info = await transporter.sendMail(mailOptions);
    console.log('Email sent:', info.response);
    return info;
  } catch (error) {
    console.error('Error sending email:', error);
    throw error;
  }
};
```

**Explanation:**

- **Nodemailer Transporter:** Configured using environment variables for service, user, and password.
- **EJS Rendering:** `ejs.renderFile` is used to render the specified EJS template with provided data.
- **Mail Options:** Defines the sender, recipient, subject, and HTML content.
- **Sending Email:** Uses `transporter.sendMail` to send the email.

### 7. Implement Email Sending with EJS Templates

Create `src/index.ts` to define the Express server and API endpoints for sending templated emails:

```javascript
// src/index.ts
import express, { Request, Response } from 'express';
import dotenv from 'dotenv';
import { sendTemplatedEmail } from './email';

dotenv.config();

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware to parse JSON
app.use(express.json());

// API Endpoint to send a welcome email
app.post('/send-welcome-email', async (req: Request, res: Response) => {
  const { to, username, isAdmin, interests } = req.body;

  if (!to || !username || !isAdmin || !interests) {
    return res.status(400).json({ message: 'Missing required fields: to, username, isAdmin, interests' });
  }

  try {
    await sendTemplatedEmail({
      to,
      subject: 'Welcome to Our Service!',
      template: 'welcomeEmail.ejs',
      data: { username, isAdmin, interests },
    });
    res.status(200).json({ message: 'Welcome email sent successfully.' });
  } catch (error) {
    res.status(500).json({ message: 'Failed to send welcome email.', error });
  }
});

// API Endpoint to check server status
app.get('/', (req: Request, res: Response) => {
  res.send('EJS Email Service is running.');
});

// Handle graceful shutdown
const shutdown = () => {
  console.log('Shutting down server...');
  process.exit(0);
};

process.on('SIGINT', shutdown);
process.on('SIGTERM', shutdown);

// Start the Express server
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

**Explanation:**

- **POST `/send-welcome-email`:** Accepts JSON data to send a welcome email using the `welcomeEmail.ejs` template.
- **GET `/`:** Simple endpoint to verify that the server is running.
- **Graceful Shutdown:** Ensures the server exits cleanly on termination signals.

### 8. Update `package.json` Scripts

Add a script to run the server with `ts-node-dev` for hot-reloading:

```json
"scripts": {
  "start": "ts-node-dev src/index.ts"
}
```

### 9. Running the Application

#### 1. Configure Environment Variables

Create a `.env` file in the root directory to store sensitive information and configuration variables:

```bash
PORT=3000
EMAIL_SERVICE=Gmail
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-email-password
```

**Important:**

- Replace `your-email@gmail.com` and `your-email-password` with your actual email credentials.
- For Gmail, you might need to enable [Less Secure Apps](https://myaccount.google.com/lesssecureapps) or use [App Passwords](https://support.google.com/accounts/answer/185833).

#### 2. Start the Server

```bash
npm start
```

You should see output similar to:

```
Server is running on port 3000
```

#### 3. Send a Test Welcome Email

Use a tool like [Postman](https://www.postman.com/) or `curl` to send a POST request to `http://localhost:3000/send-welcome-email` with a JSON body:

```json
{
  "to": "recipient@example.com",
  "username": "JohnDoe",
  "isAdmin": true,
  "interests": ["Coding", "Reading", "Gaming"]
}
```

**Using `curl`:**

```bash
curl -X POST http://localhost:3000/send-welcome-email \
  -H "Content-Type: application/json" \
  -d '{
    "to": "recipient@example.com",
    "username": "JohnDoe",
    "isAdmin": true,
    "interests": ["Coding", "Reading", "Gaming"]
  }'
```

**Expected Response:**

```json
{
  "message": "Welcome email sent successfully."
}
```

Check the recipient's inbox to verify that the email was received with the rendered EJS template.

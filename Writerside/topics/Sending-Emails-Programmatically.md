# Sending Emails Programmatically

This guide provides a hands-on approach to sending emails programmatically using Express.js with TypeScript. We'll utilize the popular [Nodemailer](https://nodemailer.com/about/) library to handle email sending. By the end of this tutorial, you'll have a simple API endpoint that can send emails based on client requests.

## Setup

### 1. Initialize the Project

First, create a new directory for your project and initialize it with npm:

```bash
mkdir express-email-ts
cd express-email-ts
npm init -y
```

### 2. Install Dependencies

Install the necessary packages:

```bash
npm install express nodemailer dotenv
npm install --save-dev typescript @types/express @types/node ts-node-dev
```

- **express**: Web framework for Node.js
- **nodemailer**: Module to send emails
- **dotenv**: Loads environment variables from a `.env` file
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
express-email-ts/
├── src/
│   ├── index.ts
│   └── email.ts
├── .env
├── package.json
└── tsconfig.json
```

## Implementation

### 1. Configure Environment Variables

Create a `.env` file in the root directory to store sensitive information like email credentials:

```bash
PORT=3000
EMAIL_SERVICE=Gmail
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-email-password
```

**Note:** For Gmail, you might need to enable [Less Secure Apps](https://myaccount.google.com/lesssecureapps) or use [App Passwords](https://support.google.com/accounts/answer/185833).

### 2. Create the Email Module

Create `src/email.ts` to handle email sending logic:

```javascript
// src/email.ts
import nodemailer from 'nodemailer';
import dotenv from 'dotenv';

dotenv.config();

interface EmailOptions {
  to: string;
  subject: string;
  text: string;
  html?: string;
}

const transporter = nodemailer.createTransport({
  service: process.env.EMAIL_SERVICE, // e.g., 'Gmail'
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASS,
  },
});

export const sendEmail = async (options: EmailOptions) => {
  const mailOptions = {
    from: process.env.EMAIL_USER,
    to: options.to,
    subject: options.subject,
    text: options.text,
    html: options.html || options.text,
  };

  try {
    const info = await transporter.sendMail(mailOptions);
    console.log('Email sent: ', info.response);
    return info;
  } catch (error) {
    console.error('Error sending email: ', error);
    throw error;
  }
};
```

### 3. Set Up the Express Server

Create `src/index.ts` to define the API endpoint:

```javascript
// src/index.ts
import express, { Request, Response } from 'express';
import { sendEmail } from './email';
import dotenv from 'dotenv';

dotenv.config();

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware to parse JSON
app.use(express.json());

app.post('/send-email', async (req: Request, res: Response) => {
  const { to, subject, text, html } = req.body;

  if (!to || !subject || !text) {
    return res.status(400).json({ message: 'Missing required fields: to, subject, text' });
  }

  try {
    await sendEmail({ to, subject, text, html });
    res.status(200).json({ message: 'Email sent successfully' });
  } catch (error) {
    res.status(500).json({ message: 'Failed to send email', error });
  }
});

app.get('/', (req: Request, res: Response) => {
  res.send('Email Service is running.');
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

### 4. Update `package.json` Scripts

Add a script to run the server with `ts-node-dev` for hot-reloading:

```json
"scripts": {
  "start": "ts-node-dev src/index.ts"
}
```

## Running the Application

1. **Start the Server**

   ```bash
   npm start
   ```

   You should see:

   ```
   Server is running on port 3000
   ```

2. **Send a Test Email**

   Use a tool like [Postman](https://www.postman.com/) or `curl` to send a POST request to `http://localhost:3000/send-email` with a JSON body:

   ```json
   {
     "to": "recipient@example.com",
     "subject": "Test Email",
     "text": "Hello! This is a test email sent from Express.js with TypeScript.",
     "html": "<p>Hello! This is a <strong>test email</strong> sent from Express.js with TypeScript.</p>"
   }
   ```

   **Using `curl`:**

   ```bash
   curl -X POST http://localhost:3000/send-email \
     -H "Content-Type: application/json" \
     -d '{
       "to": "recipient@example.com",
       "subject": "Test Email",
       "text": "Hello! This is a test email sent from Express.js with TypeScript.",
       "html": "<p>Hello! This is a <strong>test email</strong> sent from Express.js with TypeScript.</p>"
     }'
   ```

   **Expected Response:**

   ```json
   {
     "message": "Email sent successfully"
   }
   ```

   Check the recipient's inbox to verify the email was received.

## Error Handling and Validation

For production-ready applications, consider enhancing error handling and validating input data. Libraries like [Joi](https://joi.dev/) or [express-validator](https://express-validator.github.io/docs/) can help with robust validation.
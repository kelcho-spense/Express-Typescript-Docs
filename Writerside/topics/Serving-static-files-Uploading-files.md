# Serving static files &amp; Uploading files

### **7. Serving Static Files & Uploading Files Using Multer**

In Express.js, serving static files like HTML, CSS, JavaScript, and images, as well as handling file uploads (e.g., user profile pictures, documents), is a common requirement for many web applications. Express provides built-in functionality for serving static files, and file uploads can be managed using **Multer**, a popular middleware for handling multipart/form-data, which is primarily used for uploading files.

---

### **Serving Static Files in Express**

To serve static files (e.g., images, CSS files, JavaScript files), you can use the built-in middleware `express.static`. This middleware serves the files directly from the directory you specify.

#### Example: Basic Setup for Serving Static Files

```javascript
import express from 'express';
import path from 'path';

const app = express();
const PORT = 3000;

// Serve static files from the "public" directory
app.use(express.static(path.join(__dirname, 'public')));

app.get('/', (req, res) => {
  res.send('Welcome to the homepage!');
});

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

#### Directory Structure:
```
src/
  ├── app.ts
  └── public/
      ├── index.html
      ├── styles.css
      └── script.js
```

In this example:
- **`express.static(path.join(__dirname, 'public'))`**: This middleware serves static files from the `public` directory. Any file inside the `public` folder can be accessed directly via a URL. For example, an image located at `public/images/logo.png` can be accessed via `http://localhost:3000/images/logo.png`.

#### Serving Static Files at a Specific Route:
You can also mount static file serving to a specific route. For example:

```javascript
app.use('/static', express.static(path.join(__dirname, 'public')));
```

Now, files in the `public` folder will be served from `http://localhost:3000/static/`. For example, the file `public/images/logo.png` will be accessible via `http://localhost:3000/static/images/logo.png`.

---

### **Uploading Files Using Multer**

**Multer** is a middleware for handling `multipart/form-data`, primarily used for uploading files in Express applications. It allows you to handle file uploads easily and store them locally or upload them to cloud storage (e.g., AWS S3, Google Cloud).

#### Installation:

```bash
npm install multer
npm install @types/multer --save-dev
```

#### Basic File Upload Example

1. **Set up Multer in your Express application**:
   Multer requires setting up a storage engine that defines how and where to store uploaded files.

```javascript
import express from 'express';
import multer from 'multer';
import path from 'path';

const app = express();
const PORT = 3000;

// Set up Multer storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/'); // Directory where files will be saved
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname)); // Custom filename
  }
});

const upload = multer({ storage });

app.use(express.static('public'));

// Simple file upload route
app.post('/upload', upload.single('file'), (req, res) => {
  res.send('File uploaded successfully!');
});

app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
```

In this example:
- **`multer.diskStorage`**: Defines the storage engine. The uploaded file is stored in the `uploads` directory, and the file name is appended with a unique timestamp.
- **`upload.single('file')`**: Middleware to handle single file uploads. The string `'file'` should match the name attribute of the file input in your HTML form.

#### HTML Form for File Upload (Public Folder):

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>File Upload</title>
</head>
<body>
  <h1>Upload a File</h1>
  <form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="file" />
    <button type="submit">Upload</button>
  </form>
</body>
</html>
```

In this form:
- **`enctype="multipart/form-data"`**: This is important for handling file uploads. Without it, the file won’t be included in the form data sent to the server.

#### Accessing Uploaded File Information:

After the file is uploaded, you can access its details via `req.file`.

```javascript
app.post('/upload', upload.single('file'), (req, res) => {
  console.log(req.file); // Information about the uploaded file
  res.send('File uploaded successfully!');
});
```

The `req.file` object contains useful information about the uploaded file, such as:
- `filename`: The name of the file on the server.
- `originalname`: The original name of the uploaded file.
- `mimetype`: The MIME type of the file.
- `size`: The size of the uploaded file.

---

### **Uploading Multiple Files**

You can also configure Multer to handle multiple file uploads at once.

```javascript
app.post('/uploads', upload.array('files', 5), (req, res) => {
  console.log(req.files); // Array of uploaded files
  res.send('Multiple files uploaded successfully!');
});
```

In this example:
- **`upload.array('files', 5)`**: This allows the client to upload up to 5 files at once. The `files` field in the HTML form should match the name attribute in the `input` tag.

#### Example HTML Form for Multiple Files:

```html
<form action="/uploads" method="post" enctype="multipart/form-data">
  <input type="file" name="files" multiple />
  <button type="submit">Upload</button>
</form>
```

---

### **Handling File Types and Size Limits**

You can add validation in Multer to control the file type and size.

#### Restrict File Type:

You can use the `fileFilter` option to allow only certain types of files (e.g., images).

```javascript
const upload = multer({
  storage,
  fileFilter: (req, file, cb) => {
    const fileTypes = /jpeg|jpg|png|gif/;
    const extName = fileTypes.test(path.extname(file.originalname).toLowerCase());
    const mimeType = fileTypes.test(file.mimetype);

    if (extName && mimeType) {
      return cb(null, true);
    } else {
      cb(new Error('Only images are allowed!'));
    }
  }
});
```

#### Restrict File Size:

You can restrict the size of uploaded files using the `limits` option.

```javascript
const upload = multer({
  storage,
  limits: { fileSize: 1024 * 1024 * 2 } // Limit file size to 2MB
});
```

---

### **Organizing File Upload Logic**

For larger applications, you should organize file upload logic into a more modular structure. You can separate concerns by moving your file upload configurations and logic into separate files.

#### Example Structure:
```
src/
  ├── app.ts
  ├── controllers/
  │   └── uploadController.ts
  ├── routes/
  │   └── uploadRoutes.ts
  └── config/
      └── multerConfig.ts
```

- **`multerConfig.ts`**: Configuration for file uploads.
- **`uploadController.ts`**: Logic for handling uploaded files.
- **`uploadRoutes.ts`**: Defines the routes for handling file uploads.

#### Example `multerConfig.ts`:

```javascript
import multer from 'multer';
import path from 'path';

const storage = multer.diskStorage({
  destination: 'uploads/',
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
  }
});

export const upload = multer({
  storage,
  limits: { fileSize: 1024 * 1024 * 5 }, // 5MB limit
  fileFilter: (req, file, cb) => {
    const fileTypes = /jpeg|jpg|png|gif/;
    const extName = fileTypes.test(path.extname(file.originalname).toLowerCase());
    const mimeType = fileTypes.test(file.mimetype);

    if (extName && mimeType) {
      return cb(null, true);
    } else {
      cb(new Error('Only images are allowed!'));
    }
  }
});
```

#### Example `uploadController.ts`:

```javascript
import { Request, Response } from 'express';

export const uploadFile = (req: Request, res: Response) => {
  if (!req.file) {
    return res.status(400).send('No file uploaded');
  }
  res.send('File uploaded successfully!');
};
```

#### Example `uploadRoutes.ts`:

```javascript
import express from 'express';
import { upload } from '../config/multerConfig';
import

 { uploadFile } from '../controllers/uploadController';

const router = express.Router();

router.post('/upload', upload.single('file'), uploadFile);

export default router;
```

#### Example `app.ts`:

```javascript
import express from 'express';
import uploadRoutes from './routes/uploadRoutes';

const app = express();
app.use(express.json());
app.use('/api', uploadRoutes);

app.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});
```

---

### Summary

- **Serving Static Files**: Use the `express.static()` middleware to serve static assets (like HTML, CSS, images) from a directory. These files can be accessed directly via their URL path.
- **File Uploads with Multer**: Use Multer to handle file uploads by configuring storage options (e.g., where to store files) and handling multipart/form-data. You can restrict file size, type, and name uploads for security and consistency.
- **Organizing Upload Logic**: For larger applications, separate concerns into different files (controllers, routes, configurations) to maintain modularity and clean code.

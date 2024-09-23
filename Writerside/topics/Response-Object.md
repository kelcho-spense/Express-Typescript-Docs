# Response Object

The **Response object (`res`)** in Express is used to send the HTTP response to the client after a request is processed. It contains various methods that allow you to manipulate the response in different ways, such as sending JSON, setting the status code, sending files, or ending the request-response cycle.

### **a. What is the Response Object?**

The `res` object represents the HTTP response that an Express app sends when it receives an incoming request. It provides various methods to send different types of responses to the client, such as JSON data, plain text, HTML, or files.

#### Example:
```javascript
app.get('/example', (req, res) => {
  res.status(200).json({ message: 'Hello, World!' });
});
```

---

### **b. Response Methods**

Express provides several useful methods within the `res` object to handle various response types. Here’s a breakdown of commonly used methods.

---

### **i. `res.json()`**

The `res.json()` method is used to send a JSON-formatted response. This method automatically sets the `Content-Type` header to `application/json`.

#### Example:
```javascript
app.get('/users', (req, res) => {
  const users = [{ id: 1, name: 'John Doe' }, { id: 2, name: 'Jane Doe' }];
  res.json(users);
});
```

In this example, a JSON object (array of users) is sent as the response. The `res.json()` method is ideal for RESTful APIs, where you often need to send structured JSON data to clients.

#### How It Works:
- Automatically converts JavaScript objects to JSON strings.
- Sets the `Content-Type` header to `application/json`.

Example Output:
```json
[
  { "id": 1, "name": "John Doe" },
  { "id": 2, "name": "Jane Doe" }
]
```

---

### **ii. `res.status()`**

The `res.status()` method sets the HTTP status code of the response. Status codes are essential for indicating the result of an HTTP request (e.g., `200` for success, `404` for not found, `500` for server errors).

#### Example:
```javascript
app.get('/notfound', (req, res) => {
  res.status(404).json({ message: 'Resource not found' });
});
```

In this example, a 404 (Not Found) status is returned, along with a JSON message.

#### How It Works:
- Sets the HTTP status code for the response.
- Must be called before sending the final response with `res.send()`, `res.json()`, etc.

Common Status Codes:
- `200` – OK (successful request)
- `201` – Created (resource successfully created)
- `400` – Bad Request (client-side error)
- `404` – Not Found (resource not found)
- `500` – Internal Server Error (server-side error)

---

### **iii. `res.send()`**

The `res.send()` method is used to send a response body of various types, including strings, buffers, or objects. If an object is passed, Express will automatically convert it to JSON (similar to `res.json()`, but less explicit).

#### Example:
```javascript
app.get('/hello', (req, res) => {
  res.send('Hello, World!');
});
```

This sends a plain text response to the client.

#### How It Works:
- Can send strings, arrays, buffers, or objects.
- Automatically sets the `Content-Type` based on the argument type (e.g., `text/plain` for strings, `application/json` for objects).

Example:
```javascript
app.get('/data', (req, res) => {
  res.send({ message: 'This is an object response' });
});
```

---

### **iv. `res.redirect()`**

The `res.redirect()` method is used to redirect the client to a different URL. It sends a `302 Found` status code by default, but this can be changed to another redirect status code like `301 Moved Permanently`.

#### Example:
```javascript
app.get('/old-route', (req, res) => {
  res.redirect('/new-route');
});
```

In this example, a client requesting `/old-route` will be redirected to `/new-route`.

#### How It Works:
- By default, the status code is `302` (temporary redirect).
- You can pass a status code as the first argument to specify a different type of redirect.

Example:
```javascript
app.get('/permanent-redirect', (req, res) => {
  res.redirect(301, '/new-permanent-route');
});
```

In this case, the server responds with a `301 Moved Permanently` status, indicating that the resource has been moved to a new location.

---

### **v. `res.download()`**

The `res.download()` method prompts the client to download a file from the server. It automatically sets the appropriate headers so that the client knows it's receiving a file to download, rather than to render in the browser.

#### Example:
```javascript
app.get('/download', (req, res) => {
  const filePath = './files/sample.pdf';
  res.download(filePath, 'sample.pdf', (err) => {
    if (err) {
      res.status(500).send('Error downloading file');
    }
  });
});
```

In this example, the client will download the file `sample.pdf` from the server.

#### How It Works:
- Automatically sets the `Content-Disposition` header to trigger the download.
- The second argument is optional and allows you to specify a custom filename for the downloaded file.

---

### **vi. `res.end()`**

The `res.end()` method is used to end the response process without sending any data. It is useful in situations where you need to terminate the connection, and no further data is to be sent.

#### Example:
```javascript
app.get('/close', (req, res) => {
  res.status(204).end(); // 204 No Content
});
```

In this example, the `res.end()` method is used to end the response without sending any content back to the client.

#### How It Works:
- Ends the response process.
- You should call this when you’ve finished writing to the response and want to terminate the connection.
- It does not send any data or body content, just ends the response.

---

### Summary

- **`res.json()`**: Sends a JSON response. Automatically sets the `Content-Type` header to `application/json`.
- **`res.status()`**: Sets the HTTP status code for the response. Should be called before `res.send()` or `res.json()`.
- **`res.send()`**: Sends a response of various types (strings, objects, etc.). Automatically converts objects to JSON if needed.
- **`res.redirect()`**: Redirects the client to a different URL. The default status code is `302` (Found), but it can be changed to `301` for permanent redirects.
- **`res.download()`**: Prompts the client to download a file from the server.
- **`res.end()`**: Ends the response process without sending any additional data. Used when no further data needs to be sent to the client.

This collection of response methods helps you handle a variety of use cases, whether you're sending JSON data, redirecting users, or handling file downloads.
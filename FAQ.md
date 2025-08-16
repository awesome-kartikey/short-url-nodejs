# FAQ.md

# Frequently Asked Questions (FAQ)

**Q1: How is the short ID generated? Is it unique?**

A: The short ID is generated using the `shortid` npm package. This package generates short, non-sequential, URL-friendly unique identifiers. Additionally, the `shortId` field in the MongoDB `url` schema has a unique index, ensuring that the database enforces uniqueness.

**Q2: How does the authentication system work?**

A: The system uses a simple session-based authentication mechanism.

1.  When a user logs in successfully (`controllers/user.js -> handleUserLogin`), a unique session ID is generated using `uuidv4`.
2.  This session ID is stored as a key in an in-memory map (`service/auth.js -> sessionIdToUserMap`), with the user's data as the value.
3.  The session ID is sent back to the user's browser as an HTTP cookie named `uid`.
4.  For subsequent requests to protected routes, the `middlewares/auth.js -> restrictToLoggedinUserOnly` middleware checks for the `uid` cookie.
5.  It retrieves the user data from the in-memory map using the session ID found in the cookie.
6.  If the user is found, the request proceeds; otherwise, the user is redirected to the login page.
7.  The `checkAuth` middleware performs a similar check but doesn't redirect, only attaching the user object to the request if found, allowing certain routes (like the home page) to display different content based on login status.

**Q3: Is the user session persistent if the server restarts?**

A: No. The current implementation stores session data in an in-memory JavaScript `Map` (`sessionIdToUserMap` in `service/auth.js`). This map is cleared whenever the Node.js process restarts. For persistent sessions suitable for production, a database-backed session store (like Redis or storing sessions in MongoDB) would be necessary.

**Q4: How can I see how many times my short URL has been clicked?**

A: When you are logged in, the home page (`/`) displays a table listing all the short URLs you have created. One of the columns in this table shows the total number of clicks for each URL. You can also get detailed visit history (timestamps) via the API endpoint `GET /url/analytics/:shortId`.

**Q5: What database is used?**

A: The application uses MongoDB as its database. Mongoose is used as the Object Data Modeling (ODM) library to interact with MongoDB, define schemas, and model application data.

**Q6: Why was EJS chosen as the templating engine?**

A: EJS (Embedded JavaScript templates) is a simple templating language that lets you generate HTML markup with plain JavaScript. It's easy to learn and integrates well with Express, making it a common choice for rendering dynamic HTML content directly from the server in Node.js applications.

**Q7: Is there any rate limiting?**

A: No, the current implementation does not include any rate limiting for generating URLs or accessing them. In a production scenario, rate limiting should be added to prevent abuse.

**Q8: Are passwords stored securely?**

A: No. Currently, passwords are stored in plain text in the database (`models/user.js` schema defines password as String). **This is highly insecure and not suitable for production.** Passwords should always be hashed using a strong algorithm like bcrypt before storing them.

---

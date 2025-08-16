# ARCHITECTURE.md

# System Architecture

This project follows a variation of the Model-View-Controller (MVC) architectural pattern, commonly used in web applications built with frameworks like Express.js. It also includes a `service` layer for specific concerns like authentication state management.

## Project Structure

```
short-url-nodejs/
├── connect.js           # Handles MongoDB database connection setup.
├── index.js             # Main application entry point. Sets up Express, middleware, routes.
├── package.json         # Project metadata and dependencies.
├── .env                 # Environment variables (e.g., DB connection string) - (Needs to be created)
├── controllers/         # Request handlers (logic for routes).
│   ├── url.js           # Logic for URL shortening and analytics.
│   └── user.js          # Logic for user signup and login.
├── middlewares/         # Custom middleware functions.
│   └── auth.js          # Authentication and authorization checks.
├── models/              # Mongoose schemas defining data structures.
│   ├── url.js           # Schema for URL documents.
│   └── user.js          # Schema for User documents.
├── routes/              # Defines application routes/endpoints.
│   ├── staticRouter.js  # Routes for serving EJS views (HTML pages).
│   ├── url.js           # API routes related to URLs (/url).
│   └── user.js          # API routes related to users (/user).
├── service/             # Business logic or services decoupled from controllers.
│   └── auth.js          # Handles session ID to user mapping (in-memory session).
└── views/               # EJS templates for the user interface.
    ├── home.ejs         # Home page: URL generation form and list of URLs.
    ├── login.ejs        # Login form page.
    └── signup.ejs       # Signup form page.
```

## Major Components

1.  **`index.js` (Entry Point):**

    - Initializes the Express application.
    - Connects to MongoDB using `connect.js`.
    - Loads essential middleware (`express.json`, `express.urlencoded`, `cookie-parser`).
    - Mounts routers (`urlRoute`, `staticRoute`, `userRoute`) to specific paths.
    - Defines the core redirection logic for `/url/:shortId`.
    - Starts the HTTP server.

2.  **`connect.js` (Database Connector):**

    - Provides a function `connectToMongoDB` that uses Mongoose to establish a connection to the MongoDB database specified in the environment variable.
    - Includes basic error handling for the connection attempt.

3.  **`routes/` (Routers):**

    - Define the application's endpoints.
    - `user.js`: Handles `/user` (signup) and `/user/login` (login) POST requests.
    - `url.js`: Handles `/url` (POST for creating short URLs) and `/url/analytics/:shortId` (GET for analytics). Protected by `restrictToLoggedinUserOnly` middleware.
    - `staticRouter.js`: Handles routes serving HTML pages (`/`, `/signup`, `/login`). Uses `checkAuth` middleware to potentially pass user data to views.

4.  **`controllers/` (Controllers):**

    - Contain the core logic executed when a route is hit.
    - Interact with models (database operations) and services.
    - Format responses (rendering views or sending JSON).
    - `user.js`: `handleUserSignup`, `handleUserLogin`.
    - `url.js`: `handleGenerateNewShortURL`, `handleGetAnalytics`.

5.  **`models/` (Models):**

    - Define the structure of data stored in MongoDB using Mongoose Schemas.
    - `user.js`: Defines the `User` schema (name, email, password).
    - `url.js`: Defines the `URL` schema (shortId, redirectURL, visitHistory, createdBy reference to User).

6.  **`middlewares/` (Middleware):**

    - Functions executed during the request-response cycle.
    - `auth.js`:
      - `restrictToLoggedinUserOnly`: Checks for a valid session cookie and user mapping. Redirects to `/login` if authentication fails. Used for routes requiring login.
      - `checkAuth`: Checks for a valid session cookie and attaches the user object to `req.user` if found, but does _not_ block the request if authentication fails. Used for routes that behave differently depending on login status (e.g., showing login/signup links vs. user content).

7.  **`service/auth.js` (Auth Service):**

    - Abstracts the session management logic.
    - Uses an in-memory `Map` (`sessionIdToUserMap`) to store active sessions (mapping session IDs from cookies to user objects).
    - Provides `setUser` and `getUser` functions for managing this map. **Note:** This is not persistent storage.

8.  **`views/` (Views):**
    - EJS templates used to render the HTML presented to the user.
    - Dynamically display data passed from controllers (e.g., user info, list of URLs, generated short ID).

## Data Flow Examples

1.  **User Login:**

    - Browser sends POST request to `/user/login` with email/password.
    - Express routes request to `userRoute`.
    - `userRoute` maps `/login` path to `handleUserLogin` in `userController`.
    - `handleUserLogin` queries the `User` model to find a matching user.
    - If user found:
      - Generates a `sessionId` using `uuidv4`.
      - Calls `setUser` in `authService` to store `sessionId -> user` mapping in the in-memory map.
      - Sets a `uid` cookie with the `sessionId` on the response.
      - Redirects the browser to `/`.
    - If user not found: Renders the `login.ejs` view with an error message.

2.  **URL Shortening:**

    - User (logged in) submits form on `/` page (POST request to `/url`).
    - Express routes request to `urlRoute`.
    - `restrictToLoggedinUserOnly` middleware runs:
      - Checks `uid` cookie.
      - Uses `getUser` from `authService` to retrieve user from map.
      - Attaches user object to `req.user`. If fails, redirects to `/login`.
    - `urlRoute` maps `/` path to `handleGenerateNewShortURL` in `urlController`.
    - `handleGenerateNewShortURL`:
      - Generates `shortID` using `shortid`.
      - Creates a new document using the `URL` model, storing `shortID`, original URL (`body.url`), and `createdBy` (from `req.user._id`).
      - Renders the `home.ejs` view, passing the newly generated `id` (shortID) to be displayed.

3.  **URL Redirection:**
    - Browser sends GET request to `/url/:shortId`.
    - `index.js` matches this route directly.
    - The route handler:
      - Extracts `shortId` from `req.params`.
      - Uses `URL.findOneAndUpdate` to find the document with the matching `shortId`.
      - Atomically pushes a new timestamp object into the `visitHistory` array.
      - Retrieves the `redirectURL` from the found document.
      - Sends a 302 Redirect response to the browser, pointing to the `redirectURL`.

## Design Decisions

- **MVC Pattern:** Chosen for its standard structure, separating concerns (data, presentation, logic) which improves maintainability.
- **Express.js:** A minimal and flexible Node.js web application framework, suitable for building APIs and web applications quickly.
- **MongoDB/Mongoose:** NoSQL database offering flexibility. Mongoose provides schema definition, validation, and business logic hooks, simplifying interaction with MongoDB.
- **EJS Templating:** Simple server-side rendering for the basic UI requirements of this project. Avoids the need for a separate frontend framework for this simple use case.
- **In-Memory Session Management:** Simplest way to implement session state for demonstration purposes. **Not suitable for production** due to lack of persistence and scalability.
- **`shortid` Library:** Provides a convenient way to generate short, unique IDs suitable for URLs.
- **Middleware for Auth:** Centralizes authentication logic, making routes cleaner and ensuring consistent security checks. The distinction between `restrictToLoggedinUserOnly` (enforces login) and `checkAuth` (checks login status without enforcing) allows flexibility in route handling.
- **Environment Variables (`dotenv`):** Used to keep sensitive configuration like database connection strings out of the source code.

---

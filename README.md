# README.md

# Awesome Kartikey Short URL - Node.js

A simple URL shortening service built with Node.js, Express, and MongoDB. It allows users to sign up, log in, generate short URLs for long URLs, and track the number of clicks on their shortened links.

## Features

- User Authentication (Signup/Login)
- Generate unique short IDs for original URLs.
- Redirect users from short URLs to the original URL.
- Track visit history (timestamps) for each short URL.
- Display user-specific generated URLs and click counts.
- Simple web interface using EJS templates.
- RESTful API endpoint for analytics.

## Tech Stack

- **Backend:** Node.js, Express.js
- **Database:** MongoDB with Mongoose ODM
- **Templating Engine:** EJS
- **Authentication:** Custom session management using Cookies and in-memory storage (`service/auth.js`)
- **Short ID Generation:** `shortid`
- **Environment Variables:** `dotenv`
- **Development:** `nodemon`
- **Other:** `cookie-parser`, `uuid` (for session IDs)

## Setup Instructions

1.  **Clone the repository:**

    ```bash
    git clone <your-repository-url>
    cd short-url-nodejs
    ```

2.  **Install dependencies:**

    ```bash
    npm install
    ```

3.  **Set up Environment Variables:**
    Create a `.env` file in the root of the project directory and add your MongoDB connection string:

    ```env
    MONGODB=mongodb://<your_mongodb_host>:<port>/<database_name>
    # Example: MONGODB=mongodb://127.0.0.1:27017/short-url
    ```

    Replace the placeholder with your actual MongoDB connection details.

4.  **Run the application:**
    ```bash
    npm start
    ```
    This command uses `nodemon` to start the server, which will automatically restart upon file changes. The server will typically start on `http://localhost:8001`.

## Usage

1.  **Navigate:** Open your web browser and go to `http://localhost:8001`.
2.  **Signup/Login:**
    - If you are a new user, click the link to navigate to `/signup` and create an account.
    - If you already have an account, navigate to `/login` and enter your credentials.
3.  **Generate Short URL:**
    - Once logged in, you'll be redirected to the home page (`/`).
    - Enter a long URL into the input field and click "Generate".
    - The generated short URL (e.g., `http://localhost:8001/url/xxxxxx`) will be displayed.
4.  **Use Short URL:**
    - Copy the generated short URL and use it in your browser or share it. It will redirect to the original long URL.
5.  **View Analytics:**
    - The home page displays a table of the URLs you've created, including the short ID, original URL, and the total number of clicks.
    - You can also access raw analytics data (visit timestamps) via the API endpoint: `GET /url/analytics/:shortId` (replace `:shortId` with the actual short ID). This endpoint requires you to be logged in.

---

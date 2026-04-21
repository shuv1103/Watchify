# Watchify Backend

Production-oriented Node.js/Express backend for Watchify, a video platform API that supports user authentication, profile/media management, channel metadata, and watch-history retrieval.

---

## 1) System Overview

This service is a **modular monolith** built on:
- **Node.js + Express** for HTTP APIs
- **MongoDB + Mongoose** for persistence
- **JWT + cookies** for auth/session flows
- **Multer + Cloudinary** for media ingestion and storage

At runtime, the application boots environment variables, connects to MongoDB, configures middleware, mounts versioned routes, and serves static assets from `public/`.

---

## 2) Tech Stack

- **Runtime:** Node.js (ES modules enabled)
- **Web Framework:** Express 
- **Database:** MongoDB via Mongoose 
- **Authentication:** JWT (`jsonwebtoken`) + cookie transport (`cookie-parser`)
- **File Uploads:** `multer` (disk storage temp staging) + Cloudinary SDK
- **Security Utilities:** `bcrypt` for password hashing
- **Developer Tooling:** `nodemon`, `prettier`

---

## 3) Project Structure

```text
Watchify/
├── package.json
├── package-lock.json
├── readme.md
├── public/
│   └── temp/
│       ├── 21BCE10487.jpg
│       └── coding.png
└── src/
    ├── app.js
    ├── constants.js
    ├── index.js
    ├── controllers/
    │   └── user.controller.js
    ├── db/
    │   └── index.js
    ├── middlewares/
    │   ├── auth.middleware.js
    │   └── multer.middleware.js
    ├── models/
    │   ├── subscription.model.js
    │   ├── user.model.js
    │   └── video.model.js
    ├── routes/
    │   └── user.routes.js
    └── utils/
        ├── ApiError.js
        ├── ApiResponse.js
        ├── asyncHandler.js
        └── cloudinary.js
```

### Root files
- **`package.json`**: Metadata, dependency graph, and `dev` start script using `nodemon` + dotenv preload.
- **`package-lock.json`**: Deterministic dependency lockfile for reproducible builds.
- **`readme.md`**: Project documentation (this file).

### `public/`
- **Purpose:** Static-file root served by Express (`express.static("public")`).
- **`public/temp/`**: Temporary upload staging area used by Multer before files are pushed to Cloudinary.
- Existing image files under `temp/` appear to be sample/dev artifacts.

### `src/index.js` (Application bootstrap)
- Loads env vars from `./env`.
- Connects to MongoDB using `connectDB()`.
- Starts HTTP listener only after successful DB connection.

### `src/app.js` (Express app factory)
- Configures CORS, JSON/body parsers, cookie parser, static directory.
- Mounts user routes at `/api/v1/users`.
- Exports configured `app` for bootstrap in `index.js`.

### `src/constants.js`
- Defines central constants, currently `DB_NAME = "Watchify"`.

### `src/db/index.js`
- Encapsulates DB connection (`mongoose.connect`) and process-fatal handling on failure.

### `src/routes/user.routes.js`
- Declares user-auth/profile routes and middleware composition.
- Uses Multer for multipart endpoints (`register`, `avatar`, `cover-image`).
- Applies JWT verification middleware to protected routes.

### `src/controllers/user.controller.js`
Contains core user-domain business logic:
- `registerUser`
- `loginUser`
- `logoutUser`
- `refreshAccessToken`
- `changeCurrentPassword`
- `getCurrentUser`
- `updateAccountDetails`
- `updateUserAvatar`
- `updateUserCoverImage`
- `getUserChannelProfile`
- `getWatchHistory`

Also includes helper `generateAccessAndRefreshTokens(userId)` for auth token lifecycle.

### `src/middlewares/`
- **`auth.middleware.js`**: JWT verification (`accessToken` from cookies or Bearer token), injects `req.user`.
- **`multer.middleware.js`**: Multer disk-storage strategy to `./public/temp`, preserving original file name.

### `src/models/`
- **`user.model.js`**:
  - Core identity profile and auth fields.
  - Password hashing in pre-save hook.
  - Instance methods for password validation and token generation.
- **`subscription.model.js`**:
  - Join-like structure from `subscriber -> channel` (both `User` refs).
- **`video.model.js`**:
  - Video metadata, owner relation, and aggregate pagination plugin.

### `src/utils/`
- **`ApiError.js`**: Structured application error type.
- **`ApiResponse.js`**: Consistent success response envelope.
- **`asyncHandler.js`**: Promise-wrapper for async Express handlers.
- **`cloudinary.js`**: Cloudinary client config and local-file upload helper.

---

## 4) API Surface (Current Routes)

Base path: `http://localhost:<PORT>/api/v1/users`

### Public routes
- `POST /register` (multipart: `avatar` required, `coverImage` optional)
- `POST /login`
- `POST /refresh-token`

### Protected routes (`verifyJWT`)
- `POST /logout`
- `POST /change-password`
- `GET /current-user`
- `PATCH /avatar` (multipart: `avatar`)
- `PATCH /cover-image` (multipart: `coverImage`)
- `GET /channel/:username`
- `GET /history`

> Note: Route `PATCH /update-account` is declared with auth middleware but no controller attached yet.

---

## 5) Data Models

### User
- Identity: `username`, `email`, `fullName`
- Media: `avatar`, `coverImage`
- Auth: `password`, refresh-token field
- Engagement: `watchHistory` relation to videos

### Subscription
- `subscriber` (User ObjectId)
- `channel` (User ObjectId)

### Video
- Storage/media fields (`videoFile`, `thumbnail`)
- Metadata (`title`, `description`, `duration`, `views`, `isPublished`)
- Ownership (`owner` User reference)

---

## 6) Local Setup

## Prerequisites
- Node.js 18+
- MongoDB instance
- Cloudinary account

## Install
```bash
npm install
```

## Environment
Create `env` file at project root:

```bash
PORT=3000
CORS_ORIGIN=http://localhost:5173
MONGODB_URI=mongodb://localhost:27017

ACCESS_TOKEN_SECRET=<strong-random-secret>
ACCESS_TOKEN_EXPIRY=1d
REFRESH_TOKEN_SECRET=<strong-random-secret>
REFRESH_TOKEN_EXPIRY=10d

CLOUDINARY_CLOUD_NAME=<cloud-name>
CLOUDINARY_API_KEY=<api-key>
CLOUDINARY_API_SECRET=<api-secret>
```

## Run (development)
```bash
npm run dev

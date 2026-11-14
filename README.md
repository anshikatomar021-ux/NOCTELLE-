# Noctelle
> Where every connection has a soundtrack.

---

## Architecture and Master Plan

Welcome to the architectural specification and master development plan for **Noctelle**. This document serves as the permanent source of truth for all engineering phases.

---

### 1. Architecture Overview

Noctelle is built using the **MERN** architecture (**M**ongoDB, **E**xpress, **R**eact, **N**ode.js). Here is how the three core layers communicate in simple terms:

1. **The Client (React SPA in the Browser):**
   - The user opens their browser and loads our React application.
   - React is a Single Page Application (SPA), which means the browser downloads the page shell once. As the user clicks around, pages change instantly without a full page refresh.
   - When the user performs an action (like playing a song or liking a post), React makes asynchronous HTTP requests using the browser's built-in `fetch` API.
2. **The Server (Node.js + Express REST API):**
   - Node.js runs our JavaScript backend server, and Express organizes our backend into clean URL routes (endpoints like `/api/posts` or `/api/music/search`).
   - The server acts as a secure gatekeeper. It checks security tokens (JWT), validates user inputs, executes business logic, and ensures no private API keys ever reach the user's browser.
3. **The Database (MongoDB + Mongoose):**
   - MongoDB stores all permanent data (user accounts, playlists, posts, comments) as flexible JSON-like documents.
   - Mongoose is the translator (Object Data Modeling library) in Express that makes sure data saved to MongoDB conforms to our schema rules and allows us to run readable queries like `Post.findById(id)`.

#### Request Flow Diagram: Liking a Post

```text
[ USER ACTION ]
User clicks the "Like" heart button on a post
       │
       ▼
[ REACT FRONTEND ]
React triggers an event handler and sends an HTTP request:
POST /api/posts/65f1a2b3c/like (credentials: 'include' sends the session cookie)
       │
       ▼
[ EXPRESS BACKEND ]
Express receives the request on port 5000
       │
       ├──► 1. Auth Middleware (auth.js): Reads httpOnly cookie, verifies JWT, attaches req.user
       ├──► 2. Router (postRoutes.js): Directs request to postController.toggleLike
       └──► 3. Controller (postController.js): Checks if user already liked the post
                   │
                   ▼
[ MONGOOSE & MONGODB ]
Mongoose executes atomic update:
db.posts.updateOne({ _id: postId }, { $addToSet: { likes: userId } })
MongoDB updates the document in the 'noctelle' database and returns success
       │
       ▼
[ RESPONSE BACK TO CLIENT ]
Controller sends JSON response: { success: true, likesCount: 42, isLiked: true }
       │
       ▼
[ REACT STATE UPDATE ]
React receives the 200 OK response, updates local state, and lights up the heart icon
with a vibrant violet glow—without reloading the page or pausing the music.
```

---

### 2. Tech Stack Justification & Strict Dependency Rule

Every technology is chosen to keep the application lightweight, fast, modern, and beginner-friendly:

- **React (Vite):** Blazing fast component-driven UI library that updates only what changed on screen, allowing background music to play continuously while pages change.
- **Node.js & Express:** Event-driven, non-blocking JavaScript runtime and backend framework allowing us to use one single language (JavaScript) across the entire stack.
- **MongoDB & Mongoose:** Schema-flexible document database ideal for social feeds, playlists, and nested comments without painful database migrations.
- **Tailwind CSS:** Modern utility-first CSS framework enabling our custom midnight/violet glassmorphism aesthetic with zero CSS file bloat.
- **Native HTML5 Audio / Web Audio API:** The browser's native audio engine handles streaming, volume, and playback scrubbing without requiring heavy external media player libraries.
- **JWT (JSON Web Tokens) & bcryptjs:** Industry standard for secure, stateless user authentication and one-way password hashing.

#### Exact Package Manifest

To prevent dependency bloat, we strictly use built-in browser and Node.js capabilities (such as the native `fetch` API instead of Axios). Every package listed below is strictly justified:

#### Client Packages (`/client`)
- **`react` & `react-dom`**: Core React library for rendering components and virtual DOM reconciliation.
- **`react-router-dom`**: Client-side page navigation that keeps the audio player alive during route changes.
- **`lucide-react`**: Ultra-clean, modern icon set (play, pause, heart, disc, search, user) essential for an intuitive music UI.
- *Dev Dependencies*:
  - **`vite` & `@vitejs/plugin-react`**: Next-generation frontend build tool with near-instant hot module replacement.
  - **`tailwindcss`, `postcss`, `autoprefixer`**: Utility CSS engine and vendor prefixing.

#### Server Packages (`/server`)
- **`express`**: Minimalist web server framework for handling REST API routing.
- **`mongoose`**: Elegant MongoDB object modeling, schema validation, and query builder.
- **`dotenv`**: Loads sensitive configuration (database URI, JWT secret, API keys) from a `.env` file.
- **`jsonwebtoken`**: Issues and cryptographically verifies JWT tokens for user sessions.
- **`bcryptjs`**: Hashes user passwords with salt rounds in pure JavaScript (avoids tricky C++ build dependencies on Windows/Mac/Linux).
- **`cookie-parser`**: Parses incoming HTTP request cookie headers so our backend can securely extract the authentication token.
- **`cors`**: Manages Cross-Origin Resource Sharing so our React frontend can communicate safely with our Express backend during development.
- *Dev Dependencies*:
  - **`nodemon`**: Development utility that automatically restarts the Node server when code changes are saved.

*Approval note on non-mandatory packages:* `cookie-parser` is added solely to support secure `httpOnly` cookie authentication (which protects students from token theft), and `lucide-react` provides lightweight vector icons. No additional packages will be installed without prior discussion.

---

### 3. Minimal Folder Structure

A clean, predictable project structure with zero fluff, no duplicate components, and a single clear purpose for every file:

```text
noctelle/
├── README.md                           # Master project documentation and architecture blueprint
├── client/                             # React frontend single page application
│   ├── index.html                      # HTML entry point with viewport and title metadata
│   ├── package.json                    # Frontend dependencies and dev/build scripts
│   ├── vite.config.js                  # Vite configuration and backend API proxy settings
│   ├── tailwind.config.js              # Noctelle dark violet color tokens and custom utilities
│   ├── postcss.config.js               # PostCSS configuration for Tailwind CSS
│   └── src/
│       ├── main.jsx                    # React application bootstrap and DOM mount
│       ├── App.jsx                     # Root application shell (Nav, persistent player, router)
│       ├── index.css                   # Global styles, Tailwind directives, glassmorphic classes
│       ├── context/
│       │   ├── AuthContext.jsx         # Manages user login/logout state and current session
│       │   └── PlayerContext.jsx       # Manages audio playback, current track, queue, and volume
│       ├── pages/
│       │   ├── LandingPage.jsx         # Public welcome screen with brand tagline and sign-in CTA
│       │   ├── LoginPage.jsx           # User sign-in form with email and password
│       │   ├── RegisterPage.jsx        # Account registration form with validation
│       │   ├── FeedPage.jsx            # Social feed displaying music posts from followed creators
│       │   ├── ExplorePage.jsx         # Music discovery hub for browsing tracks and genres
│       │   ├── PlaylistPage.jsx        # Playlist view, creator, and track management
│       │   ├── ProfilePage.jsx         # User profile displaying bio, playlists, and user posts
│       │   ├── SearchPage.jsx          # Unified search for tracks, artists, users, and playlists
│       │   └── AdminPage.jsx           # Admin control panel for moderation and platform metrics
│       ├── components/
│       │   ├── Navbar.jsx              # Top header with logo, search shortcut, and profile avatar
│       │   ├── Sidebar.jsx             # Left navigation links (Feed, Explore, Playlists, Profile)
│       │   ├── MusicPlayer.jsx         # Bottom persistent dock with track info, scrubber, and volume
│       │   ├── PostCard.jsx            # Feed item with attached playable track, likes, and comment toggle
│       │   ├── CommentSection.jsx      # Expandable comment list and input field for a post
│       │   ├── TrackItem.jsx           # Reusable track row with play button, duration, and playlist menu
│       │   └── NotificationDropdown.jsx # Dropdown list showing recent likes, comments, and follows
│       └── utils/
│           ├── api.js                  # Native fetch wrapper with base URL and cookie credentials
│           └── formatters.js           # Utilities for formatting audio duration (mm:ss) and dates
└── server/                             # Express REST API backend
    ├── package.json                    # Backend dependencies and startup scripts
    ├── .env.example                    # Template of required environment variables
    ├── server.js                       # Express app bootstrap, middleware, and database initialization
    ├── config/
    │   └── db.js                       # MongoDB connection logic and lifecycle event handlers
    ├── middleware/
    │   ├── auth.js                     # Validates JWT cookie and injects authenticated user into req.user
    │   ├── admin.js                    # Restricts route access to users with the 'admin' role
    │   └── errorHandler.js             # Centralized error handler returning clean JSON error responses
    ├── models/
    │   ├── User.js                     # Schema for user credentials, profile bio, avatar, and role
    │   ├── Track.js                    # Schema for cached music tracks (stream URL, title, artist, art)
    │   ├── Playlist.js                 # Schema for user playlists referencing Track documents
    │   ├── Post.js                     # Schema for social posts with text and attached Track reference
    │   ├── Comment.js                  # Schema for comments referencing parent Post and author User
    │   └── Notification.js             # Schema for activity alerts (likes, comments, new followers)
    ├── controllers/
    │   ├── authController.js           # Handles registration, login, logout, and self session checks
    │   ├── musicController.js          # Proxies external music searches and trending track requests
    │   ├── playlistController.js       # Handles playlist creation, editing, deletion, and track additions
    │   ├── postController.js           # Handles creating posts, fetching feed, and toggling likes
    │   ├── commentController.js        # Handles posting and removing comments on posts
    │   ├── userController.js           # Handles public profile lookup, bio edits, and follow toggling
    │   ├── notificationController.js   # Fetches unread alerts and marks notifications as read
    │   └── adminController.js          # Computes system statistics and provides moderation actions
    ├── routes/
    │   ├── authRoutes.js               # Endpoints for authentication (/api/auth)
    │   ├── musicRoutes.js              # Endpoints for music discovery and streaming (/api/music)
    │   ├── playlistRoutes.js           # Endpoints for playlist CRUD operations (/api/playlists)
    │   ├── postRoutes.js               # Endpoints for social feed posts and likes (/api/posts)
    │   ├── commentRoutes.js            # Endpoints for post comments (/api/posts/:postId/comments)
    │   ├── userRoutes.js               # Endpoints for user profiles and follows (/api/users)
    │   ├── notificationRoutes.js       # Endpoints for user notifications (/api/notifications)
    │   └── adminRoutes.js              # Protected endpoints for administration (/api/admin)
    └── services/
        └── musicService.js             # Server-side HTTP client querying the external Jamendo music API
```

---

### 4. Database Design

The database name is `noctelle`. Below are all collections, field types, relationship strategies, and index optimizations.

#### Collections & Schemas

##### 1. `users`
- `_id`: ObjectId (Primary key)
- `username`: String (required, unique, lowercase, trimmed)
- `email`: String (required, unique, lowercase, trimmed)
- `password`: String (required, bcrypt hash)
- `displayName`: String (required, display name)
- `bio`: String (default: "")
- `avatarUrl`: String (default: standard avatar URL)
- `role`: String (enum: `['user', 'admin']`, default: `'user'`)
- `followers`: `[ObjectId]` (References to `users` collection)
- `following`: `[ObjectId]` (References to `users` collection)
- `createdAt`, `updatedAt`: Date (automatic timestamps)

##### 2. `tracks` (Local cache of interacted/played tracks)
- `_id`: ObjectId (Primary key)
- `externalId`: String (required, unique; e.g. Jamendo track ID)
- `title`: String (required)
- `artistName`: String (required)
- `albumName`: String (default: "")
- `duration`: Number (duration in seconds)
- `audioUrl`: String (playable MP3 stream URL)
- `coverUrl`: String (album art image URL)
- `license`: String (Creative Commons license details)
- `createdAt`: Date (timestamp)

##### 3. `playlists`
- `_id`: ObjectId (Primary key)
- `userId`: ObjectId (required, Reference to `users`)
- `title`: String (required, trimmed)
- `description`: String (default: "")
- `coverUrl`: String (default: "")
- `tracks`: `[ObjectId]` (Ordered array of References to `tracks`)
- `isPublic`: Boolean (default: `true`)
- `createdAt`, `updatedAt`: Date (automatic timestamps)

##### 4. `posts`
- `_id`: ObjectId (Primary key)
- `userId`: ObjectId (required, Reference to `users` - author)
- `caption`: String (required, max 500 characters)
- `trackId`: ObjectId (optional, Reference to `tracks` - attached soundtrack)
- `likes`: `[ObjectId]` (Array of References to `users` who liked the post)
- `commentsCount`: Number (default: `0`, denormalized counter for fast feed rendering)
- `createdAt`, `updatedAt`: Date (automatic timestamps)

##### 5. `comments`
- `_id`: ObjectId (Primary key)
- `postId`: ObjectId (required, Reference to `posts`)
- `userId`: ObjectId (required, Reference to `users` - author)
- `content`: String (required, max 300 characters)
- `createdAt`: Date (timestamp)

##### 6. `notifications`
- `_id`: ObjectId (Primary key)
- `recipientId`: ObjectId (required, Reference to `users` - receives alert)
- `senderId`: ObjectId (required, Reference to `users` - triggered alert)
- `type`: String (enum: `['like', 'comment', 'follow']`, required)
- `postId`: ObjectId (optional, Reference to `posts` if related to post activity)
- `isRead`: Boolean (default: `false`)
- `createdAt`: Date (timestamp)

#### Relationship Strategy: Referenced vs Embedded

- **Referenced (`userId`, `trackId`, `postId`):**
  - *Why:* Playlists, Posts, Comments, and Notifications reference User and Track documents. If a track's audio stream URL or a user's avatar changes, it updates in ONE place without requiring updates across millions of embedded duplicates.
- **Separate Comments Collection (Referenced):**
  - *Why:* In MongoDB, documents have a strict 16MB size limit. If popular posts have thousands of comments, embedding comments directly inside the `posts` document causes unbounded array growth and slows down the main feed. Keeping comments in their own collection with a `postId` reference allows clean pagination.
- **Array of User IDs for `likes`:**
  - *Why:* An array of user IDs on each post allows fast checking if the current user already liked the post (`likes.includes(currentUserId)`) and allows atomic toggling using `$addToSet` and `$pull`.

#### Database Indexes & Performance Justification

Indexes are like book indexes: instead of scanning every document in a collection, MongoDB jumps straight to the record.
1. `users.username` (Unique index): Enables instant $O(1)$ lookup during login and guarantees no two users share a username.
2. `users.email` (Unique index): Guarantees email uniqueness and speeds up login lookups.
3. `tracks.externalId` (Unique index): Prevents duplicate cached tracks from the music API and enables instant cache hits.
4. `tracks.title` & `tracks.artistName` (Text index): Powers instant local search when users search their saved library.
5. `playlists.userId`: Allows instant fetching of all playlists owned by a user without scanning the entire collection.
6. `posts.createdAt` (`-1` descending): Crucial for sorting the social feed chronologically with zero in-memory sort penalty.
7. `posts.userId` + `posts.createdAt`: Optimizes fetching a user's post timeline on their profile page.
8. `comments.postId` + `comments.createdAt`: Loads comments for a specific post in chronological order instantly.
9. `notifications.recipientId` + `notifications.isRead`: Powers the notification bell badge to find unread alerts instantly.

---

### 5. Music Source Decision

#### Recommended Source: Jamendo Developer API
We recommend the **Jamendo API** as the primary legal music source.

- **Legality & Licensing:** Jamendo offers over 500,000 legal, Creative Commons-licensed full-length audio tracks. Independent artists upload music specifically for legal public listening and streaming.
- **Free-Tier Limits:** The Jamendo Developer free tier is completely free for non-commercial educational projects. It provides a generous rate limit (over 35,000 requests per 24 hours per client ID), far exceeding what our application needs.
- **Audio Reliability:** Unlike other APIs that return 30-second snippets or decentralized torrent-like streams, Jamendo returns high-quality direct MP3 stream URLs that play seamlessly in the native browser `<audio>` tag.

#### Keeping API Keys 100% Private (The Backend Proxy Pattern)
Under no circumstances will the Jamendo API client ID or secret be placed in the React client code.

```text
[ Browser / React ]  ──► GET /api/music/search?q=synthwave  ──► [ Express Backend ]
                                                                       │
                                                       (Reads JAMENDO_CLIENT_ID from .env)
                                                                       │
                                                                       ▼
                                                       GET https://api.jamendo.com/v3.0/tracks/
                                                       ?client_id=xxx&search=synthwave
                                                                       │
[ Browser / React ]  ◄── Returns clean sanitized JSON  ◄───────────────┘
```

1. The frontend calls our own Express endpoint: `GET /api/music/search?q=synthwave`.
2. Our backend service (`musicService.js`) reads the secret `JAMENDO_CLIENT_ID` from the server `.env` file.
3. Express calls Jamendo server-to-server, receives the raw payload, filters out unnecessary fields, and formats the response.
4. Express returns clean JSON (`title`, `artist`, `duration`, `audioUrl`, `coverUrl`) to React.
5. Anyone inspecting browser network tools will only ever see calls to our own domain (`/api/music/*`).

#### Song Storage Strategy: "Cache-on-Interaction"
We do **not** download gigabytes of audio files to our server, nor do we attempt to scrape Jamendo's 500,000 song database into MongoDB.
- When users browse or search music, tracks are displayed dynamically from the API.
- The moment a user **likes a track**, **adds it to a playlist**, or **attaches it to a post**, our backend checks if that `externalId` is already in our `tracks` collection.
- If it does not exist, we save a lightweight record (metadata, artwork URL, and stream URL) in our MongoDB `tracks` collection.
- Playlists and posts simply store the MongoDB `ObjectId` referencing that track. This gives us lightning-fast database relationships with near-zero storage costs.

---

### 6. Key Decisions with Tradeoffs

#### 1. JWT Storage: `httpOnly` Cookie vs `localStorage`
- **`localStorage` Tradeoff:** Easy to write (`localStorage.setItem('token', token)`), but highly insecure. Any malicious third-party script or Cross-Site Scripting (XSS) vulnerability can steal the token and impersonate the user.
- **`httpOnly` Cookie Tradeoff:** The server sets the cookie with the `HttpOnly; SameSite=Lax; Secure` flags. JavaScript running in the browser cannot read the cookie at all, making it immune to token theft via XSS. The browser automatically sends it with every request when `credentials: 'include'` is set.
- **Recommendation:** **`httpOnly` Cookie**. It provides production-grade security, simplifies frontend code (no manual `Authorization` headers needed on every fetch), and teaches proper real-world authentication practices.

#### 2. Profile Pictures & Cover Images: File Uploads vs Curated Image URLs
- **File Upload Tradeoff:** Uploading files to the backend server's local disk fails on free hosting platforms (like Render or Railway) because their file systems are *ephemeral* (any restart or redeploy wipes all uploaded files). Using AWS S3 or Cloudinary requires credit cards, complex SDKs, and storage fees.
- **Image URLs & DiceBear Fallback Tradeoff:** Users can provide an image link from the web (Unsplash, Pinterest, etc.). For default avatars, we use the free, zero-config **DiceBear API** (`https://api.dicebear.com/7.x/identicon/svg?seed=username`), which automatically generates a unique, gorgeous avatar based on the user's username.
- **Recommendation:** **Curated Image URLs with DiceBear Avatars**. It is 100% free, requires zero file storage, never breaks when deployed to free host containers, and provides a polished, beautiful user interface out of the box.

#### 3. Persistent Music Player Across Page Navigation
- In standard websites, clicking a link causes the browser to reload the page, immediately stopping audio playback.
- **Our Solution:** In our React Single Page Application, the `<MusicPlayer />` component is placed permanently at the bottom of the root `App.jsx` layout, outside the `<Routes>` container.
- Audio playback state (current track, play status, progress, queue) is managed by `PlayerContext.jsx`.
- When the user clicks between `/feed`, `/explore`, `/playlists`, or `/profile`, React Router swaps only the page content inside the `<main>` view. The music player stays mounted, and the music plays continuously without a single millisecond of interruption.

---

### 7. MVP Definition (10 Core Features)

To build a high-quality product without getting overwhelmed, we separate the 10 features into **Must Have (MVP)** and **Nice to Have**:

#### Must Have (MVP — The Core Experience)
1. **User Authentication:** User registration, login, secure `httpOnly` cookie session, and protected private routes.
2. **Music Streaming & Persistent Player:** Play, pause, seek, volume control, and continuous playback while navigating pages.
3. **Music Discovery & Search:** Browse trending tracks and search legal music by title, artist, or genre via Jamendo.
4. **Playlists:** Create custom playlists, add and remove tracks, and play an entire playlist.
5. **Social Feed with Music:** Create posts with text thoughts accompanied by an attached playable soundtrack.
6. **User Profiles:** View profile bio, avatar, user playlists, and shared posts.

#### Nice to Have (Post-MVP Enhancements)
7. **Follow System:** Follow/unfollow other music creators and filter the feed by followed users.
8. **Comments System:** Interactive discussion threads underneath each music post.
9. **In-App Notifications:** Notification bell alerting users when someone likes their post or follows them.
10. **Admin Moderation & Global Search:** Search across all site content simultaneously; admin dashboard to monitor system health and moderate inappropriate posts.

---

### 8. Development Roadmap (14 Phases)

Each phase is designed to be completed and verified independently before moving to the next:

#### Phase 1: Plan & Architecture
- **Goal:** Define the complete architectural blueprint, tech stack, data models, and styling rules.
- **Files Touched:** `README.md`
- **Verification:** `README.md` exists at the root, contains sections 1–9, and the mentor approves the plan.

#### Phase 2: Project Setup
- **Goal:** Initialize `/server` and `/client` folders, install exact dependencies, configure Vite, Tailwind CSS, and verify server/client boot.
- **Files Touched:** `client/package.json`, `client/vite.config.js`, `client/tailwind.config.js`, `client/src/index.css`, `server/package.json`, `server/.env.example`, `server/server.js`, `server/config/db.js`
- **Verification:** Running `npm run dev` in both folders boots the Express server connected to MongoDB on port 5000 and the Vite React app on port 5173 with dark violet styles active.

#### Phase 3: Auth Backend
- **Goal:** Build user registration, login, logout, and session check with hashed passwords and secure `httpOnly` JWT cookies.
- **Files Touched:** `server/models/User.js`, `server/middleware/auth.js`, `server/controllers/authController.js`, `server/routes/authRoutes.js`
- **Verification:** Tested via curl/ThunderClient: registering creates a user in MongoDB with a hashed password; logging in sets an `httpOnly` cookie; calling `/api/auth/me` returns the authenticated user data.

#### Phase 4: Auth Frontend & App Shell
- **Goal:** Build the global `AuthContext`, Landing page, Login page, Register page, protected route wrapper, and the navigation shell (`Navbar` and `Sidebar`).
- **Files Touched:** `client/src/context/AuthContext.jsx`, `client/src/utils/api.js`, `client/src/pages/LandingPage.jsx`, `client/src/pages/LoginPage.jsx`, `client/src/pages/RegisterPage.jsx`, `client/src/components/Navbar.jsx`, `client/src/components/Sidebar.jsx`, `client/src/App.jsx`
- **Verification:** User can sign up, log in, see their username in the navbar, refresh the page while staying logged in, and log out cleanly.

#### Phase 5: Music Data & Discovery
- **Goal:** Implement the Jamendo API proxy service on the backend and create the Explore page on the frontend to browse and search songs.
- **Files Touched:** `server/services/musicService.js`, `server/controllers/musicController.js`, `server/routes/musicRoutes.js`, `client/src/pages/ExplorePage.jsx`, `client/src/components/TrackItem.jsx`
- **Verification:** Visiting `/explore` displays real trending tracks with cover art; typing in the search bar displays matching songs from Jamendo without exposing any API keys in the browser.

#### Phase 6: Music Player
- **Goal:** Create `PlayerContext` and the persistent bottom `MusicPlayer` bar with play/pause, scrub slider, volume, and track queue management.
- **Files Touched:** `client/src/context/PlayerContext.jsx`, `client/src/components/MusicPlayer.jsx`, `client/src/App.jsx`, `client/src/utils/formatters.js`
- **Verification:** Clicking any track starts audio playback; the progress bar moves; seeking jumps the song; navigating from Explore to another page does not interrupt playback.

#### Phase 7: Playlists
- **Goal:** Build complete playlist CRUD: create a playlist, add tracks from Explore or Search, view playlist details, and delete a playlist.
- **Files Touched:** `server/models/Playlist.js`, `server/models/Track.js`, `server/controllers/playlistController.js`, `server/routes/playlistRoutes.js`, `client/src/pages/PlaylistPage.jsx`
- **Verification:** User can create a playlist titled "Midnight Vibes", add a song to it, view the playlist track list, and click "Play All" to stream the tracks in sequence.

#### Phase 8: Profiles & Follow
- **Goal:** Build public user profile pages, avatar and bio editing, and the follow/unfollow toggle.
- **Files Touched:** `server/controllers/userController.js`, `server/routes/userRoutes.js`, `client/src/pages/ProfilePage.jsx`
- **Verification:** Visiting `/profile/:username` shows the user's playlists, bio, and follower count; clicking "Follow" increments the follower counter instantly.

#### Phase 9: Social Feed
- **Goal:** Implement the music-sharing social feed where users create posts with text thoughts and attach a song, with a toggleable "Like" button.
- **Files Touched:** `server/models/Post.js`, `server/controllers/postController.js`, `server/routes/postRoutes.js`, `client/src/pages/FeedPage.jsx`, `client/src/components/PostCard.jsx`
- **Verification:** User creates a post attached to a song; the post appears on the feed with a playable track widget; clicking the heart icon increments the like count and persists on refresh.

#### Phase 10: Notifications
- **Goal:** Build notification triggers for likes, comments, and new followers, plus a dropdown menu in the navbar showing unread notifications.
- **Files Touched:** `server/models/Notification.js`, `server/controllers/notificationController.js`, `server/routes/notificationRoutes.js`, `client/src/components/NotificationDropdown.jsx`
- **Verification:** When User A likes User B's post, User B sees a notification badge on the bell icon and can mark it as read.

#### Phase 11: Global Search
- **Goal:** Unified search interface querying tracks, artists, users, and playlists from a single search bar.
- **Files Touched:** `client/src/pages/SearchPage.jsx`, `server/controllers/musicController.js`, `server/controllers/userController.js`
- **Verification:** Typing a query into the global search returns categorized result tabs: "Tracks", "Artists", "People", and "Playlists".

#### Phase 12: Admin Dashboard
- **Goal:** Build an administrative panel for users with the `admin` role to view platform statistics and moderate posts or users.
- **Files Touched:** `server/middleware/admin.js`, `server/controllers/adminController.js`, `server/routes/adminRoutes.js`, `client/src/pages/AdminPage.jsx`
- **Verification:** An admin user can access `/admin`, view total registered users and posts, and delete inappropriate posts. Standard users are denied access with a 403 Forbidden error.

#### Phase 13: Polish & Animations
- **Goal:** Add subtle CSS transitions, glassmorphic glows, skeleton loading states, and responsive mobile navigation drawers.
- **Files Touched:** `client/src/index.css`, `client/src/components/*`
- **Verification:** The application feels silky smooth; buttons have subtle hover glows; track lists show animated skeleton placeholders while loading; UI scales cleanly on mobile viewports.

#### Phase 14: Deployment & Final README
- **Goal:** Prepare production build scripts, create environment deployment guides, and document setup steps for free hosting (Render / Railway / Vercel).
- **Files Touched:** `README.md`, `server/server.js`, `client/vite.config.js`
- **Verification:** Both frontend and backend build cleanly with zero errors (`npm run build`), and the production app runs seamlessly in an isolated environment.

---

### 9. Design System Preview

Noctelle's visual identity captures a tranquil, late-night music sanctuary: **midnight backgrounds with soft violet luminescence**.

#### Color Tokens
- **Background Deep (`bg-base`):** `#0B0813` (Deep midnight black with a hint of purple)
- **Surface Dark (`bg-surface`):** `#130F23` (Dark midnight slate for sidebars and headers)
- **Surface Card (`bg-card`):** `#1D1735` (Elevated card background)
- **Surface Glass (`bg-glass`):** `rgba(29, 23, 53, 0.65)` (Translucent violet-tinted glass)
- **Primary Accent (`accent-primary`):** `#8B5CF6` (Vibrant electric violet)
- **Primary Glow / Hover (`accent-hover`):** `#A78BFA` (Soft glowing lavender)
- **Secondary Neon (`accent-secondary`):** `#C084FC` (Neon purple/magenta for highlights and active audio waves)
- **Text Primary (`text-primary`):** `#F8FAFC` (Crisp white for headings and active titles)
- **Text Secondary (`text-secondary`):** `#94A3B8` (Muted cool gray for artist names and body text)
- **Text Muted (`text-muted`):** `#64748B` (Subtle gray for timestamps and inactive icons)
- **Border Glass (`border-glass`):** `rgba(139, 92, 246, 0.18)` (Ultra-thin glowing border)
- **Border Active (`border-active`):** `rgba(139, 92, 246, 0.45)` (Highlighted border for inputs and active cards)
- **Status Success:** `#10B981` (Emerald green for active playback indicators)
- **Status Danger:** `#EF4444` (Rose red for errors and delete actions)

#### Typography
- **Primary Font:** `Inter`, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif.
- **Monospace / Numbers:** `ui-monospace`, "JetBrains Mono", SFMono-Regular, monospace (used for audio timecodes like `03:42`).
- **Hierarchy:**
  - Display / Hero: `2.25rem` (36px), ExtraBold, tracking tight
  - Section Headings: `1.5rem` (24px), SemiBold
  - Card Titles: `1.125rem` (18px), Medium
  - Body Text: `0.9375rem` (15px), Regular
  - Meta & Captions: `0.75rem` (12px), Medium

#### Spacing & Border Radius
- **Spacing Scale:** Standard 4px grid (`4px`, `8px`, `12px`, `16px`, `24px`, `32px`, `48px`).
- **Border Radius:**
  - Standard Inputs & Buttons: `8px` (`rounded-lg`)
  - Content Cards & Modals: `12px` (`rounded-xl`)
  - Badges, Pills & Floating Player: `9999px` (`rounded-full`)

#### Glassmorphism Rules
To achieve the signature Noctelle night-sky atmosphere:
```css
/* Glass Card Utility */
.glass-panel {
  background: rgba(29, 23, 53, 0.65);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(139, 92, 246, 0.18);
  box-shadow: 0 8px 32px 0 rgba(11, 8, 19, 0.4),
              0 0 16px 0 rgba(139, 92, 246, 0.08);
}

/* Violet Glow on Hover */
.violet-glow:hover {
  box-shadow: 0 0 20px rgba(139, 92, 246, 0.35);
}
```

---

*This document serves as our architectural agreement and foundation for Noctelle. No code or folder will be generated until this blueprint is confirmed.*

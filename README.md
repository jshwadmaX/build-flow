# ⚡ BuildFlow

> A low-code website builder for software engineering teams — drag, drop, configure, export.

BuildFlow is a full-stack web application that lets users visually construct websites by dragging pre-built components onto a canvas, configuring their properties in real-time, previewing the result live, and exporting clean HTML/CSS — all without writing a single line of code.

---

## ✨ Features

- **🖱️ Drag & Drop Editor** — Drag components from the sidebar onto a freeform canvas using `@dnd-kit`. Reposition elements by dragging.
- **🧩 Rich Component Library** — 7 ready-to-use components: `Text`, `Button`, `Image`, `Card`, `Hero`, `Navbar`, `Footer`.
- **⚙️ Live Property Panel** — Select any component to edit its text, colors, font sizes, links, and more in a right-side panel with instant visual feedback.
- **📄 Multi-Page Projects** — Each project supports multiple pages. Add pages from within the editor and switch between them seamlessly.
- **💾 Auto-Save** — Canvas state is automatically saved to MongoDB via a `bulk-save` API call whenever a change is made.
- **👁️ Live Preview** — Open a full rendered preview of any page in a new tab at any time.
- **📤 HTML Export** — Generate and download a self-contained, production-ready `index.html` file with embedded CSS for any page.
- **🔐 Authentication** — Secure JWT-based registration and login. All projects and components are scoped to the authenticated user.
- **🛡️ Protected Routes** — The Dashboard and Editor are gated behind authentication; unauthenticated users are redirected to the landing page.

---

## 🏗️ Architecture

```
lowcode-builder/
├── client/                         # React + Vite frontend
│   └── src/
│       ├── App.jsx                 # Root router
│       ├── pages/
│       │   ├── Landing.jsx         # Public marketing / hero page
│       │   ├── Login.jsx           # Authentication — login form
│       │   ├── Register.jsx        # Authentication — registration form
│       │   ├── Dashboard.jsx       # Project list: create, open, delete
│       │   ├── Editor.jsx          # Main drag-and-drop editor
│       │   └── Preview.jsx         # Read-only live page preview
│       ├── components/
│       │   ├── Navbar.jsx          # Top navigation bar
│       │   ├── ProtectedRoute.jsx  # Auth guard for private routes
│       │   └── editor/
│       │       ├── LeftSidebar.jsx     # Component palette (drag sources)
│       │       ├── Canvas.jsx          # Drop target, renders placed components
│       │       ├── ComponentRenderer.jsx # Renders a single component on canvas
│       │       └── RightPanel.jsx      # Property editor for selected component
│       │   └── rendered/
│       │       └── BlockRenderer.jsx   # Renders components for the Preview page
│       ├── store/
│       │   └── useStore.js         # Zustand global state (auth + editor)
│       └── lib/
│           ├── api.js              # Axios instance with auth header injection
│           └── codeGenerator.js   # HTML/CSS export engine
│
└── server/                         # Node.js + Express backend
    ├── server.js                   # Entry point, MongoDB connection
    ├── middleware/
    │   └── auth.js                 # JWT verification middleware
    ├── models/
    │   ├── User.js                 # Mongoose User schema (bcrypt hashed passwords)
    │   ├── Project.js              # Mongoose Project schema (name, pages[])
    │   └── Component.js            # Mongoose Component schema (type, x, y, props…)
    └── routes/
        ├── auth.js                 # POST /register, POST /login
        ├── projects.js             # CRUD + page management for projects
        └── components.js           # CRUD + bulk-save for canvas components
```

---

## 🔄 User Flow

```
Landing Page
     │
     ▼
Register / Login  ──► JWT stored in localStorage
     │
     ▼
Dashboard  ──► Create / Open / Delete projects
     │
     ▼
Editor  ──────────────────────────────────────────────────────┐
  │                                                            │
  ├─ Left Sidebar: drag a component type onto the canvas       │
  ├─ Canvas: drop → component appears, drag to reposition      │
  ├─ Right Panel: click component → edit props live            │
  ├─ Auto-saves to MongoDB on every change (bulk-save)         │
  ├─ Add Pages / Switch Pages                                  │
  └─ Export HTML  or  Open Preview ────────────────────────────┘
                                           │
                                           ▼
                                    Preview Page
                                (rendered, read-only)
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend Framework | React 18, Vite 5 |
| Styling | Tailwind CSS 3 |
| Routing | React Router DOM v6 |
| State Management | Zustand v4 |
| Drag & Drop | `@dnd-kit/core`, `@dnd-kit/sortable`, `@dnd-kit/modifiers` |
| HTTP Client | Axios |
| Backend Framework | Node.js, Express 4 |
| Database | MongoDB + Mongoose 8 |
| Authentication | JWT (`jsonwebtoken`) + bcrypt (`bcryptjs`) |
| Dev Server | Nodemon |

---

## 🚀 Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) v18+
- [MongoDB](https://www.mongodb.com/) running locally **or** a [MongoDB Atlas](https://www.mongodb.com/atlas) connection string

---

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd lowcode-builder
```

---

### 2. Set Up the Backend

```bash
cd server
npm install
```

Create a `.env` file in the `server/` directory:

```env
# MongoDB connection string
MONGO_URI=mongodb://localhost:27017/lowcode

# JWT secret — change this to a long random string in production!
JWT_SECRET=your_super_secret_jwt_key_here

# Server port (default: 5000)
PORT=5000

# Frontend URL for CORS
CLIENT_URL=http://localhost:5173
```

Start the backend:

```bash
# Development (auto-restart on file changes)
npm run dev

# Production
npm start
```

Server starts at **`http://localhost:5000`**. MongoDB connection is logged on startup.

---

### 3. Set Up the Frontend

```bash
cd ../client
npm install
npm run dev
```

Frontend starts at **`http://localhost:5173`** and proxies `/api/*` requests to the backend automatically via Vite config.

---

## 📡 API Reference

Base URL: `http://localhost:5000/api`

### Authentication

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/auth/register` | ❌ | Register a new user |
| `POST` | `/auth/login` | ❌ | Login and receive JWT |

#### `POST /auth/register`
```json
// Request
{ "name": "Alice", "email": "alice@example.com", "password": "secret123" }

// Response 201
{ "token": "<jwt>", "user": { "id": "...", "name": "Alice", "email": "alice@example.com" } }
```

#### `POST /auth/login`
```json
// Request
{ "email": "alice@example.com", "password": "secret123" }

// Response 200
{ "token": "<jwt>", "user": { "id": "...", "name": "Alice", "email": "alice@example.com" } }
```

---

### Projects

All project routes require `Authorization: Bearer <token>`.

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/projects` | List all projects for the logged-in user |
| `POST` | `/projects` | Create a new project |
| `GET` | `/projects/:id` | Get a single project |
| `PUT` | `/projects/:id` | Update project name or pages array |
| `DELETE` | `/projects/:id` | Delete project and all its components |
| `POST` | `/projects/:id/pages` | Add a new page to a project |

#### `POST /projects`
```json
// Request
{ "name": "My Website" }

// Response 201
{ "_id": "...", "name": "My Website", "pages": [{ "id": "page-...", "name": "Page 1" }] }
```

---

### Components

All component routes require `Authorization: Bearer <token>`.

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/components?projectId=&pageId=` | Get all components for a page |
| `POST` | `/components/bulk-save` | Replace all components on a page (auto-save) |
| `POST` | `/components` | Create a single component |
| `PUT` | `/components/:id` | Update a component |
| `DELETE` | `/components/:id` | Delete a component |

#### `POST /components/bulk-save`
```json
// Request
{
  "projectId": "...",
  "pageId": "page-123",
  "components": [
    { "type": "Text", "x": 50, "y": 100, "width": 300, "height": 50, "props": { "text": "Hello World" } }
  ]
}

// Response 200  — array of saved component documents
```

---

## 🗄️ Database Schema

### User
```js
{
  name:      String   (required),
  email:     String   (required, unique),
  password:  String   (bcrypt hashed, minlength: 6),
  createdAt: Date,
  updatedAt: Date
}
```

### Project
```js
{
  userId:    ObjectId  → User,
  name:      String    (required),
  pages:     [{ id: String, name: String }],
  createdAt: Date,
  updatedAt: Date
}
```

### Component
```js
{
  projectId: ObjectId  → Project,
  pageId:    String,
  type:      Enum ['Text','Button','Image','Card','Hero','Navbar','Footer'],
  x:         Number    (canvas left offset, px),
  y:         Number    (canvas top offset, px),
  width:     Number    (px),
  height:    Number    (px),
  props:     Mixed     (component-specific properties),
  createdAt: Date,
  updatedAt: Date
}
```

---

## 🧩 Component Library

Each component type has a set of editable `props` surfaced in the Right Panel:

| Component | Key Props |
|-----------|-----------|
| **Text** | `text`, `fontSize`, `color` |
| **Button** | `label`, `bg` (background color), `fontSize`, `linkTo` (page ID) |
| **Image** | `src` (URL), `alt` |
| **Card** | `title`, `titleColor`, `body` |
| **Hero** | `title`, `titleColor`, `subtitle`, `cta` (button label), `bg` (gradient/color) |
| **Navbar** | `brand` (logo text), `links` (comma-separated) |
| **Footer** | `text`, `bg` |

---

## 📤 HTML Export

The `codeGenerator.js` produces a **self-contained, zero-dependency HTML file** with:

- Embedded CSS reset and component styles
- Absolute positioning matching the canvas layout pixel-perfectly
- A `navigateTo()` helper stub for buttons with page links
- Compatible with any static host (GitHub Pages, Netlify, Vercel, etc.)

```js
import { generateHTML, downloadCode } from './lib/codeGenerator';

const html = generateHTML(project, pageId, components);
downloadCode(html, 'index.html'); // triggers browser download
```

---

## 🌐 Pages

| Route | Page | Auth Required | Description |
|-------|------|:---:|-------------|
| `/` | Landing | ❌ | Marketing / hero page |
| `/login` | Login | ❌ | Sign in to your account |
| `/register` | Register | ❌ | Create a new account |
| `/dashboard` | Dashboard | ✅ | View and manage your projects |
| `/editor/:projectId` | Editor | ✅ | Drag-and-drop canvas editor |
| `/preview/:projectId` | Preview | ❌ | Live read-only page preview |

---

## ⚙️ Environment Variables

### Server (`server/.env`)

| Variable | Required | Default | Description |
|----------|:--------:|---------|-------------|
| `MONGO_URI` | No | `mongodb://localhost:27017/lowcode` | MongoDB connection string |
| `JWT_SECRET` | **Yes** | `supersecretkey` | Secret for signing JWT tokens — **change in production** |
| `PORT` | No | `5000` | Port the API server listens on |
| `CLIENT_URL` | No | `http://localhost:5173` | Allowed CORS origin |

---

## 📦 Dependencies

### Backend
| Package | Version | Purpose |
|---------|---------|---------|
| `express` | ^4.18.2 | HTTP server framework |
| `mongoose` | ^8.0.3 | MongoDB ODM |
| `jsonwebtoken` | ^9.0.2 | JWT creation & verification |
| `bcryptjs` | ^2.4.3 | Password hashing |
| `cors` | ^2.8.5 | Cross-origin resource sharing |
| `dotenv` | ^16.3.1 | Environment variable loading |
| `nodemon` | ^3.0.2 | Dev auto-restart |

### Frontend
| Package | Version | Purpose |
|---------|---------|---------|
| `react` | ^18.2.0 | UI library |
| `react-router-dom` | ^6.21.0 | Client-side routing |
| `zustand` | ^4.4.7 | Lightweight global state |
| `@dnd-kit/core` | ^6.3.1 | Drag and drop primitives |
| `@dnd-kit/sortable` | ^8.0.0 | Sortable drag lists |
| `@dnd-kit/modifiers` | ^9.0.0 | Drag constraints |
| `axios` | ^1.6.2 | HTTP client |
| `tailwindcss` | ^3.4.0 | Utility-first CSS |
| `vite` | ^5.0.8 | Build tool and dev server |

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "feat: add your feature"`
4. Push and open a Pull Request

---

## 📄 License

This project is licensed under the ISC License.

---

<div align="center">
  <p>Built with ❤️ using React, Express, MongoDB, and @dnd-kit</p>
</div>

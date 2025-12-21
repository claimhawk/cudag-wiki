# Project Overview: `sl/mole-inference-www`

---

## 1. Overview

`sl/mole-inference-www` is a **Next.js web application** serving as the **frontend interface for a machine learning inference service**, likely for molecular property prediction or chemical structure analysis (inferred from the name "mole-inference"). The application provides users with a web-based UI to upload molecular structures (e.g., images or files), initiate inference requests, and view results — possibly for research, drug discovery, or chemical engineering workflows.

The project is built with **Next.js App Router** (v13+), leveraging server components, client components, and API routes for full-stack functionality. It includes authentication, server status monitoring, and file upload handling via a dropzone component. The backend inference logic is likely handled by a separate service (e.g., `proxy.ts` or `api.ts`), which this frontend communicates with via API routes.

---

## 2. Architecture

The project follows a **standard Next.js App Directory structure** with clear separation of concerns:

```
src/
├── app/                  # Next.js App Router pages and layout
│   ├── layout.tsx        # Root layout component (global styles, auth wrapper, etc.)
│   ├── page.tsx          # Home page (main UI for inference)
│   ├── login/page.tsx    # Authentication page
│   └── api/              # API routes (server-side endpoints)
│       └── auth/route.ts # Authentication API route
├── components/           # Reusable UI components
│   ├── ServerStatus.tsx  # Displays server health status
│   ├── ImageDropzone.tsx # File upload component (handles image uploads)
│   └── InferencePanel.tsx # Main inference UI panel (user interaction)
├── lib/                  # Shared utilities, types, and API clients
│   ├── api.ts            # API client for backend inference service
│   └── types.ts          # Shared type definitions (e.g., inference response)
├── proxy.ts              # Proxy layer for API requests (possibly for CORS or routing)
├── next.config.ts        # Next.js configuration (e.g., custom webpack, plugins)
├── vercel.json           # Vercel deployment configuration
├── package.json          # Project dependencies and scripts
├── tsconfig.json         # TypeScript configuration
└── system.md             # System-level documentation (e.g., deployment, architecture notes)
```

### Key Directories:

- **`src/app/`**: Contains Next.js pages and layout. The `layout.tsx` likely wraps all pages with authentication guards or global UI elements. The `page.tsx` is the main inference UI.
- **`src/components/`**: Reusable UI components. `ImageDropzone.tsx` handles file uploads, `InferencePanel.tsx` manages inference workflow, and `ServerStatus.tsx` monitors backend health.
- **`src/lib/`**: Shared logic. `api.ts` likely contains fetch wrappers or Axios clients for calling inference APIs. `types.ts` defines interfaces for API responses.
- **`src/proxy.ts`**: May act as a reverse proxy or API gateway to route requests to backend services (e.g., for handling CORS, authentication, or service discovery).
- **`src/app/api/`**: Next.js API routes — `auth/route.ts` handles login/logout or token validation.

---

## 3. Key Components

### `src/components/ImageDropzone.tsx`
- **Purpose**: Enables users to drag-and-drop image files (likely molecular structures like PNG, SVG, or SMILES images).
- **Functionality**: Validates file types, uploads to backend via `fetch` or `axios`, and triggers inference.
- **Dependencies**: Likely uses `react-dropzone` or similar library for drag-and-drop handling.

### `src/components/InferencePanel.tsx`
- **Purpose**: Main UI component for inference workflow.
- **Functionality**: Displays upload status, inference progress, results (e.g., predicted properties, confidence scores), and error handling.
- **State Management**: Likely uses React state or context to manage inference state (e.g., `isProcessing`, `result`, `error`).

### `src/components/ServerStatus.tsx`
- **Purpose**: Monitors the health of the backend inference service.
- **Functionality**: Fetches `/health` endpoint (likely defined in `api.ts` or `proxy.ts`) and displays status (e.g., “Online”, “Offline”, “Degraded”).
- **Use Case**: Ensures users are aware of backend availability before initiating inference.

### `src/lib/api.ts`
- **Purpose**: Encapsulates HTTP requests to the backend inference service.
- **Functionality**: Defines functions like `uploadMolecule`, `getInferenceResult`, `checkServerStatus`.
- **Example Usage**:
  ```ts
  const result = await api.uploadMolecule(file, { model: "mole-1.0" });
  ```

### `src/app/api/auth/route.ts`
- **Purpose**: Handles authentication for protected routes.
- **Functionality**: Validates JWT tokens, manages session state, redirects to login if unauthorized.
- **Use Case**: Ensures only authenticated users can access inference features.

### `src/proxy.ts`
- **Purpose**: Acts as a proxy or middleware layer for API requests.
- **Functionality**: May handle:
  - CORS headers
  - Request routing (e.g., `/api/inference` → `http://backend:8080/inference`)
  - Authentication middleware
  - Rate limiting or caching
- **Use Case**: Decouples frontend from backend URL changes or adds security layers.

---

## 4. Data Flow

The data flow follows a **client → proxy → backend → client** pattern:

1. **User Uploads File** → `ImageDropzone.tsx` captures file.
2. **File Sent to Backend** → `api.ts` sends file to `/api/upload` (via `proxy.ts` if configured).
3. **Backend Processes Request** → Backend service (e.g., ML model) processes the file and returns result.
4. **Result Returned to Frontend** → `api.ts` receives result and updates state in `InferencePanel.tsx`.
5. **UI Updates** → `InferencePanel.tsx` renders result, confidence, or error messages.
6. **Server Status Checked** → `ServerStatus.tsx` periodically polls `/health` endpoint (via `api.ts`) and updates UI.

**Authentication Flow**:
- User visits `/login` → `login/page.tsx` renders login form.
- On submit → `auth/route.ts` validates credentials → returns JWT token.
- Token stored in `localStorage` or `cookies` → `layout.tsx` checks token and redirects to `/` if authenticated.

---

## 5. Getting Started

### Prerequisites

- Node.js 18+ (or LTS)
- npm, yarn, pnpm, or bun (any package manager)
- Basic knowledge of React, TypeScript, and Next.js

### Setup Steps

1. **Clone the repository**:
   ```bash
   git clone https://github.com/your-org/sl-mole-inference-www.git
   cd sl-mole-inference-www
   ```

2. **Install dependencies**:
   ```bash
   npm install
   # or
   yarn install
   # or
   pnpm install
   ```

3. **Start the development server**:
   ```bash
   npm run dev
   # or
   yarn dev
   # or
   pnpm dev
   # or
   bun dev
   ```

4. **Open the app**:
   Visit [http://localhost:3000](http://localhost:3000) in your browser.

5. **Login (if required)**:
   Navigate to `/login` and authenticate via the provided form. The app will redirect to the main inference page upon successful login.

6. **Test Inference**:
   - Drag and drop a molecular image (e.g., PNG or SVG) into the `ImageDropzone`.
   - Wait for the `InferencePanel` to display results.
   - Check `ServerStatus` to ensure backend is healthy.

### Useful Files to Explore

- `src/app/page.tsx`: Main UI for inference.
- `src/components/ImageDropzone.tsx`: File upload logic.
- `src/lib/api.ts`: Backend communication layer.
- `src/app/api/auth/route.ts`: Authentication logic.
- `src/proxy.ts`: Proxy configuration (if used).
- `vercel.json`: Deployment settings for Vercel.

---

## Additional Notes

- The project uses **TypeScript** for type safety and **Next.js App Router** for routing.
- **Geist font** is configured via `next/font` for consistent UI styling.
- **Deployment** is streamlined via Vercel (as per `vercel.json` and README).
- **System documentation** is available in `system.md` — review it for architecture, deployment, or backend integration details.

---

This project is designed for **rapid prototyping and deployment** of ML inference interfaces. It is extensible — new models, upload formats, or UI components can be added with minimal changes to the core architecture.
# Project Overview: `visualizer/frontend`

---

## 1. Overview

This project is a **Next.js-based frontend visualization application** designed to render and interact with complex data structures — likely from a backend or model inference pipeline — through intuitive UI components. Based on the presence of components like `AttentionHeatmap`, `LayerSelector`, and `AdapterSelector`, it appears to be a **machine learning model visualization tool**, possibly for transformer-based architectures, enabling users to:

- Upload images for analysis.
- Visualize attention maps or layer activations.
- Select different model layers or adapters for comparison.
- Interact with visual outputs in real-time.

The project solves the problem of **making complex model behaviors (e.g., attention weights, feature maps) accessible and interpretable to users without requiring deep technical knowledge** — bridging the gap between model developers and end-users or researchers.

---

## 2. Architecture

The project follows a **Next.js App Router** structure (v13+), with a clear separation between layout, page, and component layers.

### Key Directories & Files:

- `src/app/` — The core Next.js App Router directory.
  - `layout.tsx`: Root layout component that wraps all pages with shared UI (e.g., navigation, theme, global styles).
  - `page.tsx`: The main landing page — likely the entry point for the visualization interface.
- `src/components/` — Reusable UI components.
  - `ImageUpload.tsx`: Handles image input and upload logic.
  - `AttentionHeatmap.tsx`: Renders attention maps or heatmaps over image regions.
  - `LayerSelector.tsx`: Allows users to select which model layer to visualize.
  - `AdapterSelector.tsx`: Enables selection of different model adapters (e.g., LoRA, QLoRA, etc.).
- `next.config.ts`: Custom Next.js configuration (e.g., asset optimization, image handling, or environment variables).
- `next-env.d.ts`: Type definitions for the Next.js environment.
- `package.json`: Project dependencies and scripts.
- `tsconfig.json`: TypeScript configuration.
- `README.md`: This documentation (you’re reading it!).

> **Note**: The project uses TypeScript and is configured for Next.js 13+ App Router. It also leverages `next/font` for optimized font loading (Geist from Vercel).

---

## 3. Key Components

### `ImageUpload.tsx`
- **Purpose**: Manages image file selection and upload to the backend or for local processing.
- **Functionality**: Likely triggers model inference or visualization generation upon upload.
- **Dependencies**: May interact with `useState`, `useEffect`, or custom hooks for file handling.

### `AttentionHeatmap.tsx`
- **Purpose**: Renders attention weights as a heatmap overlay on the input image.
- **Functionality**: Accepts attention scores (e.g., from transformer layers) and visualizes them using color gradients.
- **Dependencies**: Likely uses `React`, `d3` or similar for rendering, and may accept props like `attentionMap`, `imageDimensions`.

### `LayerSelector.tsx`
- **Purpose**: Allows users to choose which layer of the model to visualize.
- **Functionality**: Dropdown or list component that updates the visualization state when a layer is selected.
- **Dependencies**: May connect to state management (e.g., `useState`, `useContext`, or Redux).

### `AdapterSelector.tsx`
- **Purpose**: Lets users switch between different model adapters (e.g., LoRA, QLoRA, etc.).
- **Functionality**: Updates model weights or inference pipeline based on selected adapter.
- **Dependencies**: May trigger re-fetching of model weights or re-initializing inference logic.

### `src/app/page.tsx`
- **Purpose**: The main UI entry point — orchestrates component rendering and state.
- **Functionality**: Likely contains:
  - Image upload form.
  - Layer and adapter selectors.
  - Heatmap display area.
  - State management for selected model, layer, adapter, and image.
- **Dependencies**: Uses `useState`, `useEffect`, and may call API endpoints for inference.

---

## 4. Data Flow

The data flow follows a **client-side UI → API → Model → Visualization** pattern:

1. **User Interaction**:
   - User uploads an image via `ImageUpload.tsx`.
   - User selects a layer and/or adapter via `LayerSelector.tsx` and `AdapterSelector.tsx`.

2. **State Management**:
   - State (e.g., selected layer, adapter, image) is managed in `page.tsx` using `useState` or context.

3. **Inference Trigger**:
   - On upload or selection change, `page.tsx` triggers an API call (likely to a backend endpoint or local inference service).

4. **Data Retrieval**:
   - Backend returns attention maps or layer activations (e.g., as JSON or base64-encoded image).

5. **Visualization Rendering**:
   - `AttentionHeatmap.tsx` receives the attention map data and renders it over the input image.
   - UI updates in real-time as state changes.

> **Note**: The project likely uses `fetch` or `useSWR`/`useQuery` for API calls. If the backend is local, it may use `fetch` or `axios`.

---

## 5. Getting Started

### Prerequisites

- Node.js 18+ (or LTS)
- npm, yarn, pnpm, or bun (any package manager)
- Basic knowledge of React, TypeScript, and Next.js

### Setup Steps

1. **Clone the repository** (if not already done).

2. **Install dependencies**:

   ```bash
   npm install
   # or
   yarn install
   # or
   pnpm install
   # or
   bun install
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

4. **Access the app**:

   Open [http://localhost:3000](http://localhost:3000) in your browser.

5. **Start Exploring**:

   - Upload an image via `ImageUpload.tsx`.
   - Use `LayerSelector.tsx` and `AdapterSelector.tsx` to explore different model behaviors.
   - Observe the `AttentionHeatmap.tsx` visualization update in real-time.

### Development Tips

- Modify `src/app/page.tsx` to change the UI or logic.
- Add new components to `src/components/` and import them in `page.tsx`.
- Customize `next.config.ts` for custom build settings.
- Use `tsconfig.json` to adjust TypeScript settings if needed.

---

## Additional Notes

- The project is built with **TypeScript**, so type safety is enforced.
- It uses **Next.js App Router**, so routing is handled via `app/` directory structure.
- The `layout.tsx` component likely provides global styling, theme, or navigation.
- If the project connects to a backend, ensure the API endpoints are configured (e.g., in `app/page.tsx` or a custom API route).

---

This project is ready for development, testing, and deployment. For deployment, use [Vercel](https://vercel.com) (as suggested in the original README) or any other Next.js-compatible platform.
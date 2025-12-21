# Annotator Project Overview

## 1. Overview

**Annotator** is a visual annotation tool designed for creating high-quality training data for Vision-Language Models (VLMs) to perform computer use tasks — specifically, UI grounding, interaction prediction, and task execution simulation. It enables users to annotate screenshots of desktop environments by defining elements (e.g., icons, text fields, buttons) with bounding boxes, labels, and masks, and to define tasks that involve interactions with those elements, including prior state conditions.

This tool is particularly tailored for fine-tuning models like **Qwen3-VL** to understand and interact with graphical user interfaces. The output is a ZIP archive containing:
- The original screenshot,
- An annotated version with bounding boxes overlaid,
- A masked version where dynamic content regions are blurred or replaced to preserve privacy and variability.

The core problem this tool solves is the lack of high-quality, structured, and diverse training data for VLMs to learn computer use — a task requiring precise visual grounding, semantic understanding, and contextual awareness of UI states.

---

## 2. Architecture

The project follows a **modular, TypeScript-based, Next.js application architecture** with clear separation of concerns between UI, state, utilities, services, and APIs.

### Key Directories and Their Purpose

- **`src/types/`**: Defines TypeScript interfaces for data structures used throughout the app.
  - `dataTypes.ts`, `fieldTypes.ts`, `annotation.ts`, `import.ts` — Define schema for annotations, elements, tasks, and import formats.
- **`src/utils/`**: Utility functions for common operations.
  - `elementTypeUtils.ts` — Handles element type logic (e.g., icon, text, button).
  - `gridUtils.ts` — Manages grid drawing and cell positioning.
  - `toleranceUtils.ts` — Handles coordinate tolerance for element matching.
- **`src/hooks/`**: React hooks for managing state and side effects.
  - `useToolState.ts`, `useAnnotationState.ts`, `useImageState.ts`, `useViewState.ts`, `useDataTypes.ts` — Manage UI state, annotation state, image loading, and view configuration.
- **`src/lib/`**: Core services and utilities not tied to React hooks.
  - `FilesystemService.ts` — Handles file I/O (loading/saving ZIPs).
  - `schema/SchemaInference.ts` — Infers schema from annotation data (e.g., auto-generate field types).
- **`src/services/`**: Business logic services.
  - `imageService.ts` — Image processing (resizing, cropping, overlays).
  - `exportService.ts` — Generates ZIP exports (original, annotated, masked).
  - `ocrService.ts` — Optional OCR for extracting text from images.
  - `cudagService.ts` — Likely GPU-accelerated operations (e.g., image processing or inference).
- **`src/components/ui/`**: Reusable UI components (e.g., palettes, panels, modals).
- **`src/app/api/`**: REST endpoints for server-side operations.
  - `ocr/route.ts`, `generators/route.ts`, `export/route.ts`, `import/route.ts`, `import/selective/route.ts` — Handle API requests for OCR, task generation, export, and selective import.

---

## 3. Key Components

### 3.1. Annotation State Management

- **`src/hooks/useAnnotationState.ts`** — Manages the current annotation state including selected elements, active tools, and task definitions.
- **`src/hooks/useToolState.ts`** — Controls active tool mode (e.g., `Single`, `Grid`) and tool-specific behavior.
- **`src/hooks/useImageState.ts`** — Manages loaded image state (dimensions, filename, file path).

### 3.2. Element and Grid System

- **`src/utils/gridUtils.ts`** — Draws and manages grid overlays (e.g., 11×20 grid for icon slots). Exposes methods for `drawGrid()`, `getCellPosition()`, and `getGridDimensions()`.
- **`src/utils/elementTypeUtils.ts`** — Defines element types (e.g., `Icon`, `Text`, `Button`) and their properties (label, coordinates, mask toggle, alignment).

### 3.3. Task Definition and Prior States

- **`src/types/annotation.ts`** — Defines `Task` and `Element` interfaces, including `priorStates` (e.g., `open`, `expanded`, `hasSelection`).
- **`src/services/exportService.ts`** — Serializes tasks and elements into export-ready formats.
- **`src/components/ui/TaskEditor.tsx`** — UI component for editing tasks (action type, target element, prompt, prior states).

### 3.4. Export Pipeline

- **`src/services/exportService.ts`** — Orchestrates export: generates annotated image (with bounding boxes), masked image (with dynamic regions blurred), and ZIP archive.
- **`src/lib/FilesystemService.ts`** — Handles file writing and ZIP creation.
- **`src/app/api/export/route.ts`** — Exposes `/api/export` endpoint to trigger export via API.

### 3.5. OCR and Schema Inference

- **`src/services/ocrService.ts`** — Uses OCR to extract text from annotated regions (optional, for auto-labeling).
- **`src/lib/schema/SchemaInference.ts`** — Infers schema from annotation data to auto-generate field types or validate structure.

---

## 4. Data Flow

The data flow is structured around **image loading → annotation → task definition → export**.

### Step 1: Image Loading

- User drops a screenshot or ZIP file.
- `src/hooks/useImageState.ts` loads the image and stores metadata (dimensions, filename).
- `src/services/imageService.ts` processes the image for display and annotation.

### Step 2: Annotation

- User selects tool mode (`Single` or `Grid`) via `src/hooks/useToolState.ts`.
- User draws elements (icons, text, buttons) using `src/utils/elementTypeUtils.ts` and `src/utils/gridUtils.ts`.
- Elements are stored in annotation state managed by `src/hooks/useAnnotationState.ts`.
- `src/services/imageService.ts` overlays bounding boxes on the image for visual feedback.

### Step 3: Task Definition

- User defines tasks via `src/components/ui/TaskEditor.tsx`.
- Tasks include:
  - Action type (e.g., `click`, `type`, `select`)
  - Target element (via ID or coordinates)
  - Prompt text
  - Prior states (e.g., `hasSelection`, `expanded`)
- Tasks are stored in `src/hooks/useAnnotationState.ts`.

### Step 4: Export

- User triggers export via UI or `/api/export` endpoint.
- `src/services/exportService.ts`:
  - Generates annotated image (bounding boxes drawn).
  - Generates masked image (dynamic regions blurred).
  - Uses `src/lib/FilesystemService.ts` to create ZIP.
- ZIP contains:
  - Original image
  - Annotated image
  - Masked image
- Exported data is ready for model training.

---

## 5. Getting Started

### Prerequisites

- Node.js 18+
- npm or yarn
- Next.js 14+ (app directory)

### Setup

1. **Clone the repository**:
   ```bash
   git clone <repository-url>
   cd annotator
   ```

2. **Install dependencies**:
   ```bash
   npm install
   ```

3. **Run development server**:
   ```bash
   npm run dev
   ```

   The app will be available at `http://localhost:3000`.

4. **Start API server** (if needed):
   The API routes are auto-generated by Next.js. You can test endpoints via:
   ```bash
   curl http://localhost:3000/api/export
   ```

### Key Development Files to Explore

- **`src/app/api/export/route.ts`** — Entry point for export API.
- **`src/services/exportService.ts`** — Core logic for generating export ZIP.
- **`src/hooks/useAnnotationState.ts`** — State management for annotations.
- **`src/utils/gridUtils.ts`** — Grid drawing logic.
- **`src/components/ui/TaskEditor.tsx`** — UI for task definition.

### Testing

- Use `src/app/api/ocr/route.ts` to test OCR functionality.
- Use `src/app/api/import/route.ts` to test import functionality.
- Test export with sample screenshots in `public/` (or via file upload).

---

This project is designed for developers who want to build, extend, or fine-tune VLMs for computer use tasks. Its modular architecture and clear separation of concerns make it easy to extend or modify specific components (e.g., add new element types, integrate with different OCR engines, or customize export formats).
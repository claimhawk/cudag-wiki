```markdown
# Project Overview: Annotator

## 1. Overview

**Annotator** is a visual annotation tool designed for creating high-quality, structured training data for Vision-Language Models (VLMs) — specifically for fine-tuning models like Qwen3-VL to perform computer use tasks such as UI grounding, interaction prediction, and task-based instruction following.

The tool enables users to:
- Load desktop screenshots or previously exported ZIPs.
- Annotate UI elements (e.g., icons, text fields, buttons) with bounding boxes, labels, and masks.
- Define spatial grids for structured UI layouts (e.g., 11×20 grid for desktop icon slots).
- Configure “prior states” for elements (e.g., “open”, “expanded”, “has selection”) to generate diverse training examples.
- Export annotated data in three formats:
  - Original screenshot
  - Annotated screenshot with bounding box overlays
  - Masked version with dynamic content regions blurred or filled

This solves the problem of manually curating large-scale, diverse, and context-aware UI interaction datasets — critical for training VLMs to understand and interact with real-world desktop environments.

---

## 2. Architecture

The project is built using **Next.js 14+** with **TypeScript**, following a component-based, hook-driven architecture. The codebase is organized into logical modules:

### Core Directories

- **`src/types/`**: Defines TypeScript interfaces for data structures.
  - `dataTypes.ts` — Defines global data types (e.g., `ElementType`, `Task`, `AnnotationState`)
  - `annotation.ts` — Core annotation schema (elements, tasks, grids)

- **`src/utils/`**: Utility functions for core operations.
  - `elementTypeUtils.ts` — Element type validation and conversion
  - `gridUtils.ts` — Grid generation, cell positioning, and dimension calculations
  - `toleranceUtils.ts` — Tolerance calculations for element matching (e.g., click region fuzziness)
  - `index.ts` — Re-exports all utilities for easy import

- **`src/hooks/`**: React hooks for state management.
  - `useToolState.ts` — Manages current tool mode (e.g., draw, edit, grid)
  - `useAnnotationState.ts` — Manages annotation canvas state (selected elements, active tasks)
  - `useDataTypes.ts` — Manages available element types and their configurations
  - `useViewState.ts` — Manages UI view state (zoom, pan, layer visibility)
  - `useImageState.ts` — Manages loaded image metadata and canvas rendering
  - `index.ts` — Re-exports all hooks

- **`src/services/`**: Business logic and API services.
  - `exportService.ts` — Handles ZIP export logic (original, annotated, masked)
  - `cudagService.ts` — Likely handles “CUDA”-related logic (possibly GPU-accelerated annotation or inference — context unclear)
  - `chandraService.ts` — Possibly a legacy or placeholder service (name suggests “Chandra” — may be a misnomer or placeholder)
  - `index.ts` — Re-exports all services

- **`src/app/`**: Next.js application routes and layout.
  - `layout.tsx` — Root layout component (provides global styles, navigation, and context providers)
  - `page.tsx` — Main app component (entry point for the annotator UI)
  - `api/generators/` — API routes for saving/loading annotations and configurations
    - `save/route.ts`, `load/route.ts` — Save/load annotation state from server
    - `save-config/route.ts` — Save configuration (e.g., grid settings, element types)
    - `route.ts` — Likely a generic generator route (possibly for model inference or data generation)

- **`src/components/`**: Reusable UI components.
  - `AnnotationCanvas.tsx` — Main canvas for drawing and editing elements
  - `ElementList.tsx` — List of annotated elements with edit controls
  - `TaskList.tsx` — List of tasks with prior state configuration
  - `DataTypesModal.tsx` — Modal for selecting or configuring element types
  - `GenerateButton.tsx` — Button to trigger export or generation
  - `index.ts` — Re-exports all components

---

## 3. Key Components

### Core State Management

- **`useAnnotationState.ts`** — Maintains the current annotation state including:
  - Selected elements
  - Active task
  - Current tool mode (draw, edit, grid)
  - Canvas transformation state (zoom, pan)

- **`useToolState.ts`** — Manages tool selection (e.g., “draw”, “grid”, “select”) and triggers tool-specific behaviors.

### Annotation Canvas

- **`AnnotationCanvas.tsx`** — The central UI component. Renders the image, overlays elements, and handles mouse/touch interactions for drawing and editing.

### Element and Task Management

- **`ElementList.tsx`** — Displays all annotated elements with edit controls (label, coordinates, mask toggle, alignment).
- **`TaskList.tsx`** — Displays tasks with action type, target element, prompt, and prior states (e.g., “open”, “has selection”).

### Export System

- **`exportService.ts`** — Orchestrates ZIP export:
  - Combines original image, annotated image (with bounding boxes), and masked image (with dynamic content replaced).
  - Uses `gridUtils.ts` to calculate grid positions for annotation.
  - Uses `elementTypeUtils.ts` to validate element types and configurations.

### Grid System

- **`gridUtils.ts`** — Generates and manages grid annotations (e.g., 11×20 grid for desktop icons).
  - Calculates cell positions and dimensions.
  - Supports configurable grid dimensions.

### Data Types

- **`dataTypes.ts`** — Defines the schema for element types (e.g., `Icon`, `TextBox`, `Button`, `DateTime`).
- **`useDataTypes.ts`** — Provides hooks to manage available element types and their properties.

---

## 4. Data Flow

1. **User Loads Image** → `useImageState.ts` updates image metadata (dimensions, filename) → `AnnotationCanvas.tsx` renders image.

2. **User Selects Tool** → `useToolState.ts` updates tool mode → triggers tool-specific logic (e.g., draw mode activates canvas drawing).

3. **User Draws Element** → `AnnotationCanvas.tsx` captures mouse/touch coordinates → `useAnnotationState.ts` adds element to annotation state.

4. **User Edits Element** → `ElementList.tsx` displays element → user modifies label, coordinates, mask, alignment → `useAnnotationState.ts` updates element.

5. **User Configures Task** → `TaskList.tsx` displays task → user sets action, target, prompt, prior states → `useAnnotationState.ts` stores task.

6. **User Exports** → `GenerateButton.tsx` triggers `exportService.ts` → generates ZIP with:
   - Original image
   - Annotated image (with bounding boxes)
   - Masked image (with dynamic content replaced)

7. **Data Persistence** → `save/route.ts` and `load/route.ts` handle saving/loading annotation state to/from server (likely for collaborative or versioned workflows).

---

## 5. Getting Started

### Prerequisites

- Node.js 18+ (or LTS)
- Yarn or npm
- Next.js 14+ (with App Router)

### Setup

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd annotator
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Start the development server:
   ```bash
   npm run dev
   ```

4. Open `http://localhost:3000` in your browser.

### Key Files to Explore

- `src/app/page.tsx` — Entry point for the app.
- `src/components/AnnotationCanvas.tsx` — Core UI component.
- `src/hooks/useAnnotationState.ts` — Core state management.
- `src/services/exportService.ts` — Export logic.
- `src/utils/gridUtils.ts` — Grid annotation logic.

### Development Tips

- Use `src/hooks/useToolState.ts` to toggle between draw/edit/grid modes.
- Use `src/components/ElementList.tsx` to inspect and edit annotated elements.
- Use `src/components/TaskList.tsx` to configure interaction tasks with prior states.
- Use `src/services/exportService.ts` to test export functionality.

### Testing

- Run unit tests (if available):
  ```bash
  npm test
  ```

- Run component tests:
  ```bash
  npm run test:component
  ```

---

## Notes

- The project appears to be in active development or recently refactored — some files (e.g., `chandraService.ts`, `cudagService.ts`) may be placeholders or legacy code.
- The `next-env.d.ts` and `next.config.ts` files are standard Next.js configuration files.
- The project uses TypeScript with strict typing — ensure all interfaces are correctly implemented.
- The API routes (`/api/generators/...`) suggest server-side state persistence — useful for collaborative annotation or model training pipelines.

---
```

This overview provides a comprehensive, technical understanding of the `annotator` project, suitable for documentation, onboarding, or architectural review.
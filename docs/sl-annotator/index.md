# Project Overview: `sl/annotator`

---

## 1. Overview

**Project Name**: `sl/annotator`  
**Purpose**: A visual annotation tool designed to generate high-quality, structured training data for Vision-Language Models (VLMs) — specifically for fine-tuning models like Qwen3-VL to perform computer use tasks such as UI grounding, interaction prediction, and task-based reasoning.

**Problem Solved**:  
Vision-Language Models require large-scale, annotated datasets that capture not only visual regions but also their semantic context, interaction affordances, and dynamic states (e.g., “expanded”, “selected”, “focused”). Manual annotation of UI elements and their associated tasks is time-consuming and error-prone. This tool automates and streamlines the creation of such datasets by enabling users to:

- Load desktop screenshots.
- Annotate UI elements with bounding boxes, labels, and masks.
- Define interaction tasks with target elements, prompts, and prior states (e.g., “when the login dialog is open and the username field has focus”).
- Export annotated data in standardized formats (original, annotated, masked) for model training.

The tool is built as a web-based application using Next.js, leveraging TypeScript for type safety and modular architecture.

---

## 2. Architecture

The project follows a **modular, component-driven architecture** with clear separation of concerns. The codebase is organized into logical layers:

### Key Directories:

- **`src/types/`**: Defines TypeScript interfaces and type definitions for data structures used across the app.
  - `dataTypes.ts`, `fieldTypes.ts`, `annotation.ts`, `import.ts` — Define core data models for annotations, fields, imports, and element metadata.
  - These types are consumed by hooks, services, and components to ensure type safety.

- **`src/utils/`**: Contains utility functions for common operations.
  - `elementTypeUtils.ts` — Utility functions for managing element types (e.g., `text`, `button`, `grid`).
  - `gridUtils.ts` — Logic for generating, validating, and manipulating grid annotations (e.g., 11×20 grid for icon slots).
  - `toleranceUtils.ts` — Helper functions for coordinate tolerance and snapping.
  - `index.ts` — Re-exports utilities for easy import.

- **`src/hooks/`**: React hooks for managing state and side effects.
  - `useToolState.ts`, `useAnnotationState.ts`, `useImageState.ts`, `useViewState.ts`, `useDataTypes.ts` — Manage UI state, annotation state, image loading, view modes, and data type configurations.
  - These hooks are used by components to react to user interactions and state changes.

- **`src/lib/`**: Core services and domain logic.
  - `FilesystemService.ts` — Handles file I/O (loading, saving, exporting).
  - `schema/` — Contains schema inference logic (`SchemaInference.ts`) for auto-detecting element types from annotations.
  - `index.ts` — Re-exports core library modules.

- **`src/services/`**: Business logic and external service integrations.
  - `imageService.ts` — Handles image loading, resizing, and preprocessing.
  - `ocrService.ts` — Integrates with OCR APIs to extract text from annotated regions.
  - `exportService.ts` — Manages ZIP export logic (original, annotated, masked).
  - `cudagService.ts` — Likely handles GPU-accelerated operations (e.g., for image processing or OCR).
  - `index.ts` — Re-exports services for easy consumption.

- **`src/components/ui/`**: Reusable UI components.
  - `index.ts` — Re-exports UI components (e.g., palettes, modals, panels).

- **`src/app/api/`**: REST endpoints for backend services.
  - `ocr/route.ts`, `generators/route.ts`, `export/route.ts`, `import/route.ts`, `import/selective/route.ts` — Expose API endpoints for OCR, annotation generation, export, and selective import.

---

## 3. Key Components

### 1. **Annotation System**

- **Core Data Model**: Defined in `src/types/annotation.ts` and `src/types/dataTypes.ts`.
  - Represents elements (e.g., `button`, `textbox`, `grid`) with properties: `label`, `coordinates`, `mask`, `alignment`, `elementId`.
  - Tasks are defined as `{ actionType, targetElement, prompt, priorStates }` — stored in `src/types/fieldTypes.ts`.

### 2. **Grid Annotation System**

- **Logic**: Implemented in `src/utils/gridUtils.ts`.
  - Generates grid layouts (e.g., 11×20) with configurable dimensions.
  - Supports element numbering and coordinate mapping.
  - Used in the “Grid Annotation” state shown in screenshots.

### 3. **Element Editing Panel**

- **State Management**: `src/hooks/useAnnotationState.ts` and `src/hooks/useToolState.ts` manage selected elements and their properties.
- **UI**: Components in `src/components/ui/` render editing controls for label, coordinates, mask toggle, and alignment.

### 4. **Task Editor with Prior States**

- **Data Model**: `src/types/fieldTypes.ts` defines task structure.
- **Logic**: `src/services/exportService.ts` and `src/hooks/useToolState.ts` handle task validation and state persistence.
- **UI**: Task panel renders action type, target element, prompt, and prior states (e.g., “open”, “selected”, “focused”).

### 5. **Export System**

- **Core Logic**: `src/services/exportService.ts`.
  - Generates ZIP files containing:
    - Original screenshot.
    - Annotated image with bounding box overlays.
    - Masked image with dynamic content regions filled (e.g., password fields blurred).
  - Uses `FilesystemService.ts` for file writing.

### 6. **OCR Integration**

- **Service**: `src/services/ocrService.ts`.
  - Integrates with external OCR APIs to extract text from annotated regions.
  - Used to auto-generate text labels or validate annotations.

---

## 4. Data Flow

The system follows a **state-driven, event-based data flow**:

1. **User Interaction → Hooks → State Updates**
   - User clicks “Load Image” → `useImageState.ts` updates image state.
   - User selects “Grid Mode” → `useToolState.ts` updates tool mode.
   - User draws an element → `useAnnotationState.ts` adds element to annotation list.

2. **State → Components → UI Rendering**
   - Components (e.g., `src/components/ui/`) consume state from hooks to render UI (e.g., element palette, editing panel, task editor).

3. **Annotation State → Services → Export**
   - When user clicks “Export”, `useToolState.ts` triggers `exportService.ts`.
   - `exportService.ts` generates annotated and masked images using `imageService.ts` and `FilesystemService.ts`.
   - ZIP is created and downloaded.

4. **OCR → Annotation State**
   - When OCR is triggered (e.g., via API endpoint), `ocrService.ts` processes image regions.
   - Results are merged into annotation state via `useAnnotationState.ts`.

5. **API Endpoints → Services → External Systems**
   - `/api/ocr`, `/api/export`, `/api/import` endpoints are exposed via `src/app/api/` routes.
   - These routes call corresponding services (e.g., `ocrService.ts`, `exportService.ts`) to handle requests.

---

## 5. Getting Started

### Prerequisites

- Node.js 18+ (for Next.js 14+)
- Yarn or npm
- Optional: Docker (for local development with services)

### Setup

1. **Clone the repository**:
   ```bash
   git clone https://github.com/your-org/sl/annotator.git
   cd sl/annotator
   ```

2. **Install dependencies**:
   ```bash
   npm install
   ```

3. **Start the development server**:
   ```bash
   npm run dev
   ```

   The app will be available at `http://localhost:3000`.

### Key Development Commands

- `npm run build` — Build for production.
- `npm run export` — Manually trigger export logic (for testing).
- `npm run ocr` — Run OCR service locally (if configured).
- `npm run test` — Run unit/integration tests.

### Key Files to Explore

- `src/app/page.tsx` — Main app entry point.
- `src/hooks/useAnnotationState.ts` — Core state management for annotations.
- `src/services/exportService.ts` — Export logic.
- `src/utils/gridUtils.ts` — Grid annotation logic.
- `src/app/api/export/route.ts` — Export API endpoint.

### Development Tips

- Use `src/types/` to understand data models.
- Use `src/hooks/` to understand state management.
- Use `src/services/` to understand business logic.
- Use `src/app/api/` to understand API endpoints.

---

This project is designed for developers familiar with React, TypeScript, and Next.js. It is extensible — new element types, export formats, or OCR integrations can be added by extending the type definitions, services, and hooks.
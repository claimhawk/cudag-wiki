```markdown
1. **Purpose**  
This file deploys a Gradio-based OCR web interface via Modal, allowing users to upload images and extract structured text with timing metrics. It acts as a frontend proxy that sends image data to a backend OCR inference service and displays results in a browser-accessible UI.

2. **Key Components**  
- `web()`: Modal function that hosts the Gradio app, processes image uploads, and forwards them to an OCR endpoint.  
- `run_ocr()`: Core OCR inference handler that encodes images to base64, sends them to the backend, and returns text + timing data.  
- `demo`: Gradio Interface object defining input/output schema and UI layout.

3. **Algorithm Notes**  
- Uses base64 encoding for image transmission to the backend.  
- Implements error handling for timeouts and request failures.  
- Aggregates inference timing metrics from backend response for user feedback.

4. **Dependencies**  
- `modal`: For cloud deployment and container management.  
- `gradio`: For building the web UI.  
- `requests`: For HTTP communication with OCR backend.  
- `Pillow`: For image processing and encoding.  
- `base64`, `io`, `os`, `subprocess`: Standard libraries for utility functions.

5. **Usage Example**  
Deploy the app with:  
```bash
modal deploy modal/ocr_gradio.py
```  
Access via the generated Modal public URL (e.g., `https://<app>.modal.run`).
```
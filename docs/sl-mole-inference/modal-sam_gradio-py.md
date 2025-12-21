```markdown
1. **Purpose**  
This file deploys a Gradio-based SAM3 segmentation web app using Modal, allowing users to upload images and prompts to detect and annotate objects via an external SAM inference endpoint. The app returns annotated images, JSON grounding results, and inference timing info.

2. **Key Components**  
- `run_sam`: Processes image and prompt, sends request to SAM endpoint, draws bounding boxes, and returns results.  
- `draw_boxes`: Annotates image with colored boxes and confidence scores.  
- `web`: Modal function launching the Gradio interface on port 7860.  
- `SAM_ENDPOINT`: Environment variable for inference backend URL (defaults to Modal-hosted endpoint).  
- `demo`: Gradio interface with image input, text prompt, and three output components.

3. **Algorithm Notes**  
- Uses `requests` to call a remote SAM inference service.  
- Annotates results with color-coded bounding boxes and scores.  
- Handles errors, timeouts, and empty responses gracefully.  
- Leverages Modal’s concurrency and auto-scaling for web server deployment.

4. **Dependencies**  
- `modal` (for deployment)  
- `gradio>=4.0.0` (UI framework)  
- `requests>=2.31.0` (HTTP client)  
- `Pillow>=10.0.0` (image processing)  
- `base64`, `io`, `os`, `json`, `typing` (built-ins)

5. **Usage Example**  
Deploy with:  
```bash
modal deploy modal/sam_gradio.py
```  
Access via the generated Modal web URL (e.g., `https://<app>.modal.run`). Upload an image and enter a prompt (e.g., “Find the cat”) to receive annotated results.
```
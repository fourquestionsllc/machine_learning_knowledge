Fine-tuning a YOLO model with your own dataset and labels is a multi-step process. This process involves preparing your dataset, training the model, saving the model, and finally deploying the model using OpenCV and FastAPI for inference.

### High-Level Workflow:

1. **Prepare Your Custom Dataset**: Label your images and create annotation files.
2. **Train the YOLO Model**: Fine-tune a pre-trained YOLO model using your dataset.
3. **Save the Fine-Tuned Model**: Save the weights of the fine-tuned model.
4. **Use OpenCV to Load and Run the Model**: Inference using OpenCV.
5. **Create a FastAPI Service**: Serve the model via FastAPI.

---

### Step 1: Prepare Your Custom Dataset

The custom dataset should include:

- **Images**: Your object detection images (e.g., `.jpg`, `.png`).
- **Labels**: YOLO format text files (`.txt`) for each image, with each line containing:
  - Class index
  - Bounding box (center_x, center_y, width, height) normalized by image width and height

For example:
```
0 0.5 0.5 0.3 0.4
1 0.7 0.6 0.2 0.3
```

Where `0` and `1` are the class indices (according to your custom classes list), and the rest are the normalized bounding box coordinates.

### Step 2: Fine-Tune YOLO

You can use the **Darknet framework** (official framework for YOLO) for training. YOLO models require specific configurations (like class count and bounding box layer adjustments) to train on your dataset.

1. **Download YOLOv4 and Prepare Dataset:**
   - Download the YOLOv4 weights, config files, and class file from the official YOLO repository: https://github.com/AlexeyAB/darknet.
   - Set up the dataset in the appropriate folder structure.

2. **Modify the Configuration File:**
   - Update the `.cfg` file (e.g., `yolov4.cfg`) to match the number of classes in your dataset by modifying:
     - The number of filters in the last convolutional layer: `filters=(classes + 5) * 3` (where `classes` is the number of your custom classes).
     - Update the `classes` parameter in the `[yolo]` layer to match your number of classes.

3. **Train the Model Using Darknet:**
   - Command to train:
   ```bash
   ./darknet detector train data/obj.data yolov4.cfg yolov4.weights
   ```
   - During training, you'll monitor the loss function and eventually save the model after several epochs.

Once training is complete, save the weights (e.g., `yolov4_final.weights`).

---

### Step 3: Use OpenCV for Inference

1. **Install OpenCV**:
   If you haven't installed OpenCV yet, you can install it via `pip`:
   ```bash
   pip install opencv-python opencv-python-headless
   ```

2. **Code to Load the Trained YOLO Model and Make Predictions**:

Here’s how to load the fine-tuned YOLO model and perform object detection using OpenCV.

```python
import cv2
import numpy as np

# Load YOLO model (fine-tuned)
net = cv2.dnn.readNet("yolov4_final.weights", "yolov4.cfg")

# Load class labels
with open("coco.names", "r") as f:
    classes = [line.strip() for line in f.readlines()]

# Get YOLO layer names
layer_names = net.getLayerNames()
output_layers = [layer_names[i - 1] for i in net.getUnconnectedOutLayers()]

def detect_objects(image):
    # Prepare image for YOLO
    height, width, channels = image.shape
    blob = cv2.dnn.blobFromImage(image, 0.00392, (416, 416), (0, 0, 0), True, crop=False)
    net.setInput(blob)
    outs = net.forward(output_layers)

    # Process the outputs
    class_ids = []
    confidences = []
    boxes = []
    for out in outs:
        for detection in out:
            scores = detection[5:]
            class_id = np.argmax(scores)
            confidence = scores[class_id]
            
            if confidence > 0.5:  # Confidence threshold
                center_x = int(detection[0] * width)
                center_y = int(detection[1] * height)
                w = int(detection[2] * width)
                h = int(detection[3] * height)
                
                x = int(center_x - w / 2)
                y = int(center_y - h / 2)

                boxes.append([x, y, w, h])
                confidences.append(float(confidence))
                class_ids.append(class_id)

    # Apply non-maxima suppression
    indices = cv2.dnn.NMSBoxes(boxes, confidences, 0.5, 0.4)
    return indices, boxes, class_ids, confidences

# Load an image to test
image = cv2.imread('test_image.jpg')
indices, boxes, class_ids, confidences = detect_objects(image)

# Display results
if len(indices) > 0:
    for i in indices.flatten():
        x, y, w, h = boxes[i]
        label = str(classes[class_ids[i]])
        confidence = str(round(confidences[i], 2))
        
        # Draw bounding boxes and labels
        cv2.rectangle(image, (x, y), (x + w, y + h), (0, 255, 0), 2)
        cv2.putText(image, label + " " + confidence, (x, y - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 2)

# Show the output
cv2.imshow("Detection", image)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

### Step 4: Serve the Model Using FastAPI

Next, let's create a FastAPI application to serve the YOLO model and accept HTTP requests for object detection.

1. **Install FastAPI and Uvicorn**:
   ```bash
   pip install fastapi uvicorn
   ```

2. **FastAPI Code for Serving YOLO Model**:

Create a file, e.g., `main.py` to serve the model using FastAPI.

```python
from fastapi import FastAPI, File, UploadFile
from fastapi.responses import JSONResponse
import cv2
import numpy as np
from io import BytesIO

app = FastAPI()

# Load YOLO model (fine-tuned)
net = cv2.dnn.readNet("yolov4_final.weights", "yolov4.cfg")
with open("coco.names", "r") as f:
    classes = [line.strip() for line in f.readlines()]
layer_names = net.getLayerNames()
output_layers = [layer_names[i - 1] for i in net.getUnconnectedOutLayers()]

def detect_objects(image):
    height, width, channels = image.shape
    blob = cv2.dnn.blobFromImage(image, 0.00392, (416, 416), (0, 0, 0), True, crop=False)
    net.setInput(blob)
    outs = net.forward(output_layers)

    class_ids = []
    confidences = []
    boxes = []
    for out in outs:
        for detection in out:
            scores = detection[5:]
            class_id = np.argmax(scores)
            confidence = scores[class_id]
            
            if confidence > 0.5:
                center_x = int(detection[0] * width)
                center_y = int(detection[1] * height)
                w = int(detection[2] * width)
                h = int(detection[3] * height)
                
                x = int(center_x - w / 2)
                y = int(center_y - h / 2)

                boxes.append([x, y, w, h])
                confidences.append(float(confidence))
                class_ids.append(class_id)

    indices = cv2.dnn.NMSBoxes(boxes, confidences, 0.5, 0.4)
    return indices, boxes, class_ids, confidences

@app.post("/predict/")
async def predict(file: UploadFile = File(...)):
    # Read the uploaded image
    img_bytes = await file.read()
    img_array = np.frombuffer(img_bytes, np.uint8)
    image = cv2.imdecode(img_array, cv2.IMREAD_COLOR)
    
    indices, boxes, class_ids, confidences = detect_objects(image)
    
    results = []
    if len(indices) > 0:
        for i in indices.flatten():
            x, y, w, h = boxes[i]
            label = str(classes[class_ids[i]])
            confidence = str(round(confidences[i], 2))
            results.append({
                "label": label,
                "confidence": confidence,
                "box": [x, y, w, h]
            })

    return JSONResponse(content={"detections": results})

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### Step 5: Running the FastAPI Server

To run the FastAPI server:

```bash
uvicorn main:app --reload
```

This will start a FastAPI server on `http://localhost:8000`. You can now send POST requests to the `/predict/` endpoint to upload images and get object detection results.

### Example Request Using `curl`:

```bash
curl -X 'POST' \
  'http://127.0.0.1:8000/predict/' \
  -H 'accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'file=@test_image.jpg'
```

### Conclusion:
This code provides an end-to-end solution for fine-tuning a YOLO model on your own dataset, performing inference using OpenCV, and serving the model with FastAPI.

Here's a complete example of object detection using OpenCV. We will use a **Haar Cascade Classifier** to detect faces in an image. This is one of the simplest and most commonly used methods in OpenCV for object detection. OpenCV comes with pre-trained classifiers for detecting objects like faces, eyes, etc.

### Steps to Set Up the Object Detection Example:
1. **Install OpenCV** (if not installed already):
   ```bash
   pip install opencv-python
   pip install opencv-python-headless
   ```

2. **Download Haar Cascade Classifier for Face Detection**:
   You can download the pre-trained Haar Cascade classifier for face detection from OpenCV's GitHub or use the one already available in OpenCV. We'll use the default one:
   - `haarcascade_frontalface_default.xml`

3. **Write the Code**:
   This code will read an image, apply face detection, and then display the results (highlighting detected faces with rectangles).

### Code for Object Detection (Face Detection) in OpenCV

```python
import cv2

# 1. Load the pre-trained Haar Cascade face detector model
# You can download the xml file from OpenCV's GitHub or use the one bundled with OpenCV
face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')

# 2. Read the image from file
image = cv2.imread('test_image.jpg')  # Replace with your image file path

# 3. Convert the image to grayscale
gray_image = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

# 4. Perform face detection
# The scaleFactor compensates for any size variations in the image. minNeighbors is the sensitivity level.
faces = face_cascade.detectMultiScale(gray_image, scaleFactor=1.1, minNeighbors=5, minSize=(30, 30))

# 5. Draw rectangles around the detected faces
for (x, y, w, h) in faces:
    # Drawing a rectangle around each face
    cv2.rectangle(image, (x, y), (x + w, y + h), (0, 255, 0), 2)

# 6. Display the output image
cv2.imshow('Face Detection', image)

# 7. Wait for any key press to close the image window
cv2.waitKey(0)
cv2.destroyAllWindows()

# 8. Optionally, save the output image to a file
cv2.imwrite('output_image.jpg', image)
```

### Explanation of the Code:

1. **Loading the Haar Cascade Classifier**:
   - We use OpenCV's built-in Haar cascade for detecting faces: `cv2.CascadeClassifier()`.
   - This is a machine learning-based classifier used for object detection.

2. **Reading and Preprocessing the Image**:
   - `cv2.imread()` loads the image.
   - `cv2.cvtColor()` converts the image to grayscale. Face detection generally works better in grayscale, as it reduces the complexity.

3. **Face Detection**:
   - `detectMultiScale()` detects objects (in this case, faces). It works by scanning the image at multiple scales.
     - `scaleFactor`: compensates for faces appearing smaller or larger due to distance or resolution.
     - `minNeighbors`: specifies how many neighbors each candidate rectangle should have to be considered a face.
     - `minSize`: defines the minimum size of objects to detect.

4. **Drawing Rectangles on Detected Faces**:
   - We loop through the list of detected faces and draw a rectangle around each face using `cv2.rectangle()`.

5. **Displaying and Saving the Image**:
   - `cv2.imshow()` shows the image with detected faces in a window.
   - `cv2.imwrite()` saves the image with rectangles around the faces to a file.

### Running the Code:

1. Place an image file (like `test_image.jpg`) in the same directory as the script, or replace the path with your image's path.
2. Run the script. It will display the image with faces detected (highlighted with rectangles).
3. The output image will also be saved as `output_image.jpg` in the same directory.

### Example of Expected Output:
The program will output an image with green rectangles drawn around detected faces. If no faces are detected, the image will be shown without any rectangles.

### Notes:
- **Haar Cascades**: This is a classical method for object detection. While fast, it may not be as accurate as modern methods (like using deep learning models). However, it’s a great starting point for real-time applications.
- **Accuracy**: The performance and accuracy depend on the quality of the image and the size of the objects being detected.

### Using a Video Stream:
You can modify the code to work with a webcam in real-time by replacing the image loading part with video capture:

```python
# For real-time face detection from webcam
cap = cv2.VideoCapture(0)  # Open webcam (0 is default camera)

while True:
    ret, frame = cap.read()
    if not ret:
        break
    
    gray_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    faces = face_cascade.detectMultiScale(gray_frame, scaleFactor=1.1, minNeighbors=5, minSize=(30, 30))
    
    for (x, y, w, h) in faces:
        cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 2)
    
    cv2.imshow('Face Detection (Webcam)', frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```

- This code will open the webcam, continuously detect faces in the video stream, and display the output in real-time.
- Press 'q' to quit the video stream. 

This demonstrates a simple object detection example with OpenCV using Haar cascades, suitable for both image and video analysis.

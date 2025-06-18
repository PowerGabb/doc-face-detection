# Face Emotion Detection Backend

Backend API Python untuk deteksi wajah, emosi, umur, dan gender menggunakan Flask dan DeepFace.

## Fitur

- ✅ Deteksi wajah dalam gambar
- ✅ Analisis emosi (7 emosi: angry, disgust, fear, happy, neutral, sad, surprise)
- ✅ Prediksi umur
- ✅ Prediksi gender dengan confidence score
- ✅ Support upload file gambar atau base64
- ✅ CORS enabled untuk frontend integration
- ✅ Logging untuk debugging

## Teknologi

- **Python 3.8+** - Programming language
- **Flask** - Web framework
- **DeepFace** - Face analysis library
- **OpenCV** - Computer vision library
- **TensorFlow** - Machine learning framework
- **Pillow** - Image processing
- **Flask-CORS** - Cross-origin resource sharing

## Instalasi

### 1. Buat Virtual Environment
```bash
python -m venv venv

# Windows
venv\Scripts\activate

# Linux/Mac
source venv/bin/activate
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

### 3. Jalankan Server
```bash
# Development mode
python app.py

# Production mode dengan Gunicorn
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

Server akan berjalan di `http://localhost:5000`

## API Endpoints

### 1. Health Check
```
GET /health
```
Response:
```json
{
  "status": "OK",
  "message": "Face detection server is running",
  "models_loaded": true
}
```

### 2. Model Status
```
GET /api/face/status
```
Response:
```json
{
  "success": true,
  "models_loaded": true,
  "available_models": {
    "emotion": true,
    "age": true,
    "gender": true,
    "face_detection": true
  }
}
```

### 3. Initialize Models
```
POST /api/face/initialize
```
Response:
```json
{
  "success": true,
  "message": "Models initialized successfully"
}
```

### 4. Detect Faces (Base64)
```
POST /api/face/detect
Content-Type: application/json

{
  "image": "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQ..."
}
```

### 5. Detect Faces (File Upload)
```
POST /api/face/detect-upload
Content-Type: multipart/form-data

Form data:
- image: [file]
```

### Response Format (Detect Faces)
```json
{
  "success": true,
  "message": "Detected 1 face(s)",
  "faces": [
    {
      "id": 1,
      "boundingBox": {
        "x": 100,
        "y": 50,
        "width": 150,
        "height": 180
      },
      "age": 25,
      "gender": "woman",
      "genderConfidence": 95.5,
      "emotions": {
        "angry": 2.1,
        "disgust": 0.5,
        "fear": 1.2,
        "happy": 85.3,
        "neutral": 8.9,
        "sad": 1.5,
        "surprise": 0.5
      },
      "dominantEmotion": "happy",
      "emotionConfidence": 85.3
    }
  ],
  "imageInfo": {
    "width": 640,
    "height": 480
  }
}
```

## Struktur Project

```
face-backend/
├── app.py                   # Main Flask application
├── requirements.txt         # Python dependencies
├── README.md               # Documentation
├── .env.example            # Environment variables template
├── .gitignore              # Git ignore file
└── models/                 # DeepFace models (auto-downloaded)
```

## Penggunaan

### 1. Deteksi dengan Base64
```python
import requests
import base64

# Read and encode image
with open('image.jpg', 'rb') as f:
    image_data = base64.b64encode(f.read()).decode('utf-8')

response = requests.post('http://localhost:5000/api/face/detect', 
    json={'image': f'data:image/jpeg;base64,{image_data}'})

result = response.json()
print(result['faces'])
```

### 2. Deteksi dengan File Upload
```python
import requests

with open('image.jpg', 'rb') as f:
    files = {'image': f}
    response = requests.post('http://localhost:5000/api/face/detect-upload', files=files)

result = response.json()
print(result['faces'])
```

### 3. JavaScript/Frontend Integration
```javascript
// Base64 detection
const response = await fetch('http://localhost:5000/api/face/detect', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    image: 'data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQ...'
  })
});

const result = await response.json();
console.log(result.faces);

// File upload detection
const formData = new FormData();
formData.append('image', fileInput.files[0]);

const response = await fetch('http://localhost:5000/api/face/detect-upload', {
  method: 'POST',
  body: formData
});

const result = await response.json();
console.log(result.faces);
```

## Error Handling

Semua endpoint mengembalikan error dalam format:
```json
{
  "success": false,
  "error": "Error type",
  "message": "Detailed error message"
}
```

Common errors:
- `400` - Bad request (no image provided, invalid image)
- `500` - Server error (model loading failed, processing error)

## Konfigurasi

### Environment Variables
```bash
PORT=5000                    # Server port (default: 5000)
FLASK_ENV=development        # Flask environment
FLASK_DEBUG=True            # Debug mode
```

### Model Configuration
DeepFace akan otomatis mendownload model yang diperlukan pada first run:
- Face detection: OpenCV Haar Cascade
- Emotion recognition: FER model
- Age prediction: VGG-Face
- Gender prediction: VGG-Face

## Development

### Start Development Server
```bash
python app.py
```

### Production Deployment
```bash
# Install production dependencies
pip install gunicorn

# Run with Gunicorn
gunicorn -w 4 -b 0.0.0.0:5000 app:app

# Or with specific configuration
gunicorn -c gunicorn.conf.py app:app
```

### Docker Deployment
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libglib2.0-0 \
    libsm6 \
    libxext6 \
    libxrender-dev \
    libgomp1 \
    libglib2.0-0 \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:5000", "app:app"]
```

## Troubleshooting

### 1. Model Loading Issues
- Pastikan koneksi internet untuk download model pertama kali
- Check logs untuk error detail
- Restart server jika model loading gagal

### 2. OpenCV Issues
- Windows: Install Visual C++ Redistributable
- Linux: `sudo apt-get install python3-opencv`
- macOS: `brew install opencv`

### 3. Memory Issues
- Reduce image size sebelum processing
- Increase server memory allocation
- Use model caching untuk multiple requests

### 4. Performance Optimization
- Preload models pada startup
- Use GPU acceleration jika tersedia
- Implement image resizing untuk input besar
- Add request queuing untuk high load

## Model Information

### DeepFace Models
- **Emotion**: FER-2013 trained model
- **Age**: VGG-Face based regression
- **Gender**: VGG-Face based classification
- **Face Detection**: OpenCV Haar Cascade

### Supported Image Formats
- JPEG/JPG
- PNG
- BMP
- TIFF
- WebP

## License

MIT License

## Contributing

1. Fork the repository
2. Create feature branch
3. Commit changes
4. Push to branch
5. Create Pull Request

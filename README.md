# 📸 Camera to OneDrive – HTML App + Power Automate Flow

This project captures photos directly from a device camera using HTML/JS and automatically uploads them to **OneDrive** using a **Power Automate Flow (HTTP Trigger)**.

---

## 🚀 Features

* Capture up to **10 photos** using device camera
* Delete preview images before upload
* Send all images to OneDrive via Power Automate
* Unique file naming using **userId + timestamp + random number**
* Clean UI, responsive layout

---

## 📂 Project Structure

```
project/
│
├── index.html        # Main Web App (Camera + Submit)
├── flow.json         # Power Automate Flow Schema
└── README.md         # Documentation
```

---

# 1. HTML Code (index.html)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Camera App to OneDrive</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh; padding: 20px;
        }
        .container { max-width: 400px; margin: 0 auto; background: white; border-radius: 20px; padding: 30px; box-shadow: 0 20px 40px rgba(0,0,0,0.1); }
        h1 { text-align: center; color: #333; margin-bottom: 30px; }
        #cameraSection { text-align: center; }
        button {
            background: #0078d4; color: white; border: none; padding: 15px 30px;
            border-radius: 25px; font-size: 16px; cursor: pointer; margin: 10px;
            transition: all 0.3s; width: 100%;
        }
        button:hover { background: #106ebe; transform: translateY(-2px); }
        button:disabled { background: #ccc; cursor: not-allowed; transform: none; }
        #video { width: 100%; max-width: 350px; height: 450px; border-radius: 15px; background: #000; }
        #preview {
            display: grid; grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
            gap: 10px; margin: 20px 0;
        }
        .preview-img {
            width: 80px; height: 80px; border-radius: 10px; object-fit: cover;
            position: relative; border: 3px solid #0078d4;
        }
        .delete-btn {
            position: absolute; top: -6px; right: -6px; background: red; color: white;
            border: none; border-radius: 8px; width: 32px; height: 32px; cursor: pointer;
            font-size: 18px; line-height: 32px; text-align: center; padding: 0;
        }
        #userIdInput { width: 100%; padding: 15px; border: 2px solid #ddd; border-radius: 10px; font-size: 16px; margin: 10px 0; }
        .status { padding: 10px; border-radius: 10px; margin: 10px 0; text-align: center; }
        .success { background: #d4edda; color: #155724; }
        .error { background: #f8d7da; color: #721c24; }
    </style>
</head>
<body>
    <div class="container">
        <h1>📸 Camera</h1>
        <div id="cameraSection">
            <video id="video" autoplay playsinline></video>
            <button onclick="capturePic()">📷 Capture Photo</button>
            <input type="text" id="userIdInput" placeholder="Your Name / Phone ID">
            <div id="preview"></div>
            <button id="submitBtn" onclick="submitPics()" disabled>Submit to OneDrive</button>
            <div id="status"></div>
        </div>
    </div>

    <script>
        let capturedPics = [];
        const FLOW_URL = "YOUR_FLOW_URL_HERE";

        async function startApp() {
            const stream = await navigator.mediaDevices.getUserMedia({ video: true });
            document.getElementById('video').srcObject = stream;
        }

        function capturePic() {
            const video = document.getElementById('video');
            const canvas = document.createElement('canvas');
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            canvas.getContext('2d').drawImage(video, 0, 0);

            const dataUrl = canvas.toDataURL('image/jpeg');
            capturedPics.push(dataUrl);
            updatePreview();
        }

        function updatePreview() {
            const preview = document.getElementById('preview');
            preview.innerHTML = '';

            capturedPics.forEach((pic, index) => {
                const div = document.createElement('div');
                div.className = 'preview-img';
                div.innerHTML = `
                    <img src="${pic}" style="width:100%;height:100%;border-radius:10px;">
                    <button class="delete-btn" onclick="deletePic(${index})">×</button>
                `;
                preview.appendChild(div);
            });

            document.getElementById('submitBtn').disabled = capturedPics.length === 0;
        }

        function deletePic(index) {
            capturedPics.splice(index, 1);
            updatePreview();
        }

        async function submitPics() {
            const userId = document.getElementById('userIdInput').value || "anonymous";

            const payload = {
                images: capturedPics,
                userId: userId,
                timestamp: new Date().toISOString()
            };

            const res = await fetch(FLOW_URL, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            if (res.ok) {
                document.getElementById('status').innerText = "Uploaded Successfully!";
                capturedPics = [];
                updatePreview();
            }
        }

        window.onload = startApp;
    </script>
</body>
</html>
```

---

# 2. Power Automate Flow JSON (flow.json)

```json
{
    "type": "object",
    "properties": {
        "images": {
            "type": "array",
            "items": {
                "type": "string"
            }
        },
        "userId": {
            "type": "string"
        },
        "timestamp": {
            "type": "string"
        }
    }
}
```

---

# 3. Power Automate — File Name Formula

```
concat(
    triggerBody()?['userId'], '_', string(utcNow()), '_', string(rand(1,100000)), '.jpg'
)
```

# 4. Base64 → Binary Conversion

```
base64ToBinary(
  replace(items('Apply_to_each'), 'data:image/jpeg;base64,', '')
)
```

---

# 📌 How to Use

### **1️⃣ Upload HTML file to GitHub Pages**

* Go to your repo → `index.html` upload
* Settings → Pages → Deploy from root

### **2️⃣ Create Power Automate Flow**

* Trigger → **When an HTTP request is received**
* Paste JSON schema
* Apply to each → Create File

### **3️⃣ Paste your Flow URL inside HTML**

Replace:

```
const FLOW_URL = "YOUR_FLOW_URL_HERE";
```

---

# 🎉 Done!

You now have a complete **GitHub-ready README.md** with everything included.

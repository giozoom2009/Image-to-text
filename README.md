<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Image to Text Extractor</title>
    <script src='https://cdnjs.cloudflare.com/ajax/libs/tesseract.js/5.1.0/tesseract.min.js'></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .container {
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            padding: 40px;
            max-width: 800px;
            width: 100%;
        }

        h1 {
            color: #333;
            text-align: center;
            margin-bottom: 30px;
            font-size: 2em;
        }

        .upload-area {
            border: 3px dashed #667eea;
            border-radius: 15px;
            padding: 40px;
            text-align: center;
            background: #f8f9ff;
            cursor: pointer;
            transition: all 0.3s;
            margin-bottom: 20px;
        }

        .upload-area:hover {
            background: #eef0ff;
            border-color: #764ba2;
        }

        .upload-area.dragover {
            background: #e0e7ff;
            border-color: #764ba2;
        }

        #fileInput {
            display: none;
        }

        .upload-icon {
            font-size: 3em;
            margin-bottom: 10px;
        }

        .upload-text {
            color: #666;
            font-size: 1.1em;
        }

        .preview-container {
            margin: 20px 0;
            text-align: center;
        }

        #imagePreview {
            max-width: 100%;
            max-height: 400px;
            border-radius: 10px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
        }

        .hidden {
            display: none;
        }

        .progress-container {
            margin: 20px 0;
        }

        .progress-bar {
            width: 100%;
            height: 30px;
            background: #e0e0e0;
            border-radius: 15px;
            overflow: hidden;
        }

        .progress-fill {
            height: 100%;
            background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
            width: 0%;
            transition: width 0.3s;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-weight: bold;
        }

        .result-container {
            margin-top: 30px;
        }

        .result-label {
            font-weight: bold;
            color: #333;
            margin-bottom: 10px;
            font-size: 1.1em;
        }

        #extractedText {
            width: 100%;
            min-height: 200px;
            padding: 15px;
            border: 2px solid #e0e0e0;
            border-radius: 10px;
            font-size: 1em;
            font-family: monospace;
            resize: vertical;
        }

        .button-group {
            margin-top: 20px;
            display: flex;
            gap: 10px;
            justify-content: center;
        }

        button {
            padding: 12px 30px;
            border: none;
            border-radius: 8px;
            font-size: 1em;
            cursor: pointer;
            transition: all 0.3s;
            font-weight: bold;
        }

        .btn-primary {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }

        .btn-primary:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
        }

        .btn-secondary {
            background: #f0f0f0;
            color: #333;
        }

        .btn-secondary:hover {
            background: #e0e0e0;
        }

        .status-message {
            text-align: center;
            color: #667eea;
            font-weight: bold;
            margin: 15px 0;
        }

        .language-selector {
            margin-bottom: 20px;
            text-align: center;
        }

        .language-selector label {
            font-weight: bold;
            color: #333;
            margin-right: 10px;
            font-size: 1.1em;
        }

        .language-selector select {
            padding: 10px 20px;
            border: 2px solid #667eea;
            border-radius: 8px;
            font-size: 1em;
            cursor: pointer;
            background: white;
            color: #333;
        }

        .language-selector select:focus {
            outline: none;
            border-color: #764ba2;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📸 Image to Text Extractor</h1>
        
        <div class="language-selector">
            <label for="languageSelect">Select Language:</label>
            <select id="languageSelect">
                <option value="eng">English</option>
                <option value="spa">Spanish</option>
                <option value="fra">French</option>
                <option value="deu">German</option>
                <option value="ita">Italian</option>
                <option value="por">Portuguese</option>
                <option value="rus">Russian</option>
                <option value="ara">Arabic</option>
                <option value="chi_sim">Chinese (Simplified)</option>
                <option value="chi_tra">Chinese (Traditional)</option>
                <option value="jpn">Japanese</option>
                <option value="kor">Korean</option>
                <option value="hin">Hindi</option>
                <option value="tur">Turkish</option>
                <option value="pol">Polish</option>
                <option value="nld">Dutch</option>
                <option value="swe">Swedish</option>
                <option value="vie">Vietnamese</option>
            </select>
        </div>
        
        <div class="upload-area" id="uploadArea">
            <div class="upload-icon">📁</div>
            <div class="upload-text">Click to upload or drag & drop an image here</div>
            <input type="file" id="fileInput" accept="image/*">
        </div>

        <div class="preview-container hidden" id="previewContainer">
            <img id="imagePreview" alt="Preview">
        </div>

        <div class="progress-container hidden" id="progressContainer">
            <div class="progress-bar">
                <div class="progress-fill" id="progressFill">0%</div>
            </div>
            <div class="status-message" id="statusMessage"></div>
        </div>

        <div class="result-container hidden" id="resultContainer">
            <div class="result-label">Extracted Text:</div>
            <textarea id="extractedText" readonly></textarea>
            <div class="button-group">
                <button class="btn-primary" onclick="copyText()">📋 Copy Text</button>
                <button class="btn-primary" onclick="downloadText()">💾 Download as TXT</button>
                <button class="btn-secondary" onclick="reset()">🔄 Upload New Image</button>
            </div>
        </div>
    </div>

    <script>
        const uploadArea = document.getElementById('uploadArea');
        const fileInput = document.getElementById('fileInput');
        const imagePreview = document.getElementById('imagePreview');
        const previewContainer = document.getElementById('previewContainer');
        const progressContainer = document.getElementById('progressContainer');
        const progressFill = document.getElementById('progressFill');
        const statusMessage = document.getElementById('statusMessage');
        const resultContainer = document.getElementById('resultContainer');
        const extractedText = document.getElementById('extractedText');
        const languageSelect = document.getElementById('languageSelect');

        // Click to upload
        uploadArea.addEventListener('click', () => fileInput.click());

        // File input change
        fileInput.addEventListener('change', (e) => {
            handleFile(e.target.files[0]);
        });

        // Drag and drop
        uploadArea.addEventListener('dragover', (e) => {
            e.preventDefault();
            uploadArea.classList.add('dragover');
        });

        uploadArea.addEventListener('dragleave', () => {
            uploadArea.classList.remove('dragover');
        });

        uploadArea.addEventListener('drop', (e) => {
            e.preventDefault();
            uploadArea.classList.remove('dragover');
            handleFile(e.dataTransfer.files[0]);
        });

        function handleFile(file) {
            if (!file || !file.type.startsWith('image/')) {
                alert('Please upload a valid image file');
                return;
            }

            const reader = new FileReader();
            reader.onload = (e) => {
                imagePreview.src = e.target.result;
                previewContainer.classList.remove('hidden');
                processImage(e.target.result);
            };
            reader.readAsDataURL(file);
        }

        function processImage(imageSrc) {
            progressContainer.classList.remove('hidden');
            resultContainer.classList.add('hidden');
            const selectedLanguage = languageSelect.value;
            
            Tesseract.recognize(
                imageSrc,
                selectedLanguage,
                {
                    logger: (m) => {
                        if (m.status === 'recognizing text') {
                            const progress = Math.round(m.progress * 100);
                            progressFill.style.width = progress + '%';
                            progressFill.textContent = progress + '%';
                            statusMessage.textContent = 'Extracting text...';
                        }
                    }
                }
            ).then(({ data: { text } }) => {
                extractedText.value = text;
                progressContainer.classList.add('hidden');
                resultContainer.classList.remove('hidden');
            }).catch(err => {
                console.error(err);
                statusMessage.textContent = 'Error processing image';
                statusMessage.style.color = 'red';
            });
        }

        function copyText() {
            extractedText.select();
            document.execCommand('copy');
            
            const btn = event.target;
            const originalText = btn.textContent;
            btn.textContent = '✓ Copied!';
            setTimeout(() => {
                btn.textContent = originalText;
            }, 2000);
        }

        function downloadText() {
            const text = extractedText.value;
            const blob = new Blob([text], { type: 'text/plain' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'extracted-text.txt';
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
            
            const btn = event.target;
            const originalText = btn.textContent;
            btn.textContent = '✓ Downloaded!';
            setTimeout(() => {
                btn.textContent = originalText;
            }, 2000);
        }

        function reset() {
            fileInput.value = '';
            previewContainer.classList.add('hidden');
            progressContainer.classList.add('hidden');
            resultContainer.classList.add('hidden');
            progressFill.style.width = '0%';
        }
    </script>
</body>
</html>

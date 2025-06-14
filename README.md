<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>8x8 Grid Overlay Tool</title>
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
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            padding: 30px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
        }

        h1 {
            text-align: center;
            color: #333;
            margin-bottom: 30px;
            font-size: 2.5em;
            font-weight: 300;
        }

        .upload-area {
            border: 3px dashed #667eea;
            border-radius: 15px;
            padding: 40px;
            text-align: center;
            margin-bottom: 30px;
            transition: all 0.3s ease;
            cursor: pointer;
            background: linear-gradient(45deg, rgba(102, 126, 234, 0.05), rgba(118, 75, 162, 0.05));
        }

        .upload-area:hover {
            border-color: #764ba2;
            background: linear-gradient(45deg, rgba(102, 126, 234, 0.1), rgba(118, 75, 162, 0.1));
            transform: translateY(-2px);
        }

        .upload-area.dragover {
            border-color: #764ba2;
            background: linear-gradient(45deg, rgba(102, 126, 234, 0.15), rgba(118, 75, 162, 0.15));
        }

        #fileInput {
            display: none;
        }

        .upload-text {
            font-size: 1.2em;
            color: #666;
            margin-bottom: 10px;
        }

        .upload-hint {
            color: #999;
            font-size: 0.9em;
        }

        .controls {
            display: flex;
            gap: 15px;
            margin-bottom: 30px;
            flex-wrap: wrap;
            align-items: center;
        }

        .control-group {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        label {
            font-weight: 500;
            color: #333;
        }

        input[type="range"], input[type="color"], input[type="number"] {
            padding: 5px;
            border: 2px solid #ddd;
            border-radius: 8px;
            outline: none;
            transition: border-color 0.3s ease;
        }

        input[type="range"]:focus, input[type="color"]:focus, input[type="number"]:focus {
            border-color: #667eea;
        }

        button {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 25px;
            cursor: pointer;
            font-size: 1em;
            transition: all 0.3s ease;
            font-weight: 500;
        }

        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 20px rgba(102, 126, 234, 0.3);
        }

        button:disabled {
            opacity: 0.5;
            cursor: not-allowed;
            transform: none;
            box-shadow: none;
        }

        .images-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }

        .image-container {
            background: white;
            border-radius: 15px;
            padding: 20px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
            transition: transform 0.3s ease;
        }

        .image-container:hover {
            transform: translateY(-5px);
        }

        .image-name {
            font-size: 0.9em;
            color: #666;
            margin-bottom: 10px;
            font-weight: 500;
        }

        canvas {
            max-width: 100%;
            height: auto;
            border-radius: 10px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
        }

        .download-btn {
            margin-top: 10px;
            width: 100%;
            background: linear-gradient(135deg, #43a047 0%, #66bb6a 100%);
        }

        .download-btn:hover {
            box-shadow: 0 8px 20px rgba(67, 160, 71, 0.3);
        }

        .stats {
            text-align: center;
            margin: 20px 0;
            padding: 15px;
            background: rgba(102, 126, 234, 0.1);
            border-radius: 10px;
            color: #333;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>8×8 Grid Overlay Tool</h1>
        
        <div class="upload-area" onclick="document.getElementById('fileInput').click()">
            <div class="upload-text">📁 Click to select images or drag and drop</div>
            <div class="upload-hint">Supports JPG, PNG, GIF • Multiple files allowed</div>
            <input type="file" id="fileInput" accept="image/*" multiple>
        </div>

        <div class="controls">
            <div class="control-group">
                <label for="gridSize">Grid Size:</label>
                <input type="number" id="gridSize" value="8" min="2" max="20">
                <span>×</span>
                <input type="number" id="gridSizeY" value="8" min="2" max="20">
            </div>
            
            <div class="control-group">
                <label for="lineWidth">Line Width:</label>
                <input type="range" id="lineWidth" min="1" max="10" value="2">
                <span id="lineWidthValue">2px</span>
            </div>
            
            <div class="control-group">
                <label for="gridColor">Grid Color:</label>
                <input type="color" id="gridColor" value="#ff0000">
            </div>
            
            <div class="control-group">
                <label for="opacity">Opacity:</label>
                <input type="range" id="opacity" min="0.1" max="1" step="0.1" value="0.8">
                <span id="opacityValue">80%</span>
            </div>
            
            <button onclick="downloadAll()" id="downloadAllBtn" disabled>Download All</button>
            <button onclick="clearAll()">Clear All</button>
        </div>

        <div class="stats" id="stats" style="display: none;">
            <strong>Processed: <span id="imageCount">0</span> images</strong>
        </div>

        <div class="images-grid" id="imagesGrid"></div>
    </div>

    <script>
        let imageData = []; // Store original image data

        // File input handling
        document.getElementById('fileInput').addEventListener('change', handleFiles);
        
        // Drag and drop handling
        const uploadArea = document.querySelector('.upload-area');
        
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
            const files = Array.from(e.dataTransfer.files).filter(file => file.type.startsWith('image/'));
            if (files.length > 0) {
                processFiles(files);
            }
        });

        // Control updates
        document.getElementById('lineWidth').addEventListener('input', function() {
            updateLineWidthDisplay();
            updateAllGrids();
        });
        
        document.getElementById('opacity').addEventListener('input', function() {
            updateOpacityDisplay();
            updateAllGrids();
        });
        
        document.getElementById('gridSize').addEventListener('input', updateAllGrids);
        document.getElementById('gridSizeY').addEventListener('input', updateAllGrids);
        document.getElementById('gridColor').addEventListener('input', updateAllGrids);

        function updateLineWidthDisplay() {
            document.getElementById('lineWidthValue').textContent = document.getElementById('lineWidth').value + 'px';
        }

        function updateOpacityDisplay() {
            const opacity = document.getElementById('opacity').value;
            document.getElementById('opacityValue').textContent = Math.round(opacity * 100) + '%';
        }

        function handleFiles(event) {
            const files = Array.from(event.target.files);
            processFiles(files);
        }

        function processFiles(files) {
            const imageFiles = files.filter(file => file.type.startsWith('image/'));
            
            if (imageFiles.length === 0) {
                alert('Please select valid image files.');
                return;
            }

            imageFiles.forEach(file => {
                const reader = new FileReader();
                reader.onload = function(e) {
                    createImageWithGrid(e.target.result, file.name);
                };
                reader.readAsDataURL(file);
            });
        }

        function createImageWithGrid(imageSrc, fileName) {
            const img = new Image();
            img.onload = function() {
                const canvas = document.createElement('canvas');
                const ctx = canvas.getContext('2d');
                
                // Set canvas size to image size
                canvas.width = img.width;
                canvas.height = img.height;
                
                // Store original image data
                const imageInfo = {
                    img: img,
                    fileName: fileName,
                    canvas: canvas,
                    ctx: ctx
                };
                
                imageData.push(imageInfo);
                
                // Draw image and grid
                redrawImageWithGrid(imageInfo);
                
                // Create container
                const container = document.createElement('div');
                container.className = 'image-container';
                container.dataset.imageIndex = imageData.length - 1;
                
                const nameDiv = document.createElement('div');
                nameDiv.className = 'image-name';
                nameDiv.textContent = fileName;
                
                const downloadBtn = document.createElement('button');
                downloadBtn.className = 'download-btn';
                downloadBtn.textContent = '⬇ Download';
                downloadBtn.onclick = () => downloadImage(canvas, fileName);
                
                container.appendChild(nameDiv);
                container.appendChild(canvas);
                container.appendChild(downloadBtn);
                
                document.getElementById('imagesGrid').appendChild(container);
                
                updateStats();
            };
            img.src = imageSrc;
        }

        function redrawImageWithGrid(imageInfo) {
            const { img, canvas, ctx } = imageInfo;
            
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Draw original image
            ctx.drawImage(img, 0, 0);
            
            // Draw grid with uniform spacing
            drawUniformGrid(ctx, canvas.width, canvas.height);
        }

        function drawUniformGrid(ctx, width, height) {
            const gridSizeX = parseInt(document.getElementById('gridSize').value);
            const gridSizeY = parseInt(document.getElementById('gridSizeY').value);
            const lineWidth = parseInt(document.getElementById('lineWidth').value);
            const color = document.getElementById('gridColor').value;
            const opacity = parseFloat(document.getElementById('opacity').value);
            
            // Save current context state
            ctx.save();
            
            // Set grid properties
            ctx.strokeStyle = color;
            ctx.lineWidth = lineWidth;
            ctx.globalAlpha = opacity;
            ctx.lineCap = 'round';
            
            // Calculate uniform cell dimensions
            const cellWidth = width / gridSizeX;
            const cellHeight = height / gridSizeY;
            
            ctx.beginPath();
            
            // Draw vertical lines
            for (let i = 1; i < gridSizeX; i++) {
                const x = Math.round(i * cellWidth) + 0.5; // +0.5 for crisp lines
                ctx.moveTo(x, 0);
                ctx.lineTo(x, height);
            }
            
            // Draw horizontal lines
            for (let i = 1; i < gridSizeY; i++) {
                const y = Math.round(i * cellHeight) + 0.5; // +0.5 for crisp lines
                ctx.moveTo(0, y);
                ctx.lineTo(width, y);
            }
            
            ctx.stroke();
            
            // Restore context state
            ctx.restore();
        }

        function updateAllGrids() {
            imageData.forEach(imageInfo => {
                redrawImageWithGrid(imageInfo);
            });
        }

        function downloadImage(canvas, fileName) {
            try {
                const link = document.createElement('a');
                link.download = fileName.replace(/\.[^/.]+$/, '') + '_grid.png';
                link.href = canvas.toDataURL('image/png');
                link.click();
            } catch (error) {
                console.error('Download failed:', error);
                alert('Download failed. Please try again.');
            }
        }

        function downloadAll() {
            if (imageData.length === 0) {
                alert('No images to download.');
                return;
            }
            
            // Show download progress
            const downloadBtn = document.getElementById('downloadAllBtn');
            const originalText = downloadBtn.textContent;
            
            downloadBtn.disabled = true;
            downloadBtn.textContent = 'Preparing Downloads...';
            
            // Sequential download with user confirmation for each
            let currentIndex = 0;
            
            function downloadNext() {
                if (currentIndex >= imageData.length) {
                    downloadBtn.disabled = false;
                    downloadBtn.textContent = originalText;
                    alert('All downloads completed!');
                    return;
                }
                
                const imageInfo = imageData[currentIndex];
                downloadBtn.textContent = `Downloading ${currentIndex + 1}/${imageData.length}...`;
                
                try {
                    // Create download link
                    const link = document.createElement('a');
                    link.download = imageInfo.fileName.replace(/\.[^/.]+$/, '') + '_grid.png';
                    link.href = imageInfo.canvas.toDataURL('image/png');
                    
                    // Trigger download
                    document.body.appendChild(link);
                    link.click();
                    document.body.removeChild(link);
                    
                    currentIndex++;
                    
                    // Continue with next download after a short delay
                    setTimeout(downloadNext, 500);
                    
                } catch (error) {
                    console.error('Download failed for:', imageInfo.fileName, error);
                    if (confirm(`Download failed for ${imageInfo.fileName}. Continue with next image?`)) {
                        currentIndex++;
                        setTimeout(downloadNext, 100);
                    } else {
                        downloadBtn.disabled = false;
                        downloadBtn.textContent = originalText;
                    }
                }
            }
            
            // Start the download sequence
            downloadNext();
        }

        function clearAll() {
            document.getElementById('imagesGrid').innerHTML = '';
            imageData = [];
            document.getElementById('fileInput').value = '';
            updateStats();
        }

        function updateStats() {
            const count = imageData.length;
            document.getElementById('imageCount').textContent = count;
            document.getElementById('stats').style.display = count > 0 ? 'block' : 'none';
            document.getElementById('downloadAllBtn').disabled = count === 0;
        }

        // Initialize display values
        updateLineWidthDisplay();
        updateOpacityDisplay();
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Million Pixel Website</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            font-family: Arial, sans-serif;
            overflow: hidden;
        }
        #app {
            display: flex;
            height: 100vh;
        }
        #sidebar {
            width: 250px;
            background-color: #f0f0f0;
            padding: 20px;
            box-shadow: 2px 0 5px rgba(0,0,0,0.1);
            z-index: 10;
        }
        #pixelCanvas {
            flex-grow: 1;
            cursor: crosshair;
        }
        h1 {
            margin-top: 0;
        }
        #zoomControls {
            margin-top: 20px;
        }
        button {
            padding: 5px 10px;
            margin-right: 5px;
        }
        #coordinates {
            margin-top: 20px;
        }
        #stats {
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div id="app">
        <div id="sidebar">
            <h1>Million Pixel Website</h1>
            <div id="zoomControls">
                <button id="zoomIn">Zoom In</button>
                <button id="zoomOut">Zoom Out</button>
                <button id="resetZoom">Reset Zoom</button>
            </div>
            <div id="coordinates">
                <p>X: <span id="xCoord"></span></p>
                <p>Y: <span id="yCoord"></span></p>
            </div>
            <div id="stats">
                <p>Pixels Sold: <span id="pixelsSold">0</span></p>
                <p>Total Revenue: $<span id="totalRevenue">0</span></p>
            </div>
        </div>
        <canvas id="pixelCanvas"></canvas>
    </div>

    <script>
        const canvas = document.getElementById('pixelCanvas');
        const ctx = canvas.getContext('2d');
        const pixelsSoldElement = document.getElementById('pixelsSold');
        const totalRevenueElement = document.getElementById('totalRevenue');
        const xCoordElement = document.getElementById('xCoord');
        const yCoordElement = document.getElementById('yCoord');

        const CANVAS_WIDTH = 1000;
        const CANVAS_HEIGHT = 1000;
        const PIXEL_SIZE = 1;
        let zoomLevel = 1;
        let offsetX = 0;
        let offsetY = 0;

        let pixelsSold = 0;
        let totalRevenue = 0;
        let pixelData = new Array(CANVAS_WIDTH * CANVAS_HEIGHT).fill(false);

        function initCanvas() {
            canvas.width = window.innerWidth - 250; // Sidebar width
            canvas.height = window.innerHeight;
            drawPixels();
        }

        function drawPixels() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            const visibleWidth = canvas.width / zoomLevel;
            const visibleHeight = canvas.height / zoomLevel;

            for (let y = 0; y < CANVAS_HEIGHT; y++) {
                for (let x = 0; x < CANVAS_WIDTH; x++) {
                    if (x >= offsetX && x < offsetX + visibleWidth &&
                        y >= offsetY && y < offsetY + visibleHeight) {
                        const index = y * CANVAS_WIDTH + x;
                        ctx.fillStyle = pixelData[index] ? 'red' : '#eee';
                        ctx.fillRect(
                            (x - offsetX) * zoomLevel,
                            (y - offsetY) * zoomLevel,
                            zoomLevel,
                            zoomLevel
                        );
                    }
                }
            }
        }

        function updateStats() {
            pixelsSoldElement.textContent = pixelsSold;
            totalRevenueElement.textContent = totalRevenue;
        }

        function handleClick(event) {
            const rect = canvas.getBoundingClientRect();
            const x = Math.floor(offsetX + (event.clientX - rect.left) / zoomLevel);
            const y = Math.floor(offsetY + (event.clientY - rect.top) / zoomLevel);

            if (x >= 0 && x < CANVAS_WIDTH && y >= 0 && y < CANVAS_HEIGHT) {
                const index = y * CANVAS_WIDTH + x;
                if (!pixelData[index]) {
                    pixelData[index] = true;
                    pixelsSold++;
                    totalRevenue++;
                    updateStats();
                    drawPixels();
                }
            }
        }

        function handleMouseMove(event) {
            const rect = canvas.getBoundingClientRect();
            const x = Math.floor(offsetX + (event.clientX - rect.left) / zoomLevel);
            const y = Math.floor(offsetY + (event.clientY - rect.top) / zoomLevel);

            xCoordElement.textContent = x;
            yCoordElement.textContent = y;
        }

        function handleZoom(direction) {
            const oldZoom = zoomLevel;
            zoomLevel = Math.max(1, Math.min(20, zoomLevel + direction));

            // Adjust offset to keep the center point fixed
            const centerX = offsetX + canvas.width / (2 * oldZoom);
            const centerY = offsetY + canvas.height / (2 * oldZoom);
            offsetX = Math.max(0, Math.min(CANVAS_WIDTH - canvas.width / zoomLevel,
                centerX - canvas.width / (2 * zoomLevel)));
            offsetY = Math.max(0, Math.min(CANVAS_HEIGHT - canvas.height / zoomLevel,
                centerY - canvas.height / (2 * zoomLevel)));

            drawPixels();
        }

        function resetZoom() {
            zoomLevel = 1;
            offsetX = 0;
            offsetY = 0;
            drawPixels();
        }

        // Event Listeners
        canvas.addEventListener('click', handleClick);
        canvas.addEventListener('mousemove', handleMouseMove);
        document.getElementById('zoomIn').addEventListener('click', () => handleZoom(1));
        document.getElementById('zoomOut').addEventListener('click', () => handleZoom(-1));
        document.getElementById('resetZoom').addEventListener('click', resetZoom);
        window.addEventListener('resize', initCanvas);

        initCanvas();
    </script>
</body>
</html>

<script>
    import { createWorker } from 'tesseract.js';
    import Tesseract from 'tesseract.js';

    // Function to apply a threshold filter to the canvas
    function applyThresholdFilter(ctx, width, height) {
        const imageData = ctx.getImageData(0, 0, width, height);
        const data = imageData.data;

        const threshold = 60; // You can adjust this threshold

        for (let i = 0; i < data.length; i += 4) {
            const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;  // Average RGB values
            const value = avg >= threshold ? 255 : 0;  // Apply threshold

            // Set new pixel values (black or white)
            data[i] = data[i + 1] = data[i + 2] = value;  // RGB channels
            // Alpha (transparency) remains unchanged (data[i + 3])
        }

        // Update the canvas with the threshold image
        ctx.putImageData(imageData, 0, 0);
    }

    // Function to crop the image from the canvas
    function cropCanvas(canvas, canvasFull, x, y, width, height) {
        const croppedCtx = canvas.getContext('2d');

        // Set the size of the new canvas to the crop size
        canvas.width = width;
        canvas.height = height;

        // Get the cropped image data from the original canvas
        const imageData = canvasFull.getContext('2d').getImageData(x, y, width, height);

        // Draw the cropped image data onto the new canvas
        croppedCtx.putImageData(imageData, 0, 0);

        return canvas;  // Return the new canvas with the cropped image
    }

    async function loadCanvasFull(canvasFull) {
        const ctx = canvasFull.getContext('2d');

        // Load the image
        const img = new Image();
        img.src = 'image.png';  // Replace with the correct image path

        return new Promise((resolve, reject) => {
            img.onload = () => {
                canvasFull.width = img.width;
                canvasFull.height = img.height;
                // Draw the image on the canvas
                ctx.drawImage(img, 0, 0);

                // Apply a threshold filter
                applyThresholdFilter(ctx, canvasFull.width, canvasFull.height);
            
                // Resolve the promise after everything is done
                resolve();
            };

            img.onerror = () => {
                reject(new Error('Image failed to load'));
            };
        });
    }

    async function recognize(canvas) {
        const worker = await createWorker('eng', 1, {
            // logger: m => console.log(m),
        });
        
          (async () => {
            await worker.setParameters({
                tessedit_pageseg_mode: Tesseract.PSM.SINGLE_TEXT,
                tessedit_char_whitelist: '0123456789',
            });
            const { data } = await worker.recognize(canvas);
            console.log(data.text);
            await worker.terminate();
        })();
    }

    async function logTesseract() {
        const canvasFull = document.getElementById('canvas-full');
        const canvasIndentified = document.getElementById('canvas-indentified');
        const canvasMisindentified = document.getElementById('canvas-misindentified');

        loadCanvasFull(canvasFull).then(() => {
            cropCanvas(canvasIndentified, canvasFull, 1360, 370, 180, 40);
            cropCanvas(canvasMisindentified, canvasFull, 1360, 425, 180, 40);
            recognize(canvasIndentified);
            recognize(canvasMisindentified);
        });
    }
</script>

<style>
  #canvas-full {
    display: none;
  }
</style>

<div class="container">
    <h1>Statistic calculator</h1>
    <button on:click={logTesseract}>Log Tesseract</button>
</div>

<canvas id="canvas-full"></canvas>
<canvas id="canvas-indentified"></canvas>
<canvas id="canvas-misindentified"></canvas>

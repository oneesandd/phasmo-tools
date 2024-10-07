<script lang="ts">
    import { createWorker } from 'tesseract.js';

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

    let video; // Global variable for the video element
    let stream; // Global variable for the media stream
    const targetPixelX = 1160; // Replace with your target pixel X coordinate
    const targetPixelY = 240; // Replace with your target pixel Y coordinate
    const targetRGB = { r: 255, g: 255, b: 255 }; // Replace with your target RGB coloR

    async function startScreenCapture() {
        try {
            // Start capturing the screen if not already started
            if (!stream) {
                stream = await startCapture({
                    video: {
                        displaySurface: "monitor",  // Use "monitor" or "browser" as needed
                    },
                    audio: false,
                });

                // Create the video element if it doesn't exist
                video = document.createElement('video');
                video.srcObject = stream;
                video.play();
            }
        } catch (error) {
            console.error('Error capturing the screen:', error);
        }
    }

    async function loadScreenForOCR(canvasFull) {
        const ctx = canvasFull.getContext('2d');

        // Ensure the video is loaded and ready
        if (video) {
            // Set the canvas size to match the video
            canvasFull.width = video.videoWidth;
            canvasFull.height = video.videoHeight;

            // Draw the current video frame onto the canvas
            ctx.drawImage(video, 0, 0);

            // Apply a threshold filter if needed
            applyThresholdFilter(ctx, canvasFull.width, canvasFull.height);
        }
    }

    async function recognize(canvas) {
        const worker = await createWorker('eng', 1, {
            // logger: m => console.log(m),
        });

        // Return a promise that resolves with the recognized text
        const { data } = await worker.recognize(canvas);
        const recognizedText = data.text;

        // Terminate the worker after recognition
        await worker.terminate();
        console.log(recognizedText);

        return recognizedText; // Return the recognized text
    }

    function containsIgnoreCase(str, search) {
        return str.toLowerCase().includes(search.toLowerCase());
    }

    async function checkVideoPixelColor() {
        const tempCanvas = document.createElement('canvas');
        const tempCtx = tempCanvas.getContext('2d');

        // Set the canvas size to match the video dimensions
        tempCanvas.width = video.videoWidth;
        tempCanvas.height = video.videoHeight;

        // Draw the current video frame onto the temporary canvas
        tempCtx.drawImage(video, 0, 0);

        // Get pixel data from the temporary canvas
        const imageData = tempCtx.getImageData(targetPixelX, targetPixelY, 1, 1);
        const pixel = imageData.data;

        // Calculate the allowable range for each RGB component
        const tolerance = 0.01; // 1%
        const allowableR = targetRGB.r * tolerance;
        const allowableG = targetRGB.g * tolerance;
        const allowableB = targetRGB.b * tolerance;

        // Check if the pixel matches the target RGB color within the allowable range
        const isColorMatch =
            (pixel[0] >= targetRGB.r - allowableR && pixel[0] <= targetRGB.r + allowableR) &&
            (pixel[1] >= targetRGB.g - allowableG && pixel[1] <= targetRGB.g + allowableG) &&
            (pixel[2] >= targetRGB.b - allowableB && pixel[2] <= targetRGB.b + allowableB);

        //console.log(pixel); // Log the pixel data for debugging
        return isColorMatch;
    }

    async function logTesseract() {
        const canvasFull = document.getElementById('canvas-full');
        await startScreenCapture();  // Start capturing the screen once

        // Check the pixel color periodically
        const pixelCheckInterval = setInterval(async () => {
            if (await checkVideoPixelColor()) {
                await processFrame(canvasFull);
            }
        }, 100); // Check every second
    }

    async function processFrame(canvasFull) {
        await loadScreenForOCR(canvasFull);  

        const canvasObjectives = document.getElementById('canvas-objectives');
        cropCanvas(canvasObjectives, canvasFull, 400, 670, 500, 300);
        //const objectives = await recognize(canvasObjectives);

        const keywordMap = {
            "cru": "Crucifix",
            "croc": "Crucifix",
            "sensor": "Motion Sensor",
            // "fire": "Firelight",
            "emf": "EMF Reader",
            "parabolic": "Parabolic Microphone",
            "parobolic": "Parabolic Microphone",
            // "photo": "Ghost Photo"
        };

        //const textObjectives = document.getElementById('objectives');
        //textObjectives.innerHTML = ''; // Clear previous results

        //for (const key in keywordMap) {
            //if (containsIgnoreCase(objectives, key)) {
                //console.log(keywordMap[key]);
                //textObjectives.innerHTML += `${keywordMap[key]} <br>`;
            //}
        //}
    }

    function startCapture(displayMediaOptions) {
      return navigator.mediaDevices
        .getDisplayMedia(displayMediaOptions)
        .catch((err) => {
          console.error(err);
          return null;
        });
    }
</script>

<style>
  #canvas-full {
    display: none;
  }
</style>

<div class="container">
    <h1>Objectives</h1>
    <button on:click={logTesseract}>Log Tesseract</button>
</div>

<canvas id="canvas-full"></canvas>
<canvas id="canvas-objectives"></canvas>
<h2 id="objectives"></h2>

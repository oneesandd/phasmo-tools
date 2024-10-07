<script lang="ts">
    import { createWorker } from 'tesseract.js';
    import Tesseract from 'tesseract.js';

    let currentXP = 0;
    let currentLevel = 1;
    let desiredLevel = 1;
    let meanXPPerMatch = 3700;
    let timePerMatch = 2.75; // Time in minutes
    let matchesRequired = 0;
    let totalTimeRequired = 0; // Time in minutes

    function calculateXP(level) {
        if (level <= 100) {
            return Math.floor(100 * Math.pow(level - 1, 1.73));
        } else {
            return Math.floor(100 * Math.pow(level - 1, 1.73)) - 889 + 4468929;
        }
    }

    function calculateMatches(currentXP, currentLevel, desiredLevel, meanXPPerMatch) {
        const currentTotalXP = calculateXP(currentLevel) + currentXP;
        const requiredTotalXP = calculateXP(desiredLevel);
        const xpNeeded = requiredTotalXP - currentTotalXP;

        if (xpNeeded > 0 && meanXPPerMatch > 0) {
            return Math.ceil(xpNeeded / meanXPPerMatch);
        }
        return 0;
    }

    function calculateTotalTime(matches, timePerMatch) {
        return matches * timePerMatch;
    }

    function formatTime(requiredMinutes) {
        const dDays = Math.floor(requiredMinutes / (24 * 60));
        const dHours = Math.floor((requiredMinutes % (24 * 60)) / 60);
        const dMinutes = requiredMinutes % 60;

        const hHours = Math.floor(requiredMinutes / 60);
        const hMinutes = Math.floor(requiredMinutes % 60);

        const mMinutes = Math.floor(requiredMinutes);

        return { dDays, dHours, dMinutes, hHours, hMinutes, mMinutes };
    }

    async function logTesseract() {

        const canvas = document.getElementById('myCanvas');
        const ctx = canvas.getContext('2d');

        // Load the image
        const img = new Image();
        img.src = 'image.png';  // Replace with the correct image path

        // Set the canvas size when the image is loaded
        img.onload = function () {
            canvas.width = img.width;
            canvas.height = img.height;
            // Draw the image on the canvas
            ctx.drawImage(img, 0, 0);

            // Apply a threshold filter
            applyThresholdFilter(ctx, canvas.width, canvas.height);

            const croppedCanvas = cropImage(canvas, 1360, 370, 180, 40);
            // Draw the cropped image back onto the original canvas (or a new one)
            ctx.clearRect(0, 0, canvas.width, canvas.height); // Clear the canvas first
            ctx.drawImage(croppedCanvas, 0, 0); // Draw the cropped image on the canvas
        };

        const applyThresholdFilter = (ctx, width, height) => {
            const imageData = ctx.getImageData(0, 0, width, height);
            const data = imageData.data;

            const threshold = 128; // You can adjust this threshold

            for (let i = 0; i < data.length; i += 4) {
                const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;  // Average RGB values
                const value = avg >= threshold ? 255 : 0;  // Apply threshold

                // Set new pixel values (black or white)
                data[i] = data[i + 1] = data[i + 2] = value;  // RGB channels
                // Alpha (transparency) remains unchanged (data[i+3])
            }

            // Update the canvas with the threshold image
            ctx.putImageData(imageData, 0, 0);
        };

        const cropImage = (canvas, x, y, width, height) => {
            const croppedCanvas = document.createElement('canvas');
            const croppedCtx = croppedCanvas.getContext('2d');

            // Set the size of the new canvas to the crop size
            croppedCanvas.width = width;
            croppedCanvas.height = height;

            // Get the cropped image data from the original canvas
            const imageData = canvas.getContext('2d').getImageData(x, y, width, height);

            // Draw the cropped image data onto the new canvas
            croppedCtx.putImageData(imageData, 0, 0);

            return croppedCanvas;  // Return the new canvas with the cropped image
        };

        const worker = await createWorker('eng', 1,{
            logger: m => console.log(m),
        });
        (async () => {
          await worker.setParameters({
              tessedit_pageseg_mode: Tesseract.PSM.SINGLE_TEXT,
              // user_defined_dpi: '70',
              tessedit_char_whitelist: '0123456789',
          });
          const { data } = await worker.recognize(canvas,
              // { rectangle }
          );
          console.log(data.text);
          await worker.terminate();
        })();
    }

    $: matchesRequired = calculateMatches(currentXP, currentLevel, desiredLevel, meanXPPerMatch);
    $: totalTimeRequired = calculateTotalTime(matchesRequired, timePerMatch);
    $: timeFormatted = formatTime(totalTimeRequired);
</script>

<style>
    * {
        box-sizing: border-box;
    }

    body {
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        margin: 0;
        padding: 0;
        background-color: #f9f9f9;
    }

    .container {
        max-width: 600px;
        margin: 3rem auto;
        padding: 2rem;
        background-color: #ffffff;
        border-radius: 16px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        text-align: center;
    }

    h1 {
        font-size: 1.8rem;
        color: #333;
        margin-bottom: 1rem;
    }

    label {
        display: block;
        font-size: 0.9rem;
        margin-bottom: 0.5rem;
        font-weight: 500;
        color: #555;
        text-align: left;
    }

    input {
        width: 100%;
        padding: 0.75rem;
        margin-bottom: 1.5rem;
        border-radius: 8px;
        border: 1px solid #ddd;
        font-size: 1rem;
        background-color: #f1f1f1;
        transition: border-color 0.3s;
    }

    input:focus {
        border-color: #007bff;
        outline: none;
    }

    .result {
        font-size: 1.2rem;
        color: #007bff;
        font-weight: bold;
        margin-top: 1rem;
    }

    .button {
        display: inline-block;
        background-color: #007bff;
        color: white;
        padding: 0.75rem 1.5rem;
        border-radius: 8px;
        font-size: 1rem;
        border: none;
        cursor: pointer;
        transition: background-color 0.3s;
    }

    .button:hover {
        background-color: #0056b3;
    }
</style>


<div class="container">
    <h1>XP Match Calculator</h1>

    <label for="currentLevel">Current Level</label>
    <input type="number" id="currentLevel" bind:value={currentLevel} min="1" max="999" />

    <label for="currentXP">Current XP</label>
    <input type="number" id="currentXP" bind:value={currentXP} min="0" />

    <label for="desiredLevel">Desired Level</label>
    <input type="number" id="desiredLevel" bind:value={desiredLevel} min="1" max="999" />

    <label for="meanXPPerMatch">Mean XP per Match</label>
    <input type="number" id="meanXPPerMatch" bind:value={meanXPPerMatch} min="1" />

    <label for="timePerMatch">Time per Match (minutes)</label>
    <input type="number" id="timePerMatch" bind:value={timePerMatch} min="1" />

    <p class="result">{matchesRequired} matches</p>
    <p class="result">
        {timeFormatted.dDays} days {timeFormatted.dHours} hours {timeFormatted.dMinutes} minutes investigating
    </p>
    <p class="result">
        {timeFormatted.hHours} hours {timeFormatted.hMinutes} minutes investigating
    </p>
    <p class="result">
        {timeFormatted.mMinutes} minutes investigating
    </p>
    <button on:click={logTesseract}>Log Tesseract</button>
</div>
<canvas id="myCanvas"></canvas>

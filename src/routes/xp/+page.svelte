<script>
    let currentXP = 0;
    let currentLevel = 1;
    let desiredLevel = 1;
    let meanXPPerMatch = 3700;
    let timePerMatch = 2; // Time in minutes
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
        {timeFormatted.dDays} days {timeFormatted.dHours} hours {timeFormatted.dMinutes} minutes
    </p>
    <p class="result">
        {timeFormatted.hHours} hours {timeFormatted.hMinutes} minutes
    </p>
    <p class="result">
        {timeFormatted.mMinutes} minutes
    </p>
</div>

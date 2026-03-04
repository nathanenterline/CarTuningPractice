# CarTuningPractice
This is a HTML file that will let you practice ECU tuning in a safe space

## Instructions
- copy code into a text editor
- save as .html
- open file

## The Goal
You want to find the "Maximum Brake Torque" (MBT)

## The Risk 
In the table, if you increase the numbers (ignition Advance) too high while the throttle is high (load lvl 4), the status will switch to ENGINE KNOCK

## The Strategy
Slowly increase the timing values in the table
- watch the Horsepower gauge
- If the HP stops going up, or you see the Knock warning, you've gone too far. Back the timing off by 2-3 degrees

# Code

<!DOCTYPE html>
<html>
<head>
    <title>Pro-Tune ECU Simulator</title>
    <style>
        body { font-family: sans-serif; background: #1a1a1a; color: #eee; padding: 20px; }
        .dashboard { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        .panel { background: #2a2a2a; padding: 20px; border-radius: 8px; border: 1px solid #444; }
        .gauge { text-align: center; font-size: 24px; margin-bottom: 10px; }
        .value { color: #00ff00; font-family: monospace; font-size: 32px; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        td, th { border: 1px solid #555; padding: 5px; text-align: center; cursor: pointer; }
        .selected { background: #444; border: 2px solid #00ff00; }
        input { width: 50px; background: #333; color: white; border: 1px solid #666; }
        .warning { color: #ff4444; font-weight: bold; animation: blink 0.5s infinite; }
        @keyframes blink { 0% { opacity: 1; } 50% { opacity: 0; } }
    </style>
</head>
<body>
    <h2>ECU Tuning Simulator v1.0</h2>
    
    <div class="dashboard">
        <div class="panel">
            <h3>Engine Telemetry</h3>
            <div class="gauge">RPM: <span id="rpmVal" class="value">1000</span></div>
            <div class="gauge">Boost: <span id="boostVal" class="value">0.0</span> psi</div>
            <div class="gauge">AFR: <span id="afrVal" class="value">14.7</span></div>
            <div class="gauge">Horsepower: <span id="hpVal" class="value">0</span></div>
            <div id="status" style="margin-top:20px;"></div>
        </div>

        <div class="panel">
            <h3>Test Bench Controls</h3>
            <label>Throttle Position: </label>
            <input type="range" id="throttle" min="0" max="100" value="0">
            <p>Target AFR (Stoich): 14.7:1</p>
            <p><i>Instructions: Adjust the table values below to maximize HP without causing engine knock or running too lean.</i></p>
        </div>
    </div>

    <div class="panel" style="margin-top:20px;">
        <h3>Ignition Timing Table (Degrees BTDC)</h3>
        <table id="ignitionTable"></table>
        <p><small>Higher timing increases HP but risks "Knock" (Engine Damage) at high load.</small></p>
    </div>

    <script>
        let rpm = 1000;
        let throttle = 0;
        let knock = false;
        
        // Simulating a 4x4 Ignition Map (RPM vs Load)
        const timingMap = [
            [10, 12, 15, 18], // Low Load
            [8, 10, 13, 16],
            [5, 8, 11, 14],
            [2, 5, 8, 10]    // High Load/Boost
        ];

        function updateSim() {
            throttle = document.getElementById('throttle').value;
            
            // Basic Physics Simulation
            let targetRpm = 1000 + (throttle * 60);
            rpm += (targetRpm - rpm) * 0.1;
            
            let boost = (throttle > 50) ? (throttle - 50) * 0.4 : 0;
            
            // Map Lookup Logic
            let rpmIdx = Math.min(Math.floor(rpm / 2000), 3);
            let loadIdx = Math.min(Math.floor(throttle / 25), 3);
            let currentTiming = timingMap[loadIdx][rpmIdx];

            // AFR Logic (Simplified)
            let afr = 14.7 - (boost * 0.15); 
            
            // Power Calculation
            let hp = (rpm * 0.05) + (boost * 15) + (currentTiming * 2);
            
            // Danger Zones
            let statusBox = document.getElementById('status');
            if (currentTiming > 25 && boost > 10) {
                statusBox.innerHTML = "<span class='warning'>ENGINE KNOCK DETECTED!</span>";
                hp *= 0.5; // Power loss from knock
            } else {
                statusBox.innerHTML = "Engine Status: Optimal";
            }

            document.getElementById('rpmVal').innerText = Math.round(rpm);
            document.getElementById('boostVal').innerText = boost.toFixed(1);
            document.getElementById('afrVal').innerText = afr.toFixed(1);
            document.getElementById('hpVal').innerText = Math.round(hp);
            
            requestAnimationFrame(updateSim);
        }

        function buildTable() {
            const table = document.getElementById('ignitionTable');
            let html = "<tr><th>Load/RPM</th><th>2k</th><th>4k</th><th>6k</th><th>8k</th></tr>";
            for(let r=0; r<4; r++) {
                html += `<tr><td>Load Lvl ${r+1}</td>`;
                for(let c=0; c<4; c++) {
                    html += `<td><input type='number' value='${timingMap[r][c]}' onchange='timingMap[${r}][${c}] = parseInt(this.value)'></td>`;
                }
                html += "</tr>";
            }
            table.innerHTML = html;
        }

        buildTable();
        updateSim();
    </script>
</body>
</html>

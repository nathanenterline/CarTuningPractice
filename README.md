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


        /* Toolbar */
        .window-header { background: #000080; color: white; padding: 4px 8px; font-weight: bold; display: flex; justify-content: space-between; }
        .toolbar { background: #f0f0f0; border-bottom: 1px solid #ccc; padding: 5px; display: flex; gap: 5px; align-items: center; }
        .btn { background: #fff; border: 1px solid #999; padding: 4px 10px; cursor: pointer; border-radius: 3px; font-weight: bold;}
        .btn:hover { background: #e6f2ff; border-color: #0066cc; }
        .btn.active { background: #cce4ff; border-color: #004499; box-shadow: inset 0 1px 3px rgba(0,0,0,0.2); }
        .btn-danger { color: red; }
        
        /* Main Layout */
        .main-content { display: flex; flex: 1; overflow: hidden; }
        
        /* Sidebar */
        .sidebar { width: 250px; background: #fff; border-right: 1px solid #aaa; padding: 5px; overflow-y: auto; font-family: monospace; }
        .tree-item { padding: 4px 5px; cursor: pointer; border: 1px solid transparent; }
        .tree-item:hover { background: #e6f2ff; }
        .tree-header { font-weight: bold; background: #333; color: white; padding: 2px; margin-top: 5px; }
        .tree-active { background: #cce4ff; border: 1px solid #0066cc; font-weight: bold; }
        .hex-addr { color: #888; font-size: 10px; display: inline-block; width: 45px; }

        /* Workspace */
        .workspace { flex: 1; display: flex; flex-direction: column; background: #111; position: relative; }
        #viewContainer { flex: 1; position: relative; background: #000; }
        #tableContainer { display: none; background: #fff; flex: 1; overflow: auto; padding: 10px; }
        
        /* Dashboard & Game UI */
        .dashboard { background: #222; color: #fff; padding: 10px; border-top: 2px solid #555; display: flex; gap: 15px; align-items: center; z-index: 10; flex-wrap: wrap;}
        .telemetry-box { background: #000; border: 1px solid #555; padding: 5px 10px; font-family: 'Courier New', monospace; font-size: 16px; color: #00ff00; min-width: 90px; text-align: center; }
        .telemetry-label { font-size: 10px; color: #aaa; display: block; text-transform: uppercase;}
        
        .score-box { background: #002200; border: 1px solid #00ff00; color: #00ff00; font-weight: bold; font-size: 24px; padding: 5px 15px; text-align: center;}
        .health-bar-container { width: 150px; background: #440000; border: 1px solid #ff0000; height: 20px; position: relative;}
        .health-bar-fill { background: #00ff00; height: 100%; width: 100%; transition: width 0.2s, background 0.2s; }
        .health-text { position: absolute; width: 100%; text-align: center; top: 2px; font-size: 11px; font-weight: bold; mix-blend-mode: difference; color: white;}

        /* HTML Table */
        table { border-collapse: collapse; width: 100%; font-family: 'Courier New', monospace; font-size: 14px; }
        th, td { border: 1px solid #ccc; padding: 5px; text-align: center; }
        th { background: #e0e0e0; }
        td input { width: 50px; border: none; text-align: center; font-family: inherit; font-size: inherit; background: transparent; }
        td input:focus { outline: 2px solid #0066cc; background: #e6f2ff; }
        .changed { color: red; font-weight: bold; }
        
        /* Overlay for Game Over */
        #gameOverScreen { display: none; position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(150,0,0,0.8); color: white; z-index: 100; flex-direction: column; justify-content: center; align-items: center; text-align: center;}
        #gameOverScreen h1 { font-size: 50px; margin: 0; text-shadow: 2px 2px 0 #000; }
    </style>
</head>
<body>

    <div class="window-header">
        <span>WinOLS - [Project: Max Power Challenge]</span>
        <span>_ □ X</span>
    </div>
    
    <div class="toolbar">
        <button class="btn" onclick="setView('3d')" id="btn3d">🧊 3D Map</button>
        <button class="btn" onclick="setView('2d')" id="btn2d">📈 2D Heatmap</button>
        <button class="btn" onclick="setView('txt')" id="btntxt">📝 TXT Editor</button>
        <span style="margin-left: 15px; font-weight: bold; color: #333;">Dyno Throttle:</span>
        <input type="range" id="throttle" min="0" max="100" value="0" style="width: 200px;">
        <button class="btn btn-danger" onclick="resetGame()" style="margin-left: auto;">🔄 Rebuild Engine</button>
    </div>

    <div class="main-content">
        <div class="sidebar">
            <div class="tree-header">Maps (Click to Edit)</div>
            
            <div class="tree-item" id="nav-boost" onclick="setActiveMap('boost')">
                <span class="hex-addr">1C2A0</span> 💨 Boost Target Map [6x6]
            </div>
            
            <div class="tree-item tree-active" id="nav-ignition" onclick="setActiveMap('ignition')">
                <span class="hex-addr">1CB72</span> ⚡ Ignition Timing [6x6]
            </div>
            
            <div style="margin-top: 20px; padding: 10px; background: #f9f9f9; border: 1px solid #ccc; font-family: sans-serif;">
                <b>Tuning Guide:</b><br>
                1. Add Boost at high Load/RPM for power.<br>
                2. If the engine Knocks, you must lower Ignition Timing in that specific area.<br>
                3. Too little timing kills power. Balance is key.
            </div>
        </div>

        <div class="workspace">
            <div id="viewContainer"></div>
            <div id="tableContainer">
                <h3 id="tableTitle">Ignition Timing (°BTDC)</h3>
                <table id="mapTable"></table>
            </div>
            
            <div id="gameOverScreen">
                <h1>ENGINE BLOWN</h1>
                <p>You ran too much timing with too much boost.</p>
                <h2>Final Score: <span id="finalScore"></span> HP</h2>
                <button class="btn" style="font-size: 20px; padding: 10px 20px;" onclick="resetGame()">Try Again</button>
            </div>
            
            <div class="dashboard">
                <div><span class="telemetry-label">Load (Pedal %)</span><div class="telemetry-box" id="dispLoad">0</div></div>
                <div><span class="telemetry-label">Engine RPM</span><div class="telemetry-box" id="dispRpm" style="color:#00ffff;">1000</div></div>
                <div><span class="telemetry-label">Boost (PSI)</span><div class="telemetry-box" id="dispBoost" style="color:#ffaa00;">0.0</div></div>
                <div><span class="telemetry-label">Timing (°)</span><div class="telemetry-box" id="dispTiming" style="color:#ffaa00;">10.0</div></div>
                
                <div style="margin-left: auto; display: flex; gap: 15px; align-items: center;">
                    <div>
                        <span class="telemetry-label">Engine Health</span>
                        <div class="health-bar-container">
                            <div class="health-bar-fill" id="healthFill"></div>
                            <div class="health-text" id="healthText">100%</div>
                        </div>
                    </div>
                    <div>
                        <span class="telemetry-label">Live Horsepower</span>
                        <div class="score-box" id="dispHp" style="color: #ff4444;">0</div>
                    </div>
                    <div>
                        <span class="telemetry-label">Max HP Record</span>
                        <div class="score-box" id="dispMaxHp" style="color: gold;">0</div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // --- DATA & GAME STATE ---
        const rpmAxis = [1000, 2000, 3000, 4000, 5000, 6000];
        const loadAxis = [0, 20, 40, 60, 80, 100];
        
        let maps = {
            ignition: {
                title: "Ignition Timing Advance (°BTDC)",
                zTitle: "Timing (°)",
                data: [
                    [10, 12, 14, 16, 18, 20],
                    [8,  11, 13, 15, 17, 19],
                    [6,  9,  12, 14, 16, 18],
                    [4,  7,  10, 13, 15, 17],
                    [2,  5,  8,  11, 13, 15],
                    [0,  3,  6,  9,  11, 13]
                ],
                factory: [] // Saved on init
            },
            boost: {
                title: "Target Boost Pressure (PSI)",
                zTitle: "Boost (PSI)",
                data: [
                    [0, 0, 0, 0, 0, 0],
                    [0, 0, 0, 0, 0, 0],
                    [0, 1, 3, 5, 5, 3],
                    [0, 3, 6, 8, 8, 6],
                    [0, 4, 8, 10, 10, 8],
                    [0, 5, 10, 12, 12, 10]
                ],
                factory: []
            }
        };

        // Deep copy factory settings
        maps.ignition.factory = JSON.parse(JSON.stringify(maps.ignition.data));
        maps.boost.factory = JSON.parse(JSON.stringify(maps.boost.data));

        let activeMap = 'ignition';
        let currentView = '3d';
        
        // Physics State
        let engineRpm = 1000;
        let actualBoost = 0;
        
        // Game State
        let engineHealth = 100;
        let maxHp = 0;
        let gameActive = true;

        // --- UI & RENDER LOGIC ---
        function setActiveMap(mapName) {
            activeMap = mapName;
            document.getElementById('nav-ignition').classList.remove('tree-active');
            document.getElementById('nav-boost').classList.remove('tree-active');
            document.getElementById('nav-' + mapName).classList.add('tree-active');
            setView(currentView); // Refresh current view with new data
        }

        function setView(view) {
            currentView = view;
            document.getElementById('btn3d').classList.remove('active');
            document.getElementById('btn2d').classList.remove('active');
            document.getElementById('btntxt').classList.remove('active');
            document.getElementById('btn' + view).classList.add('active');

            if (view === 'txt') {
                document.getElementById('viewContainer').style.display = 'none';
                document.getElementById('tableContainer').style.display = 'block';
                renderTable();
            } else {
                document.getElementById('viewContainer').style.display = 'block';
                document.getElementById('tableContainer').style.display = 'none';
                renderPlotly(view);
            }
        }

        function renderPlotly(mode) {
            let mapData = maps[activeMap];
            
            const data = [{
                z: mapData.data,
                x: rpmAxis,
                y: loadAxis,
                type: mode === '3d' ? 'surface' : 'heatmap',
                colorscale: activeMap === 'ignition' ? 'Jet' : 'Viridis',
                colorbar: { title: mapData.zTitle }
            }];

            const layout = {
                title: mapData.title,
                paper_bgcolor: '#111',
                plot_bgcolor: '#111',
                font: { color: '#fff' },
                margin: { l: 50, r: 50, b: 50, t: 50 },
                scene: {
                    xaxis: { title: 'RPM', gridcolor: '#444' },
                    yaxis: { title: 'Load (%)', gridcolor: '#444' },
                    zaxis: { title: mapData.zTitle, gridcolor: '#444' },
                    camera: { eye: {x: -1.5, y: -1.5, z: 1.2} }
                }
            };
            // Use react for performance so it updates rather than destroying the canvas
            Plotly.react('viewContainer', data, layout, {responsive: true});
        }

        function renderTable() {
            let mapData = maps[activeMap];
            document.getElementById('tableTitle').innerText = mapData.title;

            let html = "<tr><th>Load \\ RPM</th>";
            rpmAxis.forEach(rpm => html += `<th>${rpm}</th>`);
            html += "</tr>";

            for (let i = 0; i < loadAxis.length; i++) {
                html += `<tr><th>${loadAxis[i]}%</th>`;
                for (let j = 0; j < rpmAxis.length; j++) {
                    let val = mapData.data[i][j];
                    let isChanged = val !== mapData.factory[i][j];
                    let colorClass = isChanged ? "class='changed'" : "";
                    
                    html += `<td><input type="number" step="0.5" value="${val}" ${colorClass}
                             onchange="updateMapValue(${i}, ${j}, this)"></td>`;
                }
                html += "</tr>";
            }
            document.getElementById('mapTable').innerHTML = html;
        }

        function updateMapValue(loadIdx, rpmIdx, inputElem) {
            maps[activeMap].data[loadIdx][rpmIdx] = parseFloat(inputElem.value);
            inputElem.classList.add('changed');
        }

        function resetGame() {
            maps.ignition.data = JSON.parse(JSON.stringify(maps.ignition.factory));
            maps.boost.data = JSON.parse(JSON.stringify(maps.boost.factory));
            engineHealth = 100;
            maxHp = 0;
            document.getElementById('throttle').value = 0;
            document.getElementById('dispMaxHp').innerText = 0;
            document.getElementById('gameOverScreen').style.display = 'none';
            gameActive = true;
            setView(currentView);
        }

        // --- ENGINE PHYSICS & GAME LOOP ---
        function getInterpolatedValue(mapArray, rpm, load) {
            let rIdx = 0; let lIdx = 0;
            for(let i=0; i<rpmAxis.length; i++) if(rpm >= rpmAxis[i]) rIdx = i;
            for(let i=0; i<loadAxis.length; i++) if(load >= loadAxis[i]) lIdx = i;
            return mapArray[lIdx][rIdx];
        }

        function updateSim() {
            if (!gameActive) {
                requestAnimationFrame(updateSim);
                return;
            }

            let throttle = parseFloat(document.getElementById('throttle').value);
            
            // RPM Physics
            let targetRpm = 1000 + (throttle * 50);
            engineRpm += (targetRpm - engineRpm) * 0.05;

            // Map Lookups
            let liveTiming = getInterpolatedValue(maps.ignition.data, engineRpm, throttle);
            let targetBoost = getInterpolatedValue(maps.boost.data, engineRpm, throttle);

            // Turbo Spool (Lag) Physics
            actualBoost += (targetBoost - actualBoost) * 0.08;
            if (actualBoost < 0) actualBoost = 0;

            // --- THE DYNO MATH ---
            // Base power from NA Airflow
            let basePower = (engineRpm * throttle) / 2500; 
            
            // Boost multiplies air density. 14.7 psi = 1 atmosphere = ~double power.
            let boostMultiplier = 1 + (actualBoost / 14.7); 
            
            // Timing modifies efficiency. Ideal timing gives max power, too low kills it.
            let timingMultiplier = 1 + (liveTiming * 0.035); 
            if (timingMultiplier < 0.2) timingMultiplier = 0.2; // Floor to prevent negative HP
            
            let hp = basePower * boostMultiplier * timingMultiplier * 15; // Scaler for realistic numbers

            // --- THE KNOCK ALGORITHM ---
            // More boost raises cylinder pressure, meaning fuel explodes easier. 
            // Therefore, you must lower timing as boost goes up.
            let knockThreshold = 35; // The limit of the engine block
            let knockFactor = liveTiming + (actualBoost * 1.5); 
            
            let isKnocking = false;
            let healthColor = "#00ff00";

            if (throttle > 20) {
                if (knockFactor > knockThreshold) {
                    isKnocking = true;
                    // Severe power penalty during knock
                    hp *= 0.3; 
                    // Damage the engine based on how bad the knock is
                    let damage = (knockFactor - knockThreshold) * 0.2;
                    engineHealth -= damage;
                    healthColor = "#ff0000";
                } else if (knockFactor > knockThreshold - 5) {
                    // Danger zone - pinging
                    healthColor = "#ffff00"; 
                }
            }

            // Check Game Over
            if (engineHealth <= 0) {
                engineHealth = 0;
                gameActive = false;
                document.getElementById('finalScore').innerText = Math.round(maxHp);
                document.getElementById('gameOverScreen').style.display = 'flex';
                hp = 0;
            }

            // Update High Score
            if (!isKnocking && hp > maxHp && engineHealth > 0) {
                maxHp = hp;
                document.getElementById('dispMaxHp').innerText = Math.round(maxHp);
            }

            // --- UI UPDATES ---
            document.getElementById('dispLoad').innerText = throttle.toFixed(0);
            document.getElementById('dispRpm').innerText = Math.round(engineRpm);
            document.getElementById('dispBoost').innerText = actualBoost.toFixed(1);
            document.getElementById('dispTiming').innerText = liveTiming.toFixed(1);
            document.getElementById('dispHp').innerText = Math.round(hp);
            
            // Health Bar Update
            let healthFill = document.getElementById('healthFill');
            healthFill.style.width = engineHealth + '%';
            healthFill.style.background = healthColor;
            document.getElementById('healthText').innerText = Math.round(engineHealth) + '%';
            
            if (isKnocking && engineHealth > 0) {
                document.getElementById('healthText').innerText = "KNOCKING!";
            }

            requestAnimationFrame(updateSim);
        }

        // Initialize App
        setActiveMap('ignition');
        updateSim();
        
        // Handle window resizing
        window.addEventListener('resize', () => {
            if(currentView !== 'txt') Plotly.Plots.resize('viewContainer');
        });
    </script>
</body>
</html>
</html>

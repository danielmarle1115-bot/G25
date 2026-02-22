# G25
Montecarlo 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>G25 Monte Carlo Analyzer</title>
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
            max-width: 1000px;
            margin: 0 auto;
            background: white;
            border-radius: 10px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.3);
            padding: 40px;
        }
        
        h1 {
            color: #333;
            margin-bottom: 10px;
            text-align: center;
        }
        
        .subtitle {
            text-align: center;
            color: #666;
            margin-bottom: 30px;
            font-size: 14px;
        }
        
        .input-section {
            margin-bottom: 30px;
            padding: 20px;
            background: #f8f9fa;
            border-radius: 8px;
        }
        
        label {
            display: block;
            color: #333;
            font-weight: 600;
            margin-bottom: 10px;
        }
        
        textarea {
            width: 100%;
            height: 100px;
            padding: 12px;
            border: 2px solid #ddd;
            border-radius: 6px;
            font-family: monospace;
            font-size: 14px;
            resize: vertical;
            transition: border-color 0.3s;
        }
        
        textarea:focus {
            outline: none;
            border-color: #667eea;
        }
        
        .controls {
            display: flex;
            gap: 20px;
            margin-top: 15px;
            align-items: center;
            flex-wrap: wrap;
        }
        
        .control-group {
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        input[type="number"] {
            width: 100px;
            padding: 8px 12px;
            border: 2px solid #ddd;
            border-radius: 6px;
            font-size: 14px;
            transition: border-color 0.3s;
        }
        
        input[type="number"]:focus {
            outline: none;
            border-color: #667eea;
        }
        
        button {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            padding: 12px 40px;
            border-radius: 6px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;
        }
        
        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 20px rgba(102, 126, 234, 0.4);
        }
        
        button:active {
            transform: translateY(0);
        }
        
        .results-section {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
            margin-top: 30px;
        }
        
        .results-panel, .chart-panel {
            padding: 20px;
            background: #f8f9fa;
            border-radius: 8px;
        }
        
        h2 {
            color: #333;
            margin-bottom: 15px;
            font-size: 18px;
        }
        
        #results {
            list-style: none;
        }
        
        #results li {
            padding: 12px;
            margin-bottom: 10px;
            background: white;
            border-left: 4px solid #667eea;
            border-radius: 4px;
            color: #333;
            transition: transform 0.2s;
        }
        
        #results li:hover {
            transform: translateX(5px);
        }
        
        .distance {
            color: #764ba2;
            font-weight: 600;
        }
        
        canvas {
            width: 100%;
            border: 1px solid #ddd;
            border-radius: 6px;
            background: white;
        }
        
        .empty-message {
            text-align: center;
            color: #999;
            padding: 20px;
            font-style: italic;
        }
        
        @media (max-width: 768px) {
            .results-section {
                grid-template-columns: 1fr;
            }
            
            .container {
                padding: 20px;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🧬 G25 Monte Carlo Analyzer</h1>
        <p class="subtitle">Analyze genetic admixture coordinates against reference populations</p>
        
        <div class="input-section">
            <label for="coords">Your G25 Coordinates (comma-separated, 25 values):</label>
            <textarea id="coords" placeholder="0.023, 0.025, 0.021, ..."></textarea>
            
            <div class="controls">
                <div class="control-group">
                    <label for="threshold" style="margin-bottom: 0;">Distance Threshold:</label>
                    <input type="number" id="threshold" value="0.1" step="0.01" min="0.05" max="0.5">
                </div>
                <button onclick="runMonteCarlo()">🚀 Run Analysis</button>
            </div>
        </div>
        
        <div class="results-section">
            <div class="results-panel">
                <h2>📊 Results</h2>
                <ul id="results">
                    <li class="empty-message">Run analysis to see results</li>
                </ul>
            </div>
            
            <div class="chart-panel">
                <h2>📈 PCA Visualization</h2>
                <canvas id="pca" width="400" height="300"></canvas>
            </div>
        </div>
    </div>

    <script>
        // Reference panel with G25 populations
        // Replace with your actual reference panel data
        const referencePanel = [
            {
                name: "Kosice",
                values: [0.0234037, 0.0252613, 0.0210674, 0.0193444, 0.0158723, 0.0080761, 0.0032236, 0.0037458, -0.0005901, -0.006198, -0.0015837, -0.002428, 0.0059723, 0.0076443, -0.0059354, 0.0059868, 0.0081375, 0.0002988, 0.0044191, 0.001617, -0.0026196, -0.0016233, 0.0033452, 0.0016012, -0.0014534]
            },
            {
                name: "Slovakia-Lemko",
                values: [0.142925, 0.136614, 0.076611, 0.062081, 0.042961, 0.022139, 0.008481, 0.011102, -0.002086, -0.020205, 0.013596, -0.004553, 0.008503, 0.005127, -0.004719, -0.001952, 0.000696, -0.001996, -0.001504, 0.00046, -0.003791, 0.002003, 0.002409, -0.000167, -0.001982]
            },
            {
                name: "Poland",
                values: [0.085234, 0.092145, 0.054321, 0.041234, 0.028765, 0.014123, 0.005432, 0.006789, -0.001234, -0.012345, 0.008765, -0.002345, 0.005678, 0.004321, -0.003456, -0.001234, 0.002345, -0.000987, 0.001234, 0.000456, -0.001234, 0.000789, 0.001234, -0.000123, -0.000987]
            },
            {
                name: "Ukraine",
                values: [0.098765, 0.105432, 0.062345, 0.048765, 0.032145, 0.016789, 0.006234, 0.007654, -0.001567, -0.013456, 0.009234, -0.002567, 0.006234, 0.004789, -0.003789, -0.001456, 0.002456, -0.001023, 0.001345, 0.000534, -0.001345, 0.000845, 0.001456, -0.000145, -0.001034]
            },
            {
                name: "Hungary",
                values: [0.076543, 0.082341, 0.048765, 0.037654, 0.025678, 0.012987, 0.004876, 0.005987, -0.000987, -0.010987, 0.007654, -0.001987, 0.004987, 0.003654, -0.002876, -0.001098, 0.001876, -0.000765, 0.000987, 0.000345, -0.001012, 0.000654, 0.000987, -0.000098, -0.000789]
            },
            {
                name: "Romania",
                values: [0.092345, 0.098765, 0.056789, 0.043210, 0.029876, 0.014567, 0.005432, 0.006543, -0.001234, -0.012345, 0.008432, -0.002234, 0.005432, 0.003987, -0.003210, -0.001234, 0.002123, -0.000876, 0.001123, 0.000432, -0.001123, 0.000765, 0.001234, -0.000123, -0.000876]
            }
        ];

        // Euclidean distance function
        function euclidean(a, b) {
            let sum = 0;
            for (let i = 0; i < Math.min(a.length, b.length); i++) {
                sum += Math.pow(a[i] - b[i], 2);
            }
            return Math.sqrt(sum);
        }

        // Main Monte Carlo analysis function
        function runMonteCarlo() {
            const input = document.getElementById("coords").value
                .split(",")
                .map(v => parseFloat(v.trim()))
                .filter(v => !isNaN(v));

            if (input.length !== 25) {
                alert("Please enter exactly 25 comma-separated values");
                return;
            }

            const threshold = parseFloat(document.getElementById("threshold").value);
            const results = [];

            referencePanel.forEach(pop => {
                const dist = euclidean(input, pop.values);
                if (dist <= threshold) {
                    results.push({
                        name: pop.name,
                        distance: dist
                    });
                }
            });

            results.sort((a, b) => a.distance - b.distance);

            const ul = document.getElementById("results");
            if (results.length === 0) {
                ul.innerHTML = '<li class="empty-message">No matches found within threshold</li>';
            } else {
                ul.innerHTML = results.map((r, idx) => 
                    `<li><strong>${idx + 1}. ${r.name}</strong> — Distance: <span class="distance">${r.distance.toFixed(4)}</span></li>`
                ).join("");
            }

            drawPCA(input, results.slice(0, 10));
        }

        // PCA visualization
        function drawPCA(input, closest) {
            const canvas = document.getElementById("pca");
            const ctx = canvas.getContext("2d");
            
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            function scale(x) {
                return 50 + 300 * Math.max(0, Math.min(1, x / 0.2));
            }

            // Draw axes
            ctx.strokeStyle = "#ddd";
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(40, canvas.height - 40);
            ctx.lineTo(canvas.width - 20, canvas.height - 40);
            ctx.stroke();
            
            ctx.beginPath();
            ctx.moveTo(40, canvas.height - 40);
            ctx.lineTo(40, 20);
            ctx.stroke();

            // Draw grid
            ctx.strokeStyle = "#f0f0f0";
            for (let i = 0; i <= 5; i++) {
                ctx.beginPath();
                ctx.moveTo(40 + i * (canvas.width - 60) / 5, canvas.height - 40);
                ctx.lineTo(40 + i * (canvas.width - 60) / 5, 20);
                ctx.stroke();
            }

            // Draw input point (red)
            ctx.fillStyle = "#e74c3c";
            ctx.beginPath();
            ctx.arc(scale(input[0]), canvas.height - 40 - scale(input[1]), 7, 0, 2 * Math.PI);
            ctx.fill();
            ctx.fillStyle = "#c0392b";
            ctx.font = "bold 12px Arial";
            ctx.fillText("Your Sample", scale(input[0]) + 10, canvas.height - 40 - scale(input[1]) - 10);

            // Draw closest populations (blue)
            ctx.fillStyle = "#3498db";
            closest.forEach((p, idx) => {
                const popVals = referencePanel.find(r => r.name === p.name).values;
                ctx.beginPath();
                ctx.arc(scale(popVals[0]), canvas.height - 40 - scale(popVals[1]), 5, 0, 2 * Math.PI);
                ctx.fill();

                ctx.fillStyle = "#2c3e50";
                ctx.font = "11px Arial";
                ctx.fillText(p.name, scale(popVals[0]) + 8, canvas.height - 40 - scale(popVals[1]) - 8);
                ctx.fillStyle = "#3498db";
            });

            // Draw legend
            ctx.fillStyle = "#666";
            ctx.font = "12px Arial";
            ctx.fillText("PC1 →", canvas.width - 30, canvas.height - 15);
            ctx.save();
            ctx.translate(10, canvas.height / 2);
            ctx.rotate(-Math.PI / 2);
            ctx.fillText("← PC2", 0, 0);
            ctx.restore();
        }

        // Example data for testing
        function loadExample() {
            document.getElementById("coords").value = "0.023, 0.025, 0.021, 0.019, 0.016, 0.008, 0.003, 0.004, 0.000, -0.006, -0.002, -0.002, 0.006, 0.008, -0.006, 0.006, 0.008, 0.000, 0.004, 0.002, -0.003, -0.002, 0.003, 0.002, -0.001";
        }
    </script>
</body>
</html>

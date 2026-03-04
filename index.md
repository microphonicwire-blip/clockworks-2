<!DOCTYPE html>
<html>
<head>
    <title>WoS Dashboard</title>
    <style>
        body { background: #111; color: #0f0; font-family: monospace; padding: 20px; }
        h1 { color: #0f0; }
        .card { background: #1a1a1a; border: 1px solid #0f0; padding: 15px; margin: 10px 0; border-radius: 6px; }
        .label { color: #888; font-size: 12px; }
        .value { font-size: 20px; margin-top: 4px; }
    </style>
</head>
<body>
    <h1>🚀 WoS Ship Dashboard</h1>
    <div class="card">
        <div class="label">POSITION</div>
        <div class="value" id="position">--</div>
    </div>
    <div class="card">
        <div class="label">HEALTH</div>
        <div class="value" id="health">--</div>
    </div>
    <div class="card">
        <div class="label">LAST UPDATE</div>
        <div class="value" id="time">--</div>
    </div>

    <script>
        async function fetchData() {
            try {
                const res = await fetch("https://cold-wildflower-4f31.microphonicwire.workers.dev");
                const data = await res.json();

                document.getElementById("position").textContent = data.position ?? "--";
                document.getElementById("health").textContent = 
                    data.health != null ? (data.health * 100).toFixed(1) + "%" : "--";
                document.getElementById("time").textContent = 
                    data.time != null ? new Date(data.time * 1000).toLocaleTimeString() : "--";
            } catch (e) {
                console.error("Fetch failed:", e);
            }
        }

        fetchData();
        setInterval(fetchData, 3000);
    </script>
</body>
</html>

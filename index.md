<!DOCTYPE html>
<html>
<head>
    <title>WoS Dashboard</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { background: #111; color: #0f0; font-family: monospace; padding: 20px; }
        h1 { color: #0f0; margin-bottom: 20px; }
        #ships { display: flex; flex-wrap: wrap; gap: 10px; margin-bottom: 20px; }
        .ship-btn {
            background: #1a1a1a; border: 1px solid #0f0; color: #0f0;
            padding: 10px 20px; cursor: pointer; font-family: monospace;
            font-size: 14px; border-radius: 4px;
        }
        .ship-btn:hover { background: #0f0; color: #111; }
        .ship-btn.active { background: #0f0; color: #111; }
        #panel { display: none; }
        #panel-title {
            font-size: 20px; margin-bottom: 15px;
            display: flex; align-items: center; gap: 15px;
        }
        .delete-btn {
            background: none; border: 1px solid #f00; color: #f00;
            padding: 4px 10px; cursor: pointer; font-family: monospace;
            font-size: 12px; border-radius: 4px;
        }
        .delete-btn:hover { background: #f00; color: #111; }
        .row {
            background: #1a1a1a; border: 1px solid #333;
            padding: 12px 16px; margin-bottom: 8px; border-radius: 4px;
            display: flex; gap: 20px;
        }
        .row-key { color: #888; min-width: 150px; }
        .row-value { color: #0f0; }
        #no-ships { color: #555; }
    </style>
</head>
<body>
    <h1>🚀 WoS Dashboard</h1>
    <div id="ships"><span id="no-ships">No ships online.</span></div>
    <div id="panel">
        <div id="panel-title">
            <span id="panel-name"></span>
            <button class="delete-btn" onclick="deleteShip()">Remove</button>
        </div>
        <div id="values"></div>
    </div>

    <script>
        const URL = "https://cold-wildflower-4f31.microphonicwire.workers.dev";
        let selectedShip = null;
        let knownShips = [];

        async function fetchShips() {
            try {
                const res = await fetch(URL);
                const ships = await res.json();

                if (JSON.stringify(ships) !== JSON.stringify(knownShips)) {
                    knownShips = ships;
                    renderShipButtons();
                }
            } catch (e) {}
        }

        function renderShipButtons() {
            const container = document.getElementById("ships");
            const noShips = document.getElementById("no-ships");

            container.innerHTML = "";

            if (knownShips.length === 0) {
                container.appendChild(noShips);
                return;
            }

            knownShips.forEach(name => {
                const btn = document.createElement("button");
                btn.className = "ship-btn" + (name === selectedShip ? " active" : "");
                btn.textContent = name;
                btn.onclick = () => selectShip(name);
                container.appendChild(btn);
            });

            if (selectedShip && !knownShips.includes(selectedShip)) {
                selectedShip = null;
                document.getElementById("panel").style.display = "none";
            }
        }

        async function selectShip(name) {
            selectedShip = name;
            renderShipButtons();
            document.getElementById("panel").style.display = "block";
            document.getElementById("panel-name").textContent = name;
            await fetchValues();
        }

        async function fetchValues() {
            if (!selectedShip) return;
            try {
                const res = await fetch(URL + "?name=" + encodeURIComponent(selectedShip));
                const values = await res.json();

                const container = document.getElementById("values");
                container.innerHTML = "";

                Object.entries(values).forEach(([key, val]) => {
                    const row = document.createElement("div");
                    row.className = "row";
                    row.innerHTML = `<span class="row-key">${key}</span><span class="row-value">${val}</span>`;
                    container.appendChild(row);
                });
            } catch (e) {}
        }

        async function deleteShip() {
            if (!selectedShip) return;
            await fetch(URL + "?name=" + encodeURIComponent(selectedShip), { method: "DELETE" });
            selectedShip = null;
            document.getElementById("panel").style.display = "none";
            await fetchShips();
        }

        fetchShips();
        setInterval(fetchShips, 2000);
        setInterval(fetchValues, 500);
    </script>
</body>
</html>

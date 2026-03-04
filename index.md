<!DOCTYPE html>
<html lang="en">
<head>
    <title>WoS Dashboard</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }

        body {
            background: #0d0d0d;
            color: #e0e0e0;
            font-family: 'Segoe UI', monospace;
            min-height: 100vh;
        }

        .sidebar {
            position: fixed;
            left: 0; top: 0; bottom: 0;
            width: 220px;
            background: #141414;
            border-right: 1px solid #222;
            padding: 20px 0;
            overflow-y: auto;
        }

        .sidebar-title {
            color: #555;
            font-size: 10px;
            text-transform: uppercase;
            letter-spacing: 2px;
            padding: 0 20px 15px;
        }

        .ship-btn {
            display: block;
            width: 100%;
            background: none;
            border: none;
            color: #aaa;
            text-align: left;
            padding: 10px 20px;
            cursor: pointer;
            font-family: inherit;
            font-size: 14px;
            border-left: 3px solid transparent;
            transition: all 0.15s;
        }

        .ship-btn:hover { background: #1a1a1a; color: #fff; }

        .ship-btn.active {
            background: #1a1a1a;
            color: #fff;
            border-left-color: #4f9eff;
        }

        .ship-dot {
            display: inline-block;
            width: 7px; height: 7px;
            border-radius: 50%;
            background: #2ecc71;
            margin-right: 8px;
        }

        .main {
            margin-left: 220px;
            padding: 30px;
            min-height: 100vh;
        }

        .top-bar {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 30px;
        }

        .top-bar h1 {
            font-size: 22px;
            font-weight: 500;
            color: #fff;
        }

        .top-bar .subtitle {
            font-size: 12px;
            color: #444;
            margin-top: 4px;
        }

        .delete-btn {
            background: none;
            border: 1px solid #c0392b;
            color: #c0392b;
            padding: 7px 16px;
            cursor: pointer;
            font-family: inherit;
            font-size: 13px;
            border-radius: 5px;
            transition: all 0.15s;
        }

        .delete-btn:hover { background: #c0392b; color: #fff; }

        .grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
            gap: 12px;
        }

        .card {
            background: #141414;
            border: 1px solid #1f1f1f;
            border-radius: 8px;
            padding: 18px 20px;
            transition: border-color 0.15s;
        }

        .card:hover { border-color: #333; }

        .card-key {
            font-size: 11px;
            color: #555;
            text-transform: uppercase;
            letter-spacing: 1.5px;
            margin-bottom: 8px;
        }

        .card-value {
            font-size: 18px;
            color: #e0e0e0;
            word-break: break-word;
        }

        .last-update {
            font-size: 11px;
            color: #333;
            margin-top: 25px;
        }

        .empty {
            color: #333;
            font-size: 14px;
            margin-top: 20px;
        }

        .no-selection {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 60vh;
            color: #2a2a2a;
            font-size: 16px;
        }
    </style>
</head>
<body>

<div class="sidebar">
    <div class="sidebar-title">Ships</div>
    <div id="ship-list"><div style="padding:0 20px;color:#333;font-size:13px">None online</div></div>
</div>

<div class="main">
    <div id="no-selection" class="no-selection">← Select a ship</div>

    <div id="panel" style="display:none">
        <div class="top-bar">
            <div>
                <h1 id="panel-name"></h1>
                <div class="subtitle" id="panel-update"></div>
            </div>
            <button class="delete-btn" onclick="deleteShip()">Remove Ship</button>
        </div>
        <div class="grid" id="values"></div>
        <div class="last-update" id="last-update"></div>
    </div>
</div>

<script>
    const WORKER = "https://cold-wildflower-4f31.microphonicwire.workers.dev";
    let selectedShip = null;
    let knownShips = [];

    async function fetchShips() {
        try {
            const res = await fetch(WORKER);
            const ships = await res.json();
            const names = ships.map(s => s.name);

            if (JSON.stringify(names) !== JSON.stringify(knownShips)) {
                knownShips = names;
                renderSidebar(ships);
            }

            if (selectedShip && !knownShips.includes(selectedShip)) {
                selectedShip = null;
                document.getElementById("panel").style.display = "none";
                document.getElementById("no-selection").style.display = "flex";
            }
        } catch (e) {}
    }

    function renderSidebar(ships) {
        const list = document.getElementById("ship-list");
        list.innerHTML = "";

        if (ships.length === 0) {
            list.innerHTML = '<div style="padding:0 20px;color:#333;font-size:13px">None online</div>';
            return;
        }

        ships.forEach(ship => {
            const btn = document.createElement("button");
            btn.className = "ship-btn" + (ship.name === selectedShip ? " active" : "");
            btn.innerHTML = `<span class="ship-dot"></span>${ship.name}`;
            btn.onclick = () => selectShip(ship.name);
            list.appendChild(btn);
        });
    }

    async function selectShip(name) {
        selectedShip = name;
        document.getElementById("no-selection").style.display = "none";
        document.getElementById("panel").style.display = "block";
        document.getElementById("panel-name").textContent = name;
        renderSidebar(knownShips.map(n => ({ name: n })));
        await fetchValues();
    }

    async function fetchValues() {
        if (!selectedShip) return;
        try {
            const res = await fetch(WORKER + "?name=" + encodeURIComponent(selectedShip));
            const data = await res.json();
            if (!data.values) return;

            const container = document.getElementById("values");
            container.innerHTML = "";

            Object.entries(data.values).forEach(([key, val]) => {
                const card = document.createElement("div");
                card.className = "card";
                card.innerHTML = `<div class="card-key">${key}</div><div class="card-value">${val}</div>`;
                container.appendChild(card);
            });

            if (data.updated_at) {
                const t = new Date(data.updated_at);
                document.getElementById("panel-update").textContent =
                    "Last updated: " + t.toLocaleTimeString();
            }
        } catch (e) {}
    }

    async function deleteShip() {
        if (!selectedShip) return;
        try {
            await fetch(WORKER + "?name=" + encodeURIComponent(selectedShip), { method: "DELETE" });
            selectedShip = null;
            document.getElementById("panel").style.display = "none";
            document.getElementById("no-selection").style.display = "flex";
            await fetchShips();
        } catch (e) {
            console.error("Delete failed:", e);
        }
    }

    fetchShips();
    setInterval(fetchShips, 3000);
    setInterval(fetchValues, 2000);
</script>

</body>
</html>

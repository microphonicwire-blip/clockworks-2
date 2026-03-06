<!DOCTYPE html>
<html lang="en">
<head>
    <title>WoS Dashboard</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }

        :root {
            --bg:      #090909;
            --bg2:     #0d0d0d;
            --bg3:     #101010;
            --bg4:     #141414;
            --b1:      #181818;
            --b2:      #222;
            --b3:      #2a2a2a;
            --t1:      #e8e8e8;
            --t2:      #888;
            --t3:      #555;
            --t4:      #333;
            --t5:      #1e1e1e;
            --accent:  #4f9eff;
            --green:   #2ecc71;
            --red:     #e74c3c;
            --orange:  #f39c12;
        }

        html, body { height: 100%; overflow: hidden; }

        body {
            background: var(--bg);
            color: var(--t1);
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
            display: flex;
            flex-direction: column;
        }

        /* ── TOPBAR ── */
        .topbar {
            height: 50px;
            background: var(--bg3);
            border-bottom: 1px solid var(--b1);
            display: flex;
            align-items: center;
            padding: 0 18px;
            gap: 8px;
            flex-shrink: 0;
            overflow-x: auto;
        }
        .topbar::-webkit-scrollbar { display: none; }

        #no-ships-msg {
            font-size: 12px;
            color: var(--t4);
        }

        .ship-tab {
            display: flex;
            align-items: stretch;
            border: 1px solid var(--b2);
            border-radius: 5px;
            overflow: hidden;
            flex-shrink: 0;
            transition: border-color 0.15s;
        }
        .ship-tab.active {
            border-color: rgba(79,158,255,0.35);
            background: rgba(79,158,255,0.05);
        }

        .ship-tab-btn {
            background: none;
            border: none;
            color: var(--t3);
            padding: 6px 13px;
            cursor: pointer;
            font-family: inherit;
            font-size: 12px;
            transition: color 0.15s;
        }
        .ship-tab-btn:hover { color: var(--t1); }
        .ship-tab.active .ship-tab-btn { color: var(--accent); }

        .ship-tab-del {
            background: none;
            border: none;
            border-left: 1px solid var(--b2);
            color: var(--t5);
            padding: 0 8px;
            cursor: pointer;
            font-size: 10px;
            transition: all 0.15s;
        }
        .ship-tab-del:hover { color: var(--red); background: rgba(231,76,60,0.1); }

        .topbar-right {
            margin-left: auto;
            flex-shrink: 0;
        }

        .status-pill {
            display: flex;
            align-items: center;
            gap: 7px;
            background: var(--bg2);
            border: 1px solid var(--b1);
            border-radius: 20px;
            padding: 5px 13px;
            font-size: 11px;
            color: var(--t3);
            white-space: nowrap;
        }

        .status-dot {
            width: 7px; height: 7px;
            border-radius: 50%;
            flex-shrink: 0;
        }
        .status-dot.online {
            background: var(--green);
            animation: statusPulse 2s ease-in-out infinite;
        }
        .status-dot.offline { background: var(--red); }

        @keyframes statusPulse {
            0%,100% { box-shadow: 0 0 0 0 rgba(46,204,113,0.5); }
            50%      { box-shadow: 0 0 0 5px rgba(46,204,113,0); }
        }

        /* ── MAIN ── */
        .main {
            flex: 1;
            display: flex;
            flex-direction: column;
            min-height: 0;
            overflow: hidden;
        }

        .centered-empty {
            flex: 1;
            display: flex;
            align-items: center;
            justify-content: center;
            color: var(--t5);
            font-size: 13px;
        }

        /* ── MESSAGE AREA ── */
        .message-area {
            flex: 1;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 28px 24px 20px;
            min-height: 0;
            background: var(--bg2);
            position: relative;
            overflow: hidden;
        }

        /* subtle animated grid lines */
        .message-area::before {
            content: '';
            position: absolute;
            inset: 0;
            background-image:
                linear-gradient(var(--b1) 1px, transparent 1px),
                linear-gradient(90deg, var(--b1) 1px, transparent 1px);
            background-size: 40px 40px;
            opacity: 0.4;
            pointer-events: none;
        }

        .msg-label {
            font-size: 9px;
            text-transform: uppercase;
            letter-spacing: 3px;
            color: var(--t4);
            margin-bottom: 22px;
            position: relative;
        }

        .msg-text {
            font-size: clamp(16px, 3vw, 30px);
            color: #fff;
            text-align: center;
            max-width: 680px;
            line-height: 1.5;
            word-break: break-word;
            position: relative;
        }

        .msg-text.waiting { color: var(--t4); font-size: 16px; }

        .msg-text.pop {
            animation: msgPop 0.4s cubic-bezier(0.16,1,0.3,1);
        }

        @keyframes msgPop {
            from { opacity: 0; transform: translateY(10px); filter: blur(6px); }
            to   { opacity: 1; transform: translateY(0);   filter: blur(0);   }
        }

        .msg-meta {
            margin-top: 18px;
            display: flex;
            align-items: center;
            gap: 8px;
            opacity: 0;
            transition: opacity 0.4s;
            position: relative;
        }
        .msg-meta.visible { opacity: 1; }

        .meta-chip {
            display: flex;
            align-items: center;
            gap: 7px;
            background: var(--bg3);
            border: 1px solid var(--b1);
            border-radius: 4px;
            padding: 4px 11px;
        }
        .meta-key {
            font-size: 9px;
            text-transform: uppercase;
            letter-spacing: 1px;
            color: var(--t4);
        }
        .meta-val {
            font-size: 12px;
            color: var(--t2);
        }
        .meta-dot { color: var(--t5); font-size: 10px; }

        /* ── LOG PANEL ── */
        .log-panel {
            height: 145px;
            flex-shrink: 0;
            border-top: 1px solid var(--b1);
            background: var(--bg);
            overflow-y: auto;
            padding: 8px 18px 10px;
        }
        .log-panel::-webkit-scrollbar { width: 3px; }
        .log-panel::-webkit-scrollbar-track { background: transparent; }
        .log-panel::-webkit-scrollbar-thumb { background: var(--b2); border-radius: 2px; }

        .log-cols {
            display: flex;
            gap: 10px;
            font-size: 9px;
            text-transform: uppercase;
            letter-spacing: 1.5px;
            color: var(--t5);
            margin-bottom: 5px;
            padding-bottom: 4px;
            border-bottom: 1px solid var(--b1);
            position: sticky;
            top: 0;
            background: var(--bg);
        }
        .log-cols .lc { flex: 1; }
        .lc-time  { max-width: 65px !important; }
        .lc-ship  { max-width: 80px !important; }
        .lc-plr   { max-width: 90px !important; }
        .lc-msg   { flex: 3 !important; }
        .lc-ping  { max-width: 50px !important; text-align: right; }

        .log-row {
            display: flex;
            gap: 10px;
            font-size: 11px;
            padding: 3px 0;
            border-bottom: 1px solid #0e0e0e;
            color: var(--t4);
            transition: color 0.2s;
        }
        .log-row:hover { color: var(--t2); }
        .log-row.fresh { color: var(--t3); animation: logSlide 0.25s ease; }
        .log-row > span {
            flex: 1;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
        }

        @keyframes logSlide {
            from { opacity: 0; transform: translateX(-5px); }
            to   { opacity: 1; transform: translateX(0); }
        }

        /* ── BOTTOM BAR ── */
        .bottombar {
            flex-shrink: 0;
            height: clamp(48px, 12.5vh, 88px);
            background: var(--bg3);
            border-top: 1px solid var(--b1);
            display: flex;
            align-items: center;
            padding: 0 18px;
            gap: 0;
            overflow-x: auto;
        }
        .bottombar::-webkit-scrollbar { display: none; }

        .bstat {
            display: flex;
            flex-direction: column;
            justify-content: center;
            gap: 4px;
            padding: 0 22px;
            border-right: 1px solid var(--b1);
            height: 100%;
        }
        .bstat:first-child { padding-left: 0; }
        .bstat:last-child  { border-right: none; }

        .bs-label {
            font-size: 8px;
            text-transform: uppercase;
            letter-spacing: 1.5px;
            color: var(--t5);
        }
        .bs-val {
            font-family: 'Consolas', 'SF Mono', monospace;
            font-size: 15px;
            color: var(--t3);
            transition: color 0.3s;
        }
        .bs-val.good { color: var(--green); }
        .bs-val.ok   { color: var(--orange); }
        .bs-val.bad  { color: var(--red); }
    </style>
</head>
<body>

<!-- TOPBAR -->
<div class="topbar">
    <div id="ship-tabs" style="display:flex;gap:8px;align-items:center;"></div>
    <span id="no-ships-msg">No ships online</span>
    <div class="topbar-right">
        <div class="status-pill">
            <div class="status-dot offline" id="status-dot"></div>
            <span id="status-text">Disconnected</span>
        </div>
    </div>
</div>

<!-- MAIN -->
<div class="main">

    <div class="centered-empty" id="empty-view">Waiting for a ship to come online...</div>

    <div id="dash-view" style="display:none;flex:1;flex-direction:column;min-height:0;overflow:hidden;">

        <div class="message-area">
            <div class="msg-label">Last Transmission</div>
            <div class="msg-text waiting" id="msg-text">Waiting for transmission...</div>
            <div class="msg-meta" id="msg-meta">
                <div class="meta-chip">
                    <span class="meta-key">From</span>
                    <span class="meta-val" id="msg-sender">—</span>
                </div>
                <span class="meta-dot">·</span>
                <div class="meta-chip">
                    <span class="meta-key">At</span>
                    <span class="meta-val" id="msg-time">—</span>
                </div>
            </div>
        </div>

        <div class="log-panel">
            <div class="log-cols">
                <span class="lc lc-time">Time</span>
                <span class="lc lc-ship">Ship</span>
                <span class="lc lc-plr">Player</span>
                <span class="lc lc-msg">Message</span>
                <span class="lc lc-ping">Ping</span>
            </div>
            <div id="log-entries">
                <div style="color:var(--t5);font-size:11px;padding:8px 0">No transmissions yet</div>
            </div>
        </div>

    </div>
</div>

<!-- BOTTOM BAR -->
<div class="bottombar">
    <div class="bstat">
        <span class="bs-label">Worker Ping</span>
        <span class="bs-val" id="bs-worker">—</span>
    </div>
    <div class="bstat">
        <span class="bs-label">Pages Ping</span>
        <span class="bs-val" id="bs-pages">—</span>
    </div>
    <div class="bstat">
        <span class="bs-label">Last Recv</span>
        <span class="bs-val" id="bs-recv" style="font-size:12px">—</span>
    </div>
    <div class="bstat">
        <span class="bs-label">Ships Online</span>
        <span class="bs-val" id="bs-ships">0</span>
    </div>
    <div class="bstat">
        <span class="bs-label">Total Msgs</span>
        <span class="bs-val" id="bs-msgs">0</span>
    </div>
</div>

<script>
    const WORKER = "https://cold-wildflower-4f31.microphonicwire.workers.dev";

    let selectedShip  = null;
    let ships         = [];
    let logEntries    = [];
    let lastMsgKey    = null;
    let lastDataTime  = 0;
    let totalMsgs     = 0;
    let workerPing    = null;
    let pagesPing     = null;

    // ── helpers ──────────────────────────────────────────────
    function pingClass(ms) {
        if (ms == null) return '';
        if (ms < 150)   return 'good';
        if (ms < 400)   return 'ok';
        return 'bad';
    }
    function fms(ms) { return ms != null ? ms + 'ms' : '—'; }
    function now()   { return new Date().toLocaleTimeString(); }

    // ── fetch ships ───────────────────────────────────────────
    async function fetchShips() {
        const t = Date.now();
        try {
            const res = await fetch(WORKER);
            workerPing = Date.now() - t;
            ships = res.ok ? (await res.json()) : [];
            if (!Array.isArray(ships)) ships = [];
        } catch {
            workerPing = Date.now() - t;
            ships = [];
        }

        renderTabs();
        updateBottomBar();

        if (!selectedShip && ships.length > 0) selectShip(ships[0].name);
        if (selectedShip && !ships.find(s => s.name === selectedShip)) {
            selectedShip = null;
            showEmpty();
        }
    }

    // ── fetch values ──────────────────────────────────────────
    async function fetchValues() {
        if (!selectedShip) return;
        const t = Date.now();
        try {
            const res = await fetch(WORKER + "?name=" + encodeURIComponent(selectedShip));
            pagesPing = Date.now() - t;
            if (!res.ok) throw new Error();
            const data = await res.json();

            if (data && data.values && Object.keys(data.values).length > 0) {
                const key = data.updated_at + "|" + JSON.stringify(data.values);
                if (key !== lastMsgKey) {
                    lastMsgKey   = key;
                    lastDataTime = Date.now();
                    updateDisplay(data.values, data.updated_at);
                }
            }

            setOnline(lastDataTime > 0 && Date.now() - lastDataTime < 12000);
        } catch {
            pagesPing = Date.now() - t;
            setOnline(false);
        }
        updateBottomBar();
    }

    // ── render tabs ───────────────────────────────────────────
    function renderTabs() {
        const tabsEl    = document.getElementById('ship-tabs');
        const noShipsEl = document.getElementById('no-ships-msg');
        tabsEl.innerHTML = '';

        if (ships.length === 0) {
            noShipsEl.style.display = 'block';
            document.getElementById('bs-ships').textContent = '0';
            return;
        }

        noShipsEl.style.display = 'none';
        document.getElementById('bs-ships').textContent = ships.length;

        ships.forEach(ship => {
            const wrap = document.createElement('div');
            wrap.className = 'ship-tab' + (ship.name === selectedShip ? ' active' : '');

            const btn = document.createElement('button');
            btn.className = 'ship-tab-btn';
            btn.textContent = ship.name;
            btn.onclick = () => selectShip(ship.name);

            const del = document.createElement('button');
            del.className = 'ship-tab-del';
            del.textContent = '✕';
            del.title = 'Remove ship';
            del.onclick = e => { e.stopPropagation(); deleteShip(ship.name); };

            wrap.appendChild(btn);
            wrap.appendChild(del);
            tabsEl.appendChild(wrap);
        });
    }

    // ── select ship ───────────────────────────────────────────
    function selectShip(name) {
        selectedShip = name;
        lastMsgKey   = null;
        lastDataTime = 0;

        document.getElementById('empty-view').style.display  = 'none';
        document.getElementById('dash-view').style.display   = 'flex';

        renderTabs();
        fetchValues();
    }

    function showEmpty() {
        document.getElementById('empty-view').style.display = 'flex';
        document.getElementById('dash-view').style.display  = 'none';
        setOnline(false);
        renderTabs();
    }

    // ── connection status ─────────────────────────────────────
    function setOnline(state) {
        const dot  = document.getElementById('status-dot');
        const text = document.getElementById('status-text');
        if (state) {
            dot.className  = 'status-dot online';
            text.textContent = (selectedShip || '?') + ' · Live';
            text.style.color = 'var(--green)';
        } else {
            dot.className  = 'status-dot offline';
            text.textContent = selectedShip ? selectedShip + ' · No Signal' : 'Disconnected';
            text.style.color = 'var(--t3)';
        }
    }

    // ── update display ────────────────────────────────────────
    function updateDisplay(values, updatedAt) {
        const text   = values.Text   ?? values.text   ?? null;
        const player = values.Player ?? values.player ?? null;
        const time   = updatedAt ? new Date(updatedAt).toLocaleTimeString() : now();

        const msgEl  = document.getElementById('msg-text');
        const metaEl = document.getElementById('msg-meta');

        if (text) {
            msgEl.classList.remove('waiting', 'pop');
            void msgEl.offsetWidth;
            msgEl.classList.add('pop');
            msgEl.textContent = text;
        }

        document.getElementById('msg-sender').textContent = player || '—';
        document.getElementById('msg-time').textContent   = time;
        metaEl.classList.add('visible');

        totalMsgs++;
        logEntries.unshift({ time, ship: selectedShip || '—', player: player || '—', msg: text || JSON.stringify(values), ping: pagesPing });
        if (logEntries.length > 100) logEntries.pop();
        renderLog();
        updateBottomBar();
    }

    // ── log ───────────────────────────────────────────────────
    function renderLog() {
        const el = document.getElementById('log-entries');
        if (logEntries.length === 0) {
            el.innerHTML = '<div style="color:var(--t5);font-size:11px;padding:8px 0">No transmissions yet</div>';
            return;
        }
        el.innerHTML = logEntries.map((e, i) => `
            <div class="log-row ${i === 0 ? 'fresh' : ''}">
                <span class="lc-time">${e.time}</span>
                <span class="lc-ship">${e.ship}</span>
                <span class="lc-plr">${e.player}</span>
                <span class="lc-msg">${e.msg}</span>
                <span class="lc-ping">${fms(e.ping)}</span>
            </div>`).join('');
    }

    // ── bottom bar ────────────────────────────────────────────
    function updateBottomBar() {
        const wEl = document.getElementById('bs-worker');
        wEl.textContent = fms(workerPing);
        wEl.className   = 'bs-val ' + pingClass(workerPing);

        const pEl = document.getElementById('bs-pages');
        pEl.textContent = fms(pagesPing);
        pEl.className   = 'bs-val ' + pingClass(pagesPing);

        if (lastDataTime) document.getElementById('bs-recv').textContent = new Date(lastDataTime).toLocaleTimeString();
        document.getElementById('bs-msgs').textContent = totalMsgs;
    }

    // ── delete ship ───────────────────────────────────────────
    async function deleteShip(name) {
        try {
            const res = await fetch(WORKER + "?name=" + encodeURIComponent(name), { method: "DELETE" });
            if (!res.ok) throw new Error("Delete failed");
            if (selectedShip === name) { selectedShip = null; showEmpty(); }
            await fetchShips();
        } catch (e) { console.error("Delete error:", e); }
    }

    // ── init ──────────────────────────────────────────────────
    fetchShips();
    setInterval(fetchShips,  3000);
    setInterval(fetchValues, 1000);
</script>
</body>
</html>

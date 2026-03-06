<!DOCTYPE html>
<html lang="en">
<head>
    <title>WoS Dashboard</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@300;400;500;600&family=IBM+Plex+Sans:wght@300;400;500&display=swap" rel="stylesheet">
    <style>
        *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

        :root {
            --bg:     #07070a;
            --bg2:    #0a0a0e;
            --bg3:    #0d0d12;
            --bg4:    #111118;
            --b1:     #16161f;
            --b2:     #1e1e2a;
            --b3:     #262635;
            --t1:     #e4e4f0;
            --t2:     #9090a8;
            --t3:     #565668;
            --t4:     #303040;
            --t5:     #1a1a24;
            --accent: #4f9eff;
            --green:  #2ecc71;
            --red:    #e74c3c;
            --orange: #f39c12;
            --mono:   'IBM Plex Mono', monospace;
            --sans:   'IBM Plex Sans', sans-serif;
        }

        html, body {
            height: 100%;
            overflow: hidden;
            background: var(--bg);
            color: var(--t1);
            font-family: var(--sans);
        }

        body { display: flex; flex-direction: column; }

        /* scrollbar */
        ::-webkit-scrollbar { width: 3px; height: 3px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: var(--b2); border-radius: 2px; }

        /* ── TOPBAR ───────────────────────────────────────────── */
        .topbar {
            height: 46px;
            min-height: 46px;
            background: var(--bg3);
            border-bottom: 1px solid var(--b1);
            display: flex;
            align-items: center;
            padding: 0 14px;
            gap: 7px;
            overflow-x: auto;
            flex-shrink: 0;
        }
        .topbar::-webkit-scrollbar { display: none; }

        #no-ships-msg {
            font-family: var(--mono);
            font-size: 11px;
            color: var(--t4);
            letter-spacing: 0.05em;
        }

        .ship-tab {
            display: flex;
            align-items: stretch;
            border: 1px solid var(--b2);
            border-radius: 4px;
            overflow: hidden;
            flex-shrink: 0;
            transition: border-color 0.15s, background 0.15s;
        }
        .ship-tab:hover { border-color: var(--b3); }
        .ship-tab.active {
            border-color: rgba(79,158,255,0.3);
            background: rgba(79,158,255,0.04);
        }
        .ship-tab-btn {
            background: none;
            border: none;
            color: var(--t3);
            padding: 5px 12px;
            cursor: pointer;
            font-family: var(--mono);
            font-size: 11px;
            letter-spacing: 0.05em;
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
            font-size: 9px;
            font-family: var(--mono);
            transition: color 0.15s, background 0.15s;
        }
        .ship-tab-del:hover { color: var(--red); background: rgba(231,76,60,0.08); }

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
            padding: 4px 12px;
            font-family: var(--mono);
            font-size: 10px;
            color: var(--t3);
            white-space: nowrap;
            letter-spacing: 0.05em;
        }
        .status-dot {
            width: 6px; height: 6px;
            border-radius: 50%;
            flex-shrink: 0;
            background: var(--t4);
        }
        .status-dot.online {
            background: var(--green);
            animation: pulse 2s ease-in-out infinite;
        }
        .status-dot.offline { background: var(--red); }
        @keyframes pulse {
            0%,100% { box-shadow: 0 0 0 0 rgba(46,204,113,0.5); }
            50%      { box-shadow: 0 0 0 4px rgba(46,204,113,0); }
        }

        /* ── MAIN ─────────────────────────────────────────────── */
        .main {
            flex: 1;
            display: flex;
            flex-direction: column;
            min-height: 0;
            overflow: hidden;
            position: relative;
        }

        .empty-view {
            flex: 1;
            display: flex;
            align-items: center;
            justify-content: center;
            color: var(--t5);
            font-family: var(--mono);
            font-size: 12px;
            letter-spacing: 0.1em;
        }

        .dash-view {
            flex: 1;
            display: none;
            flex-direction: column;
            min-height: 0;
        }

        /* ── MESSAGE AREA ─────────────────────────────────────── */
        .message-area {
            flex: 1;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 24px 20px 16px;
            min-height: 0;
            background: var(--bg2);
            position: relative;
            overflow: hidden;
        }

        /* grid bg */
        .message-area::before {
            content: '';
            position: absolute;
            inset: 0;
            background-image:
                linear-gradient(var(--b1) 1px, transparent 1px),
                linear-gradient(90deg, var(--b1) 1px, transparent 1px);
            background-size: 36px 36px;
            opacity: 0.5;
            pointer-events: none;
        }

        /* scan line */
        .message-area::after {
            content: '';
            position: absolute;
            left: 0; right: 0;
            height: 2px;
            background: linear-gradient(90deg, transparent, rgba(79,158,255,0.15), transparent);
            animation: scan 4s linear infinite;
            pointer-events: none;
        }
        @keyframes scan {
            from { top: -2px; }
            to   { top: 100%; }
        }

        .msg-label {
            font-family: var(--mono);
            font-size: 9px;
            text-transform: uppercase;
            letter-spacing: 0.25em;
            color: var(--t4);
            margin-bottom: 20px;
            position: relative;
            z-index: 1;
        }

        .msg-text {
            font-family: var(--mono);
            font-size: clamp(14px, 2.8vw, 28px);
            font-weight: 500;
            color: var(--t1);
            text-align: center;
            max-width: 640px;
            line-height: 1.5;
            word-break: break-word;
            position: relative;
            z-index: 1;
            transition: color 0.3s;
        }
        .msg-text.waiting {
            color: var(--t4);
            font-size: 13px;
            font-weight: 300;
        }
        .msg-text.pop { animation: msgPop 0.45s cubic-bezier(0.16,1,0.3,1) forwards; }
        @keyframes msgPop {
            from { opacity: 0; transform: translateY(12px); filter: blur(4px); }
            to   { opacity: 1; transform: translateY(0);   filter: blur(0); }
        }

        .msg-meta {
            margin-top: 16px;
            display: flex;
            align-items: center;
            gap: 6px;
            opacity: 0;
            transition: opacity 0.4s;
            position: relative;
            z-index: 1;
        }
        .msg-meta.visible { opacity: 1; }

        .meta-chip {
            display: flex;
            align-items: center;
            gap: 6px;
            background: var(--bg3);
            border: 1px solid var(--b1);
            border-radius: 3px;
            padding: 3px 10px;
        }
        .meta-key {
            font-family: var(--mono);
            font-size: 8px;
            text-transform: uppercase;
            letter-spacing: 0.15em;
            color: var(--t4);
        }
        .meta-val {
            font-family: var(--mono);
            font-size: 11px;
            color: var(--t2);
        }
        .meta-sep { color: var(--t5); font-size: 9px; }

        /* ── LOG PANEL ────────────────────────────────────────── */
        .log-panel {
            height: 130px;
            min-height: 130px;
            flex-shrink: 0;
            border-top: 1px solid var(--b1);
            background: var(--bg);
            overflow-y: auto;
            padding: 6px 14px 8px;
        }

        .log-cols {
            display: flex;
            gap: 8px;
            font-family: var(--mono);
            font-size: 8px;
            text-transform: uppercase;
            letter-spacing: 0.15em;
            color: var(--t4);
            margin-bottom: 4px;
            padding-bottom: 4px;
            border-bottom: 1px solid var(--b1);
            position: sticky;
            top: 0;
            background: var(--bg);
            z-index: 2;
        }
        .lc      { flex: 1; color: var(--t4); }
        .lc-time { max-width: 68px !important; }
        .lc-ship { max-width: 80px !important; }
        .lc-plr  { max-width: 95px !important; }
        .lc-msg  { flex: 3 !important; }
        .lc-ping { max-width: 48px !important; text-align: right; }

        .log-row {
            display: flex;
            gap: 8px;
            font-family: var(--mono);
            font-size: 10px;
            padding: 2px 0;
            border-bottom: 1px solid #0c0c10;
            color: var(--t3);
            transition: color 0.2s;
        }
        .log-row:hover { color: var(--t2); }
        .log-row.fresh { color: var(--t2); animation: logSlide 0.2s ease; }
        .log-row > span {
            flex: 1;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
        }
        @keyframes logSlide {
            from { opacity: 0; transform: translateX(-4px); }
            to   { opacity: 1; transform: translateX(0); }
        }

        /* ── BOTTOM BAR ───────────────────────────────────────── */
        .bottombar {
            flex-shrink: 0;
            height: 64px;
            min-height: 64px;
            background: var(--bg3);
            border-top: 1px solid var(--b1);
            display: flex;
            align-items: stretch;
            overflow-x: auto;
            padding: 0 2px;
        }
        .bottombar::-webkit-scrollbar { display: none; }

        .bstat {
            display: flex;
            flex-direction: column;
            justify-content: center;
            gap: 3px;
            padding: 0 18px;
            border-right: 1px solid var(--b1);
            flex-shrink: 0;
        }
        .bstat:last-child { border-right: none; }

        .bs-label {
            font-family: var(--mono);
            font-size: 7px;
            text-transform: uppercase;
            letter-spacing: 0.18em;
            color: var(--t4);
        }
        .bs-val {
            font-family: var(--mono);
            font-size: 14px;
            font-weight: 500;
            color: var(--t3);
            transition: color 0.3s;
            white-space: nowrap;
        }
        .bs-val.good { color: var(--green); }
        .bs-val.ok   { color: var(--orange); }
        .bs-val.bad  { color: var(--red); }
        .bs-val.small { font-size: 10px; }
    </style>
</head>
<body>

<!-- TOPBAR -->
<div class="topbar">
    <div id="ship-tabs" style="display:flex;gap:7px;align-items:center;"></div>
    <span id="no-ships-msg">NO SHIPS ONLINE</span>
    <div class="topbar-right">
        <div class="status-pill">
            <div class="status-dot offline" id="status-dot"></div>
            <span id="status-text">DISCONNECTED</span>
        </div>
    </div>
</div>

<!-- MAIN -->
<div class="main">
    <div class="empty-view" id="empty-view">WAITING FOR SHIP TO COME ONLINE</div>

    <div class="dash-view" id="dash-view">
        <div class="message-area">
            <div class="msg-label">Last Transmission</div>
            <div class="msg-text waiting" id="msg-text">Awaiting transmission...</div>
            <div class="msg-meta" id="msg-meta">
                <div class="meta-chip">
                    <span class="meta-key">From</span>
                    <span class="meta-val" id="msg-sender">—</span>
                </div>
                <span class="meta-sep">·</span>
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
                <div style="color:var(--t4);font-family:var(--mono);font-size:10px;padding:6px 0;letter-spacing:0.05em;">No transmissions yet</div>
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
        <span class="bs-val small" id="bs-recv">—</span>
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

    let selectedShip = null;
    let ships        = [];
    let logEntries   = [];
    let lastMsgKey   = null;
    let lastDataTime = 0;
    let totalMsgs    = 0;
    let workerPing   = null;
    let pagesPing    = null;

    function pingClass(ms) {
        if (ms == null) return '';
        if (ms < 150)   return 'good';
        if (ms < 400)   return 'ok';
        return 'bad';
    }
    function fms(ms) { return ms != null ? ms + 'ms' : '—'; }

    async function fetchShips() {
        const t = Date.now();
        try {
            const res = await fetch(WORKER);
            workerPing = Date.now() - t;
            ships = res.ok ? await res.json() : [];
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

    function setOnline(state) {
        const dot  = document.getElementById('status-dot');
        const text = document.getElementById('status-text');
        if (state) {
            dot.className    = 'status-dot online';
            text.textContent = (selectedShip || '?') + ' · LIVE';
            text.style.color = 'var(--green)';
        } else {
            dot.className    = 'status-dot offline';
            text.textContent = selectedShip ? selectedShip + ' · NO SIGNAL' : 'DISCONNECTED';
            text.style.color = 'var(--t3)';
        }
    }

    function updateDisplay(values, updatedAt) {
        const text   = values.Text   || values.text   || null;
        const player = values.Player || values.player || null;
        const time   = updatedAt ? new Date(updatedAt).toLocaleTimeString() : new Date().toLocaleTimeString();

        const msgEl  = document.getElementById('msg-text');
        const metaEl = document.getElementById('msg-meta');

        if (text) {
            msgEl.classList.remove('waiting', 'pop');
            void msgEl.offsetWidth;
            msgEl.classList.remove('waiting');
            msgEl.classList.add('pop');
            msgEl.textContent = text;
        }

        document.getElementById('msg-sender').textContent = player || '—';
        document.getElementById('msg-time').textContent   = time;
        metaEl.classList.add('visible');

        totalMsgs++;
        logEntries.unshift({
            time,
            ship:   selectedShip || '—',
            player: player || '—',
            msg:    text || JSON.stringify(values),
            ping:   pagesPing
        });
        if (logEntries.length > 120) logEntries.pop();
        renderLog();
        updateBottomBar();
    }

    function renderLog() {
        const el = document.getElementById('log-entries');
        if (logEntries.length === 0) {
            el.innerHTML = '<div style="color:var(--t4);font-family:var(--mono);font-size:10px;padding:6px 0">No transmissions yet</div>';
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

    async function deleteShip(name) {
        try {
            await fetch(WORKER + "?name=" + encodeURIComponent(name), { method: "DELETE" });
            if (selectedShip === name) { selectedShip = null; showEmpty(); }
            await fetchShips();
        } catch (e) { console.error("Delete error:", e); }
    }

    fetchShips();
    setInterval(fetchShips,  3000);
    setInterval(fetchValues, 1000);
</script>
</body>
</html>

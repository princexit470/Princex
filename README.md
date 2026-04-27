
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>XIT - Hybrid Messenger & Gaming</title>
    <style>
        :root {
            --xit-black: #000000; --wa-green: #25D366; --wa-bg: #e5ddd5;
            --modal-bg: rgba(0,0,0,0.85); --glass: rgba(255, 255, 255, 0.1);
        }
        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        body, html { margin: 0; padding: 0; height: 100%; width: 100%; background: #fff; overflow: hidden; }

        /* Animations */
        @keyframes slideIn { from { transform: translateX(100%); } to { transform: translateX(0); } }
        @keyframes pop { from { transform: scale(0.8); opacity: 0; } to { transform: scale(1); opacity: 1; } }

        /* Global UI */
        header { background: var(--xit-black); color: white; z-index: 1000; position: relative; }
        .top-bar { display: flex; align-items: center; justify-content: space-between; padding: 15px; font-size: 20px; font-weight: bold; }
        .tabs { display: flex; background: #000; border-bottom: 1px solid #333; }
        .tab { flex: 1; padding: 12px; text-align: center; color: #888; cursor: pointer; transition: 0.3s; font-weight: 600; }
        .tab.active { color: white; border-bottom: 3px solid white; }

        .view-container { display: flex; width: 300%; height: calc(100vh - 105px); transition: transform 0.4s cubic-bezier(0.1, 0.7, 0.1, 1); }
        .view { width: 33.333%; height: 100%; overflow-y: auto; background: #fff; }

        /* Sidebar */
        #sidebar { position: fixed; left: -285px; top: 0; width: 280px; height: 100%; background: white; z-index: 3000; transition: 0.3s; box-shadow: 5px 0 15px rgba(0,0,0,0.2); }
        #sidebar.open { left: 0; }
        .overlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.5); z-index: 2500; }

        /* Game List Styling */
        .game-card { display: flex; align-items: center; padding: 15px; border-bottom: 1px solid #eee; margin: 10px; background: #f9f9f9; border-radius: 12px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
        .game-info { flex: 1; margin-left: 15px; }
        .game-btn { background: var(--wa-green); color: white; border: none; padding: 10px 20px; border-radius: 20px; font-weight: bold; cursor: pointer; transition: 0.2s; }
        .game-btn:active { transform: scale(0.9); }

        /* Game Screen Fullscreen */
        #game-screen { position: fixed; inset: 0; background: #fff; z-index: 5000; display: none; flex-direction: column; animation: slideIn 0.3s ease-out; }
        .game-header { background: #000; color: #fff; padding: 15px; display: flex; align-items: center; justify-content: space-between; }
        
        .voice-nodes { display: flex; gap: 10px; padding: 10px; background: #f0f0f0; overflow-x: auto; border-bottom: 1px solid #ddd; }
        .node { display: flex; flex-direction: column; align-items: center; min-width: 70px; }
        .avatar { width: 45px; height: 45px; border-radius: 50%; background: #444; color: white; display: flex; align-items: center; justify-content: center; font-weight: bold; border: 2px solid var(--wa-green); position: relative; }
        .node.audience .avatar { border-color: #ff9800; }
        .mic-off-icon { position: absolute; top: -2px; right: -2px; background: red; color: white; border-radius: 50%; font-size: 10px; width: 16px; height: 16px; display: flex; align-items: center; justify-content: center; }

        #game-stage { flex: 1; background: var(--wa-bg); display: flex; align-items: center; justify-content: center; position: relative; }
        
        /* Tic Tac Toe Board */
        .ttt-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; width: 300px; height: 300px; padding: 10px; background: rgba(0,0,0,0.05); border-radius: 15px; }
        .cell { background: white; border-radius: 10px; display: flex; align-items: center; justify-content: center; font-size: 45px; font-weight: bold; box-shadow: 0 4px 6px rgba(0,0,0,0.1); cursor: pointer; transition: 0.1s; }
        .cell:active { background: #eee; }

        .game-footer { padding: 15px; display: flex; gap: 10px; border-top: 1px solid #ddd; }
        .f-btn { flex: 1; padding: 15px; border: none; border-radius: 10px; font-weight: bold; color: white; font-size: 16px; }

        #login-screen { position: fixed; inset: 0; background: #000; z-index: 9999; display: flex; flex-direction: column; align-items: center; justify-content: center; color: white; }
    </style>
</head>
<body>

    <div id="login-screen">
        <h1 style="letter-spacing: 5px;">XIT</h1>
        <input type="text" id="username" placeholder="Username" style="padding: 15px; border-radius: 10px; width: 80%; border: none; margin-bottom: 20px;">
        <button onclick="login()" style="padding: 15px 40px; border-radius: 30px; border: none; background: white; font-weight: bold;">START</button>
    </div>

    <div id="sidebar">
        <div style="padding: 50px 20px 20px; background: #000; color: #fff;">
            <div id="my-initial" style="width: 60px; height: 60px; background: var(--wa-green); border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 24px; font-weight: bold; margin-bottom: 10px;">?</div>
            <b id="my-name-display">@username</b>
        </div>
        <div style="padding: 20px; border-bottom: 1px solid #eee;" onclick="switchTab(1); toggleSidebar(false);">🎮 Online Games</div>
        <div style="padding: 20px; border-bottom: 1px solid #eee;" onclick="localStorage.clear(); location.reload();">🚪 Logout</div>
    </div>
    <div class="overlay" id="overlay" onclick="toggleSidebar(false)"></div>

    <div id="game-screen">
        <div class="game-header">
            <div onclick="exitGame()" style="cursor: pointer; font-size: 20px;">✕ Exit</div>
            <div id="game-title">Waiting...</div>
            <div onclick="copyGameLink()" style="background: white; color: black; padding: 5px 12px; border-radius: 15px; font-size: 12px; font-weight: bold; cursor: pointer;">🔗 Link</div>
        </div>
        <div class="voice-nodes" id="voice-nodes"></div>
        <div id="game-stage">
            <div id="match-status" style="text-align: center; color: #667781;">
                <div class="spinner" style="border: 4px solid #f3f3f3; border-top: 4px solid #000; border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; margin: 0 auto 15px;"></div>
                Searching for online players...
            </div>
        </div>
        <div class="game-footer">
            <button class="f-btn" id="mic-btn" style="background: #000;" onclick="toggleMic()">🎤 Mic: ON</button>
            <button class="f-btn" style="background: #ff4d4d;" onclick="exitGame()">⛔ Quit</button>
        </div>
    </div>

    <header>
        <div class="top-bar"><span onclick="toggleSidebar(true)">☰ XIT</span> <span>X</span></div>
        <div class="tabs">
            <div class="tab active" id="tab0" onclick="switchTab(0)">CHATS</div>
            <div class="tab" id="tab1" onclick="switchTab(1)">GAMES</div>
            <div class="tab" id="tab2" onclick="switchTab(2)">REQS</div>
        </div>
    </header>

    <div class="view-container" id="view-container">
        <div class="view"></div>
        <div class="view" id="game-list">
            <h3 style="padding: 15px;">Live Multiplayer</h3>
            </div>
        <div class="view"></div>
    </div>

    <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>

    <script>
        const firebaseConfig = {
            apiKey: "AIzaSyB6I0uj92aNft_TLwtYXhLNBi136LTYr8g",
            authDomain: "prince-a6721.firebaseapp.com",
            databaseURL: "https://prince-a6721-default-rtdb.firebaseio.com",
            projectId: "prince-a6721",
            storageBucket: "prince-a6721.firebasestorage.app",
            appId: "1:320131851435:web:fe98ec2fa7641aa41583ee"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        let myUser = localStorage.getItem('xit_user');
        let currentRoom = null;
        let isMicEnabled = true;
        let userRole = 'player';

        const gameConfigs = [
            { id: 'ttt', name: 'Tic Tac Toe', icon: '❌', max: 2 },
            { id: 'quiz', name: 'Brain Battle', icon: '💡', max: 2 },
            { id: 'ludo', name: 'Ludo Express', icon: '🎲', max: 4 }
        ];

        function login() {
            let u = document.getElementById('username').value.trim().toLowerCase();
            if(u) { localStorage.setItem('xit_user', u); location.reload(); }
        }

        if(myUser) {
            document.getElementById('login-screen').style.display = 'none';
            document.getElementById('my-name-display').innerText = "@" + myUser;
            document.getElementById('my-initial').innerText = myUser[0].toUpperCase();
            initApp();
        }

        function initApp() {
            const list = document.getElementById('game-list');
            gameConfigs.forEach(g => {
                list.innerHTML += `
                    <div class="game-card">
                        <span style="font-size: 30px;">${g.icon}</span>
                        <div class="game-info"><b>${g.name}</b><br><small>Online Match</small></div>
                        <button class="game-btn" onclick="matchmake('${g.id}', ${g.max})">Auto Join</button>
                    </div>`;
            });

            // Check if URL has room link
            const urlParams = new URLSearchParams(window.location.search);
            if(urlParams.has('room')) {
                const room = urlParams.get('room');
                const gid = urlParams.get('gid');
                const m = urlParams.get('m');
                promptJoin(gid, room, m);
            }
        }

        function promptJoin(gid, room, m) {
            if(confirm("Join Room " + room + " as Audience? (OK for Audience, Cancel for Player)")) {
                joinRoom(gid, room, m, 'audience');
            } else {
                joinRoom(gid, room, m, 'player');
            }
        }

        async function matchmake(gid, max) {
            document.getElementById('game-screen').style.display = 'flex';
            const matchRef = db.ref(`matchmaking/${gid}`);
            
            // Transaction for Atomic Matchmaking
            matchRef.transaction((data) => {
                if(!data) return { roomID: "R" + Date.now(), count: 1 };
                if(data.count < max) {
                    return { roomID: data.roomID, count: data.count + 1 };
                } else {
                    return { roomID: "R" + Date.now(), count: 1 };
                }
            }, (error, committed, snapshot) => {
                if(committed) {
                    const room = snapshot.val().roomID;
                    joinRoom(gid, room, max, 'player');
                }
            });
        }

        function joinRoom(gid, room, max, role) {
            currentRoom = room;
            userRole = role;
            document.getElementById('game-screen').style.display = 'flex';
            
            const sessionRef = db.ref(`game_sessions/${room}`);
            const path = role === 'player' ? `players/${myUser}` : `audience/${myUser}`;
            
            sessionRef.child(path).set({ mic: true, ts: Date.now() });
            sessionRef.child(path).onDisconnect().remove();

            sessionRef.on('value', snap => {
                const data = snap.val() || {};
                const players = data.players || {};
                const audience = data.audience || {};
                updateVoiceUI(players, audience);

                const pCount = Object.keys(players).length;
                if(pCount >= max) {
                    document.getElementById('game-title').innerText = gid.toUpperCase();
                    if(gid === 'ttt') runTTT(room, players);
                    else document.getElementById('game-stage').innerHTML = "<h2>Game Started!</h2>";
                }
            });
        }

        function updateVoiceUI(p, a) {
            const container = document.getElementById('voice-nodes');
            container.innerHTML = "";
            [...Object.keys(p).map(n=>({n, r:'player'})), ...Object.keys(a).map(n=>({n, r:'audience'}))].forEach(u => {
                container.innerHTML += `
                    <div class="node ${u.r}">
                        <div class="avatar">${u.n[0].toUpperCase()}</div>
                        <span style="font-size: 10px;">${u.n === myUser ? 'You' : u.n}</span>
                    </div>`;
            });
        }

        // --- Tic Tac Toe Fixed Logic ---
        function runTTT(room, players) {
            const stage = document.getElementById('game-stage');
            stage.innerHTML = `<div class="ttt-grid" id="grid"></div>`;
            const grid = document.getElementById('grid');
            for(let i=0; i<9; i++) grid.innerHTML += `<div class="cell" id="c${i}" onclick="tttClick(${i})"></div>`;

            db.ref(`game_sessions/${room}/state`).on('value', snap => {
                const state = snap.val() || Array(9).fill("");
                state.forEach((val, i) => { document.getElementById(`c${i}`).innerText = val; });
            });
        }

        async function tttClick(i) {
            if(userRole !== 'player') return;
            const ref = db.ref(`game_sessions/${currentRoom}`);
            const snap = await ref.get();
            const players = Object.keys(snap.val().players).sort();
            const myMark = players[0] === myUser ? "X" : "O";
            
            let state = snap.val().state || Array(9).fill("");
            const xC = state.filter(s=>s==="X").length;
            const oC = state.filter(s=>s==="O").length;
            const turn = xC <= oC ? "X" : "O";

            if(turn === myMark && !state[i]) {
                state[i] = myMark;
                ref.child('state').set(state);
            }
        }

        function toggleMic() {
            isMicEnabled = !isMicEnabled;
            document.getElementById('mic-btn').innerText = isMicEnabled ? "🎤 Mic: ON" : "🔇 Mic: OFF";
            const p = userRole === 'player' ? 'players' : 'audience';
            db.ref(`game_sessions/${currentRoom}/${p}/${myUser}/mic`).set(isMicEnabled);
        }

        function copyGameLink() {
            const url = window.location.origin + window.location.pathname + `?room=${currentRoom}&gid=ttt&m=2`;
            navigator.clipboard.writeText(url).then(() => alert("Link Copied! Share with friends."));
        }

        function exitGame() {
            if(currentRoom) {
                const p = userRole === 'player' ? 'players' : 'audience';
                db.ref(`game_sessions/${currentRoom}/${p}/${myUser}`).remove();
            }
            document.getElementById('game-screen').style.display = 'none';
            window.history.replaceState({}, '', window.location.pathname);
            currentRoom = null;
        }

        function switchTab(i) {
            document.getElementById('view-container').style.transform = `translateX(-${i * 33.333}%)`;
            document.querySelectorAll('.tab').forEach((t, idx) => t.classList.toggle('active', idx === i));
        }

        function toggleSidebar(o) {
            document.getElementById('sidebar').classList.toggle('open', o);
            document.getElementById('overlay').style.display = o ? 'block' : 'none';
        }
    </script>
</body>
</html>

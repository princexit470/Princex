<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>XIT - Pro Gaming</title>
    <style>
        :root { --xit-black: #000000; --wa-green: #25D366; --bg: #f4f7f6; }
        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; font-family: 'Segoe UI', sans-serif; }
        body, html { margin: 0; padding: 0; height: 100%; width: 100%; background: var(--bg); overflow: hidden; }

        /* UI Elements */
        header { background: var(--xit-black); color: white; position: relative; z-index: 10; }
        .top-bar { padding: 15px; font-weight: bold; font-size: 20px; display: flex; justify-content: space-between; }
        .tabs { display: flex; background: #111; }
        .tab { flex: 1; padding: 12px; text-align: center; color: #888; cursor: pointer; font-weight: bold; border-bottom: 3px solid transparent; }
        .tab.active { color: white; border-bottom-color: white; }

        .view-container { display: flex; width: 300%; height: calc(100vh - 105px); transition: 0.3s; }
        .view { width: 33.333%; height: 100%; overflow-y: auto; background: white; }

        /* Game Screen */
        #game-screen { position: fixed; top: 0; left: 100%; width: 100%; height: 100%; background: #fff; z-index: 9999; transition: 0.3s; display: flex; flex-direction: column; }
        #game-screen.open { left: 0; }
        .game-header { background: #000; color: #fff; padding: 15px; display: flex; justify-content: space-between; align-items: center; }
        
        .voice-section { background: #eee; padding: 10px; display: flex; gap: 10px; overflow-x: auto; border-bottom: 1px solid #ccc; }
        .user-card { display: flex; flex-direction: column; align-items: center; min-width: 75px; position: relative; }
        .avatar { width: 45px; height: 45px; border-radius: 50%; background: #333; color: white; display: flex; align-items: center; justify-content: center; font-weight: bold; border: 2px solid var(--wa-green); }
        .audience .avatar { border-color: #ff9800; }
        .mute-label { position: absolute; bottom: 15px; right: 10px; background: red; border-radius: 50%; width: 15px; height: 15px; display: none; }

        #game-canvas { flex: 1; display: flex; align-items: center; justify-content: center; background: #e5ddd5; position: relative; }
        
        /* Tic Tac Toe Grid */
        .ttt-grid { display: grid; grid-template-columns: repeat(3, 90px); grid-template-rows: repeat(3, 90px); gap: 10px; }
        .cell { background: white; border-radius: 10px; display: flex; align-items: center; justify-content: center; font-size: 35px; font-weight: bold; box-shadow: 0 4px 8px rgba(0,0,0,0.1); cursor: pointer; }

        .controls { padding: 15px; display: flex; gap: 10px; background: #fff; border-top: 1px solid #ddd; }
        .btn { flex: 1; padding: 12px; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; }
        .btn-mic { background: #000; color: white; }
        .btn-quit { background: #ff4d4d; color: white; }

        /* Game List */
        .game-item { display: flex; align-items: center; justify-content: space-between; padding: 15px; border-bottom: 1px solid #eee; }
        .join-btn { background: var(--wa-green); color: white; border: none; padding: 8px 15px; border-radius: 5px; font-weight: bold; }

        #login-screen { position: fixed; inset: 0; background: #000; z-index: 10001; display: flex; flex-direction: column; align-items: center; justify-content: center; color: white; }
        input { padding: 12px; border-radius: 8px; border: none; width: 80%; max-width: 300px; margin-bottom: 15px; }
    </style>
</head>
<body>

    <div id="login-screen">
        <h1>XIT GAMING</h1>
        <input type="text" id="username" placeholder="Enter Username">
        <button class="btn" style="background:white; width:200px;" onclick="doLogin()">Get Started</button>
    </div>

    <div id="game-screen">
        <div class="game-header">
            <span onclick="quitGame()" style="cursor:pointer; font-size:20px;">✕ Exit</span>
            <b id="game-name">Loading...</b>
            <span id="player-count">0/0</span>
        </div>
        
        <div class="voice-section" id="voice-section"></div>

        <div id="game-canvas">
            <div id="game-msg">Finding Players...</div>
        </div>

        <div class="controls">
            <button class="btn btn-mic" id="mic-btn" onclick="toggleMyMic()">🎤 Mic: ON</button>
            <button class="btn btn-quit" onclick="quitGame()">⛔ Quit</button>
        </div>
    </div>

    <header>
        <div class="top-bar"><span>XIT</span></div>
        <div class="tabs">
            <div class="tab active" id="tab0" onclick="switchTab(0)">CHATS</div>
            <div class="tab" id="tab1" onclick="switchTab(1)">GAMES</div>
            <div class="tab" id="tab2" onclick="switchTab(2)">REQS</div>
        </div>
    </header>

    <div class="view-container" id="view-container">
        <div class="view"></div> <div class="view" id="game-list"></div> <div class="view"></div> </div>

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
        let myMicStatus = true;
        let myRole = 'player';

        const games = [
            { id: 'ttt', name: 'Tic Tac Toe', max: 2, icon: '❌' },
            { id: 'quiz', name: 'Brain Battle', max: 2, icon: '💡' },
            { id: 'ludo', name: 'Ludo Express', max: 4, icon: '🎲' }
        ];

        function doLogin() {
            let u = document.getElementById('username').value.trim();
            if(u) { localStorage.setItem('xit_user', u); location.reload(); }
        }

        if(myUser) {
            document.getElementById('login-screen').style.display = 'none';
            initApp();
        }

        function initApp() {
            const list = document.getElementById('game-list');
            list.innerHTML = '<h3 style="padding:15px;">Live Multiplayer Games</h3>';
            games.forEach(g => {
                list.innerHTML += `
                    <div class="game-item">
                        <span>${g.icon} <b>${g.name}</b></span>
                        <button class="join-btn" onclick="startMatchmaking('${g.id}', ${g.max})">Auto Join</button>
                    </div>`;
            });
        }

        async function startMatchmaking(gameId, maxPlayers) {
            document.getElementById('game-screen').classList.add('open');
            document.getElementById('game-name').innerText = "Matchmaking...";
            
            const matchRef = db.ref(`matchmaking/${gameId}`);
            const snap = await matchRef.once('value');
            let roomID = null;

            if(snap.exists()) {
                const rooms = snap.val();
                for(let id in rooms) {
                    if(rooms[id].count < maxPlayers) { roomID = id; break; }
                }
            }

            if(!roomID) {
                roomID = "ROOM_" + Math.random().toString(36).substr(2, 6).toUpperCase();
                await matchRef.child(roomID).set({ count: 1 });
                myRole = 'player';
            } else {
                let currentCount = snap.val()[roomID].count;
                if(currentCount < maxPlayers) {
                    await matchRef.child(roomID).update({ count: currentCount + 1 });
                    myRole = 'player';
                } else {
                    // Agar game full hai toh check audience
                    const audSnap = await db.ref(`game_sessions/${roomID}/audience`).once('value');
                    if(audSnap.size < 5) {
                        myRole = 'audience';
                    } else {
                        alert("Room is full!"); return;
                    }
                }
            }

            currentRoom = roomID;
            enterRoom(gameId, roomID, maxPlayers);
        }

        function enterRoom(gameId, roomID, maxPlayers) {
            const sessionRef = db.ref(`game_sessions/${roomID}`);
            const myPath = myRole === 'player' ? `players/${myUser}` : `audience/${myUser}`;
            
            sessionRef.child(myPath).set({ mic: true, ts: Date.now() });
            sessionRef.child(myPath).onDisconnect().remove();

            sessionRef.on('value', snap => {
                const data = snap.val() || {};
                const players = data.players || {};
                const audience = data.audience || {};
                
                updateVoiceUI(players, audience);
                
                const pCount = Object.keys(players).length;
                document.getElementById('player-count').innerText = `${pCount}/${maxPlayers}`;

                if(pCount >= maxPlayers) {
                    document.getElementById('game-name').innerText = games.find(g=>g.id===gameId).name;
                    if(gameId === 'ttt') initTTT(roomID, players);
                }
            });
        }

        function updateVoiceUI(players, audience) {
            const bar = document.getElementById('voice-section');
            bar.innerHTML = "";
            
            // Render Players
            Object.keys(players).forEach(name => {
                bar.innerHTML += createAvatarNode(name, 'player', players[name].mic);
            });
            
            // Render Audience
            Object.keys(audience).forEach(name => {
                bar.innerHTML += createAvatarNode(name, 'audience', audience[name].mic);
            });
        }

        function createAvatarNode(name, role, mic) {
            const isMuted = localStorage.getItem('mute_'+name) === 'true';
            return `
                <div class="user-card ${role}">
                    <div class="avatar" onclick="toggleOpponentSound('${name}')">${name[0].toUpperCase()}</div>
                    <span style="font-size:10px;">${name === myUser ? 'You' : name}</span>
                    <div class="mute-label" style="display:${mic ? 'none' : 'block'}"></div>
                    ${name !== myUser ? `<button onclick="toggleOpponentSound('${name}')" style="font-size:9px; border:none;">${isMuted ? '🔇' : '🔊'}</button>` : ''}
                </div>`;
        }

        // --- Game Logic: Tic Tac Toe ---
        function initTTT(roomID, players) {
            const canvas = document.getElementById('game-canvas');
            canvas.innerHTML = '<div class="ttt-grid" id="grid"></div>';
            const grid = document.getElementById('grid');
            for(let i=0; i<9; i++) grid.innerHTML += `<div class="cell" id="c${i}" onclick="playTTT(${i})"></div>`;

            db.ref(`game_sessions/${roomID}/state`).on('value', snap => {
                const state = snap.val() || Array(9).fill("");
                state.forEach((val, i) => {
                    document.getElementById(`c${i}`).innerText = val;
                });
            });
        }

        async function playTTT(idx) {
            if(myRole !== 'player') return;
            const ref = db.ref(`game_sessions/${currentRoom}`);
            const snap = await ref.once('value');
            const players = Object.keys(snap.val().players).sort();
            const mySymbol = players[0] === myUser ? "X" : "O";
            
            let state = snap.val().state || Array(9).fill("");
            
            const xCount = state.filter(s => s === "X").length;
            const oCount = state.filter(s => s === "O").length;
            const turn = xCount <= oCount ? "X" : "O";

            if(turn === mySymbol && !state[idx]) {
                state[idx] = mySymbol;
                ref.child('state').set(state);
            }
        }

        // --- Controls ---
        function toggleMyMic() {
            myMicStatus = !myMicStatus;
            document.getElementById('mic-btn').innerText = myMicStatus ? "🎤 Mic: ON" : "🔇 Mic: OFF";
            const path = myRole === 'player' ? 'players' : 'audience';
            db.ref(`game_sessions/${currentRoom}/${path}/${myUser}/mic`).set(myMicStatus);
        }

        function toggleOpponentSound(name) {
            const current = localStorage.getItem('mute_'+name) === 'true';
            localStorage.setItem('mute_'+name, !current);
            alert(name + (current ? " Unmuted" : " Muted"));
            location.reload(); // Simple way to apply mute logic
        }

        function quitGame() {
            if(currentRoom) {
                const path = myRole === 'player' ? 'players' : 'audience';
                db.ref(`game_sessions/${currentRoom}/${path}/${myUser}`).remove();
            }
            document.getElementById('game-screen').classList.remove('open');
            currentRoom = null;
        }

        function switchTab(i) {
            document.getElementById('view-container').style.transform = `translateX(-${i * 33.333}%)`;
            document.querySelectorAll('.tab').forEach((t, idx) => t.classList.toggle('active', idx === i));
        }
    </script>
</body>
</html>


<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Number Bluff</title>
  <style>
    :root {
      --bg: #10131c;
      --panel: #1b2030;
      --panel2: #252c40;
      --text: #f5f6fb;
      --muted: #aeb7cc;
      --accent: #a78bfa;
      --green: #6ee7b7;
      --red: #fda4af;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0; background: var(--bg); color: var(--text);
      font: 16px/1.5 system-ui, -apple-system, Segoe UI, sans-serif;
    }
    main { max-width: 850px; margin: auto; padding: 24px 16px 60px; }
    h1 { font-size: clamp(2.3rem, 7vw, 4rem); margin: 0; letter-spacing: -2px; }
    h1 span { color: var(--accent); }
    h2 { margin-top: 0; }
    .subtitle { color: var(--muted); margin: 0 0 24px; }
    .panel {
      background: var(--panel); border: 1px solid #30384c;
      border-radius: 18px; padding: 20px; margin: 16px 0;
    }
    .row { display: flex; gap: 10px; flex-wrap: wrap; }
    .row > * { flex: 1; min-width: 130px; }
    input, select, button {
      width: 100%; border-radius: 10px; padding: 12px 14px;
      font: inherit; border: 1px solid #3b455d;
    }
    input, select { background: #111622; color: var(--text); }
    button {
      background: var(--accent); color: #171326; font-weight: 750;
      border: 0; cursor: pointer;
    }
    button.secondary { background: var(--panel2); color: var(--text); border: 1px solid #414b63; }
    button.danger { background: #7f1d35; color: white; }
    button:disabled { opacity: .45; cursor: not-allowed; }
    .muted { color: var(--muted); }
    .hidden { display: none !important; }
    .code { font-size: 2rem; letter-spacing: 5px; font-weight: 850; color: var(--accent); }
    .number { font-size: 4rem; font-weight: 900; text-align: center; color: var(--green); }
    .prompt { font-size: 1.3rem; font-weight: 700; }
    .notice { padding: 12px 14px; border-radius: 10px; background: #292641; margin: 12px 0; }
    .error { color: var(--red); }
    .success { color: var(--green); }
    ul { padding-left: 20px; }
    li { margin: 5px 0; }
    .player, .scoreline {
      display: flex; justify-content: space-between; align-items: center;
      gap: 12px; padding: 9px 0; border-bottom: 1px solid #30384c;
    }
    .player:last-child, .scoreline:last-child { border-bottom: 0; }
    .tag { font-size: .8rem; padding: 3px 8px; border-radius: 20px; background: var(--panel2); color: var(--muted); }
    .small { font-size: .9rem; }
    footer { color: var(--muted); font-size: .85rem; margin-top: 30px; }
  </style>
</head>
<body>
<main>
  <h1>NUMBER <span>BLUFF</span></h1>
  <p class="subtitle">One secret number. One answer. Everyone else takes a guess.</p>

  <div id="setup" class="panel">
    <h2>Join the game</h2>
    <label for="playerName">Your name</label>
    <input id="playerName" maxlength="24" placeholder="Enter your name">
    <div style="height:12px"></div>
    <div class="row">
      <button id="createBtn">Create a lobby</button>
      <button id="showJoinBtn" class="secondary">Join a lobby</button>
    </div>
    <div id="joinArea" class="hidden" style="margin-top:14px">
      <label for="lobbyCode">Lobby code</label>
      <input id="lobbyCode" maxlength="4" placeholder="ABCD" autocapitalize="characters">
      <div style="height:10px"></div>
      <button id="joinBtn">Join game</button>
    </div>
    <p id="setupMessage" class="muted small"></p>
  </div>

  <div id="game" class="hidden">
    <div class="panel">
      <div class="muted small">LOBBY CODE</div>
      <div class="code" id="codeDisplay">----</div>
      <p id="roomStatus" class="muted">Waiting for players…</p>
      <div id="players"></div>
    </div>

    <div id="hostPanel" class="panel hidden">
      <h2>Host controls</h2>
      <label for="category">Prompt category</label>
      <select id="category">
        <option value="Minecraft">Minecraft</option>
        <option value="General Knowledge">General Knowledge</option>
        <option value="Custom">Custom prompt</option>
      </select>
      <div style="height:10px"></div>
      <input id="customPrompt" class="hidden" placeholder="Write your own prompt">
      <div class="row" style="margin-top:10px">
        <div>
          <label for="roundLimit">Number of rounds</label>
          <select id="roundLimit">
            <option>3</option><option selected>5</option><option>10</option><option>15</option><option>20</option>
          </select>
        </div>
        <div>
          <label for="targetSelect">Answer player</label>
          <select id="targetSelect"></select>
        </div>
      </div>
      <div style="height:14px"></div>
      <button id="startRoundBtn">Start round</button>
      <p class="muted small">The host and answer player see the secret number. Other players must guess it.</p>
      <div id="hostSecretArea" class="hidden notice">
        <div class="muted small">SECRET NUMBER</div>
        <div id="hostSecret" class="number"></div>
        <p class="small">Only share this number with the answer player if you want to.</p>
      </div>
    </div>

    <div id="roundPanel" class="panel hidden">
      <div class="muted small" id="roundLabel">ROUND 1</div>
      <div id="promptText" class="prompt"></div>
      <p id="roleText" class="muted"></p>

      <div id="answerArea" class="hidden">
        <div class="notice">
          <div class="muted small">YOUR SECRET NUMBER</div>
          <div id="playerSecret" class="number"></div>
        </div>
        <label for="answerInput">Your answer to the prompt</label>
        <input id="answerInput" maxlength="250" placeholder="Type your answer">
        <div style="height:10px"></div>
        <button id="submitAnswerBtn">Submit answer</button>
      </div>

      <div id="guessArea" class="hidden">
        <p>Guess the secret number from <b>1 to 10</b>.</p>
        <div class="row">
          <div>
            <label for="guessInput">Your guess</label>
            <select id="guessInput">
              <option value="">Choose 1–10</option>
              <option>1</option><option>2</option><option>3</option><option>4</option><option>5</option>
              <option>6</option><option>7</option><option>8</option><option>9</option><option>10</option>
            </select>
          </div>
          <div style="align-self:end">
            <button id="submitGuessBtn">Lock in guess</button>
          </div>
        </div>
      </div>

      <div id="waitingArea" class="notice hidden">Your answer or guess has been submitted. Waiting for the round to finish…</div>
      <div id="answerReveal" class="notice hidden"></div>
      <div id="results"></div>
      <div id="hostNextArea" class="hidden">
        <button id="nextRoundBtn">Next round</button>
      </div>
    </div>

    <div class="panel">
      <h2>Leaderboard</h2>
      <div id="leaderboard"></div>
      <p class="muted small">Scoring: exact guess = 4 points, off by 1 = 2 points, off by 2 = 1 point.</p>
      <button id="leaveBtn" class="secondary">Leave this lobby</button>
    </div>
  </div>

  <footer>
    Number Bluff · Online multiplayer prototype. Use a Firebase Realtime Database project to enable lobbies.
  </footer>
</main>

<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-database-compat.js"></script>
<script>
/*
  SETUP:
  1. Create a Firebase project.
  2. Enable Authentication > Sign-in method > Anonymous.
  3. Create a Realtime Database.
  4. Replace the config values below with your Firebase web app config.
  5. Add the Realtime Database rules provided alongside this file.
*/
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  databaseURL: "https://YOUR_DATABASE_NAME-default-rtdb.firebaseio.com",
  projectId: "YOUR_PROJECT_ID",
  appId: "YOUR_APP_ID"
};

const $ = id => document.getElementById(id);
const prompts = {
  "Minecraft": [
    "How many hearts does a player have by default?",
    "How many blocks high is a standard door?",
    "How many eyes can an End Portal frame contain?",
    "How many iron ingots are needed to craft an anvil?",
    "How many obsidian blocks are needed for a minimum Nether portal?",
    "How many inventory slots does a standard chest have?",
    "How many minutes are in one Minecraft day?",
    "How many wooden planks are needed to craft a crafting table?",
    "How many sticks are needed to craft a ladder?",
    "How many pieces of wheat are needed to craft bread?",
    "How many slots are in a player's hotbar?",
    "How many string are needed to craft a bow?",
    "How many diamonds are needed to craft a diamond chestplate?",
    "How many blocks make up a full beacon base in a 3-by-3 layer?",
    "How many snowballs can one snow block craft?",
    "How many gold ingots are needed to craft a golden apple?",
    "How many Ender Pearls are needed to craft an Eye of Ender?",
    "How many books are needed to craft a bookshelf?",
    "How many wool blocks are needed to craft a bed?",
    "How many gunpowder are needed to craft TNT?"
  ],
  "General Knowledge": [
    "How many sides does a hexagon have?",
    "How many days are in a leap year?",
    "How many players are on the field for one football team?",
    "How many colours are traditionally listed in a rainbow?",
    "How many continents are commonly taught in school?",
    "How many minutes are in an hour?",
    "How many keys are on a standard modern piano?",
    "How many planets are in our Solar System?",
    "How many letters are in the English alphabet?",
    "How many rings are on the Olympic symbol?",
    "How many sides does an octagon have?",
    "How many degrees are in a right angle?",
    "How many months have 31 days?",
    "How many cards are in a standard deck without jokers?",
    "How many strings does a standard guitar have?",
    "How many teeth does a typical adult human have?",
    "How many squares are on a chessboard?",
    "How many seconds are in a minute?",
    "How many players are on court for one basketball team?",
    "How many sides does a triangle have?"
  ]
};

let auth, db, uid, roomCode, roomRef, roomData;
let isHost = false;
let currentRoomListener = null;
let busy = false;

function message(text, error = false) {
  $("setupMessage").textContent = text;
  $("setupMessage").className = error ? "error small" : "muted small";
}
function cleanCode(value) {
  return value.toUpperCase().replace(/[^A-Z0-9]/g, "").slice(0, 4);
}
function randomCode() {
  const chars = "ABCDEFGHJKLMNPQRSTUVWXYZ23456789";
  return Array.from({length:4}, () => chars[Math.floor(Math.random()*chars.length)]).join("");
}
function randomPrompt(category) {
  if (category === "Custom") return $("customPrompt").value.trim();
  const list = prompts[category] || prompts["General Knowledge"];
  return list[Math.floor(Math.random() * list.length)];
}
function safeText(value) {
  return String(value ?? "").replace(/[&<>"']/g, c => ({
    "&":"&amp;", "<":"&lt;", ">":"&gt;", '"':"&quot;", "'":"&#39;"
  })[c]);
}
function playerName() {
  return $("playerName").value.trim().slice(0,24);
}
function getPlayers() {
  return Object.entries(roomData?.players || {}).map(([id,p]) => ({id, ...p}));
}
function isRoundActive() {
  return roomData?.state?.phase === "playing";
}
function amAnswerPlayer() {
  return roomData?.state?.targetUid === uid;
}
function mySecret() {
  return roomData?.secrets?.[uid]?.number;
}
function scores() {
  return getPlayers().sort((a,b) => (b.score||0)-(a.score||0));
}
function showGame() {
  $("setup").classList.add("hidden");
  $("game").classList.remove("hidden");
  $("codeDisplay").textContent = roomCode;
}
function showError(err) {
  console.error(err);
  message(err?.message || "Something went wrong. Check your Firebase setup and database rules.", true);
}
async function initFirebase() {
  if (firebaseConfig.apiKey === "YOUR_API_KEY") {
    throw new Error("First add your Firebase web app configuration in index.html.");
  }
  if (!firebase.apps.length) firebase.initializeApp(firebaseConfig);
  auth = firebase.auth();
  db = firebase.database();
  const credential = await auth.signInAnonymously();
  uid = credential.user.uid;
}
async function createLobby() {
  if (busy) return;
  busy = true;
  try {
    if (!playerName()) throw new Error("Enter your name first.");
    await initFirebase();
    let code, ref;
    for (let i=0; i<8; i++) {
      code = randomCode();
      ref = db.ref("rooms/" + code);
      const snap = await ref.get();
      if (!snap.exists()) break;
      ref = null;
    }
    if (!ref) throw new Error("Couldn't find an available lobby code. Try again.");
    roomCode = code;
    roomRef = ref;
    isHost = true;
    const name = playerName();
    await roomRef.child("hostUid").set(uid);
    await roomRef.child("state").set({
      phase:"lobby", round:0, roundLimit:5, targetUid:null,
      prompt:"", category:"Minecraft", answer:null, result:null
    });
    await roomRef.child("players/" + uid).set({name, score:0, joinedAt:Date.now()});
    showGame();
    listenRoom();
  } catch (e) { showError(e); }
  finally { busy = false; }
}
async function joinLobby() {
  if (busy) return;
  busy = true;
  try {
    if (!playerName()) throw new Error("Enter your name first.");
    const code = cleanCode($("lobbyCode").value);
    if (code.length !== 4) throw new Error("Enter the four-character lobby code.");
    await initFirebase();
    roomCode = code;
    roomRef = db.ref("rooms/" + roomCode);
    const snap = await roomRef.get();
    if (!snap.exists()) throw new Error("Lobby not found. Check the code.");
    const room = snap.val();
    if (room.state?.phase === "finished") throw new Error("This game has already finished.");
    if (room.state?.phase === "playing" && !room.players?.[uid]) {
      throw new Error("A round is already in progress. Join after the round ends.");
    }
    isHost = room.hostUid === uid;
    await roomRef.child("players/" + uid).set({
      name:playerName(), score:0, joinedAt:Date.now()
    });
    showGame();
    listenRoom();
  } catch (e) { showError(e); }
  finally { busy = false; }
}
function listenRoom() {
  if (currentRoomListener) roomRef?.off("value", currentRoomListener);
  currentRoomListener = snap => {
    if (!snap.exists()) {
      $("roomStatus").textContent = "This lobby no longer exists.";
      return;
    }
    roomData = snap.val();
    render();
  };
  roomRef.on("value", currentRoomListener);
}
function render() {
  const state = roomData.state || {};
  $("codeDisplay").textContent = roomCode;
  $("hostPanel").classList.toggle("hidden", !isHost);
  $("roomStatus").textContent =
    state.phase === "lobby" ? (isHost ? "You are the host. Choose a player and start." : "Waiting for the host to start the game…") :
    state.phase === "playing" ? "Round in progress" :
    state.phase === "results" ? "Round results" : "Game finished";

  const players = getPlayers();
  $("players").innerHTML = "<h3>Players ("+players.length+")</h3>" +
    (players.length ? players.map(p =>
      `<div class="player"><span>${safeText(p.name)} ${p.id===roomData.hostUid ? '<span class="tag">HOST</span>':''} ${p.id===uid ? '<span class="tag">YOU</span>':''}</span><span>${p.score||0} pts</span></div>`
    ).join("") : "<p class='muted'>No players yet.</p>");

  const select = $("targetSelect");
  const oldTarget = select.value;
  select.innerHTML = players.map(p => `<option value="${p.id}">${safeText(p.name)}</option>`).join("");
  if (players.some(p => p.id === oldTarget)) select.value = oldTarget;

  $("leaderboard").innerHTML = scores().map((p,i) =>
    `<div class="scoreline"><span>${i+1}. ${safeText(p.name)}</span><b>${p.score||0}</b></div>`
  ).join("") || "<p class='muted'>Scores will appear here.</p>";

  $("roundPanel").classList.toggle("hidden", !["playing","results","finished"].includes(state.phase));
  $("roundLabel").textContent = `ROUND ${state.round || 0} / ${state.roundLimit || 5}`;
  $("promptText").textContent = state.prompt || "";
  $("roleText").textContent = state.phase === "playing"
    ? (amAnswerPlayer() ? "You are answering the prompt. Your number is private." : "Guess the answer player's secret number.")
    : state.phase === "results" ? "Round complete." : "Game complete.";

  const active = state.phase === "playing";
  const answerRole = active && amAnswerPlayer();
  const guessRole = active && !amAnswerPlayer();
  $("answerArea").classList.toggle("hidden", !answerRole);
  $("guessArea").classList.toggle("hidden", !guessRole);
  $("hostSecretArea").classList.toggle("hidden", !(isHost && active));
  $("hostSecret").textContent = roomData.secrets?.[state.targetUid]?.number ?? "—";
  $("playerSecret").textContent = answerRole ? (mySecret() ?? "—") : "";
  $("waitingArea").classList.toggle("hidden", !(active && hasSubmitted()));
  $("answerReveal").classList.toggle("hidden", state.phase !== "results" && state.phase !== "finished");
  $("hostNextArea").classList.toggle("hidden", !(isHost && state.phase === "results"));
  $("nextRoundBtn").textContent = (state.round >= state.roundLimit) ? "Finish game" : "Next round";
  $("startRoundBtn").disabled = players.length < 2 || state.phase === "playing";
  $("startRoundBtn").textContent = state.phase === "results" ? "Start next round" : "Start round";

  if (state.phase === "results" || state.phase === "finished") {
    const res = state.result || {};
    $("answerReveal").innerHTML =
      `<b>Secret number: ${res.number ?? "?"}</b><br>` +
      `Answer: ${safeText(res.answer || "No answer submitted")}`;
    const resultRows = Object.entries(res.guesses || {}).map(([id,g]) => {
      const p = roomData.players?.[id];
      if (!p) return "";
      return `<div class="scoreline"><span>${safeText(p.name)} guessed ${g.value ?? "—"} (off by ${g.distance ?? "—"})</span><b>+${g.points||0}</b></div>`;
    }).join("");
    $("results").innerHTML = "<h3>Round scores</h3>" + (resultRows || "<p class='muted'>No guesses recorded.</p>");
  } else {
    $("results").innerHTML = "";
  }
}
function hasSubmitted() {
  const state = roomData?.state || {};
  if (amAnswerPlayer()) return !!roomData?.answerStatus?.[uid];
  return !!roomData?.guessStatus?.[uid];
}
async function startRound() {
  try {
    if (!isHost) return;
    const players = getPlayers();
    if (players.length < 2) throw new Error("At least two players are needed.");
    const targetUid = $("targetSelect").value;
    if (!targetUid) throw new Error("Choose the answer player.");
    const category = $("category").value;
    const prompt = randomPrompt(category);
    if (!prompt) throw new Error("Enter a custom prompt first.");
    const oldState = roomData.state || {};
    const round = oldState.round >= oldState.roundLimit ? 1 : (oldState.round || 0) + 1;
    const roundLimit = Number($("roundLimit").value || oldState.roundLimit || 5);
    const number = Math.floor(Math.random() * 10) + 1;

    // Only the host writes the secret. The rules restrict reading it to host and target.
    await roomRef.child("secrets/" + targetUid).set({number, round});
    await roomRef.child("privateGuesses").remove();
    await roomRef.child("guessStatus").remove();
    await roomRef.child("answerStatus").remove();
    await roomRef.child("state").set({
      phase:"playing", round, roundLimit, targetUid, prompt, category,
      answer:null, result:null, startedAt:Date.now()
    });
    $("answerInput").value = "";
    $("guessInput").value = "";
  } catch (e) { showError(e); }
}
async function submitAnswer() {
  try {
    const answer = $("answerInput").value.trim();
    if (!answer) throw new Error("Enter your answer first.");
    if (!isRoundActive() || !amAnswerPlayer()) throw new Error("You are not the answer player.");
    await roomRef.child("state/answer").set(answer);
    await roomRef.child("answerStatus/" + uid).set(true);
    await maybeFinishRound();
  } catch (e) { showError(e); }
}
async function submitGuess() {
  try {
    const value = Number($("guessInput").value);
    if (!value || value < 1 || value > 10) throw new Error("Choose a number from 1 to 10.");
    if (!isRoundActive() || amAnswerPlayer()) throw new Error("You cannot submit a guess in this role.");
    await roomRef.child("privateGuesses/" + uid).set({value});
    await roomRef.child("guessStatus/" + uid).set(true);
    await maybeFinishRound();
  } catch (e) { showError(e); }
}
async function maybeFinishRound() {
  // Host resolves the round after everyone has submitted.
  if (!isHost || !isRoundActive()) return;
  const snap = await roomRef.get();
  const data = snap.val();
  const players = Object.keys(data.players || {});
  const target = data.state.targetUid;
  const allSubmitted = players.every(id => id === target
    ? !!data.answerStatus?.[id]
    : !!data.guessStatus?.[id]);
  if (!allSubmitted) return;

  const number = data.secrets?.[target]?.number;
  if (typeof number !== "number") throw new Error("Secret number missing.");
  const guesses = {};
  for (const id of players) {
    if (id === target) continue;
    const value = data.privateGuesses?.[id]?.value;
    if (typeof value !== "number") continue;
    const distance = Math.abs(value - number);
    const points = distance === 0 ? 4 : distance === 1 ? 2 : distance === 2 ? 1 : 0;
    guesses[id] = {value, distance, points};
    await roomRef.child("players/" + id + "/score").transaction(score => (score || 0) + points);
  }
  await roomRef.child("state").update({
    phase:"results",
    result:{number, answer:data.state.answer || "No answer submitted", guesses}
  });
}
async function nextRound() {
  try {
    if (!isHost) return;
    const state = roomData.state || {};
    if (state.round >= state.roundLimit) {
      await roomRef.child("state/phase").set("finished");
      return;
    }
    await startRound();
  } catch (e) { showError(e); }
}
async function leaveLobby() {
  try {
    if (roomRef && uid) await roomRef.child("players/" + uid).remove();
    if (currentRoomListener && roomRef) roomRef.off("value", currentRoomListener);
  } catch (e) { console.warn(e); }
  roomCode = null; roomRef = null; roomData = null; isHost = false;
  $("game").classList.add("hidden");
  $("setup").classList.remove("hidden");
  message("You left the lobby.");
}

$("createBtn").addEventListener("click", createLobby);
$("showJoinBtn").addEventListener("click", () => $("joinArea").classList.toggle("hidden"));
$("joinBtn").addEventListener("click", joinLobby);
$("lobbyCode").addEventListener("input", e => e.target.value = cleanCode(e.target.value));
$("category").addEventListener("change", () => $("customPrompt").classList.toggle("hidden", $("category").value !== "Custom"));
$("startRoundBtn").addEventListener("click", startRound);
$("submitAnswerBtn").addEventListener("click", submitAnswer);
$("submitGuessBtn").addEventListener("click", submitGuess);
$("nextRoundBtn").addEventListener("click", nextRound);
$("leaveBtn").addEventListener("click", leaveLobby);
</script>
</body>
</html>

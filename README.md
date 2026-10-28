<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Number Bluff</title>
<style>
:root{color-scheme:dark;--bg:#10131c;--card:#1b2030;--line:#343c50;--text:#f5f6fb;--muted:#aeb7cc;--accent:#a78bfa;--green:#6ee7b7}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--text);font:16px/1.5 system-ui,sans-serif}
main{max-width:760px;margin:auto;padding:24px 16px 50px}
h1{font-size:clamp(2.3rem,8vw,4rem);letter-spacing:-2px;margin:0}
h1 span{color:var(--accent)}
h2{margin-top:0}
p{margin:8px 0}
.muted{color:var(--muted)}
.small{font-size:.9rem}
.card{background:var(--card);border:1px solid var(--line);border-radius:16px;padding:18px;margin:16px 0}
.row{display:flex;gap:10px;flex-wrap:wrap}
.row>*{flex:1;min-width:130px}
input,select,button{font:inherit;width:100%;padding:12px;border-radius:10px;border:1px solid var(--line)}
input,select{background:#111622;color:var(--text)}
button{background:var(--accent);color:#171326;font-weight:750;border:0;cursor:pointer}
button.secondary{background:#252c40;color:var(--text);border:1px solid #414b63}
button:disabled{opacity:.45;cursor:not-allowed}
label{display:block;font-weight:650;margin:10px 0 6px}
.hidden{display:none!important}
.code{font-size:2rem;letter-spacing:5px;font-weight:900;color:var(--accent)}
.prompt{font-size:1.4rem;font-weight:750;margin:12px 0}
.number{font-size:4rem;text-align:center;font-weight:900;color:var(--green)}
.notice{background:#292641;border-radius:10px;padding:12px;margin:12px 0}
.player,.score{display:flex;justify-content:space-between;gap:12px;padding:9px 0;border-bottom:1px solid var(--line)}
.player:last-child,.score:last-child{border-bottom:0}
.tag{font-size:.75rem;background:#30384c;padding:3px 7px;border-radius:20px}
.error{color:#fda4af}
footer{color:var(--muted);font-size:.85rem;margin-top:24px}
</style>
</head>
<body>
<main>
<h1>NUMBER <span>BLUFF</span></h1>
<p class="muted">One secret number. One answer. Everyone else takes a guess.</p>

<section id="setup" class="card">
<h2>Join the game</h2>
<label for="name">Your name</label>
<input id="name" maxlength="24" placeholder="Enter your name">
<div class="row" style="margin-top:12px">
<button id="create">Create lobby</button>
<button id="showJoin" class="secondary">Join lobby</button>
</div>
<div id="joinArea" class="hidden">
<label for="codeInput">Four-letter lobby code</label>
<input id="codeInput" maxlength="4" placeholder="ABCD">
<button id="join" style="margin-top:10px">Join game</button>
</div>
<p id="error" class="error small"></p>
</section>

<section id="game" class="hidden">
<div class="card">
<div class="muted small">LOBBY CODE</div>
<div id="codeDisplay" class="code"></div>
<button id="copy" class="secondary" style="margin-top:8px">Copy invite link</button>
<h3>Players</h3>
<div id="players"></div>
</div>

<div id="hostPanel" class="card hidden">
<h2>Host controls</h2>
<label for="category">Category</label>
<select id="category">
<option value="General Knowledge">General Knowledge</option>
<option value="Minecraft">Minecraft</option>
<option value="Custom">Custom prompt</option>
</select>
<label for="customPrompt" id="customLabel" class="hidden">Your prompt</label>
<input id="customPrompt" class="hidden" maxlength="180" placeholder="Write your own prompt">
<div class="row">
<div>
<label for="roundLimit">Total rounds</label>
<select id="roundLimit">
<option>3</option><option selected>5</option><option>10</option><option>15</option><option>20</option>
</select>
</div>
<div>
<label for="target">Answer player</label>
<select id="target"></select>
</div>
</div>
<div class="row" style="margin-top:12px">
<button id="randomPrompt" class="secondary">Random prompt</button>
<button id="start">Start round</button>
</div>
<label for="manualPrompt">Or enter a prompt manually</label>
<input id="manualPrompt" placeholder="Optional prompt override">
<div id="hostSecretBox" class="notice hidden">
<div class="muted small">SECRET NUMBER</div>
<div id="hostSecret" class="number"></div>
</div>
<div id="editScores" style="margin-top:12px">
<button id="editScoreBtn" class="secondary">Edit player scores</button>
</div>
</div>

<div id="roundPanel" class="card hidden">
<div id="roundLabel" class="muted small"></div>
<div id="prompt" class="prompt"></div>
<p id="role" class="muted"></p>
<div id="secretBox" class="notice hidden">
<div class="muted small">YOUR SECRET NUMBER</div>
<div id="secret" class="number"></div>
</div>
<div id="answerArea" class="hidden">
<label for="answer">Your answer</label>
<input id="answer" maxlength="250" placeholder="Type your answer">
<button id="submitAnswer" style="margin-top:10px">Submit answer</button>
</div>
<div id="guessArea" class="hidden">
<label for="guess">Guess the number (1–10)</label>
<select id="guess">
<option value="">Choose a number</option>
<option>1</option><option>2</option><option>3</option><option>4</option><option>5</option>
<option>6</option><option>7</option><option>8</option><option>9</option><option>10</option>
</select>
<button id="submitGuess" style="margin-top:10px">Lock in guess</button>
</div>
<div id="timer" class="number">20</div>
<div id="waiting" class="notice hidden">Submitted! Waiting for the round to end…</div>
<div id="results"></div>
<button id="next" class="hidden">Next round</button>
</div>

<div class="card">
<h2>Leaderboard</h2>
<div id="leaderboard"></div>
<p class="muted small">Exact = 4 points · Off by 1 = 2 points · Off by 2 = 1 point</p>
<button id="leave" class="secondary">Leave lobby</button>
</div>
</section>
<footer>Number Bluff · Peer-to-peer multiplayer prototype</footer>
</main>

<script src="https://cdn.jsdelivr.net/npm/peerjs@1.5.4/dist/peerjs.min.js"></script>
<script>
const PREFIX = "numberbluff-room-";
const $ = id => document.getElementById(id);

const PROMPTS = {
"General Knowledge": [
"How many sides does a hexagon have?",
"How many days are in a leap year?",
"How many players are on a football team on the field?",
"How many colours are traditionally listed in a rainbow?",
"How many continents are commonly taught in school?",
"How many minutes are in an hour?",
"How many keys are on a standard piano?",
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
"How many sides does a triangle have?",
"How many colours are on the flag of Germany?",
"How many letters are in the word 'elephant'?",
"How many legs does a spider have?",
"How many bones are in the adult human body?",
"How many stripes are on the flag of the United States?",
"How many players are on a volleyball team on court?",
"How many minutes are in a day?",
"How many sides does a pentagon have?",
"How many hearts does an octopus have?",
"How many strings does a violin have?"
],
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
"How many blocks are in a full 3-by-3 beacon base layer?",
"How many snowballs can one snow block craft?",
"How many Ender Pearls are needed to craft an Eye of Ender?",
"How many books are needed to craft a bookshelf?",
"How many wool blocks are needed to craft a bed?",
"How many gunpowder are needed to craft TNT?",
"How many legs does a spider have in Minecraft?",
"How many slots does a double chest have?",
"How many iron ingots are needed for an iron pickaxe?",
"How many planks are needed to craft a boat?",
"How many diamonds are needed for a diamond sword?",
"How many obsidian blocks are needed to craft an enchanting table?",
"How many items can a standard stack hold?",
"How many gold ingots are needed to craft a golden apple?",
"How many Ender Eyes are needed to activate a full End Portal?",
"How many slots does a player's inventory hotbar contain?",
"How many blocks of iron are needed for a full beacon pyramid base?"
]
};

let peer, hostConn, isHost = false, myId = "", myName = "", roomCode = "";
let players = {}; // id -> {name, score}
let conns = {};   // host only: id -> PeerJS connection
let guesses = {}; // host only: id -> guess
let answers = {}; // host only: id -> answer
let secretNumber = null, targetId = null;
let round = 0, roundLimit = 5, phase = "lobby";
let promptText = "", endAt = 0, timerLoop = null, submitted = false;
let usedPrompts = new Set(), lastResult = null;

function setError(s=""){ $("error").textContent = s; }
function makeCode(){
  const chars="ABCDEFGHJKLMNPQRSTUVWXYZ";
  return Array.from({length:4},()=>chars[Math.floor(Math.random()*chars.length)]).join("");
}
function nameInput(){return $("name").value.trim().slice(0,24);}
function send(conn,msg){try{if(conn && conn.open) conn.send(msg);}catch(e){}}
function broadcast(msg){Object.values(conns).forEach(c=>send(c,msg));}
function showGame(){
  $("setup").classList.add("hidden");
  $("game").classList.remove("hidden");
  $("codeDisplay").textContent=roomCode;
  render();
}
function showHost(){ $("hostPanel").classList.toggle("hidden",!isHost); }
function publicState(){
  return {
    type:"state", phase, round, roundLimit, promptText, targetId,
    players, endAt,
    // Never send the secret number or unrevealed guesses in public state.
    result: phase==="results" ? lastResult : null
  };
}
function publish(){broadcast(publicState());render();}
function render(){
  if(!roomCode)return;
  showHost();
  $("players").innerHTML=Object.entries(players).map(([id,p])=>
    `<div class="player"><span>${esc(p.name)} ${id===myId?"<span class='tag'>YOU</span>":""} ${id==="host"?"<span class='tag'>HOST</span>":""}</span><b>${p.score||0} pts</b></div>`
  ).join("") || "<p class='muted'>Waiting for players…</p>";
  $("leaderboard").innerHTML=Object.entries(players)
    .sort((a,b)=>(b[1].score||0)-(a[1].score||0))
    .map(([id,p],i)=>`<div class="score"><span>${i+1}. ${esc(p.name)}</span><b>${p.score||0}</b></div>`).join("");

  const old=$("target").value;
  $("target").innerHTML=Object.entries(players).map(([id,p])=>
    `<option value="${id}">${esc(p.name)}</option>`).join("");
  if(players[old])$("target").value=old;

  $("roundPanel").classList.toggle("hidden",phase==="lobby");
  $("roundLabel").textContent=phase==="results"?
    `ROUND ${round} / ${roundLimit}`:`ROUND ${round} / ${roundLimit}`;
  $("prompt").textContent=promptText;
  $("hostSecretBox").classList.toggle("hidden",!(isHost&&phase==="playing"));
  $("hostSecret").textContent=isHost&&phase==="playing"&&secretNumber!==null?secretNumber:"";
  $("role").textContent=phase==="playing"?
    (targetId===myId?"You are answering. Keep your secret number hidden!":"Guess the answer player's number.")
    :phase==="results"?"Round results":"";
  $("secretBox").classList.toggle("hidden",!(phase==="playing"&&targetId===myId));
  $("secret").textContent=targetId===myId&&secretNumber!==null?secretNumber:"";
  $("answerArea").classList.toggle("hidden",!(phase==="playing"&&targetId===myId&&!submitted));
  $("guessArea").classList.toggle("hidden",!(phase==="playing"&&targetId!==myId&&!submitted));
  $("waiting").classList.toggle("hidden",!(phase==="playing"&&submitted));
  $("next").classList.toggle("hidden",!(isHost&&phase==="results"));
  $("next").textContent=round>=roundLimit?"Finish game":"Next round";

  if(phase==="results"&&lastResult){
    $("timer").textContent="✓";
    $("results").innerHTML=`<div class="notice"><b>Secret number: ${lastResult.number}</b><br>Answer: ${esc(lastResult.answer||"(no answer)")}</div>`+
      Object.entries(lastResult.guesses||{}).map(([id,g])=>
      `<div class="score"><span>${esc(players[id]?.name||"Player")} guessed ${g.value} (off by ${g.distance})</span><b>+${g.points}</b></div>`
      ).join("");
  }else if(phase==="playing"){
    $("results").innerHTML="";
  }
}
function esc(s){
  return String(s??"").replace(/[&<>"']/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}[c]));
}

/* Host creates a room using a PeerJS ID derived from its code. */
$("create").onclick=()=>{
  myName=nameInput();
  if(!myName)return setError("Enter your name first.");
  if(!window.Peer)return setError("PeerJS could not load. Check your internet connection.");
  setError("");
  roomCode=makeCode();
  peer=new Peer(PREFIX+roomCode.toLowerCase());
  peer.on("open",id=>{
    isHost=true;myId="host";
    players.host={name:myName,score:0};
    peer.on("connection",c=>{
      c.on("data",m=>hostMessage(c,m));
      c.on("close",()=>{delete conns[c.peer];delete players[c.peer];delete guesses[c.peer];delete answers[c.peer];publish();});
      c.on("error",()=>{delete conns[c.peer];delete players[c.peer];publish();});
      c.on("open",()=>{});
    });
    showGame();
    publish();
  });
  peer.on("error",e=>setError("Could not create lobby ("+e.type+"). Try again."));
};

/* Guests connect directly to the host. */
$("showJoin").onclick=()=>$("joinArea").classList.toggle("hidden");
$("join").onclick=()=>{
  myName=nameInput();
  const code=$("codeInput").value.trim().toUpperCase();
  if(!myName)return setError("Enter your name first.");
  if(!/^[A-Z]{4}$/.test(code))return setError("Enter a four-letter lobby code.");
  if(!window.Peer)return setError("PeerJS could not load. Check your internet connection.");
  setError("");
  roomCode=code;
  peer=new Peer();
  let finished=false;
  const timeout=setTimeout(()=>{if(!finished){setError("Could not reach the lobby. Check the code and try again.");peer.destroy();}},15000);
  peer.on("open",id=>{
    myId=id;
    hostConn=peer.connect(PREFIX+code.toLowerCase(),{reliable:true});
    hostConn.on("open",()=>{
      if(finished)return;finished=true;clearTimeout(timeout);
      send(hostConn,{type:"join",name:myName});
      showGame();
    });
    hostConn.on("data",m=>guestMessage(m));
    hostConn.on("close",()=>setError("The host disconnected. Reload to start again."));
    hostConn.on("error",()=>setError("Connection to the host was lost."));
  });
  peer.on("error",e=>{
    clearTimeout(timeout);
    if(!finished)setError(e.type==="peer-unavailable"?"Lobby not found. Check the code.":"Connection error: "+e.type);
  });
};
$("codeInput").oninput=e=>e.target.value=e.target.value.toUpperCase().replace(/[^A-Z]/g,"").slice(0,4);
$("copy").onclick=async()=>{
  const link=location.href.split("#")[0]+"#"+roomCode;
  try{await navigator.clipboard.writeText(link);$("copy").textContent="Copied!";}
  catch(e){prompt("Copy this invite link:",link);}
  setTimeout(()=>$("copy").textContent="Copy invite link",2000);
};
if(/^#[A-Z]{4}$/i.test(location.hash))$("codeInput").value=location.hash.slice(1).toUpperCase();

function hostMessage(c,m){
  if(!m||typeof m!=="object")return;
  if(m.type==="join"&&phase==="lobby"){
    conns[c.peer]=c;
    players[c.peer]={name:String(m.name||"Player").slice(0,24),score:0};
    send(c,publicState());
    publish();
  }else if(m.type==="answer"&&phase==="playing"&&m.round===round&&c.peer===targetId){
    answers[c.peer]=String(m.value||"").slice(0,250);
  }else if(m.type==="guess"&&phase==="playing"&&m.round===round&&c.peer!==targetId){
    const n=Number(m.value);
    if(Number.isInteger(n)&&n>=1&&n<=10)guesses[c.peer]=n;
  }
}
function guestMessage(m){
  if(!m||typeof m!=="object")return;
  if(m.type==="state"){
    phase=m.phase;round=m.round;roundLimit=m.roundLimit;promptText=m.promptText;
    targetId=m.targetId;players=m.players||{};endAt=m.endAt||0;lastResult=m.result||null;
    if(phase!=="playing"){secretNumber=null;submitted=false;}
    render();
    if(phase==="playing"&&endAt)startTimer();
  }else if(m.type==="secret"&&m.round===round){
    secretNumber=m.number;render();
  }else if(m.type==="result"){
    lastResult=m.result;phase="results";render();
  }
}

/* Start a round: host chooses the target and privately sends their number. */
$("category").onchange=()=>{
  const custom=$("category").value==="Custom";
  $("customPrompt").classList.toggle("hidden",!custom);
  $("customLabel").classList.toggle("hidden",!custom);
};
$("randomPrompt").onclick=()=>{
  const category=$("category").value;
  if(category==="Custom")return setError("Enter a custom prompt first.");
  const list=PROMPTS[category]||PROMPTS["General Knowledge"];
  let pool=list.filter(p=>!usedPrompts.has(p));
  if(!pool.length){usedPrompts.clear();pool=list;}
  $("manualPrompt").value=pool[Math.floor(Math.random()*pool.length)];
  setError("");
};
$("start").onclick=()=>{
  if(!isHost||Object.keys(players).length<2)return setError("At least two players are needed.");
  if(phase==="playing")return;
  const category=$("category").value;
  let p=$("manualPrompt").value.trim();
  if(!p)p=category==="Custom"?$("customPrompt").value.trim():"";
  if(!p&&category!=="Custom"){
    const list=PROMPTS[category]||PROMPTS["General Knowledge"];
    let pool=list.filter(x=>!usedPrompts.has(x));
    if(!pool.length){usedPrompts.clear();pool=list;}
    p=pool[Math.floor(Math.random()*pool.length)];
  }
  if(!p)return setError("Enter a custom prompt first.");
  promptText=p;usedPrompts.add(p);
  roundLimit=Number($("roundLimit").value)||5;
  round=round+1;
  targetId=$("target").value;
  secretNumber=Math.floor(Math.random()*10)+1;
  guesses={};answers={};lastResult=null;submitted=false;
  phase="playing";endAt=Date.now()+20000;
  // Tell the target their number privately, never in the public state.
  if(targetId==="host"){
    // Host already has the secret in local state.
  }else{
    send(conns[targetId],{type:"secret",number:secretNumber,round});
  }
  publish();
  startTimer();
  setError("");
};
function startTimer(){
  if(timerLoop)clearInterval(timerLoop);
  const tick=()=>{
    if(phase!=="playing"){clearInterval(timerLoop);return;}
    const remaining=Math.max(0,Math.ceil((endAt-Date.now())/1000));
    $("timer").textContent=remaining;
    if(remaining<=0&&isHost)finishRound();
  };
  tick();timerLoop=setInterval(tick,250);
}

$("submitAnswer").onclick=()=>{
  if(phase!=="playing"||targetId!==myId||submitted)return;
  const value=$("answer").value.trim();
  if(!value)return setError("Enter your answer first.");
  submitted=true;
  if(isHost)answers.host=value;
  else send(hostConn,{type:"answer",round,value});
  $("waiting").classList.remove("hidden");render();
};
$("submitGuess").onclick=()=>{
  if(phase!=="playing"||targetId===myId||submitted)return;
  const value=Number($("guess").value);
  if(!value)return setError("Choose a number first.");
  submitted=true;
  if(isHost)guesses.host=value;
  else send(hostConn,{type:"guess",round,value});
  $("waiting").classList.remove("hidden");render();
};

function finishRound(){
  if(!isHost||phase!=="playing")return;
  phase="results";
  const resultGuesses={};
  for(const [id,value] of Object.entries(guesses)){
    const distance=Math.abs(value-secretNumber);
    const points=distance===0?4:distance===1?2:distance===2?1:0;
    resultGuesses[id]={value,distance,points};
    if(players[id])players[id].score=(players[id].score||0)+points;
  }
  lastResult={number:secretNumber,answer:answers[targetId]||"",guesses:resultGuesses};
  publish();
  broadcast({type:"result",result:lastResult});
  render();
}
$("next").onclick=()=>{
  if(!isHost)return;
  if(round>=roundLimit){
    phase="finished";promptText="Game over!";publish();return;
  }
  phase="lobby";promptText="";secretNumber=null;targetId=null;
  publish();
};
$("editScoreBtn").onclick=()=>{
  if(!isHost)return;
  const lines=Object.entries(players).map(([id,p])=>`${id} = ${p.name}: ${p.score||0}`).join("\n");
  const id=prompt("Enter the player ID to edit:\n"+lines);
  if(!id||!players[id])return;
  const score=prompt("New score for "+players[id].name,players[id].score||0);
  if(score===null||!Number.isFinite(Number(score)))return;
  players[id].score=Number(score);
  publish();
};
$("leave").onclick=()=>{
  if(peer)peer.destroy();
  location.href=location.href.split("#")[0];
};
</script>
</body>
</html>

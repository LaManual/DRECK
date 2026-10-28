<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>On a Scale – Party Game</title>
<script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
<style>
  :root{--bg:#14161f;--card:#1f2230;--line:#2f3346;--txt:#eef0f6;--mut:#9aa0b5;--acc:#ffb547;--ok:#4cd28a;--bad:#ff6b6b}
  *{box-sizing:border-box}
  body{margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,sans-serif;background:var(--bg);color:var(--txt);min-height:100vh}
  .wrap{max-width:760px;margin:0 auto;padding:24px 16px}
  h1{font-size:2rem;margin:0 0 4px;letter-spacing:-.02em}
  h1 span{color:var(--acc)}
  h2{margin:0 0 12px;font-size:1.2rem}
  .sub{color:var(--mut);margin:0 0 20px}
  .card{background:var(--card);border:1px solid var(--line);border-radius:14px;padding:18px;margin-bottom:16px}
  input,select,textarea{width:100%;padding:11px 12px;border-radius:10px;border:1px solid var(--line);background:#151824;color:var(--txt);font-size:1rem;font-family:inherit}
  textarea{min-height:90px;resize:vertical}
  label{display:block;font-size:.85rem;color:var(--mut);margin:10px 0 6px}
  button{background:var(--acc);color:#1a1300;border:0;border-radius:10px;padding:11px 16px;font-weight:700;font-size:1rem;cursor:pointer;margin-top:10px}
  button.sec{background:#2c3044;color:var(--txt)}
  button:disabled{opacity:.4;cursor:not-allowed}
  .row{display:flex;gap:10px;flex-wrap:wrap}
  .row>*{flex:1;min-width:140px}
  .code{font-size:2.6rem;font-weight:800;letter-spacing:.3em;color:var(--acc)}
  .prompt{font-size:1.4rem;font-weight:700;line-height:1.3}
  .scale{display:flex;justify-content:space-between;color:var(--mut);font-size:.8rem;margin-top:6px}
  .secret{font-size:5rem;font-weight:900;color:var(--acc);text-align:center;line-height:1}
  .answer{font-size:1.5rem;font-style:italic;padding:12px;border-left:4px solid var(--acc);background:#171a26;border-radius:6px}
  .answer.hidden{opacity:.75;background:#1a1d2a;border-left-color:#49506a;color:var(--mut)}
  .nums{display:grid;grid-template-columns:repeat(5,1fr);gap:8px}
  .nums button{margin:0;padding:16px 0;font-size:1.3rem;background:#2c3044;color:var(--txt)}
  .nums button.sel{background:var(--acc);color:#1a1300}
  .timer{font-size:1.1rem;font-weight:800;color:var(--acc);float:right}
  table{width:100%;border-collapse:collapse}
  td,th{padding:8px 6px;border-bottom:1px solid var(--line);text-align:left}
  th{color:var(--mut);font-weight:600;font-size:.85rem}
  td input{width:80px;padding:6px}
  .pill{display:inline-block;padding:3px 9px;border-radius:99px;background:#2c3044;font-size:.8rem;margin:2px}
  .mut{color:var(--mut)}
  .err{color:var(--bad)}
  .good{color:var(--ok)}

  .leaderboard{overflow:hidden}
  .leaderboard table{border-collapse:separate;border-spacing:0 7px;width:100%}
  .leaderboard thead th{padding:0 8px 6px;color:var(--mut);font-size:.8rem;text-transform:uppercase;letter-spacing:.05em}
  .leaderboard tbody tr{background:rgba(255,255,255,.025)}
  .leaderboard tbody tr:first-child{background:rgba(255,181,71,.09);box-shadow:inset 0 0 0 1px rgba(255,181,71,.18)}
  .leaderboard td{padding:10px 8px;border-bottom:none}
  .leaderboard td:first-child{border-radius:10px 0 0 10px}
  .leaderboard td:last-child{border-radius:0 10px 10px 0}
  .leaderboard .score-input{width:72px;background:#111722;padding:6px 8px;border-radius:8px;border:1px solid var(--line);color:var(--txt)}
</style>
</head>
<body>
<div class="wrap">
  <h1>On a <span>Scale</span></h1>
  <p class="sub">Describe your secret number. Let your friends guess it.</p>
  <div id="app"></div>
</div>

<script>
const PREFIX = "onascale-game-v1-";
const CATEGORIES = {
  "Food": ["How spicy is this food?","How healthy is this meal?","How fancy is this restaurant?","How weird is this pizza topping?","How good is this breakfast?"],
  "Movies & TV": ["How scary is this movie?","How overrated is this show?","How good is this villain?","How rewatchable is this film?","How cringe is this movie scene?"],
  "Everyday Life": ["How annoying is this habit?","How embarrassing is this situation?","How useful is this object?","How stressful is this situation?","How much would you pay for this?"],
  "Animals": ["How dangerous is this animal?","How cute is this animal?","How good of a pet is this?","How smart is this animal?","How weird is this animal?"],
  "Hypothetical": ["How good is this superpower?","How bad is this way to die in a game?","How hard is this job?","How likely is this to happen in 10 years?","How good is this first date idea?"],
  "Spicy Opinions": ["How controversial is this opinion?","How toxic is this behaviour?","How sus is this excuse?","How red-flag is this?","How acceptable is this at a party?"]
};
const TIMES = [10,15,20,25,30,40,45,50,60];

let peer=null, isHost=false, myId=null, myName="", hostConn=null, conns={};
let state=null, mySecret=null, localDeadline=0, error="", myGuess=null, screen="home";
let hostTimer=null;

const esc=s=>String(s??"").replace(/[&<>\"']/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;","\"":"&quot;","'":"&#39;"}[c]));
const $=id=>document.getElementById(id);

/* ---------------- RENDER ---------------- */
function render(){
  const saved={}; let focus=document.activeElement?.id, selS, selE;
  document.querySelectorAll("#app input,#app textarea,#app select").forEach(e=>{if(e.id) saved[e.id]=e.value});
  if(focus && $(focus) && "selectionStart" in $(focus)){try{selS=$(focus).selectionStart;selE=$(focus).selectionEnd}catch(_){} }
  $("app").innerHTML = view();
  for(const id in saved){ if($(id) && !$(id).dataset.fresh) $(id).value=saved[id]; }
  if(focus && $(focus)){ $(focus).focus(); try{ if(selS!=null) $(focus).setSelectionRange(selS,selE);}catch(_){} }
  tick();
}

function view(){
  if(screen==="home") return `
    <div class="card">
      <label>Your name</label><input id="name" maxlength="16" placeholder="e.g. Manuel">
      <div class="row">
        <div><button onclick="hostRoom()">Host a room</button></div>
      </div>
      <label>Room code</label>
      <div class="row"><input id="code" maxlength="4" placeholder="ABCD" style="text-transform:uppercase"><button class="sec" onclick="joinRoom()" style="margin-top:0;flex:0 0 auto">Join</button></div>
      ${error?`<p class="err">${esc(error)}</p>`:""}
    </div>
    <div class="card mut"><h2 style="color:var(--txt)">How to play</h2>
    The host picks a prompt and a player. That player secretly gets a number from 1 to 10 and writes an answer that fits that number on the scale.
    Everyone else guesses the number: <b>4 pts</b> exact, <b>2 pts</b> off by 1, <b>1 pt</b> off by 2.</div>`;
  if(screen==="connecting") return `<div class="card">Connecting…${error?`<p class="err">${esc(error)}</p><button class="sec" onclick="location.reload()">Back</button>`:""}</div>`;
  if(!state) return `<div class="card">Waiting for host…</div>`;
  const s=state;
  let h=`<div class="card"><div class="row" style="align-items:center"><div><span class="mut">Room</span><div class="code">${esc(s.code)}</div></div>
    <div style="text-align:right" class="mut">${s.phase==="lobby"?"Lobby":`Round ${s.round} / ${s.totalRounds}`}<br>${isHost?"You are the host":"Playing as <b>"+esc(myName)+"</b>"}</div></div></div>`;
  h+=phaseView(s);
  h+=leaderboard(s);
  return h;
}

function activeName(s){return s.players.find(p=>p.id===s.activeId)?.name||"?"}

function promptCard(s){return `<div class="card"><div class="mut">Prompt</div><div class="prompt">${esc(s.prompt)}</div><div class="scale"><span>1 = least</span><span>10 = most</span></div></div>`}

function phaseView(s){
  const amActive = myId===s.activeId;
  switch(s.phase){
    case "lobby":
      if(isHost) return `<div class="card"><h2>Game settings</h2>
        <div class="row">
          <div><label>Rounds</label><input id="rounds" type="number" min="1" max="50" value="5"></div>
          <div><label>Answer time</label><select id="atime">${TIMES.map(t=>`<option value="${t}" ${t===30?"selected":""}>${t}s</option>`).join("")}</select></div>
          <div><label>Guess time</label><select id="gtime">${TIMES.map(t=>`<option value="${t}" ${t===20?"selected":""}>${t}s</option>`).join("")}</select></div>
        </div>
        <p class="mut">Share the room code with your friends. Players: ${s.players.length}</p>
        <button onclick="startGame()" ${s.players.length<2?"disabled":""}>Start game</button>
        ${s.players.length<2?`<span class="mut"> &nbsp;Need at least 2 players</span>`:""}</div>`;
      return `<div class="card">Waiting for the host to start… <div style="margin-top:8px">${s.players.map(p=>`<span class="pill">${esc(p.name)}</span>`).join("")}</div></div>`;

    case "prompt":
      if(isHost) return `<div class="card"><h2>Choose a prompt</h2>
        <label>Pick a category (random prompt)</label>
        <div class="row"><select id="cat">${Object.keys(CATEGORIES).map(c=>`<option>${esc(c)}</option>`).join("")}</select>
        <button class="sec" style="margin-top:0;flex:0 0 auto" onclick="pickCategory()">Random prompt</button></div>
        <label>…or write your own</label>
        <div class="row"><input id="custom" placeholder="How ___ is this?"><button class="sec" style="margin-top:0;flex:0 0 auto" onclick="pickCustom()">Use this</button></div></div>`;
      return `<div class="card">The host is choosing a prompt…</div>`;

    case "select":
      return promptCard(s) + (isHost ? `<div class="card"><h2>Who describes the number?</h2>
        <div class="row">${s.players.map(p=>`<button class="sec" onclick="selectPlayer('${p.id}')">${esc(p.name)}</button>`).join("")}</div>
        <button class="sec" onclick="newPrompt()">Change prompt</button></div>`
        : `<div class="card">The host is choosing a player…</div>`);

    case "ready":
      if(isHost) return promptCard(s)+`<div class="card"><h2>Ready to start</h2>
        <div class="mut">Selected player</div>
        <div class="prompt" style="margin-top:8px">${esc(activeName(s))}</div>
        <button onclick="startRound()">Start round</button>
        <button class="sec" onclick="newPrompt()">Choose a different prompt</button></div>`;
      return promptCard(s)+`<div class="card">The host is starting the round…</div>`;

    case "answer":
      if(amActive) return promptCard(s)+`<div class="card"><span class="timer" id="timer"></span><div class="mut">Your secret number</div>
        <div class="secret">${mySecret??"…"}</div>
        <label>Write something that fits a ${mySecret} on this scale</label>
        <div class="row"><input id="ans" maxlength="120" placeholder="Your answer" onkeydown="if(event.key==='Enter')sendAnswer()"><button style="margin-top:0;flex:0 0 auto" onclick="sendAnswer()">Send answer</button></div></div>`;
      return promptCard(s)+`<div class="card"><span class="timer" id="timer"></span>
        ${isHost?`<div class="mut">Secret number for ${esc(activeName(s))}</div><div class="secret">${s._secret}</div>`:""}
        <p><b>${esc(activeName(s))}</b> is writing an answer…</p></div>`;

    case "guess": {
      const visible = isHost || !s.answerHidden;
      const answerText = visible ? `<div class="answer">${esc(s.answer)}</div>` : `<div class="answer hidden">Answer hidden until the host reveals it.</div>`;
      const top = promptCard(s)+`<div class="card"><div class="mut">${esc(activeName(s))}'s answer</div>${answerText}</div>`;
      const done = s.guessed.map(id=>s.players.find(p=>p.id===id)?.name).filter(Boolean);
      const status=`<p class="mut">Guessed: ${done.length?done.map(n=>`<span class="pill">${esc(n)}</span>`).join(""):"nobody yet"}</p>`;
      if(isHost) return top+`<div class="card"><span class="timer" id="timer"></span><div class="mut">Secret number</div><div class="secret">${s._secret}</div>${status}
        ${s.answerHidden?`<button class="sec" onclick="revealAnswer()">Reveal answer</button>`:`<button class="sec" onclick="reveal()">Reveal now</button>`}</div>`;
      if(amActive) return top+`<div class="card"><span class="timer" id="timer"></span>Everyone is guessing your number…${status}</div>`;
      const mine = s.guessed.includes(myId);
      return top+`<div class="card"><span class="timer" id="timer"></span><h2>Which number was it?</h2>
        <div class="nums">${[1,2,3,4,5,6,7,8,9,10].map(n=>`<button class="${myGuess===n?"sel":""}" ${mine?"disabled":""} onclick="sendGuess(${n})">${n}</button>`).join("")}</div>
        ${mine?`<p class="good">Guess locked in: ${myGuess}</p>`:""}${status}</div>`;
    }

    case "reveal": {
      const rows=s.players.filter(p=>p.id!==s.activeId).map(p=>{
        const g=s.guesses[p.id]; const pts=g==null?0:points(g,s.secret);
        return `<tr><td>${esc(p.name)}</td><td>${g??"–"}</td><td class="${pts?"good":"mut"}">+${pts}</td></tr>`}).join("");
      return promptCard(s)+`<div class="card"><div class="mut">${esc(activeName(s))}'s answer</div><div class="answer">${esc(s.answer)}</div>
        <div class="mut" style="margin-top:14px">The number was</div><div class="secret">${s.secret}</div>
        <table style="margin-top:12px"><tr><th>Player</th><th>Guess</th><th>Points</th></tr>${rows}</table>
        ${isHost?`<button onclick="nextRound()">${s.round>=s.totalRounds?"Finish game":"Next round"}</button>`:""}</div>`;
    }

    case "end": {
      const sorted=[...s.players].sort((a,b)=>b.score-a.score);
      return `<div class="card" style="text-align:center"><div class="mut">Winner</div><div class="secret" style="font-size:3rem">${esc(sorted[0]?.name||"")}</div>
        ${isHost?`<button onclick="restart()">Play again</button>`:""}</div>`;
    }
  }
  return "";
}

function leaderboard(s){
  const sorted=[...s.players].sort((a,b)=>b.score-a.score);
  return `<div class="card leaderboard"><h2>Leaderboard</h2><table><thead><tr><th>#</th><th>Player</th><th>Points</th></tr></thead><tbody>
    ${sorted.map((p,i)=>`<tr><td>${i+1}</td><td>${esc(p.name)}${p.id===myId?" <span class='mut'>(you)</span>":""}${s.activeId===p.id&&s.phase!="lobby"?" ✍️":""}</td>
    <td>${isHost?`<input class="score-input" type="number" value="${p.score}" data-fresh="1" onchange="editScore('${p.id}',this.value)">`:p.score}</td></tr>`).join("")}
    </tbody></table>${isHost?`<p class="mut" style="font-size:.8rem">You can edit points directly.</p>`:""}</div>`;
}

function tick(){
  const el=$("timer"); if(!el) return;
  const left=Math.max(0,Math.ceil((localDeadline-Date.now())/1000));
  el.textContent=left+"s";
}
setInterval(tick,250);

/* ---------------- NETWORK ---------------- */
function getName(){
  const n=$("name").value.trim();
  if(!n){error="Please enter your name.";render();return null}
  return n;
}
function randCode(){const L="ABCDEFGHJKLMNPQRSTUVWXYZ";return Array.from({length:4},()=>L[Math.floor(Math.random()*L.length)]).join("")}

function hostRoom(){
  const n=getName(); if(!n) return;
  myName=n; isHost=true; error=""; screen="connecting"; render();
  const code=randCode();
  peer=new Peer(PREFIX+code);
  peer.on("open",id=>{
    myId=id;
    state={code,phase:"lobby",players:[],round:0,totalRounds:5,answerTime:30,guessTime:20,prompt:"",activeId:null,answer:"",guesses:{},guessed:[],secret:null,_secret:null,timeLeft:0,answerHidden:false};
    screen="game"; render();
  });
  peer.on("connection",c=>{
    c.on("data",d=>onHostData(c,d));
    c.on("close",()=>{delete conns[c.peer];});
  });
  peer.on("error",e=>{ if(e.type==="unavailable-id"){peer.destroy();hostRoom();} else {error="Connection error: "+e.type;render();}});
}

function joinRoom(){
  const n=getName(); if(!n) return;
  const code=$("code").value.trim().toUpperCase();
  if(!/^[A-Z]{4}$/.test(code)){error="Enter the 4-letter room code.";render();return}
  myName=n; error=""; screen="connecting"; render();
  peer=new Peer();
  peer.on("open",id=>{
    myId=id;
    hostConn=peer.connect(PREFIX+code,{reliable:true});
    hostConn.on("open",()=>{hostConn.send({t:"join",name:myName}); screen="game"; render();});
    hostConn.on("data",onClientData);
    hostConn.on("close",()=>{error="Disconnected from host.";screen="connecting";render();});
  });
  peer.on("error",e=>{error=e.type==="peer-unavailable"?"Room not found. Check the code.":"Connection error: "+e.type;render();});
}

function onClientData(d){
  if(d.t==="state"){
    const prevPhase=state?.phase;
    state=d.state;
    localDeadline=Date.now()+state.timeLeft*1000;
    if(state.phase!=="answer" && state.phase!=="guess"){ if(state.phase!=="reveal") mySecret=null; }
    if(state.phase==="guess" && prevPhase!=="guess") myGuess=null;
    if(state.phase!==prevPhase && $("ans")) $("ans").value="";
    render();
  } else if(d.t==="secret"){ mySecret=d.n; render(); }
}

/* ---------------- HOST LOGIC ---------------- */
function broadcast(){
  const left=Math.max(0,Math.round((localDeadline-Date.now())/1000));
  state.timeLeft = (state.phase==="answer"||state.phase==="guess")?left:0;
  const pub={...state,_secret:null};
  if(state.phase!=="reveal"){pub.secret=null; pub.guesses={};}
  if(state.phase==="guess" && state.answerHidden){ pub.answer = "Answer hidden until the host reveals it."; }
  for(const id in conns){ try{conns[id].send({t:"state",state:pub})}catch(_){} }
  render();
}

function onHostData(c,d){
  if(d.t==="join"){
    conns[c.peer]=c;
    let p=state.players.find(p=>p.id===c.peer);
    if(!p) state.players.push({id:c.peer,name:String(d.name).slice(0,16),score:0});
    broadcast();
    if(state.phase==="answer" && c.peer===state.activeId) c.send({t:"secret",n:state._secret});
  } else if(d.t==="answer" && state.phase==="answer" && c.peer===state.activeId){
    state.answer=String(d.text).slice(0,120)||"(no answer)";
    startGuessing();
  } else if(d.t==="guess" && state.phase==="guess" && c.peer!==state.activeId && !state.guessed.includes(c.peer)){
    const n=parseInt(d.n); if(n<1||n>10) return;
    state.guesses[c.peer]=n; state.guessed.push(c.peer);
    const guessers=state.players.filter(p=>p.id!==state.activeId && conns[p.id]);
    if(guessers.every(p=>state.guessed.includes(p.id))) reveal(); else broadcast();
  }
}

function setTimer(sec,fn){
  clearTimeout(hostTimer);
  localDeadline=Date.now()+sec*1000;
  hostTimer=setTimeout(fn,sec*1000);
}

function startGame(){
  state.totalRounds=Math.max(1,Math.min(50,parseInt($("rounds").value)||5));
  state.answerTime=parseInt($("atime").value); state.guessTime=parseInt($("gtime").value);
  state.players.forEach(p=>p.score=0);
  state.round=1; state.phase="prompt"; broadcast();
}
function pickCategory(){const list=CATEGORIES[$("cat").value]; let p; do{p=list[Math.floor(Math.random()*list.length)]}while(p===state.prompt&&list.length>1); setPrompt(p);}
function pickCustom(){const v=$("custom").value.trim(); if(v) setPrompt(v);}
function setPrompt(p){state.prompt=p; state.phase="select"; broadcast();}
function newPrompt(){state.phase="prompt"; broadcast();}

function selectPlayer(id){
  state.activeId=id; state._secret=1+Math.floor(Math.random()*10);
  state.answer=""; state.guesses={}; state.guessed=[]; state.secret=null; state.answerHidden=true;
  state.phase="ready";
  state.timeLeft=0;
  clearTimeout(hostTimer);
  broadcast();
}

function startRound(){
  state.phase="answer";
  setTimer(state.answerTime,()=>{ if(state.phase==="answer"){state.answer="(ran out of time)";startGuessing();} });
  broadcast();
  conns[state.activeId]?.send({t:"secret",n:state._secret});
}

function startGuessing(){
  state.phase="guess";
  state.answerHidden=true;
  setTimer(state.guessTime,()=>{ if(state.phase==="guess") reveal(); });
  broadcast();
}

function revealAnswer(){
  state.answerHidden=false;
  broadcast();
}

function points(g,s){const d=Math.abs(g-s);return d===0?4:d===1?2:d===2?1:0}
function reveal(){
  clearTimeout(hostTimer);
  state.secret=state._secret;
  state.answerHidden=false;

  const answerer=state.players.find(p=>p.id===state.activeId);
  for(const id in state.guesses){
    const p=state.players.find(p=>p.id===id);
    const pts=points(state.guesses[id],state.secret);
    if(p) p.score += pts;
    if(answerer) answerer.score += pts;
  }
  state.phase="reveal"; broadcast();
}
function nextRound(){
  if(state.round>=state.totalRounds){state.phase="end";}
  else {state.round++; state.phase="prompt"; state.activeId=null;}
  broadcast();
}
function restart(){state.phase="lobby"; state.round=0; state.activeId=null; broadcast();}
function editScore(id,v){const p=state.players.find(p=>p.id===id); if(p){p.score=parseInt(v)||0; broadcast();}}

/* ---------------- PLAYER ACTIONS ---------------- */
function sendAnswer(){const v=$("ans").value.trim(); if(!v) return; hostConn.send({t:"answer",text:v});}
function sendGuess(n){myGuess=n; hostConn.send({t:"guess",n}); render();}

render();
</script>
</body>
</html>

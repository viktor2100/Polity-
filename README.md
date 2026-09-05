# Polity-

<!-- POLITIA: Claude-independent multiplayer build.
     Deploy this single HTML file to any static host (GitHub Pages, Cloudflare Pages, Netlify, etc.).
     Multiplayer transport uses PeerJS/WebRTC; players do not need a Claude account. -->
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ПОЛИТИЯ</title>
<style>
  :root{
    --bg:#0e0c09;
    --bg2:#1b1712;
    --bg3:#241f18;
    --brass:#b3925a;
    --brass-bright:#d8b978;
    --brass-dark:#6f5a34;
    --marble:#eee7d6;
    --marble-dim:#c9c0aa;
    --ink:#0e0c09;
    --crimson:#8f2a24;
    --crimson-bright:#c33a30;
  }
  *{box-sizing:border-box;}
  html,body{height:100%;}
  body{
    margin:0;
    background:
      radial-gradient(ellipse at 50% -10%, #241f16 0%, var(--bg) 55%),
      var(--bg);
    color:var(--marble);
    font-family:Georgia, 'Iowan Old Style', 'Times New Roman', serif;
    -webkit-font-smoothing:antialiased;
  }
  .display{
    font-family:Georgia, 'Times New Roman', serif;
    font-weight:700;
    font-variant:small-caps;
    letter-spacing:.14em;
  }
  .mono{
    font-family:ui-monospace,'SF Mono','Courier New',monospace;
    letter-spacing:.02em;
  }
  #root{max-width:1040px;margin:0 auto;padding:22px 16px 70px;position:relative;}

  /* ---------- masthead / seal ---------- */
  .masthead{text-align:center;margin-bottom:22px;position:relative;}
  .seal{
    width:64px;height:64px;margin:0 auto 8px;
    border:2px solid var(--brass);border-radius:50%;
    display:flex;align-items:center;justify-content:center;
    background:radial-gradient(circle at 35% 30%, #2c2418, #17130d);
  }
  .masthead h1{
    font-size:clamp(30px,6vw,46px);margin:0;color:var(--marble);
    text-shadow:0 1px 0 #000;
  }
  .masthead .tag{
    font-family:ui-monospace,monospace;font-size:11px;letter-spacing:.3em;
    text-transform:uppercase;color:var(--brass);margin-top:4px;
  }
  .masthead:after{
    content:"";display:block;width:120px;height:2px;background:var(--brass);
    margin:12px auto 0;
  }

  /* ---------- colonnade (landing only) ---------- */
  .colonnade{position:relative;padding:0 64px;}
  .column{position:absolute;top:0;bottom:0;width:44px;}
  .column.left{left:0;} .column.right{right:0;}
  @media (max-width:760px){ .colonnade{padding:0;} .column{display:none;} }

  /* ---------- panels ---------- */
  .plaque{
    background:linear-gradient(180deg,#211c15,#191510);
    border:1.5px solid var(--brass-dark);
    border-radius:3px;
    padding:20px;margin-bottom:16px;
    box-shadow:inset 0 0 0 1px rgba(179,146,90,.15), 0 6px 18px rgba(0,0,0,.35);
  }
  h2.section-title{
    font-family:Georgia,serif;font-variant:small-caps;letter-spacing:.08em;
    font-size:19px;margin:0 0 14px;color:var(--brass-bright);
    display:flex;align-items:center;gap:10px;
    border-bottom:1px solid var(--brass-dark);padding-bottom:8px;
  }

  label{display:block;font-family:ui-monospace,monospace;font-size:10px;text-transform:uppercase;
    letter-spacing:.12em;margin-bottom:5px;color:var(--brass);}
  input[type=text],select{
    width:100%;padding:10px 12px;border:1.5px solid var(--brass-dark);border-radius:2px;
    font-size:15px;font-family:Georgia,serif;background:#100d09;color:var(--marble);
  }
  input[type=text]::placeholder{color:#6b6455;}
  .row{display:flex;gap:12px;flex-wrap:wrap;}
  .row > *{flex:1;min-width:160px;}

  button{
    font-family:ui-monospace,monospace;font-weight:700;font-size:12px;letter-spacing:.08em;
    text-transform:uppercase;
    padding:11px 20px;border:1.5px solid var(--brass);border-radius:2px;
    background:linear-gradient(180deg,#2b2418,#1a160f);color:var(--brass-bright);cursor:pointer;
    transition:transform .08s ease, background .15s ease;
  }
  button:hover{background:linear-gradient(180deg,#352b1a,#221b10);}
  button:active{transform:translateY(1px);}
  button.secondary{background:transparent;color:var(--marble-dim);border-color:#4a4438;}
  button.danger{background:linear-gradient(180deg,#a9332a,#7a221c);border-color:#e05a4f;color:#fff2ee;}
  button:disabled{opacity:.3;cursor:not-allowed;transform:none;}

  /* ---------- party standards ---------- */
  .standards-row{display:flex;gap:14px;flex-wrap:wrap;justify-content:center;margin:14px 0 6px;}
  .standard{
    width:96px;text-align:center;cursor:pointer;opacity:.55;
    filter:grayscale(.4);transition:opacity .15s, filter .15s, transform .15s;
  }
  .standard:hover{opacity:.85;transform:translateY(-3px);}
  .standard.selected{opacity:1;filter:none;transform:translateY(-5px);}
  .standard.selected .flag-shape{stroke:var(--brass-bright);stroke-width:3;}
  .standard.taken{opacity:.2;filter:grayscale(1);cursor:not-allowed;pointer-events:none;}
  .standard .flag-shape{stroke:#000;stroke-width:1.5;}
  .standard .pname{
    font-family:ui-monospace,monospace;font-size:10px;letter-spacing:.03em;
    margin-top:4px;color:var(--marble-dim);line-height:1.25;
  }

  /* ---------- players / turn strip ---------- */
  .players-strip{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:4px;}
  .player-chip{
    border:1.5px solid var(--brass-dark);padding:6px 12px;border-radius:2px;font-size:12px;
    display:flex;align-items:center;gap:7px;background:#161209;font-family:ui-monospace,monospace;
    color:var(--marble-dim);
  }
  .player-chip.turn{border-color:var(--crimson-bright);color:#ffd9d3;background:#2a1512;}
  .dot{width:11px;height:11px;border-radius:2px;border:1.5px solid #000;flex:none;}

  /* ---------- system info bar ---------- */
  .info-bar{
    display:grid;grid-template-columns:repeat(auto-fit,minmax(170px,1fr));gap:1px;
    background:var(--brass-dark);border:1.5px solid var(--brass-dark);border-radius:3px;
    overflow:hidden;margin-bottom:16px;
  }
  .info-cell{background:#171310;padding:10px 14px;}
  .info-cell .il{font-family:ui-monospace,monospace;font-size:9px;letter-spacing:.12em;
    text-transform:uppercase;color:var(--brass);margin-bottom:3px;}
  .info-cell .iv{font-size:14px;color:var(--marble);font-weight:700;}

  /* ---------- phase track ---------- */
  .phase-track{display:flex;gap:6px;margin-bottom:16px;}
  .phase-step{
    flex:1;text-align:center;padding:9px 4px;font-size:10px;font-weight:700;
    border:1.5px solid var(--brass-dark);text-transform:uppercase;letter-spacing:.1em;
    background:#171310;color:#7c745f;font-family:ui-monospace,monospace;
  }
  .phase-step.active{background:var(--crimson);border-color:var(--crimson-bright);color:#fff2ee;}

  /* ---------- board ---------- */
  .board-top{display:flex;gap:14px;align-items:stretch;flex-wrap:wrap;margin-bottom:16px;}
  .president-box{
    width:120px;flex:none;border:2px solid var(--brass);border-radius:3px;
    display:flex;flex-direction:column;align-items:center;justify-content:center;gap:6px;
    padding:10px;background:linear-gradient(180deg,#241d10,#171208);
  }
  .president-box .plabel{font-family:ui-monospace,monospace;font-size:9px;letter-spacing:.15em;color:var(--brass);}
  .president-box .pval{font-size:13px;font-weight:700;text-align:center;color:var(--marble);}

  .gov-bar-wrap{flex:1;min-width:200px;border:2px solid var(--brass);border-radius:3px;
    padding:12px;background:linear-gradient(180deg,#211c15,#171208);}
  .gov-bar-wrap .glabel{font-family:ui-monospace,monospace;font-size:9px;letter-spacing:.15em;color:var(--brass);margin-bottom:8px;}
  .gov-bar{display:flex;height:30px;border:1.5px solid #000;border-radius:2px;overflow:hidden;}
  .gov-seg{height:100%;}

  .parliament-wrap{
    border:2px solid var(--brass);border-radius:3px;padding:14px;
    background:linear-gradient(180deg,#211c15,#171208);margin-bottom:14px;text-align:center;
  }
  .parliament-wrap .plabel{font-family:ui-monospace,monospace;font-size:9px;letter-spacing:.15em;color:var(--brass);margin-bottom:2px;}
  .hemicycle-svg{width:100%;max-width:480px;height:auto;}

  .state-wrap{
    border:2px solid var(--brass);border-radius:3px;padding:14px;
    background:linear-gradient(180deg,#1c1911,#141109);margin-bottom:16px;
  }
  .state-wrap .slabel{font-family:ui-monospace,monospace;font-size:9px;letter-spacing:.15em;color:var(--brass);margin-bottom:6px;}
  .state-svg{width:100%;max-width:520px;display:block;margin:0 auto;}
  .state-legend{display:flex;gap:12px;flex-wrap:wrap;justify-content:center;margin-top:10px;}
  .legend-item{display:flex;align-items:center;gap:6px;font-family:ui-monospace,monospace;font-size:11px;color:var(--marble-dim);}
  .legend-dot{width:10px;height:10px;border-radius:50%;border:1px solid #000;}

  /* ---------- legislative agenda ---------- */
  .agenda-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(210px,1fr));gap:12px;}
  .agenda-col{border:1.5px solid var(--brass-dark);border-radius:2px;padding:10px 12px;background:#161209;}
  .agenda-col.mine{border-color:var(--brass);}
  .agenda-col h3{margin:0 0 8px;font-size:11.5px;font-family:ui-monospace,monospace;letter-spacing:.06em;
    color:var(--brass-bright);display:flex;align-items:center;gap:6px;}
  .agenda-item{display:flex;align-items:flex-start;gap:6px;font-size:11.5px;padding:3px 0;color:var(--marble-dim);}
  .agenda-item.done{color:#8fbf8c;text-decoration:line-through;opacity:.75;}
  .agenda-check{flex:none;width:13px;height:13px;border:1.5px solid var(--brass-dark);border-radius:2px;
    display:flex;align-items:center;justify-content:center;font-size:9px;margin-top:1px;color:#dff2df;}
  .agenda-item.done .agenda-check{background:#3a5d3a;border-color:#7fae7c;}

  /* ---------- TCG-style cards ---------- */
  .card-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(150px,1fr));gap:12px;}
  .playcard{
    border-radius:10px;padding:3px;cursor:pointer;position:relative;
    background:linear-gradient(160deg,var(--brass),var(--brass-dark));
  }
  .playcard.disabled{opacity:.35;cursor:not-allowed;}
  .playcard:hover:not(.disabled){transform:translateY(-3px);}
  .playcard-inner{
    border-radius:8px;padding:10px;height:100%;min-height:150px;
    display:flex;flex-direction:column;gap:6px;
  }
  .playcard.agitation .playcard-inner{background:linear-gradient(160deg,#26301f,#141a10);}
  .playcard.discredit .playcard-inner{background:linear-gradient(160deg,#3a1815,#1a0c0a);}
  .cost-gem{
    position:absolute;top:-8px;left:-8px;width:30px;height:30px;border-radius:50%;
    background:radial-gradient(circle at 35% 30%, #4a6fb0, #16233c);
    border:2px solid #7fa0e0;color:#fff;display:flex;align-items:center;justify-content:center;
    font-family:ui-monospace,monospace;font-weight:900;font-size:13px;
  }
  .pc-kind{font-family:ui-monospace,monospace;font-size:9px;letter-spacing:.1em;text-transform:uppercase;
    color:var(--brass-bright);opacity:.85;}
  .pc-title{font-size:13px;font-weight:700;line-height:1.3;flex:1;color:var(--marble);}
  .pc-bottom{display:flex;justify-content:space-between;align-items:center;}
  .value-badge{font-family:ui-monospace,monospace;font-weight:900;font-size:15px;}
  .playcard.agitation .value-badge{color:#8fd18a;}
  .playcard.discredit .value-badge{color:#e07a72;}
  .bonus-dots{display:flex;gap:3px;}
  .bonus-dot{width:9px;height:9px;border-radius:50%;border:1px solid #000;}

  @keyframes cardPlay{
    0%{ transform:translateY(0) scale(1) rotate(0deg); opacity:1; }
    55%{ transform:translateY(-16px) scale(1.06) rotate(-3deg); opacity:1; }
    100%{ transform:translateY(-46px) scale(.68) rotate(9deg); opacity:0; }
  }
  .playcard.card-playing{ animation: cardPlay .38s ease forwards; pointer-events:none; }

  .flash-badge{
    position:fixed; left:50%; top:38%; transform:translate(-50%,0) scale(1);
    font-family:ui-monospace,monospace; font-weight:900; font-size:30px;
    text-shadow:0 2px 8px rgba(0,0,0,.65); pointer-events:none; z-index:999;
    opacity:1; transition:transform 1s ease-out, opacity 1s ease-out;
  }
  .flash-badge.rise{ transform:translate(-50%,-80px) scale(1.35); opacity:0; }

  /* ---------- government formation ---------- */
  .majority-meter{height:11px;background:#241f18;border:1px solid var(--brass-dark);border-radius:6px;overflow:hidden;margin:10px 0 4px;position:relative;}
  .majority-fill{height:100%;background:linear-gradient(90deg,var(--brass-dark),var(--brass-bright));transition:width .3s ease;}
  .majority-mark{position:absolute;top:-3px;bottom:-3px;width:2px;background:var(--crimson-bright);left:50%;}
  .gov-form-row{display:flex;flex-wrap:wrap;gap:8px;margin:10px 0;}
  .gov-chip{
    display:flex;align-items:center;gap:6px;padding:6px 10px;border-radius:14px;
    border:1.5px solid var(--brass-dark);font-size:12px;cursor:pointer;background:#161209;color:var(--marble-dim);
  }
  .gov-chip.locked{opacity:.85;cursor:default;border-color:var(--brass);color:var(--marble);}
  .gov-chip.invited{border-color:var(--brass);background:#241d10;color:var(--marble);}
  .gov-chip.readonly{cursor:default;}

  /* ---------- voting ---------- */
  .vote-panel{border:2px solid var(--brass);border-radius:3px;padding:16px;background:linear-gradient(180deg,#241d10,#171208);}
  .vote-kind{font-family:ui-monospace,monospace;font-size:10px;letter-spacing:.12em;color:var(--brass);text-transform:uppercase;}
  .vote-title{font-size:17px;font-weight:700;margin:4px 0 12px;color:var(--marble);}
  .vote-status-row{display:flex;flex-wrap:wrap;gap:8px;margin-bottom:14px;}
  .vote-status-chip{display:flex;align-items:center;gap:6px;padding:5px 10px;border:1.5px solid var(--brass-dark);
    border-radius:2px;font-size:11px;font-family:ui-monospace,monospace;color:var(--marble-dim);background:#161209;}
  .vote-status-chip.voted{border-color:#7fae7c;color:#cfe8cf;}
  .vote-buttons{display:flex;gap:12px;}
  .vote-buttons button{flex:1;font-size:13px;padding:14px;}
  .vote-yes{background:linear-gradient(180deg,#3a6b3a,#25452a)!important;border-color:#7fae7c!important;color:#eafcea!important;}
  .vote-no{background:linear-gradient(180deg,#8a3128,#5c1f1a)!important;border-color:#e05a4f!important;color:#ffe9e6!important;}

  /* ---------- reveal overlay ---------- */
  .reveal-wrap{max-width:640px;margin:60px auto;text-align:center;}
  .reveal-kicker{font-family:ui-monospace,monospace;font-size:11px;letter-spacing:.2em;color:var(--brass);text-transform:uppercase;}
  .reveal-title{font-size:22px;margin:8px 0 26px;color:var(--marble);}
  .reveal-bar-row{display:flex;align-items:center;gap:10px;margin:12px 0;}
  .reveal-name{width:150px;text-align:right;font-size:12px;flex:none;}
  .reveal-track{flex:1;height:22px;background:#1c1911;border:1.5px solid var(--brass-dark);position:relative;}
  .reveal-fill{height:100%;width:0%;}
  .reveal-count{width:44px;font-family:ui-monospace,monospace;font-size:13px;flex:none;}
  .reveal-verdict{margin-top:26px;font-size:26px;font-weight:900;letter-spacing:.05em;opacity:0;transition:opacity .3s;font-family:ui-monospace,monospace;}
  .reveal-verdict.show{opacity:1;}
  .reveal-verdict.pass{color:#8fd18a;}
  .reveal-verdict.fail{color:#e07a72;}

  .log{
    background:#100d09;border:1.5px solid var(--brass-dark);padding:10px 12px;
    max-height:180px;overflow-y:auto;font-size:12px;line-height:1.6;
    font-family:ui-monospace,monospace;color:var(--marble-dim);
  }
  .log div{border-bottom:1px dashed #3a3326;padding:3px 0;}
  .log div:last-child{border-bottom:none;}

  .waiting{text-align:center;padding:36px 10px;font-size:14px;opacity:.6;font-style:italic;color:var(--marble-dim);}
  .small{font-size:12px;opacity:.75;color:var(--marble-dim);}
  .center{text-align:center;}
  .mt8{margin-top:8px;} .mt16{margin-top:16px;}
  .flex-between{display:flex;justify-content:space-between;align-items:center;gap:10px;flex-wrap:wrap;}
  .law-item{border:1.5px solid var(--brass-dark);padding:10px 12px;margin-bottom:8px;background:#161209;border-radius:2px;}
  .badge{
    display:inline-block;font-family:ui-monospace,monospace;font-size:10px;font-weight:900;padding:3px 8px;
    border:1.5px solid var(--brass);border-radius:10px;text-transform:uppercase;color:var(--brass-bright);
    letter-spacing:.06em;
  }
  .error-plaque{border-color:var(--crimson-bright);color:#ffd9d3;}
</style>

<script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
<script>
/* Politia public multiplayer transport */
window.PolitiaNet = (() => {
  let peer = null, host = false, connections = [];
  const handlers = {};
  function emit(type, data) {
    (handlers[type] || []).forEach(fn => fn(data));
  }
  function on(type, fn) {
    (handlers[type] ||= []).push(fn);
  }
  function send(data) {
    connections.forEach(c => { try { if (c.open) c.send(data); } catch(e) {} });
  }
  function startHost(roomCode) {
    return new Promise((resolve, reject) => {
      host = true;
      peer = new Peer("politia-" + roomCode, { debug: 0 });
      peer.on("open", () => resolve());
      peer.on("error", reject);
      peer.on("connection", c => {
        connections.push(c);
        c.on("data", d => emit("data", {conn:c, data:d}));
        c.on("close", () => { connections = connections.filter(x => x !== c); });
      });
    });
  }
  function join(roomCode) {
    return new Promise((resolve, reject) => {
      host = false;
      peer = new Peer(undefined, { debug: 0 });
      peer.on("open", () => {
        const c = peer.connect("politia-" + roomCode, {reliable:true});
        c.on("open", () => {
          connections = [c];
          c.on("data", d => emit("data", {conn:c, data:d}));
          resolve(c);
        });
        c.on("error", reject);
      });
      peer.on("error", reject);
    });
  }
  return {startHost, join, send, on};
})();
</script>

</head>
<body>
<div id="root"></div>

<script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
<script>
/* ============================== NETWORK (standalone, no Claude account) ============================== */
let netPeer = null;
let netHost = false;
let netConn = null;
let netConnections = [];
let netPending = {};
let netSeq = 0;

function netSend(msg, conn){
  try { (conn || netConn)?.send(msg); } catch(e) {}
}
function netBroadcast(msg){
  netConnections.forEach(c=>{ try{ if(c.open) c.send(msg); }catch(e){} });
}
function netWait(id){
  return new Promise((resolve,reject)=>{
    netPending[id]={resolve,reject};
    setTimeout(()=>{ if(netPending[id]){ delete netPending[id]; reject(new Error('Таймаут соединения')); } }, 8000);
  });
}
function netHandleMessage(msg, conn){
  if(!msg || !msg.type) return;
  if(msg.type==='room-state'){
    const p=netPending[msg.reqId];
    if(p){ delete netPending[msg.reqId]; p.resolve(msg.state); }
    else { room=msg.state; if(view==='landing') view=room.status==='playing'?'game':'lobby'; render(); }
  } else if(msg.type==='room-request' && netHost){
    netSend({type:'room-state', reqId:msg.reqId, state:room}, conn);
  } else if(msg.type==='room-save' && netHost){
    // Host is the authoritative relay. Each write is processed in arrival order.
    room=msg.state;
    netBroadcast({type:'room-state', state:room});
  }
}
function setupConn(conn){
  conn.on('open',()=>{
    if(!netConnections.includes(conn)) netConnections.push(conn);
    if(netHost) netSend({type:'room-state', state:room}, conn);
  });
  conn.on('data',msg=>netHandleMessage(msg,conn));
  conn.on('close',()=>{ netConnections=netConnections.filter(c=>c!==conn); });
}
function initHostNetwork(code){
  return new Promise((resolve,reject)=>{
    netHost=true;
    netPeer=new Peer('politia-'+code.toLowerCase(), {debug:0});
    netPeer.on('open',()=>resolve());
    netPeer.on('connection',conn=>setupConn(conn));
    netPeer.on('error',err=>{
      if(err.type==='unavailable-id') reject(new Error('Код комнаты уже занят. Попробуйте создать комнату ещё раз.'));
      else reject(err);
    });
  });
}
function initClientNetwork(code){
  return new Promise((resolve,reject)=>{
    netHost=false;
    netPeer=new Peer(undefined,{debug:0});
    netPeer.on('open',()=>{
      netConn=netPeer.connect('politia-'+code.toLowerCase(),{reliable:true});
      setupConn(netConn);
      netConn.on('open',()=>resolve());
      netConn.on('error',reject);
    });
    netPeer.on('error',reject);
  });
}
async function loadRoom(code){
  if(netHost) return room;
  if(!netConn || !netConn.open) return null;
  const reqId='q'+(++netSeq);
  netSend({type:'room-request',reqId});
  try{return await netWait(reqId);}catch(e){return null;}
}
async function saveRoom(state){
  room=state;
  if(netHost){
    netBroadcast({type:'room-state',state});
    return;
  }
  if(netConn && netConn.open) netSend({type:'room-save',state});
}

/* ============================== DATA ============================== */

const PARTIES = {
  ч: { name:'Анархисты',        color:'#2b2b2b' },
  к: { name:'Коммунисты',       color:'#a12a24' },
  з: { name:'Социал-демократы', color:'#2f7d46' },
  ж: { name:'Либералы',         color:'#e3b23c' },
  с: { name:'Консерваторы',     color:'#28456f' },
  н: { name:'Националисты',     color:'#7a4a28' },
};
const PARTY_KEYS = Object.keys(PARTIES);

const AGITATION_CARDS = [
  {id:'a1',  name:'Обещания введения прогрессивных налогов', cost:0, value:1, bonus:['к','ч']},
  {id:'a2',  name:'Обещания поддержки бизнеса',               cost:0, value:1, bonus:['с','н','ж']},
  {id:'a3',  name:'Обещания поддержки семьи',                 cost:0, value:1, bonus:['н','с']},
  {id:'a4',  name:'Обещания поддержки рабочих',                cost:0, value:1, bonus:['ч','к']},
  {id:'a5',  name:'Митинг в малом населенном пункте',          cost:1, value:1, bonus:['к','с','н']},
  {id:'a6',  name:'Крупный митинг в городе',                   cost:2, value:2, bonus:['ч','н','з','ж']},
  {id:'a7',  name:'Статья в профильном журнале',               cost:1, value:1, bonus:['з','ж','с']},
  {id:'a8',  name:'Выступление на радио',                      cost:1, value:1, bonus:['к','с']},
  {id:'a9',  name:'Эфир на телевидении',                       cost:2, value:3, bonus:[]},
  {id:'a10', name:'Реклама в прайм-тайм',                      cost:3, value:5, bonus:[]},
  {id:'a11', name:'Запустить тренд в Тик-Токе',                cost:1, value:1, bonus:['ч','н','з','ж']},
  {id:'a12', name:'Поквартирный обход',                        cost:1, value:1, bonus:['к','с']},
  {id:'a13', name:'Раздача продовольственных пакетов',         cost:2, value:2, bonus:['з','к','ч']},
  {id:'a14', name:'Рекламные листовки',                        cost:1, value:1, bonus:[]},
  {id:'a15', name:'Рекламные билборды',                        cost:2, value:2, bonus:[]},
  {id:'a16', name:'Раздача газет в людных местах',             cost:1, value:1, bonus:['к','с']},
  {id:'a17', name:'Публичный перформанс',                      cost:1, value:1, bonus:['з','ж','ч','н']},
  {id:'a18', name:'Организация концерта',                      cost:2, value:2, bonus:['ж','ч']},
  {id:'a19', name:'Подкуп избирателей',                        cost:1, value:3, bonus:[]},
  {id:'a20', name:'Телефонный обзвон',                         cost:1, value:1, bonus:['к','с']},
  {id:'a21', name:'Автопробег',                                cost:2, value:2, bonus:['с','н']},
  {id:'a22', name:'Субботники',                                cost:1, value:1, bonus:['з','к','ч']},
  {id:'a23', name:'Закупка ботов в интернете',                 cost:1, value:3, bonus:[]},
  {id:'a24', name:'Организация шествия',                       cost:1, value:2, bonus:['к','с','н']},
  {id:'a25', name:'Закрытая встреча со спонсорами',            cost:0, value:2, bonus:[]},
  {id:'a26', name:'Получение иностранного финансирования',     cost:0, value:4, bonus:[]},
  {id:'a27', name:'Таргетированная реклама в соцсетях',        cost:1, value:1, bonus:['з','ж']},
  {id:'a28', name:'Политическая голодовка',                    cost:1, value:1, bonus:['ч','н']},
  {id:'a29', name:'Инсценировка покушения',                    cost:1, value:3, bonus:[]},
  {id:'a30', name:'Коррупция',                                 cost:0, value:3, bonus:[]},
  {id:'a31', name:'Культ личности',                            cost:1, value:3, bonus:['к','ж','с','н']},
  {id:'a32', name:'Благотворительный концерт для детей',       cost:1, value:2, bonus:[]},
  {id:'a33', name:'Финансирование студенческих организаций',   cost:1, value:2, bonus:['з','ж']},
  {id:'a34', name:'Семейная ярмарка с воздушными шарами',      cost:1, value:2, bonus:[]},
];

const DISCREDIT_CARDS = [
  {id:'d1', name:'Публикация компромата',           cost:1, value:1},
  {id:'d2', name:'Распространение негативных слухов', cost:0, value:1},
  {id:'d3', name:'Плагиат в диссертации лидеров',    cost:0, value:1, requiresPartyIn:['ж','з']},
  {id:'d4', name:'Журналистское расследование',      cost:1, value:2},
  {id:'d5', name:'Пьяное интервью',                  cost:1, value:1, requiresPartyIn:['к','с']},
  {id:'d6', name:'Обвинения в коррупции',            cost:1, value:4, requiresCard:'Коррупция'},
  {id:'d7', name:'Обвинения в иностранном влиянии',  cost:1, value:4, requiresCard:'Получение иностранного финансирования'},
  {id:'d8', name:'Порча подъездов агитацией',        cost:1, value:2, requiresPartyIn:['ч','н']},
  {id:'d9', name:'Отправка провокаторов к отделению', cost:1, value:2},
  {id:'d10',name:'Разоблачение культа личности',     cost:1, value:3, requiresCard:'Культ личности'},
  {id:'d11',name:'Съемки фильма «Колыма»',           cost:1, value:1, requiresPartyIn:['з','к','ч']},
  {id:'d12',name:'Компания очернения по ТВ',         cost:2, value:4},
];

const LAWS_BY_PARTY = {
  ч: ['Закон о свободных ассоциациях граждан','Закон о свободе слова','Закон об обязательных референдумах','Закон о прозрачности органов власти','Закон о развитии местного самоуправления'],
  к: ['Закон о национализации банковского сектора','Трудовой кодекс','Закон о государственной промышленной политике','Закон о государственном жилищном строительстве','Закон о стратегических государственных инвестициях'],
  з: ['Закон о социальном государстве','Закон о свободе слова','Трудовой кодекс','Закон о государственном жилищном строительстве','Антикоррупционный закон'],
  ж: ['Закон о неприкосновенности жилища','Закон о свободе слова','Закон о прозрачности органов власти','Антикоррупционный закон','Закон о стратегических государственных инвестициях'],
  с: ['Закон о защите семьи и традиционных ценностей','Закон о прозрачности органов власти','Закон о государственной промышленной политике','Закон о государственном жилищном строительстве','Антикоррупционный закон'],
  н: ['Закон о приоритете найма граждан на работу','Закон об обязательных референдумах','Закон о развитии местного самоуправления','Закон о государственной промышленной политике','Закон о стратегических государственных инвестициях'],
};

const CONSTITUTIONAL_LAWS = [
  { name:'Закон о повышении избирательного барьера', kind:'raiseThreshold' },
  { name:'Закон об урезании финансирования партий', kind:'cutFunding' },
];

const REFORMS = [
  {name:'Административная реформа', value:1, bonus:['ж']},
  {name:'Национальная программа строительства дорог', value:1, bonus:['с']},
  {name:'Цифровизация государственных услуг', value:1, bonus:['ж']},
  {name:'Реформа школьного образования', value:1, bonus:['з']},
  {name:'Реформа здравоохранения', value:1, bonus:['з']},
  {name:'Государственная промышленная программа', value:1, bonus:['к','н']},
  {name:'Энергетическая модернизация', value:1, bonus:['ж','н']},
  {name:'Жилищная программа', value:1, bonus:['к','с']},
  {name:'Поддержка малого бизнеса', value:1, bonus:['ж']},
  {name:'Поддержка сельских территорий', value:1, bonus:['с','н']},
  {name:'Развитие общественного транспорта', value:1, bonus:['з']},
  {name:'Судебная цифровизация', value:1, bonus:['ж','с']},
  {name:'Развитие местного самоуправления', value:1, bonus:['ч']},
  {name:'Государственная программа занятости', value:1, bonus:['к','з']},
  {name:'Национальная инфраструктурная программа', value:2, bonus:PARTY_KEYS.slice()},
];

const TOTAL_SEATS = 450;
const START_BUDGET = 5;

const SCENARIOS = {
  parliamentary: { label:'Парламентская модель', minPlayers:2, maxPlayers:6, formula:'dhondt',
    formulaLabel:"Метод Д'Ондта (пропорциональная)", systemLabel:'Парламентская республика', threshold:5 },
  american:      { label:'Американская модель (2 игрока)', minPlayers:2, maxPlayers:2, formula:'plurality',
    formulaLabel:'Мажоритарная (относительное большинство)', systemLabel:'Президентская республика', threshold:0 },
  russian:       { label:'Российская модель', minPlayers:2, maxPlayers:6, formula:'dhondt',
    formulaLabel:"Метод Д'Ондта (пропорциональная)", systemLabel:'Президентско-парламентская республика', threshold:5 },
};

/* ============================== ICONS & HERALDRY ============================== */

function partyIcon(key){
  switch(key){
    case 'ч':
      return `<circle cx="20" cy="20" r="13" fill="none" stroke="currentColor" stroke-width="2.4"/>
              <path d="M20 10 L28 29 M20 10 L12 29 M14.5 23 L25.5 23" stroke="currentColor" stroke-width="2.4" fill="none" stroke-linecap="round" stroke-linejoin="round"/>`;
    case 'к':
      return `<polygon points="20,7 23.5,16.5 33.5,16.9 25.5,23 28.3,32.6 20,27 11.7,32.6 14.5,23 6.5,16.9 16.5,16.5" fill="currentColor"/>`;
    case 'з':
      return `<circle cx="20" cy="16" r="5.5" fill="currentColor"/>
              <circle cx="14" cy="19" r="4" fill="currentColor" opacity=".85"/>
              <circle cx="26" cy="19" r="4" fill="currentColor" opacity=".85"/>
              <circle cx="20" cy="22" r="4" fill="currentColor" opacity=".85"/>
              <line x1="20" y1="24" x2="20" y2="34" stroke="currentColor" stroke-width="2.2"/>`;
    case 'ж':
      return `<line x1="14" y1="8" x2="14" y2="33" stroke="currentColor" stroke-width="2.2"/>
              <path d="M14 10 L30 15 L14 20 Z" fill="currentColor"/>`;
    case 'с':
      return `<path d="M20 8 C14 14 15 19 20 22 C25 19 26 14 20 8 Z" fill="currentColor"/>
              <rect x="17" y="22" width="6" height="12" fill="currentColor" opacity=".85"/>`;
    case 'н':
      return `<path d="M13 18 C13 14 16 12 20 12 C24 12 27 14 27 18 L27 24 C27 29 24 32 20 32 C16 32 13 29 13 24 Z" fill="currentColor"/>
              <rect x="17" y="30" width="6" height="5" fill="currentColor" opacity=".85"/>`;
    default: return '';
  }
}

function standardSVG(key, size){
  const c = PARTIES[key].color;
  return `<svg viewBox="0 0 44 96" width="${size}" height="${size*96/44}">
    <path class="flag-shape" d="M4,4 L40,4 L40,68 L22,84 L4,68 Z" fill="${c}"/>
    <line x1="4" y1="4" x2="4" y2="92" stroke="#4a3f2a" stroke-width="3"/>
    <g transform="translate(2,6)" color="#fff">${partyIcon(key)}</g>
  </svg>`;
}

/* ---------- hemicycle (parliament fan) ---------- */
function hemicyclePath(cx,cy,rOuter,rInner,startDeg,endDeg){
  const rad = d => d*Math.PI/180;
  const pt = (r,deg) => [ (cx + r*Math.cos(rad(deg))).toFixed(2), (cy - r*Math.sin(rad(deg))).toFixed(2) ];
  const [x1,y1]=pt(rOuter,startDeg), [x2,y2]=pt(rOuter,endDeg);
  const [x3,y3]=pt(rInner,endDeg), [x4,y4]=pt(rInner,startDeg);
  const large = Math.abs(startDeg-endDeg) > 180 ? 1 : 0;
  return `M ${x1} ${y1} A ${rOuter} ${rOuter} 0 ${large} 1 ${x2} ${y2} L ${x3} ${y3} A ${rInner} ${rInner} 0 ${large} 0 ${x4} ${y4} Z`;
}
function renderHemicycle(seats){
  const entries = PARTY_KEYS.filter(k=>(seats[k]||0)>0).map(k=>({k,v:seats[k]}));
  const total = entries.reduce((s,e)=>s+e.v,0);
  if(!total){
    return `<svg viewBox="0 0 400 210" class="hemicycle-svg">
      <path d="${hemicyclePath(200,195,180,90,180,0)}" fill="#241f18" stroke="#4a3f2a" stroke-width="2"/>
      <text x="200" y="150" text-anchor="middle" fill="#6b6455" font-family="ui-monospace,monospace" font-size="13">нет данных</text>
    </svg>`;
  }
  let cum=0;
  const segs = entries.map(e=>{
    const startDeg = 180 - (cum/total)*180;
    cum+=e.v;
    const endDeg = 180 - (cum/total)*180;
    return `<path d="${hemicyclePath(200,195,180,90,startDeg,endDeg)}" fill="${PARTIES[e.k].color}" stroke="#0e0c09" stroke-width="2"/>`;
  }).join('');
  return `<svg viewBox="0 0 400 210" class="hemicycle-svg">${segs}
    <text x="200" y="205" text-anchor="middle" fill="var(--brass)" font-family="ui-monospace,monospace" font-size="11">${total} мандатов</text>
  </svg>`;
}

/* ---------- state population blob ---------- */
const BLOB_PATH = "M55,35 C25,45 10,90 25,130 C8,165 30,205 75,220 C110,245 175,240 210,215 C260,230 300,185 285,145 C310,105 285,60 235,50 C215,15 145,5 105,25 C85,10 60,15 55,35 Z";
const TOTAL_POP = 60;
const NEUTRAL_COLOR = '#8a8272';
function seedRand(seed){ const x=Math.sin(seed*127.1)*43758.5453; return x - Math.floor(x); }
function personFigure(x,y,scale,rot,color){
  return `<g transform="translate(${x.toFixed(1)},${y.toFixed(1)}) rotate(${rot.toFixed(1)}) scale(${scale.toFixed(2)})" stroke="#0e0c09" stroke-width="0.5" stroke-linecap="round">
    <circle cx="0" cy="-6.4" r="2.7" fill="${color}"/>
    <path d="M-3.4,7.4 C-3.7,0.6 -3,-2 0,-2 C3,-2 3.7,0.6 3.4,7.4 Z" fill="${color}"/>
    <path d="M-3.1,-0.2 L-5.3,4.4 M3.1,-0.2 L5.3,4.4" stroke="${color}" stroke-width="1.3" fill="none"/>
  </g>`;
}
function allocateSlots(support, totalSlots){
  const parties = PARTY_KEYS.filter(k=>(support[k]||0)>0);
  const totalSupport = parties.reduce((s,k)=>s+support[k],0);
  const alloc = {};
  if(totalSupport<=0 || totalSlots<=0) return alloc;
  const raw = parties.map(k=>({k, exact: support[k]/totalSupport*totalSlots}));
  let used=0;
  raw.forEach(x=>{ alloc[x.k]=Math.floor(x.exact); used+=alloc[x.k]; });
  let remainder = totalSlots-used;
  raw.sort((a,b)=>(b.exact-Math.floor(b.exact))-(a.exact-Math.floor(a.exact)));
  for(let i=0;i<remainder && raw.length;i++){ alloc[raw[i%raw.length].k]++; }
  return alloc;
}
function renderStateBlob(support){
  const totalSupport = PARTY_KEYS.reduce((s,k)=>s+(support[k]||0),0);
  const engaged = Math.min(TOTAL_POP, Math.round(TOTAL_POP*totalSupport/(totalSupport+40)));
  const alloc = allocateSlots(support, engaged);
  const owners=[];
  PARTY_KEYS.forEach(k=>{ for(let i=0;i<(alloc[k]||0);i++) owners.push(k); });
  let people='';
  for(let i=0;i<TOTAL_POP;i++){
    const s1=i*13.7, s2=i*29.3;
    const x = 25 + seedRand(s1)*270;
    const y = 25 + seedRand(s2)*210;
    const scale = 1.1 + seedRand(s1+s2)*0.9;
    const rot = -18 + seedRand(s1*3+s2)*36;
    const color = i<owners.length ? PARTIES[owners[i]].color : NEUTRAL_COLOR;
    people += personFigure(x,y,scale,rot,color);
  }
  return `<svg viewBox="0 0 320 260" class="state-svg">
    <defs><clipPath id="blobClip"><path d="${BLOB_PATH}"/></clipPath></defs>
    <path d="${BLOB_PATH}" fill="#e9e1cc" stroke="var(--brass)" stroke-width="2.5"/>
    <g clip-path="url(#blobClip)">${people}</g>
    <path d="${BLOB_PATH}" fill="none" stroke="var(--brass)" stroke-width="2.5"/>
  </svg>`;
}

/* ---------- card dependencies ---------- */
function cardMeetsPrereq(card, targetParty, r){
  r = r || room;
  if(card.requiresCard){ return (r.cardHistory[targetParty]||[]).includes(card.requiresCard); }
  if(card.requiresPartyIn){ return card.requiresPartyIn.includes(targetParty); }
  return true;
}
function validTargets(card, actingParty){
  return room.players.filter(p=>p.party!==actingParty && cardMeetsPrereq(card, p.party));
}

/* ============================== UTIL ============================== */

function uid(){ return Math.random().toString(36).slice(2,10); }
function sleep(ms){ return new Promise(res=>setTimeout(res,ms)); }
function pickRandom(arr,n){
  const pool=[...arr]; const out=[];
  for(let i=0;i<n && pool.length;i++){
    out.push(pool.splice(Math.floor(Math.random()*pool.length),1)[0]);
  }
  return out;
}
function dealHand(){
  const ag = pickRandom(AGITATION_CARDS,6).map(c=>({...c,type:'agitation'}));
  const di = pickRandom(DISCREDIT_CARDS,4).map(c=>({...c,type:'discredit'}));
  const merged=[...ag,...di];
  for(let i=merged.length-1;i>0;i--){ const j=Math.floor(Math.random()*(i+1)); [merged[i],merged[j]]=[merged[j],merged[i]]; }
  return merged.map(c=>({...c, handId:uid()}));
}

function allocateSeats(votes, totalSeats, method, thresholdPct){
  thresholdPct = thresholdPct || 0;
  const totalVotes = Object.values(votes).reduce((a,b)=>a+b,0) || 1;
  let eligible = {};
  Object.entries(votes).forEach(([k,v])=>{ if((v/totalVotes*100) >= thresholdPct) eligible[k]=v; });
  if(Object.keys(eligible).length===0){
    const top = Object.entries(votes).sort((a,b)=>b[1]-a[1])[0];
    if(top) eligible[top[0]]=top[1];
  }
  const keys = Object.keys(eligible).filter(k=>eligible[k]>0);
  const alloc = {}; keys.forEach(k=>alloc[k]=0);
  if(keys.length===0) return alloc;
  votes = eligible;
  if(method==='plurality'){
    const sorted=[...keys].sort((a,b)=>votes[b]-votes[a]);
    const winner=sorted[0];
    alloc[winner]=Math.round(totalSeats*0.55);
    let remaining=totalSeats-alloc[winner];
    const rest=sorted.slice(1);
    const restTotal=rest.reduce((s,k)=>s+votes[k],0)||1;
    rest.forEach((k,i)=>{
      alloc[k]= i===rest.length-1 ? remaining : Math.round(remaining*votes[k]/restTotal);
      remaining-=alloc[k];
    });
    return alloc;
  }
  const divisor = method==='sainte-lague' ? (n)=>(2*n-1) : (n)=>n;
  const quots=[];
  keys.forEach(k=>{ for(let n=1;n<=totalSeats;n++) quots.push({k,q:votes[k]/divisor(n)}); });
  quots.sort((x,y)=>y.q-x.q);
  for(let i=0;i<totalSeats && i<quots.length;i++) alloc[quots[i].k]++;
  return alloc;
}

/* ============================== STORAGE ============================== */

/* ============================== APP STATE ============================== */

let me = { id: uid(), name:'', roomCode:'' };
let room = null;
let view = 'landing';
let uiError = '';
let pollTimer = null;
let animatingReveal = false;
let lastAnimatedElectionTs = null;
let lastAnimatedResolutionTs = null;
let discreditPicker = null;

function render(){
  const root=document.getElementById('root');
  root.innerHTML = `
    <div class="masthead">
      <div class="seal"><svg width="34" height="34" viewBox="0 0 40 40" color="var(--brass)"><circle cx="20" cy="20" r="15" fill="none" stroke="currentColor" stroke-width="2"/><circle cx="20" cy="20" r="4" fill="currentColor"/></svg></div>
      <h1 class="display">Полития</h1>
      <div class="tag">Государство «Люмен»</div>
    </div>
    ${uiError ? `<div class="plaque error-plaque">${uiError}</div>` : ''}
    ${view==='landing' ? `<div class="colonnade">${renderLanding()}</div>` : ''}
    ${view==='lobby' ? renderLobby() : ''}
    ${view==='game' ? renderGame() : ''}
  `;
  attachHandlers();
}

/* ---------- LANDING ---------- */
function renderLanding(){
  return `
    <div class="plaque">
      <h2 class="section-title">Создать комнату</h2>
      <div class="row">
        <div><label>Ваше имя</label><input type="text" id="hostName" placeholder="Например, Виктор"></div>
        <div><label>Сценарий</label>
          <select id="scenarioSel">${Object.entries(SCENARIOS).map(([k,s])=>`<option value="${k}">${s.label}</option>`).join('')}</select>
        </div>
      </div>
      <button id="createBtn" class="mt16">Создать комнату</button>
    </div>
    <div class="plaque">
      <h2 class="section-title">Присоединиться</h2>
      <div class="row">
        <div><label>Ваше имя</label><input type="text" id="joinName" placeholder="Ваше имя"></div>
        <div><label>Код комнаты</label><input type="text" id="joinCode" placeholder="Например, XK4P"></div>
      </div>
      <button id="joinBtn" class="mt16">Войти в комнату</button>
    </div>
    <p class="small center">Держите вкладку открытой — состояние партии обновляется каждые пару секунд.</p>
  `;
}

/* ---------- LOBBY ---------- */
function renderLobby(){
  if(!room) return '<div class="waiting">Загрузка комнаты…</div>';
  const takenParties = room.players.map(p=>p.party);
  const scenario = SCENARIOS[room.scenario];
  const iAmIn = room.players.some(p=>p.id===me.id);
  const myParty = room.players.find(p=>p.id===me.id)?.party;
  return `
    <div class="plaque">
      <div class="flex-between">
        <h2 class="section-title" style="border:none;margin:0;padding:0;">Лобби — код ${room.code}</h2>
        <span class="badge">${scenario.label}</span>
      </div>
      <p class="small">Отправьте код <b style="color:var(--brass-bright)">${room.code}</b> остальным игрокам, чтобы они присоединились.</p>
      <div class="players-strip mt8">
        ${room.players.map(p=>`
          <div class="player-chip">
            ${p.party?`<span class="dot" style="background:${PARTIES[p.party].color}"></span>`:''}
            ${p.name}${p.party?` — ${PARTIES[p.party].name}`:' (выбирает штандарт)'}
          </div>
        `).join('') || '<span class="small">Пока никого нет</span>'}
      </div>
      ${iAmIn ? `
        <h2 class="section-title mt16">Выберите штандарт партии</h2>
        <div class="standards-row">
          ${PARTY_KEYS.map(k=>{
            const takenBy = takenParties.includes(k) && myParty!==k;
            const mine = myParty===k;
            return `<div class="standard ${takenBy?'taken':''} ${mine?'selected':''}" data-party="${k}">
              ${standardSVG(k,72)}
              <div class="pname">${PARTIES[k].name}</div>
            </div>`;
          }).join('')}
        </div>
      ` : ''}
      <div class="mt16 flex-between">
        <span class="small">Игроков: ${room.players.length} / нужно минимум ${scenario.minPlayers}</span>
        ${room.hostId===me.id ? `<button id="startBtn" ${room.players.length<scenario.minPlayers || room.players.some(p=>!p.party) ? 'disabled':''}>Начать игру</button>` : `<span class="small">Ждём хоста…</span>`}
      </div>
    </div>
  `;
}

function renderSystemInfo(){
  const s = SCENARIOS[room.scenario];
  return `
    <div class="info-bar">
      <div class="info-cell"><div class="il">Тип системы</div><div class="iv">${s.systemLabel}</div></div>
      <div class="info-cell"><div class="il">Электоральная формула</div><div class="iv">${s.formulaLabel}</div></div>
      <div class="info-cell"><div class="il">Заградительный барьер</div><div class="iv">${room.electoralThreshold}% голосов</div></div>
      <div class="info-cell"><div class="il">Выплата партиям за цикл</div><div class="iv">₽${room.budgetPerCycle} (+ остаток прошлых циклов)</div></div>
    </div>
  `;
}

function renderAgendaPanel(){
  const passed = room.allPassedLaws||[];
  const cols = room.players.map(p=>{
    const list = LAWS_BY_PARTY[p.party]||[];
    const mine = p.id===me.id;
    return `<div class="agenda-col ${mine?'mine':''}">
      <h3><span class="dot" style="background:${PARTIES[p.party].color}"></span>${mine?'Ваша повестка':p.name+' · '+PARTIES[p.party].name}</h3>
      ${list.map(title=>{
        const done = passed.includes(title);
        return `<div class="agenda-item ${done?'done':''}"><span class="agenda-check">${done?'✓':''}</span><span>${title}</span></div>`;
      }).join('')}
    </div>`;
  }).join('');
  return `<div class="plaque"><h2 class="section-title">Законодательная повестка</h2><div class="agenda-grid">${cols}</div></div>`;
}

/* ---------- GAME ---------- */
function renderGame(){
  if(!room) return '<div class="waiting">Загрузка…</div>';
  const meP = room.players.find(p=>p.id===me.id);
  const phaseNames = ['','Агитация','Выборы','Законы и реформы'];
  const er = room.electionResult;
  const president = er?.president;
  const gov = er?.government || [];
  const seatsForBar = er?.seats || {};
  const govTotal = Object.values(seatsForBar).reduce((a,b)=>a+b,0) || 1;
  return `
    ${renderSystemInfo()}
    <div class="phase-track">
      ${[1,2,3].map(p=>`<div class="phase-step ${room.phase===p?'active':''}">Фаза ${p} · ${phaseNames[p]}</div>`).join('')}
    </div>
    <div class="plaque">
      <div class="flex-between">
        <h2 class="section-title" style="border:none;margin:0;padding:0;">Цикл ${room.cycle}</h2>
        <span class="badge">${SCENARIOS[room.scenario].label}</span>
      </div>
      <div class="players-strip mt8">
        ${room.players.map(p=>`
          <div class="player-chip ${room.phase===1 && room.turnOrder[room.turnIndex]===p.id ? 'turn':''}">
            <span class="dot" style="background:${PARTIES[p.party].color}"></span>
            ${p.name} <span class="mono">₽${room.budgets[p.id]}</span>
          </div>
        `).join('')}
      </div>
    </div>

    ${renderAgendaPanel()}

    <div class="board-top">
      <div class="president-box">
        <div class="plabel">Президент</div>
        ${president ? `${standardSVG(president,44)}<div class="pval">${PARTIES[president].name}</div>` : `<div class="pval small">— пока нет —</div>`}
      </div>
      <div class="gov-bar-wrap">
        <div class="glabel">Правительство</div>
        <div class="gov-bar">
          ${gov.length ? gov.map(k=>`<div class="gov-seg" style="width:${Math.max(seatsForBar[k]||0,1)/govTotal*100}%;background:${PARTIES[k].color}" title="${PARTIES[k].name}"></div>`).join('') : `<div class="gov-seg" style="width:100%;background:#241f18;"></div>`}
        </div>
      </div>
    </div>

    <div class="parliament-wrap">
      <div class="plabel">Парламент</div>
      ${renderHemicycle(seatsForBar)}
    </div>

    <div class="state-wrap">
      <div class="slabel">Государство — настроения электората</div>
      ${renderStateBlob(room.phase===1 ? room.support : seatsForBar)}
      <div class="state-legend">
        ${room.players.map(p=>`<div class="legend-item"><span class="legend-dot" style="background:${PARTIES[p.party].color}"></span>${PARTIES[p.party].name}</div>`).join('')}
        <div class="legend-item"><span class="legend-dot" style="background:#8a8272"></span>Не определились</div>
      </div>
    </div>

    ${room.phase===1 ? renderPhase1(meP) : ''}
    ${room.phase===2 ? renderPhase2(meP) : ''}
    ${room.phase===3 ? renderPhase3(meP) : ''}
    <div class="plaque">
      <h2 class="section-title">Журнал событий</h2>
      <div class="log">${room.log.slice().reverse().map(l=>`<div>${l}</div>`).join('') || '<div class="small">Пока тихо</div>'}</div>
    </div>
  `;
}

function renderPhase1(meP){
  const isMyTurn = room.turnOrder[room.turnIndex]===me.id;
  const hasPassed = room.passed.includes(me.id);
  const hand = room.hands[me.id] || [];

  if(discreditPicker && meP){
    const {card} = discreditPicker;
    const targets = validTargets(card, meP.party);
    return `<div class="plaque">
      <h2 class="section-title">Против кого сыграть «${card.name}»?</h2>
      <div class="standards-row">
        ${targets.map(p=>`<div class="standard selected" data-target="${p.party}">
          ${standardSVG(p.party,72)}<div class="pname">${p.name}<br>${PARTIES[p.party].name}</div>
        </div>`).join('')}
      </div>
      <button id="cancelDiscreditBtn" class="secondary mt16">Отмена</button>
    </div>`;
  }

  return `
    <div class="plaque">
      <h2 class="section-title">Ваша рука ${isMyTurn && !hasPassed ? '— ваш ход' : ''}</h2>
      ${!meP ? `<p class="small">Вы наблюдатель в этой партии.</p>` :
        hasPassed ? `<p class="small">Вы закончили ход в этом цикле.</p>` :
        !isMyTurn ? `<p class="small">Ждём хода: ${room.players.find(p=>p.id===room.turnOrder[room.turnIndex])?.name}</p>` :
        `<p class="small">Разыгрывайте карты, пока хватает бюджета — можно сыграть подряд несколько. Когда закончите, завершите ход.</p>
        <div class="card-grid">
          ${hand.map(c=>{
            const affordable = room.budgets[me.id] >= c.cost;
            const hasTarget = c.type==='discredit' ? validTargets(c, meP.party).length>0 : true;
            const playable = affordable && hasTarget;
            return `<div class="playcard ${c.type} ${playable?'':'disabled'}" data-play="${c.handId}">
              <div class="cost-gem">${c.cost}</div>
              <div class="playcard-inner">
                <div class="pc-kind">${c.type==='agitation'?'Агитация':'Дискредитация'}</div>
                <div class="pc-title">${c.name}</div>
                ${c.type==='discredit' && !hasTarget ? `<div class="small" style="color:#e07a72;">нет подходящей цели</div>` : ''}
                <div class="pc-bottom">
                  <span class="value-badge">${c.type==='agitation'?'+':'−'}${c.value}</span>
                  <span class="bonus-dots">${(c.bonus||[]).map(b=>`<span class="bonus-dot" style="background:${PARTIES[b].color}" title="${PARTIES[b].name}"></span>`).join('')}</span>
                </div>
              </div>
            </div>`;
          }).join('')}
        </div>
        <button id="passBtn" class="mt16">Закончить ход</button>`
      }
    </div>
  `;
}

function renderPhase2(meP){
  const er = room.electionResult;
  if(!er) return `<div class="plaque"><div class="waiting">Считаем голоса…</div></div>`;
  const totalSeats = Object.values(er.seats).reduce((a,b)=>a+b,0)||1;
  const invited = er.invited && er.invited.length ? er.invited : [er.president];
  const invitedSeats = invited.reduce((s,k)=>s+(er.seats[k]||0),0);
  const isPresidentPlayer = meP && meP.party===er.president;

  if(er.government){
    return `<div class="plaque">
      <h2 class="section-title">Правительство сформировано</h2>
      <p class="small">В составе: ${er.government.map(k=>PARTIES[k].name).join(', ')}</p>
      ${room.hostId===me.id ? `<button id="toPhase3Btn" class="mt16">К законодательному процессу →</button>` : `<p class="small">Ждём хоста…</p>`}
    </div>`;
  }
  return `<div class="plaque">
    <h2 class="section-title">Формирование правительства</h2>
    <p class="small">Президентская партия — <b style="color:${PARTIES[er.president].color}">${PARTIES[er.president].name}</b>. ${isPresidentPlayer? 'Пригласите партнёров по коалиции — можно включить в правительство и проигравших:' : 'Ждём, пока президентская партия соберёт коалицию.'}</p>
    <div class="majority-meter"><div class="majority-fill" style="width:${Math.min(100,invitedSeats/totalSeats*100)}%"></div><div class="majority-mark"></div></div>
    <p class="small">${invitedSeats} из ${totalSeats} мандатов приглашено в правительство</p>
    <div class="gov-form-row">
      <div class="gov-chip locked"><span class="dot" style="background:${PARTIES[er.president].color}"></span>${PARTIES[er.president].name} (${er.seats[er.president]||0}) · президент</div>
      ${PARTY_KEYS.filter(k=>k!==er.president && (er.seats[k]||0)>0).map(k=>{
        const isIn = invited.includes(k);
        return `<div class="gov-chip ${isIn?'invited':''} ${isPresidentPlayer?'':'readonly'}" ${isPresidentPlayer?`data-invite="${k}"`:''}>
          <span class="dot" style="background:${PARTIES[k].color}"></span>${PARTIES[k].name} (${er.seats[k]})
        </div>`;
      }).join('')}
    </div>
    ${isPresidentPlayer ? `<button id="confirmGovBtn" class="mt16">Сформировать правительство</button>` : ''}
  </div>`;
}

function renderVotingPanel(pending, seats){
  const meP = room.players.find(p=>p.id===me.id);
  const seatedPlayers = room.players.filter(p=>(seats[p.party]||0)>0);
  const myVote = meP ? pending.votes[meP.id] : undefined;
  const iCanVote = meP && (seats[meP.party]||0)>0 && myVote===undefined;
  return `
    <div class="vote-panel">
      <div class="vote-kind">${pending.kind==='law'?'Законопроект':'Реформа'} · внесла ${PARTIES[pending.by].name}</div>
      <div class="vote-title">${pending.title}</div>
      <div class="vote-status-row">
        ${seatedPlayers.map(p=>{
          const voted = pending.votes[p.id]!==undefined;
          return `<div class="vote-status-chip ${voted?'voted':''}"><span class="dot" style="background:${PARTIES[p.party].color}"></span>${p.name}${voted?' ✓':' …'}</div>`;
        }).join('')}
      </div>
      ${iCanVote ? `
        <div class="vote-buttons">
          <button class="vote-yes" data-vote="yes">За</button>
          <button class="vote-no" data-vote="no">Против</button>
        </div>
      ` : myVote!==undefined ? `<p class="small">Вы проголосовали. Ждём остальных…</p>` : `<p class="small">Ваша партия не представлена в парламенте — голос не учитывается.</p>`}
    </div>
  `;
}

function renderPhase3(meP){
  const er = room.electionResult;
  const seats = er.seats;
  const item = room.phase3Queue[room.phase3QueueIndex] || null;
  const pending = room.pendingAction;
  let body;
  if(pending){
    body = renderVotingPanel(pending, seats);
  } else if(!item){
    body = `<p class="small">Повестка цикла исчерпана.</p>
      ${room.hostId===me.id ? `<button id="endCycleBtn" class="danger mt16">Завершить цикл → к фазе 1</button>` : `<p class="small">Ждём хоста…</p>`}`;
  } else {
    const actingPlayer = room.players.find(p=>p.party===item.party);
    const isMe = actingPlayer && actingPlayer.id===me.id;
    if(!isMe){
      body = `<p class="small">Ход: ${actingPlayer?actingPlayer.name:'—'} (${PARTIES[item.party].name}) — ${item.kind==='law'?'вносит законопроект':'предлагает реформу'}</p>`;
    } else if(item.kind==='law'){
      const avail = LAWS_BY_PARTY[item.party].filter(l=>!room.allPassedLaws.includes(l) && !room.proposedThisCycle.includes(l));
      const availConst = CONSTITUTIONAL_LAWS.filter(cl=>!room.proposedThisCycle.includes(cl.name));
      body = (avail.length===0 && availConst.length===0)
        ? `<p class="small">Нет доступных законопроектов в этом цикле.</p><button id="skipQueueBtn" class="secondary mt16">Передать ход</button>`
        : `<label>Выберите законопроект для внесения</label>
           <select id="lawSel">
             ${avail.length ? `<optgroup label="Партийная повестка">${avail.map(l=>`<option value="${l}">${l}</option>`).join('')}</optgroup>` : ''}
             ${availConst.length ? `<optgroup label="Конституционные законы">${availConst.map(cl=>`<option value="${cl.name}">${cl.name}</option>`).join('')}</optgroup>` : ''}
           </select>
           <div class="row mt16"><button id="proposeLawBtn">Внести на голосование</button><button id="skipQueueBtn" class="secondary">Не вносить, передать ход</button></div>`;
    } else {
      const avail = REFORMS.filter(rf=>!room.proposedThisCycle.includes(rf.name));
      body = avail.length===0
        ? `<p class="small">Все реформы уже рассмотрены в этом цикле.</p><button id="skipQueueBtn" class="secondary mt16">Передать ход</button>`
        : `<label>Выберите реформу для внесения</label>
           <select id="reformSel">${avail.map(rf=>`<option value="${rf.name}">${rf.name}</option>`).join('')}</select>
           <div class="row mt16"><button id="proposeReformBtn">Внести на голосование</button><button id="skipQueueBtn" class="secondary">Не вносить, передать ход</button></div>`;
    }
  }
  return `
    <div class="plaque">
      <h2 class="section-title">Действия цикла ${room.cycle}</h2>
      ${room.cycleActions.length ? room.cycleActions.map(l=>`
        <div class="law-item">
          <div class="flex-between"><b>${l.title}</b><span class="badge">${l.kind==='law'?'закон':'реформа'} · ${l.result==='passed'?'принят':'отклонён'}</span></div>
          <div class="small">Внёс: ${PARTIES[l.by].name} · ${l.yesSeats}/${l.totalSeats} мандатов «за»</div>
        </div>
      `).join('') : `<p class="small">Пока ничего не рассмотрено</p>`}
    </div>
    <div class="plaque">
      <h2 class="section-title">${pending ? (pending.kind==='law'?'Голосование по закону':'Голосование по реформе') : 'Ход'}</h2>
      ${body}
    </div>
  `;
}

/* ============================== ACTIONS ============================== */

async function applyRoomUpdate(r){
  room = r;
  if(r.electionResult && r.electionResult.ts && r.electionResult.ts!==lastAnimatedElectionTs){
    lastAnimatedElectionTs = r.electionResult.ts;
    await playElectionReveal(r);
  } else if(r.lastResolution && r.lastResolution.ts!==lastAnimatedResolutionTs){
    lastAnimatedResolutionTs = r.lastResolution.ts;
    await playResolutionReveal(r.lastResolution);
  }
  render();
}

async function refreshRoom(){
  if(!me.roomCode || animatingReveal) return;
  const r = await loadRoom(me.roomCode);
  if(r){
    if(view!=='game' && r.status==='playing') view='game';
    if(view==='landing') view='lobby';
    await applyRoomUpdate(r);
  } else {
    render();
  }
}
function startPolling(){
  if(pollTimer) clearInterval(pollTimer);
  pollTimer = setInterval(refreshRoom, 2000);
}

function animateBar(elBar, elCount, target, maxForPct, ms){
  return new Promise(resolve=>{
    const start = performance.now();
    function step(now){
      const t = Math.min(1,(now-start)/ms);
      const eased = 1-Math.pow(1-t,3);
      const val = Math.round(target*eased);
      elBar.style.width = (maxForPct? Math.min(100,val/maxForPct*100):0)+'%';
      elCount.textContent = val;
      if(t<1){ requestAnimationFrame(step); }
      else { elCount.textContent=target; elBar.style.width=(maxForPct?Math.min(100,target/maxForPct*100):0)+'%'; resolve(); }
    }
    requestAnimationFrame(step);
  });
}

async function playElectionReveal(r){
  animatingReveal=true;
  if(pollTimer) clearInterval(pollTimer);
  const er = r.electionResult;
  const parties = PARTY_KEYS.filter(k=>(er.votes[k]||0)>0);
  const maxVotes = Math.max(1,...parties.map(k=>er.votes[k]));
  document.getElementById('root').innerHTML = `
    <div class="reveal-wrap">
      <div class="reveal-kicker">Подсчёт голосов</div>
      <div class="reveal-title">Оглашение результатов выборов</div>
      ${parties.map(k=>`
        <div class="reveal-bar-row">
          <div class="reveal-name" style="color:${PARTIES[k].color}">${PARTIES[k].name}</div>
          <div class="reveal-track"><div class="reveal-fill" id="ev-${k}" style="background:${PARTIES[k].color}"></div></div>
          <div class="reveal-count" id="evc-${k}">0</div>
        </div>
      `).join('')}
    </div>`;
  await Promise.all(parties.map(k=>{
    const bar=document.getElementById('ev-'+k), cnt=document.getElementById('evc-'+k);
    return animateBar(bar,cnt,er.votes[k],maxVotes,1300);
  }));
  await sleep(500);
  animatingReveal=false;
  startPolling();
}

async function playResolutionReveal(res){
  animatingReveal=true;
  if(pollTimer) clearInterval(pollTimer);
  document.getElementById('root').innerHTML = `
    <div class="reveal-wrap">
      <div class="reveal-kicker">${res.kind==='law'?'Голосование по закону':'Голосование по реформе'} · внесла ${PARTIES[res.by].name}</div>
      <div class="reveal-title">${res.title}</div>
      <div class="reveal-bar-row">
        <div class="reveal-name" style="color:#8fd18a">За</div>
        <div class="reveal-track"><div class="reveal-fill" id="rv-yes" style="background:#6aa06a"></div></div>
        <div class="reveal-count" id="rvc-yes">0</div>
      </div>
      <div class="reveal-bar-row">
        <div class="reveal-name" style="color:#e07a72">Против</div>
        <div class="reveal-track"><div class="reveal-fill" id="rv-no" style="background:#a05252"></div></div>
        <div class="reveal-count" id="rvc-no">0</div>
      </div>
      <div class="reveal-verdict" id="rvVerdict"></div>
    </div>`;
  const barYes=document.getElementById('rv-yes'), cntYes=document.getElementById('rvc-yes');
  const barNo=document.getElementById('rv-no'), cntNo=document.getElementById('rvc-no');
  await Promise.all([
    animateBar(barYes,cntYes,res.yesSeats,res.totalSeats,1300),
    animateBar(barNo,cntNo,res.noSeats,res.totalSeats,1300),
  ]);
  const verdict=document.getElementById('rvVerdict');
  if(verdict){
    verdict.textContent = res.passed? 'ПРИНЯТО' : 'ОТКЛОНЕНО';
    verdict.classList.add('show', res.passed?'pass':'fail');
  }
  await sleep(1100);
  animatingReveal=false;
  startPolling();
}

async function createRoom(name, scenario){
  const code = Math.random().toString(36).slice(2,6).toUpperCase();
  me.name=name; me.roomCode=code;
  const state = {
    code, scenario, status:'lobby', hostId: me.id,
    players:[{id:me.id, name, party:null}],
    cycle:0, phase:1, turnOrder:[], turnIndex:0,
    budgets:{}, hands:{}, support:{}, passed:[], cardHistory:{},
    electoralThreshold: SCENARIOS[scenario].threshold, budgetPerCycle: START_BUDGET,
    electionResult:null, allPassedLaws:[], proposedThisCycle:[], cycleActions:[],
    reformBonus:{}, pendingAction:null, phase3Queue:[], phase3QueueIndex:0,
    lastResolution:null,
    log:[`Комната ${code} создана (${SCENARIOS[scenario].label})`],
  };
  room=state;
  try{ await initHostNetwork(code); }catch(e){ uiError=e.message||'Не удалось создать сетевую комнату'; render(); return; }
  await saveRoom(state);
  view='lobby'; startPolling(); render();
}

async function joinRoom(name, code){
  code = code.trim().toUpperCase();
  try{ await initClientNetwork(code); }catch(e){ uiError='Не удалось подключиться к комнате. Проверьте код.'; render(); return; }
  const r = await loadRoom(code);
  if(!r){ uiError='Комната не найдена. Проверьте код.'; render(); return; }
  if(r.status!=='lobby'){ uiError='Игра в этой комнате уже началась.'; render(); return; }
  if(!r.players.some(p=>p.id===me.id)){
    r.players.push({id:me.id, name, party:null});
    r.log.push(`${name} присоединился(-лась) к комнате`);
  }
  me.name=name; me.roomCode=code;
  await saveRoom(r);
  room=r; view='lobby'; uiError=''; startPolling(); render();
}

async function choosePartyAction(partyKey){
  const r = await loadRoom(room.code);
  if(!r) return;
  const taken = r.players.find(p=>p.party===partyKey && p.id!==me.id);
  if(taken) return;
  const p = r.players.find(p=>p.id===me.id);
  if(p) p.party = partyKey;
  await saveRoom(r); room=r; render();
}

async function startGameAction(){
  const r = await loadRoom(room.code);
  if(!r) return;
  r.status='playing';
  r.cycle=1; r.phase=1;
  r.turnOrder = r.players.map(p=>p.id);
  r.turnIndex = 0;
  r.support = {}; r.passed=[];
  r.players.forEach(p=>{ r.budgets[p.id]=r.budgetPerCycle; r.hands[p.id]=dealHand(); r.support[p.party]=0; });
  r.log.push(`— Цикл 1 начался. Фаза 1: агитация. —`);
  await saveRoom(r); room=r; view='game'; render();
}

async function playCardAction(handId, targetParty){
  const r = await loadRoom(room.code);
  if(!r) return;
  if(r.turnOrder[r.turnIndex]!==me.id) return;
  if(r.passed.includes(me.id)) return;
  const p = r.players.find(p=>p.id===me.id);
  const hand = r.hands[me.id]||[];
  const card = hand.find(c=>c.handId===handId);
  if(!card || r.budgets[me.id] < card.cost) return;
  if(card.type==='discredit'){
    if(!targetParty) return;
    const targetOk = r.players.some(pl=>pl.party===targetParty && pl.id!==me.id) && cardMeetsPrereq(card, targetParty, r);
    if(!targetOk) return;
  }

  r.budgets[me.id]-=card.cost;
  r.hands[me.id]=hand.filter(c=>c.handId!==handId);
  r.cardHistory[p.party] = r.cardHistory[p.party]||[];
  if(!r.cardHistory[p.party].includes(card.name)) r.cardHistory[p.party].push(card.name);

  if(card.type==='agitation'){
    let gain = card.value;
    if((card.bonus||[]).includes(p.party)) gain += 1;
    r.support[p.party] = (r.support[p.party]||0) + gain;
    r.log.push(`${p.name} (${PARTIES[p.party].name}) разыграл(а) «${card.name}»: +${gain} поддержки`);
    flashBadge('+'+gain, PARTIES[p.party].color);
  } else {
    r.support[targetParty] = Math.max(0,(r.support[targetParty]||0)-card.value);
    r.log.push(`${p.name} (${PARTIES[p.party].name}) разыграл(а) «${card.name}» против ${PARTIES[targetParty].name}: −${card.value}`);
    flashBadge('−'+card.value, '#e07a72');
  }
  await saveRoom(r); room=r; render();
}

function handleCardClick(handId){
  const hand = room.hands[me.id]||[];
  const card = hand.find(c=>c.handId===handId);
  if(!card) return;
  const cardEl = document.querySelector(`[data-play="${handId}"]`);
  if(cardEl) cardEl.classList.add('card-playing');
  if(card.type==='agitation'){
    setTimeout(()=>{ playCardAction(handId, null); }, 380);
  } else {
    setTimeout(()=>{ discreditPicker={handId, card}; render(); }, 380);
  }
}
function chooseDiscreditTarget(targetParty){
  if(!discreditPicker) return;
  const handId = discreditPicker.handId;
  discreditPicker=null;
  playCardAction(handId, targetParty);
}
function flashBadge(text, color){
  const el=document.createElement('div');
  el.className='flash-badge';
  el.textContent=text;
  el.style.color=color;
  document.body.appendChild(el);
  requestAnimationFrame(()=>{ requestAnimationFrame(()=>{ el.classList.add('rise'); }); });
  setTimeout(()=>{ el.remove(); }, 1100);
}

async function passAction(){
  const r = await loadRoom(room.code);
  if(!r) return;
  if(r.turnOrder[r.turnIndex]!==me.id) return;
  if(!r.passed.includes(me.id)) r.passed.push(me.id);
  const p=r.players.find(p=>p.id===me.id);
  r.log.push(`${p.name} закончил(а) ход в фазе агитации`);
  advanceTurn(r);
  await saveRoom(r); await applyRoomUpdate(r);
}

function advanceTurn(r){
  const active = r.players.filter(p=>!r.passed.includes(p.id));
  if(active.length===0){ runElection(r); return; }
  let next = (r.turnIndex+1) % r.turnOrder.length;
  let guard=0;
  while(r.passed.includes(r.turnOrder[next]) && guard<r.turnOrder.length+1){
    next = (next+1)%r.turnOrder.length; guard++;
  }
  r.turnIndex=next;
}

function runElection(r){
  const votes = {};
  PARTY_KEYS.forEach(k=>{
    if(r.players.some(p=>p.party===k)){
      const base = (r.support[k]||0)+(r.reformBonus[k]||0);
      const swing = 0.6 + Math.random()*0.8;   // 0.6x–1.4x campaign swing
      const chaos = Math.random()*12;          // unpredictable "the people" factor
      votes[k] = Math.max(0, Math.round(base*swing + chaos));
    }
  });
  const scenarioInfo = SCENARIOS[r.scenario];
  const seats = allocateSeats(votes, TOTAL_SEATS, scenarioInfo.formula, r.electoralThreshold);
  const presidentParty = Object.entries(votes).sort((a,b)=>b[1]-a[1])[0]?.[0] || null;
  r.electionResult = {votes, seats, president:presidentParty, government:null, invited:[presidentParty], ts:Date.now()};
  r.reformBonus = {};
  r.phase=2;
  r.log.push(`— Выборы завершены. Президентская партия: ${presidentParty?PARTIES[presidentParty].name:'—'} —`);
}

async function toggleInviteAction(partyKey){
  const r = await loadRoom(room.code);
  if(!r || !r.electionResult || r.electionResult.government) return;
  const meParty = (r.players.find(p=>p.id===me.id)||{}).party;
  if(r.electionResult.president !== meParty) return;
  const inv = r.electionResult.invited && r.electionResult.invited.length ? r.electionResult.invited : [r.electionResult.president];
  const idx = inv.indexOf(partyKey);
  if(idx>=0) inv.splice(idx,1); else inv.push(partyKey);
  r.electionResult.invited = inv;
  await saveRoom(r); room=r; render();
}

async function confirmGovernmentAction(){
  const r = await loadRoom(room.code);
  if(!r || !r.electionResult || r.electionResult.government) return;
  const meParty = (r.players.find(p=>p.id===me.id)||{}).party;
  if(r.electionResult.president !== meParty) return;
  const inv = r.electionResult.invited && r.electionResult.invited.length ? r.electionResult.invited : [r.electionResult.president];
  r.electionResult.government = inv.slice();
  r.log.push(`— Правительство сформировано: ${inv.map(k=>PARTIES[k].name).join(', ')} —`);
  await saveRoom(r); room=r; render();
}

async function toPhase3Action(){
  const r = await loadRoom(room.code);
  if(!r || !r.electionResult || !r.electionResult.government) return;
  r.phase=3;
  r.proposedThisCycle=[]; r.cycleActions=[]; r.pendingAction=null;
  const parliamentParties = PARTY_KEYS.filter(k=>(r.electionResult.seats[k]||0)>0);
  const govParties = r.electionResult.government;
  const orderedParliament = r.players.filter(p=>parliamentParties.includes(p.party)).map(p=>p.party);
  const orderedGov = r.players.filter(p=>govParties.includes(p.party)).map(p=>p.party);
  const queue=[];
  const maxLen = Math.max(orderedParliament.length, orderedGov.length);
  for(let i=0;i<maxLen;i++){
    if(orderedParliament[i]) queue.push({kind:'law', party:orderedParliament[i]});
    if(orderedGov[i]) queue.push({kind:'reform', party:orderedGov[i]});
  }
  r.phase3Queue = queue;
  r.phase3QueueIndex = 0;
  r.log.push(`— Фаза 3: законы чередуются с реформами правительства —`);
  await saveRoom(r); room=r; render();
}

async function proposeItemAction(kind, title){
  const r = await loadRoom(room.code);
  if(!r || r.pendingAction) return;
  const item = r.phase3Queue[r.phase3QueueIndex];
  if(!item || item.kind!==kind) return;
  const meParty = (r.players.find(p=>p.id===me.id)||{}).party;
  if(item.party !== meParty) return;
  r.pendingAction = { kind, title, by:item.party, votes:{} };
  const p = r.players.find(p=>p.id===me.id);
  r.log.push(`${p.name} (${PARTIES[item.party].name}) выносит на голосование ${kind==='law'?'закон':'реформу'} «${title}»`);
  await saveRoom(r); room=r; render();
}

async function skipQueueItemAction(){
  const r = await loadRoom(room.code);
  if(!r || r.pendingAction) return;
  const item = r.phase3Queue[r.phase3QueueIndex];
  if(!item) return;
  const meParty = (r.players.find(p=>p.id===me.id)||{}).party;
  if(item.party !== meParty) return;
  const p = r.players.find(p=>p.id===me.id);
  r.log.push(`${p.name} (${PARTIES[item.party].name}) передал(а) ход без внесения`);
  r.phase3QueueIndex += 1;
  await saveRoom(r); room=r; render();
}

async function castVoteAction(choice){
  const r = await loadRoom(room.code);
  if(!r || !r.pendingAction) return;
  const p = r.players.find(p=>p.id===me.id);
  if(!p) return;
  const seats = r.electionResult.seats;
  if(!(seats[p.party]>0)) return;
  if(r.pendingAction.votes[p.id]!==undefined) return;
  r.pendingAction.votes[p.id]=choice;

  const seatedPlayers = r.players.filter(pl=>(seats[pl.party]||0)>0);
  const allVoted = seatedPlayers.every(pl=>r.pendingAction.votes[pl.id]!==undefined);
  if(allVoted){
    let yesSeats=0, noSeats=0;
    seatedPlayers.forEach(pl=>{
      const v=r.pendingAction.votes[pl.id];
      if(v==='yes') yesSeats+=seats[pl.party]; else noSeats+=seats[pl.party];
    });
    const totalSeats = yesSeats+noSeats;
    const passed = yesSeats > totalSeats/2;
    const {kind,title,by} = r.pendingAction;
    const constLaw = kind==='law' ? CONSTITUTIONAL_LAWS.find(c=>c.name===title) : null;
    if(passed){
      if(constLaw){
        if(constLaw.kind==='raiseThreshold'){
          r.electoralThreshold = Math.min(15, (r.electoralThreshold||0)+3);
          r.log.push(`— Электоральный барьер повышен до ${r.electoralThreshold}% —`);
        } else if(constLaw.kind==='cutFunding'){
          r.budgetPerCycle = Math.max(1, (r.budgetPerCycle||START_BUDGET)-2);
          r.log.push(`— Финансирование партий снижено до ₽${r.budgetPerCycle} за цикл —`);
        }
      } else if(kind==='law'){
        if(!r.allPassedLaws.includes(title)) r.allPassedLaws.push(title);
      } else {
        const rf = REFORMS.find(x=>x.name===title);
        if(rf){
          r.reformBonus[by] = (r.reformBonus[by]||0) + rf.value;
          (rf.bonus||[]).forEach(b=>{ if(b!==by) r.reformBonus[b]=(r.reformBonus[b]||0)+1; });
        }
      }
    }
    r.proposedThisCycle.push(title);
    r.cycleActions.push({kind,title,by,result:passed?'passed':'rejected', yesSeats, noSeats, totalSeats});
    r.log.push(`Голосование «${title}»: ${passed?'ПРИНЯТО':'ОТКЛОНЕНО'} (${yesSeats}/${totalSeats} мандатов «за»)`);
    r.lastResolution = {kind,title,by,yesSeats,noSeats,totalSeats,passed, ts:Date.now()};
    r.pendingAction = null;
    r.phase3QueueIndex += 1;
  }
  await saveRoom(r); await applyRoomUpdate(r);
}

async function endCycleAction(){
  const r = await loadRoom(room.code);
  if(!r) return;
  r.cycle+=1; r.phase=1; r.passed=[];
  r.turnOrder=r.players.map(p=>p.id); r.turnIndex=0;
  r.support={};
  r.players.forEach(p=>{
    r.budgets[p.id]=(r.budgets[p.id]||0)+r.budgetPerCycle;
    r.hands[p.id]=dealHand();
    r.support[p.party]=0;
  });
  r.electionResult=null;
  r.pendingAction=null; r.phase3Queue=[]; r.phase3QueueIndex=0; r.proposedThisCycle=[]; r.cycleActions=[];
  r.log.push(`— Цикл ${r.cycle} начался. Фаза 1: агитация. —`);
  await saveRoom(r); room=r; render();
}

/* ============================== EVENTS ============================== */

function attachHandlers(){
  const createBtn=document.getElementById('createBtn');
  if(createBtn) createBtn.onclick=()=>{
    const name=document.getElementById('hostName').value.trim();
    const scenario=document.getElementById('scenarioSel').value;
    if(!name){ uiError='Введите имя'; render(); return; }
    uiError=''; createRoom(name, scenario);
  };
  const joinBtn=document.getElementById('joinBtn');
  if(joinBtn) joinBtn.onclick=()=>{
    const name=document.getElementById('joinName').value.trim();
    const code=document.getElementById('joinCode').value.trim();
    if(!name||!code){ uiError='Введите имя и код комнаты'; render(); return; }
    uiError=''; joinRoom(name, code);
  };
  document.querySelectorAll('.standard').forEach(el=>{
    el.onclick=()=>{ if(!el.classList.contains('taken')) choosePartyAction(el.dataset.party); };
  });
  const startBtn=document.getElementById('startBtn');
  if(startBtn) startBtn.onclick=startGameAction;

  document.querySelectorAll('.playcard').forEach(el=>{
    el.onclick=()=>{ if(!el.classList.contains('disabled')) handleCardClick(el.dataset.play); };
  });
  const passBtn=document.getElementById('passBtn');
  if(passBtn) passBtn.onclick=passAction;

  document.querySelectorAll('[data-target]').forEach(el=>{
    el.onclick=()=>chooseDiscreditTarget(el.dataset.target);
  });
  const cancelDiscreditBtn=document.getElementById('cancelDiscreditBtn');
  if(cancelDiscreditBtn) cancelDiscreditBtn.onclick=()=>{ discreditPicker=null; render(); };

  document.querySelectorAll('.gov-chip[data-invite]').forEach(el=>{
    el.onclick=()=>toggleInviteAction(el.dataset.invite);
  });
  const confirmGovBtn=document.getElementById('confirmGovBtn');
  if(confirmGovBtn) confirmGovBtn.onclick=confirmGovernmentAction;

  const toPhase3Btn=document.getElementById('toPhase3Btn');
  if(toPhase3Btn) toPhase3Btn.onclick=toPhase3Action;

  const proposeLawBtn=document.getElementById('proposeLawBtn');
  if(proposeLawBtn) proposeLawBtn.onclick=()=>{
    const sel=document.getElementById('lawSel');
    if(sel) proposeItemAction('law', sel.value);
  };
  const proposeReformBtn=document.getElementById('proposeReformBtn');
  if(proposeReformBtn) proposeReformBtn.onclick=()=>{
    const sel=document.getElementById('reformSel');
    if(sel) proposeItemAction('reform', sel.value);
  };
  const skipQueueBtn=document.getElementById('skipQueueBtn');
  if(skipQueueBtn) skipQueueBtn.onclick=skipQueueItemAction;

  document.querySelectorAll('.vote-buttons button').forEach(el=>{
    el.onclick=()=>castVoteAction(el.dataset.vote);
  });

  const endCycleBtn=document.getElementById('endCycleBtn');
  if(endCycleBtn) endCycleBtn.onclick=endCycleAction;
}

render();
</script>
</body>
</html>

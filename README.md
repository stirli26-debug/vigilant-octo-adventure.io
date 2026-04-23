<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>♠ Cribbage — Two Player</title>
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; background: #f5f4f0; color: #222; min-height: 100vh; }
  #app { max-width: 660px; margin: 0 auto; padding: 1rem; min-height: 100vh; }

  .screen { display: none; }
  .screen.active { display: block; }

  h1 { font-size: 2rem; font-weight: 600; margin-bottom: 0.25rem; }

  .card {
    display: inline-flex; flex-direction: column; justify-content: space-between;
    width: 54px; height: 78px; border-radius: 7px; border: 1.5px solid #ccc;
    background: white; color: #222; font-size: 13px; font-weight: 600;
    padding: 4px 5px; cursor: pointer; position: relative; user-select: none;
    transition: transform 0.15s, box-shadow 0.15s; vertical-align: bottom;
    box-shadow: 0 1px 3px rgba(0,0,0,0.08);
  }
  .card:hover { transform: translateY(-6px); box-shadow: 0 6px 16px rgba(0,0,0,0.15); }
  .card.selected { transform: translateY(-12px); border-color: #1D9E75; box-shadow: 0 0 0 2px #1D9E75, 0 6px 16px rgba(0,0,0,0.15); }
  .card.face-down { background: repeating-linear-gradient(135deg,#185FA5,#185FA5 4px,#0C447C 4px,#0C447C 8px); border-color: #042C53; cursor: default; }
  .card.face-down:hover { transform: none; box-shadow: 0 1px 3px rgba(0,0,0,0.08); }
  .card .suit { font-size: 17px; line-height: 1; }
  .card .bot { align-self: flex-end; transform: rotate(180deg); }
  .red { color: #C0392B; }

  .btn {
    padding: 9px 22px; border-radius: 8px; border: 1.5px solid #ccc;
    background: white; color: #222; font-size: 14px; font-weight: 500;
    cursor: pointer; transition: background 0.12s, border-color 0.12s;
  }
  .btn:hover { background: #f0eeea; border-color: #aaa; }
  .btn.primary { background: #185FA5; color: white; border-color: #185FA5; }
  .btn.primary:hover { background: #0C447C; border-color: #0C447C; }
  .btn.success { background: #1D9E75; color: white; border-color: #1D9E75; }
  .btn.success:hover { background: #0F6E56; }
  .btn:disabled { opacity: 0.4; cursor: default; pointer-events: none; }

  .input-field {
    padding: 10px 14px; border-radius: 8px; border: 1.5px solid #ccc;
    background: white; color: #222; font-size: 15px; width: 100%;
    outline: none; transition: border-color 0.15s;
  }
  .input-field:focus { border-color: #185FA5; }

  .badge { display: inline-block; padding: 3px 10px; border-radius: 20px; font-size: 12px; font-weight: 600; }
  .badge.blue { background: #E6F1FB; color: #185FA5; }
  .badge.orange { background: #FAEEDA; color: #854F0B; }
  .badge.green { background: #EAF3DE; color: #3B6D11; }

  .score-box {
    background: white; border: 1.5px solid #ddd; border-radius: 10px;
    padding: 8px 16px; display: flex; align-items: center; gap: 12px;
  }
  .score-num { font-size: 30px; font-weight: 600; min-width: 36px; text-align: center; line-height: 1; }
  .score-label { font-size: 11px; color: #888; font-weight: 500; text-transform: uppercase; letter-spacing: 0.05em; }

  .msg-box {
    padding: 12px 16px; border-radius: 8px; background: #f0eeea;
    font-size: 14px; color: #444; border-left: 3px solid #185FA5;
  }

  .cards-row { display: flex; flex-wrap: wrap; gap: 7px; min-height: 84px; align-items: flex-end; padding: 4px 0; }

  .phase-title { font-size: 20px; font-weight: 600; margin-bottom: 10px; }

  .divider { border: none; border-top: 1px solid #e8e5df; margin: 14px 0; }

  .section-label { font-size: 11px; color: #888; text-transform: uppercase; letter-spacing: 0.06em; font-weight: 600; margin-bottom: 8px; }

  .stat-card { background: white; border: 1.5px solid #e8e5df; border-radius: 10px; padding: 10px 16px; }

  #log { max-height: 110px; overflow-y: auto; background: white; border: 1px solid #e8e5df; border-radius: 8px; padding: 8px 12px; }
  #log .log-entry { font-size: 12px; color: #666; padding: 3px 0; border-bottom: 1px solid #f0eeea; }
  #log .log-entry:last-child { border-bottom: none; }

  #pegging-pile { display: flex; flex-wrap: wrap; gap: 4px; min-height: 44px; padding: 4px 0; }
  #pegging-pile .card { width: 40px; height: 58px; font-size: 11px; padding: 3px 4px; }
  #pegging-pile .card .suit { font-size: 13px; }
  #pegging-pile .card:hover { transform: none; box-shadow: 0 1px 3px rgba(0,0,0,0.08); }

  .lobby-card {
    background: white; border: 1.5px solid #e8e5df; border-radius: 14px;
    padding: 32px 28px; box-shadow: 0 2px 12px rgba(0,0,0,0.06);
  }

  .room-code {
    font-size: 44px; font-weight: 700; letter-spacing: 0.18em;
    padding: 24px; background: #f0eeea; border-radius: 12px;
    text-align: center; font-family: 'Courier New', monospace;
    color: #185FA5;
  }

  .player-header {
    display: flex; justify-content: space-between; align-items: center;
    background: white; border: 1.5px solid #e8e5df; border-radius: 12px;
    padding: 10px 16px; margin-bottom: 14px;
  }

  .top-bar { display: flex; justify-content: space-between; align-items: center; margin-bottom: 14px; }

  .spinner { display: inline-block; animation: spin 1s linear infinite; font-size: 22px; }
  @keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }

  @media (max-width: 500px) {
    .card { width: 44px; height: 64px; font-size: 11px; }
    .card .suit { font-size: 14px; }
    .room-code { font-size: 32px; }
  }
</style>
</head>
<body>
<div id="app">

  <!-- LOBBY -->
  <div id="screen-lobby" class="screen active">
    <div style="max-width: 380px; margin: 48px auto;">
      <div class="lobby-card">
        <div style="text-align:center; margin-bottom: 28px;">
          <div style="font-size: 3rem; margin-bottom: 8px;">♠</div>
          <h1>Cribbage</h1>
          <p style="color: #888; font-size: 14px; margin-top: 4px;">Two-player · Two devices</p>
        </div>
        <div style="margin-bottom: 12px;">
          <div class="section-label">Your name</div>
          <input id="player-name" class="input-field" placeholder="Enter your name" maxlength="20" />
        </div>
        <button class="btn primary" style="width:100%; margin-bottom: 20px; padding: 12px;" onclick="createGame()">
          Create New Game
        </button>
        <div style="display:flex; align-items:center; gap:10px; margin-bottom: 16px;">
          <div style="flex:1; height:1px; background:#e8e5df;"></div>
          <span style="font-size:13px; color:#aaa;">or join</span>
          <div style="flex:1; height:1px; background:#e8e5df;"></div>
        </div>
        <div style="margin-bottom: 10px;">
          <div class="section-label">Room code</div>
          <input id="join-code" class="input-field" placeholder="ABC123" maxlength="6"
            style="text-align:center; font-size:20px; letter-spacing:0.15em; text-transform:uppercase;" />
        </div>
        <button class="btn" style="width:100%; padding:12px;" onclick="joinGame()">Join Game</button>
        <div id="lobby-msg" style="margin-top:12px; font-size:13px; color:#C0392B; text-align:center; min-height:20px;"></div>
      </div>
    </div>
  </div>

  <!-- WAITING -->
  <div id="screen-waiting" class="screen">
    <div style="max-width: 400px; margin: 48px auto; text-align:center;">
      <div class="lobby-card">
        <div style="font-size: 22px; font-weight: 600; margin-bottom: 10px;">Room Created!</div>
        <p style="color: #888; font-size: 14px; margin-bottom: 20px;">Share this code with your opponent:</p>
        <div class="room-code" id="display-code"></div>
        <p style="font-size: 13px; color: #888; margin-top: 18px; margin-bottom: 6px;">Waiting for Player 2 to join...</p>
        <div class="spinner">⟳</div>
      </div>
    </div>
  </div>

  <!-- GAME -->
  <div id="screen-game" class="screen">

    <div class="player-header">
      <div style="display:flex; align-items:center; gap:8px;">
        <span class="badge blue" id="my-name-badge">You</span>
        <span style="font-size:11px; color:#aaa;" id="my-role-label"></span>
      </div>
      <div class="score-box">
        <div>
          <div class="score-label" id="my-score-label">You</div>
          <div class="score-num" id="my-score">0</div>
        </div>
        <div style="color:#ccc; font-size:20px;">·</div>
        <div>
          <div class="score-label" id="opp-score-label">Them</div>
          <div class="score-num" id="opp-score">0</div>
        </div>
      </div>
      <div style="display:flex; align-items:center; gap:8px;">
        <span style="font-size:11px; color:#aaa;" id="opp-role-label"></span>
        <span class="badge orange" id="opp-name-badge">Opponent</span>
      </div>
    </div>

    <div class="phase-title" id="phase-banner"></div>
    <div class="msg-box" id="game-msg"></div>
    <hr class="divider" />

    <!-- Starter card -->
    <div id="starter-area" style="display:none; margin-bottom:14px;">
      <div class="section-label">Starter card</div>
      <div class="cards-row" id="starter-display"></div>
    </div>

    <!-- Discard phase -->
    <div id="crib-discard-area" style="display:none; margin-bottom:16px;">
      <div class="section-label">Select 2 cards for the crib <span id="crib-owner-label" style="color:#185FA5;"></span></div>
      <div class="cards-row" id="my-hand-discard"></div>
      <div style="margin-top:12px; display:flex; gap:8px; align-items:center;">
        <button class="btn primary" id="btn-discard" onclick="discardToCrib()" disabled>Discard (0/2)</button>
        <span id="discard-waiting" style="font-size:13px; color:#888;"></span>
      </div>
    </div>

    <!-- Cut phase -->
    <div id="cut-area" style="display:none; margin-bottom:16px;">
      <button class="btn primary" id="btn-cut" onclick="cutDeck()">✂ Cut the deck for starter card</button>
    </div>

    <!-- Pegging phase -->
    <div id="pegging-area" style="display:none; margin-bottom:16px;">
      <div class="section-label">Running count: <span id="peg-count" style="color:#185FA5; font-size:14px; font-weight:700;">0</span> / 31</div>
      <div id="pegging-pile"></div>
      <hr class="divider" />
      <div class="section-label">Your hand</div>
      <div class="cards-row" id="my-hand-peg"></div>
      <div style="margin-top:12px; display:flex; gap:8px; flex-wrap:wrap;">
        <button class="btn primary" id="btn-play-card" onclick="playCard()" disabled>Play Card</button>
        <button class="btn" id="btn-go" onclick="sayGo()">Say "Go"</button>
      </div>
    </div>

    <!-- Scoring phase -->
    <div id="scoring-area" style="display:none; margin-bottom:16px;">
      <div class="section-label" id="score-phase-label"></div>
      <div class="cards-row" id="score-hand-display"></div>
      <div id="score-breakdown" style="margin-top:10px; padding:10px 14px; background:#f0eeea; border-radius:8px; font-size:14px; color:#444;"></div>
      <div style="margin-top:12px; display:flex; gap:8px; align-items:center;">
        <button class="btn success" id="btn-score-continue" onclick="continueScoring()">Accept & Continue</button>
        <span id="score-waiting" style="font-size:13px; color:#888;"></span>
      </div>
    </div>

    <!-- Opponent hand (hidden during pegging) -->
    <div id="opp-hand-area" style="display:none; margin-bottom:16px;">
      <div class="section-label">Opponent's cards (face down)</div>
      <div class="cards-row" id="opp-hand-display"></div>
    </div>

    <hr class="divider" />
    <div class="section-label">Game log</div>
    <div id="log"></div>
  </div>

  <!-- GAME OVER -->
  <div id="screen-gameover" class="screen">
    <div style="max-width: 360px; margin: 80px auto; text-align:center;">
      <div class="lobby-card">
        <div style="font-size: 60px; margin-bottom: 12px;" id="gameover-icon">🏆</div>
        <div style="font-size: 26px; font-weight: 700; margin-bottom: 8px;" id="gameover-title"></div>
        <div style="font-size: 15px; color:#888; margin-bottom: 28px;" id="gameover-score"></div>
        <button class="btn primary" style="width:100%; padding:12px;" onclick="resetToLobby()">Play Again</button>
      </div>
    </div>
  </div>

</div>

<script>
// ─── Storage (uses localStorage for cross-device sharing via same key) ─────────
// For true cross-device play, both players must access this file from the same server.
// localStorage only works on the same device/browser. For network play we use
// a simple polling approach via a shared JSON "server" simulation using BroadcastChannel
// + localStorage with storage events so both tabs/devices on same origin can sync.

// Since this is a standalone file, we use localStorage with storage events for same-origin
// and provide instructions for hosting. Game state key is 'crib_game_' + roomCode.

const SUITS = ['♠','♥','♦','♣'];
const RANKS = ['A','2','3','4','5','6','7','8','9','10','J','Q','K'];
const RED_SUITS = ['♥','♦'];

let state = null;
let myPlayer = null;
let roomCode = null;
let myName = null;
let pollInterval = null;
let selectedCards = [];
let selectedPegCard = null;
let lastStateStr = '';

// ─── Storage helpers ────────────────────────────────────────────────────────
function saveState(code, s) {
  try { localStorage.setItem('crib_game_' + code, JSON.stringify(s)); } catch(e) {}
}
function loadState(code) {
  try {
    const v = localStorage.getItem('crib_game_' + code);
    return v ? JSON.parse(v) : null;
  } catch(e) { return null; }
}

// ─── Helpers ────────────────────────────────────────────────────────────────
function cardValue(rank) {
  if (['J','Q','K'].includes(rank)) return 10;
  if (rank === 'A') return 1;
  return parseInt(rank);
}
function rankNum(r) {
  const m = {A:1,2:2,3:3,4:4,5:5,6:6,7:7,8:8,9:9,10:10,J:11,Q:12,K:13};
  return m[r] || 0;
}
function makeDeck() {
  const d = [];
  for (const s of SUITS) for (const r of RANKS) d.push({suit:s, rank:r});
  return d;
}
function shuffle(d) {
  for (let i = d.length-1; i > 0; i--) {
    const j = Math.floor(Math.random()*(i+1));
    [d[i],d[j]] = [d[j],d[i]];
  }
  return d;
}
function genCode() {
  return Math.random().toString(36).substring(2,8).toUpperCase();
}
function me() { return state[myPlayer]; }
function opp() { return myPlayer === 'p1' ? state.p2 : state.p1; }
function oppKey() { return myPlayer === 'p1' ? 'p2' : 'p1'; }
function isDealer() { return state.dealer === myPlayer; }
function isMyTurn() { return state.phase === 'pegging' && state.pegging.current_turn === myPlayer; }

// ─── Screens ─────────────────────────────────────────────────────────────────
function showScreen(id) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  document.getElementById('screen-' + id).classList.add('active');
}

function addLog(msg) {
  const el = document.getElementById('log');
  const d = document.createElement('div');
  d.className = 'log-entry';
  d.textContent = msg;
  el.prepend(d);
}

// ─── Lobby actions ───────────────────────────────────────────────────────────
async function createGame() {
  myName = document.getElementById('player-name').value.trim() || 'Player 1';
  roomCode = genCode();
  myPlayer = 'p1';
  const deck = shuffle(makeDeck());
  state = {
    phase: 'waiting',
    p1: { name: myName, score: 0, hand: [], peg_hand: [] },
    p2: { name: null, score: 0, hand: [], peg_hand: [] },
    deck, crib: [], starter: null, dealer: 'p1',
    pegging: { pile: [], current_turn: null, last_player: null, p1_said_go: false, p2_said_go: false },
    score_phase: null, score_subject: null,
    p1_discarded: false, p2_discarded: false,
    log: []
  };
  saveState(roomCode, state);
  document.getElementById('display-code').textContent = roomCode;
  showScreen('waiting');
  pollInterval = setInterval(pollWaiting, 1000);
}

async function joinGame() {
  const code = document.getElementById('join-code').value.trim().toUpperCase();
  myName = document.getElementById('player-name').value.trim() || 'Player 2';
  const msgEl = document.getElementById('lobby-msg');
  if (!code || code.length !== 6) { msgEl.textContent = 'Enter a valid 6-character code.'; return; }
  const s = loadState(code);
  if (!s) { msgEl.textContent = 'Room not found. Check the code and make sure both players are on the same device/network.'; return; }
  if (s.phase !== 'waiting') { msgEl.textContent = 'That game is already in progress.'; return; }
  roomCode = code;
  myPlayer = 'p2';
  state = s;
  state.p2.name = myName;
  state.phase = 'deal';
  dealCards(state);
  state.log.unshift(myName + ' joined! Game starting.');
  saveState(roomCode, state);
  startGameUI();
}

// ─── Deal ────────────────────────────────────────────────────────────────────
function dealCards(s) {
  s.deck = shuffle(makeDeck());
  s.p1.hand = s.deck.splice(0, 6);
  s.p2.hand = s.deck.splice(0, 6);
  s.p1.peg_hand = [];
  s.p2.peg_hand = [];
  s.crib = [];
  s.starter = null;
  s.pegging = { pile: [], current_turn: null, last_player: null, p1_said_go: false, p2_said_go: false };
  s.score_phase = null;
  s.score_subject = null;
  s.phase = 'discard';
  s.p1_discarded = false;
  s.p2_discarded = false;
}

// ─── Polling ──────────────────────────────────────────────────────────────────
function pollWaiting() {
  const s = loadState(roomCode);
  if (s && s.phase !== 'waiting') {
    clearInterval(pollInterval);
    state = s;
    startGameUI();
  }
}

function startGameUI() {
  clearInterval(pollInterval);
  showScreen('game');
  renderGame();
  pollInterval = setInterval(() => {
    const s = loadState(roomCode);
    if (!s) return;
    const str = JSON.stringify(s);
    if (str !== lastStateStr) {
      lastStateStr = str;
      state = s;
      renderGame();
    }
  }, 800);
}

// ─── Card rendering ───────────────────────────────────────────────────────────
function cardHTML(c, selectable, idx, isPeg) {
  if (!c) return '';
  const isRed = RED_SUITS.includes(c.suit);
  let cls = 'card' + (isRed ? ' red' : '');
  if (selectable) {
    if (isPeg && selectedPegCard === idx) cls += ' selected';
    else if (!isPeg && selectedCards.includes(idx)) cls += ' selected';
  }
  const onclick = selectable ? `toggleSelect(${idx}, ${!!isPeg})` : '';
  return `<div class="${cls}" onclick="${onclick}">
    <span>${c.rank}</span>
    <span class="suit">${c.suit}</span>
    <span class="bot">${c.rank}</span>
  </div>`;
}

function faceDownCard() {
  return `<div class="card face-down"></div>`;
}

window.toggleSelect = function(idx, isPeg) {
  if (isPeg) {
    selectedPegCard = (selectedPegCard === idx) ? null : idx;
    renderPegging();
  } else {
    if (selectedCards.includes(idx)) selectedCards = selectedCards.filter(x => x !== idx);
    else if (selectedCards.length < 2) selectedCards.push(idx);
    renderDiscard();
  }
};

// ─── Main render ──────────────────────────────────────────────────────────────
function renderGame() {
  if (!state) return;
  if (state.phase === 'gameover') { showGameOver(); return; }

  document.getElementById('my-name-badge').textContent = me().name || 'You';
  document.getElementById('opp-name-badge').textContent = opp().name || 'Opponent';
  document.getElementById('my-score-label').textContent = me().name || 'You';
  document.getElementById('opp-score-label').textContent = opp().name || 'Them';
  document.getElementById('my-score').textContent = me().score;
  document.getElementById('opp-score').textContent = opp().score;
  document.getElementById('my-role-label').textContent = isDealer() ? '(dealer)' : '(pone)';
  document.getElementById('opp-role-label').textContent = isDealer() ? '(pone)' : '(dealer)';

  const allAreas = ['crib-discard-area','cut-area','pegging-area','scoring-area','starter-area','opp-hand-area'];
  allAreas.forEach(id => document.getElementById(id).style.display = 'none');

  document.getElementById('phase-banner').textContent = phaseLabel();
  document.getElementById('game-msg').textContent = gameMessage();

  if (state.starter) {
    document.getElementById('starter-area').style.display = 'block';
    document.getElementById('starter-display').innerHTML = cardHTML(state.starter, false, -1);
  }

  if (state.phase === 'discard') renderDiscard();
  else if (state.phase === 'cut') renderCut();
  else if (state.phase === 'pegging') renderPegging();
  else if (state.phase === 'scoring') renderScoring();

  // Sync log
  const logEl = document.getElementById('log');
  logEl.innerHTML = '';
  (state.log || []).slice().reverse().forEach(msg => {
    const d = document.createElement('div');
    d.className = 'log-entry';
    d.textContent = msg;
    logEl.appendChild(d);
  });
}

function phaseLabel() {
  const map = { waiting:'Waiting for opponent', deal:'Dealing', discard:'Discard to Crib', cut:'Cut the Deck', pegging:'Pegging', scoring:'Counting Hands', gameover:'Game Over' };
  return map[state.phase] || state.phase;
}

function gameMessage() {
  if (state.phase === 'discard') {
    if (state[myPlayer+'_discarded']) return 'Discarded! Waiting for opponent...';
    return 'Select 2 cards to send to the ' + (isDealer() ? 'your' : "dealer's") + ' crib.';
  }
  if (state.phase === 'cut') return isDealer() ? 'You are the dealer — cut the deck to reveal the starter card.' : 'Waiting for the dealer to cut the deck...';
  if (state.phase === 'pegging') {
    if (isMyTurn()) return 'Your turn — select a card and click Play, or Say "Go" if you cannot play.';
    return "Waiting for " + (opp().name || 'opponent') + " to play...";
  }
  if (state.phase === 'scoring') {
    const sp = state.score_phase, subj = state.score_subject;
    if (!sp || !subj) return '';
    const isMe = subj === myPlayer;
    if (sp === 'crib') return isDealer() ? 'Count your crib!' : (state[state.dealer].name + ' is counting the crib...');
    return isMe ? 'Count your hand!' : (state[subj].name + ' is counting their hand...');
  }
  return '';
}

// ─── Discard ──────────────────────────────────────────────────────────────────
function renderDiscard() {
  document.getElementById('crib-discard-area').style.display = 'block';
  document.getElementById('crib-owner-label').textContent = '→ ' + (isDealer() ? 'your crib' : "dealer's crib");
  const myDone = state[myPlayer+'_discarded'];
  const hand = me().hand;
  if (myDone) {
    document.getElementById('my-hand-discard').innerHTML = '<em style="color:#aaa;font-size:13px;">Cards discarded — waiting for opponent…</em>';
    document.getElementById('btn-discard').disabled = true;
    document.getElementById('discard-waiting').textContent = '';
  } else {
    document.getElementById('my-hand-discard').innerHTML = hand.map((c,i) => cardHTML(c, true, i, false)).join('');
    const btn = document.getElementById('btn-discard');
    btn.textContent = 'Discard (' + selectedCards.length + '/2)';
    btn.disabled = selectedCards.length !== 2;
    document.getElementById('discard-waiting').textContent = state[oppKey()+'_discarded'] ? '✓ Opponent ready' : '';
  }
}

async function discardToCrib() {
  if (selectedCards.length !== 2) return;
  const hand = [...me().hand];
  const toDiscard = selectedCards.map(i => hand[i]);
  state[myPlayer].hand = hand.filter((_,i) => !selectedCards.includes(i));
  state.crib.push(...toDiscard);
  state[myPlayer+'_discarded'] = true;
  selectedCards = [];
  if (state.p1_discarded && state.p2_discarded) {
    state.phase = 'cut';
    state.p1.peg_hand = [...state.p1.hand];
    state.p2.peg_hand = [...state.p2.hand];
    state.log.push('Both players discarded. Dealer cuts the deck.');
  } else {
    state.log.push((me().name || myPlayer) + ' discarded to the crib.');
  }
  saveState(roomCode, state);
  renderGame();
}

// ─── Cut ──────────────────────────────────────────────────────────────────────
function renderCut() {
  if (isDealer()) document.getElementById('cut-area').style.display = 'block';
}

async function cutDeck() {
  const starter = state.deck.splice(0, 1)[0];
  state.starter = starter;
  let nibs = '';
  if (starter.rank === 'J') {
    state[state.dealer].score += 2;
    nibs = ' — Two for his nibs! +2 to dealer.';
    checkWin(state.dealer, state, '2 for his nibs');
  }
  if (state.phase !== 'gameover') {
    state.phase = 'pegging';
    const nonDealer = state.dealer === 'p1' ? 'p2' : 'p1';
    state.pegging.current_turn = nonDealer;
  }
  state.log.push('Starter card: ' + starter.rank + starter.suit + nibs);
  saveState(roomCode, state);
  renderGame();
}

// ─── Pegging ──────────────────────────────────────────────────────────────────
function renderPegging() {
  document.getElementById('pegging-area').style.display = 'block';
  const pile = state.pegging.pile;
  const count = pile.reduce((s,c) => s + cardValue(c.rank), 0);
  document.getElementById('peg-count').textContent = count;

  document.getElementById('pegging-pile').innerHTML = pile.map(c => {
    const isRed = RED_SUITS.includes(c.suit);
    return `<div class="card${isRed?' red':''}" style="cursor:default;width:40px;height:58px;font-size:11px;padding:3px 4px;">
      <span>${c.rank}</span><span class="suit" style="font-size:13px;">${c.suit}</span><span class="bot">${c.rank}</span></div>`;
  }).join('');

  const pegHand = me().peg_hand || [];
  selectedPegCard = (selectedPegCard !== null && selectedPegCard < pegHand.length) ? selectedPegCard : null;
  document.getElementById('my-hand-peg').innerHTML = pegHand.map((c,i) => cardHTML(c, isMyTurn(), i, true)).join('');

  const playBtn = document.getElementById('btn-play-card');
  playBtn.disabled = !isMyTurn() || selectedPegCard === null;

  const goBtn = document.getElementById('btn-go');
  const myGo = state.pegging[myPlayer+'_said_go'];
  goBtn.disabled = !isMyTurn() || myGo;
  goBtn.textContent = myGo ? '✓ Said Go' : 'Say "Go"';

  document.getElementById('opp-hand-area').style.display = 'block';
  const oppPegHand = opp().peg_hand || [];
  document.getElementById('opp-hand-display').innerHTML = oppPegHand.map(() => faceDownCard()).join('');
}

async function playCard() {
  if (selectedPegCard === null || !isMyTurn()) return;
  const hand = [...me().peg_hand];
  const card = hand[selectedPegCard];
  const pile = state.pegging.pile;
  const count = pile.reduce((s,c) => s + cardValue(c.rank), 0);
  if (count + cardValue(card.rank) > 31) {
    alert('Cannot play that card — it would exceed 31! Say "Go" instead.');
    return;
  }
  pile.push(card);
  state[myPlayer].peg_hand = hand.filter((_,i) => i !== selectedPegCard);
  selectedPegCard = null;
  state.pegging.last_player = myPlayer;
  state.pegging.p1_said_go = false;
  state.pegging.p2_said_go = false;

  const newCount = pile.reduce((s,c) => s + cardValue(c.rank), 0);
  const pts = scorePegging(pile, myPlayer, state, newCount);
  state.log.push((me().name||myPlayer) + ' plays ' + card.rank + card.suit + ' · count: ' + newCount + (pts ? ' · +' + pts + ' pts' : ''));

  if (newCount === 31) {
    state.pegging.pile = [];
    state.pegging.p1_said_go = false;
    state.pegging.p2_said_go = false;
    state.pegging.last_player = null;
  }

  if (state.phase !== 'gameover') advancePegging(state);
  saveState(roomCode, state);
  renderGame();
}

async function sayGo() {
  if (!isMyTurn()) return;
  state.pegging[myPlayer+'_said_go'] = true;
  state.log.push((me().name||myPlayer) + ' says Go.');

  const oppHandEmpty = (state[oppKey()].peg_hand || []).length === 0;
  const oppSaidGo = state.pegging[oppKey()+'_said_go'];

  if (oppSaidGo || oppHandEmpty) {
    const lastPl = state.pegging.last_player;
    if (lastPl) {
      state[lastPl].score += 1;
      state.log.push('Go! ' + state[lastPl].name + ' gets 1 point.');
      checkWin(lastPl, state, 'Go');
    }
    state.pegging.pile = [];
    state.pegging.p1_said_go = false;
    state.pegging.p2_said_go = false;
    state.pegging.last_player = null;
    if (state.phase !== 'gameover') advancePegging(state);
  } else {
    state.pegging.current_turn = oppKey();
  }
  saveState(roomCode, state);
  renderGame();
}

function scorePegging(pile, player, s, count) {
  let pts = 0;
  if (count === 15) pts += 2;
  if (count === 31) pts += 2;
  const n = pile.length;
  if (n >= 2) {
    let pairs = 1;
    for (let i = n-2; i >= 0 && pile[i].rank === pile[n-1].rank; i--) pairs++;
    if (pairs === 2) pts += 2;
    else if (pairs === 3) pts += 6;
    else if (pairs === 4) pts += 12;
  }
  if (n >= 3) {
    for (let len = n; len >= 3; len--) {
      const seg = pile.slice(n-len).map(c => rankNum(c.rank)).sort((a,b)=>a-b);
      let isRun = true;
      for (let i = 1; i < seg.length; i++) if (seg[i] !== seg[i-1]+1) { isRun = false; break; }
      if (isRun) { pts += len; break; }
    }
  }
  if (pts > 0) { s[player].score += pts; checkWin(player, s, 'pegging'); }
  return pts;
}

function advancePegging(s) {
  const p1done = (s.p1.peg_hand || []).length === 0;
  const p2done = (s.p2.peg_hand || []).length === 0;
  if (p1done && p2done) {
    if (s.pegging.last_player) {
      s[s.pegging.last_player].score += 1;
      s.log.push((s[s.pegging.last_player].name||s.pegging.last_player) + ' gets 1 for last card.');
      checkWin(s.pegging.last_player, s, 'last card');
    }
    if (s.phase === 'gameover') return;
    s.phase = 'scoring';
    const nonDealer = s.dealer === 'p1' ? 'p2' : 'p1';
    s.score_phase = 'pone';
    s.score_subject = nonDealer;
    s.p1.hand = [...(s.p1.peg_hand || [])];
    s.p2.hand = [...(s.p2.peg_hand || [])];
    s.log.push('Pegging complete. Time to count hands!');
    return;
  }
  const cur = s.pegging.current_turn;
  const next = cur === 'p1' ? 'p2' : 'p1';
  if ((s[next].peg_hand || []).length > 0) s.pegging.current_turn = next;
  else if ((s[cur].peg_hand || []).length > 0) s.pegging.current_turn = cur;
}

// ─── Scoring ──────────────────────────────────────────────────────────────────
function renderScoring() {
  document.getElementById('scoring-area').style.display = 'block';
  const sp = state.score_phase;
  const subj = state.score_subject;
  if (!subj || !sp) return;

  let hand, label;
  if (sp === 'crib') {
    hand = state.crib;
    label = (state.dealer === myPlayer ? 'Your' : state[state.dealer].name + "'s") + ' crib';
  } else {
    hand = state[subj].hand;
    label = (subj === myPlayer ? 'Your' : state[subj].name + "'s") + ' hand';
  }

  document.getElementById('score-phase-label').textContent = label;
  document.getElementById('score-hand-display').innerHTML =
    (hand||[]).map(c => cardHTML(c, false, -1)).join('') +
    (state.starter ? cardHTML(state.starter, false, -1) : '');

  const result = countHand(hand || [], state.starter, sp === 'crib');
  const parts = [];
  if (result.fifteens > 0) parts.push('Fifteens: ' + result.fifteens);
  if (result.pairs > 0) parts.push('Pairs: ' + result.pairs);
  if (result.runs > 0) parts.push('Runs: ' + result.runs);
  if (result.flush > 0) parts.push('Flush: ' + result.flush);
  if (result.nobs > 0) parts.push('Nobs: ' + result.nobs);
  parts.push('Total: ' + result.total + ' pts');
  document.getElementById('score-breakdown').textContent = parts.join('  ·  ');

  const canContinue = sp === 'crib' ? isDealer() : subj === myPlayer;
  const continueBtn = document.getElementById('btn-score-continue');
  const waitMsg = document.getElementById('score-waiting');
  continueBtn.disabled = !canContinue;
  waitMsg.textContent = canContinue ? '' : 'Waiting for ' + (state[subj]?.name || 'opponent') + '...';
}

async function continueScoring() {
  const sp = state.score_phase;
  const subj = state.score_subject;
  const isCrib = sp === 'crib';
  const hand = isCrib ? state.crib : state[subj].hand;
  const result = countHand(hand, state.starter, isCrib);
  const scorer = isCrib ? state.dealer : subj;
  state[scorer].score += result.total;
  state.log.push((state[scorer].name||scorer) + ' scores ' + result.total + ' pts (' + sp + ').');
  checkWin(scorer, state, 'hand counting');
  if (state.phase === 'gameover') { saveState(roomCode, state); renderGame(); return; }

  if (sp === 'pone') {
    state.score_phase = 'dealer_hand';
    state.score_subject = state.dealer;
  } else if (sp === 'dealer_hand') {
    state.score_phase = 'crib';
    state.score_subject = state.dealer;
  } else if (sp === 'crib') {
    state.dealer = state.dealer === 'p1' ? 'p2' : 'p1';
    dealCards(state);
    state.log.push('New hand! Dealer is now ' + state[state.dealer].name + '.');
  }
  saveState(roomCode, state);
  renderGame();
}

function countHand(hand, starter, isCrib) {
  const cards = starter ? [...hand, starter] : [...hand];
  let fifteens = 0, pairs = 0, runs = 0, flush = 0, nobs = 0;
  const n = cards.length;
  for (let mask = 1; mask < (1 << n); mask++) {
    const sub = cards.filter((_,j) => mask & (1<<j));
    if (sub.reduce((s,c) => s + cardValue(c.rank), 0) === 15) fifteens += 2;
  }
  for (let i = 0; i < n-1; i++)
    for (let j = i+1; j < n; j++)
      if (cards[i].rank === cards[j].rank) pairs += 2;
  for (let len = 5; len >= 3; len--) {
    const cnt = countRuns(cards, len);
    if (cnt > 0) { runs += cnt * len; break; }
  }
  if (hand.length === 4) {
    if (hand.every(c => c.suit === hand[0].suit)) {
      flush = isCrib ? 0 : 4;
      if (starter && starter.suit === hand[0].suit) flush = 5;
    }
  }
  if (starter) {
    for (const c of hand) {
      if (c.rank === 'J' && c.suit === starter.suit) { nobs = 1; break; }
    }
  }
  return { fifteens, pairs, runs, flush, nobs, total: fifteens + pairs + runs + flush + nobs };
}

function countRuns(cards, len) {
  let count = 0;
  function combo(start, cur) {
    if (cur.length === len) {
      const nums = cur.map(c => rankNum(c.rank)).sort((a,b)=>a-b);
      for (let i = 1; i < nums.length; i++) if (nums[i] !== nums[i-1]+1) return;
      count++;
      return;
    }
    for (let i = start; i < cards.length; i++) combo(i+1, [...cur, cards[i]]);
  }
  combo(0, []);
  return count;
}

function checkWin(player, s, reason) {
  if (s[player].score >= 121) {
    s.phase = 'gameover';
    s.winner = player;
    s.log.push('🏆 ' + s[player].name + ' wins the game! (' + reason + ')');
  }
}

// ─── Game Over ────────────────────────────────────────────────────────────────
function showGameOver() {
  clearInterval(pollInterval);
  showScreen('gameover');
  const winner = state.winner;
  const isWinner = winner === myPlayer;
  document.getElementById('gameover-icon').textContent = isWinner ? '🏆' : '😔';
  document.getElementById('gameover-title').textContent = isWinner ? 'You Win!' : (state[winner]?.name || 'Opponent') + ' wins!';
  document.getElementById('gameover-score').textContent =
    state.p1.name + ': ' + state.p1.score + ' pts  ·  ' + state.p2.name + ': ' + state.p2.score + ' pts';
}

function resetToLobby() {
  clearInterval(pollInterval);
  state = null; roomCode = null; myPlayer = null; myName = null;
  selectedCards = []; selectedPegCard = null; lastStateStr = '';
  showScreen('lobby');
}

// Listen for storage changes from other tabs/windows
window.addEventListener('storage', (e) => {
  if (roomCode && e.key === 'crib_game_' + roomCode) {
    try {
      const s = JSON.parse(e.newValue);
      if (s) { state = s; renderGame(); }
    } catch(err) {}
  }
});
</script>
</body>
</html>

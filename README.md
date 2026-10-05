<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Crash Game Demo</title>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #090d18;
  color: white;
}

.header {
  height: 65px;
  background: #111827;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 18px;
  border-bottom: 1px solid #263044;
}

.logo {
  font-size: 23px;
  font-weight: bold;
  color: #ff4057;
}

.balance {
  background: #1d2638;
  padding: 10px 15px;
  border-radius: 20px;
  font-weight: bold;
}

.container {
  max-width: 1000px;
  margin: 20px auto;
  padding: 10px;
}

.game {
  position: relative;
  height: 430px;
  overflow: hidden;
  border-radius: 15px;
  background:
    radial-gradient(circle at 50% 80%, #263a66 0, #111a2d 35%, #080c15 75%);
  border: 1px solid #263044;
}

.grid {
  position: absolute;
  inset: 0;
  opacity: .15;
  background-image:
    linear-gradient(#8ca1c7 1px, transparent 1px),
    linear-gradient(90deg, #8ca1c7 1px, transparent 1px);
  background-size: 50px 50px;
}

.multiplier {
  position: absolute;
  top: 45%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: clamp(55px, 12vw, 105px);
  font-weight: bold;
  color: white;
  text-shadow: 0 0 30px #ff334d;
  z-index: 3;
}

.status {
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(0,0,0,.35);
  padding: 8px 18px;
  border-radius: 20px;
  z-index: 3;
}

.plane {
  position: absolute;
  font-size: 55px;
  left: 12%;
  bottom: 15%;
  transition: left .08s linear, bottom .08s linear;
  filter: drop-shadow(0 0 15px #ff4057);
  z-index: 2;
}

.controls {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
  margin-top: 15px;
}

.panel {
  background: #111827;
  border: 1px solid #263044;
  border-radius: 15px;
  padding: 18px;
}

label {
  display: block;
  color: #aab4c8;
  margin-bottom: 7px;
}

input {
  width: 100%;
  padding: 13px;
  border: none;
  border-radius: 8px;
  background: #202b40;
  color: white;
  font-size: 16px;
  outline: none;
}

button {
  width: 100%;
  padding: 14px;
  margin-top: 12px;
  border: none;
  border-radius: 9px;
  color: white;
  font-size: 17px;
  font-weight: bold;
  cursor: pointer;
}

.bet {
  background: #00a878;
}

.cashout {
  background: #e5394f;
}

button:disabled {
  opacity: .45;
  cursor: not-allowed;
}

.message {
  margin-top: 10px;
  min-height: 24px;
  color: #aab4c8;
}

.history {
  margin-top: 15px;
  background: #111827;
  border: 1px solid #263044;
  border-radius: 15px;
  padding: 18px;
}

.history h2 {
  margin-top: 0;
}

.rounds {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.round {
  background: #202b40;
  padding: 8px 12px;
  border-radius: 15px;
}

.green {
  color: #26d98b;
}

.red {
  color: #ff5267;
}

.notice {
  text-align: center;
  color: #7f8ba3;
  margin: 15px 0;
  font-size: 13px;
}

@media(max-width: 700px) {
  .controls {
    grid-template-columns: 1fr;
  }

  .game {
    height: 360px;
  }
}
</style>
</head>

<body>

<header class="header">
  <div class="logo">✈ CRASH DEMO</div>
  <div class="balance">
    Balance: KSh <span id="balance">10000.00</span>
  </div>
</header>

<div class="container">

  <div class="game">

    <div class="grid"></div>

    <div class="status" id="status">
      Waiting for next round...
    </div>

    <div class="multiplier" id="multiplier">
      1.00x
    </div>

    <div class="plane" id="plane">
      ✈️
    </div>

  </div>

  <div class="controls">

    <div class="panel">

      <label>Bet amount (demo KSh)</label>

      <input
        id="betAmount"
        type="number"
        value="100"
        min="1"
        step="1"
      >

      <button
        class="bet"
        id="betButton"
        onclick="placeBet()"
      >
        PLACE BET
      </button>

      <button
        class="cashout"
        id="cashoutButton"
        onclick="cashOut()"
        disabled
      >
        CASH OUT
      </button>

      <div class="message" id="message"></div>

    </div>

    <div class="panel">

      <h3>Current Round</h3>

      <p>
        Your bet:
        <strong>KSh <span id="currentBet">0.00</span></strong>
      </p>

      <p>
        Current multiplier:
        <strong id="currentMulti">1.00x</strong>
      </p>

      <p>
        Potential cash-out:
        <strong>KSh <span id="potential">0.00</span></strong>
      </p>

    </div>

  </div>

  <div class="history">

    <h2>Previous Crashes</h2>

    <div class="rounds" id="history"></div>

  </div>

  <div class="notice">
    Demo game only. All money shown is virtual and has no cash value.
  </div>

</div>

<script>

let balance = 10000;

let multiplier = 1.00;

let crashPoint = 0;

let running = false;

let playerBet = 0;

let cashedOut = false;

let interval = null;

const balanceEl = document.getElementById("balance");
const multiplierEl = document.getElementById("multiplier");
const currentMultiEl = document.getElementById("currentMulti");
const currentBetEl = document.getElementById("currentBet");
const potentialEl = document.getElementById("potential");
const messageEl = document.getElementById("message");
const statusEl = document.getElementById("status");
const planeEl = document.getElementById("plane");

const betButton = document.getElementById("betButton");
const cashoutButton = document.getElementById("cashoutButton");

const historyEl = document.getElementById("history");


// Generate a random crash point.
// This is intentionally generated before the round,
// but it is NOT exposed to the player.
function generateCrashPoint() {

  const r = Math.random();

  let point = 1 / (1 - r);

  point = Math.max(1.01, point);

  point = Math.min(point, 50);

  return Number(point.toFixed(2));
}


// Update balance display
function updateBalance() {
  balanceEl.textContent = balance.toFixed(2);
}


// Place virtual bet
function placeBet() {

  if (running) {
    messageEl.textContent = "Wait for the current round to finish.";
    return;
  }

  const amount =
    Number(document.getElementById("betAmount").value);

  if (!amount || amount <= 0) {
    messageEl.textContent = "Enter a valid amount.";
    return;
  }

  if (amount > balance) {
    messageEl.textContent = "Insufficient demo balance.";
    return;
  }

  balance -= amount;

  playerBet = amount;

  currentBetEl.textContent = amount.toFixed(2);

  updateBalance();

  messageEl.textContent =
    "Bet placed. Waiting for the round...";

  startRound();
}


// Start a round
function startRound() {

  running = true;

  multiplier = 1.00;

  crashPoint = generateCrashPoint();

  cashedOut = false;

  betButton.disabled = true;

  cashoutButton.disabled = false;

  statusEl.textContent = "Flying...";

  multiplierEl.textContent = "1.00x";

  currentMultiEl.textContent = "1.00x";

  potentialEl.textContent =
    playerBet.toFixed(2);

  let progress = 0;

  clearInterval(interval);

  interval = setInterval(() => {

    progress += 0.035;

    multiplier +=
      0.01 + multiplier * 0.0025;

    multiplier =
      Number(multiplier.toFixed(2));

    multiplierEl.textContent =
      multiplier.toFixed(2) + "x";

    currentMultiEl.textContent =
      multiplier.toFixed(2) + "x";

    potentialEl.textContent =
      (playerBet * multiplier).toFixed(2);

    const x =
      Math.min(80, 12 + progress * 2.8);

    const y =
      Math.min(75, 15 + progress * 1.8);

    planeEl.style.left = x + "%";
    planeEl.style.bottom = y + "%";

    if (multiplier >= crashPoint) {
      crash();
    }

  }, 80);
}


// Cash out
function cashOut() {

  if (!running || cashedOut || playerBet <= 0) {
    return;
  }

  const winnings =
    playerBet * multiplier;

  balance += winnings;

  cashedOut = true;

  updateBalance();

  messageEl.textContent =
    "Cashed out at " +
    multiplier.toFixed(2) +
    "x — KSh " +
    winnings.toFixed(2);

  cashoutButton.disabled = true;
}


// Crash
function crash() {

  clearInterval(interval);

  running = false;

  multiplier =
    Number(crashPoint.toFixed(2));

  multiplierEl.textContent =
    multiplier.toFixed(2) + "x";

  currentMultiEl.textContent =
    multiplier.toFixed(2) + "x";

  statusEl.textContent =
    "💥 CRASHED";

  if (!cashedOut && playerBet > 0) {

    messageEl.textContent =
      "Round crashed at " +
      crashPoint.toFixed(2) +
      "x. Your bet was lost.";

  }

  addHistory(crashPoint);

  playerBet = 0;

  currentBetEl.textContent = "0.00";

  potentialEl.textContent = "0.00";

  betButton.disabled = false;

  cashoutButton.disabled = true;

  setTimeout(() => {

    statusEl.textContent =
      "Waiting for next round...";

    multiplierEl.textContent =
      "1.00x";

    currentMultiEl.textContent =
      "1.00x";

    planeEl.style.left = "12%";
    planeEl.style.bottom = "15%";

  }, 1800);
}


// Add crash result to history
function addHistory(point) {

  const item =
    document.createElement("span");

  item.className = "round";

  if (point >= 2) {
    item.classList.add("green");
  } else {
    item.classList.add("red");
  }

  item.textContent =
    point.toFixed(2) + "x";

  historyEl.prepend(item);

  while (historyEl.children.length > 20) {
    historyEl.removeChild(historyEl.lastChild);
  }
}


// Initial balance
updateBalance();

</script>

</body>
</html>

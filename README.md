<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Aviator Demo B</title>

<style>
*{box-sizing:border-box}
body{
  margin:0;
  font-family:Arial,sans-serif;
  background:#090d14;
  color:white;
}
header{
  padding:18px;
  background:#111827;
  display:flex;
  justify-content:space-between;
  align-items:center;
}
.logo{font-size:23px;font-weight:bold}
.balance{
  background:#1f2937;
  padding:10px 14px;
  border-radius:10px;
}
main{max-width:650px;margin:auto;padding:15px}

.game{
  margin-top:15px;
  background:#111827;
  border-radius:18px;
  padding:20px;
  text-align:center;
  overflow:hidden;
}
.status{color:#9ca3af;margin-bottom:10px}
.multiplier{
  font-size:65px;
  font-weight:bold;
  margin:35px 0;
}
.plane{
  font-size:60px;
  transition:.2s;
}

.controls{
  background:#111827;
  margin-top:15px;
  padding:18px;
  border-radius:18px;
}
input{
  width:100%;
  padding:14px;
  border:0;
  border-radius:10px;
  background:#273244;
  color:white;
  font-size:18px;
  margin-bottom:12px;
}
button{
  width:100%;
  padding:14px;
  border:0;
  border-radius:10px;
  font-size:17px;
  font-weight:bold;
  margin-top:8px;
}
.bet{background:#22c55e;color:white}
.cash{background:#f59e0b;color:white}
button:disabled{opacity:.4}

.history{
  margin-top:15px;
  background:#111827;
  padding:18px;
  border-radius:18px;
}
.history h3{margin-top:0}
.row{
  display:flex;
  justify-content:space-between;
  padding:10px 0;
  border-bottom:1px solid #273244;
}
.demo{
  text-align:center;
  color:#9ca3af;
  font-size:13px;
  margin:18px;
}
</style>
</head>

<body>

<header>
  <div class="logo">✈️ AVIATOR DEMO</div>
  <div class="balance">
    💰 <span id="balance">1000.00</span>
  </div>
</header>

<main>

<div class="game">
  <div class="status" id="status">Place your virtual bet</div>

  <div class="plane" id="plane">✈️</div>

  <div class="multiplier" id="multiplier">1.00x</div>
</div>

<div class="controls">

  <input
    id="betAmount"
    type="number"
    min="1"
    value="50"
    placeholder="Virtual bet amount"
  >

  <button
    class="bet"
    id="betButton"
    onclick="placeBet()">
    BET
  </button>

  <button
    class="cash"
    id="cashButton"
    onclick="cashOut()"
    disabled>
    CASH OUT
  </button>

</div>

<div class="history">
  <h3>Round History</h3>
  <div id="history">
    <div class="row">
      <span>No rounds yet</span>
      <span>—</span>
    </div>
  </div>
</div>

<div class="demo">
  DEMO VERSION — virtual credits only.
</div>

</main>

<script>

let balance = 1000;
let multiplier = 1;
let crashPoint = 0;
let running = false;
let hasBet = false;
let betAmount = 0;
let timer;

const balanceEl = document.getElementById("balance");
const multiplierEl = document.getElementById("multiplier");
const statusEl = document.getElementById("status");
const planeEl = document.getElementById("plane");
const betButton = document.getElementById("betButton");
const cashButton = document.getElementById("cashButton");

function updateBalance(){
  balanceEl.textContent = balance.toFixed(2);
}

function placeBet(){

  if(running){
    statusEl.textContent = "Round already running";
    return;
  }

  betAmount = Number(
    document.getElementById("betAmount").value
  );

  if(!betAmount || betAmount <= 0){
    alert("Enter a valid virtual bet.");
    return;
  }

  if(betAmount > balance){
    alert("Not enough virtual credits.");
    return;
  }

  balance -= betAmount;
  updateBalance();

  hasBet = true;

  startRound();
}

function startRound(){

  running = true;
  multiplier = 1;

  /*
    Demo-only random crash point.
    This does NOT predict real Aviator rounds.
  */
  crashPoint =
    1.10 + Math.random() * 8.90;

  betButton.disabled = true;
  cashButton.disabled = false;

  statusEl.textContent = "Flying...";
  multiplierEl.textContent = "1.00x";

  timer = setInterval(() => {

    multiplier += 0.01;

    multiplierEl.textContent =
      multiplier.toFixed(2) + "x";

    planeEl.style.transform =
      "translateY(-" + multiplier * 5 + "px)";

    if(multiplier >= crashPoint){
      crash();
    }

  },100);
}

function cashOut(){

  if(!running || !hasBet) return;

  const winnings = betAmount * multiplier;

  balance += winnings;

  updateBalance();

  addHistory(
    "Cashed out",
    multiplier.toFixed(2) + "x"
  );

  clearInterval(timer);

  running = false;
  hasBet = false;

  betButton.disabled = false;
  cashButton.disabled = true;

  statusEl.textContent =
    "✅ Cashed out at " +
    multiplier.toFixed(2) + "x";

  planeEl.style.transform = "translateY(0)";
}

function crash(){

  clearInterval(timer);

  running = false;

  addHistory(
    "💥 Crash",
    multiplier.toFixed(2) + "x"
  );

  statusEl.textContent =
    "💥 CRASHED at " +
    multiplier.toFixed(2) + "x";

  multiplierEl.textContent =
    multiplier.toFixed(2) + "x";

  betButton.disabled = false;
  cashButton.disabled = true;

  hasBet = false;

  planeEl.style.transform = "translateY(0)";
}

function addHistory(result, value){

  const history =
    document.getElementById("history");

  const row =
    document.createElement("div");

  row.className = "row";

  row.innerHTML =
    "<span>" + result + "</span>" +
    "<span>" + value + "</span>";

  history.prepend(row);

  if(history.children.length > 10){
    history.removeChild(history.lastChild);
  }
}

updateBalance();

</script>

</body>
</html>

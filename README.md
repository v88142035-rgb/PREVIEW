<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>Speed Roulette Simulator</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
  body {
    background: #000;
    color: #fff;
    font-family: Arial, sans-serif;
    padding: 20px;
  }
  .red { color: red; font-weight: bold; }
  .black { color: black; background: #ccc; padding: 2px 4px; border-radius: 4px; }
  .green { color: #00ff66; font-weight: bold; }
  .history {
    max-height: 250px;
    overflow-y: auto;
    background: #111;
    padding: 10px;
    border: 1px solid #333;
  }
  #status {
    font-weight: bold;
    padding: 5px 12px;
    border-radius: 6px;
  }
  canvas { background: #111; margin-top: 15px; }
</style>
</head>

<body>

<h1>Speed Roulette</h1>

<p>
Стратегія:
<select id="strategySelect">
  <option value="martingale">Martingale</option>
  <option value="anti">Anti-Martingale</option>
  <option value="fibonacci">Fibonacci</option>
  <option value="colorstreak">Color Streak</option>
</select>
</p>

<p>Баланс: <strong><span id="balance">55000</span></strong></p>

<p>
Статус стратегії:
<span id="status">СТАРТ</span>
</p>

<p>Раунди: <span id="rounds">0</span> |
Виграші: <span id="wins">0</span> |
Програші: <span id="losses">0</span></p>

<canvas id="balanceChart" width="800" height="280"></canvas>

<h3>Історія спінів</h3>
<div class="history" id="history"></div>

<script>
// ---------------- CONFIG ----------------
const startBalance = 1000;
let balance = startBalance;
let rounds = 0, wins = 0, losses = 0;
const baseBet = 10;
let currentBet = baseBet;
let targetColor = "red";
let lastColors = [];
let balanceHistory = [balance];

// ---------------- ROULETTE ----------------
function spinRoulette() {
  const redNums = new Set([1,3,5,7,9,12,14,16,18,19,21,23,25,27,30,32,34,36]);
  let num = Math.floor(Math.random() * 37);
  let color = num === 0 ? "green" : redNums.has(num) ? "red" : "black";
  return { num, color };
}

// ---------------- STRATEGIES ----------------
function martingale(win) {
  currentBet = win === false ? currentBet * 2 : baseBet;
  targetColor = "red";
}

function antiMartingale(win) {
  currentBet = win ? currentBet * 2 : baseBet;
  targetColor = "red";
}

let fib = [baseBet, baseBet], fibIndex = 0;
function fibonacci(win) {
  if (win) fibIndex = Math.max(0, fibIndex - 2);
  else {
    fibIndex++;
    if (fibIndex >= fib.length)
      fib.push(fib[fib.length - 1] + fib[fib.length - 2]);
  }
  currentBet = fib[fibIndex];
  targetColor = "red";
}

function colorStreak(color) {
  lastColors.push(color);
  if (lastColors.length > 5) lastColors.shift();

  if (lastColors.slice(-3).every(c => c === "red")) targetColor = "black";
  else if (lastColors.slice(-3).every(c => c === "black")) targetColor = "red";
  else targetColor = "red";

  currentBet = baseBet;
}

// ---------------- STATUS ----------------
function updateStatus() {
  const el = document.getElementById("status");

  if (balance > startBalance) {
    el.textContent = "ВИЩЕ СТАРТУ";
    el.style.color = "#00ff99";
  } else if (balance < startBalance) {
    el.textContent = "В МІНУСІ";
    el.style.color = "#ff4d4d";
  } else {
    el.textContent = "РІВНО СТАРТ";
    el.style.color = "#ffffff";
  }
}

// ---------------- CHART ----------------
const chart = new Chart(
  document.getElementById("balanceChart"),
  {
    type: "line",
    data: {
      labels: [0],
      datasets: [
        {
          label: "Баланс",
          data: balanceHistory,
          borderColor: "cyan",
          borderWidth: 2,
          tension: 0.3
        },
        {
          label: "Старт",
          data: [startBalance],
          borderColor: "#555",
          borderDash: [6,6],
          pointRadius: 0
        }
      ]
    },
    options: {
      plugins: { legend: { labels: { color: "#fff" } } },
      scales: {
        x: { ticks: { color: "#fff" } },
        y: { ticks: { color: "#fff" } }
      }
    }
  }
);

// ---------------- GAME LOOP ----------------
function playRound() {
  const strategy = document.getElementById("strategySelect").value;
  const spin = spinRoulette();
  const win = spin.color === targetColor;

  balance += win ? currentBet : -currentBet;

  if (strategy === "martingale") martingale(win);
  if (strategy === "anti") antiMartingale(win);
  if (strategy === "fibonacci") fibonacci(win);
  if (strategy === "colorstreak") colorStreak(spin.color);

  rounds++;
  win ? wins++ : losses++;

  balanceHistory.push(balance);
  chart.data.labels.push(rounds);
  chart.data.datasets[1].data.push(startBalance);
  chart.update();

  document.getElementById("balance").textContent = balance;
  document.getElementById("rounds").textContent = rounds;
  document.getElementById("wins").textContent = wins;
  document.getElementById("losses").textContent = losses;

  updateStatus();

  const h = document.createElement("div");
  h.innerHTML = `#${rounds} → <span class="${spin.color}">${spin.num}</span> | ставка ${currentBet} | ${win ? "WIN" : "LOSE"}`;
  document.getElementById("history").prepend(h);

  setTimeout(playRound, 27000 + Math.random() * 5000);
}

// ---------------- START ----------------
updateStatus();
playRound();
</script>

</body>
</html>

<!doctype html>
<html lang="pl">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Klawiaturowe klikanie z rankingiem</title>
<style>
body{font-family:system-ui,Segoe UI,Roboto,Arial;display:flex;flex-direction:column;align-items:center;padding:20px;gap:16px;background:#f9f9fc}
.panel{background:#fff;padding:20px;border-radius:12px;box-shadow:0 4px 14px rgba(0,0,0,0.05);width:100%;max-width:600px}
h1{margin:0 0 12px}
input,button{padding:8px 12px;border-radius:8px;border:1px solid #ccc;font-size:16px}
button{cursor:pointer}
#gameArea{margin-top:12px;padding:12px;border-radius:8px;background:#f4f4f9;text-align:center}
#lastKey{font-size:28px;font-weight:bold;margin-top:6px}
table{width:100%;border-collapse:collapse;margin-top:10px}
th,td{padding:6px;border-bottom:1px solid #ddd;text-align:left}
</style>
</head>
<body>

<div class="panel">
  <h1>Klawiaturowe klikanie — Ranking</h1>
  <div>
    <label>Twoje imię: <input id="playerName" placeholder="Wpisz imię"></label>
    <button id="startBtn">Start (10s)</button>
  </div>
  <div id="gameArea">
    <div>Czas: <span id="timeLeft">—</span> s</div>
    <div>Wynik: <span id="score">0</span></div>
    <div>Ostatni klawisz: <span id="lastKey">—</span></div>
  </div>

  <h2>Ranking</h2>
  <table>
    <thead><tr><th>Imię</th><th>Wynik</th></tr></thead>
    <tbody id="ranking"></tbody>
  </table>
</div>

<script>
let score = 0;
let timeLeft = 0;
let timer = null;
let gameActive = false;

// odtwarzanie kliknięcia
function playClick(){
  const ctx = new (window.AudioContext||window.webkitAudioContext)();
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  osc.type="square";
  osc.frequency.setValueAtTime(800, ctx.currentTime);
  gain.gain.setValueAtTime(0.1, ctx.currentTime);
  osc.connect(gain).connect(ctx.destination);
  osc.start();
  osc.stop(ctx.currentTime+0.05);
}

// ładowanie rankingu
function loadRanking(){
  return JSON.parse(localStorage.getItem("ranking") || "[]");
}

// zapisywanie rankingu
function saveRanking(ranking){
  localStorage.setItem("ranking", JSON.stringify(ranking));
}

// odświeżanie tabeli
function updateRankingTable(){
  const ranking = loadRanking().sort((a,b)=>b.score-a.score).slice(0,10);
  const tbody = document.getElementById("ranking");
  tbody.innerHTML = ranking.map(r=>`<tr><td>${r.name}</td><td>${r.score}</td></tr>`).join("");
}

// start gry
document.getElementById("startBtn").addEventListener("click",()=>{
  const name = document.getElementById("playerName").value.trim();
  if(!name){ alert("Podaj imię!"); return; }
  score = 0;
  timeLeft = 10;
  document.getElementById("score").textContent = score;
  document.getElementById("timeLeft").textContent = timeLeft;
  document.getElementById("lastKey").textContent = "—";
  gameActive = true;
  timer = setInterval(()=>{
    timeLeft--;
    document.getElementById("timeLeft").textContent = timeLeft;
    if(timeLeft<=0){
      clearInterval(timer);
      gameActive = false;
      // zapisz wynik
      const ranking = loadRanking();
      ranking.push({name, score});
      saveRanking(ranking);
      updateRankingTable();
      alert(`Koniec gry! Twój wynik: ${score}`);
    }
  },1000);
});

// obsługa klawiatury
window.addEventListener("keydown",(e)=>{
  if(gameActive){
    score++;
    document.getElementById("score").textContent = score;
    document.getElementById("lastKey").textContent = e.key;
    playClick();
  }
});

// inicjalizacja
updateRankingTable();
</script>

</body>
</html>

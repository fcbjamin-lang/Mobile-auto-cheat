# Mobile-auto-cheat
<!DOCTYPE html>

<html lang="de">

<head>

<meta charset="UTF-8">

<meta name="viewport" content="width=device-width, initial-scale=1">

<meta name="theme-color" content="#111827">

<title>Fortnite Loot Companion</title>

<style>

* {

  box-sizing: border-box;

}

body {

  margin: 0;

  padding: 20px;

  background: #101522;

  color: white;

  font-family: Arial, sans-serif;

}

.container {

  max-width: 600px;

  margin: auto;

}

h1 {

  text-align: center;

  color: #f5b82e;

}

.subtitle {

  text-align: center;

  color: #aab3c5;

}

.card {

  background: #1d2738;

  border-radius: 15px;

  padding: 18px;

  margin: 15px 0;

}

.stats {

  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 12px;

}

.stat {

  background: #29364b;

  padding: 16px;

  border-radius: 12px;

  text-align: center;

}

.number {

  font-size: 30px;

  font-weight: bold;

  color: #ffc247;

  margin-top: 8px;

}

button {

  border: none;

  border-radius: 10px;

  padding: 13px;

  margin: 5px 0;

  width: 100%;

  font-size: 16px;

  font-weight: bold;

  cursor: pointer;

}

.start { background: #28b76b; color: white; }

.stop { background: #e34b4b; color: white; }

.action { background: #f5b82e; color: #171717; }

.reset { background: #526078; color: white; }

input, textarea {

  width: 100%;

  padding: 12px;

  margin: 7px 0;

  border-radius: 8px;

  border: 1px solid #526078;

  background: #111827;

  color: white;

  font-size: 16px;

}

#events {

  max-height: 220px;

  overflow-y: auto;

  font-size: 14px;

  color: #d0d7e5;

}

.event {

  border-bottom: 1px solid #39465a;

  padding: 9px 0;

}

small {

  color: #aab3c5;

}

</style>

</head>

<body>

<div class="container">

<h1>🎮 Fortnite Loot Companion</h1>

<p class="subtitle">Dein persönlicher Loot-Tracker</p>

<div class="card">

  <h2>📊 Statistiken</h2>

  <div class="stats">

    <div class="stat">

      Truhen

      <div class="number" id="chests">0</div>

    </div>

    <div class="stat">

      Loot

      <div class="number" id="loot">0</div>

    </div>

    <div class="stat">

      Sessions

      <div class="number" id="sessions">0</div>

    </div>

    <div class="stat">

      Spielzeit

      <div class="number" id="timer">00:00</div>

    </div>

  </div>

</div>

<div class="card">

  <h2>⏱️ Session</h2>

  <p id="status">Status: Nicht gestartet</p>

  <button class="start" onclick="startSession()">▶ Start</button>

  <button class="stop" onclick="stopSession()">⏹ Stop</button>

</div>

<div class="card">

  <h2>🎁 Truhen-Tracker</h2>

  <button class="action" onclick="addChest()">+ Truhe gefunden</button>

</div>

<div class="card">

  <h2>💎 Loot-Tracker</h2>

  <input id="lootName" placeholder="Loot-Name (z. B. Schildtrank)">

  <button class="action" onclick="addLoot()">+ Loot hinzufügen</button>

</div>

<div class="card">

  <h2>🗺️ Routen-Notizen</h2>

  <textarea id="route" rows="4"

    placeholder="Notiere deine Lieblings-Loot-Route..."></textarea>

  <button class="action" onclick="saveRoute()">Notizen speichern</button>

  <p id="routeStatus"></p>

</div>

<div class="card">

  <h2>📜 Ereignisse</h2>

  <div id="events">Noch keine Ereignisse.</div>

  <button class="action" onclick="exportCSV()">📥 CSV exportieren</button>

</div>

<div class="card">

  <button class="reset" onclick="resetData()">Statistiken zurücksetzen</button>

  <small>Deine Daten werden lokal auf diesem Gerät gespeichert.</small>

</div>

</div>

<script>

let data = JSON.parse(localStorage.getItem("lootData")) || {

  chests: 0,

  loot: 0,

  sessions: 0,

  seconds: 0,

  events: [],

  route: ""

};

let running = false;

let startTime = null;

let interval = null;

function save() {

  localStorage.setItem("lootData", JSON.stringify(data));

}

function render() {

  document.getElementById("chests").textContent = data.chests;

  document.getElementById("loot").textContent = data.loot;

  document.getElementById("sessions").textContent = data.sessions;

  let total = data.seconds;

  if (running && startTime) {

    total += Math.floor((Date.now() - startTime) / 1000);

  }

  let minutes = Math.floor(total / 60);

  let seconds = total % 60;

  document.getElementById("timer").textContent =

    String(minutes).padStart(2, "0") + ":" +

    String(seconds).padStart(2, "0");

  document.getElementById("status").textContent =

    running ? "Status: 🟢 Session läuft" : "Status: 🔴 Gestoppt";

  const events = document.getElementById("events");

  events.innerHTML = "";

  if (data.events.length === 0) {

    events.textContent = "Noch keine Ereignisse.";

  }

  data.events.slice().reverse().forEach(event => {

    const div = document.createElement("div");

    div.className = "event";

    div.textContent = event;

    events.appendChild(div);

  });

}

function logEvent(message) {

  data.events.push(

    new Date().toLocaleString("de-DE") + " – " + message

  );

  save();

  render();

}

function startSession() {

  if (running) return;

  running = true;

  startTime = Date.now();

  data.sessions++;

  logEvent("Session gestartet");

  interval = setInterval(render, 1000);

  render();

}

function stopSession() {

  if (!running) return;

  data.seconds += Math.floor((Date.now() - startTime) / 1000);

  running = false;

  startTime = null;

  clearInterval(interval);

  logEvent("Session beendet");

  save();

  render();

}

function addChest() {

  data.chests++;

  logEvent("Truhe gefunden");

}

function addLoot() {

  const name = document.getElementById("lootName").value.trim();

  if (!name) {

    alert("Bitte einen Loot-Namen eingeben.");

    return;

  }

  data.loot++;

  logEvent("Loot aufgenommen: " + name);

  document.getElementById("lootName").value = "";

}

function saveRoute() {

  data.route = document.getElementById("route").value;

  save();

  document.getElementById("routeStatus").textContent =

    "✅ Notizen gespeichert!";

}

function exportCSV() {

  const rows = [

    ["Ereignisse"],

    ...data.events.map(event => [event])

  ];

  const csv = rows.map(row =>

    row.map(value =>

      '"' + String(value).replaceAll('"', '""') + '"'

    ).join(";")

  ).join("\n");

  const blob = new Blob(["\uFEFF" + csv], {

    type: "text/csv;charset=utf-8"

  });

  const url = URL.createObjectURL(blob);

  const a = document.createElement("a");

  a.href = url;

  a.download = "fortnite-loot-statistik.csv";

  a.click();

  URL.revokeObjectURL(url);

}

function resetData() {

  if (!confirm("Wirklich alle Statistiken zurücksetzen?")) return;

  if (running) {

    clearInterval(interval);

    running = false;

    startTime = null;

  }

  data = {

    chests: 0,

    loot: 0,

    sessions: 0,

    seconds: 0,

    events: [],

    route: ""

  };

  save();

  document.getElementById("route").value = "";

  render();

}

document.getElementById("route").value = data.route;

render();

</script>

</body>

</html>
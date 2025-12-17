<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>Real-Time Multithreaded Simulator</title>
<style>
body{font-family:Arial;margin:20px;background:#f2f2f7}
.panel{background:#fff;padding:12px;border-radius:8px;margin-top:12px;box-shadow:0 0 10px #ccc}
input,select{padding:6px;border:1px solid #ccc;border-radius:6px}
button{padding:6px 12px;border:none;border-radius:6px;cursor:pointer}
button.start{background:#2563eb;color:#fff}
table{width:100%;border-collapse:collapse;margin-top:10px}
th,td{padding:6px;border-bottom:1px solid #eee;font-size:14px}
.log{background:#111;color:#9ae6ff;height:150px;overflow:auto;padding:8px;font-family:monospace;font-size:13px;border-radius:6px}
.chip{background:#eef1ff;padding:6px 10px;border-radius:20px;margin-right:8px;font-size:13px;display:inline-block}
</style>
</head>
<body>

<h2>Real-Time Multithreaded Application Simulator</h2>

<div class="panel">
<select id="scheduler">
<option value="fcfs">FCFS</option>
<option value="priority">Priority</option>
<option value="rr">Round Robin</option>
</select>
<input id="quantum" type="number" value="1000" style="display:none;width:120px">
<button class="start" id="startBtn">Start</button>
<button id="pauseBtn">Pause</button>
<button id="resetBtn">Reset</button>
</div>
<div class="panel">
<h4>Add Thread</h4>
<input id="burst" type="number" placeholder="Burst (ms)" value="3000">
<input id="arrival" type="number" placeholder="Arrival (ms)" value="0">
<input id="priorityT" type="number" placeholder="Priority" value="1">
<button id="addThread">Add</button>

<table>
<thead><tr><th>ID</th><th>Burst</th><th>Remain</th><th>Arrival</th><th>Priority</th><th>State</th></tr></thead>
<tbody id="tbody"></tbody>
</table>
</div>

<div class="panel">
<h4>Log</h4>
<div id="log" class="log"></div>
<div class="chip">Tick: <span id="tick">0</span></div>
<div class="chip">CPU: <span id="cpu">idle</span></div>
<div class="chip">Done: <span id="done">0</span></div>
</div>

<div class="panel">
<h4>Stats</h4>
<div id="stats">Add threads and start simulation.</div>
</div>

<script>
let T=200,clock=0,threads=[],cpu=null,done=0,ready=[],loop=null;
function show(t){document.getElementById('log').innerHTML+=t+"\n"}
function render(){
let tb=document.getElementById('tbody');tb.innerHTML="";
threads.forEach(x=>{
tb.innerHTML+=`<tr><td>T${x.id}</td><td>${x.burst}</td><td>${x.rem}</td><td>${x.arr}</td><td>${x.pri}</td><td>${x.state}</td></tr>`;
});
}
document.getElementById('scheduler').onchange=function(){
document.getElementById('quantum').style.display=this.value=="rr"?"inline-block":"none";
};
document.getElementById('addThread').onclick=()=>{
let b=parseInt(burst.value),a=parseInt(arrival.value),p=parseInt(priorityT.value);
let id=threads.length+1;
threads.push({id,burst:b,rem:b,arr:a,pri:p,state:a==0?"ready":"new",st:null,en:null,wait:0});
render();show("T"+id+" added");
};

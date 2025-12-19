<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Real-Time Multithreading Simulator</title>
  <style>
    body { font-family: Arial, sans-serif; background:#f4f6f8; margin:0; padding:20px; }
    h1 { text-align:center; }
    .container { max-width:1000px; margin:auto; }
    .controls, .panel { background:#fff; padding:15px; margin-bottom:15px; border-radius:8px; box-shadow:0 2px 6px rgba(0,0,0,0.1); }
    button { padding:8px 14px; margin:5px; cursor:pointer; }
    select { padding:6px; }
    .threads { display:flex; flex-wrap:wrap; gap:10px; }
    .thread { width:120px; padding:10px; border-radius:6px; text-align:center; color:#fff; }
    .new { background:#6c757d; }
    .ready { background:#0d6efd; }
    .running { background:#198754; }
    .blocked { background:#dc3545; }
    .terminated { background:#212529; }
    .log { height:150px; overflow:auto; background:#000; color:#0f0; padding:10px; font-size:13px; }
  </style>
</head>
<body>
  <div class="container">
    <h1>Real-Time Multi-threaded Application Simulator</h1><div class="controls">
  <label>Threading Model: </label>
  <select id="model">
    <option value="many-one">Many-to-One</option>
    <option value="one-many">One-to-Many</option>
    <option value="many-many">Many-to-Many</option>
  </select>
  <button onclick="createThread()">Create Thread</button>
  <button onclick="runScheduler()">Run Scheduler</button>
  <button onclick="semaphoreWait()">Semaphore Wait</button>
  <button onclick="semaphoreSignal()">Semaphore Signal</button>
  <button onclick="resetSim()">Reset</button>
</div>

<div class="panel">
  <h3>Threads</h3>
  <div class="threads" id="threadArea"></div>
</div>

<div class="panel">
  <h3>System Log</h3>
  <div class="log" id="log"></div>
</div>

  </div><script>
  let threads = [];
  let threadId = 1;
  let semaphore = 1;

  function log(msg) {
    const logBox = document.getElementById('log');
    logBox.innerHTML += msg + '<br>';
    logBox.scrollTop = logBox.scrollHeight;
  }

  function createThread() {
    const t = { id: threadId++, state: 'new' };
    threads.push(t);
    log('Thread T' + t.id + ' created (NEW)');
    render();
  }

  function runScheduler() {
    const model = document.getElementById('model').value;
    threads.forEach(t => {
      if (t.state === 'new') t.state = 'ready';
    });
    
    let runnable = threads.filter(t => t.state === 'ready');

    if (model === 'many-one' && runnable.length > 0) {
      runnable[0].state = 'running';
      log('Many-to-One: Thread T' + runnable[0].id + ' is RUNNING');
    }

    if (model === 'one-many') {
      runnable.forEach(t => {
        t.state = 'running';
        log('One-to-Many: Thread T' + t.id + ' is RUNNING on kernel thread');
      });
    }

    if (model === 'many-many') {
      runnable.slice(0,2).forEach(t => {
        t.state = 'running';
        log('Many-to-Many: Thread T' + t.id + ' mapped to kernel thread');
      });
    }

    render();
  }

  function semaphoreWait() {
    if (semaphore > 0) {
      semaphore--;
      log('Semaphore acquired');
    } else {
      let running = threads.find(t => t.state === 'running');
      if (running) {
        running.state = 'blocked';
        log('Thread T' + running.id + ' BLOCKED (Semaphore wait)');
      }
    }
    render();
  }

  function semaphoreSignal() {
    semaphore++;
    let blocked = threads.find(t => t.state === 'blocked');
    if (blocked) {
      blocked.state = 'ready';
      log('Thread T' + blocked.id + ' moved to READY (Semaphore signal)');
    } else {
      log('Semaphore released');
    }
    render();
  }

  function resetSim() {
    threads = [];
    threadId = 1;
    semaphore = 1;
    document.getElementById('log').innerHTML = '';
    render();
  }
    function render() {
    const area = document.getElementById('threadArea');
    area.innerHTML = '';
    threads.forEach(t => {
      const div = document.createElement('div');
      div.className = 'thread ' + t.state;
      div.innerHTML = 'T' + t.id + '<br>' + t.state.toUpperCase();
      area.appendChild(div);
    });
  }
</script></body>
</html>

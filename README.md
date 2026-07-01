<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bench Clock</title>
<style>
  :root{
    --panel: #14110d;
    --panel-edge: #2a251d;
    --amber: #ffb347;
    --amber-dim: #7a5726;
    --amber-glow: rgba(255,179,71,0.55);
    --paper: #ece4d3;
    --ink: #2a2118;
    --brass: #b8894f;
    --brass-dark: #7d5c33;
    --mono: "Courier New", ui-monospace, SFMono-Regular, monospace;
    --serif: Georgia, "Times New Roman", serif;
  }

  *{ box-sizing: border-box; }

  body{
    margin:0;
    min-height:100vh;
    display:flex;
    align-items:center;
    justify-content:center;
    background:
      radial-gradient(circle at 50% 0%, #2b2620 0%, #100d09 70%);
    font-family: var(--serif);
    color: var(--paper);
    padding: 24px;
  }

  .instrument{
    width: min(560px, 100%);
    background: linear-gradient(180deg, #1b1712, #0e0c09);
    border: 1px solid var(--panel-edge);
    border-radius: 18px;
    padding: 28px 28px 22px;
    box-shadow:
      0 30px 60px rgba(0,0,0,0.55),
      inset 0 1px 0 rgba(255,255,255,0.04);
    position: relative;
  }

  .instrument::before{
    content:"";
    position:absolute; inset:10px;
    border:1px solid rgba(184,137,79,0.18);
    border-radius: 12px;
    pointer-events:none;
  }

  .plate-label{
    display:flex;
    justify-content:space-between;
    align-items:baseline;
    font-family: var(--mono);
    font-size: 11px;
    letter-spacing: 2px;
    color: var(--brass);
    text-transform: uppercase;
    margin-bottom: 18px;
    padding: 0 4px;
  }

  .plate-label span:last-child{ color: var(--amber-dim); }

  .display{
    background: var(--panel);
    border-radius: 10px;
    padding: 34px 18px 26px;
    text-align:center;
    box-shadow:
      inset 0 3px 10px rgba(0,0,0,0.7),
      inset 0 0 0 1px #000;
    position: relative;
    overflow: hidden;
  }

  .display::after{
    content:"";
    position:absolute; inset:0;
    background: repeating-linear-gradient(
      0deg,
      rgba(0,0,0,0.15) 0px,
      rgba(0,0,0,0.15) 1px,
      transparent 1px,
      transparent 3px
    );
    pointer-events:none;
  }

  #time{
    font-family: var(--mono);
    font-weight: 700;
    font-size: clamp(2.6rem, 11vw, 4.6rem);
    letter-spacing: 4px;
    color: var(--amber);
    text-shadow:
      0 0 6px var(--amber-glow),
      0 0 22px var(--amber-glow);
    line-height: 1;
    user-select: none;
  }

  #time .colon{
    animation: blink 1s steps(1) infinite;
  }

  #time.paused-blink .colon{ animation: none; }

  @keyframes blink{
    50% { opacity: 0.15; }
  }

  #meridiem{
    display:inline-block;
    font-family: var(--mono);
    font-size: 0.9rem;
    letter-spacing: 3px;
    color: var(--amber-dim);
    margin-top: 10px;
    min-height: 1em;
  }

  #meridiem.active{ color: var(--amber); text-shadow: 0 0 8px var(--amber-glow); }

  #date-line{
    font-family: var(--serif);
    font-style: italic;
    font-size: 0.85rem;
    color: rgba(236,228,211,0.55);
    text-align:center;
    margin-top: 16px;
    letter-spacing: 0.5px;
  }

  .controls{
    margin-top: 22px;
    display:flex;
    flex-direction: column;
    gap: 14px;
  }

  .row{
    display:flex;
    align-items:center;
    justify-content: space-between;
    gap: 12px;
  }

  .row-label{
    font-family: var(--mono);
    font-size: 11px;
    letter-spacing: 1.5px;
    text-transform: uppercase;
    color: var(--brass);
    white-space: nowrap;
  }

  /* Toggle switch styled like a rocker switch */
  .rocker{
    position: relative;
    width: 92px;
    height: 34px;
    background: #0a0806;
    border-radius: 6px;
    border: 1px solid var(--panel-edge);
    cursor: pointer;
    display:flex;
    align-items:center;
    font-family: var(--mono);
    font-size: 11px;
    color: var(--amber-dim);
    user-select:none;
    box-shadow: inset 0 2px 4px rgba(0,0,0,0.6);
  }

  .rocker span{
    flex:1;
    text-align:center;
    z-index:2;
    transition: color 0.2s ease;
  }

  .rocker .thumb{
    position:absolute;
    top: 3px; bottom: 3px;
    width: 44px;
    background: linear-gradient(180deg, #3a3125, #221c14);
    border: 1px solid var(--brass-dark);
    border-radius: 4px;
    left: 3px;
    transition: left 0.2s ease;
    box-shadow: 0 1px 3px rgba(0,0,0,0.5);
  }

  .rocker[data-mode="24"] .thumb{ left: 45px; }
  .rocker[data-mode="12"] span:first-child,
  .rocker[data-mode="24"] span:last-child{ color: var(--amber); }

  /* Interval control - brass knob with numeric input */
  .interval-control{
    display:flex;
    align-items:center;
    gap: 10px;
  }

  #interval-input{
    width: 64px;
    background: #0a0806;
    border: 1px solid var(--panel-edge);
    border-radius: 5px;
    color: var(--amber);
    font-family: var(--mono);
    font-size: 13px;
    text-align:center;
    padding: 6px 4px;
    box-shadow: inset 0 2px 4px rgba(0,0,0,0.6);
  }

  #interval-input:focus{
    outline: none;
    border-color: var(--brass);
  }

  .unit{
    font-family: var(--mono);
    font-size: 11px;
    color: var(--brass);
  }

  .stepper{
    display:flex;
    flex-direction: column;
    border-radius: 4px;
    overflow: hidden;
    border: 1px solid var(--panel-edge);
  }

  .stepper button{
    width: 22px;
    height: 15px;
    background: #1c1712;
    color: var(--amber-dim);
    border: none;
    cursor: pointer;
    font-family: var(--mono);
    font-size: 10px;
    line-height: 1;
  }

  .stepper button:hover{ background: #262019; color: var(--amber); }
  .stepper button:active{ background: #302819; }

  .status-line{
    font-family: var(--mono);
    font-size: 10px;
    letter-spacing: 1px;
    color: rgba(184,137,79,0.5);
    text-align:center;
    margin-top: 4px;
  }

  .led{
    display:inline-block;
    width: 6px; height: 6px;
    border-radius: 50%;
    background: var(--amber);
    box-shadow: 0 0 6px var(--amber-glow);
    margin-right: 6px;
    vertical-align: middle;
    animation: pulse 1.4s ease-in-out infinite;
  }

  @keyframes pulse{
    0%, 100% { opacity: 1; }
    50% { opacity: 0.3; }
  }

  @media (prefers-reduced-motion: reduce){
    #time .colon{ animation: none; }
    .led{ animation: none; }
  }

  @media (max-width: 420px){
    .row{ flex-wrap: wrap; }
  }
</style>
</head>
<body>

<div class="instrument">
  <div class="plate-label">
    <span>Bench Clock — Mk.I</span>
    <span id="tick-rate-label">1000ms</span>
  </div>

  <div class="display">
    <div id="time">--:--:--</div>
    <div id="meridiem"></div>
  </div>

  <div id="date-line">loading date…</div>

  <div class="controls">
    <div class="row">
      <span class="row-label">Format</span>
      <div class="rocker" id="format-toggle" data-mode="12" role="button" tabindex="0" aria-label="Toggle 12 or 24 hour format">
        <div class="thumb"></div>
        <span>12H</span>
        <span>24H</span>
      </div>
    </div>

    <div class="row">
      <span class="row-label">Refresh interval</span>
      <div class="interval-control">
        <input type="number" id="interval-input" value="1000" min="50" step="50" aria-label="Update interval in milliseconds">
        <span class="unit">ms</span>
        <div class="stepper">
          <button id="step-up" aria-label="Increase interval">▲</button>
          <button id="step-down" aria-label="Decrease interval">▼</button>
        </div>
      </div>
    </div>
  </div>

  <div class="status-line"><span class="led"></span><span id="status-text">running · updates every 1000ms</span></div>
</div>

<script>
(function(){
  const timeEl = document.getElementById('time');
  const meridiemEl = document.getElementById('meridiem');
  const dateEl = document.getElementById('date-line');
  const formatToggle = document.getElementById('format-toggle');
  const intervalInput = document.getElementById('interval-input');
  const stepUp = document.getElementById('step-up');
  const stepDown = document.getElementById('step-down');
  const statusText = document.getElementById('status-text');
  const tickLabel = document.getElementById('tick-rate-label');

  let use24Hour = false;
  let intervalMs = 1000;
  let timerId = null;

  function pad(n){ return String(n).padStart(2, '0'); }

  function render(){
    const now = new Date();
    let hours = now.getHours();
    const minutes = pad(now.getMinutes());
    const seconds = pad(now.getSeconds());
    let meridiem = '';

    if (use24Hour){
      hours = pad(hours);
    } else {
      meridiem = hours >= 12 ? 'PM' : 'AM';
      hours = hours % 12;
      if (hours === 0) hours = 12;
      hours = pad(hours);
    }

    timeEl.innerHTML = `${hours}<span class="colon">:</span>${minutes}<span class="colon">:</span>${seconds}`;

    if (use24Hour){
      meridiemEl.textContent = '';
      meridiemEl.classList.remove('active');
    } else {
      meridiemEl.textContent = meridiem;
      meridiemEl.classList.add('active');
    }

    const dateOptions = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
    dateEl.textContent = now.toLocaleDateString(undefined, dateOptions);
  }

  // The core requirement: a setInterval-driven clock whose tick rate
  // can be reconfigured at runtime by tearing down and restarting the timer.
  function startTimer(){
    if (timerId !== null){
      clearInterval(timerId);
    }
    render(); // immediate paint so the display never looks stale while waiting for the first tick
    timerId = setInterval(render, intervalMs);
    statusText.textContent = `running · updates every ${intervalMs}ms`;
    tickLabel.textContent = `${intervalMs}ms`;
  }

  function setIntervalMs(newValue){
    const clamped = Math.min(Math.max(newValue, 50), 10000);
    intervalMs = clamped;
    intervalInput.value = clamped;
    startTimer();
  }

  // 12/24 toggle
  function setFormat(mode){
    use24Hour = (mode === '24');
    formatToggle.setAttribute('data-mode', mode);
    render();
  }

  formatToggle.addEventListener('click', () => {
    setFormat(use24Hour ? '12' : '24');
  });

  formatToggle.addEventListener('keydown', (e) => {
    if (e.key === 'Enter' || e.key === ' '){
      e.preventDefault();
      setFormat(use24Hour ? '12' : '24');
    }
  });

  // Interval controls
  intervalInput.addEventListener('change', () => {
    const val = parseInt(intervalInput.value, 10);
    if (!isNaN(val)){
      setIntervalMs(val);
    }
  });

  stepUp.addEventListener('click', () => setIntervalMs(intervalMs + 50));
  stepDown.addEventListener('click', () => setIntervalMs(intervalMs - 50));

  // Boot
  setFormat('12');
  startTimer();
})();
</script>

</body>
</html>

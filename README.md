<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>贪吃蛇</title>
<style>
  * { margin:0; padding:0; box-sizing:border-box; -webkit-tap-highlight-color:transparent; }
  body {
    background:#1a1c24; min-height:100vh; display:flex; flex-direction:column;
    align-items:center; justify-content:center; font-family:system-ui,sans-serif;
    color:#e8eef2; user-select:none; padding:12px;
  }
  .top { width:100%; max-width:480px; display:flex; justify-content:space-between;
         align-items:center; margin-bottom:10px; }
  .score { background:#2f3542; padding:6px 16px; border-radius:30px;
           box-shadow:inset 0 2px 4px rgba(0,0,0,.5); font-size:15px; }
  .score span { color:#78eb96; font-weight:700; margin-left:6px; }

  /* 难度选择栏 */
  .levels { width:100%; max-width:480px; display:flex; gap:8px; margin-bottom:10px; }
  .levels button {
    flex:1; background:#2f3542; border:none; color:#9aa7b8; padding:8px 0;
    border-radius:12px; font-size:13px; cursor:pointer;
    transition:all .15s; box-shadow:inset 0 0 0 1px rgba(255,255,255,.05);
  }
  .levels button.active {
    background:#78eb96; color:#14211a; font-weight:700;
    box-shadow:0 0 14px rgba(120,235,150,.5);
  }
  button { cursor:pointer; }
  #restart {
    background:#3a4050; border:none; color:#dfe6e9; padding:8px 18px;
    border-radius:30px; font-size:14px; box-shadow:0 3px 0 #191d26;
  }
  #restart:active { transform:translateY(3px); box-shadow:none; }
  canvas { border-radius:16px; box-shadow:0 0 0 1px #3d4352, 0 10px 24px rgba(0,0,0,.5);
           background:#191d26; max-width:100%; height:auto; display:block; }
  .pad { margin-top:18px; display:grid; grid-template-columns:repeat(3,64px);
         grid-template-rows:repeat(3,64px); gap:8px; }
  .pad button { width:64px; height:64px; border-radius:16px; font-size:22px; padding:0;
                background:#3a4050; border:none; color:#dfe6e9; box-shadow:0 3px 0 #191d26; }
  .pad button:active { transform:translateY(3px); box-shadow:none; }
  .pad .empty { visibility:hidden; }
  .tip { margin-top:14px; font-size:13px; color:#7f8a9b; text-align:center; }
  @media (hover:hover) and (pointer:fine) { .pad { display:none; } }
</style>
</head>
<body>
  <div class="top">
    <div class="score">🍎 得分<span id="score">0</span></div>
    <button id="restart">↻ 重开</button>
  </div>

  <!-- 难度按钮栏 -->
  <div class="levels" id="levels">
    <button data-speed="200">简单</button>
    <button data-speed="120" class="active">普通</button>
    <button data-speed="80">困难</button>
    <button data-speed="50">地狱</button>
  </div>

  <canvas id="cv" width="480" height="480"></canvas>
  <div class="pad">
    <div class="empty"></div><button data-dir="UP">▲</button><div class="empty"></div>
    <button data-dir="LEFT">◀</button><div class="empty"></div><button data-dir="RIGHT">▶</button>
    <div class="empty"></div><button data-dir="DOWN">▼</button><div class="empty"></div>
  </div>
  <div class="tip">电脑：方向键 / WASD ｜ 手机：下方按钮 ｜ 空格重开</div>

<script>
(function(){
  const CELL=20, COLS=24, ROWS=24;
  const cv=document.getElementById('cv'), ctx=cv.getContext('2d');
  const scoreEl=document.getElementById('score');
  const levelsEl=document.getElementById('levels');

  let snake, food, dir, nextDir, score, running, over, win, timer;
  let speed = 120;   // 当前速度（毫秒/步），默认普通

  const OFF={UP:[0,-1],DOWN:[0,1],LEFT:[-1,0],RIGHT:[1,0]};
  const OPP={UP:'DOWN',DOWN:'UP',LEFT:'RIGHT',RIGHT:'LEFT'};

  // 初始化 / 重开，使用当前 speed
  function init(){
    clearInterval(timer);
    const sx=Math.floor(COLS/2), sy=Math.floor(ROWS/2);
    snake=[{x:sx,y:sy},{x:sx-1,y:sy},{x:sx-2,y:sy}];
    dir='RIGHT'; nextDir='RIGHT'; score=0; running=true; over=false; win=false;
    scoreEl.textContent='0';
    spawnFood();
    draw();
    timer=setInterval(tick, speed);
  }

  function spawnFood(){
    if(snake.length>=COLS*ROWS){ win=true; over=true; running=false; clearInterval(timer); return; }
    let x,y,ok;
    do{
      x=Math.floor(Math.random()*COLS);
      y=Math.floor(Math.random()*ROWS);
      ok=true;
      for(const s of snake) if(s.x===x&&s.y===y){ ok=false; break; }
    }while(!ok);
    food={x,y};
  }

  function tick(){
    if(!running) return;
    dir=nextDir;
    const [dx,dy]=OFF[dir];
    let hx=(snake[0].x+dx+COLS)%COLS;
    let hy=(snake[0].y+dy+ROWS)%ROWS;

    for(let i=0;i<snake.length-1;i++){
      if(snake[i].x===hx&&snake[i].y===hy){
        running=false; over=true; clearInterval(timer); draw(); return;
      }
    }

    for(let i=snake.length-1;i>0;i--){
      snake[i].x=snake[i-1].x; snake[i].y=snake[i-1].y;
    }
    snake[0].x=hx; snake[0].y=hy;

    if(hx===food.x&&hy===food.y){
      score+=10; scoreEl.textContent=score;
      const t=snake[snake.length-1];
      snake.push({x:t.x,y:t.y});
      if(snake.length>=COLS*ROWS){ win=true; over=true; running=false; clearInterval(timer); }
      else spawnFood();
    }
    draw();
  }

  function draw(){
    ctx.fillStyle='#191d26'; ctx.fillRect(0,0,cv.width,cv.height);

    ctx.strokeStyle='#2a3140'; ctx.lineWidth=1;
    for(let i=1;i<COLS;i++){ ctx.beginPath(); ctx.moveTo(i*CELL,0); ctx.lineTo(i*CELL,cv.height); ctx.stroke(); }
    for(let i=1;i<ROWS;i++){ ctx.beginPath(); ctx.moveTo(0,i*CELL); ctx.lineTo(cv.width,i*CELL); ctx.stroke(); }

    ctx.shadowColor='#f25c5c'; ctx.shadowBlur=12; ctx.fillStyle='#f25c5c';
    ctx.beginPath();
    ctx.arc(food.x*CELL+CELL/2, food.y*CELL+CELL/2, CELL/2-3, 0, Math.PI*2);
    ctx.fill(); ctx.shadowBlur=0;

    for(let i=snake.length-1;i>=0;i--){
      const s=snake[i], x=s.x*CELL, y=s.y*CELL;
      if(i===0){
        ctx.fillStyle='#78eb96'; ctx.shadowColor='#78eb96'; ctx.shadowBlur=12;
        ctx.fillRect(x+1,y+1,CELL-2,CELL-2); ctx.shadowBlur=0;
        ctx.fillStyle='#1e2a2e';
        ctx.beginPath(); ctx.arc(x+CELL*0.35,y+CELL*0.4,2.2,0,Math.PI*2); ctx.fill();
        ctx.beginPath(); ctx.arc(x+CELL*0.65,y+CELL*0.4,2.2,0,Math.PI*2); ctx.fill();
      }else{
        const r=i/snake.length;
        ctx.fillStyle='rgb('+(50+r*10|0)+','+(180-r*40|0)+','+(110-r*20|0)+')';
        ctx.fillRect(x+2,y+2,CELL-4,CELL-4);
      }
    }

    if(over){
      ctx.fillStyle='rgba(0,0,0,.72)'; ctx.fillRect(0,0,cv.width,cv.height);
      ctx.textAlign='center'; ctx.textBaseline='middle';
      ctx.font='bold 34px system-ui,sans-serif';
      ctx.fillStyle=win?'#78eb96':'#f25c5c';
      ctx.fillText(win?'🎉 恭喜通关！':'💀 游戏结束', cv.width/2, cv.height/2-30);
      ctx.font='18px system-ui,sans-serif'; ctx.fillStyle='#dfe6e9';
      ctx.fillText('得分: '+score, cv.width/2, cv.height/2+20);
      ctx.font='15px system-ui,sans-serif'; ctx.fillStyle='#a0aec0';
      ctx.fillText('按空格或点重开', cv.width/2, cv.height/2+60);
      ctx.textAlign='start'; ctx.textBaseline='alphabetic';
    }
  }

  function setDir(d){
    if(!running) return;
    if(d!==OPP[dir]) nextDir=d;
  }

  // 切换难度：更新 speed + 高亮按钮 + 重开一局
  levelsEl.addEventListener('click', e=>{
    const btn = e.target.closest('button');
    if(!btn) return;
    speed = parseInt(btn.dataset.speed, 10);
    // 更新高亮
    [...levelsEl.children].forEach(b=>b.classList.remove('active'));
    btn.classList.add('active');
    init();  // 用新速度重开
  });

  document.addEventListener('keydown',e=>{
    const k=e.key;
    if(!running&&over&&(k===' '||k==='Enter')){ e.preventDefault(); init(); return; }
    if(k==='ArrowUp'||k==='w'||k==='W') setDir('UP');
    else if(k==='ArrowDown'||k==='s'||k==='S') setDir('DOWN');
    else if(k==='ArrowLeft'||k==='a'||k==='A') setDir('LEFT');
    else if(k==='ArrowRight'||k==='d'||k==='D') setDir('RIGHT');
    if(['ArrowUp','ArrowDown','ArrowLeft','ArrowRight',' '].includes(k)) e.preventDefault();
  });

  document.querySelectorAll('.pad button').forEach(b=>{
    const fire=e=>{ e.preventDefault(); setDir(b.dataset.dir); };
    b.addEventListener('touchstart',fire,{passive:false});
    b.addEventListener('mousedown',fire);
  });

  document.getElementById('restart').addEventListener('click',init);

  init();
})();
</script>
</body>
</html>

<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<title>🎾 테니스 복식 대진표 FINAL</title>

<style>
@import url('https://cdn.jsdelivr.net/npm/pretendard@latest/dist/web/static/pretendard.css');

*{box-sizing:border-box;font-family:Pretendard}
body{
  background:linear-gradient(135deg,#e0e7ff,#fdf2f8);
  padding:40px;
  text-align:center;
}
h1{font-size:40px;margin-bottom:30px}

.groups{
  display:flex;
  justify-content:center;
  gap:30px;
  margin-bottom:40px;
}

.group{
  background:#fff;
  padding:20px;
  border-radius:24px;
  width:320px;
  box-shadow:0 10px 30px rgba(0,0,0,.1);
}

.group h2{margin-bottom:10px}

select,input{
  padding:8px;
  border-radius:10px;
  border:1px solid #ccc;
  text-align:center;
}

.players{margin-top:10px}

.player{
  display:flex;
  justify-content:center;
  gap:8px;
  margin-bottom:6px;
}

button{
  margin-top:10px;
  padding:10px 18px;
  border:none;
  border-radius:999px;
  background:#6366f1;
  color:#fff;
  font-weight:700;
  cursor:pointer;
}
button:hover{transform:scale(1.05)}

.section{
  max-width:1100px;
  margin:30px auto;
  background:#fff;
  border-radius:24px;
  padding:20px;
  box-shadow:0 10px 30px rgba(0,0,0,.1);
}

.match{
  display:grid;
  grid-template-columns:1fr 120px 1fr;
  align-items:center;
  background:#fff7ed;
  border-radius:16px;
  padding:8px 14px;
  margin-bottom:6px;
}

.team{
  display:flex;
  justify-content:center;
  gap:6px;
  font-weight:600;
}

.score{
  display:flex;
  justify-content:center;
  gap:6px;
}

.score input{
  width:50px;
  height:38px;
  font-size:18px;
  text-align:center;
}

.win{background:#dcfce7 !important}

.rank{
  background:#eef2ff;
  border-radius:14px;
  padding:6px 12px;
  margin:4px 0;
  display:flex;
  justify-content:space-between;
  font-weight:600;
}
</style>
</head>

<body>

<h1>🎾 그린타운 금호한양 월례대회 1월</h1>

<div class="groups">
  <div class="group" data-group="A">
    <h2>A조</h2>
    <select onchange="renderPlayers('A')">
      <option value="4">4명</option>
      <option value="5">5명</option>
      <option value="6" selected>6명</option>
    </select>
    <div class="players" id="players-A"></div>
    <button onclick="shufflePlayers('A')">🎲 랜덤 섞기</button>
  </div>

  <div class="group" data-group="B">
    <h2>B조</h2>
    <select onchange="renderPlayers('B')">
      <option value="4">4명</option>
      <option value="5">5명</option>
      <option value="6" selected>6명</option>
    </select>
    <div class="players" id="players-B"></div>
    <button onclick="shufflePlayers('B')">🎲 랜덤 섞기</button>
  </div>

  <div class="group" data-group="C">
    <h2>C조</h2>
    <select onchange="renderPlayers('C')">
      <option value="4">4명</option>
      <option value="5">5명</option>
      <option value="6" selected>6명</option>
    </select>
    <div class="players" id="players-C"></div>
    <button onclick="shufflePlayers('C')">🎲 랜덤 섞기</button>
  </div>
</div>

<button onclick="generateAll()">🎾 대진표 생성</button>

<div id="output"></div>

<script>
const patterns={
  4:[[1,2,3,4],[1,3,2,4],[1,4,2,3]],
  5:[[1,2,3,4],[1,3,2,5],[1,4,3,5],[1,5,2,4],[2,3,4,5]],
  6:[[1,2,3,4],[1,5,4,6],[2,3,5,6],[1,4,2,5],[2,4,3,6],[1,6,3,5]]
};

function renderPlayers(g){
  const n=document.querySelector(`[data-group="${g}"] select`).value;
  const box=document.getElementById(`players-${g}`);
  box.innerHTML='';
  for(let i=1;i<=n;i++){
    box.innerHTML+=`
      <div class="player">
        <input placeholder="이름">
        <input value="${i}" style="width:40px">
      </div>`;
  }
}

['A','B','C'].forEach(renderPlayers);

function shufflePlayers(g){
  const rows=[...document.querySelectorAll(`#players-${g} .player`)];
  const vals=rows.map(r=>({
    name:r.children[0].value||'선수',
    num:r.children[1].value
  }));
  vals.sort(()=>Math.random()-.5);
  rows.forEach((r,i)=>{
    r.children[0].value=vals[i].name;
    r.children[1].value=vals[i].num;
  });
}

function generateAll(){
  const out=document.getElementById('output');
  out.innerHTML='';

  ['A','B','C'].forEach(g=>{
    const rows=[...document.querySelectorAll(`#players-${g} .player`)];
    if(!rows.length) return;

    const players={}, scores={};
    rows.forEach(r=>{
      players[r.children[1].value]=r.children[0].value||'선수';
      scores[r.children[1].value]=0;
    });

    const section=document.createElement('div');
    section.className='section';
    section.innerHTML=`<h2>${g}조 개인 승점</h2><div id="rank-${g}"></div><h2>${g}조 대진표</h2>`;
    out.appendChild(section);

    const p=patterns[rows.length];
    p.forEach(m=>{
      const div=document.createElement('div');
      div.className='match';
      div.innerHTML=`
        <div class="team" data-p="${m[0]},${m[1]}">${players[m[0]]} ${players[m[1]]}</div>
        <div class="score">
          <input type="number" oninput="recalc()">
          :
          <input type="number" oninput="recalc()">
        </div>
        <div class="team" data-p="${m[2]},${m[3]}">${players[m[2]]} ${players[m[3]]}</div>
      `;
      section.appendChild(div);
    });

    section.dataset.players=JSON.stringify(players);
    section.dataset.scores=JSON.stringify(scores);
  });
}

function recalc(){
  document.querySelectorAll('.section').forEach(sec=>{
    const players=JSON.parse(sec.dataset.players);
    const scores={};
    Object.keys(players).forEach(k=>scores[k]=0);

    sec.querySelectorAll('.match').forEach(m=>{
      const s=[...m.querySelectorAll('input')].map(i=>+i.value||0);
      const t1=m.children[0].dataset.p.split(',');
      const t2=m.children[2].dataset.p.split(',');
      m.classList.remove('win');

      if(s[0]!==s[1]){
        const diff=s[0]-s[1];
        if(diff>0){
          t1.forEach(p=>scores[p]+=diff);
          t2.forEach(p=>scores[p]-=diff);
          m.classList.add('win');
        }else{
          t1.forEach(p=>scores[p]+=diff);
          t2.forEach(p=>scores[p]-=diff);
          m.classList.add('win');
        }
      }
    });

    const rank=sec.querySelector(`#rank-${sec.querySelector('h2').innerText[0]}`);
    rank.innerHTML='';
    Object.entries(scores)
      .sort((a,b)=>b[1]-a[1])
      .forEach(([num,pt])=>{
        rank.innerHTML+=`<div class="rank"><span>${players[num]}</span><span>${pt}</span></div>`;
      });
  });
}
</script>

</body>
</html>

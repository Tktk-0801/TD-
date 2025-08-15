<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>3人vs敵バトル（新キャラ全部入り・修正版）</title>
<style>
  body { background:#0b0b0b; color:#fff; font-family:sans-serif; text-align:center; }
  #gameContainer { width:980px; margin:18px auto; border:2px solid #fff; padding:12px; position:relative; min-height:640px; }
  .hp-bar { height:14px; background:#c33; border-radius:6px; overflow:hidden; margin-top:6px; }
  .hp-bar-inner { height:100%; background:linear-gradient(90deg,#f66,#a00); width:100%; }
  .player-wrap { display:flex; justify-content:space-between; gap:10px; margin-top:8px; }
  .player-card { background:#111; padding:8px; border-radius:8px; width:30%; box-sizing:border-box; cursor:pointer; }
  .player-card.selected { outline:3px solid gold; }
  .player-card img { width:64px; height:64px; background:#222; display:block; margin:4px auto; border-radius:6px; object-fit:cover; }
  .small { font-size:12px; color:#ccc; }
  button { margin:6px; padding:6px 10px; border-radius:6px; cursor:pointer; }
  #battleLog { background:#0d0d0d; height:200px; overflow:auto; padding:8px; border:1px solid #444; text-align:left; font-size:13px; margin-top:10px; }
  .badge { display:inline-block; background:#222; padding:2px 6px; border-radius:6px; margin-left:6px; font-size:12px; }
  .stat-line { display:flex; justify-content:space-between; align-items:center; gap:8px; }
  .label { color:#bbb; font-size:13px; }
  .skill-info { margin-top:6px; color:#9f9; text-align:left; }
  .effect-badge { display:inline-block; background:#0a0; color:#fff; padding:2px 6px; border-radius:6px; margin-left:6px; font-size:12px; }
  .damagePop{position:absolute;font-weight:800;color:#ffdddd;text-shadow:0 0 8px #f00;font-size:18px;z-index:1100;pointer-events:none}
</style>
</head>
<body>
<div id="gameContainer"></div>

<script>
// -------------------- データ（全キャラまとめて） --------------------
const players = [
  {id:"sakura", name:"サクラ", baseAtk:2000, hp:10000, maxHp:10000, skillTurn:2, currentTurn:0, image:"", skillDesc:"敵のHP10%ダメ＆2T 攻撃200%UP", effects:[], barrier:0},
  {id:"kana",   name:"カナ",   baseAtk:1400, hp:13000, maxHp:13000, skillTurn:4, currentTurn:0, image:"", skillDesc:"味方全員攻撃110%UP＆HP800バリア", effects:[], barrier:0},
  {id:"hina",   name:"ヒナ",   baseAtk:1900, hp:13000, maxHp:13000, skillTurn:17, currentTurn:0, image:"", skillDesc:"合計ATKの500%ダメージ", effects:[], barrier:0},
  {id:"mika",   name:"ミカ",   baseAtk:1200, hp:10000, maxHp:10000, skillTurn:10, currentTurn:0, image:"", skillDesc:"攻撃1540%（調整可）", effects:[], barrier:0},
  {id:"seina",  name:"セイナ",  baseAtk:1800, hp:14500, maxHp:14500, skillTurn:3, currentTurn:0, image:"", skillDesc:"2T 吸収バリア", effects:[], barrier:0},

  {id:"kayoko", name:"カヨコ", baseAtk:1000, hp:12000, maxHp:12000, skillTurn:3, currentTurn:0, image:"", skillDesc:"受けた200%分反撃（3T）", effects:[], barrier:0},
  {id:"cocona", name:"ココナ", baseAtk:300,  hp:23000, maxHp:23000, skillTurn:5, currentTurn:0, image:"", skillDesc:"味方HP70%回復 & 3T ATK200%UP", effects:[], barrier:0},
  {id:"iwori",  name:"イヲリ",  baseAtk:3000, hp:11000, maxHp:11000, skillTurn:17, currentTurn:0, image:"", skillDesc:"合計HP+ATKの250%ダメ", effects:[], barrier:0},
  {id:"ui",     name:"ウイ",    baseAtk:800,  hp:16000, maxHp:16000, skillTurn:10, currentTurn:0, image:"", skillDesc:"味方の残りスキルターンを2短縮 (例:7→5)", effects:[], barrier:0},
  {id:"kikyou", name:"キキョウ",baseAtk:3200, hp:9900,  maxHp:9900,  skillTurn:6, currentTurn:0, image:"", skillDesc:"HP-3000 自分ATK大幅UP（700%→変更可）", effects:[], barrier:0},
  {id:"kanna",  name:"カンナ",  baseAtk:1100, hp:12000, maxHp:12000, skillTurn:3, currentTurn:0, image:"", skillDesc:"自身と右側の味方ATK160%UP", effects:[], barrier:0},

  // クロコ（強化）
  {id:"kuroko", name:"クロコ", baseAtk:1500, hp:16000, maxHp:16000, skillTurn:6, currentTurn:0, image:"", skillDesc:"5T 不死身・会心倍率3倍・全員ATK300%UP", effects:[], barrier:0},

  // 新キャラ群（シロコの説明修正）
  {id:"shiroko", name:"シロコ", baseAtk:1450, hp:12000, maxHp:12000, skillTurn:3, currentTurn:0, image:"", skillDesc:"3T 敵に合計HP30%ダメ & 全員HP40%回復", effects:[], barrier:0},
  {id:"hare", name:"ハレ", baseAtk:1100, hp:11000, maxHp:11000, skillTurn:2, currentTurn:0, image:"", skillDesc:"1T 自身以外の味方ATK+130%（2Tに変更）", effects:[], barrier:0},
  {id:"wakamo", name:"ワカモ", baseAtk:3000, hp:12000, maxHp:12000, skillTurn:10, currentTurn:0, image:"", skillDesc:"攻撃時に合計ATKの追加ダメ & 5T 会心率大幅UP", effects:[], barrier:0},
  {id:"sakurako", name:"サクラコ", baseAtk:1000, hp:16000, maxHp:16000, skillTurn:10, currentTurn:0, image:"", skillDesc:"10T 自身と右端の会心ダメ3倍 & 全員HP70%回復", effects:[], barrier:0},
  {id:"sena", name:"セナ", baseAtk:900, hp:20000, maxHp:20000, skillTurn:12, currentTurn:0, image:"", skillDesc:"全員に合計攻撃力+HPの200%分回復（強力）", effects:[], barrier:0},
  {id:"koharu", name:"コハル", baseAtk:750, hp:18500, maxHp:18500, skillTurn:2, currentTurn:0, image:"", skillDesc:"1T 自身以外ATK+130%、被ダメ10%軽減、HP<=50%の仲間を60%回復（持続2T）", effects:[], barrier:0},
  {id:"toki", name:"トキ", baseAtk:1700, hp:12000, maxHp:12000, skillTurn:20, currentTurn:0, image:"", skillDesc:"アビ・エシュフ: 自身ATK/HP+40%（変換）", effects:[], barrier:0}
];

// localStorage から画像を読み込み（保存した画像が残る）
players.forEach(p=>{
  const v = localStorage.getItem('player_image_' + p.id);
  if(v) p.image = v;
});

// ---------- 敵リスト ----------
const enemies = [
  {name:"魔王ゴルゴーン", hp:50000, maxHp:50000, atk:300, behavior:["single","aoe","buff"]},
  {name:"ドラゴンキング", hp:60000, maxHp:60000, atk:350, behavior:["single","aoe","buff"]},
  {name:"終焉の巨像", hp:500000, maxHp:500000, atk:4000, behavior:["big","healRare"]},
  {name:"虚無の王", hp:400000, maxHp:400000, atk:3202, behavior:["single","big","healRare"]},
  {name:"ラルガドル", hp:230000, maxHp:230000, atk:2800, behavior:["single","aoe","healRare"]},
  {name:"凶犬ヘルガ", hp:80000, maxHp:80000, atk:1400, behavior:["single","multi"]},
  {name:"灼熱タイラント", hp:120000, maxHp:120000, atk:2000, behavior:["single","aoe","burn"]},
  {name:"星喰いワーム", hp:220000, maxHp:220000, atk:2600, behavior:["aoe","big"]},
  {name:"ヒエロニムス", hp:400000, maxHp:400000, atk:2800, behavior:["aoe","big","single"]},
  {name:"メカゴルム", hp:180000, maxHp:180000, atk:1800, behavior:["single","buff","aoe"]},
  {name:"ビナー", hp:380000, maxHp:380000, atk:1800, behavior:["single","buff","aoe"]},
  {name:"深淵の大鯨", hp:250000, maxHp:250000, atk:2000, behavior:["aoe","healRare"]}
];

// -------------------- 変数 --------------------
let selectedPlayerIndexes = [];
let selectedSlot = 0;
let currentEnemy = null;
let turnCount = 1;

// -------------------- ヘルパー --------------------
function clamp(n,a,b){ return Math.max(a, Math.min(b, n)); }
function popDamage(x, left, top, isCrit=false){
  const cont = document.getElementById('gameContainer');
  const el = document.createElement('div');
  el.className = 'damagePop';
  el.style.left = left + 'px';
  el.style.top = top + 'px';
  el.innerHTML = (isCrit?'<b style="color:#ffd700">CRIT!</b><br>':'') + Math.floor(x);
  cont.appendChild(el);
  let t=0;
  const id = setInterval(()=>{ el.style.top = (parseFloat(el.style.top) - 1) + 'px'; el.style.opacity = 1 - t/40; t++; if(t>40){ clearInterval(id); try{cont.removeChild(el);}catch(e){} } },12);
}

// -------------------- UI: ホーム --------------------
function showHome(){
  const c = document.getElementById('gameContainer');
  c.innerHTML = `<h1>3人 vs 敵 バトル（新キャラ入り）</h1><h3>敵を選べ</h3>`;
  enemies.forEach((e,i)=>{
    const btn = document.createElement('button');
    btn.innerText = e.name;
    btn.onclick = ()=>{ currentEnemy = JSON.parse(JSON.stringify(e)); currentEnemy.hp = e.hp; currentEnemy.dot = []; showCharacterSelect(); };
    c.appendChild(btn);
  });
  const note = document.createElement('div'); note.className='small';
  note.innerText = "※ キャラ画像はアップロードするとブラウザ(localStorage)に保存されます。";
  c.appendChild(document.createElement('hr'));
  c.appendChild(note);
}

// -------------------- UI: キャラ選択 --------------------
function showCharacterSelect(){
  const c = document.getElementById('gameContainer');
  c.innerHTML = `<h2>出撃キャラを3人選べ（左から順）</h2>`;
  const wrap = document.createElement('div'); wrap.style.display='flex'; wrap.style.gap='8px'; wrap.style.flexWrap='wrap';
  players.forEach((p, idx)=>{
    const card = document.createElement('div'); card.className='player-card';
    card.innerHTML = `<img id="icon_${p.id}" src="${p.image||''}" alt=""><div style="text-align:center">${p.name}<div class="small">HP:${p.maxHp} ATK:${p.baseAtk}</div></div>
      <input type="file" id="file_${p.id}" style="width:100%; margin-top:6px">`;
    card.onclick = ()=>{
      if(selectedPlayerIndexes.includes(idx)){ selectedPlayerIndexes = selectedPlayerIndexes.filter(x=>x!==idx); card.classList.remove('selected'); }
      else { if(selectedPlayerIndexes.length<3){ selectedPlayerIndexes.push(idx); card.classList.add('selected'); } else alert('3人までだ'); }
    };
    wrap.appendChild(card);

    setTimeout(()=>{ const input = document.getElementById('file_'+p.id); if(input) input.onchange = (ev)=>{ const f = ev.target.files[0]; if(!f) return; const r = new FileReader(); r.onload = e=>{ players[idx].image = e.target.result; const icon = document.getElementById('icon_'+p.id); if(icon) icon.src = e.target.result; localStorage.setItem('player_image_'+p.id, e.target.result); }; r.readAsDataURL(f); }; },50);
  });
  c.appendChild(wrap);
  const start = document.createElement('div'); start.style.marginTop='12px';
  start.innerHTML = `<button id="startBtn">戦闘開始！</button>`;
  c.appendChild(start);
  document.getElementById('startBtn').onclick = ()=>{
    if(selectedPlayerIndexes.length !== 3){ alert('3人選んでくれ'); return; }
    selectedPlayerIndexes.forEach(i=>{
      players[i].hp = players[i].maxHp;
      players[i].currentTurn = 0;
      players[i].effects = [];
      players[i].barrier = 0;
      players[i].undead = false;
      players[i].undeadTurn = 0;
      players[i].absorb = false;
      players[i].absorbTurn = 0;
      players[i].guaranteedCrit = 0;
      players[i].hinaDress = null;
    });
    selectedSlot = 0;
    turnCount = 1;
    startBattle();
  };
}

// -------------------- バトル表示 --------------------
function startBattle(){
  const c = document.getElementById('gameContainer');
  c.innerHTML = `
    <h2>敵: ${currentEnemy.name}</h2>
    <div class="enemy-section">
      <div class="hp-bar" id="enemyHpBar"><div class="hp-bar-inner" id="enemyHpInner"></div></div>
      <div id="enemyHpText" class="small"></div>
    </div>
    <div id="playersArea"></div>
    <div style="margin-top:8px">
      <button id="attackBtn">通常攻撃</button>
      <button id="skillBtn">スキル</button>
      <button id="homeBtn">ホームへ</button>
    </div>
    <div id="battleLog"></div>
    <hr>
    <div style="margin-top:8px">
      <h4>出撃スロット画像アップロード（各スロットに割り当てたキャラに保存されます）</h4>
      <input type="file" id="slot0"> <input type="file" id="slot1"> <input type="file" id="slot2">
    </div>
  `;
  renderBattle();
  attachBattleEvents();
  log(`${currentEnemy.name} とバトル開始！`);
}

function renderBattle(){
  const eInner = document.getElementById('enemyHpInner');
  const pct = clamp(currentEnemy.hp / currentEnemy.maxHp * 100, 0, 100);
  eInner.style.width = pct + '%';
  document.getElementById('enemyHpText').innerText = `HP: ${Math.max(0, Math.floor(currentEnemy.hp))} / ${currentEnemy.maxHp}`;

  const area = document.getElementById('playersArea');
  area.innerHTML = '<h3>味方（左→右）</h3>';
  const wrap = document.createElement('div'); wrap.className='player-wrap';
  selectedPlayerIndexes.forEach((pIdx,slot)=>{
    const p = players[pIdx];
    const card = document.createElement('div'); card.className='player-card';
    if(slot === selectedSlot) card.classList.add('selected');

    const currentAtk = computeAtk(p);
    const hpPct = clamp(p.hp / p.maxHp * 100, 0, 100);
    const hpBar = `<div class="hp-bar" style="background:#333"><div class="hp-bar-inner" style="width:${hpPct}%; background:linear-gradient(90deg,#6f6,#0a0)"></div></div>`;

    let extraBadges = '';
    if(p.barrier && p.barrier > 0) extraBadges += `<span class="badge">Barrier:${p.barrier}</span>`;
    if(p.absorb && p.absorbTurn>0) extraBadges += `<span class="effect-badge">吸収:${p.absorbTurn}</span>`;
    if(p.guaranteedCrit && p.guaranteedCrit>0) extraBadges += `<span class="badge">会心確定:${p.guaranteedCrit}T</span>`;
    if(p.hinaDress && p.hinaDress.turns > 0) extraBadges += `<span class="badge">DressStacks:${p.hinaDress.stacks}</span>`;

    card.innerHTML = `
      <img src="${p.image||''}" alt="${p.name}">
      <div style="text-align:center"><b>${p.name}</b> ${extraBadges}</div>
      <div class="small">HP: ${Math.max(0,Math.floor(p.hp))} / ${p.maxHp}</div>
      ${hpBar}
      <div class="stat-line"><div class="label">基礎ATK</div><div>${p.baseAtk}</div></div>
      <div class="stat-line"><div class="label">現在ATK</div><div>${currentAtk}</div></div>
      <div class="small">スキルまで: ${Math.max(0, p.skillTurn - p.currentTurn)}ターン</div>
      <div class="skill-info" id="skill_${p.id}">${p.skillDesc}</div>
    `;
    card.onclick = ()=>{ selectedSlot = slot; renderBattle(); log(`選択: ${p.name}`); };
    wrap.appendChild(card);
  });
  area.appendChild(wrap);
}

// -------------------- 計算（ATK/会心倍率など） --------------------
function computeAtk(p){
  let mul = 1.0;
  if(p.effects && p.effects.length){
    p.effects.forEach(e=>{
      if(e.type === 'atkMulti') mul *= e.mult;
    });
  }
  if(p.hinaDress && (p.hinaDress.stacks || p.hinaDress.stacks === 0)){
    mul *= (1 + 1.5 * (p.hinaDress.stacks || 0));
  }
  return Math.max(0, Math.floor(p.baseAtk * mul));
}
function getCritMultiplier(p){
  let mult = 2.0;
  if(p.effects && p.effects.length){
    p.effects.forEach(e=>{ if(e.type === 'critMult') mult = e.mult; });
  }
  return mult;
}
function tryCrit(p){
  if(p.guaranteedCrit && p.guaranteedCrit > 0) return true;
  let extra = 0;
  if(p.effects && p.effects.length){
    p.effects.forEach(e=>{ if(e.type === 'critChance') extra += (e.add || 0); });
  }
  return Math.random() < (0.15 + extra);
}
function doDamageVisual(dmg, isCrit=false){
  popDamage(dmg, 420 + Math.random()*140, 120 + Math.random()*40, isCrit);
}

// -------------------- プレイヤー通常攻撃 --------------------
function playerAttack(){
  if(selectedSlot === null || selectedSlot === undefined){ alert('攻撃する枠選べ'); return; }
  const pIdx = selectedPlayerIndexes[selectedSlot];
  const p = players[pIdx];
  const atk = computeAtk(p);
  const isCrit = tryCrit(p);
  const critMult = isCrit ? getCritMultiplier(p) : 1.0;
  const dmg = Math.floor(atk * critMult);
  currentEnemy.hp -= dmg;
  log(`${p.name}の通常攻撃！ 敵に ${dmg}ダメージ ${isCrit? '(会心!)':''}`, 'attack');
  doDamageVisual(dmg, isCrit);

  if(p.effects && p.effects.length){
    p.effects.forEach(e=>{
      if(e.type === 'onAttackAdd' && e.percent){
        const totalAtk = selectedPlayerIndexes.reduce((s,idx)=> s + computeAtk(players[idx]), 0);
        const add = Math.floor(totalAtk * e.percent);
        currentEnemy.hp -= add;
        log(`${p.name} の追撃効果で追加 ${add} ダメージ`, 'attack'); doDamageVisual(add);
      }
    });
  }

  advanceCharge();
  setTimeout(()=>{ enemyAction(); renderBattle(); checkBattleEnd(); }, 450);
  renderBattle();
}

// -------------------- スキル使用ラッパー --------------------
function useSkill(){
  if(selectedSlot === null || selectedSlot === undefined){ alert('スキル使う枠選んで'); return; }
  const pIdx = selectedPlayerIndexes[selectedSlot];
  const p = players[pIdx];
  if(p.currentTurn < p.skillTurn){ alert('スキルまだ溜まってねー'); return; }

  playBigEffect(()=>{ useSkillFor(pIdx, selectedSlot); renderBattle(); checkBattleEnd(); });
  setTimeout(()=>{ advanceCharge(); enemyAction(); renderBattle(); checkBattleEnd(); }, 700);
}

// -------------------- スキル実装（各キャラ） --------------------
function useSkillFor(playerIndex, slotIndex){
  const p = players[playerIndex];

  if(p.id === 'sakura'){
    const dmg = Math.floor(currentEnemy.hp * 0.10);
    currentEnemy.hp -= dmg;
    selectedPlayerIndexes.forEach(idx=> players[idx].effects.push({type:'atkMulti', mult:3.0, turns:2}));
    log(`サクラ: 刹那の残刀！ 敵に${dmg}ダメージ。味方ATK+200% 2T`, 'attack');
    doDamageVisual(dmg);
  } else if(p.id === 'kana'){
    selectedPlayerIndexes.forEach(idx=>{ players[idx].effects.push({type:'atkMulti', mult:1.1, turns:2}); players[idx].barrier += 800; });
    log('カナ: 天使との聖約！ 味方全員ATK110%UP＆HP800バリア', 'heal');
  } else if(p.id === 'hina'){
    const totalAtk = selectedPlayerIndexes.reduce((s,idx)=> s + computeAtk(players[idx]), 0);
    const dmg = Math.floor(totalAtk * 5.0);
    currentEnemy.hp -= dmg;
    log(`ヒナ: ダークエレクトロ！ 敵に${dmg}ダメージ`, 'attack'); doDamageVisual(dmg);
  } else if(p.id === 'mika'){
    const dmg = Math.floor(p.baseAtk * 15.4);
    currentEnemy.hp -= dmg;
    log(`ミカ: 超火力！ ${dmg}ダメージ`, 'attack'); doDamageVisual(dmg);
  } else if(p.id === 'mika_b'){
    const dmg = Math.floor(p.baseAtk * 30.0);
    currentEnemy.hp -= dmg;
    selectedPlayerIndexes.forEach(idx=> players[idx].effects.push({type:'atkMulti', mult:3.0, turns:4}));
    selectedPlayerIndexes.forEach(idx=> players[idx].guaranteedCrit = (players[idx].guaranteedCrit || 0) + 4);
    selectedPlayerIndexes.forEach(idx=> players[idx].effects.push({type:'critMult', mult:3.0, turns:4}));
    log(`ミカ(水着): ${dmg}ダメージ & 全員4T ATK200%UP & 会心確定4T`, 'attack'); doDamageVisual(dmg,true);
  } else if(p.id === 'seina'){
    selectedPlayerIndexes.forEach(idx=>{ players[idx].absorb = true; players[idx].absorbTurn = 2; });
    log('セイナ: 2T 吸収バリア展開（ダメ→HP変換）', 'heal');
  }

  // クロコ強化
  else if(p.id === 'kuroko'){
    selectedPlayerIndexes.forEach(idx=>{
      players[idx].undead = true;
      players[idx].undeadTurn = 5;
      players[idx].effects.push({type:'atkMulti', mult:4.0, turns:5});
      players[idx].effects.push({type:'critMult', mult:3.0, turns:5});
    });
    log('クロコ: 5T 不死身＆会心倍率3倍＆全員ATK300%UP', 'heal');
  }

  // シロコ：合計を30%に変更（3ターンで分割）
  else if(p.id === 'shiroko'){
    // 合計ダメージ（敵最大HPの30%）を3回に分ける
    const total = Math.floor(currentEnemy.maxHp * 0.30);
    const per = Math.floor(total / 3);
    currentEnemy.dot = currentEnemy.dot || [];
    currentEnemy.dot.push({tag:'shiroko', remaining:3, perTurn:per});
    // 味方HP40%回復（即時）
    selectedPlayerIndexes.forEach(idx=>{
      players[idx].hp = Math.min(players[idx].maxHp, players[idx].hp + Math.floor(players[idx].maxHp * 0.40));
    });
    log(`シロコ: 爆撃衝動！ 敵に合計${total}を3Tで分割して与える（${per}×3）、味方HPを40%回復`, 'attack');
    doDamageVisual(per);
  } else if(p.id === 'hare'){
    selectedPlayerIndexes.forEach((idx,slot)=>{
      if(slot !== slotIndex){
        players[idx].effects.push({type:'atkMulti', mult:2.3, turns:2}); // ← 変更: 持続2ターン
      }
    });
    log('ハレ: 2T 自身以外の味方ATK+130%', 'heal');
  } else if(p.id === 'wakamo'){
    p.effects.push({type:'onAttackAdd', percent:1.0, turns:5});
    p.effects.push({type:'critChance', add:0.40, turns:5});
    log('ワカモ: 5T 攻撃時に合計ATKの100%追撃 & 会心率大幅UP', 'heal');
  } else if(p.id === 'sakurako'){
    const lastSlotIdx = selectedPlayerIndexes.length - 1;
    const targets = [slotIndex, lastSlotIdx].filter((v,i,arr)=> arr.indexOf(v)===i);
    targets.forEach(tslot=>{
      if(selectedPlayerIndexes[tslot] !== undefined){
        const tIdx = selectedPlayerIndexes[tslot];
        players[tIdx].effects.push({type:'critMult', mult:3.0, turns:10});
      }
    });
    selectedPlayerIndexes.forEach(idx=> players[idx].hp = Math.min(players[idx].maxHp, players[idx].hp + Math.floor(players[idx].maxHp * 0.70)));
    log('サクラコ: 自身&右端の会心倍率を3倍（10T）＆味方HPを70%回復', 'heal');
  } else if(p.id === 'sena'){
    const totalAtk = selectedPlayerIndexes.reduce((s,idx)=> s + computeAtk(players[idx]), 0);
    const totalHP = selectedPlayerIndexes.reduce((s,idx)=> s + players[idx].hp, 0);
    const healAll = Math.floor((totalAtk + totalHP) * 2.0 / selectedPlayerIndexes.length);
    selectedPlayerIndexes.forEach(idx=> players[idx].hp = Math.min(players[idx].maxHp, players[idx].hp + healAll));
    log(`セナ: 強力回復！ 味方全員に ${healAll} 回復（計算式ベース）`, 'heal');
  } else if(p.id === 'koharu'){
    selectedPlayerIndexes.forEach((idx,slot)=>{
      if(slot !== slotIndex){
        players[idx].effects.push({type:'atkMulti', mult:2.3, turns:2}); // ← 変更: 持続2ターン
        players[idx].effects.push({type:'damageReduce', add:0.10, turns:2}); // ← 変更: 持続2ターン
        if(players[idx].hp <= players[idx].maxHp * 0.5){
          players[idx].hp = Math.min(players[idx].maxHp, players[idx].hp + Math.floor(players[idx].maxHp * 0.60));
          log(`${players[idx].name} は HP50%以下のため60%回復`, 'heal');
        }
      }
    });
    log('コハル: 2T 自身以外ATK+130%、被ダメ10%軽減、対象HP<=50%なら60%回復', 'heal');
  } else if(p.id === 'toki'){
    p.baseAtk = Math.floor(p.baseAtk * 1.4);
    p.maxHp = Math.floor(p.maxHp * 1.4);
    p.hp = Math.min(p.maxHp, Math.floor(p.hp * 1.4));
    log('トキ: アビ・エシュフモード転換。自身のATK・HPが40%上昇（恒久）', 'heal');
  } else {
    log(`${p.name} のスキルは未実装`, 'normal');
  }

  p.currentTurn = 0;
  p.skillReady = false;
}

// -------------------- 敵行動 --------------------
function enemyAction(){
  // まずは敵に付いている継続ダメージ（dot）を処理する（shiroko等）
  if(currentEnemy.dot && currentEnemy.dot.length){
    for(let i = currentEnemy.dot.length -1; i >=0; i--){
      const d = currentEnemy.dot[i];
      if(d.remaining > 0){
        const dmg = d.perTurn;
        currentEnemy.hp = Math.max(0, currentEnemy.hp - dmg);
        log(`継続効果(${d.tag}) が敵に ${dmg} ダメージ（残 ${d.remaining-1}）`, 'attack');
        doDamageVisual(dmg);
        d.remaining--;
      }
      if(d.remaining <= 0){
        currentEnemy.dot.splice(i,1);
      }
    }
  }

  const behavior = currentEnemy.behavior;
  const a = behavior[(turnCount-1) % behavior.length];

  if(a === 'single'){
    const slot = Math.floor(Math.random() * selectedPlayerIndexes.length);
    const pIdx = selectedPlayerIndexes[slot];
    const t = players[pIdx];
    const dmgBase = currentEnemy.atk;
    if(t.undead && t.undeadTurn>0){ log(`${t.name} は不死身で無効化！`, 'heal'); }
    else if(t.absorb && t.absorbTurn>0){ t.hp = Math.min(t.maxHp, t.hp + dmgBase); log(`${t.name}の吸収が${dmgBase}をHPに変換！`, 'heal'); doDamageVisual(dmgBase); }
    else {
      let dmg = dmgBase;
      if(t.barrier > 0){
        if(dmg <= t.barrier){ t.barrier -= dmg; log(`${t.name}のバリアが${dmg}を吸収！ 残:${t.barrier}`,'heal'); dmg = 0; }
        else { dmg -= t.barrier; t.barrier = 0; }
      }
      if(dmg>0){
        const reflect = (t.effects||[]).find(e=> e.type === 'reflectMult');
        if(reflect){ const refD = Math.floor(dmg * reflect.mult); currentEnemy.hp -= refD; log(`${t.name}が反撃！ ${refD}を返した！`, 'attack'); doDamageVisual(refD); }
        const dr = (t.effects||[]).find(e=> e.type === 'damageReduce');
        if(dr && dr.add){ dmg = Math.floor(dmg * (1 - dr.add)); }
        t.hp = Math.max(0, t.hp - dmg);
        log(`${currentEnemy.name} が ${t.name} に ${dmg}ダメージ`, 'attack'); doDamageVisual(dmg);
      }
    }
  } else if(a === 'aoe'){
    selectedPlayerIndexes.forEach((pIdx,slot)=>{ const t = players[pIdx]; let dmg = currentEnemy.atk;
      if(t.undead && t.undeadTurn>0){ log(`${t.name}は不死身で無効化！`,'heal'); }
      else if(t.absorb && t.absorbTurn>0){ t.hp = Math.min(t.maxHp, t.hp + dmg); log(`${t.name}の吸収が${dmg}をHPに変換！`,'heal'); doDamageVisual(dmg); }
      else {
        if(t.barrier > 0){ if(dmg <= t.barrier){ t.barrier -= dmg; log(`${t.name}のバリアが${dmg}吸収！ 残:${t.barrier}`); dmg = 0; } else { dmg -= t.barrier; t.barrier = 0; } }
        if(dmg>0){ const reflect = (t.effects||[]).find(e=> e.type === 'reflectMult'); if(reflect){ const refD = Math.floor(dmg * reflect.mult); currentEnemy.hp -= refD; log(`${t.name}が反撃！ ${refD}`,'attack'); doDamageVisual(refD); } const dr = (t.effects||[]).find(e=> e.type === 'damageReduce'); if(dr && dr.add){ dmg = Math.floor(dmg * (1 - dr.add)); } t.hp = Math.max(0, t.hp - dmg); doDamageVisual(dmg); }
      }
    });
    log(`${currentEnemy.name} の全体攻撃！`, 'attack'); playSmallEffect();
  } else if(a === 'buff'){
    currentEnemy.atk = Math.floor(currentEnemy.atk * 1.12);
    log(`${currentEnemy.name} が攻撃上昇！`, 'attack');
  } else if(a === 'healRare'){
    if(Math.random() < 0.35){ const heal = Math.floor(currentEnemy.maxHp * 0.06); currentEnemy.hp = Math.min(currentEnemy.maxHp, currentEnemy.hp + heal); log(`${currentEnemy.name} が希に回復 +${heal}`,'heal'); playSmallEffect(); } else log(`${currentEnemy.name} は回復を控えた`,'normal');
  } else if(a === 'big'){
    selectedPlayerIndexes.forEach(pIdx=>{ players[pIdx].hp = Math.max(0, players[pIdx].hp - currentEnemy.atk * 2); });
    log(`${currentEnemy.name} の大技！ 全員大ダメージ`, 'attack'); playBigEffect();
  } else {
    log(`${currentEnemy.name} の奇行`,'normal');
  }

  // 継続効果デクリメント & ドレスヒナスタック処理
  selectedPlayerIndexes.forEach(idx=>{
    const p = players[idx];
    if(p.absorbTurn > 0){ p.absorbTurn--; if(p.absorbTurn===0) p.absorb=false; }
    if(p.undeadTurn > 0){ p.undeadTurn--; if(p.undeadTurn===0) p.undead=false; }
    if(p.effects && p.effects.length){
      p.effects.forEach(e=> e.turns = Math.max(0, e.turns - 1));
      p.effects = p.effects.filter(e=> e.turns > 0);
    }
    if(p.guaranteedCrit && p.guaranteedCrit>0) p.guaranteedCrit--;
    if(p.hinaDress && p.hinaDress.turns > 0){
      p.hinaDress.stacks = (p.hinaDress.stacks || 0) + 1;
      p.hinaDress.turns--;
      if(p.hinaDress.turns === 0){
        log(`${p.name} のドレス効果が切れた。（最終スタック ${p.hinaDress.stacks}）`, 'normal');
      } else {
        log(`${p.name} のドレス効果：スタック +1（合計 ${p.hinaDress.stacks}）`, 'normal');
      }
    }
  });

  turnCount++;
}

// -------------------- 更新（チャージ） --------------------
function advanceCharge(){
  selectedPlayerIndexes.forEach(idx=>{ const p = players[idx]; p.currentTurn++; if(p.currentTurn >= p.skillTurn) p.skillReady = true; });
}

// -------------------- エフェクト --------------------
function playBigEffect(cb){
  const cont = document.getElementById('gameContainer');
  const el = document.createElement('div'); el.style.position='absolute'; el.style.left='50%'; el.style.top='40px'; el.style.transform='translateX(-50%)'; el.style.width='420px'; el.style.height='220px'; el.style.borderRadius='50%'; el.style.background='radial-gradient(circle,#ff6a6a,transparent)'; el.style.opacity='0.95'; el.style.zIndex=999;
  cont.appendChild(el);
  setTimeout(()=>{ el.style.transition='opacity 0.6s, transform 0.6s'; el.style.opacity='0'; el.style.transform='translateX(-50%) scale(1.6)'; },80);
  setTimeout(()=>{ try{cont.removeChild(el);}catch(e){} if(cb) cb(); },700);
}
function playSmallEffect(){ const c=document.getElementById('gameContainer'); const e=document.createElement('div'); e.style.position='absolute'; e.style.left='50%'; e.style.top='60px'; e.style.transform='translateX(-50%)'; e.style.width='260px'; e.style.height='160px'; e.style.borderRadius='50%'; e.style.background='radial-gradient(circle,#6affb0,transparent)'; e.style.opacity='0.95'; c.appendChild(e); setTimeout(()=>{ e.style.transition='opacity 0.5s, transform 0.5s'; e.style.opacity='0'; e.style.transform='translateX(-50%) scale(1.4)'; },50); setTimeout(()=>{ try{c.removeChild(e);}catch(e){} },520); }

// -------------------- ログ・勝敗 --------------------
function log(msg, type='normal'){ const logDiv = document.getElementById('battleLog'); const div = document.createElement('div'); div.innerHTML = (type==='attack'?`<span style="color:#ff9999">${msg}</span>`: type==='heal'?`<span style="color:#9ff">${msg}</span>` : `<span>${msg}</span>`); logDiv.appendChild(div); logDiv.scrollTop = logDiv.scrollHeight; }

function checkBattleEnd(){
  if(currentEnemy.hp <= 0){ alert('勝利！'); selectedPlayerIndexes = []; showHome(); return; }
  const totalHP = selectedPlayerIndexes.reduce((s,idx)=> s + Math.max(0, players[idx].hp), 0);
  if(totalHP <= 0){ alert('敗北...'); selectedPlayerIndexes = []; showHome(); return; }
}

// -------------------- イベントバインド --------------------
function attachBattleEvents(){
  document.getElementById('attackBtn').onclick = playerAttack;
  document.getElementById('skillBtn').onclick = useSkill;
  document.getElementById('homeBtn').onclick = ()=>{ if(confirm('戻る？戦闘は中断されるぜ')){ selectedPlayerIndexes = []; showHome(); } };

  [0,1,2].forEach(slot=>{
    const input = document.getElementById('slot'+slot);
    if(!input) return;
    input.onchange = e=>{
      const f = e.target.files[0]; if(!f) return;
      const r = new FileReader();
      r.onload = ev=>{
        const pIdx = selectedPlayerIndexes[slot];
        if(pIdx === undefined){ alert('そのスロット未割当'); return; }
        players[pIdx].image = ev.target.result;
        localStorage.setItem('player_image_' + players[pIdx].id, ev.target.result);
        renderBattle();
      };
      r.readAsDataURL(f);
    };
  });
}

// -------------------- 初期表示 --------------------
showHome();

</script>
</body>
</html>

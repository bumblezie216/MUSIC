<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<title>My Music Player</title>
<style>
:root{--bg:#fdf6fb;--card:#fff;--text:#2a1f2d;--muted:#8a7a8f;--accent:#d6459b;--line:#f0dfe9;
box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#1a1320;--card:#251b2d;--text:#f6ecf8;--muted:#a898ae;--accent:#ff6fb8;--line:#3a2c44}}
:root[data-theme="dark"]{--bg:#1a1320;--card:#251b2d;--text:#f6ecf8;--muted:#a898ae;--accent:#ff6fb8;--line:#3a2c44}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--text);font-family:-apple-system,system-ui,"Segoe UI",Roboto,sans-serif;max-width:520px;margin:0 auto;padding:16px}
h1{font-size:22px;margin:8px 0 14px}
.card{background:var(--card);border:1px solid var(--line);border-radius:20px;padding:18px;margin-bottom:14px}
.now{font-size:18px;font-weight:600;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.sub{color:var(--muted);font-size:13px;margin-top:2px}
input[type=range]{width:100%;accent-color:var(--accent);margin:14px 0 2px}
.times{display:flex;justify-content:space-between;color:var(--muted);font-size:12px}
.ctrl{display:flex;justify-content:center;align-items:center;gap:14px;margin-top:10px}
button{font:inherit;color:var(--text);background:none;border:none;cursor:pointer}
.ib{width:48px;height:48px;border-radius:50%;font-size:22px;background:var(--line)}
.ib.on{background:var(--accent);color:#fff}
.play{width:64px;height:64px;font-size:28px;background:var(--accent);color:#fff}
.add{display:block;width:100%;padding:14px;border-radius:14px;background:var(--accent);color:#fff;font-weight:600;text-align:center}
ul{list-style:none;margin:0;padding:0}
li{display:flex;align-items:center;gap:8px;padding:10px 4px;border-bottom:1px solid var(--line)}
li:last-child{border:none}
li .t{flex:1;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;font-size:15px}
li.cur .t{color:var(--accent);font-weight:600}
li .x{color:var(--muted);font-size:18px;padding:6px 10px}
.empty{color:var(--muted);text-align:center;padding:18px 0;font-size:14px}
.note{color:var(--muted);font-size:12px;line-height:1.4;text-align:center;margin-top:10px}
</style>
</head>
<body>
<h1>🎵 My Music Player</h1>

<div id="install" class="card" style="text-align:center">
  <button id="shareBtn" class="add" style="font-size:20px;padding:22px">📲 Tap here to Add to Home Screen</button>
  <div class="sub" id="shareHelp" style="margin-top:10px">Then in the menu that opens, scroll down and tap <b>Add to Home Screen</b>, then <b>Add</b>.</div>
</div>

<div class="card">
  <div class="now" id="title">Nothing playing</div>
  <div class="sub" id="sub">Add songs to get started</div>
  <input type="range" id="seek" min="0" max="100" value="0" step="0.1" aria-label="Seek">
  <div class="times"><span id="cur">0:00</span><span id="dur">0:00</span></div>
  <div class="ctrl">
    <button class="ib" id="shuf" aria-label="Shuffle">🔀</button>
    <button class="ib" id="prev" aria-label="Previous">⏮</button>
    <button class="ib play" id="play" aria-label="Play or pause">▶</button>
    <button class="ib" id="next" aria-label="Next">⏭</button>
    <button class="ib" id="rep" aria-label="Repeat">🔁</button>
  </div>
</div>

<label class="add">＋ Add songs from Files
  <input type="file" id="file" accept="audio/*" multiple hidden>
</label>

<div class="card" style="margin-top:14px">
  <ul id="list"></ul>
  <div class="empty" id="empty">No songs yet</div>
</div>

<div class="note">Tap play, then you can lock your phone or switch apps and the music keeps going. For the best result on iPhone, tap Share → Add to Home Screen.</div>

<audio id="audio" preload="auto" playsinline></audio>

<script>
const $=id=>document.getElementById(id);
const audio=$('audio');
let songs=[],idx=-1,shuffle=false,repeat=false,urls={},memoryOnly=false;

/* ---------- storage (IndexedDB, with fallback) ---------- */
function db(){return new Promise((res,rej)=>{
  try{const r=indexedDB.open('music-player',1);
  r.onupgradeneeded=()=>r.result.createObjectStore('songs',{keyPath:'id',autoIncrement:true});
  r.onsuccess=()=>res(r.result);r.onerror=()=>rej(r.error);}catch(e){rej(e)}});}
async function dbAll(){try{const d=await db();return await new Promise((res,rej)=>{
  const q=d.transaction('songs').objectStore('songs').getAll();q.onsuccess=()=>res(q.result);q.onerror=()=>rej(q.error)})}catch(e){memoryOnly=true;return[]}}
async function dbAdd(s){try{const d=await db();return await new Promise((res,rej)=>{
  const q=d.transaction('songs','readwrite').objectStore('songs').add(s);q.onsuccess=()=>res(q.result);q.onerror=()=>rej(q.error)})}catch(e){memoryOnly=true;return Date.now()+Math.random()}}
async function dbDel(id){try{const d=await db();d.transaction('songs','readwrite').objectStore('songs').delete(id)}catch(e){}}

/* ---------- helpers ---------- */
const fmt=t=>isFinite(t)?Math.floor(t/60)+':'+String(Math.floor(t%60)).padStart(2,'0'):'0:00';
const clean=n=>n.replace(/\.[^.]+$/,'');
function urlFor(s){return urls[s.id]||(urls[s.id]=URL.createObjectURL(s.blob))}

/* ---------- list ---------- */
function render(){
  const ul=$('list');ul.innerHTML='';
  songs.forEach((s,i)=>{
    const li=document.createElement('li');if(i===idx)li.className='cur';
    const t=document.createElement('button');t.className='t';t.textContent=s.name;t.onclick=()=>load(i,true);
    const x=document.createElement('button');x.className='x';x.textContent='✕';x.setAttribute('aria-label','Remove');
    x.onclick=()=>remove(i);
    li.append(t,x);ul.append(li);
  });
  $('empty').style.display=songs.length?'none':'block';
}

/* ---------- playback ---------- */
function load(i,play){
  if(i<0||i>=songs.length)return;
  idx=i;const s=songs[i];
  audio.src=urlFor(s);
  $('title').textContent=s.name;
  $('sub').textContent=(i+1)+' of '+songs.length;
  setMeta(s);render();
  if(play)tryPlay();
}
function setMeta(s){
  try{if('mediaSession' in navigator){
    navigator.mediaSession.metadata=new MediaMetadata({title:s.name,artist:'My Music Player',album:'My Songs'});
  }}catch(e){}
}
function nextIndex(dir){
  if(!songs.length)return -1;
  if(shuffle&&dir>0&&songs.length>1){let n;do{n=Math.floor(Math.random()*songs.length)}while(n===idx);return n}
  return (idx+dir+songs.length)%songs.length;
}
function next(){const n=nextIndex(1);if(n>=0)load(n,true)}
function prev(){if(audio.currentTime>3){audio.currentTime=0;return}const n=nextIndex(-1);if(n>=0)load(n,true)}
function toggle(){
  if(idx<0&&songs.length){load(0,true);return}
  if(audio.paused)tryPlay();else audio.pause();
}
async function remove(i){
  const s=songs[i];await dbDel(s.id);
  if(urls[s.id]){URL.revokeObjectURL(urls[s.id]);delete urls[s.id]}
  songs.splice(i,1);
  if(i===idx){audio.pause();audio.removeAttribute('src');idx=-1;$('title').textContent='Nothing playing';$('sub').textContent=songs.length?'Pick a song':'Add songs to get started'}
  else if(i<idx)idx--;
  render();
}

/* ---------- error handling + fallback ---------- */
let tried={};
function showErr(m){$('sub').textContent=m}
function tryPlay(){
  try{const p=audio.play();if(p&&p.catch)p.catch(e=>{if(e&&e.name!=='AbortError')showErr('Couldn’t play ('+(e.name||'error')+'). Tap play again.')})}catch(e){showErr('Couldn’t play: '+e.message)}
}
audio.onerror=()=>{
  const s=songs[idx];if(!s)return;
  if(!tried[s.id]){
    tried[s.id]=true;showErr('Trying another way…');
    const r=new FileReader();
    r.onload=()=>{const b=new Blob([s.blob],{type:s.blob.type||'audio/mpeg'});audio.src=URL.createObjectURL(b);audio.load();tryPlay()};
    r.onerror=()=>showErr('Couldn’t read this file');
    r.readAsArrayBuffer(s.blob);
  }else{
    const c=audio.error?audio.error.code:0;
    showErr('This file won’t play (error '+c+', type '+(s.blob.type||'unknown')+'). Try an .mp3 or .m4a saved on your phone.');
  }
};

/* ---------- events ---------- */
$('play').onclick=toggle;$('next').onclick=next;$('prev').onclick=prev;
$('shuf').onclick=()=>{shuffle=!shuffle;$('shuf').classList.toggle('on',shuffle)};
$('rep').onclick=()=>{repeat=!repeat;$('rep').classList.toggle('on',repeat)};
audio.onplay=()=>{$('play').textContent='⏸';try{navigator.mediaSession.playbackState='playing'}catch(e){}};
audio.onpause=()=>{$('play').textContent='▶';try{navigator.mediaSession.playbackState='paused'}catch(e){}};
audio.onended=()=>{if(repeat){audio.currentTime=0;audio.play().catch(()=>{})}else next()};
audio.onloadedmetadata=()=>{$('dur').textContent=fmt(audio.duration);updatePos()};
let seeking=false;
audio.ontimeupdate=()=>{
  if(!seeking&&audio.duration)$('seek').value=audio.currentTime/audio.duration*100;
  $('cur').textContent=fmt(audio.currentTime);updatePos();
};
$('seek').oninput=()=>{seeking=true;if(audio.duration)$('cur').textContent=fmt($('seek').value/100*audio.duration)};
$('seek').onchange=()=>{if(audio.duration)audio.currentTime=$('seek').value/100*audio.duration;seeking=false};
function updatePos(){
  try{if('mediaSession' in navigator&&isFinite(audio.duration)&&audio.duration>0)
    navigator.mediaSession.setPositionState({duration:audio.duration,playbackRate:audio.playbackRate,position:Math.min(audio.currentTime,audio.duration)});}catch(e){}
}
try{
  const ms=navigator.mediaSession;
  ms.setActionHandler('play',()=>audio.play());
  ms.setActionHandler('pause',()=>audio.pause());
  ms.setActionHandler('previoustrack',prev);
  ms.setActionHandler('nexttrack',next);
  ms.setActionHandler('seekto',d=>{audio.currentTime=d.seekTime});
  ms.setActionHandler('seekbackward',d=>{audio.currentTime=Math.max(0,audio.currentTime-(d.seekOffset||10))});
  ms.setActionHandler('seekforward',d=>{audio.currentTime=Math.min(audio.duration||1e9,audio.currentTime+(d.seekOffset||10))});
}catch(e){}

$('file').onchange=async e=>{
  const files=[...e.target.files];
  for(const f of files){
    const s={name:clean(f.name),blob:f};
    s.id=await dbAdd(s);
    songs.push(s);
  }
  e.target.value='';
  render();
  if(idx<0&&songs.length)load(0,false);
  if(memoryOnly)$('sub').textContent='Songs won’t be saved in this browser mode';
};

/* ---------- add to home screen helper ---------- */
if(window.navigator.standalone||matchMedia('(display-mode: standalone)').matches){$('install').style.display='none'}
$('shareBtn').onclick=async()=>{
  try{
    if(navigator.share){await navigator.share({title:'My Music Player',url:location.href})}
    else{$('shareHelp').innerHTML='Tap the <b>Share</b> button at the bottom of Safari (square with an arrow), then <b>Add to Home Screen</b>.'}
  }catch(e){if(e&&e.name!=='AbortError')$('shareHelp').innerHTML='Tap the <b>Share</b> button at the bottom of Safari (square with an arrow), then <b>Add to Home Screen</b>.'}
};

/* ---------- start ---------- */
(async()=>{songs=await dbAll();render();if(songs.length){load(0,false);$('sub').textContent=songs.length+' saved song'+(songs.length>1?'s':'')}})();
</script>
</body>
</html>

<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Packet Journey — Ghifari Diaz</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
<style>
  :root{
    --navy-950:#070b14;
    --navy-900:#0b1120;
    --navy-800:#101a2e;
    --cyan:#38e0ff;
    --violet:#a78bfa;
    --amber:#fbbf24;
    --ink-100:#e7ecf5;
    --ink-400:#7c8aa8;
    --line:#1c2740;
  }
  *{box-sizing:border-box; margin:0; padding:0;}
  html,body{background:var(--navy-950); color:var(--ink-100); font-family:'Inter',sans-serif; overflow-x:hidden;}
  ::selection{background:var(--cyan); color:var(--navy-950);}

  .journey{ position:relative; height:650vh; }
  .pin{
    position:sticky; top:0; height:100vh; width:100%;
    overflow:hidden;
    display:flex; align-items:center; justify-content:center;
    background:
      radial-gradient(circle at 20% 20%, rgba(56,224,255,0.07), transparent 40%),
      radial-gradient(circle at 80% 75%, rgba(167,139,250,0.08), transparent 45%),
      var(--navy-950);
  }

  .grid-bg{
    position:absolute; inset:0;
    background-image:
      linear-gradient(var(--line) 1px, transparent 1px),
      linear-gradient(90deg, var(--line) 1px, transparent 1px);
    background-size: 48px 48px;
    opacity:0.35;
    mask-image: radial-gradient(ellipse 70% 60% at 50% 50%, black 40%, transparent 80%);
  }

  svg.path-svg{ position:absolute; inset:0; width:100%; height:100%; }
  .route{ fill:none; stroke:var(--line); stroke-width:2; stroke-dasharray:4 8; }
  .route-glow{ fill:none; stroke:var(--cyan); stroke-width:1.5; opacity:0.5; filter:blur(1px); }

  .node-dot{ fill:var(--navy-800); stroke:var(--ink-400); stroke-width:1.5; }
  .node-dot.active{ stroke:var(--cyan); }

  .packet{
    position:absolute; top:0; left:0;
    width:14px; height:14px; border-radius:50%;
    background:var(--cyan);
    box-shadow: 0 0 12px 3px rgba(56,224,255,0.9), 0 0 30px 8px rgba(56,224,255,0.35);
    transform:translate(-50%,-50%);
    z-index:5;
    pointer-events:none;
  }

  .hud{
    position:absolute; top:28px; left:28px;
    font-family:'JetBrains Mono', monospace;
    font-size:12px; line-height:1.6; color:var(--ink-400);
    background:rgba(11,17,32,0.6);
    border:1px solid var(--line);
    padding:14px 18px;
    border-radius:6px;
    backdrop-filter: blur(6px);
    min-width:230px;
    z-index:10;
  }
  .hud .row{ display:flex; justify-content:space-between; gap:16px; }
  .hud .k{ color:var(--ink-400); }
  .hud .v{ color:var(--cyan); }
  .hud .v.amber{ color:var(--amber); }
  .hud-title{ color:var(--ink-100); font-weight:500; margin-bottom:8px; letter-spacing:0.05em; }

  .progress-rail{
    position:absolute; right:28px; top:50%; transform:translateY(-50%);
    width:2px; height:180px; background:var(--line); z-index:10;
  }
  .progress-fill{
    position:absolute; left:0; top:0; width:100%; height:0%;
    background:linear-gradient(var(--cyan), var(--violet));
  }

  .panel{
    position:absolute;
    max-width:460px;
    padding:0 8vw;
    opacity:0;
    z-index:8;
  }
  .panel .eyebrow{
    font-family:'JetBrains Mono', monospace;
    font-size:12px; letter-spacing:0.12em; color:var(--cyan);
    text-transform:uppercase; margin-bottom:14px;
    display:flex; align-items:center; gap:10px;
  }
  .panel .eyebrow::before{ content:''; width:20px; height:1px; background:var(--cyan); }
  .panel h2{
    font-family:'Space Grotesk', sans-serif;
    font-weight:600; font-size:clamp(28px,4vw,44px);
    line-height:1.15; margin-bottom:16px; color:var(--ink-100);
  }
  .panel p{ color:var(--ink-400); font-size:16px; line-height:1.65; }
  .panel .accent{ color:var(--amber); }

  .stack-row{ display:flex; flex-wrap:wrap; gap:10px; margin-top:18px; }
  .stack-chip{
    font-family:'JetBrains Mono', monospace; font-size:12px;
    padding:6px 12px; border:1px solid var(--line); border-radius:20px;
    color:var(--ink-100);
  }

  .cta{
    display:inline-flex; align-items:center; gap:8px; margin-top:24px;
    font-family:'JetBrains Mono', monospace; font-size:13px;
    color:var(--navy-950); background:var(--cyan);
    padding:10px 18px; border-radius:6px; font-weight:500;
  }

  .scroll-hint{
    position:absolute; bottom:32px; left:50%; transform:translateX(-50%);
    font-family:'JetBrains Mono', monospace; font-size:11px; color:var(--ink-400);
    letter-spacing:0.15em; text-transform:uppercase;
    animation: pulse 2s ease-in-out infinite;
    z-index:10;
  }
  @keyframes pulse{ 0%,100%{opacity:0.4;} 50%{opacity:1;} }

  .end-space{ height:20vh; }
</style>
</head>
<body>

<section class="journey">
  <div class="pin" id="pin">
    <div class="grid-bg"></div>

    <svg class="path-svg" id="pathSvg" viewBox="0 0 1000 1000" preserveAspectRatio="none">
      <path id="routePath" class="route" d="" />
      <path id="routeGlow" class="route-glow" d="" />
      <g id="nodesGroup"></g>
    </svg>

    <div class="packet" id="packet"></div>

    <div class="hud">
      <div class="hud-title">// PACKET_INSPECTOR</div>
      <div class="row"><span class="k">hop</span><span class="v" id="hudHop">00 / 06</span></div>
      <div class="row"><span class="k">node</span><span class="v" id="hudNode">CLIENT</span></div>
      <div class="row"><span class="k">latency</span><span class="v" id="hudLatency">0ms</span></div>
      <div class="row"><span class="k">status</span><span class="v amber" id="hudStatus">CONNECTING</span></div>
    </div>

    <div class="progress-rail"><div class="progress-fill" id="progressFill"></div></div>

    <!-- Panels -->
    <div class="panel" id="panel0" style="left:6vw; top:24vh;">
      <div class="eyebrow">01 · Client Request</div>
      <h2>Halo, saya <span class="accent">Ghifari Diaz</span>.</h2>
      <p>Mahasiswa Teknik Informatika semester 6, fokus backend development. Setiap request punya perjalanan — begitu juga setiap sistem yang saya bangun.</p>
    </div>

    <div class="panel" id="panel1" style="right:6vw; top:20vh;">
      <div class="eyebrow">02 · DNS Resolve</div>
      <h2>Mencari <span class="accent">arah yang tepat.</span></h2>
      <p>Belajar sambil membangun — dari CRUD sederhana hingga sistem HR dengan alur approval berlapis di lingkungan perbankan.</p>
    </div>

    <div class="panel" id="panel2" style="left:6vw; top:22vh;">
      <div class="eyebrow">03 · Load Balancer</div>
      <h2>Stack yang <span class="accent">menopang beban.</span></h2>
      <p>Tools yang saya andalkan sehari-hari:</p>
      <div class="stack-row">
        <span class="stack-chip">Laravel</span>
        <span class="stack-chip">PHP</span>
        <span class="stack-chip">MySQL</span>
        <span class="stack-chip">Next.js</span>
        <span class="stack-chip">React Three Fiber</span>
        <span class="stack-chip">Tailwind</span>
      </div>
    </div>

    <div class="panel" id="panel3" style="right:6vw; top:20vh;">
      <div class="eyebrow">04 · API Gateway</div>
      <h2>HR System <span class="accent">Bank Kerta.</span></h2>
      <p>Laravel 12 + FilamentPHP v3 — absensi geofencing, payroll otomatis, dan alur cuti multi-role sekuensial (pending → validasi SDM → disetujui).</p>
    </div>

    <div class="panel" id="panel4" style="left:6vw; top:20vh;">
      <div class="eyebrow">05 · Database</div>
      <h2>Proyek <span class="accent">lainnya.</span></h2>
      <p>Sistem Iuran Warga (Laravel + Midtrans), chat bridge multi-client (Python asyncio), studi clustering K-Means vs DBSCAN, dan lainnya.</p>
    </div>

    <div class="panel" id="panel5" style="right:6vw; top:22vh;">
      <div class="eyebrow">06 · 200 OK</div>
      <h2>Mari <span class="accent">terhubung.</span></h2>
      <p>Request sudah sampai tujuan. Terbuka untuk kolaborasi dan diskusi proyek.</p>
      <div class="cta">get_in_touch() →</div>
    </div>

    <div class="scroll-hint" id="scrollHint">↓ scroll to trace the request</div>
  </div>
</section>
<div class="end-space"></div>

<script>
gsap.registerPlugin(ScrollTrigger);

// Zigzag route across a 1000x1000 viewBox, 6 nodes
const points = [
  {x:120, y:180}, // 0 client
  {x:820, y:150}, // 1 dns
  {x:150, y:400}, // 2 lb
  {x:850, y:430}, // 3 api
  {x:150, y:680}, // 4 db
  {x:820, y:720}  // 5 response
];

function buildPath(pts){
  let d = `M ${pts[0].x} ${pts[0].y}`;
  for(let i=1;i<pts.length;i++){
    const prev = pts[i-1];
    const curr = pts[i];
    const midX = (prev.x + curr.x)/2;
    d += ` C ${midX} ${prev.y}, ${midX} ${curr.y}, ${curr.x} ${curr.y}`;
  }
  return d;
}

const dPath = buildPath(points);
document.getElementById('routePath').setAttribute('d', dPath);
document.getElementById('routeGlow').setAttribute('d', dPath);

const nodesGroup = document.getElementById('nodesGroup');
const nodeLabels = ['CLIENT','DNS','LOAD BALANCER','API GATEWAY','DATABASE','RESPONSE'];
points.forEach((p,i)=>{
  const c = document.createElementNS('http://www.w3.org/2000/svg','circle');
  c.setAttribute('cx', p.x); c.setAttribute('cy', p.y);
  c.setAttribute('r', 7); c.setAttribute('class','node-dot');
  c.id = 'node-'+i;
  nodesGroup.appendChild(c);
});

const pathEl = document.getElementById('routePath');
const totalLength = pathEl.getTotalLength();
const packet = document.getElementById('packet');
const svgEl = document.getElementById('pathSvg');

function svgPointToScreen(pt){
  const svgRect = svgEl.getBoundingClientRect();
  const vb = svgEl.viewBox.baseVal;
  const scaleX = svgRect.width / vb.width;
  const scaleY = svgRect.height / vb.height;
  return { x: pt.x * scaleX, y: pt.y * scaleY };
}

const panels = [0,1,2,3,4,5].map(i=>document.getElementById('panel'+i));
const hudHop = document.getElementById('hudHop');
const hudNode = document.getElementById('hudNode');
const hudLatency = document.getElementById('hudLatency');
const hudStatus = document.getElementById('hudStatus');
const progressFill = document.getElementById('progressFill');
const scrollHint = document.getElementById('scrollHint');

function updateFrame(progress){
  const lenAt = progress * totalLength;
  const pt = pathEl.getPointAtLength(lenAt);
  const screenPt = svgPointToScreen(pt);
  packet.style.transform = `translate(${screenPt.x - svgEl.getBoundingClientRect().left + svgEl.getBoundingClientRect().left}px, ${screenPt.y}px) translate(-50%,-50%)`;
  packet.style.left = screenPt.x + 'px';
  packet.style.top = screenPt.y + 'px';
  packet.style.transform = 'translate(-50%,-50%)';

  const seg = 1/6;
  const nodeIndex = Math.min(5, Math.floor(progress / seg));
  const localT = (progress % seg) / seg;

  hudHop.textContent = String(nodeIndex+1).padStart(2,'0') + ' / 06';
  hudNode.textContent = nodeLabels[nodeIndex];
  hudLatency.textContent = Math.round(8 + progress*140 + Math.sin(progress*40)*4) + 'ms';
  hudStatus.textContent = progress > 0.97 ? '200 OK' : (progress < 0.03 ? 'CONNECTING' : 'IN TRANSIT');
  progressFill.style.height = (progress*100) + '%';
  scrollHint.style.opacity = progress > 0.02 ? 0 : 0.7;

  for(let i=0;i<6;i++){
    const nodeEl = document.getElementById('node-'+i);
    nodeEl.classList.toggle('active', i <= nodeIndex);
  }

  panels.forEach((p, i)=>{
    const center = i*seg + seg/2;
    const dist = Math.abs(progress - center);
    const fade = 1 - Math.min(1, dist / (seg*0.6));
    p.style.opacity = Math.max(0, fade);
    const dir = i % 2 === 0 ? 1 : -1;
    p.style.transform = `translateY(${(1-fade)*24*dir}px)`;
  });
}

ScrollTrigger.create({
  trigger: '.journey',
  start: 'top top',
  end: 'bottom bottom',
  scrub: 0.4,
  onUpdate: (self) => updateFrame(self.progress)
});

updateFrame(0);
window.addEventListener('resize', ()=> updateFrame(ScrollTrigger.getAll()[0]?.progress || 0));
</script>
</body>
</html>
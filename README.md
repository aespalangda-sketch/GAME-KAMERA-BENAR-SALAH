[APLIKASI_GAME_KAMERA_BENAR-SALAH.html](https://github.com/user-attachments/files/32448549/APLIKASI_GAME_KAMERA_BENAR-SALAH.2.html)
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Kuis Interaktif A/B</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@500;600;700&family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
<style>
  :root{
    --bg:#070B14;
    --a:#3B82F6;
    --b:#F59E0B;
    --correct:#22C55E;
    --wrong:#EF4444;
    --text:#F8FAFC;
    --muted:#94A3B8;
    --panel:rgba(7,11,20,0.86);
  }
  *{box-sizing:border-box; margin:0; padding:0;}
  html,body{
    width:100%; height:100%; overflow:hidden;
    background:var(--bg); color:var(--text);
    font-family:'Inter',system-ui,sans-serif;
  }
  #stage{position:fixed; inset:0;}
  #ownerWatermark{
    position:absolute; bottom:10px; right:14px; z-index:30;
    font-family:'Space Grotesk',sans-serif; font-weight:600;
    font-size:.75rem; letter-spacing:.06em;
    color:rgba(255,255,255,.55);
    text-shadow:0 1px 2px rgba(0,0,0,.6);
    pointer-events:none; user-select:none;
  }

  /* ---- Camera layer ---- */
  #cam-wrap{position:absolute; inset:0; overflow:hidden; background:#000;}
  #video{
    position:absolute; top:0; left:0;
    width:100%; height:100%;
    transform:scaleX(-1);
    object-fit:cover;
  }
  #cam-placeholder{
    position:absolute; inset:0; display:flex; flex-direction:column;
    align-items:center; justify-content:center; gap:18px;
    background:radial-gradient(circle at 50% 40%, #131C30 0%, #070B14 70%);
  }
  #cam-placeholder p{color:var(--muted); font-size:1.1rem; max-width:480px; text-align:center;}
  #startCamBtn{
    font-family:'Space Grotesk',sans-serif; font-weight:600; font-size:1.05rem;
    background:var(--a); color:#fff; border:none; padding:16px 32px;
    border-radius:999px; cursor:pointer; letter-spacing:.2px;
  }
  #startCamBtn:hover{filter:brightness(1.08);}

  /* ---- Divider ---- */
  #divider{
    position:absolute; top:0; bottom:0; left:50%; width:2px;
    background:linear-gradient(to bottom, transparent 0%, rgba(255,255,255,.55) 12%, rgba(255,255,255,.55) 88%, transparent 100%);
    z-index:5; transform:translateX(-1px);
  }

  /* ---- Zones (color wash on check) ---- */
  .zone{
    position:absolute; top:0; bottom:0; width:50%;
    transition:background .4s ease;
    z-index:4; pointer-events:none;
  }
  #zoneA{left:0; background:rgba(59,130,246,0);}
  #zoneB{left:50%; background:rgba(245,158,11,0);}
  #zoneA.correct{background:rgba(34,197,94,.38);}
  #zoneA.wrong{background:rgba(239,68,68,.38);}
  #zoneB.correct{background:rgba(34,197,94,.38);}
  #zoneB.wrong{background:rgba(239,68,68,.38);}

  /* ---- Area atas: soal + pilihan jawaban dijadikan satu panel yang menempel di atas ---- */
  #topArea{
    position:absolute; top:0; left:0; right:0; z-index:10;
    display:flex; flex-direction:column; align-items:stretch;
  }
  /* ---- Question box (top) ---- */
  #qbox{
    background:rgba(7,11,20,.95);
    padding:54px 40px 5px;
    display:flex; flex-direction:column; align-items:center; gap:3px;
    text-align:center;
  }
  #qcounter{
    font-family:'Space Grotesk',sans-serif; font-weight:600; font-size:.78rem;
    color:var(--muted); letter-spacing:.5px;
  }
  #qtext{
    font-family:'Space Grotesk',sans-serif; font-weight:700;
    font-size:clamp(1.05rem, 1.9vw, 1.6rem); line-height:1.2; max-width:1100px;
  }

  /* ---- Baris pilihan jawaban (sekarang tepat di bawah soal, bukan di dasar layar) ---- */
  #answersRow{display:flex; width:100%;}
  .answer-card{
    width:50%;
    padding:8px 40px 12px;
    display:flex; align-items:center; gap:16px;
    background:linear-gradient(180deg, rgba(7,11,20,.95) 0%, rgba(7,11,20,.85) 55%, rgba(7,11,20,0) 100%);
  }
  #cardA{justify-content:flex-start;}
  #cardB{justify-content:flex-start; text-align:right; flex-direction:row-reverse;}
  .letter{
    font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:2.4rem;
    width:66px; height:66px; min-width:66px; border-radius:16px;
    display:flex; align-items:center; justify-content:center; color:#fff;
  }
  #cardA .letter{background:var(--a);}
  #cardB .letter{background:var(--b);}
  .ans-text{
    font-size:clamp(1.2rem, 2vw, 1.65rem); font-weight:700; line-height:1.25; max-width:520px;
  }

  /* ---- Center check button ---- */
  #checkBtnCenter{
    position:absolute; bottom:18px; left:50%; transform:translateX(-50%);
    z-index:16; font-family:'Space Grotesk',sans-serif; font-weight:600;
    font-size:.85rem; letter-spacing:.2px; color:#fff; border:none;
    background:var(--correct); padding:10px 22px; border-radius:999px;
    cursor:pointer; box-shadow:0 6px 18px rgba(0,0,0,.45);
  }
  #checkBtnCenter:hover{filter:brightness(1.08);}
  #checkBtnCenter:disabled{opacity:.45; cursor:not-allowed;}

  /* ---- Timer badge ---- */
  #timerBadge{
    position:absolute; top:16px; left:16px; z-index:17;
    width:60px; height:60px; border-radius:50%;
    background:rgba(7,11,20,.9); border:3px solid var(--a);
    display:flex; align-items:center; justify-content:center;
    font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:1.5rem;
    color:var(--text); transition:border-color .25s ease, color .25s ease;
  }
  #timerBadge.urgent{border-color:var(--wrong); color:var(--wrong); animation:tickPulse .5s infinite alternate;}
  #timerBadge.done{border-color:var(--correct); color:var(--correct);}
  @keyframes tickPulse{from{transform:scale(1);} to{transform:scale(1.14);}}

  /* ---- Result banner ---- */
  #resultBanner{
    position:absolute; top:50%; left:50%; transform:translate(-50%,-50%);
    z-index:20; font-family:'Space Grotesk',sans-serif; font-weight:700;
    font-size:clamp(2rem,5vw,4rem); padding:18px 46px; border-radius:20px;
    background:rgba(0,0,0,.55); backdrop-filter:blur(6px);
    opacity:0; pointer-events:none; transition:opacity .3s ease;
  }
  #resultBanner.show{opacity:1;}
  #resultBanner.correct-flash{color:var(--correct);}
  #resultBanner.wrong-flash{color:var(--wrong);}

  /* ---- Teacher control panel ---- */
  #panel{
    position:absolute; top:16px; right:16px; z-index:30;
    background:var(--panel); border:1px solid rgba(255,255,255,.08);
    border-radius:16px; padding:14px; display:flex; flex-direction:column; gap:8px;
    width:220px; transition:transform .25s ease;
  }
  #panel.collapsed{transform:translateX(calc(100% + 16px));}
  #panelToggle{
    position:absolute; top:16px; right:16px; z-index:31;
    width:44px; height:44px; border-radius:12px; border:1px solid rgba(255,255,255,.12);
    background:var(--panel); color:var(--text); font-size:1.2rem; cursor:pointer;
  }
  #panel h3{font-family:'Space Grotesk',sans-serif; font-size:.85rem; color:var(--muted); font-weight:600; margin-bottom:2px;}
  .btn{
    font-family:'Inter',sans-serif; font-weight:600; font-size:.92rem;
    border:none; border-radius:10px; padding:11px 12px; cursor:pointer; color:#fff;
    text-align:left;
  }
  .btn:disabled{opacity:.4; cursor:not-allowed;}
  #checkBtn{background:var(--correct);}
  #nextBtn{background:var(--a);}
  #prevBtn{background:rgba(255,255,255,.12); color:var(--text);}
  #fsBtn{background:rgba(255,255,255,.12); color:var(--text);}
  #resetBtn{background:rgba(239,68,68,.85);}

  #finishScreen{
    position:absolute; inset:0; z-index:40; display:none;
    align-items:center; justify-content:center; flex-direction:column; gap:16px;
    background:rgba(7,11,20,.94); text-align:center; padding:20px;
  }
  #finishScreen h2{font-family:'Space Grotesk',sans-serif; font-size:clamp(1.8rem,4vw,3rem);}
  #finishScreen p{color:var(--muted); font-size:1.05rem; max-width:520px;}
</style>
<style id="mx-addon-style">
  /* ===== TAMBAHAN: halaman awal, editor soal, notifikasi (kode asli di atas tidak diubah) ===== */
  .mx-screen{position:absolute; inset:0; display:flex; z-index:50; background:var(--bg); color:var(--text); font-family:'Inter',system-ui,sans-serif;}
  .mx-screen button:focus-visible,.mx-screen input:focus-visible,.mx-screen textarea:focus-visible,#mxToast button:focus-visible{outline:2px solid #fff; outline-offset:2px;}

  .mx-btn{font-family:'Space Grotesk',sans-serif; font-weight:600; font-size:.95rem; line-height:1.2; color:var(--text); background:rgba(255,255,255,.1); border:1px solid rgba(255,255,255,.14); border-radius:12px; padding:11px 18px; cursor:pointer;}
  .mx-btn:hover{background:rgba(255,255,255,.18);}
  .mx-btn:disabled{opacity:.55; cursor:default;}
  .mx-btn.primary{background:var(--a); border-color:var(--a); color:#fff;}
  .mx-btn.primary:hover{filter:brightness(1.1);}
  .mx-btn.danger{background:rgba(239,68,68,.14); border-color:rgba(239,68,68,.5); color:#FCA5A5;}
  .mx-btn.danger:hover{background:rgba(239,68,68,.26);}
  .mx-btn.go{background:var(--correct); border-color:var(--correct); color:#fff; font-size:1.35rem; padding:18px 42px; border-radius:999px; box-shadow:0 8px 24px rgba(0,0,0,.45);}
  .mx-btn.go:hover{filter:brightness(1.08);}

  /* ---- Halaman awal ---- */
  #mxStart{overflow-y:auto; padding:28px 16px; text-align:center;}
  .mx-wash{position:absolute; top:0; bottom:0; width:50%; pointer-events:none;}
  .mx-wash-a{left:0; background:linear-gradient(90deg, rgba(59,130,246,.24), rgba(59,130,246,0) 90%);}
  .mx-wash-b{right:0; background:linear-gradient(270deg, rgba(245,158,11,.22), rgba(245,158,11,0) 90%);}
  .mx-mid{position:absolute; top:0; bottom:0; left:50%; width:2px; transform:translateX(-1px); pointer-events:none;
    background:linear-gradient(to bottom, transparent 0%, rgba(255,255,255,.55) 12%, rgba(255,255,255,.55) 88%, transparent 100%);}
  .mx-sheet{position:relative; z-index:1; margin:auto; width:100%; max-width:720px; padding:38px 40px 30px; border-radius:24px;
    background:var(--panel); border:1px solid rgba(255,255,255,.1); backdrop-filter:blur(8px);}
  .mx-title{font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:clamp(2rem,5.2vw,3.5rem); line-height:1.1; letter-spacing:-.5px;}
  .mx-sub{color:var(--muted); font-size:1.05rem; margin-top:12px;}
  .mx-play{display:flex; align-items:center; justify-content:center; gap:22px; margin:30px 0 16px;}
  .mx-tile{font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:2.6rem; width:72px; height:72px; min-width:72px; border-radius:18px;
    display:flex; align-items:center; justify-content:center; color:#fff;}
  .mx-tile.a{background:var(--a);}
  .mx-tile.b{background:var(--b);}
  .mx-summary{color:var(--muted); font-size:.95rem; min-height:1.4em;}
  .mx-summary.warn{color:#FBBF24;}

  .mx-card{margin-top:24px; padding:18px 20px; border-radius:16px; background:rgba(255,255,255,.05); border:1px solid rgba(255,255,255,.08); text-align:left;}
  .mx-card-row{display:flex; flex-wrap:wrap; align-items:center; justify-content:space-between; gap:12px 18px;}
  .mx-card-title{font-family:'Space Grotesk',sans-serif; font-weight:600; font-size:1.05rem;}
  .mx-hint{color:var(--muted); font-size:.85rem; margin-top:4px; max-width:340px;}
  .mx-stepper{display:flex; align-items:center; gap:8px;}
  .mx-stepper button{width:40px; height:40px; border-radius:10px; border:1px solid rgba(255,255,255,.14); background:rgba(255,255,255,.1); color:var(--text); font-size:1.3rem; line-height:1; cursor:pointer;}
  .mx-stepper button:hover{background:rgba(255,255,255,.18);}
  .mx-stepper input{width:76px; height:40px; text-align:center; border-radius:10px; border:1px solid rgba(255,255,255,.18); background:#0D1424; color:var(--text);
    font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:1.2rem; -moz-appearance:textfield; appearance:textfield;}
  .mx-stepper input::-webkit-outer-spin-button,.mx-stepper input::-webkit-inner-spin-button{-webkit-appearance:none; margin:0;}
  .mx-unit{color:var(--muted); font-size:.95rem;}
  .mx-chips{display:flex; flex-wrap:wrap; gap:8px; margin-top:14px;}
  .mx-chip{font-family:'Space Grotesk',sans-serif; font-weight:600; font-size:.9rem; padding:7px 15px; border-radius:999px; cursor:pointer;
    color:var(--text); background:transparent; border:1px solid rgba(255,255,255,.22);}
  .mx-chip:hover{background:rgba(255,255,255,.1);}
  .mx-chip[aria-pressed="true"]{background:var(--a); border-color:var(--a); color:#fff;}
  .mx-actions{display:flex; flex-wrap:wrap; justify-content:center; gap:10px; margin-top:16px;}
  .mx-links{display:flex; flex-wrap:wrap; justify-content:center; gap:4px 22px; margin-top:16px;}
  .mx-link{background:none; border:none; color:var(--muted); font:600 .9rem 'Inter',sans-serif; text-decoration:underline; cursor:pointer; padding:6px;}
  .mx-link:hover{color:var(--text);}

  /* ---- Editor soal ---- */
  #mxEditor{flex-direction:column;}
  .mx-ed-head{display:flex; flex-wrap:wrap; align-items:center; gap:12px 18px; padding:14px 24px; background:#0A101D; border-bottom:1px solid rgba(255,255,255,.1);}
  .mx-ed-title h2{font-family:'Space Grotesk',sans-serif; font-size:1.4rem; line-height:1.2;}
  .mx-ed-meta{color:var(--muted); font-size:.85rem; margin-top:2px;}
  .mx-ed-meta .warn{color:#FBBF24;}
  .mx-ed-actions{display:flex; flex-wrap:wrap; gap:8px; margin-left:auto;}
  .mx-ed-scroll{flex:1; overflow-y:auto; padding:20px 24px 80px;}
  .mx-ed-inner{max-width:960px; margin:0 auto;}
  .mx-q{display:grid; grid-template-columns:40px 1fr auto; gap:14px; padding:16px; margin-bottom:12px; border-radius:14px;
    background:rgba(255,255,255,.04); border:1px solid rgba(255,255,255,.08);}
  .mx-q.mx-incomplete{border-color:rgba(245,158,11,.65);}
  .mx-q-num{width:40px; height:40px; border-radius:10px; display:flex; align-items:center; justify-content:center;
    font-family:'Space Grotesk',sans-serif; font-weight:700; color:var(--muted); background:rgba(255,255,255,.08);}
  .mx-q-body{display:flex; flex-direction:column; gap:12px; min-width:0;}
  .mx-two{display:grid; grid-template-columns:1fr 1fr; gap:12px;}
  .mx-f{display:flex; flex-direction:column; gap:6px; font-size:.82rem; font-weight:600; color:var(--muted); min-width:0;}
  .mx-f.ok span{color:var(--correct);}
  .mx-f.no span{color:#F87171;}
  .mx-in{width:100%; background:#0D1424; border:1px solid rgba(255,255,255,.16); color:var(--text); border-radius:10px; padding:10px 12px;
    font:500 1rem 'Inter',sans-serif; line-height:1.4; resize:vertical;}
  .mx-in:focus{border-color:var(--a); outline:none;}
  .mx-q-ctrl{display:flex; flex-direction:column; gap:6px;}
  .mx-icon{width:36px; height:36px; border-radius:9px; border:1px solid rgba(255,255,255,.14); background:rgba(255,255,255,.08); color:var(--text); font-size:1rem; cursor:pointer;}
  .mx-icon:hover:not(:disabled){background:rgba(255,255,255,.18);}
  .mx-icon:disabled{opacity:.3; cursor:default;}
  .mx-icon.del:hover:not(:disabled){background:rgba(239,68,68,.35);}

  .mx-pos{display:flex; flex-wrap:wrap; align-items:center; gap:8px 12px;}
  .mx-pos-label{font-size:.82rem; font-weight:600; color:var(--muted);}
  .mx-seg{display:inline-flex; border:1px solid rgba(255,255,255,.18); border-radius:10px; overflow:hidden;}
  .mx-seg-btn{background:transparent; border:none; color:var(--text); font:600 .88rem 'Space Grotesk',sans-serif; padding:8px 14px; cursor:pointer;}
  .mx-seg-btn + .mx-seg-btn{border-left:1px solid rgba(255,255,255,.18);}
  .mx-seg-btn:hover{background:rgba(255,255,255,.1);}
  .mx-seg-btn[aria-pressed="true"][data-v="acak"]{background:rgba(255,255,255,.24);}
  .mx-seg-btn[aria-pressed="true"][data-v="a"]{background:var(--a); color:#fff;}
  .mx-seg-btn[aria-pressed="true"][data-v="b"]{background:var(--b); color:#fff;}
  .mx-fs{width:44px; height:44px; min-width:44px; padding:0; border-radius:12px; border:1px solid rgba(255,255,255,.16); background:rgba(255,255,255,.1);
    color:var(--text); display:inline-flex; align-items:center; justify-content:center; cursor:pointer;}
  .mx-fs:hover{background:rgba(255,255,255,.2);}
  .mx-fs:focus-visible{outline:2px solid #fff; outline-offset:2px;}
  .mx-fs svg{width:22px; height:22px;}
  #mxFsStart{position:absolute; top:16px; right:16px; z-index:55; background:rgba(7,11,20,.7); backdrop-filter:blur(6px);}
  @media (max-width:900px){ #mxStart{padding-top:76px;} }
  /* ---- Arena: panel guru disembunyikan; kontrol dipindah ke tombol kecil ---- */
  #panel,#panelToggle,#checkBtnCenter{display:none !important;}
  .mx-game-btn{font-family:'Space Grotesk',sans-serif; font-weight:600; font-size:.9rem; line-height:1.2; color:#fff; border:none; cursor:pointer;
    padding:10px 20px; border-radius:999px; box-shadow:0 6px 18px rgba(0,0,0,.45);}
  .mx-game-btn:hover{filter:brightness(1.12);}
  .mx-game-btn:focus-visible{outline:2px solid #fff; outline-offset:2px;}
  .mx-game-btn.neutral{background:rgba(7,11,20,.92); border:1px solid rgba(255,255,255,.22);}
  .mx-game-btn.go{background:var(--correct);}
  #mxGameMenu{position:absolute; top:24px; left:88px; z-index:18; height:44px; display:inline-flex; align-items:center; gap:6px; padding:0 16px;}
  #mxGameBtns{position:absolute; bottom:18px; left:50%; transform:translateX(-50%); z-index:16; display:flex; gap:8px;}
  /* pilihan jawaban kini di bawah soal (bukan di dasar layar), jadi tidak lagi bertabrakan dengan tombol tengah */
  .ans-text{max-width:min(520px, calc(50vw - 140px));}
  @media (max-width:860px){ #mxGameBtns .mx-game-btn{padding:9px 14px; font-size:.82rem;} }
  .mx-empty{color:var(--muted); text-align:center; padding:40px 0;}
  .mx-ed-foot{display:flex; justify-content:center; margin-top:18px;}

  /* ---- Notifikasi ---- */
  #mxToast{position:absolute; left:50%; bottom:24px; transform:translate(-50%,20px); z-index:90; display:flex; align-items:center; gap:14px;
    max-width:min(560px,92vw); padding:12px 18px; border-radius:14px; background:#111A2E; border:1px solid rgba(255,255,255,.18);
    color:var(--text); font-size:.95rem; box-shadow:0 10px 30px rgba(0,0,0,.5); opacity:0; pointer-events:none; transition:opacity .2s ease, transform .2s ease;}
  #mxToast.show{opacity:1; transform:translate(-50%,0); pointer-events:auto;}
  #mxToast button{background:none; border:none; color:#93C5FD; font:700 .95rem 'Inter',sans-serif; cursor:pointer; text-decoration:underline; white-space:nowrap;}

  @media (max-width:720px){
    .mx-sheet{padding:28px 20px 24px;}
    .mx-tile{display:none;}
    .mx-q{grid-template-columns:32px 1fr;}
    .mx-q-num{width:32px; height:32px;}
    .mx-q-ctrl{grid-column:1 / -1; flex-direction:row;}
    .mx-two{grid-template-columns:1fr;}
    .mx-ed-head,.mx-ed-scroll{padding-left:14px; padding-right:14px;}
    .mx-ed-actions{margin-left:0;}
  }
</style>
</head>
<body>

<div id="stage">
  <div id="cam-wrap">
    <video id="video" autoplay muted playsinline></video>
    <div id="cam-placeholder">
      <p>Aktifkan kamera agar siswa dapat memilih sisi A (kiri) atau B (kanan) di depan layar.</p>
      <button id="startCamBtn">Aktifkan Kamera</button>
    </div>
  </div>

  <div id="divider"></div>
  <div class="zone" id="zoneA"></div>
  <div class="zone" id="zoneB"></div>
  <div id="timerBadge">10</div>
  <div id="ownerWatermark">AES</div>

  <div id="topArea">
    <div id="qbox">
      <div id="qcounter">SOAL <span id="qNum">1</span> / <span id="qTotal">10</span></div>
      <div id="qtext"></div>
    </div>

    <div id="answersRow">
      <div class="answer-card" id="cardA">
        <div class="letter">A</div>
        <div class="ans-text" id="ansAText"></div>
      </div>
      <div class="answer-card" id="cardB">
        <div class="letter">B</div>
        <div class="ans-text" id="ansBText"></div>
      </div>
    </div>
  </div>

  <div id="resultBanner"></div>

  <button id="checkBtnCenter">Periksa Jawaban</button>

  <button id="panelToggle">&#9881;</button>
  <div id="panel">
    <h3>Panel Guru</h3>
    <button class="btn" id="prevBtn">&larr; Soal Sebelumnya</button>
    <button class="btn" id="nextBtn">Soal Berikutnya &rarr;</button>
    <button class="btn" id="timerBtn">Ulangi Waktu 10 Detik</button>
    <button class="btn" id="fsBtn">Layar Penuh</button>
    <button class="btn" id="resetBtn">Ulangi dari Awal</button>
  </div>

  <div id="finishScreen">
    <h2>Kuis Selesai!</h2>
    <p>Seluruh 10 soal telah dibahas. Klik tombol di bawah untuk mengulang dari soal pertama.</p>
    <button class="btn" id="restartBtn" style="background:var(--a); padding:14px 28px; font-size:1rem;">Mulai Ulang</button>
  </div>
</div>

<script>
const questions = [
  {q:"Ibu kota negara Indonesia adalah...", correct:"Jakarta", wrong:"Bandung"},
  {q:"Alat yang digunakan untuk memasukkan data ke dalam komputer disebut...", correct:"Perangkat Input", wrong:"Perangkat Output"},
  {q:"Hasil dari 5 \u00d7 6 adalah...", correct:"30", wrong:"35"},
  {q:"Bahasa yang digunakan untuk membuat halaman web adalah...", correct:"HTML", wrong:"Microsoft Excel"},
  {q:"Planet terbesar di tata surya kita adalah...", correct:"Jupiter", wrong:"Bumi"},
  {q:"CPU merupakan singkatan dari...", correct:"Central Processing Unit", wrong:"Computer Personal Unit"},
  {q:"Hewan yang dikenal sebagai raja hutan adalah...", correct:"Singa", wrong:"Gajah"},
  {q:"Pada flowchart, simbol berbentuk oval digunakan untuk menandai...", correct:"Mulai / Selesai (Start/End)", wrong:"Proses"},
  {q:"1 kilogram sama dengan...", correct:"1000 gram", wrong:"100 gram"},
  {q:"Kumpulan instruksi yang memberi perintah kepada komputer disebut...", correct:"Program / Algoritma", wrong:"Hardware"},
  {q:"Matahari terbit dari arah...", correct:"Timur", wrong:"Barat"},
  {q:"RAM pada komputer berfungsi untuk...", correct:"Menyimpan data sementara saat komputer menyala", wrong:"Menyimpan data secara permanen selamanya"},
  {q:"Benua terluas di dunia adalah...", correct:"Asia", wrong:"Eropa"},
  {q:"Kegiatan menyusun langkah-langkah untuk menyelesaikan masalah disebut...", correct:"Algoritma", wrong:"Antivirus"},
  {q:"Proklamasi Kemerdekaan Indonesia dibacakan pada tanggal...", correct:"17 Agustus 1945", wrong:"20 Mei 1945"},
  {q:"Jaringan komputer yang menghubungkan perangkat di seluruh dunia disebut...", correct:"Internet", wrong:"LAN"},
  {q:"Air akan berubah menjadi es apabila...", correct:"Didinginkan", wrong:"Dipanaskan"},
  {q:"File yang berisi gambar biasanya memiliki format...", correct:".jpg atau .png", wrong:".exe"},
  {q:"Bahasa resmi negara Indonesia adalah...", correct:"Bahasa Indonesia", wrong:"Bahasa Inggris"},
  {q:"Proses mencari dan memperbaiki kesalahan pada program disebut...", correct:"Debugging", wrong:"Downloading"}
];

let idx = 0;
let checked = false;
let currentCorrectSide = 'a';
let timeLeft = 10;
let timerInterval = null;

const qNumEl = document.getElementById('qNum');
const qtextEl = document.getElementById('qtext');
const ansAEl = document.getElementById('ansAText');
const ansBEl = document.getElementById('ansBText');
const zoneA = document.getElementById('zoneA');
const zoneB = document.getElementById('zoneB');
const banner = document.getElementById('resultBanner');
const finishScreen = document.getElementById('finishScreen');
const stage = document.getElementById('stage');
const checkBtnCenter = document.getElementById('checkBtnCenter');
const timerBadge = document.getElementById('timerBadge');

function startTimer(){
  clearInterval(timerInterval);
  timeLeft = 10;
  timerBadge.textContent = timeLeft;
  timerBadge.classList.remove('urgent','done');
  timerInterval = setInterval(() => {
    timeLeft--;
    if(timeLeft <= 0){
      clearInterval(timerInterval);
      timerBadge.textContent = '0';
      timerBadge.classList.remove('urgent');
      timerBadge.classList.add('done');
    } else {
      timerBadge.textContent = timeLeft;
      if(timeLeft <= 3) timerBadge.classList.add('urgent');
    }
  }, 1000);
}

document.getElementById('qTotal').textContent = questions.length;

function loadQuestion(){
  const item = questions[idx];
  qNumEl.textContent = idx + 1;
  qtextEl.textContent = item.q;
  currentCorrectSide = Math.random() < 0.5 ? 'a' : 'b';
  ansAEl.textContent = currentCorrectSide === 'a' ? item.correct : item.wrong;
  ansBEl.textContent = currentCorrectSide === 'b' ? item.correct : item.wrong;
  zoneA.classList.remove('correct','wrong');
  zoneB.classList.remove('correct','wrong');
  banner.classList.remove('show','correct-flash','wrong-flash');
  checked = false;
  checkBtnCenter.disabled = false;
  finishScreen.style.display = 'none';
  startTimer();
}

function checkAnswer(){
  if(checked) return;
  checked = true;
  const correctZone = currentCorrectSide === 'a' ? zoneA : zoneB;
  const wrongZone = currentCorrectSide === 'a' ? zoneB : zoneA;
  correctZone.classList.add('correct');
  wrongZone.classList.add('wrong');
  banner.textContent = 'JAWABAN: ' + currentCorrectSide.toUpperCase();
  banner.classList.add('show','correct-flash');
  checkBtnCenter.disabled = true;
}

function nextQuestion(){
  if(idx >= questions.length - 1){
    clearInterval(timerInterval);
    finishScreen.style.display = 'flex';
    return;
  }
  idx++;
  loadQuestion();
}

function prevQuestion(){
  if(idx === 0) return;
  idx--;
  loadQuestion();
}

function resetQuiz(){
  idx = 0;
  loadQuestion();
  finishScreen.style.display = 'none';
}

document.getElementById('checkBtnCenter').addEventListener('click', checkAnswer);
document.getElementById('timerBtn').addEventListener('click', startTimer);
document.getElementById('nextBtn').addEventListener('click', nextQuestion);
document.getElementById('prevBtn').addEventListener('click', prevQuestion);
document.getElementById('resetBtn').addEventListener('click', resetQuiz);
document.getElementById('restartBtn').addEventListener('click', resetQuiz);

document.getElementById('fsBtn').addEventListener('click', () => {
  if(!document.fullscreenElement){
    stage.requestFullscreen().catch(()=>{});
  } else {
    document.exitFullscreen().catch(()=>{});
  }
});

const panel = document.getElementById('panel');
document.getElementById('panelToggle').addEventListener('click', () => {
  panel.classList.toggle('collapsed');
});

// Camera
const video = document.getElementById('video');
const camPlaceholder = document.getElementById('cam-placeholder');
document.getElementById('startCamBtn').addEventListener('click', async () => {
  try{
    const stream = await navigator.mediaDevices.getUserMedia({
      video:{
        facingMode:'user',
        width:{ideal:1280},
        height:{ideal:720}
      },
      audio:false
    });
    video.srcObject = stream;
    camPlaceholder.style.display = 'none';
  }catch(err){
    camPlaceholder.querySelector('p').textContent = 'Kamera tidak dapat diakses. Periksa izin kamera pada perangkat/browser lalu coba lagi.';
  }
});

loadQuestion();
</script>
<!-- ===== TAMBAHAN: menu awal, editor soal, timer otomatis, simpan lokal, impor/ekspor JSON ===== -->
<script id="mx-addon-script">
(function(){
  'use strict';

  var st = document.getElementById('stage');
  var KEY_Q = 'kuisBK.v1.soal';
  var KEY_T = 'kuisBK.v1.timer';
  var MIN_T = 3, MAX_T = 120;
  var PRESETS = [5, 10, 15, 20, 30, 60];

  // Salinan soal bawaan (sebelum diganti oleh soal tersimpan)
  function normSide(v){
    var s = String(v === undefined || v === null ? '' : v).trim().toLowerCase();
    if(s === 'a' || s === 'kiri' || s === 'left' || s === 'l') return 'a';
    if(s === 'b' || s === 'kanan' || s === 'right' || s === 'r') return 'b';
    return 'acak';
  }
  function cp(x){ return {q:x.q, correct:x.correct, wrong:x.wrong, side:normSide(x.side)}; }

  var DEFAULT_BANK = questions.map(function(x){ return cp(x); });
  var bank = null;
  var timerSeconds = 10;
  var saveTimer = null;
  var toastTimer = null;

  /* ---------- Markup ---------- */
  st.insertAdjacentHTML('beforeend', `
    <div id="mxStart" class="mx-screen">
      <div class="mx-wash mx-wash-a"></div>
      <div class="mx-wash mx-wash-b"></div>
      <div class="mx-mid"></div>
      <div class="mx-sheet">
        <h1 class="mx-title">Kuis Interaktif</h1>
        <p class="mx-sub">Siswa memilih sisi A di kiri atau sisi B di kanan layar.</p>

        <div class="mx-play">
          <div class="mx-tile a">A</div>
          <button type="button" class="mx-btn go" id="mxPlay">Mulai kuis</button>
          <div class="mx-tile b">B</div>
        </div>
        <p class="mx-summary" id="mxSummary"></p>

        <div class="mx-card">
          <div class="mx-card-row">
            <div>
              <div class="mx-card-title">Timer per soal</div>
              <div class="mx-hint">Saat waktu habis, layar otomatis menampilkan jawaban yang benar.</div>
            </div>
            <div class="mx-stepper">
              <button type="button" id="mxTMinus" aria-label="Kurangi timer">&minus;</button>
              <input type="number" id="mxTInput" min="3" max="120" inputmode="numeric" aria-label="Detik per soal">
              <button type="button" id="mxTPlus" aria-label="Tambah timer">+</button>
              <span class="mx-unit">detik</span>
            </div>
          </div>
          <div class="mx-chips" id="mxChips"></div>
        </div>

        <div class="mx-actions">
          <button type="button" class="mx-btn" id="mxEditBtn">Edit soal</button>
          <button type="button" class="mx-btn" id="mxCamBtn">Aktifkan kamera</button>
        </div>
        <div class="mx-links">
          <button type="button" class="mx-link" id="mxImportS">Impor soal (.json)</button>
          <button type="button" class="mx-link" id="mxExportS">Ekspor soal (.json)</button>
        </div>
      </div>
    </div>

    <button type="button" class="mx-fs" id="mxFsStart" title="Layar penuh" aria-label="Layar penuh"></button>

    <div id="mxEditor" class="mx-screen" style="display:none">
      <div class="mx-ed-head">
        <button type="button" class="mx-btn" id="mxEdBack">&larr; Kembali</button>
        <div class="mx-ed-title">
          <h2>Edit soal</h2>
          <div class="mx-ed-meta"><span id="mxEdInfo"></span> <span id="mxSaved"></span></div>
        </div>
        <div class="mx-ed-actions">
          <button type="button" class="mx-btn primary" id="mxAddTop">Tambah soal</button>
          <button type="button" class="mx-btn" id="mxImportE">Impor JSON</button>
          <button type="button" class="mx-btn" id="mxExportE">Ekspor JSON</button>
          <button type="button" class="mx-btn danger" id="mxResetE">Kembalikan soal awal</button>
          <button type="button" class="mx-fs" id="mxFsEd" title="Layar penuh" aria-label="Layar penuh"></button>
        </div>
      </div>
      <div class="mx-ed-scroll" id="mxScroll">
        <div class="mx-ed-inner">
          <div id="mxList"></div>
          <div class="mx-ed-foot"><button type="button" class="mx-btn primary" id="mxAddBottom">Tambah soal</button></div>
        </div>
      </div>
    </div>

    <button type="button" class="mx-game-btn neutral" id="mxGameMenu" style="display:none" aria-label="Kembali ke menu utama">&larr; Menu</button>
    <div id="mxGameBtns" style="display:none">
      <button type="button" class="mx-game-btn neutral" id="mxRepeat">Ulangi</button>
      <button type="button" class="mx-game-btn go" id="mxNext">Soal Selanjutnya</button>
    </div>

    <input type="file" id="mxFile" accept=".json,application/json" style="display:none">
    <div id="mxToast" role="status" aria-live="polite"><span id="mxToastMsg"></span><button type="button" id="mxToastAct" style="display:none"></button></div>
  `);

  function $(id){ return document.getElementById(id); }
  var startEl = $('mxStart'), edEl = $('mxEditor'), listEl = $('mxList'), scrollEl = $('mxScroll');
  var summaryEl = $('mxSummary'), tInput = $('mxTInput'), chipsEl = $('mxChips'), camBtn = $('mxCamBtn');
  var edInfo = $('mxEdInfo'), edSaved = $('mxSaved'), fileEl = $('mxFile');
  var toastEl = $('mxToast'), toastMsg = $('mxToastMsg'), toastAct = $('mxToastAct');
  var fsStartEl = $('mxFsStart');
  var gameMenuEl = $('mxGameMenu'), gameBtnsEl = $('mxGameBtns');
  function gameUI(on){ gameMenuEl.style.display = on ? '' : 'none'; gameBtnsEl.style.display = on ? '' : 'none'; }

  /* ---------- Data & penyimpanan lokal ---------- */
  function norm(o){
    if(!o || typeof o !== 'object') return null;
    function pick(keys){
      for(var i = 0; i < keys.length; i++){
        var v = o[keys[i]];
        if(v !== undefined && v !== null) return String(v);
      }
      return '';
    }
    return {
      q: pick(['q','pertanyaan','soal','question']),
      correct: pick(['correct','benar','jawabanBenar','jawaban_benar']),
      wrong: pick(['wrong','salah','jawabanSalah','jawaban_salah']),
      side: normSide(pick(['side','posisi','letak']))
    };
  }
  function clampT(v){
    if(v === undefined || v === null || v === '') return null;
    var n = Math.round(Number(v));
    if(!isFinite(n)) return null;
    return Math.min(MAX_T, Math.max(MIN_T, n));
  }
  function isComplete(it){ return it.q.trim() !== '' && it.correct.trim() !== '' && it.wrong.trim() !== ''; }
  function validBank(){
    return bank.filter(isComplete).map(function(it){
      return {q:it.q.trim(), correct:it.correct.trim(), wrong:it.wrong.trim(), side:normSide(it.side)};
    });
  }

  function load(){
    try{
      var raw = localStorage.getItem(KEY_Q);
      if(raw){
        var arr = JSON.parse(raw);
        if(Array.isArray(arr)) bank = arr.map(norm).filter(Boolean);
      }
      var t = clampT(localStorage.getItem(KEY_T));
      if(t !== null) timerSeconds = t;
    }catch(e){ /* penyimpanan tidak tersedia: pakai soal bawaan */ }
    if(!bank) bank = DEFAULT_BANK.map(function(x){ return cp(x); });
  }

  function setSaved(ok){
    edSaved.textContent = ok ? 'Tersimpan otomatis di perangkat ini.' : 'Browser menolak penyimpanan. Gunakan Ekspor JSON agar soal tidak hilang.';
    edSaved.className = ok ? '' : 'warn';
  }
  function persist(){
    try{
      localStorage.setItem(KEY_Q, JSON.stringify(bank));
      localStorage.setItem(KEY_T, String(timerSeconds));
      setSaved(true);
    }catch(e){ setSaved(false); }
  }
  function queueSave(){
    edSaved.textContent = 'Menyimpan...';
    edSaved.className = '';
    clearTimeout(saveTimer);
    saveTimer = setTimeout(persist, 300);
  }
  function flushSave(){
    if(saveTimer){ clearTimeout(saveTimer); saveTimer = null; persist(); }
  }
  window.addEventListener('pagehide', flushSave);

  /* ---------- Notifikasi ---------- */
  function hideToast(){ toastEl.classList.remove('show'); }
  function toast(msg, actLabel, actFn){
    toastMsg.textContent = msg;
    if(actLabel){
      toastAct.style.display = 'inline-block';
      toastAct.textContent = actLabel;
      toastAct.onclick = function(){ hideToast(); if(actFn) actFn(); };
    } else {
      toastAct.style.display = 'none';
      toastAct.onclick = null;
    }
    toastEl.classList.add('show');
    clearTimeout(toastTimer);
    toastTimer = setTimeout(hideToast, actLabel ? 7000 : 4000);
  }

  /* ---------- Timer ---------- */
  var timerBtnNew = (function(){
    var old = $('timerBtn');
    var fresh = old.cloneNode(true);          // buang listener lama yang terpasang ke startTimer bawaan
    old.parentNode.replaceChild(fresh, old);
    fresh.addEventListener('click', function(){ startTimer(); });
    return fresh;
  })();

  // Menggantikan startTimer bawaan: durasi bisa diatur, dan otomatis memeriksa jawaban saat waktu habis.
  startTimer = function(){
    clearInterval(timerInterval);
    timeLeft = timerSeconds;
    timerBadge.textContent = timeLeft;
    timerBadge.classList.remove('urgent','done');
    timerInterval = setInterval(function(){
      timeLeft--;
      if(timeLeft <= 0){
        clearInterval(timerInterval);
        timerBadge.textContent = '0';
        timerBadge.classList.remove('urgent');
        timerBadge.classList.add('done');
        checkAnswer();                         // layar otomatis hijau/merah sesuai logika game
      } else {
        timerBadge.textContent = timeLeft;
        if(timeLeft <= 3) timerBadge.classList.add('urgent');
      }
    }, 1000);
  };

  // Membungkus loadQuestion bawaan: jika guru menentukan letak jawaban benar (A/B), pakai itu. "Acak" = perilaku asli.
  (function(){
    var origLoad = loadQuestion;
    loadQuestion = function(){
      origLoad();
      var it = questions[idx];
      if(it && (it.side === 'a' || it.side === 'b')){
        currentCorrectSide = it.side;
        ansAEl.textContent = it.side === 'a' ? it.correct : it.wrong;
        ansBEl.textContent = it.side === 'b' ? it.correct : it.wrong;
      }
    };
  })();

  function buildChips(){
    PRESETS.forEach(function(n){
      var b = document.createElement('button');
      b.type = 'button';
      b.className = 'mx-chip';
      b.textContent = n + ' detik';
      b.dataset.v = n;
      b.addEventListener('click', function(){ setTimer(n); });
      chipsEl.appendChild(b);
    });
  }
  function syncTimerUI(){
    tInput.value = timerSeconds;
    Array.prototype.forEach.call(chipsEl.children, function(c){
      c.setAttribute('aria-pressed', String(Number(c.dataset.v) === timerSeconds));
    });
    timerBtnNew.textContent = 'Ulangi Waktu ' + timerSeconds + ' Detik';
    refreshSummary();
  }
  function setTimer(n){
    timerSeconds = n;
    syncTimerUI();
    persist();
  }
  $('mxTMinus').addEventListener('click', function(){ setTimer(Math.max(MIN_T, timerSeconds - 1)); });
  $('mxTPlus').addEventListener('click', function(){ setTimer(Math.min(MAX_T, timerSeconds + 1)); });
  tInput.addEventListener('change', function(){
    var n = clampT(tInput.value);
    if(n === null) n = timerSeconds;
    setTimer(n);
  });
  tInput.addEventListener('keydown', function(e){ if(e.key === 'Enter') tInput.blur(); });

  /* ---------- Halaman awal ---------- */
  function refreshSummary(){
    var n = validBank().length, inc = bank.length - n;
    if(n === 0){
      summaryEl.textContent = 'Belum ada soal yang lengkap. Buka Edit soal untuk menambahkannya.';
    } else {
      summaryEl.textContent = n + ' soal siap dimainkan, ' + timerSeconds + ' detik per soal.' +
        (inc > 0 ? ' ' + inc + ' soal belum lengkap tidak ikut dimainkan.' : '');
    }
    summaryEl.classList.toggle('warn', n === 0 || inc > 0);
  }

  function refreshCam(){
    var on = camPlaceholder.style.display === 'none';
    camBtn.textContent = on ? 'Kamera aktif' : 'Aktifkan kamera';
    camBtn.disabled = on;
  }
  camBtn.addEventListener('click', function(){ $('startCamBtn').click(); });
  video.addEventListener('playing', refreshCam);

  function showMenu(){
    clearInterval(timerInterval);
    timerBadge.textContent = timerSeconds;
    timerBadge.classList.remove('urgent','done');
    edEl.style.display = 'none';
    startEl.style.display = 'flex';
    fsStartEl.style.display = 'flex';
    gameUI(false);
    hideToast();
    refreshSummary();
    refreshCam();
  }

  function applyBank(list){
    questions.splice(0, questions.length);
    list.forEach(function(x){ questions.push(x); });
    $('qTotal').textContent = questions.length;
    var fp = finishScreen.querySelector('p');
    if(fp) fp.textContent = 'Seluruh ' + questions.length + ' soal telah dibahas. Klik tombol di bawah untuk mengulang dari soal pertama.';
  }

  function play(){
    flushSave();
    var list = validBank();
    if(!list.length){ toast('Belum ada soal yang lengkap. Buka Edit soal untuk menambahkannya.'); return; }
    applyBank(list);
    startEl.style.display = 'none';
    fsStartEl.style.display = 'none';
    resetQuiz();
    gameUI(true);
  }
  $('mxPlay').addEventListener('click', play);

  /* ---------- Editor soal ---------- */
  function updateInfo(){
    var inc = bank.filter(function(x){ return !isComplete(x); }).length;
    edInfo.innerHTML = '';
    edInfo.appendChild(document.createTextNode(bank.length + ' soal'));
    if(inc > 0){
      var w = document.createElement('span');
      w.className = 'warn';
      w.textContent = ', ' + inc + ' belum lengkap dan tidak akan dimainkan.';
      edInfo.appendChild(w);
    } else {
      edInfo.appendChild(document.createTextNode('.'));
    }
  }

  function field(label, cls, tag, item, key, card){
    var wrap = document.createElement('label');
    wrap.className = 'mx-f ' + cls;
    var span = document.createElement('span');
    span.textContent = label;
    var el = document.createElement(tag);
    el.className = 'mx-in';
    if(tag === 'textarea') el.rows = 2; else el.type = 'text';
    el.value = item[key];
    el.addEventListener('input', function(){
      item[key] = el.value;
      card.classList.toggle('mx-incomplete', !isComplete(item));
      updateInfo();
      queueSave();
    });
    wrap.appendChild(span);
    wrap.appendChild(el);
    return wrap;
  }
  function iconBtn(text, title, cls, disabled, onClick){
    var b = document.createElement('button');
    b.type = 'button';
    b.className = 'mx-icon ' + cls;
    b.textContent = text;
    b.title = title;
    b.setAttribute('aria-label', title);
    b.disabled = !!disabled;
    b.addEventListener('click', onClick);
    return b;
  }
  function move(i, d){
    var j = i + d;
    if(j < 0 || j >= bank.length) return;
    var t = bank[i]; bank[i] = bank[j]; bank[j] = t;
    render();
    queueSave();
  }
  function remove(i){
    var removed = bank.splice(i, 1)[0];
    render();
    queueSave();
    toast('Soal ' + (i + 1) + ' dihapus.', 'Urungkan', function(){
      bank.splice(i, 0, removed);
      render();
      queueSave();
    });
  }
  function addQuestion(){
    bank.push({q:'', correct:'', wrong:'', side:'acak'});
    render();
    queueSave();
    scrollEl.scrollTop = scrollEl.scrollHeight;
    var areas = listEl.querySelectorAll('textarea');
    if(areas.length) areas[areas.length - 1].focus();
  }

  function render(){
    var keep = scrollEl.scrollTop;
    listEl.textContent = '';
    if(!bank.length){
      var p = document.createElement('p');
      p.className = 'mx-empty';
      p.textContent = 'Belum ada soal. Tambahkan soal baru atau impor file JSON.';
      listEl.appendChild(p);
    }
    bank.forEach(function(item, i){
      var card = document.createElement('div');
      card.className = 'mx-q' + (isComplete(item) ? '' : ' mx-incomplete');

      var num = document.createElement('div');
      num.className = 'mx-q-num';
      num.textContent = i + 1;

      var body = document.createElement('div');
      body.className = 'mx-q-body';
      body.appendChild(field('Pertanyaan', '', 'textarea', item, 'q', card));
      var two = document.createElement('div');
      two.className = 'mx-two';
      two.appendChild(field('Jawaban benar', 'ok', 'input', item, 'correct', card));
      two.appendChild(field('Jawaban salah', 'no', 'input', item, 'wrong', card));
      body.appendChild(two);

      var pos = document.createElement('div');
      pos.className = 'mx-pos';
      var posLabel = document.createElement('span');
      posLabel.className = 'mx-pos-label';
      posLabel.textContent = 'Letak jawaban benar';
      var seg = document.createElement('div');
      seg.className = 'mx-seg';
      seg.setAttribute('role', 'group');
      seg.setAttribute('aria-label', 'Letak jawaban benar untuk soal ' + (i + 1));
      [['acak','Acak'],['a','A (kiri)'],['b','B (kanan)']].forEach(function(o){
        var sb = document.createElement('button');
        sb.type = 'button';
        sb.className = 'mx-seg-btn';
        sb.dataset.v = o[0];
        sb.textContent = o[1];
        sb.setAttribute('aria-pressed', String(item.side === o[0]));
        sb.addEventListener('click', function(){
          item.side = o[0];
          Array.prototype.forEach.call(seg.children, function(c){ c.setAttribute('aria-pressed', String(c.dataset.v === item.side)); });
          queueSave();
        });
        seg.appendChild(sb);
      });
      pos.appendChild(posLabel);
      pos.appendChild(seg);
      body.appendChild(pos);

      var ctrl = document.createElement('div');
      ctrl.className = 'mx-q-ctrl';
      ctrl.appendChild(iconBtn('\u2191', 'Pindah ke atas', '', i === 0, function(){ move(i, -1); }));
      ctrl.appendChild(iconBtn('\u2193', 'Pindah ke bawah', '', i === bank.length - 1, function(){ move(i, 1); }));
      ctrl.appendChild(iconBtn('\u2715', 'Hapus soal', 'del', false, function(){ remove(i); }));

      card.appendChild(num);
      card.appendChild(body);
      card.appendChild(ctrl);
      listEl.appendChild(card);
    });
    updateInfo();
    scrollEl.scrollTop = keep;
  }

  function openEditor(){
    startEl.style.display = 'none';
    fsStartEl.style.display = 'none';
    gameUI(false);
    edEl.style.display = 'flex';
    setSaved(true);
    render();
    scrollEl.scrollTop = 0;
  }
  function closeEditor(){
    flushSave();
    edEl.style.display = 'none';
    startEl.style.display = 'flex';
    fsStartEl.style.display = 'flex';
    gameUI(false);
    refreshSummary();
  }
  $('mxEditBtn').addEventListener('click', openEditor);
  $('mxEdBack').addEventListener('click', closeEditor);
  $('mxAddTop').addEventListener('click', addQuestion);
  $('mxAddBottom').addEventListener('click', addQuestion);
  $('mxResetE').addEventListener('click', function(){
    if(!confirm('Kembalikan ke ' + DEFAULT_BANK.length + ' soal awal? Semua perubahan soal saat ini akan hilang.')) return;
    bank = DEFAULT_BANK.map(function(x){ return cp(x); });
    render();
    persist();
    toast('Soal dikembalikan ke soal awal.');
  });

  /* ---------- Impor / Ekspor JSON ---------- */
  function exportJSON(){
    flushSave();
    var data = { timer: timerSeconds, questions: bank.map(function(x){ return cp(x); }) };
    var blob = new Blob([JSON.stringify(data, null, 2)], {type:'application/json'});
    var url = URL.createObjectURL(blob);
    var a = document.createElement('a');
    a.href = url;
    a.download = 'soal-kuis.json';
    document.body.appendChild(a);
    a.click();
    a.remove();
    setTimeout(function(){ URL.revokeObjectURL(url); }, 1500);
    toast(bank.length + ' soal diekspor ke file JSON.');
  }

  function handleImport(text){
    var data = JSON.parse(text.replace(/^\uFEFF/, ''));
    var arr = Array.isArray(data) ? data : (data && (data.questions || data.soal));
    if(!Array.isArray(arr)) throw new Error('bentuk file tidak dikenali');
    var list = arr.map(norm).filter(Boolean);
    if(!list.length){ toast('File tidak berisi soal.'); return; }
    if(bank.length && !confirm('Impor akan mengganti ' + bank.length + ' soal yang ada dengan ' + list.length + ' soal dari file. Lanjutkan?')) return;
    bank = list;
    var t = (data && !Array.isArray(data)) ? clampT(data.timer !== undefined ? data.timer : data.waktu) : null;
    if(t !== null) timerSeconds = t;
    persist();
    syncTimerUI();
    if(edEl.style.display !== 'none') render();
    refreshSummary();
    var bad = list.filter(function(x){ return !isComplete(x); }).length;
    toast(list.length + ' soal diimpor' + (t !== null ? ', timer ' + t + ' detik' : '') + '.' + (bad ? ' ' + bad + ' soal belum lengkap.' : ''));
  }

  function pickFile(){ fileEl.value = ''; fileEl.click(); }
  fileEl.addEventListener('change', function(){
    var f = fileEl.files && fileEl.files[0];
    if(!f) return;
    var r = new FileReader();
    r.onload = function(){
      try{ handleImport(String(r.result)); }
      catch(e){ toast('File tidak bisa dibaca. Pastikan isinya JSON yang benar.'); }
      fileEl.value = '';
    };
    r.onerror = function(){ toast('File tidak bisa dibuka.'); };
    r.readAsText(f);
  });
  ['mxImportS','mxImportE'].forEach(function(id){ $(id).addEventListener('click', pickFile); });
  ['mxExportS','mxExportE'].forEach(function(id){ $(id).addEventListener('click', exportJSON); });

  /* ---------- Tombol "Menu utama" di panel guru & layar selesai ---------- */
  gameMenuEl.addEventListener('click', showMenu);
  $('mxNext').addEventListener('click', function(){ nextQuestion(); });
  $('mxRepeat').addEventListener('click', function(){ loadQuestion(); });   // ulangi soal ini: warna dihapus, timer mulai lagi

  var menuBtn2 = document.createElement('button');
  menuBtn2.type = 'button';
  menuBtn2.className = 'btn';
  menuBtn2.textContent = 'Menu Utama';
  menuBtn2.style.cssText = 'background:rgba(255,255,255,.12); color:var(--text); padding:14px 28px; font-size:1rem;';
  finishScreen.appendChild(menuBtn2);
  menuBtn2.addEventListener('click', showMenu);

  /* ---------- Layar penuh (halaman awal & editor) ---------- */
  var SVG_OPEN = '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">';
  var ICON_EXPAND = SVG_OPEN + '<path d="M8 3H5a2 2 0 0 0-2 2v3M16 3h3a2 2 0 0 1 2 2v3M8 21H5a2 2 0 0 1-2-2v-3M16 21h3a2 2 0 0 0 2-2v-3"/></svg>';
  var ICON_COMPRESS = SVG_OPEN + '<path d="M3 8h3a2 2 0 0 0 2-2V3M21 8h-3a2 2 0 0 1-2-2V3M3 16h3a2 2 0 0 1 2 2v3M21 16h-3a2 2 0 0 0-2 2v3"/></svg>';
  var fsBtns = [$('mxFsStart'), $('mxFsEd')];
  function fsActive(){ return !!(document.fullscreenElement || document.webkitFullscreenElement); }
  function syncFs(){
    var on = fsActive();
    var label = on ? 'Keluar layar penuh' : 'Layar penuh';
    fsBtns.forEach(function(b){
      b.innerHTML = on ? ICON_COMPRESS : ICON_EXPAND;
      b.title = label;
      b.setAttribute('aria-label', label);
    });
  }
  function toggleFs(){
    if(fsActive()){
      var ex = document.exitFullscreen || document.webkitExitFullscreen;
      var r1 = ex.call(document);
      if(r1 && r1.catch) r1.catch(function(){});
      return;
    }
    var req = st.requestFullscreen || st.webkitRequestFullscreen;
    if(!req){ toast('Browser ini tidak mendukung layar penuh.'); return; }
    var r = req.call(st);
    if(r && r.catch) r.catch(function(){ toast('Browser memblokir layar penuh.'); });
  }
  fsBtns.forEach(function(b){ b.addEventListener('click', toggleFs); });
  document.addEventListener('fullscreenchange', syncFs);
  document.addEventListener('webkitfullscreenchange', syncFs);
  syncFs();

  /* ---------- Pintasan keyboard di arena (pengganti panel guru) ---------- */
  document.addEventListener('keydown', function(e){
    if(e.ctrlKey || e.metaKey || e.altKey) return;
    var t = e.target;
    if(t && (t.tagName === 'INPUT' || t.tagName === 'TEXTAREA' || t.isContentEditable)) return;
    if(startEl.style.display !== 'none' || edEl.style.display !== 'none') return;   // hanya saat di arena
    if(e.key === 'ArrowRight'){ e.preventDefault(); nextQuestion(); }
    else if(e.key === 'ArrowLeft'){ e.preventDefault(); prevQuestion(); }
    else if(e.key === 'r' || e.key === 'R'){ loadQuestion(); }
    else if(e.key === 'f' || e.key === 'F'){ toggleFs(); }
  });

  /* ---------- Mulai ---------- */
  load();
  buildChips();
  syncTimerUI();
  showMenu();   // menghentikan timer awal bawaan dan menampilkan halaman awal
})();
</script>

<script id="mx-sound-addon">
/* =======================================================================
   TAMBAHAN SUARA & EFEK SUARA
   Tidak ada satu baris pun kode di atas yang diubah/dihapus.
   Modul ini hanya "mendengarkan" (MutationObserver + event listener
   tambahan) perubahan yang sudah terjadi di layar, lalu memutar suara.
   Semua suara dibuat langsung lewat Web Audio API (synth), jadi tidak
   perlu file audio eksternal / koneksi internet.
   ======================================================================= */
(function(){
  'use strict';

  /* ---------- Saklar suara nyala/mati (tersimpan di browser) ---------- */
  var SOUND_KEY = 'mxSoundEnabled';
  var soundEnabled = (function(){
    var v = null;
    try{ v = localStorage.getItem(SOUND_KEY); }catch(e){}
    return v === null ? true : v === '1';
  })();
  function setSoundEnabled(v){
    soundEnabled = v;
    try{ localStorage.setItem(SOUND_KEY, v ? '1' : '0'); }catch(e){}
  }

  /* ---------- Mesin suara (Web Audio API) ---------- */
  var actx = null;
  function ctx(){
    if(!actx){
      var AC = window.AudioContext || window.webkitAudioContext;
      if(!AC) return null;
      actx = new AC();
    }
    if(actx.state === 'suspended'){ actx.resume().catch(function(){}); }
    return actx;
  }
  // Buka AudioContext begitu ada interaksi pertama dari pengguna (wajib di browser modern)
  ['click','touchstart','keydown'].forEach(function(evt){
    document.addEventListener(evt, function unlock(){ ctx(); document.removeEventListener(evt, unlock); }, {once:true, passive:true});
  });

  function tone(freq, dur, opts){
    if(!soundEnabled) return;
    var c = ctx();
    if(!c) return;
    opts = opts || {};
    var t0 = c.currentTime + (opts.delay || 0);
    var osc = c.createOscillator();
    var gain = c.createGain();
    osc.type = opts.type || 'sine';
    osc.frequency.setValueAtTime(freq, t0);
    if(opts.freqEnd) osc.frequency.exponentialRampToValueAtTime(Math.max(20, opts.freqEnd), t0 + dur);
    var vol = opts.volume === undefined ? 0.5 : opts.volume;
    gain.gain.setValueAtTime(0.0001, t0);
    gain.gain.exponentialRampToValueAtTime(Math.max(0.0002, vol), t0 + 0.015);
    gain.gain.exponentialRampToValueAtTime(0.0001, t0 + dur);
    osc.connect(gain);
    gain.connect(c.destination);
    osc.start(t0);
    osc.stop(t0 + dur + 0.03);
  }

  /* --- Efek: detak hitung mundur (makin dekat 0 = makin besar & tinggi) --- */
  function sfxTick(urgent){
    tone(urgent ? 1250 : 820, urgent ? 0.16 : 0.09, { type:'square', volume: urgent ? 1.0 : 0.5 });
    if(urgent){ tone(1250, 0.16, { type:'square', volume:0.5, delay:0.02 }); } // lapis kedua biar makin nendang saat genting
  }
  /* --- Efek: waktu habis (alarm besar, keras) --- */
  function sfxTimeUp(){
    tone(880, 0.55, { type:'sawtooth', freqEnd:220, volume:1.0 });
    tone(440, 0.55, { type:'sawtooth', freqEnd:110, volume:0.9, delay:0.08 });
    tone(1760, 0.18, { type:'square', volume:0.8, delay:0.02 });
  }
  /* --- Efek: jawaban dibuka (reveal) --- */
  function sfxReveal(){
    tone(660, 0.12, { volume:0.65 });
    tone(880, 0.14, { volume:0.7, delay:0.11 });
    tone(1175, 0.22, { type:'triangle', volume:0.85, delay:0.23 });
  }
  /* --- Efek: klik tombol umum --- */
  function sfxClick(){
    tone(520, 0.055, { type:'triangle', volume:0.3 });
  }
  /* --- Efek: kuis selesai (fanfare) --- */
  function sfxFinish(){
    [523.25, 659.25, 783.99, 1046.5].forEach(function(f, i){
      tone(f, 0.28, { type:'triangle', volume:0.75, delay: i * 0.14 });
    });
  }

  /* ---------- Pasang pengamat setelah DOM siap ---------- */
  function whenReady(fn){
    if(document.readyState === 'loading'){ document.addEventListener('DOMContentLoaded', fn); }
    else { fn(); }
  }

  whenReady(function(){
    /* --- Tombol nyala/mati suara (dibuat lewat JS, tidak mengubah markup asli) --- */
    var soundBtn = document.createElement('button');
    soundBtn.type = 'button';
    soundBtn.id = 'mxSoundToggle';
    soundBtn.title = 'Nyalakan/matikan suara';
    soundBtn.setAttribute('aria-label', 'Nyalakan atau matikan suara');
    soundBtn.textContent = soundEnabled ? '\uD83D\uDD0A' : '\uD83D\uDD07';
    soundBtn.style.cssText = 'position:fixed;top:16px;left:86px;z-index:26;width:44px;height:44px;'
      + 'border-radius:50%;border:1px solid rgba(255,255,255,.14);background:rgba(7,11,20,.9);'
      + 'color:#fff;font-size:1.25rem;line-height:1;cursor:pointer;display:flex;align-items:center;'
      + 'justify-content:center;font-family:system-ui,sans-serif;';
    soundBtn.addEventListener('click', function(e){
      e.stopPropagation();
      var wasOn = soundEnabled;
      setSoundEnabled(!soundEnabled);
      soundBtn.textContent = soundEnabled ? '\uD83D\uDD0A' : '\uD83D\uDD07';
      if(!wasOn && soundEnabled){ ctx(); sfxClick(); } // beri bunyi konfirmasi hanya saat dinyalakan
    });
    document.body.appendChild(soundBtn);

    var timerBadge = document.getElementById('timerBadge');
    var banner = document.getElementById('resultBanner');
    var finishScreen = document.getElementById('finishScreen');

    /* --- Amati angka hitung mundur: bunyi tiap detik, makin keras saat genting, alarm besar saat 0 --- */
    if(timerBadge){
      var lastText = timerBadge.textContent;
      var doneHandled = timerBadge.classList.contains('done');
      var tbObserver = new MutationObserver(function(){
        var isDone = timerBadge.classList.contains('done');
        var isUrgent = timerBadge.classList.contains('urgent');
        var text = timerBadge.textContent;

        if(isDone){
          if(!doneHandled){ doneHandled = true; sfxTimeUp(); }
          lastText = text;
          return;
        }
        doneHandled = false;

        if(text !== lastText){
          lastText = text;
          sfxTick(isUrgent);
        }
      });
      tbObserver.observe(timerBadge, {
        childList:true, characterData:true, subtree:true,
        attributes:true, attributeFilter:['class']
      });
    }

    /* --- Amati saat jawaban benar/salah ditampilkan (banner "JAWABAN: ...") --- */
    if(banner){
      var bannerShown = banner.classList.contains('show');
      var bnObserver = new MutationObserver(function(){
        var shown = banner.classList.contains('show');
        if(shown && !bannerShown){ sfxReveal(); }
        bannerShown = shown;
      });
      bnObserver.observe(banner, { attributes:true, attributeFilter:['class'] });
    }

    /* --- Amati saat layar "selesai" muncul --- */
    if(finishScreen){
      var finishShown = finishScreen.style.display !== 'none' && finishScreen.style.display !== '';
      var fsObserver = new MutationObserver(function(){
        var shown = finishScreen.style.display === 'flex';
        if(shown && !finishShown){ sfxFinish(); }
        finishShown = shown;
      });
      fsObserver.observe(finishScreen, { attributes:true, attributeFilter:['style'] });
    }

    /* --- Bunyi klik ringan untuk semua tombol (tidak mengganti listener yang sudah ada, hanya menambah) --- */
    document.addEventListener('click', function(e){
      var el = e.target.closest('button, .btn, .mx-btn, .mx-seg-btn, .mx-chip, .mx-icon-btn');
      if(el && el.id !== 'mxSoundToggle'){ sfxClick(); }
    }, true);
  });
})();
</script>

<script id="mx-autofit-addon">
/* =======================================================================
   TAMBAHAN: penyesuaian otomatis ukuran teks soal & jawaban
   Tidak ada satu baris pun kode di atas yang diubah/dihapus.
   Modul ini hanya "mendengarkan" (MutationObserver) setiap kali teks
   soal/jawaban berganti, lalu mengecilkan ukuran hurufnya SEDIKIT DEMI
   SEDIKIT jika teksnya panjang, agar tidak ada bagian yang terpotong
   di luar layar. Soal/jawaban yang pendek tetap tampil besar seperti
   biasa.
   ======================================================================= */
(function(){
  'use strict';

  function whenReady(fn){
    if(document.readyState === 'loading'){ document.addEventListener('DOMContentLoaded', fn); }
    else { fn(); }
  }

  whenReady(function(){
    var topArea = document.getElementById('topArea');
    var qtextEl = document.getElementById('qtext');
    var ansAEl = document.getElementById('ansAText');
    var ansBEl = document.getElementById('ansBText');
    if(!topArea || !qtextEl || !ansAEl || !ansBEl) return;

    var MIN_SCALE = 0.5;     // batas paling kecil, supaya tetap terbaca
    var STEP = 0.04;         // besar penurunan tiap percobaan
    var MAX_PANEL_RATIO = 0.62; // panel soal+jawaban maksimal memakai porsi ini dari tinggi layar

    var rafId = null;
    function scheduleFit(){
      if(rafId) return;
      rafId = window.requestAnimationFrame(function(){ rafId = null; fit(); });
    }

    function fit(){
      /* kembalikan dulu ke ukuran alami (dari CSS), supaya soal/jawaban
         pendek balik ke besar normal setelah sebelumnya sempat mengecil */
      qtextEl.style.fontSize = '';
      ansAEl.style.fontSize = '';
      ansBEl.style.fontSize = '';

      var maxHeight = window.innerHeight * MAX_PANEL_RATIO;
      var qBase = parseFloat(getComputedStyle(qtextEl).fontSize) || 0;
      var aBase = parseFloat(getComputedStyle(ansAEl).fontSize) || 0;
      var bBase = parseFloat(getComputedStyle(ansBEl).fontSize) || 0;
      if(!qBase || !aBase || !bBase) return;

      var scale = 1;
      var guard = 0;
      while(topArea.getBoundingClientRect().height > maxHeight && scale > MIN_SCALE && guard < 20){
        scale -= STEP;
        qtextEl.style.fontSize = (qBase * scale) + 'px';
        ansAEl.style.fontSize = (aBase * scale) + 'px';
        ansBEl.style.fontSize = (bBase * scale) + 'px';
        guard++;
      }
    }

    var mo = new MutationObserver(scheduleFit);
    [qtextEl, ansAEl, ansBEl].forEach(function(el){
      mo.observe(el, { childList:true, characterData:true, subtree:true });
    });
    window.addEventListener('resize', scheduleFit);
    window.addEventListener('orientationchange', scheduleFit);

    scheduleFit();
  });
})();
</script>

</body>
</html>

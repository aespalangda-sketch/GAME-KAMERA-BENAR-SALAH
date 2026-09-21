[APLIKASI_GAME_KAMERA_BENAR-SALAH (2).html](https://github.com/user-attachments/files/32451456/APLIKASI_GAME_KAMERA_BENAR-SALAH.2.html)
<!DOCTYPE html>
<html lang="id" class="mx-locked">
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
<style id="mx-gate-style">
  /* ===== TAMBAHAN: gerbang password & status kamera ===== */
  html.mx-locked #stage{visibility:hidden;}
  #mxGate{
    position:fixed; inset:0; z-index:2147483647;
    display:flex; align-items:center; justify-content:center; padding:20px;
    background:var(--bg); color:var(--text);
    font-family:'Inter',system-ui,sans-serif; overflow:auto;
  }
  .mxg-wash{position:absolute; top:0; bottom:0; width:50%; pointer-events:none;}
  .mxg-wash.a{left:0; background:linear-gradient(90deg, rgba(59,130,246,.24), rgba(59,130,246,0) 90%);}
  .mxg-wash.b{right:0; background:linear-gradient(270deg, rgba(245,158,11,.22), rgba(245,158,11,0) 90%);}
  .mxg-mid{position:absolute; top:0; bottom:0; left:50%; width:2px; transform:translateX(-1px); pointer-events:none;
    background:linear-gradient(to bottom, transparent 0%, rgba(255,255,255,.55) 12%, rgba(255,255,255,.55) 88%, transparent 100%);}
  .mxg-card{
    position:relative; z-index:1; width:100%; max-width:440px; margin:auto;
    padding:36px 34px 28px; border-radius:24px; text-align:center;
    background:var(--panel); border:1px solid rgba(255,255,255,.1); backdrop-filter:blur(8px);
  }
  .mxg-card h1{font-family:'Space Grotesk',sans-serif; font-weight:700; font-size:clamp(1.8rem,5vw,2.4rem); line-height:1.1; letter-spacing:-.4px;}
  .mxg-sub{color:var(--muted); font-size:1.02rem; margin:10px 0 24px;}
  .mxg-field{display:flex; gap:8px; align-items:stretch;}
  #mxGatePw{
    flex:1; min-width:0; font:600 1.15rem 'Space Grotesk',sans-serif; letter-spacing:.06em;
    color:var(--text); background:rgba(255,255,255,.08); border:1px solid rgba(255,255,255,.18);
    border-radius:12px; padding:14px 16px;
  }
  #mxGatePw::placeholder{color:var(--muted); letter-spacing:0; font-weight:500;}
  #mxGatePw:focus-visible,#mxGateEye:focus-visible,#mxGateGo:focus-visible{outline:2px solid #fff; outline-offset:2px;}
  #mxGateEye{
    font:600 .9rem 'Space Grotesk',sans-serif; color:var(--text); cursor:pointer;
    background:rgba(255,255,255,.1); border:1px solid rgba(255,255,255,.14); border-radius:12px; padding:0 16px;
  }
  #mxGateEye:hover{background:rgba(255,255,255,.18);}
  #mxGateErr{min-height:1.5em; margin:12px 0 4px; color:#FCA5A5; font-size:.95rem; font-weight:500;}
  #mxGateGo{
    width:100%; margin-top:6px; font:600 1.1rem 'Space Grotesk',sans-serif; color:#fff; cursor:pointer;
    background:var(--a); border:none; border-radius:999px; padding:15px 24px;
  }
  #mxGateGo:hover{filter:brightness(1.08);}
  #mxGateGo:disabled{opacity:.55; cursor:default; filter:none;}
  .mxg-note{color:var(--muted); font-size:.88rem; margin-top:18px;}
  .mxg-card.shake{animation:mxgShake .38s;}
  @keyframes mxgShake{0%,100%{transform:translateX(0);}20%{transform:translateX(-9px);}40%{transform:translateX(8px);}60%{transform:translateX(-6px);}80%{transform:translateX(4px);}}
  @media (prefers-reduced-motion:reduce){.mxg-card.shake{animation:none;}}

  #mxCamStatus{margin:12px auto 0; max-width:560px; font-size:.95rem; line-height:1.45; color:var(--muted);}
  #mxCamStatus:empty{display:none;}
  #mxCamStatus.err{color:#FCA5A5;}
</style>
</head>
<body>

<!-- ===== TAMBAHAN: gerbang password. Aplikasi (#stage) tetap terkunci sampai password benar. ===== -->
<div id="mxGate" role="dialog" aria-modal="true" aria-labelledby="mxGateTitle">
  <div class="mxg-wash a"></div>
  <div class="mxg-wash b"></div>
  <div class="mxg-mid"></div>
  <form id="mxGateForm" class="mxg-card" autocomplete="off" novalidate>
    <h1 id="mxGateTitle">Kuis Interaktif</h1>
    <p class="mxg-sub">Masukkan password untuk membuka aplikasi.</p>
    <div class="mxg-field">
      <input id="mxGatePw" type="password" placeholder="Password" aria-label="Password"
             autocomplete="off" autocapitalize="off" autocorrect="off" spellcheck="false">
      <button type="button" id="mxGateEye" aria-pressed="false">Lihat</button>
    </div>
    <p id="mxGateErr" role="alert" aria-live="assertive"></p>
    <button type="submit" id="mxGateGo">Masuk</button>
    <p class="mxg-note">Setelah masuk, browser akan meminta izin kamera. Pilih Izinkan.</p>
  </form>
</div>

<script id="mx-gate-script">
(function(){
  'use strict';

  /* Password tidak disimpan sebagai teks biasa, hanya sidik jarinya (SHA-256 + garam). */
  var SALT = 'mx.kuis.v1|';
  var HASH = 'e48c470f53d5ee0fb6fa634a721407e0df1ececd1eaeb2297cc2ef16d4704fed';
  var MAX_TRIES = 5, LOCK_SECONDS = 30;

  /* SHA-256 murni JavaScript: tetap jalan di browser IFP lama dan di alamat http. */
  function sha256(str){
    var mp = Math.pow, mw = mp(2, 32), i, j, out = '';
    var words = [], hash = [], k = [], composite = {};
    var bits = str.length * 8;
    str = unescape(encodeURIComponent(str)); bits = str.length * 8;
    for(var cand = 2, n = 0; n < 64; cand++){
      if(!composite[cand]){
        for(i = 0; i < 313; i += cand) composite[i] = cand;
        hash[n] = (mp(cand, .5) * mw) | 0;
        k[n++] = (mp(cand, 1 / 3) * mw) | 0;
      }
    }
    str += '\x80';
    while(str.length % 64 - 56) str += '\x00';
    for(i = 0; i < str.length; i++){
      j = str.charCodeAt(i);
      words[i >> 2] |= j << ((3 - i) % 4) * 8;
    }
    words[words.length] = ((bits / mw) | 0);
    words[words.length] = (bits);
    for(j = 0; j < words.length;){
      var w = words.slice(j, j += 16), old = hash;
      hash = hash.slice(0, 8);
      for(i = 0; i < 64; i++){
        var w15 = w[i - 15], w2 = w[i - 2];
        var a = hash[0], e = hash[4];
        var t1 = hash[7] + (((e >>> 6) | (e << 26)) ^ ((e >>> 11) | (e << 21)) ^ ((e >>> 25) | (e << 7)))
               + ((e & hash[5]) ^ (~e & hash[6])) + k[i]
               + (w[i] = (i < 16) ? w[i] : (
                   w[i - 16]
                   + (((w15 >>> 7) | (w15 << 25)) ^ ((w15 >>> 18) | (w15 << 14)) ^ (w15 >>> 3))
                   + w[i - 7]
                   + (((w2 >>> 17) | (w2 << 15)) ^ ((w2 >>> 19) | (w2 << 13)) ^ (w2 >>> 10))
                 ) | 0);
        var t2 = (((a >>> 2) | (a << 30)) ^ ((a >>> 13) | (a << 19)) ^ ((a >>> 22) | (a << 10)))
               + ((a & hash[1]) ^ (a & hash[2]) ^ (hash[1] & hash[2]));
        hash = [(t1 + t2) | 0].concat(hash);
        hash[4] = (hash[4] + t1) | 0;
      }
      for(i = 0; i < 8; i++) hash[i] = (hash[i] + old[i]) | 0;
    }
    for(i = 0; i < 8; i++){
      for(j = 3; j + 1; j--){
        var b = (hash[i] >> (j * 8)) & 255;
        out += ((b < 16) ? 0 : '') + b.toString(16);
      }
    }
    return out;
  }

  var root = document.documentElement;
  var gate = document.getElementById('mxGate');
  var form = document.getElementById('mxGateForm');
  var pw = document.getElementById('mxGatePw');
  var eye = document.getElementById('mxGateEye');
  var err = document.getElementById('mxGateErr');
  var go = document.getElementById('mxGateGo');
  var fails = 0, lockTimer = null;

  eye.addEventListener('click', function(){
    var show = pw.type === 'password';
    pw.type = show ? 'text' : 'password';
    eye.textContent = show ? 'Sembunyikan' : 'Lihat';
    eye.setAttribute('aria-pressed', show ? 'true' : 'false');
    pw.focus();
  });

  function startLock(){
    var left = LOCK_SECONDS;
    go.disabled = true; pw.disabled = true;
    err.textContent = 'Terlalu banyak percobaan. Coba lagi dalam ' + left + ' detik.';
    lockTimer = setInterval(function(){
      left--;
      if(left <= 0){
        clearInterval(lockTimer); lockTimer = null;
        fails = 0; go.disabled = false; pw.disabled = false;
        err.textContent = ''; pw.focus();
      } else {
        err.textContent = 'Terlalu banyak percobaan. Coba lagi dalam ' + left + ' detik.';
      }
    }, 1000);
  }

  function unlock(){
    root.classList.remove('mx-locked');
    var stage = document.getElementById('stage');   // dicari saat dibuka: elemen ini baru ada setelah skrip ini dibaca
    if(stage) stage.removeAttribute('inert');
    if(gate && gate.parentNode) gate.parentNode.removeChild(gate);
    // Tutup keyboard layar (di IFP keyboard sering berupa overlay yang membuat kotak izin gagal muncul),
    // lalu minta izin kamera sesaat kemudian, saat layar sudah bersih.
    try{ if(document.activeElement && document.activeElement.blur) document.activeElement.blur(); }catch(_){}
    setTimeout(function(){
      if(typeof window.mxStartCamera === 'function') window.mxStartCamera();
    }, 700);
  }

  form.addEventListener('submit', function(e){
    e.preventDefault();
    if(lockTimer) return;
    var attempt = pw.value.trim();
    if(sha256(SALT + attempt) === HASH){
      unlock();
      return;
    }
    fails++;
    pw.value = '';
    if(fails >= MAX_TRIES){ startLock(); return; }
    err.textContent = 'Password salah. Periksa huruf besar/kecil lalu coba lagi.';
    form.classList.remove('shake'); void form.offsetWidth; form.classList.add('shake');
    pw.focus();
  });

  // Jangan biarkan tombol pintasan game menangkap ketikan password
  gate.addEventListener('keydown', function(e){ e.stopPropagation(); });

  try{ pw.focus(); }catch(_){}
})();
</script>

<div id="stage" inert>
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
const camMsg = camPlaceholder.querySelector('p');
const CAM_IDLE_MSG = camMsg.textContent;
let camStream = null;
let camBusy = false;

// Tampilkan pesan di layar kamera dan di halaman awal (kolom status di bawah tombol kamera)
function camNotify(text, isError){
  camMsg.textContent = text || CAM_IDLE_MSG;
  const el = document.getElementById('mxCamStatus');
  if(el){ el.textContent = text || ''; el.classList.toggle('err', !!isError); }
}
// Memberi tahu tombol "Aktifkan kamera" di halaman awal agar memperbarui tampilannya
function camChanged(){ video.dispatchEvent(new Event('camchange')); }
function camActive(){
  return !!(camStream && camStream.getTracks().some(t => t.readyState === 'live'));
}

function camErrorText(err){
  const n = err && err.name;
  if(n === 'NotAllowedError' || n === 'PermissionDeniedError' || n === 'SecurityError'){
    return 'Izin kamera belum diberikan. Jika muncul tulisan "Situs ini tidak dapat meminta izin Anda", ' +
           'tutup dulu semua menu melayang atau overlay aplikasi lain di layar (toolbar mengambang, perekam layar, dan sejenisnya), ' +
           'lalu tekan Aktifkan kamera lagi. Jika masih gagal, izinkan Kamera untuk Chrome di Pengaturan perangkat ' +
           '(Aplikasi > Chrome > Izin > Kamera), atau ketuk ikon gembok dekat alamat web lalu ubah Kamera menjadi Izinkan.';
  }
  if(n === 'NotFoundError' || n === 'DevicesNotFoundError' || n === 'OverconstrainedError'){
    return 'Kamera tidak ditemukan. Pastikan kamera terpasang dan tidak dinonaktifkan, lalu tekan Aktifkan kamera lagi.';
  }
  if(n === 'NotReadableError' || n === 'TrackStartError' || n === 'AbortError'){
    return 'Kamera sedang dipakai aplikasi lain. Tutup aplikasi atau tab yang memakai kamera, lalu tekan Aktifkan kamera lagi.';
  }
  return 'Kamera tidak dapat dibuka (' + (n || 'kesalahan tidak dikenal') + '). Tekan Aktifkan kamera untuk mencoba lagi.';
}

function onCamEnded(){
  camStream = null;
  video.srcObject = null;
  camPlaceholder.style.display = '';
  camNotify('Kamera terputus. Tekan Aktifkan kamera untuk menyambungkan lagi.', true);
  camChanged();
}

async function startCamera(){
  if(camBusy || camActive()) return;

  // Tanpa HTTPS browser tidak menyediakan kamera dan tidak akan menampilkan kotak izin
  if(!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia){
    camPlaceholder.style.display = '';
    camNotify(
      window.isSecureContext === false
        ? 'Kamera hanya bisa dipakai lewat alamat aman. Buka aplikasi memakai link yang diawali https://'
        : 'Browser ini belum mendukung kamera. Gunakan Chrome, Edge, atau Safari versi terbaru.',
      true
    );
    camChanged();
    return;
  }

  camBusy = true;
  camNotify('Meminta izin kamera. Pilih Izinkan pada kotak yang muncul.', false);
  try{
    let stream;
    try{
      stream = await navigator.mediaDevices.getUserMedia({
        video:{ facingMode:'user', width:{ideal:1280}, height:{ideal:720} },
        audio:false
      });
    }catch(first){
      const n = first && first.name;
      // Izin ditolak atau kamera dipakai: tidak perlu dicoba ulang. Selain itu, coba pengaturan paling sederhana.
      if(n === 'NotAllowedError' || n === 'PermissionDeniedError' || n === 'SecurityError' || n === 'NotReadableError') throw first;
      stream = await navigator.mediaDevices.getUserMedia({ video:true, audio:false });
    }
    camStream = stream;
    stream.getVideoTracks().forEach(t => t.addEventListener('ended', onCamEnded));
    video.srcObject = stream;
    camPlaceholder.style.display = 'none';
    camNotify('', false);
    try{ await video.play(); }catch(_){ /* autoplay sudah diatur lewat atribut video */ }
  }catch(err){
    camPlaceholder.style.display = '';
    camNotify(camErrorText(err), true);
  }finally{
    camBusy = false;
    camChanged();
  }
}

document.getElementById('startCamBtn').addEventListener('click', startCamera);
window.mxStartCamera = startCamera;

// Jika izin kamera diubah menjadi "Izinkan" lewat pengaturan browser, kamera langsung menyala
try{
  if(navigator.permissions && navigator.permissions.query){
    navigator.permissions.query({ name:'camera' }).then(p => {
      p.onchange = () => {
        if(p.state === 'granted' && !document.documentElement.classList.contains('mx-locked')) startCamera();
      };
    }).catch(() => {});
  }
}catch(_){}

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
        <p id="mxCamStatus" role="status" aria-live="polite"></p>
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
  ['playing', 'emptied', 'camchange'].forEach(function(ev){ video.addEventListener(ev, refreshCam); });

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

<!-- ===== TAMBAHAN BARU: menu pilih Jenjang, Kelas, Mapel & Tingkat kesulitan ===== -->
<style id="mx-preset-style">
  .mxp-card{margin-top:16px; padding:18px 20px; border-radius:16px; background:rgba(255,255,255,.05); border:1px solid rgba(255,255,255,.08); text-align:left;}
  .mxp-title{font-family:'Space Grotesk',sans-serif; font-weight:600; font-size:1.05rem;}
  .mxp-hint{color:var(--muted); font-size:.85rem; margin-top:4px;}
  .mxp-grid{display:grid; grid-template-columns:1fr 1fr; gap:10px; margin-top:14px;}
  .mxp-field label{display:block; font-size:.78rem; color:var(--muted); margin-bottom:4px;}
  .mxp-field select{width:100%; height:42px; border-radius:10px; border:1px solid rgba(255,255,255,.18); background:#0D1424; color:var(--text); padding:0 10px; font-family:'Inter',sans-serif; font-size:.92rem;}
  .mxp-actions{display:flex; flex-wrap:wrap; gap:10px; margin-top:14px;}
  .mxp-btn{font-family:'Space Grotesk',sans-serif; font-weight:600; font-size:.92rem; border:none; padding:12px 20px; border-radius:999px; cursor:pointer; background:var(--a); color:#fff;}
  .mxp-btn:hover{filter:brightness(1.08);}
  .mxp-btn.secondary{background:rgba(255,255,255,.1); color:var(--text); border:1px solid rgba(255,255,255,.16);}
  .mxp-status{font-size:.82rem; color:var(--muted); margin-top:10px;}
  @media (max-width:520px){ .mxp-grid{grid-template-columns:1fr;} }
</style>
<script id="mx-preset-script">
(function(){
  'use strict';

  // ---- Bank soal siap pakai (baru sebagian mapel, sisanya menyusul) ----
  // format tiap soal: {q, correct, wrong, side:'acak'}
  var PRESET_BANK = {
    matematika: {
      mudah: [
        {q:'5 + 3 = 8', correct:'Benar', wrong:'Salah'},
        {q:'10 - 4 = 5', correct:'Salah', wrong:'Benar'},
        {q:'2 x 6 = 12', correct:'Benar', wrong:'Salah'},
        {q:'9 dibagi 3 sama dengan 3', correct:'Benar', wrong:'Salah'},
        {q:'7 lebih besar dari 10', correct:'Salah', wrong:'Benar'},
        {q:'Bilangan genap terkecil adalah 0', correct:'Benar', wrong:'Salah'},
        {q:'4 x 4 = 16', correct:'Benar', wrong:'Salah'},
        {q:'15 - 7 = 9', correct:'Salah', wrong:'Benar'},
        {q:'Segitiga memiliki 3 sisi', correct:'Benar', wrong:'Salah'},
        {q:'Persegi memiliki 5 sisi', correct:'Salah', wrong:'Benar'},
        {q:'100 dibagi 10 sama dengan 10', correct:'Benar', wrong:'Salah'},
        {q:'6 + 6 = 11', correct:'Salah', wrong:'Benar'},
        {q:'Bilangan ganjil setelah 5 adalah 6', correct:'Salah', wrong:'Benar'},
        {q:'3 x 3 = 9', correct:'Benar', wrong:'Salah'},
        {q:'Lingkaran tidak memiliki sudut', correct:'Benar', wrong:'Salah'},
        {q:'20 - 5 = 15', correct:'Benar', wrong:'Salah'},
        {q:'8 + 9 = 17', correct:'Benar', wrong:'Salah'},
        {q:'Setengah dari 10 adalah 4', correct:'Salah', wrong:'Benar'},
        {q:'Segi empat memiliki 4 sudut', correct:'Benar', wrong:'Salah'},
        {q:'7 x 2 = 12', correct:'Salah', wrong:'Benar'},
        {q:'Bilangan 1 adalah bilangan genap', correct:'Salah', wrong:'Benar'},
        {q:'50 + 50 = 100', correct:'Benar', wrong:'Salah'},
        {q:'Satu jam terdiri dari 60 menit', correct:'Benar', wrong:'Salah'},
        {q:'Satu minggu terdiri dari 8 hari', correct:'Salah', wrong:'Benar'},
        {q:'12 dibagi 4 sama dengan 3', correct:'Benar', wrong:'Salah'},
      ],
      sedang: [
        {q:'Luas persegi panjang = panjang x lebar', correct:'Benar', wrong:'Salah'},
        {q:'1/2 sama nilainya dengan 0,5', correct:'Benar', wrong:'Salah'},
        {q:'Sudut siku-siku besarnya 60 derajat', correct:'Salah', wrong:'Benar'},
        {q:'Bilangan prima terkecil adalah 2', correct:'Benar', wrong:'Salah'},
        {q:'Keliling lingkaran dihitung dengan phi x diameter', correct:'Benar', wrong:'Salah'},
        {q:'20% dari 200 adalah 40', correct:'Benar', wrong:'Salah'},
        {q:'Luas segitiga = 1/2 x alas x tinggi', correct:'Benar', wrong:'Salah'},
        {q:'Bilangan negatif lebih besar dari bilangan positif', correct:'Salah', wrong:'Benar'},
        {q:'FPB dari 12 dan 18 adalah 6', correct:'Benar', wrong:'Salah'},
        {q:'KPK dari 4 dan 6 adalah 12', correct:'Benar', wrong:'Salah'},
        {q:'Sudut dalam segitiga sama sisi masing-masing 60 derajat', correct:'Benar', wrong:'Salah'},
        {q:'Pecahan 3/4 lebih besar dari 1/2', correct:'Benar', wrong:'Salah'},
        {q:'Bilangan bulat mencakup bilangan negatif, nol, dan positif', correct:'Benar', wrong:'Salah'},
        {q:'Rata-rata dari 2, 4, 6 adalah 5', correct:'Salah', wrong:'Benar'},
        {q:'Persegi adalah bangun datar dengan 4 sisi sama panjang', correct:'Benar', wrong:'Salah'},
        {q:'Volume kubus dihitung dengan sisi x sisi x sisi', correct:'Benar', wrong:'Salah'},
        {q:'3 kuadrat sama dengan 6', correct:'Salah', wrong:'Benar'},
        {q:'Garis sejajar tidak akan pernah berpotongan', correct:'Benar', wrong:'Salah'},
        {q:'Median adalah nilai tengah dari data yang sudah diurutkan', correct:'Benar', wrong:'Salah'},
        {q:'Modus adalah data yang paling jarang muncul', correct:'Salah', wrong:'Benar'},
        {q:'Skala peta 1:1000 berarti 1 cm mewakili 1000 cm sebenarnya', correct:'Benar', wrong:'Salah'},
        {q:'Perbandingan senilai terjadi jika kedua nilai naik atau turun bersamaan', correct:'Benar', wrong:'Salah'},
        {q:'Pecahan campuran terdiri dari bilangan bulat dan pecahan', correct:'Benar', wrong:'Salah'},
        {q:'Sudut lancip besarnya lebih dari 90 derajat', correct:'Salah', wrong:'Benar'},
        {q:'Diagonal persegi panjang memiliki panjang yang sama', correct:'Benar', wrong:'Salah'},
      ],
      sulit: [
        {q:'Akar dari 144 adalah 12', correct:'Benar', wrong:'Salah'},
        {q:'Persamaan kuadrat selalu punya dua akar real', correct:'Salah', wrong:'Benar'},
        {q:'Hasil dari 2 pangkat 5 adalah 32', correct:'Benar', wrong:'Salah'},
        {q:'Turunan dari x kuadrat adalah 2x', correct:'Benar', wrong:'Salah'},
        {q:'Logaritma dari 100 basis 10 adalah 2', correct:'Benar', wrong:'Salah'},
        {q:'Jumlah sudut dalam segitiga selalu 180 derajat', correct:'Benar', wrong:'Salah'},
        {q:'Fungsi kuadrat grafiknya berbentuk parabola', correct:'Benar', wrong:'Salah'},
        {q:'Matriks identitas memiliki determinan 0', correct:'Salah', wrong:'Benar'},
        {q:'Integral adalah kebalikan dari turunan', correct:'Benar', wrong:'Salah'},
        {q:'Limit suatu fungsi selalu ada di setiap titik', correct:'Salah', wrong:'Benar'},
        {q:'Sin 90 derajat sama dengan 1', correct:'Benar', wrong:'Salah'},
        {q:'Cos 0 derajat sama dengan 0', correct:'Salah', wrong:'Benar'},
        {q:'Deret aritmatika memiliki beda yang tetap', correct:'Benar', wrong:'Salah'},
        {q:'Deret geometri memiliki rasio yang tetap', correct:'Benar', wrong:'Salah'},
        {q:'Vektor memiliki besar dan arah', correct:'Benar', wrong:'Salah'},
        {q:'Determinan matriks 2x2 dihitung dengan ad dikurangi bc', correct:'Benar', wrong:'Salah'},
        {q:'Peluang suatu kejadian selalu bernilai antara 0 dan 1', correct:'Benar', wrong:'Salah'},
        {q:'Bilangan kompleks tidak memiliki bagian imajiner', correct:'Salah', wrong:'Benar'},
        {q:'Grafik fungsi linear berbentuk garis lurus', correct:'Benar', wrong:'Salah'},
        {q:'Persamaan lingkaran memiliki bentuk umum x kuadrat + y kuadrat = r kuadrat', correct:'Benar', wrong:'Salah'},
        {q:'Notasi sigma digunakan untuk menyatakan penjumlahan berurutan', correct:'Benar', wrong:'Salah'},
        {q:'Nilai e (bilangan Euler) kira-kira sama dengan 2,718', correct:'Benar', wrong:'Salah'},
        {q:'Fungsi eksponen selalu menurun', correct:'Salah', wrong:'Benar'},
        {q:'Barisan Fibonacci dimulai dari 0 dan 1', correct:'Benar', wrong:'Salah'},
        {q:'Trigonometri hanya berlaku untuk segitiga siku-siku saja', correct:'Salah', wrong:'Benar'},
      ]
    },
    bindo: {
      mudah: [
        {q:'Kalimat tanya diakhiri tanda tanya (?)', correct:'Benar', wrong:'Salah'},
        {q:'Huruf kapital dipakai di awal kalimat', correct:'Benar', wrong:'Salah'},
        {q:'Sinonim artinya lawan kata', correct:'Salah', wrong:'Benar'},
        {q:'Pantun memiliki sampiran dan isi', correct:'Benar', wrong:'Salah'},
        {q:'Kata kerja disebut juga verba', correct:'Benar', wrong:'Salah'},
        {q:'Cerpen adalah cerita yang sangat panjang', correct:'Salah', wrong:'Benar'},
        {q:'Kalimat perintah diakhiri tanda seru', correct:'Benar', wrong:'Salah'},
        {q:'Huruf vokal dalam bahasa Indonesia ada 5', correct:'Benar', wrong:'Salah'},
        {q:'Dongeng termasuk karya sastra fiksi', correct:'Benar', wrong:'Salah'},
        {q:'Kata benda disebut juga nomina', correct:'Benar', wrong:'Salah'},
        {q:'Paragraf terdiri dari beberapa kalimat', correct:'Benar', wrong:'Salah'},
        {q:'Puisi selalu ditulis dalam bentuk prosa', correct:'Salah', wrong:'Benar'},
        {q:'Huruf konsonan adalah huruf selain huruf vokal', correct:'Benar', wrong:'Salah'},
        {q:'Kata sambung digunakan untuk menghubungkan kalimat', correct:'Benar', wrong:'Salah'},
        {q:'Membaca nyaring dilakukan dengan suara yang sangat pelan', correct:'Salah', wrong:'Benar'},
        {q:'Tanda titik digunakan di akhir kalimat berita', correct:'Benar', wrong:'Salah'},
        {q:'Fabel adalah cerita yang tokohnya hewan', correct:'Benar', wrong:'Salah'},
        {q:'Kamus digunakan untuk mencari arti kata', correct:'Benar', wrong:'Salah'},
        {q:'Kalimat harus memiliki subjek dan predikat', correct:'Benar', wrong:'Salah'},
        {q:'Legenda adalah cerita yang sepenuhnya berdasarkan kejadian nyata', correct:'Salah', wrong:'Benar'},
        {q:'Huruf abjad dalam bahasa Indonesia berjumlah 26', correct:'Benar', wrong:'Salah'},
        {q:'Tanda koma digunakan untuk memisahkan unsur dalam kalimat', correct:'Benar', wrong:'Salah'},
        {q:'Narasi adalah karangan yang menceritakan suatu peristiwa', correct:'Benar', wrong:'Salah'},
        {q:'Kata ulang adalah kata yang diulang, contohnya rumah-rumah', correct:'Benar', wrong:'Salah'},
        {q:'Awalan dan akhiran disebut juga imbuhan', correct:'Benar', wrong:'Salah'},
      ],
      sedang: [
        {q:'Antonim artinya lawan kata', correct:'Benar', wrong:'Salah'},
        {q:'Teks eksposisi bertujuan menghibur pembaca', correct:'Salah', wrong:'Benar'},
        {q:'Kalimat majemuk terdiri dari dua klausa atau lebih', correct:'Benar', wrong:'Salah'},
        {q:'Ide pokok biasanya ada di kalimat utama paragraf', correct:'Benar', wrong:'Salah'},
        {q:'Majas personifikasi membandingkan benda mati seolah hidup', correct:'Benar', wrong:'Salah'},
        {q:'Teks prosedur berisi langkah-langkah melakukan sesuatu', correct:'Benar', wrong:'Salah'},
        {q:'Teks deskripsi menggambarkan sesuatu secara rinci', correct:'Benar', wrong:'Salah'},
        {q:'Kalimat efektif harus ditulis bertele-tele agar jelas', correct:'Salah', wrong:'Benar'},
        {q:'Majas hiperbola menggunakan pernyataan yang berlebihan', correct:'Benar', wrong:'Salah'},
        {q:'Teks laporan hasil observasi berisi hasil pengamatan', correct:'Benar', wrong:'Salah'},
        {q:'Kata baku sesuai dengan kaidah bahasa yang berlaku', correct:'Benar', wrong:'Salah'},
        {q:'Paragraf induktif meletakkan ide pokok di awal paragraf', correct:'Salah', wrong:'Benar'},
        {q:'Kata tidak baku boleh dipakai dalam tulisan resmi', correct:'Salah', wrong:'Benar'},
        {q:'Surat resmi menggunakan bahasa baku', correct:'Benar', wrong:'Salah'},
        {q:'Wawancara adalah tanya jawab untuk memperoleh informasi', correct:'Benar', wrong:'Salah'},
        {q:'Teks negosiasi bertujuan mencapai kesepakatan', correct:'Benar', wrong:'Salah'},
        {q:'Majas metafora adalah perbandingan langsung tanpa kata pembanding', correct:'Benar', wrong:'Salah'},
        {q:'Unsur intrinsik cerita meliputi tema, tokoh, dan alur', correct:'Benar', wrong:'Salah'},
        {q:'Alur cerita hanya bisa maju, tidak bisa mundur', correct:'Salah', wrong:'Benar'},
        {q:'Latar cerita meliputi tempat, waktu, dan suasana', correct:'Benar', wrong:'Salah'},
        {q:'Teks berita harus memuat unsur 5W+1H', correct:'Benar', wrong:'Salah'},
        {q:'Iklan bertujuan mempersuasi atau membujuk pembaca', correct:'Benar', wrong:'Salah'},
        {q:'Kalimat langsung ditulis menggunakan tanda petik', correct:'Benar', wrong:'Salah'},
        {q:'Resensi buku sama dengan ringkasan buku tanpa penilaian', correct:'Salah', wrong:'Benar'},
        {q:'Teks eksplanasi menjelaskan proses terjadinya suatu fenomena', correct:'Benar', wrong:'Salah'},
      ],
      sulit: [
        {q:'Kalimat efektif boleh bertele-tele asalkan panjang', correct:'Salah', wrong:'Benar'},
        {q:'Konjungsi berfungsi menghubungkan kata atau kalimat', correct:'Benar', wrong:'Salah'},
        {q:'Teks argumentasi berisi pendapat disertai alasan', correct:'Benar', wrong:'Salah'},
        {q:'Kata baku selalu sesuai dengan KBBI', correct:'Benar', wrong:'Salah'},
        {q:'Paragraf deduktif meletakkan ide pokok di akhir paragraf', correct:'Salah', wrong:'Benar'},
        {q:'Resensi adalah ulasan atau penilaian terhadap suatu karya', correct:'Benar', wrong:'Salah'},
        {q:'Karya ilmiah harus didukung data dan fakta', correct:'Benar', wrong:'Salah'},
        {q:'Majas ironi menyatakan sesuatu yang bertentangan dengan maksud sebenarnya', correct:'Benar', wrong:'Salah'},
        {q:'Kalimat ambigu memiliki makna yang jelas dan tunggal', correct:'Salah', wrong:'Benar'},
        {q:'Diksi adalah pilihan kata dalam sebuah karya tulis', correct:'Benar', wrong:'Salah'},
        {q:'Sudut pandang orang pertama menggunakan kata ganti aku atau saya', correct:'Benar', wrong:'Salah'},
        {q:'Teks anekdot bertujuan mengkritik secara halus melalui humor', correct:'Benar', wrong:'Salah'},
        {q:'Kohesi adalah keterkaitan bentuk antarkalimat dalam paragraf', correct:'Benar', wrong:'Salah'},
        {q:'Koherensi tidak berkaitan dengan keterpaduan makna dalam paragraf', correct:'Salah', wrong:'Benar'},
        {q:'Esai adalah karangan yang mengungkapkan pendapat pribadi penulis', correct:'Benar', wrong:'Salah'},
        {q:'Gaya bahasa dan majas adalah dua istilah yang sama sekali tidak berkaitan', correct:'Salah', wrong:'Benar'},
        {q:'Karya sastra periode 1920-an disebut angkatan Balai Pustaka', correct:'Benar', wrong:'Salah'},
        {q:'Puisi kontemporer selalu terikat pada rima dan bait', correct:'Salah', wrong:'Benar'},
        {q:'Analisis wacana mengkaji bahasa dalam konteks penggunaannya', correct:'Benar', wrong:'Salah'},
        {q:'Teks ulasan film termasuk jenis teks yang objektif dan evaluatif', correct:'Benar', wrong:'Salah'},
        {q:'Frasa adalah gabungan kata yang tidak membentuk klausa', correct:'Benar', wrong:'Salah'},
        {q:'Klausa selalu memiliki subjek dan predikat', correct:'Benar', wrong:'Salah'},
        {q:'Denotasi adalah makna kias dari suatu kata', correct:'Salah', wrong:'Benar'},
        {q:'Konotasi adalah makna kias atau tidak sebenarnya dari suatu kata', correct:'Benar', wrong:'Salah'},
        {q:'Sinestesia adalah majas yang menggabungkan dua indera berbeda', correct:'Benar', wrong:'Salah'},
      ]
    },
    pkn: {
      mudah: [
        {q:'Pancasila memiliki 5 sila', correct:'Benar', wrong:'Salah'},
        {q:'Bendera Indonesia berwarna merah putih', correct:'Benar', wrong:'Salah'},
        {q:'Bahasa persatuan Indonesia adalah bahasa Inggris', correct:'Salah', wrong:'Benar'},
        {q:'Presiden adalah kepala negara Indonesia', correct:'Benar', wrong:'Salah'},
        {q:'Gotong royong mencerminkan sikap kebersamaan bangsa Indonesia', correct:'Benar', wrong:'Salah'},
        {q:'Indonesia merdeka pada tanggal 17 Agustus 1945', correct:'Benar', wrong:'Salah'},
        {q:'Lambang negara Indonesia adalah Garuda Pancasila', correct:'Benar', wrong:'Salah'},
        {q:'Lagu kebangsaan Indonesia adalah Indonesia Raya', correct:'Benar', wrong:'Salah'},
        {q:'Semboyan negara Indonesia adalah Bhinneka Tunggal Ika', correct:'Benar', wrong:'Salah'},
        {q:'Indonesia memiliki satu suku bangsa saja', correct:'Salah', wrong:'Benar'},
        {q:'Sila pertama Pancasila berbunyi Ketuhanan Yang Maha Esa', correct:'Benar', wrong:'Salah'},
        {q:'Musyawarah dilakukan untuk mencapai mufakat', correct:'Benar', wrong:'Salah'},
        {q:'Hak asasi manusia harus dihormati oleh semua orang', correct:'Benar', wrong:'Salah'},
        {q:'Kewajiban adalah sesuatu yang harus dilaksanakan', correct:'Benar', wrong:'Salah'},
        {q:'Setiap warga negara wajib menaati hukum', correct:'Benar', wrong:'Salah'},
        {q:'Pemilu digunakan untuk memilih presiden dan wakil rakyat', correct:'Benar', wrong:'Salah'},
        {q:'Toleransi berarti menghormati perbedaan', correct:'Benar', wrong:'Salah'},
        {q:'Indonesia adalah negara kesatuan', correct:'Benar', wrong:'Salah'},
        {q:'Setiap orang boleh melanggar aturan jika tidak ketahuan', correct:'Salah', wrong:'Benar'},
        {q:'Kerja sama membuat pekerjaan menjadi lebih ringan', correct:'Benar', wrong:'Salah'},
        {q:'Rumah adat adalah salah satu contoh keberagaman budaya', correct:'Benar', wrong:'Salah'},
        {q:'Upacara bendera dilakukan untuk menghormati jasa pahlawan', correct:'Benar', wrong:'Salah'},
        {q:'Anak-anak tidak memiliki hak untuk berpendapat', correct:'Salah', wrong:'Benar'},
        {q:'Menghormati orang tua adalah salah satu sikap terpuji', correct:'Benar', wrong:'Salah'},
        {q:'Norma adalah aturan yang mengatur kehidupan bermasyarakat', correct:'Benar', wrong:'Salah'},
      ],
      sedang: [
        {q:'UUD 1945 adalah dasar hukum tertinggi di Indonesia', correct:'Benar', wrong:'Salah'},
        {q:'Indonesia menganut sistem pemerintahan monarki', correct:'Salah', wrong:'Benar'},
        {q:'DPR bertugas membuat undang-undang bersama pemerintah', correct:'Benar', wrong:'Salah'},
        {q:'Setiap warga negara punya hak dan kewajiban yang sama di mata hukum', correct:'Benar', wrong:'Salah'},
        {q:'Bhinneka Tunggal Ika berarti berbeda-beda tetap satu jua', correct:'Benar', wrong:'Salah'},
        {q:'Pemilu di Indonesia diadakan setiap 10 tahun sekali', correct:'Salah', wrong:'Benar'},
        {q:'Demokrasi Pancasila mengutamakan musyawarah untuk mufakat', correct:'Benar', wrong:'Salah'},
        {q:'Lembaga eksekutif bertugas menjalankan pemerintahan', correct:'Benar', wrong:'Salah'},
        {q:'Lembaga legislatif bertugas membuat undang-undang', correct:'Benar', wrong:'Salah'},
        {q:'Otonomi daerah memberikan kewenangan kepada daerah mengatur wilayahnya', correct:'Benar', wrong:'Salah'},
        {q:'Setiap warga negara wajib membayar pajak sesuai ketentuan', correct:'Benar', wrong:'Salah'},
        {q:'Kedaulatan rakyat berarti kekuasaan tertinggi berada di tangan rakyat', correct:'Benar', wrong:'Salah'},
        {q:'Partai politik tidak berperan dalam proses demokrasi', correct:'Salah', wrong:'Benar'},
        {q:'Wawasan Nusantara memandang Indonesia sebagai satu kesatuan wilayah', correct:'Benar', wrong:'Salah'},
        {q:'Globalisasi tidak berpengaruh terhadap kehidupan berbangsa', correct:'Salah', wrong:'Benar'},
        {q:'Hak politik warga negara antara lain hak memilih dan dipilih', correct:'Benar', wrong:'Salah'},
        {q:'Konstitusi mengatur struktur dan mekanisme ketatanegaraan', correct:'Benar', wrong:'Salah'},
        {q:'Setiap keputusan dalam musyawarah harus dipaksakan oleh satu pihak', correct:'Salah', wrong:'Benar'},
        {q:'Integrasi nasional penting untuk menjaga persatuan bangsa', correct:'Benar', wrong:'Salah'},
        {q:'Sistem multipartai berarti hanya ada satu partai politik', correct:'Salah', wrong:'Benar'},
        {q:'Warga negara asing memiliki hak yang sama persis dengan warga negara Indonesia', correct:'Salah', wrong:'Benar'},
        {q:'Nilai-nilai Pancasila harus diterapkan dalam kehidupan sehari-hari', correct:'Benar', wrong:'Salah'},
        {q:'Pemerintah pusat dan daerah harus bekerja sama membangun negara', correct:'Benar', wrong:'Salah'},
        {q:'Hukum berlaku sama untuk semua warga negara tanpa kecuali', correct:'Benar', wrong:'Salah'},
        {q:'Perlindungan hak asasi manusia menjadi tanggung jawab negara', correct:'Benar', wrong:'Salah'},
      ],
      sulit: [
        {q:'MPR berwenang mengubah dan menetapkan UUD', correct:'Benar', wrong:'Salah'},
        {q:'Indonesia menganut sistem pemerintahan presidensial', correct:'Benar', wrong:'Salah'},
        {q:'Otonomi daerah berarti daerah tidak boleh mengatur urusannya sendiri', correct:'Salah', wrong:'Benar'},
        {q:'Lembaga yudikatif bertugas mengadili pelanggaran hukum', correct:'Benar', wrong:'Salah'},
        {q:'Hak asasi manusia bisa dilanggar dengan alasan apa pun', correct:'Salah', wrong:'Benar'},
        {q:'Checks and balances menjaga keseimbangan antar lembaga negara', correct:'Benar', wrong:'Salah'},
        {q:'Trias politica membagi kekuasaan menjadi eksekutif, legislatif, dan yudikatif', correct:'Benar', wrong:'Salah'},
        {q:'Konstitusi hanya boleh diubah oleh presiden seorang diri', correct:'Salah', wrong:'Benar'},
        {q:'Mahkamah Konstitusi berwenang menguji undang-undang terhadap UUD', correct:'Benar', wrong:'Salah'},
        {q:'Politik luar negeri Indonesia bersifat bebas aktif', correct:'Benar', wrong:'Salah'},
        {q:'Desentralisasi adalah penyerahan wewenang dari pusat ke daerah', correct:'Benar', wrong:'Salah'},
        {q:'Warga negara tidak memiliki kewajiban terhadap pertahanan negara', correct:'Salah', wrong:'Benar'},
        {q:'Sistem hukum Indonesia menganut asas praduga tak bersalah', correct:'Benar', wrong:'Salah'},
        {q:'Amandemen UUD 1945 telah dilakukan sebanyak empat kali', correct:'Benar', wrong:'Salah'},
        {q:'Ideologi terbuka dapat berkembang mengikuti zaman tanpa mengubah nilai dasarnya', correct:'Benar', wrong:'Salah'},
        {q:'Negara hukum berarti kekuasaan berada di atas hukum', correct:'Salah', wrong:'Benar'},
        {q:'Hak asasi manusia bersifat universal dan berlaku untuk semua orang', correct:'Benar', wrong:'Salah'},
        {q:'Good governance menekankan transparansi dan akuntabilitas pemerintahan', correct:'Benar', wrong:'Salah'},
        {q:'Politik identitas selalu memperkuat persatuan bangsa', correct:'Salah', wrong:'Benar'},
        {q:'Konvensi ketatanegaraan adalah kebiasaan dalam praktik bernegara yang tidak tertulis', correct:'Benar', wrong:'Salah'},
        {q:'Sistem pemilu proporsional memperhitungkan perolehan suara partai secara proporsional', correct:'Benar', wrong:'Salah'},
        {q:'Kedaulatan negara Indonesia bersifat mutlak tanpa batas hukum internasional', correct:'Salah', wrong:'Benar'},
        {q:'Bela negara hanya menjadi tugas anggota TNI', correct:'Salah', wrong:'Benar'},
        {q:'Reformasi 1998 mendorong perubahan menuju sistem yang lebih demokratis', correct:'Benar', wrong:'Salah'},
        {q:'Wawasan kebangsaan penting untuk menjaga keutuhan NKRI', correct:'Benar', wrong:'Salah'},
      ]
    },
    ipa: {
      mudah: [
        {q:'Matahari terbit dari arah timur', correct:'Benar', wrong:'Salah'},
        {q:'Tumbuhan membutuhkan air untuk hidup', correct:'Benar', wrong:'Salah'},
        {q:'Ikan bernapas menggunakan paru-paru', correct:'Salah', wrong:'Benar'},
        {q:'Air membeku pada suhu 0 derajat Celsius', correct:'Benar', wrong:'Salah'},
        {q:'Manusia memiliki 2 mata', correct:'Benar', wrong:'Salah'},
        {q:'Bumi mengelilingi matahari', correct:'Benar', wrong:'Salah'},
        {q:'Tumbuhan hijau membuat makanan sendiri melalui fotosintesis', correct:'Benar', wrong:'Salah'},
        {q:'Batu termasuk makhluk hidup', correct:'Salah', wrong:'Benar'},
        {q:'Kucing berkembang biak dengan bertelur', correct:'Salah', wrong:'Benar'},
        {q:'Manusia bernapas menggunakan paru-paru', correct:'Benar', wrong:'Salah'},
        {q:'Air, tanah, dan udara termasuk sumber daya alam', correct:'Benar', wrong:'Salah'},
        {q:'Besi jika dipanaskan akan memuai', correct:'Benar', wrong:'Salah'},
        {q:'Bulan bercahaya sendiri seperti matahari', correct:'Salah', wrong:'Benar'},
        {q:'Hewan yang makan tumbuhan disebut herbivora', correct:'Benar', wrong:'Salah'},
        {q:'Manusia memiliki rangka yang menyusun tubuhnya', correct:'Benar', wrong:'Salah'},
        {q:'Air laut rasanya tawar', correct:'Salah', wrong:'Benar'},
        {q:'Tumbuhan bernapas melalui stomata pada daun', correct:'Benar', wrong:'Salah'},
        {q:'Daur hidup kupu-kupu dimulai dari telur', correct:'Benar', wrong:'Salah'},
        {q:'Besi termasuk benda yang dapat berkarat', correct:'Benar', wrong:'Salah'},
        {q:'Benda padat dapat berubah menjadi cair jika dipanaskan', correct:'Benar', wrong:'Salah'},
        {q:'Magnet dapat menarik benda dari logam tertentu', correct:'Benar', wrong:'Salah'},
        {q:'Bumi berbentuk kotak', correct:'Salah', wrong:'Benar'},
        {q:'Hewan karnivora memakan daging', correct:'Benar', wrong:'Salah'},
        {q:'Tanaman memerlukan cahaya matahari untuk fotosintesis', correct:'Benar', wrong:'Salah'},
        {q:'Udara termasuk benda gas', correct:'Benar', wrong:'Salah'},
      ],
      sedang: [
        {q:'Fotosintesis terjadi pada bagian daun', correct:'Benar', wrong:'Salah'},
        {q:'Darah manusia dipompa oleh hati', correct:'Salah', wrong:'Benar'},
        {q:'Gaya gravitasi menarik benda ke arah bumi', correct:'Benar', wrong:'Salah'},
        {q:'Rantai makanan menggambarkan aliran energi antar makhluk hidup', correct:'Benar', wrong:'Salah'},
        {q:'Logam adalah penghantar panas yang buruk', correct:'Salah', wrong:'Benar'},
        {q:'Oksigen dibutuhkan manusia untuk bernapas', correct:'Benar', wrong:'Salah'},
        {q:'Ekosistem terdiri dari komponen biotik dan abiotik', correct:'Benar', wrong:'Salah'},
        {q:'Fungsi jantung memompa darah ke seluruh tubuh', correct:'Benar', wrong:'Salah'},
        {q:'Karbon dioksida dihasilkan saat manusia bernapas', correct:'Benar', wrong:'Salah'},
        {q:'Tumbuhan tidak memerlukan air untuk fotosintesis', correct:'Salah', wrong:'Benar'},
        {q:'Gerhana matahari terjadi ketika bulan berada di antara bumi dan matahari', correct:'Benar', wrong:'Salah'},
        {q:'Perubahan wujud dari cair ke gas disebut menguap', correct:'Benar', wrong:'Salah'},
        {q:'Simbiosis mutualisme menguntungkan kedua belah pihak', correct:'Benar', wrong:'Salah'},
        {q:'Zat tunggal terdiri dari unsur dan senyawa', correct:'Benar', wrong:'Salah'},
        {q:'Campuran homogen memiliki komponen yang tidak dapat dibedakan lagi', correct:'Benar', wrong:'Salah'},
        {q:'Gaya gesek selalu mempercepat gerak benda', correct:'Salah', wrong:'Benar'},
        {q:'Sistem pencernaan manusia dimulai dari mulut', correct:'Benar', wrong:'Salah'},
        {q:'Paru-paru adalah organ pernapasan utama manusia', correct:'Benar', wrong:'Salah'},
        {q:'Tekanan udara di dataran tinggi lebih besar daripada di dataran rendah', correct:'Salah', wrong:'Benar'},
        {q:'Pelapukan batuan dapat disebabkan oleh air dan suhu', correct:'Benar', wrong:'Salah'},
        {q:'Energi tidak dapat diciptakan atau dimusnahkan, hanya berubah bentuk', correct:'Benar', wrong:'Salah'},
        {q:'Listrik statis dapat timbul akibat gesekan dua benda', correct:'Benar', wrong:'Salah'},
        {q:'Sistem saraf manusia berpusat pada otak dan sumsum tulang belakang', correct:'Benar', wrong:'Salah'},
        {q:'Bunyi tidak dapat merambat melalui zat cair', correct:'Salah', wrong:'Benar'},
        {q:'Fungsi ginjal adalah menyaring darah dan membentuk urine', correct:'Benar', wrong:'Salah'},
      ],
      sulit: [
        {q:'Mitokondria adalah tempat respirasi sel', correct:'Benar', wrong:'Salah'},
        {q:'Hukum Newton pertama membahas tentang percepatan benda', correct:'Salah', wrong:'Benar'},
        {q:'Atom terdiri dari proton, neutron, dan elektron', correct:'Benar', wrong:'Salah'},
        {q:'Reaksi eksoterm melepaskan kalor ke lingkungan', correct:'Benar', wrong:'Salah'},
        {q:'DNA menyimpan informasi genetik makhluk hidup', correct:'Benar', wrong:'Salah'},
        {q:'Tekanan udara semakin tinggi seiring bertambahnya ketinggian tempat', correct:'Salah', wrong:'Benar'},
        {q:'Hukum Newton kedua menyatakan gaya sama dengan massa dikali percepatan', correct:'Benar', wrong:'Salah'},
        {q:'Reaksi endoterm menyerap kalor dari lingkungan', correct:'Benar', wrong:'Salah'},
        {q:'Fotosintesis menghasilkan oksigen dan glukosa', correct:'Benar', wrong:'Salah'},
        {q:'Sel hewan memiliki dinding sel seperti sel tumbuhan', correct:'Salah', wrong:'Benar'},
        {q:'Isotop adalah atom yang memiliki jumlah proton sama tetapi neutron berbeda', correct:'Benar', wrong:'Salah'},
        {q:'Hukum kekekalan energi menyatakan energi dapat diciptakan', correct:'Salah', wrong:'Benar'},
        {q:'Ikatan kovalen terbentuk dari pemakaian bersama elektron', correct:'Benar', wrong:'Salah'},
        {q:'Mutasi genetik dapat terjadi akibat radiasi', correct:'Benar', wrong:'Salah'},
        {q:'Gelombang elektromagnetik dapat merambat tanpa medium', correct:'Benar', wrong:'Salah'},
        {q:'Reaksi redoks melibatkan transfer elektron', correct:'Benar', wrong:'Salah'},
        {q:'Hukum kedua termodinamika menyatakan entropi alam semesta cenderung meningkat', correct:'Benar', wrong:'Salah'},
        {q:'Katalis tidak berpengaruh pada laju reaksi kimia', correct:'Salah', wrong:'Benar'},
        {q:'Sel darah merah berfungsi mengangkut oksigen ke seluruh tubuh', correct:'Benar', wrong:'Salah'},
        {q:'Hormon insulin berfungsi menurunkan kadar gula darah', correct:'Benar', wrong:'Salah'},
        {q:'Pergerakan lempeng tektonik dapat menyebabkan gempa bumi', correct:'Benar', wrong:'Salah'},
        {q:'Cahaya tampak adalah satu-satunya bagian dari spektrum elektromagnetik', correct:'Salah', wrong:'Benar'},
        {q:'Sistem periodik unsur disusun berdasarkan nomor atom', correct:'Benar', wrong:'Salah'},
        {q:'Homeostasis adalah kemampuan tubuh menjaga keseimbangan internal', correct:'Benar', wrong:'Salah'},
        {q:'Radioaktivitas adalah pemancaran partikel atau energi dari inti atom yang tidak stabil', correct:'Benar', wrong:'Salah'},
      ]
    },
    binggris: {
      mudah: [
        {q:'"Cat" artinya kucing', correct:'Benar', wrong:'Salah'},
        {q:'"Book" artinya meja', correct:'Salah', wrong:'Benar'},
        {q:'"Good morning" dipakai untuk menyapa di pagi hari', correct:'Benar', wrong:'Salah'},
        {q:'"I am" adalah bentuk singkat dari "I is"', correct:'Salah', wrong:'Benar'},
        {q:'"Red" adalah nama warna', correct:'Benar', wrong:'Salah'},
        {q:'"Dog" artinya anjing', correct:'Benar', wrong:'Salah'},
        {q:'"One, two, three" adalah urutan angka 1, 2, 3', correct:'Benar', wrong:'Salah'},
        {q:'"Monday" adalah nama hari', correct:'Benar', wrong:'Salah'},
        {q:'"Apple" artinya jeruk', correct:'Salah', wrong:'Benar'},
        {q:'"Thank you" digunakan untuk mengucapkan terima kasih', correct:'Benar', wrong:'Salah'},
        {q:'"Big" adalah lawan kata dari "small"', correct:'Benar', wrong:'Salah'},
        {q:'"Sun" artinya bulan', correct:'Salah', wrong:'Benar'},
        {q:'"Water" artinya air', correct:'Benar', wrong:'Salah'},
        {q:'"Family" berarti keluarga', correct:'Benar', wrong:'Salah'},
        {q:'"Happy" berarti sedih', correct:'Salah', wrong:'Benar'},
        {q:'"School" artinya sekolah', correct:'Benar', wrong:'Salah'},
        {q:'"Blue" adalah nama warna biru', correct:'Benar', wrong:'Salah'},
        {q:'"Car" artinya mobil', correct:'Benar', wrong:'Salah'},
        {q:'"Night" artinya siang', correct:'Salah', wrong:'Benar'},
        {q:'"Please" digunakan untuk meminta sesuatu dengan sopan', correct:'Benar', wrong:'Salah'},
        {q:'"Teacher" artinya guru', correct:'Benar', wrong:'Salah'},
        {q:'"Ten" adalah angka sepuluh', correct:'Benar', wrong:'Salah'},
        {q:'"Cold" adalah lawan kata dari "hot"', correct:'Benar', wrong:'Salah'},
        {q:'"Bird" artinya ikan', correct:'Salah', wrong:'Benar'},
        {q:'"Yes" dan "no" adalah kata untuk menjawab iya dan tidak', correct:'Benar', wrong:'Salah'},
      ],
      sedang: [
        {q:'Kata kerja bentuk ketiga dari "go" adalah "gone"', correct:'Benar', wrong:'Salah'},
        {q:'"She go to school every day" adalah kalimat yang gramatikal', correct:'Salah', wrong:'Benar'},
        {q:'"Yesterday" digunakan dalam simple past tense', correct:'Benar', wrong:'Salah'},
        {q:'Kata sifat (adjective) menerangkan kata benda', correct:'Benar', wrong:'Salah'},
        {q:'"Will" digunakan untuk menyatakan kejadian di masa lalu', correct:'Salah', wrong:'Benar'},
        {q:'"Because" adalah kata penghubung sebab akibat', correct:'Benar', wrong:'Salah'},
        {q:'Present continuous tense menggunakan rumus "to be + verb-ing"', correct:'Benar', wrong:'Salah'},
        {q:'"Do" dan "does" digunakan dalam kalimat tanya simple present tense', correct:'Benar', wrong:'Salah'},
        {q:'Kata benda jamak selalu ditambah huruf s tanpa pengecualian', correct:'Salah', wrong:'Benar'},
        {q:'"There is" digunakan untuk benda tunggal', correct:'Benar', wrong:'Salah'},
        {q:'"There are" digunakan untuk benda jamak', correct:'Benar', wrong:'Salah'},
        {q:'Kata keterangan (adverb) menerangkan kata kerja', correct:'Benar', wrong:'Salah'},
        {q:'"Must" digunakan untuk menyatakan keharusan yang kuat', correct:'Benar', wrong:'Salah'},
        {q:'Simple past tense digunakan untuk kejadian yang sedang berlangsung sekarang', correct:'Salah', wrong:'Benar'},
        {q:'"Can" digunakan untuk menyatakan kemampuan', correct:'Benar', wrong:'Salah'},
        {q:'Kalimat tanya dalam bahasa Inggris selalu diawali dengan kata kerja', correct:'Salah', wrong:'Benar'},
        {q:'"A" dan "an" adalah kata sandang tak tentu', correct:'Benar', wrong:'Salah'},
        {q:'"The" adalah kata sandang tertentu', correct:'Benar', wrong:'Salah'},
        {q:'Kata ganti "they" digunakan untuk orang ketiga jamak', correct:'Benar', wrong:'Salah'},
        {q:'"Never" berarti selalu', correct:'Salah', wrong:'Benar'},
        {q:'Comparative degree digunakan untuk membandingkan dua hal', correct:'Benar', wrong:'Salah'},
        {q:'Superlative degree digunakan untuk membandingkan lebih dari dua hal', correct:'Benar', wrong:'Salah'},
        {q:'"Although" dan "but" memiliki fungsi yang sama sebagai kata hubung pertentangan', correct:'Benar', wrong:'Salah'},
        {q:'Preposition of time seperti "at, on, in" digunakan untuk menunjukkan waktu', correct:'Benar', wrong:'Salah'},
        {q:'Kalimat perintah (imperative) selalu diawali dengan subjek', correct:'Salah', wrong:'Benar'},
      ],
      sulit: [
        {q:'Present perfect tense menggunakan "have/has + verb 3"', correct:'Benar', wrong:'Salah'},
        {q:'Kalimat pasif (passive voice) subjeknya melakukan aksi', correct:'Salah', wrong:'Benar'},
        {q:'"Although" digunakan untuk menyatakan pertentangan', correct:'Benar', wrong:'Salah'},
        {q:'Conditional sentence type 2 membahas kejadian yang pasti terjadi', correct:'Salah', wrong:'Benar'},
        {q:'"Idiom" adalah ungkapan yang maknanya tidak selalu harfiah', correct:'Benar', wrong:'Salah'},
        {q:'Reported speech mengubah kalimat langsung menjadi tidak langsung', correct:'Benar', wrong:'Salah'},
        {q:'Past perfect tense menunjukkan kejadian yang terjadi sebelum kejadian lain di masa lalu', correct:'Benar', wrong:'Salah'},
        {q:'Gerund adalah kata kerja yang berfungsi sebagai kata benda dengan akhiran -ing', correct:'Benar', wrong:'Salah'},
        {q:'Modal verbs seperti should, must, can selalu diikuti bentuk kata kerja ketiga', correct:'Salah', wrong:'Benar'},
        {q:'Relative clause menggunakan kata seperti "who, which, that"', correct:'Benar', wrong:'Salah'},
        {q:'Conditional sentence type 3 membahas penyesalan terhadap masa lalu', correct:'Benar', wrong:'Salah'},
        {q:'Direct speech dan indirect speech memiliki struktur yang sama persis', correct:'Salah', wrong:'Benar'},
        {q:'Causative verbs seperti "have" dan "get" digunakan untuk menyuruh orang lain melakukan sesuatu', correct:'Benar', wrong:'Salah'},
        {q:'Subjunctive mood digunakan untuk menyatakan harapan atau situasi hipotetis', correct:'Benar', wrong:'Salah'},
        {q:'Phrasal verb terdiri dari kata kerja dan partikel', correct:'Benar', wrong:'Salah'},
        {q:'Future perfect tense digunakan untuk kejadian yang belum pasti terjadi', correct:'Salah', wrong:'Benar'},
        {q:'Parallel structure penting dalam penulisan kalimat yang baik', correct:'Benar', wrong:'Salah'},
        {q:'Ellipsis dalam gramatika berarti penghilangan bagian kalimat yang sudah jelas dari konteks', correct:'Benar', wrong:'Salah'},
        {q:'Inversion selalu digunakan dalam setiap kalimat bahasa Inggris', correct:'Salah', wrong:'Benar'},
        {q:'Tag question digunakan untuk meminta konfirmasi', correct:'Benar', wrong:'Salah'},
        {q:'Non-defining relative clause diapit tanda koma', correct:'Benar', wrong:'Salah'},
        {q:'Discourse markers membantu menghubungkan ide dalam sebuah teks', correct:'Benar', wrong:'Salah'},
        {q:'Collocation adalah kombinasi kata yang biasa digunakan bersama', correct:'Benar', wrong:'Salah'},
        {q:'Register dalam bahasa mengacu pada tingkat formalitas bahasa yang digunakan', correct:'Benar', wrong:'Salah'},
        {q:'Cohesion dan coherence memiliki makna yang identik tanpa perbedaan', correct:'Salah', wrong:'Benar'},
      ]
    },
    agama_islam: {
      mudah: [
        {q:'Rukun Islam ada 5', correct:'Benar', wrong:'Salah'},
        {q:'Shalat lima waktu wajib dikerjakan setiap hari', correct:'Benar', wrong:'Salah'},
        {q:'Al-Quran adalah kitab suci umat Islam', correct:'Benar', wrong:'Salah'},
        {q:'Puasa Ramadhan dilakukan setiap bulan sepanjang tahun', correct:'Salah', wrong:'Benar'},
        {q:'Nabi Muhammad SAW adalah nabi terakhir', correct:'Benar', wrong:'Salah'},
        {q:'Zakat termasuk salah satu rukun Islam', correct:'Benar', wrong:'Salah'},
        {q:'Shalat menghadap ke arah kiblat', correct:'Benar', wrong:'Salah'},
        {q:'Membaca basmalah dianjurkan sebelum memulai suatu pekerjaan', correct:'Benar', wrong:'Salah'},
        {q:'Adzan adalah panggilan untuk melaksanakan shalat', correct:'Benar', wrong:'Salah'},
        {q:'Kabah terletak di kota Madinah', correct:'Salah', wrong:'Benar'},
        {q:'Bulan puasa disebut bulan Ramadhan', correct:'Benar', wrong:'Salah'},
        {q:'Idul Fitri dirayakan setelah bulan puasa', correct:'Benar', wrong:'Salah'},
        {q:'Berbohong adalah perbuatan terpuji dalam Islam', correct:'Salah', wrong:'Benar'},
        {q:'Wudhu dilakukan sebelum shalat', correct:'Benar', wrong:'Salah'},
        {q:'Membantu orang lain adalah perbuatan terpuji', correct:'Benar', wrong:'Salah'},
        {q:'Al-Quran diturunkan kepada Nabi Muhammad SAW', correct:'Benar', wrong:'Salah'},
        {q:'Shalat Jumat wajib bagi laki-laki muslim', correct:'Benar', wrong:'Salah'},
        {q:'Anak-anak tidak perlu belajar mengaji', correct:'Salah', wrong:'Benar'},
        {q:'Berdoa dilakukan untuk memohon kepada Allah SWT', correct:'Benar', wrong:'Salah'},
        {q:'Menyayangi sesama makhluk termasuk akhlak mulia', correct:'Benar', wrong:'Salah'},
        {q:'Rukun Islam yang pertama adalah syahadat', correct:'Benar', wrong:'Salah'},
        {q:'Shalat subuh dikerjakan sebanyak 2 rakaat', correct:'Benar', wrong:'Salah'},
        {q:'Idul Adha dikenal juga sebagai hari raya kurban', correct:'Benar', wrong:'Salah'},
        {q:'Mencuri adalah perbuatan yang dilarang dalam Islam', correct:'Benar', wrong:'Salah'},
        {q:'Masjid adalah tempat ibadah umat Islam', correct:'Benar', wrong:'Salah'},
      ],
      sedang: [
        {q:'Rukun Iman ada 6', correct:'Benar', wrong:'Salah'},
        {q:'Wudhu tidak diperlukan sebelum shalat', correct:'Salah', wrong:'Benar'},
        {q:'Haji wajib dilaksanakan bagi yang mampu', correct:'Benar', wrong:'Salah'},
        {q:'Sedekah hanya boleh diberikan saat bulan Ramadhan', correct:'Salah', wrong:'Benar'},
        {q:'Asmaul Husna adalah nama-nama baik Allah SWT', correct:'Benar', wrong:'Salah'},
        {q:'Akhlak terpuji dianjurkan dalam ajaran Islam', correct:'Benar', wrong:'Salah'},
        {q:'Malaikat Jibril bertugas menyampaikan wahyu kepada para nabi', correct:'Benar', wrong:'Salah'},
        {q:'Puasa dapat berkurang pahalanya apabila disertai perbuatan tercela', correct:'Benar', wrong:'Salah'},
        {q:'Zakat fitrah dikeluarkan pada bulan Ramadhan menjelang Idul Fitri', correct:'Benar', wrong:'Salah'},
        {q:'Shalat berjamaah lebih utama daripada shalat sendirian', correct:'Benar', wrong:'Salah'},
        {q:'Al-Quran terdiri dari 30 juz', correct:'Benar', wrong:'Salah'},
        {q:'Nabi Ibrahim AS dikenal sebagai bapak para nabi', correct:'Benar', wrong:'Salah'},
        {q:'Setiap perbuatan baik dicatat sebagai pahala oleh malaikat', correct:'Benar', wrong:'Salah'},
        {q:'Tayamum dilakukan hanya jika air sangat berlebihan', correct:'Salah', wrong:'Benar'},
        {q:'Iman kepada kitab-kitab Allah termasuk rukun iman', correct:'Benar', wrong:'Salah'},
        {q:'Qadha dan qadar berkaitan dengan takdir Allah SWT', correct:'Benar', wrong:'Salah'},
        {q:'Sunnah adalah perbuatan yang dianjurkan tetapi tidak wajib', correct:'Benar', wrong:'Salah'},
        {q:'Riba adalah tambahan yang diharamkan dalam transaksi keuangan Islam', correct:'Benar', wrong:'Salah'},
        {q:'Ikhlas berarti melakukan sesuatu semata-mata karena Allah SWT', correct:'Benar', wrong:'Salah'},
        {q:'Ghibah adalah perbuatan terpuji yang dianjurkan dalam Islam', correct:'Salah', wrong:'Benar'},
        {q:'Shalat tarawih dilaksanakan pada bulan Ramadhan', correct:'Benar', wrong:'Salah'},
        {q:'Nabi Muhammad SAW lahir di kota Mekkah', correct:'Benar', wrong:'Salah'},
        {q:'Hijrah adalah perpindahan Nabi Muhammad SAW dari Mekkah ke Madinah', correct:'Benar', wrong:'Salah'},
        {q:'Munafik adalah orang yang selalu berkata benar dan berbuat sesuai perkataannya', correct:'Salah', wrong:'Benar'},
        {q:'Menuntut ilmu adalah kewajiban bagi setiap muslim', correct:'Benar', wrong:'Salah'},
      ],
      sulit: [
        {q:'Ijma adalah kesepakatan para ulama dalam suatu hukum', correct:'Benar', wrong:'Salah'},
        {q:'Qiyas adalah salah satu sumber hukum Islam', correct:'Benar', wrong:'Salah'},
        {q:'Hadits adalah perkataan, perbuatan, dan ketetapan Nabi Muhammad SAW', correct:'Benar', wrong:'Salah'},
        {q:'Fikih membahas tentang tata cara ibadah dan muamalah', correct:'Benar', wrong:'Salah'},
        {q:'Tajwid tidak berkaitan dengan cara membaca Al-Quran', correct:'Salah', wrong:'Benar'},
        {q:'Muamalah mengatur hubungan sosial antar manusia', correct:'Benar', wrong:'Salah'},
        {q:'Ushul fikih adalah ilmu tentang dasar-dasar penetapan hukum Islam', correct:'Benar', wrong:'Salah'},
        {q:'Hadits shahih adalah hadits yang sanad dan matannya terpercaya', correct:'Benar', wrong:'Salah'},
        {q:'Ijtihad adalah usaha sungguh-sungguh ulama untuk menetapkan hukum yang belum ada dalilnya secara jelas', correct:'Benar', wrong:'Salah'},
        {q:'Nasakh adalah penghapusan suatu hukum oleh hukum lain yang datang kemudian', correct:'Benar', wrong:'Salah'},
        {q:'Aqidah membahas tentang keyakinan dasar dalam Islam', correct:'Benar', wrong:'Salah'},
        {q:'Tasawuf berkaitan dengan aspek batin dan penyucian jiwa dalam Islam', correct:'Benar', wrong:'Salah'},
        {q:'Semua hadits memiliki tingkat keshahihan yang sama', correct:'Salah', wrong:'Benar'},
        {q:'Maqashid syariah adalah tujuan-tujuan pokok disyariatkannya hukum Islam', correct:'Benar', wrong:'Salah'},
        {q:'Khilafiyah adalah perbedaan pendapat di kalangan ulama dalam masalah fikih', correct:'Benar', wrong:'Salah'},
        {q:'Al-Quran dan hadits adalah dua sumber utama hukum Islam', correct:'Benar', wrong:'Salah'},
        {q:'Mazhab dalam fikih hanya ada satu dan tidak ada perbedaan pendapat', correct:'Salah', wrong:'Benar'},
        {q:'Zakat mal dikenakan atas harta yang telah mencapai nisab dan haul', correct:'Benar', wrong:'Salah'},
        {q:'Wakaf adalah menahan harta untuk dimanfaatkan bagi kepentingan umum', correct:'Benar', wrong:'Salah'},
        {q:'Munakahat adalah bagian fikih yang membahas tentang pernikahan', correct:'Benar', wrong:'Salah'},
        {q:'Jinayat adalah bagian fikih yang membahas tentang hukum pidana Islam', correct:'Benar', wrong:'Salah'},
        {q:'Dalil aqli tidak memiliki peran dalam penetapan hukum Islam', correct:'Salah', wrong:'Benar'},
        {q:'Istihsan adalah salah satu metode ijtihad dalam menetapkan hukum', correct:'Benar', wrong:'Salah'},
        {q:'Fatwa adalah pendapat hukum yang dikeluarkan oleh ulama atau lembaga berwenang', correct:'Benar', wrong:'Salah'},
        {q:'Perbuatan manusia dalam Islam terbagi menjadi lima hukum: wajib, sunnah, mubah, makruh, dan haram', correct:'Benar', wrong:'Salah'},
      ]
    },
    ips: {
      mudah: [
        {q:'Indonesia terletak di benua Asia', correct:'Benar', wrong:'Salah'},
        {q:'Ibu kota Indonesia adalah Jakarta', correct:'Benar', wrong:'Salah'},
        {q:'Indonesia terdiri dari satu pulau saja', correct:'Salah', wrong:'Benar'},
        {q:'Pasar adalah tempat jual beli barang', correct:'Benar', wrong:'Salah'},
        {q:'Gunung Everest ada di Indonesia', correct:'Salah', wrong:'Benar'},
        {q:'Koperasi adalah usaha bersama berasaskan kekeluargaan', correct:'Benar', wrong:'Salah'},
        {q:'Indonesia dilalui garis khatulistiwa', correct:'Benar', wrong:'Salah'},
        {q:'Indonesia memiliki dua musim, yaitu hujan dan kemarau', correct:'Benar', wrong:'Salah'},
        {q:'Uang digunakan sebagai alat tukar dalam kegiatan jual beli', correct:'Benar', wrong:'Salah'},
        {q:'Petani bekerja di bidang pertanian', correct:'Benar', wrong:'Salah'},
        {q:'Nelayan bekerja mencari ikan di laut', correct:'Benar', wrong:'Salah'},
        {q:'Peta digunakan untuk menunjukkan lokasi suatu wilayah', correct:'Benar', wrong:'Salah'},
        {q:'Indonesia hanya memiliki satu bahasa daerah', correct:'Salah', wrong:'Benar'},
        {q:'Kegiatan ekonomi meliputi produksi, distribusi, dan konsumsi', correct:'Benar', wrong:'Salah'},
        {q:'Candi Borobudur terletak di provinsi Jawa Tengah', correct:'Benar', wrong:'Salah'},
        {q:'Suku Batak berasal dari Sumatera Utara', correct:'Benar', wrong:'Salah'},
        {q:'Gotong royong adalah bentuk kerja sama dalam masyarakat', correct:'Benar', wrong:'Salah'},
        {q:'Indonesia adalah negara kepulauan', correct:'Benar', wrong:'Salah'},
        {q:'Sungai dapat dimanfaatkan untuk irigasi pertanian', correct:'Benar', wrong:'Salah'},
        {q:'Setiap daerah di Indonesia memiliki budaya yang sama persis', correct:'Salah', wrong:'Benar'},
        {q:'Transportasi darat, laut, dan udara digunakan untuk berpindah tempat', correct:'Benar', wrong:'Salah'},
        {q:'Bank adalah lembaga yang menyimpan dan meminjamkan uang', correct:'Benar', wrong:'Salah'},
        {q:'Sumber daya alam harus dijaga kelestariannya', correct:'Benar', wrong:'Salah'},
        {q:'Peninggalan sejarah perlu dilestarikan', correct:'Benar', wrong:'Salah'},
        {q:'Kerja sama antarwilayah dapat mempererat persatuan', correct:'Benar', wrong:'Salah'},
      ],
      sedang: [
        {q:'Letak geografis Indonesia berada di antara dua benua dan dua samudra', correct:'Benar', wrong:'Salah'},
        {q:'Inflasi adalah kenaikan harga barang secara terus-menerus', correct:'Benar', wrong:'Salah'},
        {q:'Urbanisasi adalah perpindahan penduduk dari kota ke desa', correct:'Salah', wrong:'Benar'},
        {q:'Sumber daya alam terbagi menjadi dapat diperbarui dan tidak dapat diperbarui', correct:'Benar', wrong:'Salah'},
        {q:'Globalisasi mempermudah pertukaran informasi antarnegara', correct:'Benar', wrong:'Salah'},
        {q:'Kegiatan ekonomi hanya terdiri dari produksi saja', correct:'Salah', wrong:'Benar'},
        {q:'Letak geologis Indonesia berada pada pertemuan tiga lempeng tektonik dunia', correct:'Benar', wrong:'Salah'},
        {q:'Permintaan dan penawaran memengaruhi harga barang di pasar', correct:'Benar', wrong:'Salah'},
        {q:'Interaksi sosial terjadi antara individu dengan individu maupun kelompok', correct:'Benar', wrong:'Salah'},
        {q:'Mobilitas sosial adalah perpindahan status sosial seseorang', correct:'Benar', wrong:'Salah'},
        {q:'Revolusi hijau bertujuan meningkatkan produksi pertanian', correct:'Benar', wrong:'Salah'},
        {q:'Kolonialisme Belanda di Indonesia berlangsung sekitar 3,5 abad', correct:'Benar', wrong:'Salah'},
        {q:'Perdagangan bebas berarti tidak ada hambatan tarif antarnegara', correct:'Benar', wrong:'Salah'},
        {q:'Lembaga sosial hanya terdiri dari keluarga saja', correct:'Salah', wrong:'Benar'},
        {q:'Konflik sosial dapat terjadi karena perbedaan kepentingan', correct:'Benar', wrong:'Salah'},
        {q:'Kepadatan penduduk memengaruhi ketersediaan lahan', correct:'Benar', wrong:'Salah'},
        {q:'Perubahan sosial dapat terjadi secara cepat maupun lambat', correct:'Benar', wrong:'Salah'},
        {q:'Sistem ekonomi Indonesia adalah ekonomi Pancasila', correct:'Benar', wrong:'Salah'},
        {q:'Perdagangan antarpulau termasuk kegiatan distribusi', correct:'Benar', wrong:'Salah'},
        {q:'Letak astronomis tidak memengaruhi iklim suatu wilayah', correct:'Salah', wrong:'Benar'},
        {q:'Industrialisasi dapat meningkatkan pertumbuhan ekonomi suatu negara', correct:'Benar', wrong:'Salah'},
        {q:'Peta topografi menggambarkan ketinggian permukaan bumi', correct:'Benar', wrong:'Salah'},
        {q:'Organisasi ASEAN beranggotakan negara-negara di kawasan Asia Tenggara', correct:'Benar', wrong:'Salah'},
        {q:'Perubahan sosial selalu membawa dampak negatif', correct:'Salah', wrong:'Benar'},
        {q:'Distribusi pendapatan yang merata mengurangi kesenjangan sosial', correct:'Benar', wrong:'Salah'},
      ],
      sulit: [
        {q:'Letak astronomis Indonesia berada antara 6 derajat LU-11 derajat LS dan 95 derajat BT-141 derajat BT', correct:'Benar', wrong:'Salah'},
        {q:'Kolonialisme adalah praktik penguasaan wilayah oleh bangsa lain', correct:'Benar', wrong:'Salah'},
        {q:'APBN adalah singkatan dari Anggaran Pendapatan dan Belanja Negara', correct:'Benar', wrong:'Salah'},
        {q:'Interaksi sosial hanya terjadi antar individu, tidak antar kelompok', correct:'Salah', wrong:'Benar'},
        {q:'Perdagangan internasional dipengaruhi oleh keunggulan komparatif', correct:'Benar', wrong:'Salah'},
        {q:'Revolusi industri tidak berpengaruh pada perkembangan ekonomi dunia', correct:'Salah', wrong:'Benar'},
        {q:'Teori kependudukan Malthus membahas pertumbuhan penduduk dan pangan', correct:'Benar', wrong:'Salah'},
        {q:'Kebijakan moneter dikendalikan oleh bank sentral suatu negara', correct:'Benar', wrong:'Salah'},
        {q:'Kebijakan fiskal berkaitan dengan pengaturan pajak dan belanja negara', correct:'Benar', wrong:'Salah'},
        {q:'Pasar monopoli dikuasai oleh banyak penjual', correct:'Salah', wrong:'Benar'},
        {q:'Produk domestik bruto mengukur nilai total produksi suatu negara', correct:'Benar', wrong:'Salah'},
        {q:'Teori dependensi menjelaskan ketergantungan negara berkembang pada negara maju', correct:'Benar', wrong:'Salah'},
        {q:'Perang Dunia II berakhir pada tahun 1945', correct:'Benar', wrong:'Salah'},
        {q:'Konferensi Asia Afrika diselenggarakan di Bandung tahun 1955', correct:'Benar', wrong:'Salah'},
        {q:'Zona ekonomi eksklusif Indonesia adalah 200 mil laut dari garis pantai', correct:'Benar', wrong:'Salah'},
        {q:'Stratifikasi sosial tidak ditemukan dalam masyarakat modern', correct:'Salah', wrong:'Benar'},
        {q:'Deforestasi dapat menyebabkan perubahan iklim', correct:'Benar', wrong:'Salah'},
        {q:'Teori pembangunan berkelanjutan mempertimbangkan aspek ekonomi, sosial, dan lingkungan', correct:'Benar', wrong:'Salah'},
        {q:'Perdagangan bilateral melibatkan lebih dari dua negara', correct:'Salah', wrong:'Benar'},
        {q:'Konflik antarnegara dapat diselesaikan melalui diplomasi', correct:'Benar', wrong:'Salah'},
        {q:'Migrasi internasional dapat memengaruhi struktur demografi suatu negara', correct:'Benar', wrong:'Salah'},
        {q:'Kebijakan proteksionisme membatasi masuknya barang impor', correct:'Benar', wrong:'Salah'},
        {q:'Teori pusat pertumbuhan menjelaskan konsentrasi kegiatan ekonomi pada wilayah tertentu', correct:'Benar', wrong:'Salah'},
        {q:'Perang Dingin terjadi antara blok Barat dan blok Timur', correct:'Benar', wrong:'Salah'},
        {q:'Reformasi agraria bertujuan mendistribusikan lahan secara lebih adil', correct:'Benar', wrong:'Salah'},
      ]
    }
  };
  var MAPEL_LIST = [
    {v:'bindo', label:'Bahasa Indonesia'},
    {v:'binggris', label:'Bahasa Inggris'},
    {v:'pkn', label:'PKn'},
    {v:'agama_islam', label:'Pendidikan Agama Islam'},
    {v:'agama_lain', label:'Pendidikan Agama Lainnya'},
    {v:'matematika', label:'Matematika'},
    {v:'informatika', label:'Informatika'},
    {v:'seni_budaya', label:'Seni Budaya'},
    {v:'prakarya', label:'Prakarya'},
    {v:'ipa', label:'IPA'},
    {v:'ips', label:'IPS'},
    {v:'sosiologi', label:'Sosiologi'},
    {v:'koding', label:'Koding'},
    {v:'penjas', label:'Penjas'},
    {v:'umum', label:'Pengetahuan Umum'}
  ];

  var JENJANG = {
    sd: {label:'SD', kelas:[1,2,3,4,5,6]},
    smp: {label:'SMP/MTs', kelas:[7,8,9]},
    sma: {label:'SMA/SMK', kelas:[10,11,12]}
  };

  function ready(fn){
    if(document.readyState !== 'loading') fn();
    else document.addEventListener('DOMContentLoaded', fn);
  }

  ready(function(){
    var sheet = document.querySelector('#mxStart .mx-sheet');
    var timerCard = document.querySelector('#mxStart .mx-card');
    if(!sheet || !timerCard) return;

    var mapelOptions = MAPEL_LIST.map(function(m){
      return '<option value="'+m.v+'">'+m.label+'</option>';
    }).join('');

    var html = ''
      + '<div class="mxp-card" id="mxpCard">'
      + '  <div class="mxp-title">Pilih mapel &amp; jenjang siap pakai</div>'
      + '  <div class="mxp-hint">Pilih jenjang, kelas, mapel dan tingkat kesulitan, lalu muat soalnya. Soal tetap bisa diedit setelah dimuat.</div>'
      + '  <div class="mxp-grid">'
      + '    <div class="mxp-field"><label for="mxpJenjang">Jenjang</label>'
      + '      <select id="mxpJenjang">'
      + '        <option value="sd">SD (Kelas 1-6)</option>'
      + '        <option value="smp">SMP/MTs (Kelas 7-9)</option>'
      + '        <option value="sma">SMA/SMK (Kelas 10-12)</option>'
      + '      </select></div>'
      + '    <div class="mxp-field"><label for="mxpKelas">Kelas</label>'
      + '      <select id="mxpKelas"></select></div>'
      + '    <div class="mxp-field"><label for="mxpMapel">Mapel</label>'
      + '      <select id="mxpMapel">'+mapelOptions+'</select></div>'
      + '    <div class="mxp-field"><label for="mxpTingkat">Tingkat kesulitan</label>'
      + '      <select id="mxpTingkat">'
      + '        <option value="mudah">Mudah</option>'
      + '        <option value="sedang" selected>Sedang</option>'
      + '        <option value="sulit">Sulit</option>'
      + '      </select></div>'
      + '  </div>'
      + '  <div class="mxp-actions">'
      + '    <button type="button" class="mxp-btn" id="mxpLoad">Muat soal ini</button>'
      + '    <button type="button" class="mxp-btn secondary" id="mxpLoadEdit">Muat lalu buka Edit soal</button>'
      + '  </div>'
      + '  <div class="mxp-status" id="mxpStatus"></div>'
      + '</div>';

    timerCard.insertAdjacentHTML('afterend', html);

    var jenjangEl = document.getElementById('mxpJenjang');
    var kelasEl = document.getElementById('mxpKelas');
    var mapelEl = document.getElementById('mxpMapel');
    var tingkatEl = document.getElementById('mxpTingkat');
    var statusEl = document.getElementById('mxpStatus');

    function refreshKelas(){
      var j = JENJANG[jenjangEl.value] || JENJANG.sd;
      kelasEl.innerHTML = j.kelas.map(function(k){ return '<option value="'+k+'">Kelas '+k+'</option>'; }).join('');
    }
    jenjangEl.addEventListener('change', refreshKelas);
    refreshKelas();

    function findMapelLabel(v){
      for(var i=0;i<MAPEL_LIST.length;i++){ if(MAPEL_LIST[i].v === v) return MAPEL_LIST[i].label; }
      return v;
    }

    function doLoad(openEditor){
      var mapel = mapelEl.value, tingkat = tingkatEl.value;
      var jenjangLabel = (JENJANG[jenjangEl.value] || JENJANG.sd).label;
      var kelas = kelasEl.value;
      var set = PRESET_BANK[mapel] && PRESET_BANK[mapel][tingkat];

      if(!set || !set.length){
        statusEl.textContent = 'Bank soal untuk "' + findMapelLabel(mapel) + '" tingkat ' + tingkat + ' sedang disiapkan dan belum tersedia. Mapel yang sudah tersedia: Matematika, Bahasa Indonesia, PKn, IPA, Bahasa Inggris, Pendidikan Agama Islam, IPS. Mapel lain bisa diisi manual lewat "Edit soal".';
        return;
      }

      var current = null;
      try{ current = localStorage.getItem('kuisBK.v1.soal'); }catch(e){}
      var hasCurrent = false;
      try{ hasCurrent = current && JSON.parse(current).length > 0; }catch(e){}

      if(hasCurrent && !confirm('Ganti soal saat ini dengan ' + set.length + ' soal siap pakai (' + findMapelLabel(mapel) + ' - ' + jenjangLabel + ' Kelas ' + kelas + ' - ' + tingkat + ')? Soal lama akan digantikan (masih bisa diedit lagi setelah dimuat).')){
        return;
      }

      var newBank = set.map(function(it){ return {q:it.q, correct:it.correct, wrong:it.wrong, side:'acak'}; });
      try{
        localStorage.setItem('kuisBK.v1.soal', JSON.stringify(newBank));
        if(openEditor) localStorage.setItem('kuisBK.v1.openEditor', '1');
      }catch(e){
        statusEl.textContent = 'Browser menolak penyimpanan lokal. Coba gunakan browser lain atau mode bukan privat.';
        return;
      }
      location.reload();
    }

    document.getElementById('mxpLoad').addEventListener('click', function(){ doLoad(false); });
    document.getElementById('mxpLoadEdit').addEventListener('click', function(){ doLoad(true); });

    // Jika diminta buka editor otomatis setelah reload
    try{
      if(localStorage.getItem('kuisBK.v1.openEditor') === '1'){
        localStorage.removeItem('kuisBK.v1.openEditor');
        var editBtn = document.getElementById('mxEditBtn');
        if(editBtn) setTimeout(function(){ editBtn.click(); }, 50);
      }
    }catch(e){}
  });
})();
</script>

</body>
</html>

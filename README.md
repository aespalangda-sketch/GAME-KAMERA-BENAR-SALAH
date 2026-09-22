[APLIKASI_GAME_KAMERA_PILIHAN_JAWABAN_3_PENJAS_FIXED_3_(1).html](https://github.com/user-attachments/files/32519389/APLIKASI_GAME_KAMERA_PILIHAN_JAWABAN_3_PENJAS_FIXED_3_.1.html)
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
    transition:transform .2s ease;
  }
  #camZoomWrap{display:none; flex-direction:column; gap:4px; margin-top:2px;}
  #camZoomWrap label{font-size:.72rem; color:var(--muted); letter-spacing:.2px;}
  #camZoomSlider{width:100%;}
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
  .mxg-social{margin-top:16px; padding-top:16px; border-top:1px solid rgba(255,255,255,.1);}
  .mxg-social-text{color:var(--muted); font-size:.85rem; margin-bottom:10px;}
  .mxg-social-row{display:flex; align-items:center; justify-content:center; gap:14px;}
  .mxg-social-row a{
    display:flex; align-items:center; justify-content:center;
    width:44px; height:44px; border-radius:12px;
    background:rgba(255,255,255,.08); border:1px solid rgba(255,255,255,.14);
    color:var(--text); transition:filter .15s, background .15s;
  }
  .mxg-social-row a:hover{background:rgba(255,255,255,.18); filter:brightness(1.08);}
  .mxg-social-row svg{width:22px; height:22px;}
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
    <div class="mxg-social">
      <p class="mxg-social-text">Hubungi admin untuk mendapatkan password game</p>
      <div class="mxg-social-row">
        <a href="https://wa.me/6285397772345" target="_blank" rel="noopener noreferrer" aria-label="Hubungi Admin via WhatsApp" title="Admin">
          <svg viewBox="0 0 24 24" fill="currentColor" aria-hidden="true"><path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.095 3.2 5.076 4.487.709.306 1.263.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347z"/><path d="M12.001 2.003c-5.514 0-9.997 4.483-9.997 9.997 0 1.763.462 3.486 1.34 5.004L2 22l5.116-1.318a9.958 9.958 0 0 0 4.885 1.278h.004c5.514 0 9.997-4.483 9.997-9.997 0-2.67-1.04-5.18-2.929-7.069a9.933 9.933 0 0 0-7.072-2.891zm0 18.164h-.003a8.153 8.153 0 0 1-4.158-1.14l-.298-.177-3.037.783.81-2.96-.194-.304a8.15 8.15 0 0 1-1.25-4.362c0-4.51 3.67-8.18 8.184-8.18a8.13 8.13 0 0 1 5.786 2.399 8.126 8.126 0 0 1 2.395 5.788c0 4.51-3.671 8.18-8.235 8.18z"/></svg>
        </a>
        <a href="https://instagram.com/aes_435" target="_blank" rel="noopener noreferrer" aria-label="Instagram @aes_435" title="Instagram @aes_435">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" aria-hidden="true"><rect x="3" y="3" width="18" height="18" rx="5" ry="5"/><circle cx="12" cy="12" r="4"/><circle cx="17.4" cy="6.6" r="1.1" fill="currentColor" stroke="none"/></svg>
        </a>
        <a href="https://www.tiktok.com/@aes_435" target="_blank" rel="noopener noreferrer" aria-label="TikTok @aes_435" title="TikTok @aes_435">
          <svg viewBox="0 0 24 24" fill="currentColor" aria-hidden="true"><path d="M16.5 3c.36 1.94 1.6 3.34 3.5 3.8v2.63a6.86 6.86 0 0 1-3.5-1.17v6.1a5.64 5.64 0 1 1-5.64-5.64c.2 0 .4.01.6.04v2.7a2.97 2.97 0 1 0 2.4 2.9V3h2.64z"/></svg>
        </a>
      </div>
    </div>
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

  <button type="button" class="mx-game-btn neutral" id="camModeBtn"
    style="position:absolute; top:16px; right:16px; z-index:32;">Kamera: Luas</button>
  <div id="camZoomWrap"
    style="position:absolute; top:64px; right:16px; z-index:32; display:none; flex-direction:column; gap:6px;
    background:rgba(7,11,20,.92); border:1px solid rgba(255,255,255,.14); border-radius:12px; padding:10px 12px; width:190px;">
    <label for="camZoomSlider" style="font-size:.72rem; color:var(--muted); letter-spacing:.2px;">Perbesar tampilan kamera</label>
    <input type="range" id="camZoomSlider" min="105" max="200" step="5" value="135" style="width:100%;">
  </div>

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

function shuffleQuestions(){
  for(let i = questions.length - 1; i > 0; i--){
    const j = Math.floor(Math.random() * (i + 1));
    const tmp = questions[i]; questions[i] = questions[j]; questions[j] = tmp;
  }
}

function resetQuiz(){
  shuffleQuestions();
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

// ---- Mode Kamera: Luas (bawaan, lebar penuh) vs Biasa (di-zoom sedikit) ----
// Webcam laptop/IFP biasanya wide-angle sehingga area yang tertangkap terlalu luas.
// Karena browser umumnya tidak bisa mengatur optical zoom kamera, mode "Biasa" di sini
// memperbesar (crop) tampilan video secara visual agar area yang terlihat lebih pas/dekat,
// dan guru bisa menyesuaikan sendiri besarannya lewat penggeser.
(function(){
  var camModeBtn = document.getElementById('camModeBtn');
  var camZoomWrap = document.getElementById('camZoomWrap');
  var camZoomSlider = document.getElementById('camZoomSlider');
  var LS_MODE = 'kuisBK.v1.camMode';
  var LS_ZOOM = 'kuisBK.v1.camZoomPct';

  function applyZoom(pct){
    var scale = (parseInt(pct, 10) || 135) / 100;
    video.style.transform = 'scaleX(-1) scale(' + scale + ')';
  }

  function setMode(mode){
    if(mode === 'zoom'){
      applyZoom(camZoomSlider.value);
      camModeBtn.textContent = 'Kamera: Zoom (ke Luas)';
      camZoomWrap.style.display = 'flex';
    } else {
      mode = 'wide';
      video.style.transform = 'scaleX(-1) scale(1)';
      camModeBtn.textContent = 'Kamera: Luas (ke Zoom)';
      camZoomWrap.style.display = 'none';
    }
    camModeBtn.dataset.mode = mode;
    try{ localStorage.setItem(LS_MODE, mode); }catch(e){}
  }

  camModeBtn.addEventListener('click', function(){
    setMode(camModeBtn.dataset.mode === 'zoom' ? 'wide' : 'zoom');
  });

  camZoomSlider.addEventListener('input', function(){
    applyZoom(camZoomSlider.value);
    try{ localStorage.setItem(LS_ZOOM, camZoomSlider.value); }catch(e){}
  });

  var savedMode = 'wide', savedZoom = 135;
  try{
    savedMode = localStorage.getItem(LS_MODE) || 'wide';
    savedZoom = parseInt(localStorage.getItem(LS_ZOOM), 10) || 135;
  }catch(e){}
  camZoomSlider.value = savedZoom;
  setMode(savedMode);
})();

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
        <p style="margin-top:16px; font-size:0.75rem; line-height:1.5; text-align:center; opacity:0.8;">Aplikasi ini adalah karya asli AES (@aes_435). Jika Anda mendapatkan aplikasi ini dari pihak lain (bukan langsung dari admin resmi di atas), kemungkinan ini hasil jual-beli tanpa izin. Mohon laporkan ke kontak di atas.</p>
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
  .mxp-field select, .mxp-field input[type="number"]{width:100%; height:42px; border-radius:10px; border:1px solid rgba(255,255,255,.18); background:#0D1424; color:var(--text); padding:0 10px; font-family:'Inter',sans-serif; font-size:.92rem;}
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
        {q:'Hasil dari 5 + 3 adalah...', correct:'8', wrong:'9'},
        {q:'Hasil dari 10 - 4 adalah...', correct:'6', wrong:'5'},
        {q:'Hasil dari 2 x 6 adalah...', correct:'12', wrong:'10'},
        {q:'Hasil dari 9 dibagi 3 adalah...', correct:'3', wrong:'4'},
        {q:'Manakah pernyataan yang benar tentang 7 dan 10?', correct:'7 lebih kecil dari 10', wrong:'7 lebih besar dari 10'},
        {q:'Bilangan genap terkecil adalah...', correct:'0', wrong:'2'},
        {q:'Hasil dari 4 x 4 adalah...', correct:'16', wrong:'12'},
        {q:'Hasil dari 15 - 7 adalah...', correct:'8', wrong:'9'},
        {q:'Segitiga memiliki jumlah sisi sebanyak...', correct:'3 sisi', wrong:'4 sisi'},
        {q:'Persegi memiliki jumlah sisi sebanyak...', correct:'4 sisi', wrong:'5 sisi'},
        {q:'Hasil dari 100 dibagi 10 adalah...', correct:'10', wrong:'100'},
        {q:'Hasil dari 6 + 6 adalah...', correct:'12', wrong:'11'},
        {q:'Bilangan ganjil setelah 5 adalah...', correct:'7', wrong:'6'},
        {q:'Hasil dari 3 x 3 adalah...', correct:'9', wrong:'6'},
        {q:'Lingkaran memiliki jumlah sudut sebanyak...', correct:'Tidak memiliki sudut', wrong:'4 sudut'},
        {q:'Hasil dari 20 - 5 adalah...', correct:'15', wrong:'10'},
        {q:'Hasil dari 8 + 9 adalah...', correct:'17', wrong:'16'},
        {q:'Setengah dari 10 adalah...', correct:'5', wrong:'4'},
        {q:'Segi empat memiliki jumlah sudut sebanyak...', correct:'4 sudut', wrong:'3 sudut'},
        {q:'Hasil dari 7 x 2 adalah...', correct:'14', wrong:'12'},
        {q:'Bilangan 1 termasuk bilangan...', correct:'Ganjil', wrong:'Genap'},
        {q:'Hasil dari 50 + 50 adalah...', correct:'100', wrong:'90'},
        {q:'Satu jam terdiri dari...', correct:'60 menit', wrong:'100 menit'},
        {q:'Satu minggu terdiri dari...', correct:'7 hari', wrong:'8 hari'},
        {q:'Hasil dari 12 dibagi 4 adalah...', correct:'3', wrong:'4'},

        {q:'Hasil dari 30 - 10 adalah...', correct:'20', wrong:'10'},
        {q:'Hasil dari 6 x 5 adalah...', correct:'30', wrong:'35'},
        {q:'Manakah pernyataan yang benar mengenai bilangan 100 dan 99?', correct:'100 lebih besar dari 99', wrong:'100 lebih kecil dari 99'},
        {q:'Setengah jam sama dengan...', correct:'30 menit', wrong:'45 menit'},
        {q:'Hasil dari 9 + 9 adalah...', correct:'18', wrong:'19'},
      ],
      sedang: [
        {q:'Luas persegi panjang dihitung dengan rumus...', correct:'Panjang x lebar', wrong:'Panjang + lebar'},
        {q:'Pecahan 1/2 sama nilainya dengan...', correct:'0,5', wrong:'0,2'},
        {q:'Sudut siku-siku besarnya...', correct:'90 derajat', wrong:'60 derajat'},
        {q:'Bilangan prima terkecil adalah...', correct:'2', wrong:'1'},
        {q:'Keliling lingkaran dihitung dengan rumus...', correct:'Phi x diameter', wrong:'Phi x jari-jari'},
        {q:'20% dari 200 adalah...', correct:'40', wrong:'20'},
        {q:'Luas segitiga dihitung dengan rumus...', correct:'1/2 x alas x tinggi', wrong:'Alas x tinggi'},
        {q:'Manakah pernyataan yang benar tentang bilangan negatif dan positif?', correct:'Bilangan negatif lebih kecil dari bilangan positif', wrong:'Bilangan negatif lebih besar dari bilangan positif'},
        {q:'FPB dari 12 dan 18 adalah...', correct:'6', wrong:'4'},
        {q:'KPK dari 4 dan 6 adalah...', correct:'12', wrong:'24'},
        {q:'Setiap sudut dalam segitiga sama sisi besarnya...', correct:'60 derajat', wrong:'90 derajat'},
        {q:'Manakah pernyataan yang benar tentang pecahan 3/4 dan 1/2?', correct:'3/4 lebih besar dari 1/2', wrong:'3/4 lebih kecil dari 1/2'},
        {q:'Bilangan bulat mencakup...', correct:'Bilangan negatif, nol, dan positif', wrong:'Hanya bilangan positif'},
        {q:'Rata-rata dari 2, 4, dan 6 adalah...', correct:'4', wrong:'5'},
        {q:'Persegi adalah bangun datar dengan...', correct:'4 sisi sama panjang', wrong:'4 sisi berbeda panjang'},
        {q:'Volume kubus dihitung dengan rumus...', correct:'Sisi x sisi x sisi', wrong:'Sisi x sisi'},
        {q:'3 kuadrat (3 pangkat 2) sama dengan...', correct:'9', wrong:'6'},
        {q:'Dua garis sejajar akan...', correct:'Tidak pernah berpotongan', wrong:'Selalu berpotongan'},
        {q:'Median adalah...', correct:'Nilai tengah dari data yang sudah diurutkan', wrong:'Nilai yang paling sering muncul'},
        {q:'Modus adalah...', correct:'Data yang paling sering muncul', wrong:'Data yang paling jarang muncul'},
        {q:'Skala peta 1:1000 berarti 1 cm pada peta mewakili...', correct:'1000 cm sebenarnya', wrong:'100 cm sebenarnya'},
        {q:'Perbandingan senilai terjadi jika...', correct:'Kedua nilai naik atau turun bersamaan', wrong:'Satu nilai naik, satu nilai turun'},
        {q:'Pecahan campuran terdiri dari...', correct:'Bilangan bulat dan pecahan', wrong:'Dua bilangan pecahan'},
        {q:'Sudut lancip besarnya...', correct:'Kurang dari 90 derajat', wrong:'Lebih dari 90 derajat'},
        {q:'Kedua diagonal pada persegi panjang memiliki...', correct:'Panjang yang sama', wrong:'Panjang yang berbeda'},

        {q:'Rata-rata dari 10, 20, dan 30 adalah...', correct:'20', wrong:'25'},
        {q:'Pada garis bilangan, bilangan bulat negatif terletak di sebelah...', correct:'Kiri nol', wrong:'Kanan nol'},
        {q:'Segitiga siku-siku memiliki satu sudut sebesar...', correct:'90 derajat', wrong:'60 derajat'},
        {q:'Perbandingan 2:4 senilai dengan...', correct:'1:2', wrong:'1:4'},
        {q:'Jajar genjang memiliki...', correct:'Dua pasang sisi sejajar', wrong:'Satu pasang sisi sejajar'},
      ],
      sulit: [
        {q:'Akar kuadrat dari 144 adalah...', correct:'12', wrong:'14'},
        {q:'Persamaan kuadrat...', correct:'Tidak selalu punya dua akar real', wrong:'Selalu punya dua akar real'},
        {q:'Hasil dari 2 pangkat 5 adalah...', correct:'32', wrong:'25'},
        {q:'Turunan dari x kuadrat adalah...', correct:'2x', wrong:'x'},
        {q:'Logaritma dari 100 dengan basis 10 adalah...', correct:'2', wrong:'10'},
        {q:'Jumlah seluruh sudut dalam segitiga selalu...', correct:'180 derajat', wrong:'360 derajat'},
        {q:'Grafik fungsi kuadrat berbentuk...', correct:'Parabola', wrong:'Garis lurus'},
        {q:'Determinan dari matriks identitas adalah...', correct:'1', wrong:'0'},
        {q:'Integral merupakan operasi kebalikan dari...', correct:'Turunan', wrong:'Perkalian'},
        {q:'Limit suatu fungsi...', correct:'Tidak selalu ada di setiap titik', wrong:'Selalu ada di setiap titik'},
        {q:'Nilai sin 90 derajat adalah...', correct:'1', wrong:'0'},
        {q:'Nilai cos 0 derajat adalah...', correct:'1', wrong:'0'},
        {q:'Deret aritmatika memiliki...', correct:'Beda (selisih) yang tetap', wrong:'Rasio yang tetap'},
        {q:'Deret geometri memiliki...', correct:'Rasio yang tetap', wrong:'Beda yang tetap'},
        {q:'Vektor adalah besaran yang memiliki...', correct:'Besar dan arah', wrong:'Besar saja tanpa arah'},
        {q:'Determinan matriks 2x2 [[a,b],[c,d]] dihitung dengan...', correct:'ad dikurangi bc', wrong:'ad ditambah bc'},
        {q:'Nilai peluang suatu kejadian selalu berada di antara...', correct:'0 dan 1', wrong:'0 dan 10'},
        {q:'Bilangan kompleks memiliki...', correct:'Bagian real dan bagian imajiner', wrong:'Hanya bagian real'},
        {q:'Grafik fungsi linear berbentuk...', correct:'Garis lurus', wrong:'Parabola'},
        {q:'Bentuk umum persamaan lingkaran adalah...', correct:'x² + y² = r²', wrong:'x + y = r'},
        {q:'Notasi sigma (Σ) digunakan untuk menyatakan...', correct:'Penjumlahan berurutan', wrong:'Perkalian berurutan'},
        {q:'Nilai bilangan Euler (e) kira-kira sama dengan...', correct:'2,718', wrong:'3,14'},
        {q:'Fungsi eksponen...', correct:'Bisa naik atau menurun, tergantung basisnya', wrong:'Selalu menurun'},
        {q:'Barisan Fibonacci dimulai dari angka...', correct:'0 dan 1', wrong:'1 dan 2'},
        {q:'Trigonometri berlaku untuk...', correct:'Semua jenis segitiga, tidak hanya siku-siku', wrong:'Segitiga siku-siku saja'},

        {q:'Nilai pi (π) mendekati...', correct:'3,14', wrong:'2,71'},
        {q:'Fungsi linear memiliki derajat pangkat tertinggi...', correct:'1', wrong:'2'},
        {q:'Matriks persegi adalah matriks yang memiliki...', correct:'Jumlah baris dan kolom yang sama', wrong:'Jumlah baris dan kolom yang berbeda'},
        {q:'Median dari data berjumlah ganjil adalah...', correct:'Nilai tengah setelah data diurutkan', wrong:'Rata-rata dua nilai tengah'},
        {q:'Bilangan irasional...', correct:'Tidak dapat dinyatakan sebagai pecahan biasa', wrong:'Dapat dinyatakan sebagai pecahan biasa'},
      ]
    },
    bindo: {
      mudah: [
        {q:'Kalimat tanya diakhiri dengan tanda...', correct:'Tanda tanya (?)', wrong:'Tanda seru (!)'},
        {q:'Huruf kapital dipakai di...', correct:'Awal kalimat', wrong:'Tengah kalimat'},
        {q:'Sinonim artinya...', correct:'Persamaan kata', wrong:'Lawan kata'},
        {q:'Pantun terdiri dari bagian...', correct:'Sampiran dan isi', wrong:'Bait dan rima saja'},
        {q:'Kata kerja disebut juga...', correct:'Verba', wrong:'Nomina'},
        {q:'Cerpen adalah cerita yang...', correct:'Pendek', wrong:'Sangat panjang'},
        {q:'Kalimat perintah biasanya diakhiri dengan tanda...', correct:'Tanda seru (!)', wrong:'Tanda tanya (?)'},
        {q:'Jumlah huruf vokal dalam bahasa Indonesia ada...', correct:'5', wrong:'6'},
        {q:'Dongeng termasuk karya sastra...', correct:'Fiksi', wrong:'Nonfiksi'},
        {q:'Kata benda disebut juga...', correct:'Nomina', wrong:'Verba'},
        {q:'Paragraf terdiri dari...', correct:'Beberapa kalimat', wrong:'Satu kata saja'},
        {q:'Puisi biasanya ditulis dalam bentuk...', correct:'Larik dan bait', wrong:'Prosa'},
        {q:'Huruf konsonan adalah...', correct:'Huruf selain huruf vokal', wrong:'Huruf a, i, u, e, o'},
        {q:'Kata sambung (konjungsi) digunakan untuk...', correct:'Menghubungkan kata atau kalimat', wrong:'Mengakhiri kalimat'},
        {q:'Membaca nyaring dilakukan dengan suara yang...', correct:'Jelas dan lantang', wrong:'Sangat pelan'},
        {q:'Tanda titik digunakan di akhir kalimat...', correct:'Berita', wrong:'Tanya'},
        {q:'Fabel adalah cerita yang tokohnya...', correct:'Hewan', wrong:'Manusia'},
        {q:'Kamus digunakan untuk...', correct:'Mencari arti kata', wrong:'Mencari alamat rumah'},
        {q:'Kalimat yang baik harus memiliki minimal...', correct:'Subjek dan predikat', wrong:'Judul dan penutup'},
        {q:'Legenda adalah...', correct:'Cerita rakyat yang dipercaya berkaitan dengan asal-usul suatu tempat', wrong:'Cerita yang sepenuhnya berdasarkan kejadian nyata'},
        {q:'Jumlah huruf abjad dalam bahasa Indonesia ada...', correct:'26', wrong:'24'},
        {q:'Tanda koma digunakan untuk...', correct:'Memisahkan unsur dalam kalimat', wrong:'Mengakhiri sebuah paragraf'},
        {q:'Narasi adalah karangan yang...', correct:'Menceritakan suatu peristiwa', wrong:'Menggambarkan ciri suatu benda'},
        {q:'Kata ulang adalah kata yang diulang, contohnya...', correct:'Rumah-rumah', wrong:'Rumah sakit'},
        {q:'Awalan dan akhiran disebut juga...', correct:'Imbuhan', wrong:'Kata dasar'},

        {q:'Kata tanya "mengapa" digunakan untuk menanyakan...', correct:'Alasan', wrong:'Waktu'},
        {q:'Huruf kapital (huruf besar) digunakan di...', correct:'Awal kalimat atau nama', wrong:'Tengah kalimat saja'},
        {q:'Cerita rakyat termasuk karya sastra...', correct:'Lisan yang diwariskan turun-temurun', wrong:'Tulisan modern karya penulis terkenal'},
        {q:'Tanda seru digunakan untuk kalimat yang menunjukkan...', correct:'Perintah atau seruan', wrong:'Pertanyaan'},
        {q:'Kata ganti "kamu" termasuk kata ganti orang...', correct:'Kedua', wrong:'Ketiga'},
      ],
      sedang: [
        {q:'Antonim artinya...', correct:'Lawan kata', wrong:'Persamaan kata'},
        {q:'Teks eksposisi bertujuan untuk...', correct:'Memberikan informasi atau penjelasan', wrong:'Menghibur pembaca'},
        {q:'Kalimat majemuk terdiri dari...', correct:'Dua klausa atau lebih', wrong:'Satu klausa saja'},
        {q:'Ide pokok biasanya terdapat pada...', correct:'Kalimat utama paragraf', wrong:'Kalimat penjelas'},
        {q:'Majas personifikasi adalah majas yang...', correct:'Membandingkan benda mati seolah hidup', wrong:'Membandingkan dua hal secara langsung'},
        {q:'Teks prosedur berisi...', correct:'Langkah-langkah melakukan sesuatu', wrong:'Opini penulis'},
        {q:'Teks deskripsi berfungsi untuk...', correct:'Menggambarkan sesuatu secara rinci', wrong:'Menceritakan urutan peristiwa'},
        {q:'Kalimat efektif seharusnya ditulis secara...', correct:'Ringkas dan jelas', wrong:'Bertele-tele'},
        {q:'Majas hiperbola menggunakan pernyataan yang...', correct:'Berlebihan', wrong:'Merendahkan diri'},
        {q:'Teks laporan hasil observasi berisi...', correct:'Hasil pengamatan', wrong:'Karangan fiksi'},
        {q:'Kata baku adalah kata yang...', correct:'Sesuai dengan kaidah bahasa yang berlaku', wrong:'Digunakan sehari-hari secara santai'},
        {q:'Paragraf induktif meletakkan ide pokok di...', correct:'Akhir paragraf', wrong:'Awal paragraf'},
        {q:'Dalam tulisan resmi, kata tidak baku...', correct:'Tidak boleh dipakai', wrong:'Boleh dipakai'},
        {q:'Surat resmi menggunakan bahasa...', correct:'Baku', wrong:'Gaul'},
        {q:'Wawancara adalah kegiatan...', correct:'Tanya jawab untuk memperoleh informasi', wrong:'Membaca puisi di depan umum'},
        {q:'Teks negosiasi bertujuan untuk...', correct:'Mencapai kesepakatan', wrong:'Menceritakan pengalaman pribadi'},
        {q:'Majas metafora adalah perbandingan...', correct:'Langsung tanpa kata pembanding', wrong:'Menggunakan kata "seperti" atau "bagai"'},
        {q:'Unsur intrinsik cerita meliputi...', correct:'Tema, tokoh, dan alur', wrong:'Latar belakang penulis saja'},
        {q:'Alur cerita bisa berjalan secara...', correct:'Maju, mundur, atau campuran', wrong:'Hanya maju saja'},
        {q:'Latar cerita meliputi...', correct:'Tempat, waktu, dan suasana', wrong:'Tokoh dan wataknya saja'},
        {q:'Teks berita harus memuat unsur...', correct:'5W + 1H', wrong:'Sampiran dan isi'},
        {q:'Iklan bertujuan untuk...', correct:'Mempersuasi atau membujuk pembaca', wrong:'Memberi kritik terhadap pembaca'},
        {q:'Kalimat langsung ditulis dengan menggunakan tanda...', correct:'Petik', wrong:'Kurung'},
        {q:'Resensi buku berisi...', correct:'Ringkasan disertai penilaian terhadap buku', wrong:'Ringkasan buku tanpa penilaian'},
        {q:'Teks eksplanasi menjelaskan...', correct:'Proses terjadinya suatu fenomena', wrong:'Cara membuat sesuatu'},

        {q:'Kalimat tunggal memiliki...', correct:'Satu subjek dan satu predikat', wrong:'Dua klausa atau lebih'},
        {q:'Teks persuasi bertujuan untuk...', correct:'Meyakinkan atau membujuk pembaca', wrong:'Menggambarkan suasana secara rinci'},
        {q:'Sinonim dan antonim adalah dua istilah yang...', correct:'Berbeda makna (persamaan dan lawan kata)', wrong:'Memiliki makna yang sama'},
        {q:'Paragraf campuran memiliki ide pokok di...', correct:'Awal dan akhir paragraf', wrong:'Tengah paragraf saja'},
        {q:'Kalimat retoris adalah kalimat tanya yang...', correct:'Tidak memerlukan jawaban karena hanya penegasan', wrong:'Selalu memerlukan jawaban pasti'},
      ],
      sulit: [
        {q:'Kalimat efektif seharusnya...', correct:'Ringkas dan tidak bertele-tele', wrong:'Boleh bertele-tele asalkan panjang'},
        {q:'Konjungsi berfungsi untuk...', correct:'Menghubungkan kata atau kalimat', wrong:'Mengganti kata benda'},
        {q:'Teks argumentasi berisi...', correct:'Pendapat yang disertai alasan', wrong:'Langkah-langkah kegiatan'},
        {q:'Kata baku adalah kata yang selalu sesuai dengan...', correct:'KBBI', wrong:'Bahasa gaul sehari-hari'},
        {q:'Paragraf deduktif meletakkan ide pokok di...', correct:'Awal paragraf', wrong:'Akhir paragraf'},
        {q:'Resensi adalah...', correct:'Ulasan atau penilaian terhadap suatu karya', wrong:'Ringkasan tanpa penilaian'},
        {q:'Karya ilmiah harus didukung oleh...', correct:'Data dan fakta', wrong:'Imajinasi penulis'},
        {q:'Majas ironi menyatakan sesuatu yang...', correct:'Bertentangan dengan maksud sebenarnya', wrong:'Sesuai persis dengan maksud sebenarnya'},
        {q:'Kalimat ambigu memiliki makna yang...', correct:'Ganda atau tidak jelas', wrong:'Jelas dan tunggal'},
        {q:'Diksi adalah...', correct:'Pilihan kata dalam sebuah karya tulis', wrong:'Susunan kalimat dalam paragraf'},
        {q:'Sudut pandang orang pertama menggunakan kata ganti...', correct:'Aku atau saya', wrong:'Dia atau mereka'},
        {q:'Teks anekdot bertujuan untuk...', correct:'Mengkritik secara halus melalui humor', wrong:'Menjelaskan proses ilmiah'},
        {q:'Kohesi adalah keterkaitan...', correct:'Bentuk (kata/kalimat) antarkalimat dalam paragraf', wrong:'Makna antarkalimat dalam paragraf'},
        {q:'Koherensi berkaitan dengan...', correct:'Keterpaduan makna dalam paragraf', wrong:'Keterpaduan bentuk kata dalam paragraf'},
        {q:'Esai adalah karangan yang...', correct:'Mengungkapkan pendapat pribadi penulis', wrong:'Berisi data statistik tanpa opini'},
        {q:'Gaya bahasa dan majas merupakan dua istilah yang...', correct:'Berkaitan erat satu sama lain', wrong:'Sama sekali tidak berkaitan'},
        {q:'Karya sastra periode 1920-an disebut angkatan...', correct:'Balai Pustaka', wrong:'Angkatan 66'},
        {q:'Puisi kontemporer umumnya...', correct:'Bebas dari aturan rima dan bait yang ketat', wrong:'Selalu terikat pada rima dan bait'},
        {q:'Analisis wacana mengkaji bahasa dalam...', correct:'Konteks penggunaannya', wrong:'Tata bahasa formal semata'},
        {q:'Teks ulasan film termasuk jenis teks yang...', correct:'Objektif dan evaluatif', wrong:'Subjektif tanpa penilaian'},
        {q:'Frasa adalah gabungan kata yang...', correct:'Tidak membentuk klausa', wrong:'Selalu memiliki subjek dan predikat'},
        {q:'Klausa selalu memiliki...', correct:'Subjek dan predikat', wrong:'Hanya predikat saja'},
        {q:'Denotasi adalah...', correct:'Makna sebenarnya dari suatu kata', wrong:'Makna kias dari suatu kata'},
        {q:'Konotasi adalah...', correct:'Makna kias atau tidak sebenarnya dari suatu kata', wrong:'Makna sebenarnya dari suatu kata'},
        {q:'Majas sinestesia adalah majas yang...', correct:'Menggabungkan dua indera berbeda', wrong:'Membandingkan benda mati seolah hidup'},

        {q:'Anafora adalah gaya bahasa pengulangan kata di...', correct:'Awal beberapa baris atau kalimat', wrong:'Akhir beberapa baris atau kalimat'},
        {q:'Karya sastra dalam penggunaan bahasanya...', correct:'Boleh bebas bereksplorasi, tidak selalu baku', wrong:'Harus selalu mengikuti kaidah bahasa baku'},
        {q:'Teks biografi adalah kisah hidup seseorang yang ditulis oleh...', correct:'Orang lain', wrong:'Diri sendiri'},
        {q:'Autobiografi ditulis oleh...', correct:'Diri sendiri tentang kisah hidupnya', wrong:'Orang lain tentang tokoh tersebut'},
        {q:'Majas litotes digunakan untuk...', correct:'Merendahkan diri walau kenyataannya tidak demikian', wrong:'Membesar-besarkan sesuatu secara berlebihan'},
      ]
    },
    pkn: {
      mudah: [
        {q:'Jumlah sila dalam Pancasila ada...', correct:'5 sila', wrong:'4 sila'},
        {q:'Warna bendera Indonesia adalah...', correct:'Merah putih', wrong:'Merah kuning'},
        {q:'Bahasa persatuan bangsa Indonesia adalah...', correct:'Bahasa Indonesia', wrong:'Bahasa Inggris'},
        {q:'Kepala negara Republik Indonesia adalah...', correct:'Presiden', wrong:'Perdana Menteri'},
        {q:'Sikap yang mencerminkan kebersamaan bangsa Indonesia disebut...', correct:'Gotong royong', wrong:'Individualisme'},
        {q:'Indonesia memproklamasikan kemerdekaan pada tanggal...', correct:'17 Agustus 1945', wrong:'17 Agustus 1950'},
        {q:'Lambang negara Indonesia adalah...', correct:'Garuda Pancasila', wrong:'Burung Cendrawasih'},
        {q:'Lagu kebangsaan Indonesia berjudul...', correct:'Indonesia Raya', wrong:'Bagimu Negeri'},
        {q:'Semboyan negara Indonesia adalah...', correct:'Bhinneka Tunggal Ika', wrong:'Bersatu Kita Teguh'},
        {q:'Jumlah suku bangsa di Indonesia...', correct:'Terdiri dari banyak suku bangsa', wrong:'Hanya satu suku bangsa'},
        {q:'Bunyi sila pertama Pancasila adalah...', correct:'Ketuhanan Yang Maha Esa', wrong:'Kemanusiaan yang Adil dan Beradab'},
        {q:'Tujuan dilakukannya musyawarah adalah untuk mencapai...', correct:'Mufakat', wrong:'Keputusan sepihak'},
        {q:'Sikap yang benar terhadap hak asasi manusia adalah...', correct:'Menghormatinya', wrong:'Mengabaikannya'},
        {q:'Sesuatu yang harus dilaksanakan oleh setiap warga negara disebut...', correct:'Kewajiban', wrong:'Hak'},
        {q:'Sikap warga negara yang baik terhadap hukum adalah...', correct:'Menaati hukum', wrong:'Melanggar hukum'},
        {q:'Pemilihan umum (pemilu) digunakan untuk memilih...', correct:'Presiden dan wakil rakyat', wrong:'Kepala sekolah'},
        {q:'Sikap menghormati perbedaan disebut...', correct:'Toleransi', wrong:'Diskriminasi'},
        {q:'Bentuk negara Indonesia adalah...', correct:'Negara kesatuan', wrong:'Negara federasi'},
        {q:'Sikap terhadap aturan yang benar adalah...', correct:'Mematuhi aturan meskipun tidak diawasi', wrong:'Melanggar aturan asal tidak ketahuan'},
        {q:'Manfaat kerja sama dalam menyelesaikan pekerjaan adalah...', correct:'Membuat pekerjaan lebih ringan', wrong:'Membuat pekerjaan lebih berat'},
        {q:'Rumah adat merupakan salah satu contoh dari...', correct:'Keberagaman budaya', wrong:'Keberagaman agama'},
        {q:'Tujuan dilaksanakannya upacara bendera adalah untuk...', correct:'Menghormati jasa para pahlawan', wrong:'Mengisi waktu luang'},
        {q:'Hak anak dalam menyampaikan pendapat...', correct:'Tetap dimiliki dan harus dihormati', wrong:'Tidak dimiliki sama sekali'},
        {q:'Menghormati orang tua termasuk sikap...', correct:'Terpuji', wrong:'Tercela'},
        {q:'Aturan yang mengatur kehidupan bermasyarakat disebut...', correct:'Norma', wrong:'Dongeng'},
        {q:'Pendidikan bagi setiap warga negara Indonesia merupakan...', correct:'Hak', wrong:'Larangan'},
        {q:'Simbol sila kedua Pancasila adalah...', correct:'Rantai', wrong:'Bintang'},
        {q:'Jumlah provinsi di Indonesia saat ini...', correct:'Lebih dari 30 provinsi', wrong:'Hanya 10 provinsi'},
        {q:'Pemberlakuan hukum di Indonesia berlaku untuk...', correct:'Seluruh warga negara tanpa kecuali', wrong:'Hanya sebagian warga negara saja'},
        {q:'Cinta tanah air merupakan salah satu...', correct:'Nilai kebangsaan yang penting', wrong:'Sikap yang tidak perlu ditanamkan'},
      ],
      sedang: [
        {q:'Dasar hukum tertinggi di Indonesia adalah...', correct:'UUD 1945', wrong:'Peraturan Daerah'},
        {q:'Sistem pemerintahan yang dianut Indonesia adalah...', correct:'Republik (presidensial)', wrong:'Monarki'},
        {q:'Lembaga yang bertugas membuat undang-undang bersama pemerintah adalah...', correct:'DPR', wrong:'Mahkamah Agung'},
        {q:'Kedudukan warga negara di mata hukum adalah...', correct:'Sama, memiliki hak dan kewajiban setara', wrong:'Berbeda sesuai status sosial'},
        {q:'Arti dari semboyan Bhinneka Tunggal Ika adalah...', correct:'Berbeda-beda tetap satu jua', wrong:'Satu untuk semua, semua untuk satu'},
        {q:'Pemilihan umum di Indonesia diadakan setiap...', correct:'5 tahun sekali', wrong:'10 tahun sekali'},
        {q:'Demokrasi Pancasila lebih mengutamakan...', correct:'Musyawarah untuk mufakat', wrong:'Pemungutan suara tanpa musyawarah'},
        {q:'Lembaga yang bertugas menjalankan pemerintahan disebut lembaga...', correct:'Eksekutif', wrong:'Yudikatif'},
        {q:'Lembaga yang bertugas membuat undang-undang disebut lembaga...', correct:'Legislatif', wrong:'Eksekutif'},
        {q:'Otonomi daerah memberikan kewenangan kepada daerah untuk...', correct:'Mengatur wilayahnya sendiri', wrong:'Melepaskan diri dari NKRI'},
        {q:'Kewajiban warga negara terhadap negara dalam hal keuangan adalah...', correct:'Membayar pajak sesuai ketentuan', wrong:'Menghindari pajak semaksimal mungkin'},
        {q:'Kedaulatan rakyat berarti kekuasaan tertinggi berada di tangan...', correct:'Rakyat', wrong:'Presiden'},
        {q:'Peran partai politik dalam proses demokrasi adalah...', correct:'Sangat penting sebagai sarana partisipasi politik', wrong:'Tidak memiliki peran sama sekali'},
        {q:'Wawasan Nusantara memandang Indonesia sebagai...', correct:'Satu kesatuan wilayah', wrong:'Kumpulan wilayah yang terpisah-pisah'},
        {q:'Pengaruh globalisasi terhadap kehidupan berbangsa...', correct:'Berpengaruh besar, baik positif maupun negatif', wrong:'Sama sekali tidak berpengaruh'},
        {q:'Contoh hak politik warga negara adalah...', correct:'Hak memilih dan dipilih', wrong:'Hak memperoleh warisan'},
        {q:'Konstitusi berfungsi untuk mengatur...', correct:'Struktur dan mekanisme ketatanegaraan', wrong:'Harga barang di pasar'},
        {q:'Keputusan dalam musyawarah seharusnya diambil melalui...', correct:'Kesepakatan bersama', wrong:'Paksaan salah satu pihak'},
        {q:'Integrasi nasional berfungsi untuk menjaga...', correct:'Persatuan bangsa', wrong:'Perpecahan antarsuku'},
        {q:'Sistem multipartai berarti dalam suatu negara terdapat...', correct:'Lebih dari satu partai politik', wrong:'Hanya satu partai politik'},
        {q:'Hak warga negara asing dibandingkan warga negara Indonesia...', correct:'Tidak sama persis, ada pembatasan tertentu', wrong:'Sama persis tanpa pembatasan'},
        {q:'Nilai-nilai Pancasila seharusnya...', correct:'Diterapkan dalam kehidupan sehari-hari', wrong:'Hanya dihafalkan tanpa diamalkan'},
        {q:'Hubungan pemerintah pusat dan daerah dalam membangun negara sebaiknya...', correct:'Bekerja sama', wrong:'Berjalan sendiri-sendiri'},
        {q:'Pemberlakuan hukum bagi warga negara seharusnya...', correct:'Sama tanpa kecuali', wrong:'Berbeda tergantung jabatan'},
        {q:'Perlindungan hak asasi manusia menjadi tanggung jawab...', correct:'Negara', wrong:'Individu semata'},
        {q:'Pemilihan presiden di Indonesia dilakukan secara...', correct:'Langsung oleh rakyat', wrong:'Tidak langsung melalui MPR'},
        {q:'Peraturan daerah terhadap UUD 1945 seharusnya...', correct:'Tidak boleh bertentangan', wrong:'Boleh bertentangan'},
        {q:'Wakil rakyat di DPR dipilih melalui...', correct:'Pemilu legislatif', wrong:'Penunjukan presiden'},
        {q:'Sistem pemerintahan Indonesia menerapkan prinsip...', correct:'Pemisahan kekuasaan', wrong:'Pemusatan seluruh kekuasaan pada satu lembaga'},
        {q:'Hak berserikat dan berkumpul bagi warga negara...', correct:'Dijamin oleh konstitusi', wrong:'Dilarang oleh konstitusi'},
      ],
      sulit: [
        {q:'Lembaga yang berwenang mengubah dan menetapkan UUD adalah...', correct:'MPR', wrong:'DPR'},
        {q:'Sistem pemerintahan yang dianut Indonesia adalah...', correct:'Presidensial', wrong:'Parlementer'},
        {q:'Otonomi daerah memberikan hak kepada daerah untuk...', correct:'Mengatur urusannya sendiri', wrong:'Tidak boleh mengatur urusannya sendiri'},
        {q:'Lembaga yang bertugas mengadili pelanggaran hukum disebut lembaga...', correct:'Yudikatif', wrong:'Legislatif'},
        {q:'Pelanggaran hak asasi manusia...', correct:'Tidak dapat dibenarkan dengan alasan apa pun', wrong:'Boleh dilakukan dengan alasan tertentu'},
        {q:'Prinsip checks and balances berfungsi untuk...', correct:'Menjaga keseimbangan antar lembaga negara', wrong:'Memusatkan kekuasaan pada satu lembaga'},
        {q:'Konsep trias politica membagi kekuasaan negara menjadi...', correct:'Eksekutif, legislatif, dan yudikatif', wrong:'Pusat, daerah, dan provinsi'},
        {q:'Perubahan konstitusi di Indonesia dilakukan melalui...', correct:'Mekanisme amandemen oleh MPR', wrong:'Keputusan presiden seorang diri'},
        {q:'Lembaga yang berwenang menguji undang-undang terhadap UUD adalah...', correct:'Mahkamah Konstitusi', wrong:'Mahkamah Agung'},
        {q:'Politik luar negeri Indonesia menganut prinsip...', correct:'Bebas aktif', wrong:'Memihak salah satu blok kekuatan'},
        {q:'Penyerahan wewenang dari pemerintah pusat ke daerah disebut...', correct:'Desentralisasi', wrong:'Sentralisasi'},
        {q:'Kewajiban warga negara terhadap pertahanan negara...', correct:'Tetap ada dan harus dilaksanakan', wrong:'Tidak ada sama sekali'},
        {q:'Asas yang dianut sistem hukum Indonesia terhadap tersangka adalah...', correct:'Praduga tak bersalah', wrong:'Praduga bersalah'},
        {q:'UUD 1945 telah mengalami amandemen sebanyak...', correct:'Empat kali', wrong:'Dua kali'},
        {q:'Ciri ideologi terbuka adalah dapat berkembang mengikuti zaman...', correct:'Tanpa mengubah nilai dasarnya', wrong:'Dengan mengganti nilai dasarnya'},
        {q:'Dalam konsep negara hukum, kedudukan hukum terhadap kekuasaan adalah...', correct:'Hukum berada di atas kekuasaan', wrong:'Kekuasaan berada di atas hukum'},
        {q:'Sifat hak asasi manusia adalah...', correct:'Universal, berlaku untuk semua orang', wrong:'Hanya berlaku bagi golongan tertentu'},
        {q:'Prinsip good governance menekankan pentingnya...', correct:'Transparansi dan akuntabilitas', wrong:'Kerahasiaan dan sentralisasi kekuasaan'},
        {q:'Politik identitas yang berlebihan dapat...', correct:'Mengancam persatuan bangsa', wrong:'Selalu memperkuat persatuan bangsa'},
        {q:'Kebiasaan dalam praktik bernegara yang tidak tertulis disebut...', correct:'Konvensi ketatanegaraan', wrong:'Undang-undang'},
        {q:'Ciri utama sistem pemilu proporsional adalah...', correct:'Kursi dibagi sesuai perolehan suara partai secara proporsional', wrong:'Pemenang tunggal mengambil semua kursi di daerah pemilihan'},
        {q:'Kedaulatan negara Indonesia dalam pergaulan internasional...', correct:'Tetap tunduk pada hukum internasional yang disepakati', wrong:'Bersifat mutlak tanpa batas hukum internasional'},
        {q:'Kewajiban bela negara menjadi tanggung jawab...', correct:'Seluruh warga negara', wrong:'Hanya anggota TNI'},
        {q:'Reformasi tahun 1998 mendorong perubahan Indonesia menuju sistem yang...', correct:'Lebih demokratis', wrong:'Lebih otoriter'},
        {q:'Wawasan kebangsaan berperan penting untuk menjaga...', correct:'Keutuhan NKRI', wrong:'Kepentingan kelompok tertentu'},
        {q:'Proses pemberhentian pejabat negara melalui mekanisme konstitusional disebut...', correct:'Impeachment', wrong:'Referendum'},
        {q:'Susunan peraturan perundang-undangan di Indonesia bersifat...', correct:'Berhierarki dari tertinggi ke terendah', wrong:'Sejajar tanpa tingkatan'},
        {q:'Bentuk partisipasi politik rakyat secara langsung untuk memutuskan suatu hal disebut...', correct:'Referendum', wrong:'Delegasi'},
        {q:'Sifat konstitusi Indonesia terhadap kemungkinan perubahan...', correct:'Dapat diamandemen melalui mekanisme tertentu', wrong:'Kaku dan tidak dapat diubah sama sekali'},
        {q:'Ciri sistem pemerintahan parlementer adalah...', correct:'Kepala negara dan kepala pemerintahan dipisahkan secara tegas', wrong:'Kepala negara dan kepala pemerintahan dijabat orang yang sama'},
      ]
    },
    ipa: {
      mudah: [
        {q:'Matahari terbit dari arah...', correct:'Timur', wrong:'Barat'},
        {q:'Zat yang sangat dibutuhkan tumbuhan untuk hidup adalah...', correct:'Air', wrong:'Pasir'},
        {q:'Ikan bernapas menggunakan...', correct:'Insang', wrong:'Paru-paru'},
        {q:'Air membeku pada suhu...', correct:'0 derajat Celsius', wrong:'100 derajat Celsius'},
        {q:'Jumlah mata yang dimiliki manusia adalah...', correct:'2', wrong:'4'},
        {q:'Bumi berevolusi dengan mengelilingi...', correct:'Matahari', wrong:'Bulan'},
        {q:'Proses tumbuhan hijau membuat makanan sendiri disebut...', correct:'Fotosintesis', wrong:'Respirasi'},
        {q:'Manakah yang termasuk makhluk hidup?', correct:'Kucing', wrong:'Batu'},
        {q:'Cara berkembang biak kucing adalah...', correct:'Melahirkan', wrong:'Bertelur'},
        {q:'Alat pernapasan manusia adalah...', correct:'Paru-paru', wrong:'Insang'},
        {q:'Air, tanah, dan udara termasuk...', correct:'Sumber daya alam', wrong:'Sumber daya buatan'},
        {q:'Jika dipanaskan, besi akan...', correct:'Memuai', wrong:'Menyusut'},
        {q:'Bulan tampak bercahaya karena...', correct:'Memantulkan cahaya matahari', wrong:'Menghasilkan cahaya sendiri'},
        {q:'Hewan yang makan tumbuhan disebut...', correct:'Herbivora', wrong:'Omnivora'},
        {q:'Fungsi rangka pada tubuh manusia adalah...', correct:'Menopang dan membentuk tubuh', wrong:'Memompa darah'},
        {q:'Rasa air laut adalah...', correct:'Asin', wrong:'Tawar'},
        {q:'Tumbuhan bernapas melalui lubang kecil pada daun yang disebut...', correct:'Stomata', wrong:'Kelopak'},
        {q:'Tahap pertama daur hidup kupu-kupu adalah...', correct:'Telur', wrong:'Kepompong'},
        {q:'Besi dapat berkarat jika terkena...', correct:'Air dan udara', wrong:'Minyak'},
        {q:'Benda padat dapat mencair jika...', correct:'Dipanaskan', wrong:'Didinginkan'},
        {q:'Magnet dapat menarik benda yang terbuat dari...', correct:'Besi', wrong:'Plastik'},
        {q:'Bentuk bumi mendekati...', correct:'Bola', wrong:'Kotak'},
        {q:'Harimau dan singa termasuk hewan...', correct:'Karnivora', wrong:'Herbivora'},
        {q:'Energi yang digunakan tumbuhan untuk fotosintesis berasal dari...', correct:'Cahaya matahari', wrong:'Angin'},
        {q:'Udara termasuk benda berwujud...', correct:'Gas', wrong:'Padat'},

        {q:'Hewan yang hidup di air disebut hewan...', correct:'Akuatik', wrong:'Terestrial'},
        {q:'Agar manusia dapat melihat benda, diperlukan...', correct:'Cahaya', wrong:'Suara'},
        {q:'Fungsi akar pada tumbuhan adalah...', correct:'Menyerap air dan mineral dari tanah', wrong:'Menghasilkan biji'},
        {q:'Pada ukuran yang sama, benda yang lebih berat adalah...', correct:'Besi', wrong:'Kapas'},
        {q:'Satelit alami Bumi adalah...', correct:'Bulan', wrong:'Mars'},
      ],
      sedang: [
        {q:'Fotosintesis pada tumbuhan terjadi terutama di bagian...', correct:'Daun', wrong:'Akar'},
        {q:'Organ yang memompa darah ke seluruh tubuh adalah...', correct:'Jantung', wrong:'Hati'},
        {q:'Gaya yang menarik benda ke arah bumi disebut...', correct:'Gaya gravitasi', wrong:'Gaya magnet'},
        {q:'Rantai makanan menggambarkan...', correct:'Aliran energi antar makhluk hidup', wrong:'Perpindahan tempat tinggal hewan'},
        {q:'Logam bersifat sebagai penghantar panas yang...', correct:'Baik', wrong:'Buruk'},
        {q:'Gas yang dibutuhkan manusia untuk bernapas adalah...', correct:'Oksigen', wrong:'Karbon dioksida'},
        {q:'Komponen penyusun ekosistem terdiri dari...', correct:'Komponen biotik dan abiotik', wrong:'Komponen biotik saja'},
        {q:'Pembuluh darah yang membawa darah dari jantung ke seluruh tubuh disebut...', correct:'Arteri', wrong:'Vena'},
        {q:'Gas sisa pernapasan yang dihasilkan manusia adalah...', correct:'Karbon dioksida', wrong:'Oksigen'},
        {q:'Bahan baku fotosintesis adalah...', correct:'Air dan karbon dioksida', wrong:'Oksigen dan glukosa'},
        {q:'Gerhana matahari terjadi ketika...', correct:'Bulan berada di antara Bumi dan Matahari', wrong:'Bumi berada di antara Matahari dan Bulan'},
        {q:'Perubahan wujud dari cair menjadi gas disebut...', correct:'Menguap', wrong:'Membeku'},
        {q:'Simbiosis mutualisme adalah hubungan yang...', correct:'Menguntungkan kedua belah pihak', wrong:'Merugikan salah satu pihak'},
        {q:'Zat tunggal terdiri dari...', correct:'Unsur dan senyawa', wrong:'Campuran homogen dan heterogen'},
        {q:'Campuran yang komponennya tidak dapat dibedakan lagi disebut...', correct:'Campuran homogen', wrong:'Campuran heterogen'},
        {q:'Gaya gesek pada benda yang bergerak cenderung...', correct:'Memperlambat gerak benda', wrong:'Mempercepat gerak benda'},
        {q:'Sistem pencernaan manusia dimulai dari...', correct:'Mulut', wrong:'Lambung'},
        {q:'Pertukaran oksigen dan karbon dioksida di paru-paru terjadi di...', correct:'Alveolus', wrong:'Kerongkongan'},
        {q:'Tekanan udara di dataran tinggi dibandingkan dataran rendah adalah...', correct:'Lebih kecil', wrong:'Lebih besar'},
        {q:'Pelapukan batuan dapat disebabkan oleh...', correct:'Air dan perubahan suhu', wrong:'Bunyi dan magnet'},
        {q:'Kipas angin mengubah energi listrik menjadi energi...', correct:'Gerak', wrong:'Kimia'},
        {q:'Listrik statis dapat timbul akibat...', correct:'Gesekan dua benda', wrong:'Pemanasan air'},
        {q:'Pusat sistem saraf manusia adalah...', correct:'Otak dan sumsum tulang belakang', wrong:'Jantung dan paru-paru'},
        {q:'Medium yang tidak dapat dilalui bunyi adalah...', correct:'Ruang hampa udara', wrong:'Air'},
        {q:'Fungsi ginjal adalah...', correct:'Menyaring darah dan membentuk urine', wrong:'Memompa darah ke seluruh tubuh'},

        {q:'Rangkaian listrik seri memiliki...', correct:'Satu jalur arus listrik', wrong:'Lebih dari satu jalur arus listrik'},
        {q:'Jika satu lampu dilepas, lampu lainnya tetap menyala pada rangkaian...', correct:'Paralel', wrong:'Seri'},
        {q:'Alat yang digunakan untuk mengukur suhu adalah...', correct:'Termometer', wrong:'Neraca'},
        {q:'Titik didih setiap zat...', correct:'Berbeda-beda', wrong:'Sama untuk semua zat'},
        {q:'Pengikisan tanah oleh air atau angin disebut...', correct:'Erosi', wrong:'Sedimentasi'},
      ],
      sulit: [
        {q:'Tempat berlangsungnya respirasi sel adalah...', correct:'Mitokondria', wrong:'Ribosom'},
        {q:'Hukum I Newton membahas tentang...', correct:'Kelembaman benda', wrong:'Percepatan benda'},
        {q:'Partikel penyusun atom adalah...', correct:'Proton, neutron, dan elektron', wrong:'Proton, neutron, dan molekul'},
        {q:'Reaksi yang melepaskan kalor ke lingkungan disebut...', correct:'Reaksi eksoterm', wrong:'Reaksi endoterm'},
        {q:'Molekul penyimpan informasi genetik makhluk hidup adalah...', correct:'DNA', wrong:'ATP'},
        {q:'Di dataran tinggi, titik didih air cenderung...', correct:'Lebih rendah', wrong:'Lebih tinggi'},
        {q:'Rumus Hukum II Newton adalah...', correct:'F = m x a', wrong:'F = m / a'},
        {q:'Pada reaksi endoterm, suhu campuran reaksi cenderung...', correct:'Menurun', wrong:'Meningkat'},
        {q:'Hasil fotosintesis adalah...', correct:'Glukosa dan oksigen', wrong:'Karbon dioksida dan air'},
        {q:'Struktur yang dimiliki sel tumbuhan tetapi tidak dimiliki sel hewan adalah...', correct:'Dinding sel', wrong:'Inti sel'},
        {q:'Atom dengan jumlah proton sama tetapi jumlah neutron berbeda disebut...', correct:'Isotop', wrong:'Isobar'},
        {q:'Hukum kekekalan energi menyatakan bahwa energi...', correct:'Tidak dapat diciptakan maupun dimusnahkan', wrong:'Dapat diciptakan dari ketiadaan'},
        {q:'Ikatan kovalen terbentuk melalui...', correct:'Pemakaian bersama pasangan elektron', wrong:'Serah terima elektron'},
        {q:'Salah satu penyebab mutasi genetik adalah...', correct:'Paparan radiasi', wrong:'Olahraga teratur'},
        {q:'Gelombang elektromagnetik dapat merambat...', correct:'Tanpa memerlukan medium', wrong:'Hanya melalui udara'},
        {q:'Reaksi redoks melibatkan...', correct:'Transfer elektron', wrong:'Transfer proton'},
        {q:'Menurut hukum kedua termodinamika, entropi alam semesta cenderung...', correct:'Meningkat', wrong:'Menurun'},
        {q:'Fungsi katalis dalam reaksi kimia adalah...', correct:'Mempercepat laju reaksi', wrong:'Menambah jumlah hasil reaksi'},
        {q:'Sel darah merah berfungsi...', correct:'Mengangkut oksigen ke seluruh tubuh', wrong:'Melawan kuman penyakit'},
        {q:'Hormon insulin berfungsi...', correct:'Menurunkan kadar gula darah', wrong:'Menaikkan kadar gula darah'},
        {q:'Gempa tektonik disebabkan oleh...', correct:'Pergerakan lempeng tektonik', wrong:'Aktivitas magma gunung api'},
        {q:'Cahaya tampak merupakan...', correct:'Sebagian kecil dari spektrum elektromagnetik', wrong:'Seluruh spektrum elektromagnetik'},
        {q:'Sistem periodik unsur modern disusun berdasarkan...', correct:'Nomor atom', wrong:'Massa atom'},
        {q:'Kemampuan tubuh menjaga keseimbangan lingkungan internal disebut...', correct:'Homeostasis', wrong:'Metabolisme'},
        {q:'Pemancaran partikel atau energi dari inti atom yang tidak stabil disebut...', correct:'Radioaktivitas', wrong:'Elektrolisis'},

        {q:'Fotosintesis berlangsung optimal pada saat...', correct:'Siang hari ketika ada cahaya', wrong:'Malam hari'},
        {q:'Protein yang mempercepat reaksi biokimia dalam tubuh disebut...', correct:'Enzim', wrong:'Hormon'},
        {q:'Tokoh yang meletakkan dasar ilmu pewarisan sifat (genetika) adalah...', correct:'Gregor Mendel', wrong:'Charles Darwin'},
        {q:'Sel prokariotik tidak memiliki...', correct:'Membran inti sel yang jelas', wrong:'Materi genetik'},
        {q:'Arah perpindahan zat pada peristiwa difusi adalah...', correct:'Dari konsentrasi tinggi ke rendah', wrong:'Dari konsentrasi rendah ke tinggi'},
      ]
    },
    binggris: {
      mudah: [
        {q:'"Cat" artinya...', correct:'Kucing', wrong:'Anjing'},
        {q:'"Book" artinya...', correct:'Buku', wrong:'Meja'},
        {q:'"Good morning" digunakan untuk menyapa pada waktu...', correct:'Pagi hari', wrong:'Malam hari'},
        {q:'Bentuk to be yang benar untuk subjek "I" adalah...', correct:'"I am"', wrong:'"I is"'},
        {q:'"Red" adalah nama...', correct:'Warna (merah)', wrong:'Angka'},
        {q:'"Dog" artinya...', correct:'Anjing', wrong:'Kucing'},
        {q:'"One, two, three" adalah urutan angka...', correct:'1, 2, 3', wrong:'4, 5, 6'},
        {q:'"Monday" adalah nama...', correct:'Hari (Senin)', wrong:'Bulan'},
        {q:'"Apple" artinya...', correct:'Apel', wrong:'Jeruk'},
        {q:'"Thank you" digunakan untuk mengucapkan...', correct:'Terima kasih', wrong:'Permintaan maaf'},
        {q:'"Big" adalah lawan kata dari...', correct:'"Small"', wrong:'"Fast"'},
        {q:'"Sun" artinya...', correct:'Matahari', wrong:'Bulan'},
        {q:'"Water" artinya...', correct:'Air', wrong:'Api'},
        {q:'"Family" berarti...', correct:'Keluarga', wrong:'Teman'},
        {q:'"Happy" berarti...', correct:'Senang', wrong:'Sedih'},
        {q:'"School" artinya...', correct:'Sekolah', wrong:'Rumah sakit'},
        {q:'"Blue" adalah nama warna...', correct:'Biru', wrong:'Hijau'},
        {q:'"Car" artinya...', correct:'Mobil', wrong:'Sepeda'},
        {q:'"Night" artinya...', correct:'Malam', wrong:'Siang'},
        {q:'"Please" digunakan untuk...', correct:'Meminta sesuatu dengan sopan', wrong:'Mengucapkan selamat tinggal'},
        {q:'"Teacher" artinya...', correct:'Guru', wrong:'Murid'},
        {q:'"Ten" adalah angka...', correct:'Sepuluh', wrong:'Delapan'},
        {q:'"Cold" adalah lawan kata dari...', correct:'"Hot"', wrong:'"Warm"'},
        {q:'"Bird" artinya...', correct:'Burung', wrong:'Ikan'},
        {q:'"Yes" dan "no" adalah kata untuk menjawab...', correct:'Iya dan tidak', wrong:'Sebelum dan sesudah'},

        {q:'"Fish" artinya...', correct:'Ikan', wrong:'Burung'},
        {q:'"Milk" artinya...', correct:'Susu', wrong:'Air'},
        {q:'"Small" adalah lawan kata dari...', correct:'"Large"', wrong:'"Tall"'},
        {q:'"Green" artinya warna...', correct:'Hijau', wrong:'Merah'},
        {q:'"House" artinya...', correct:'Rumah', wrong:'Sekolah'},
      ],
      sedang: [
        {q:'Kata kerja bentuk ketiga (past participle) dari "go" adalah...', correct:'"Gone"', wrong:'"Goed"'},
        {q:'Bentuk kalimat yang benar secara gramatikal adalah...', correct:'"She goes to school every day"', wrong:'"She go to school every day"'},
        {q:'Kata "yesterday" biasa digunakan dalam kalimat...', correct:'Simple past tense', wrong:'Simple future tense'},
        {q:'Kata sifat (adjective) berfungsi untuk menerangkan...', correct:'Kata benda', wrong:'Kata kerja'},
        {q:'"Will" digunakan untuk menyatakan kejadian di masa...', correct:'Depan', wrong:'Lalu'},
        {q:'"Because" adalah kata penghubung yang menyatakan...', correct:'Sebab akibat', wrong:'Pertentangan'},
        {q:'Rumus present continuous tense adalah...', correct:'To be + verb-ing', wrong:'Have/has + verb 3'},
        {q:'"Do" dan "does" digunakan dalam kalimat tanya bentuk...', correct:'Simple present tense', wrong:'Simple past tense'},
        {q:'Kata benda jamak dalam bahasa Inggris...', correct:'Memiliki beberapa bentuk tidak beraturan (irregular)', wrong:'Selalu ditambah huruf s tanpa pengecualian'},
        {q:'"There is" digunakan untuk benda...', correct:'Tunggal', wrong:'Jamak'},
        {q:'"There are" digunakan untuk benda...', correct:'Jamak', wrong:'Tunggal'},
        {q:'Kata keterangan (adverb) berfungsi menerangkan...', correct:'Kata kerja', wrong:'Kata benda'},
        {q:'"Must" digunakan untuk menyatakan...', correct:'Keharusan yang kuat', wrong:'Kemungkinan yang lemah'},
        {q:'Simple past tense digunakan untuk kejadian yang...', correct:'Sudah terjadi di masa lalu', wrong:'Sedang berlangsung sekarang'},
        {q:'"Can" digunakan untuk menyatakan...', correct:'Kemampuan', wrong:'Keharusan'},
        {q:'Kalimat tanya dalam bahasa Inggris biasanya diawali dengan...', correct:'Kata bantu (auxiliary) atau kata tanya', wrong:'Kata kerja utama'},
        {q:'"A" dan "an" adalah kata sandang...', correct:'Tak tentu', wrong:'Tentu'},
        {q:'"The" adalah kata sandang...', correct:'Tertentu', wrong:'Tak tentu'},
        {q:'Kata ganti "they" digunakan untuk orang...', correct:'Ketiga jamak', wrong:'Kedua tunggal'},
        {q:'"Never" berarti...', correct:'Tidak pernah', wrong:'Selalu'},
        {q:'Comparative degree digunakan untuk membandingkan...', correct:'Dua hal', wrong:'Lebih dari dua hal'},
        {q:'Superlative degree digunakan untuk membandingkan...', correct:'Lebih dari dua hal', wrong:'Dua hal'},
        {q:'"Although" dan "but" memiliki fungsi yang sama sebagai kata hubung...', correct:'Pertentangan', wrong:'Sebab akibat'},
        {q:'Preposition of time seperti "at, on, in" digunakan untuk menunjukkan...', correct:'Waktu', wrong:'Tempat'},
        {q:'Kalimat perintah (imperative) biasanya diawali dengan...', correct:'Kata kerja, tanpa subjek', wrong:'Subjek terlebih dahulu'},

        {q:'"Whose" digunakan untuk menanyakan...', correct:'Kepemilikan', wrong:'Waktu'},
        {q:'Simple future tense biasanya menggunakan...', correct:'"Will" atau "going to"', wrong:'"Have" atau "has"'},
        {q:'Kata "enough" biasanya diletakkan...', correct:'Setelah kata sifat, misalnya "big enough"', wrong:'Sebelum kata sifat, misalnya "enough big"'},
        {q:'Kata "much" digunakan untuk kata benda yang...', correct:'Tidak bisa dihitung (uncountable)', wrong:'Bisa dihitung (countable)'},
        {q:'"Neither...nor" digunakan untuk menyatakan...', correct:'Dua pilihan yang sama-sama tidak berlaku', wrong:'Dua pilihan yang sama-sama berlaku'},
      ],
      sulit: [
        {q:'Rumus present perfect tense adalah...', correct:'Have/has + verb 3', wrong:'Will + verb 1'},
        {q:'Dalam kalimat pasif (passive voice), subjeknya...', correct:'Dikenai aksi (menerima tindakan)', wrong:'Melakukan aksi'},
        {q:'"Although" digunakan untuk menyatakan...', correct:'Pertentangan', wrong:'Sebab akibat'},
        {q:'Conditional sentence type 2 membahas...', correct:'Pengandaian yang tidak nyata pada masa sekarang', wrong:'Kejadian yang pasti terjadi'},
        {q:'"Idiom" adalah ungkapan yang maknanya...', correct:'Tidak selalu harfiah', wrong:'Selalu harfiah'},
        {q:'Reported speech berfungsi untuk mengubah kalimat...', correct:'Langsung menjadi tidak langsung', wrong:'Aktif menjadi pasif'},
        {q:'Past perfect tense menunjukkan kejadian yang...', correct:'Terjadi sebelum kejadian lain di masa lalu', wrong:'Sedang terjadi saat ini'},
        {q:'Gerund adalah kata kerja berakhiran -ing yang berfungsi sebagai...', correct:'Kata benda', wrong:'Kata sifat'},
        {q:'Modal verbs seperti should, must, can diikuti oleh...', correct:'Kata kerja bentuk dasar (verb 1)', wrong:'Kata kerja bentuk ketiga (verb 3)'},
        {q:'Relative clause menggunakan kata penghubung seperti...', correct:'"Who, which, that"', wrong:'"And, but, or"'},
        {q:'Conditional sentence type 3 membahas...', correct:'Penyesalan terhadap masa lalu', wrong:'Kejadian yang mungkin terjadi di masa depan'},
        {q:'Direct speech dan indirect speech memiliki struktur yang...', correct:'Berbeda', wrong:'Sama persis'},
        {q:'Causative verbs seperti "have" dan "get" digunakan untuk...', correct:'Menyuruh orang lain melakukan sesuatu', wrong:'Menyatakan kejadian di masa depan'},
        {q:'Subjunctive mood digunakan untuk menyatakan...', correct:'Harapan atau situasi hipotetis', wrong:'Fakta yang sudah pasti terjadi'},
        {q:'Phrasal verb terdiri dari kata kerja dan...', correct:'Partikel (kata depan/adverb)', wrong:'Kata benda'},
        {q:'Future perfect tense digunakan untuk kejadian yang...', correct:'Akan selesai sebelum waktu tertentu di masa depan', wrong:'Belum pasti terjadi'},
        {q:'Parallel structure penting dalam...', correct:'Penulisan kalimat yang baik', wrong:'Pengucapan kata yang benar'},
        {q:'Ellipsis dalam gramatika berarti...', correct:'Penghilangan bagian kalimat yang sudah jelas dari konteks', wrong:'Penambahan kata yang berulang'},
        {q:'Inversion digunakan...', correct:'Hanya pada situasi tertentu, tidak di setiap kalimat', wrong:'Selalu digunakan dalam setiap kalimat'},
        {q:'Tag question digunakan untuk...', correct:'Meminta konfirmasi', wrong:'Menyatakan perintah'},
        {q:'Non-defining relative clause biasanya diapit oleh tanda...', correct:'Koma', wrong:'Kurung'},
        {q:'Discourse markers berfungsi untuk...', correct:'Menghubungkan ide dalam sebuah teks', wrong:'Mengubah kalimat aktif menjadi pasif'},
        {q:'Collocation adalah...', correct:'Kombinasi kata yang biasa digunakan bersama', wrong:'Kata yang memiliki arti berlawanan'},
        {q:'Register dalam bahasa mengacu pada...', correct:'Tingkat formalitas bahasa yang digunakan', wrong:'Jumlah kosakata yang dikuasai'},
        {q:'Cohesion dan coherence merupakan dua istilah yang...', correct:'Berbeda maknanya', wrong:'Identik tanpa perbedaan'},

        {q:'Mixed conditional menggabungkan...', correct:'Dua jenis waktu berbeda dalam satu kalimat pengandaian', wrong:'Dua subjek berbeda dalam satu kalimat'},
        {q:'Passive voice dalam present perfect tense...', correct:'Dapat digunakan', wrong:'Tidak dapat digunakan'},
        {q:'Cleft sentence digunakan untuk...', correct:'Memberi penekanan pada bagian tertentu dari kalimat', wrong:'Menyingkat kalimat menjadi lebih pendek'},
        {q:'Ellipsis dalam bahasa Inggris berarti...', correct:'Penghilangan kata yang sudah jelas dari konteks', wrong:'Penambahan kata yang berulang'},
        {q:'Hedging digunakan untuk...', correct:'Melunakkan pernyataan agar tidak terlalu tegas', wrong:'Mempertegas pernyataan secara mutlak'},
      ]
    },
    agama_islam: {
      mudah: [
        {q:'Jumlah rukun Islam adalah...', correct:'5', wrong:'6'},
        {q:'Jumlah shalat wajib dalam sehari semalam adalah...', correct:'5 waktu', wrong:'3 waktu'},
        {q:'Kitab suci umat Islam adalah...', correct:'Al-Quran', wrong:'Taurat'},
        {q:'Puasa Ramadhan dilaksanakan selama...', correct:'Satu bulan penuh', wrong:'Sepanjang tahun'},
        {q:'Nabi dan rasul terakhir adalah...', correct:'Nabi Muhammad SAW', wrong:'Nabi Isa AS'},
        {q:'Rukun Islam yang ketiga adalah...', correct:'Zakat', wrong:'Puasa'},
        {q:'Saat shalat, umat Islam menghadap ke...', correct:'Arah kiblat', wrong:'Arah matahari'},
        {q:'Bacaan yang dianjurkan sebelum memulai suatu pekerjaan adalah...', correct:'Basmalah', wrong:'Hamdalah'},
        {q:'Panggilan tanda masuknya waktu shalat disebut...', correct:'Adzan', wrong:'Khutbah'},
        {q:'Kabah terletak di kota...', correct:'Mekkah', wrong:'Madinah'},
        {q:'Bulan puasa wajib bagi umat Islam disebut bulan...', correct:'Ramadhan', wrong:'Muharram'},
        {q:'Hari raya yang dirayakan setelah bulan puasa adalah...', correct:'Idul Fitri', wrong:'Idul Adha'},
        {q:'Sifat yang terpuji dalam Islam adalah...', correct:'Jujur', wrong:'Berbohong'},
        {q:'Bersuci dari hadas kecil dengan air disebut...', correct:'Wudhu', wrong:'Tayamum'},
        {q:'Membantu orang lain yang kesulitan termasuk perbuatan...', correct:'Terpuji', wrong:'Tercela'},
        {q:'Al-Quran diturunkan kepada...', correct:'Nabi Muhammad SAW', wrong:'Nabi Daud AS'},
        {q:'Shalat Jumat dilaksanakan secara...', correct:'Berjamaah di masjid', wrong:'Sendirian di rumah'},
        {q:'Belajar membaca Al-Quran sebaiknya dimulai...', correct:'Sejak kecil', wrong:'Setelah dewasa saja'},
        {q:'Berdoa dilakukan untuk memohon kepada...', correct:'Allah SWT', wrong:'Malaikat'},
        {q:'Menyayangi sesama makhluk termasuk...', correct:'Akhlak mulia', wrong:'Akhlak tercela'},
        {q:'Rukun Islam yang pertama adalah...', correct:'Syahadat', wrong:'Shalat'},
        {q:'Jumlah rakaat shalat Subuh adalah...', correct:'2 rakaat', wrong:'4 rakaat'},
        {q:'Hari raya kurban dikenal juga sebagai...', correct:'Idul Adha', wrong:'Idul Fitri'},
        {q:'Perbuatan yang dilarang dalam Islam adalah...', correct:'Mencuri', wrong:'Bersedekah'},
        {q:'Tempat ibadah umat Islam disebut...', correct:'Masjid', wrong:'Madrasah'},

        {q:'Surah yang wajib dibaca pada setiap rakaat shalat adalah...', correct:'Al-Fatihah', wrong:'Al-Ikhlas'},
        {q:'Malaikat yang bertugas mencatat amal manusia adalah...', correct:'Raqib dan Atid', wrong:'Munkar dan Nakir'},
        {q:'Selain menahan lapar dan haus, orang berpuasa juga harus menjaga diri dari...', correct:'Perkataan dan perbuatan buruk', wrong:'Membaca Al-Quran'},
        {q:'Cara terbaik memuliakan Al-Quran adalah...', correct:'Membaca, memahami, dan mengamalkannya', wrong:'Hanya menyimpannya di lemari'},
        {q:'Dua jenis zakat dalam Islam adalah...', correct:'Zakat fitrah dan zakat mal', wrong:'Zakat fitrah dan zakat haji'},
      ],
      sedang: [
        {q:'Jumlah rukun iman adalah...', correct:'6', wrong:'5'},
        {q:'Hukum berwudhu sebelum melaksanakan shalat adalah...', correct:'Wajib (syarat sah shalat)', wrong:'Mubah (boleh ditinggalkan)'},
        {q:'Haji wajib dilaksanakan oleh muslim yang...', correct:'Mampu secara fisik dan harta', wrong:'Berusia di atas 60 tahun'},
        {q:'Sedekah dapat diberikan pada...', correct:'Waktu kapan saja', wrong:'Bulan Ramadhan saja'},
        {q:'Asmaul Husna adalah...', correct:'Nama-nama baik Allah SWT', wrong:'Nama-nama para nabi'},
        {q:'Sifat jujur dan amanah termasuk akhlak...', correct:'Mahmudah (terpuji)', wrong:'Madzmumah (tercela)'},
        {q:'Malaikat yang bertugas menyampaikan wahyu kepada para nabi adalah...', correct:'Jibril', wrong:'Mikail'},
        {q:'Malaikat yang bertugas meniup sangkakala pada hari kiamat adalah...', correct:'Israfil', wrong:'Izrail'},
        {q:'Zakat fitrah dikeluarkan pada waktu...', correct:'Menjelang Idul Fitri', wrong:'Menjelang Idul Adha'},
        {q:'Pahala shalat berjamaah dibandingkan shalat sendirian adalah...', correct:'Lebih utama', wrong:'Lebih rendah'},
        {q:'Jumlah juz dalam Al-Quran adalah...', correct:'30', wrong:'24'},
        {q:'Nabi yang dikenal sebagai bapak para nabi adalah...', correct:'Nabi Ibrahim AS', wrong:'Nabi Nuh AS'},
        {q:'Malaikat penjaga pintu surga adalah...', correct:'Ridwan', wrong:'Malik'},
        {q:'Tayamum dilakukan sebagai pengganti wudhu apabila...', correct:'Tidak ada air atau tidak dapat memakai air', wrong:'Air tersedia sangat berlebihan'},
        {q:'Iman kepada kitab-kitab Allah merupakan rukun iman yang...', correct:'Ketiga', wrong:'Kelima'},
        {q:'Ketetapan Allah SWT atas segala sesuatu yang terjadi pada makhluk disebut...', correct:'Takdir', wrong:'Ikhtiar'},
        {q:'Perbuatan yang jika dikerjakan berpahala dan jika ditinggalkan tidak berdosa disebut...', correct:'Sunnah', wrong:'Wajib'},
        {q:'Tambahan yang diharamkan dalam transaksi utang piutang disebut...', correct:'Riba', wrong:'Infak'},
        {q:'Melakukan amal semata-mata karena Allah SWT disebut...', correct:'Ikhlas', wrong:'Riya'},
        {q:'Membicarakan keburukan orang lain di belakangnya disebut...', correct:'Ghibah', wrong:'Tabayyun'},
        {q:'Shalat sunnah malam hari yang khusus dikerjakan pada bulan Ramadhan adalah...', correct:'Tarawih', wrong:'Dhuha'},
        {q:'Nabi Muhammad SAW lahir di kota...', correct:'Mekkah', wrong:'Thaif'},
        {q:'Hijrah adalah perpindahan Nabi Muhammad SAW dari...', correct:'Mekkah ke Madinah', wrong:'Madinah ke Mekkah'},
        {q:'Orang yang berpura-pura beriman padahal hatinya ingkar disebut...', correct:'Munafik', wrong:'Mukmin'},
        {q:'Menuntut ilmu hukumnya...', correct:'Wajib bagi setiap muslim', wrong:'Hanya wajib bagi ulama'},

        {q:'Shalat Id dilaksanakan pada hari raya...', correct:'Idul Fitri dan Idul Adha', wrong:'Isra Miraj dan Maulid Nabi'},
        {q:'Tawaf dilakukan dengan mengelilingi Kabah sebanyak...', correct:'7 kali', wrong:'5 kali'},
        {q:'Kitab Taurat diturunkan kepada...', correct:'Nabi Musa AS', wrong:'Nabi Isa AS'},
        {q:'Contoh puasa sunnah adalah puasa...', correct:'Senin dan Kamis', wrong:'Ramadhan'},
        {q:'Setelah selesai shalat, umat Islam dianjurkan untuk...', correct:'Berdzikir dan berdoa', wrong:'Segera tidur'},
      ],
      sulit: [
        {q:'Kesepakatan para ulama dalam menetapkan suatu hukum disebut...', correct:'Ijma', wrong:'Qiyas'},
        {q:'Menetapkan hukum perkara baru dengan menyamakannya pada perkara lama yang sudah ada hukumnya disebut...', correct:'Qiyas', wrong:'Ijma'},
        {q:'Hadits adalah segala perkataan, perbuatan, dan ketetapan dari...', correct:'Nabi Muhammad SAW', wrong:'Para ulama'},
        {q:'Ilmu yang membahas hukum tata cara ibadah dan muamalah disebut...', correct:'Fikih', wrong:'Ilmu falak'},
        {q:'Ilmu yang mempelajari cara membaca Al-Quran dengan benar disebut...', correct:'Tajwid', wrong:'Tafsir'},
        {q:'Muamalah adalah aturan Islam yang mengatur hubungan...', correct:'Antar manusia', wrong:'Manusia dengan Allah SWT saja'},
        {q:'Ilmu tentang dasar dan kaidah penetapan hukum Islam disebut...', correct:'Ushul fikih', wrong:'Ilmu kalam'},
        {q:'Hadits yang sanad dan matannya terpercaya disebut hadits...', correct:'Shahih', wrong:'Dhaif'},
        {q:'Usaha sungguh-sungguh ulama menetapkan hukum yang tidak ada dalil jelasnya disebut...', correct:'Ijtihad', wrong:'Taklid'},
        {q:'Penghapusan suatu hukum oleh hukum lain yang datang kemudian disebut...', correct:'Nasakh', wrong:'Takhsis'},
        {q:'Ilmu yang membahas keyakinan dasar dalam Islam disebut...', correct:'Aqidah', wrong:'Muamalah'},
        {q:'Aspek batin dan penyucian jiwa dalam Islam dibahas dalam ilmu...', correct:'Tasawuf', wrong:'Fikih'},
        {q:'Berdasarkan tingkat kualitasnya, hadits terbagi menjadi...', correct:'Shahih, hasan, dan dhaif', wrong:'Shahih dan mutawatir saja'},
        {q:'Tujuan-tujuan pokok disyariatkannya hukum Islam disebut...', correct:'Maqashid syariah', wrong:'Ijtihad'},
        {q:'Perbedaan pendapat di kalangan ulama dalam masalah fikih disebut...', correct:'Khilafiyah', wrong:'Ijma'},
        {q:'Dua sumber utama hukum Islam adalah...', correct:'Al-Quran dan hadits', wrong:'Ijma dan qiyas'},
        {q:'Mazhab fikih yang paling banyak dianut umat Islam di Indonesia adalah...', correct:'Mazhab Syafii', wrong:'Mazhab Hanbali'},
        {q:'Zakat mal wajib dikeluarkan jika harta telah mencapai...', correct:'Nisab dan haul', wrong:'Ijab dan kabul'},
        {q:'Menahan harta pokok untuk dimanfaatkan hasilnya bagi kepentingan umum disebut...', correct:'Wakaf', wrong:'Hibah'},
        {q:'Bagian fikih yang membahas pernikahan disebut...', correct:'Munakahat', wrong:'Jinayat'},
        {q:'Bagian fikih yang membahas hukum pidana Islam disebut...', correct:'Jinayat', wrong:'Faraidh'},
        {q:'Dalil yang bersumber dari Al-Quran dan hadits disebut dalil...', correct:'Naqli', wrong:'Aqli'},
        {q:'Metode ijtihad yang mengutamakan pertimbangan yang lebih baik dan maslahat adalah...', correct:'Istihsan', wrong:'Taklid'},
        {q:'Pendapat hukum dari ulama atau lembaga berwenang atas suatu persoalan disebut...', correct:'Fatwa', wrong:'Khutbah'},
        {q:'Lima hukum perbuatan manusia dalam Islam adalah...', correct:'Wajib, sunnah, mubah, makruh, haram', wrong:'Wajib, sunnah, mubah, haram, batil'},

        {q:'Hadits yang tidak memenuhi syarat hadits shahih maupun hasan disebut hadits...', correct:'Dhaif', wrong:'Shahih'},
        {q:'Hadits yang diriwayatkan oleh banyak perawi pada setiap tingkatan sehingga mustahil berdusta disebut...', correct:'Mutawatir', wrong:'Ahad'},
        {q:'Kajian tentang sejarah kehidupan Nabi Muhammad SAW disebut...', correct:'Sirah nabawiyah', wrong:'Ilmu hadits'},
        {q:'Menurut istilah fikih, bid\'ah berarti...', correct:'Sesuatu yang baru dalam urusan agama', wrong:'Perbuatan yang wajib dikerjakan'},
        {q:'Ilmu yang bertujuan menjelaskan makna ayat-ayat Al-Quran disebut...', correct:'Tafsir', wrong:'Qiraat'},
      ]
    },
    ips: {
      mudah: [
        {q:'Indonesia terletak di benua...', correct:'Asia', wrong:'Eropa'},
        {q:'Kota Jakarta merupakan ibu kota dari negara...', correct:'Indonesia', wrong:'Malaysia'},
        {q:'Wilayah Indonesia terdiri dari...', correct:'Ribuan pulau', wrong:'Satu pulau saja'},
        {q:'Tempat bertemunya penjual dan pembeli untuk jual beli barang disebut...', correct:'Pasar', wrong:'Museum'},
        {q:'Gunung Everest terletak di pegunungan...', correct:'Himalaya', wrong:'Jayawijaya'},
        {q:'Koperasi adalah usaha bersama yang berasaskan...', correct:'Kekeluargaan', wrong:'Persaingan bebas'},
        {q:'Garis khayal yang membagi bumi menjadi belahan utara dan selatan disebut...', correct:'Khatulistiwa', wrong:'Meridian utama'},
        {q:'Indonesia beriklim tropis dengan dua musim, yaitu...', correct:'Hujan dan kemarau', wrong:'Semi dan gugur'},
        {q:'Alat tukar yang sah dalam kegiatan jual beli adalah...', correct:'Uang', wrong:'Kartu pelajar'},
        {q:'Orang yang bekerja mengolah lahan untuk menanam padi atau sayuran disebut...', correct:'Petani', wrong:'Nelayan'},
        {q:'Orang yang bekerja menangkap ikan di laut disebut...', correct:'Nelayan', wrong:'Peternak'},
        {q:'Gambaran permukaan bumi pada bidang datar yang menunjukkan lokasi suatu wilayah disebut...', correct:'Peta', wrong:'Grafik'},
        {q:'Jumlah bahasa daerah di Indonesia adalah...', correct:'Ratusan bahasa daerah', wrong:'Hanya satu bahasa daerah'},
        {q:'Tiga kegiatan pokok ekonomi adalah...', correct:'Produksi, distribusi, dan konsumsi', wrong:'Produksi, promosi, dan rekreasi'},
        {q:'Candi Borobudur terletak di provinsi...', correct:'Jawa Tengah', wrong:'Bali'},
        {q:'Suku Batak berasal dari provinsi...', correct:'Sumatera Utara', wrong:'Sulawesi Selatan'},
        {q:'Kerja sama warga untuk menyelesaikan pekerjaan bersama disebut...', correct:'Gotong royong', wrong:'Persaingan'},
        {q:'Indonesia disebut sebagai negara...', correct:'Kepulauan', wrong:'Daratan luas'},
        {q:'Air sungai dapat dimanfaatkan dalam bidang pertanian untuk...', correct:'Pengairan sawah (irigasi)', wrong:'Pembakaran lahan'},
        {q:'Budaya antardaerah di Indonesia bersifat...', correct:'Beragam', wrong:'Sama persis'},
        {q:'Pesawat terbang termasuk jenis transportasi...', correct:'Udara', wrong:'Laut'},
        {q:'Lembaga yang menyimpan dan meminjamkan uang disebut...', correct:'Bank', wrong:'Terminal'},
        {q:'Sumber daya alam sebaiknya dimanfaatkan dengan cara...', correct:'Bijak dan dijaga kelestariannya', wrong:'Dihabiskan sebanyak-banyaknya'},
        {q:'Sikap yang tepat terhadap peninggalan sejarah adalah...', correct:'Merawat dan melestarikannya', wrong:'Mencoret-coretnya'},
        {q:'Kerja sama antarwilayah dapat...', correct:'Mempererat persatuan', wrong:'Memicu perpecahan'},
        {q:'Batik adalah warisan budaya Indonesia berupa...', correct:'Kain bermotif dari lilin (malam)', wrong:'Alat musik tradisional'},
        {q:'Provinsi Bali terletak di pulau...', correct:'Bali', wrong:'Jawa'},
        {q:'Ekspor adalah kegiatan...', correct:'Menjual barang ke luar negeri', wrong:'Membeli barang dari luar negeri'},
        {q:'Impor adalah kegiatan...', correct:'Membeli barang dari luar negeri', wrong:'Membeli barang dari dalam negeri saja'},
        {q:'Tempat menyimpan dan memamerkan benda bersejarah adalah...', correct:'Museum', wrong:'Kantor pos'},
      ],
      sedang: [
        {q:'Indonesia terletak di antara dua benua, yaitu...', correct:'Asia dan Australia', wrong:'Afrika dan Eropa'},
        {q:'Kenaikan harga barang secara terus-menerus disebut...', correct:'Inflasi', wrong:'Deflasi'},
        {q:'Perpindahan penduduk dari desa ke kota disebut...', correct:'Urbanisasi', wrong:'Transmigrasi'},
        {q:'Berdasarkan kemampuan pulihnya, sumber daya alam dibagi menjadi...', correct:'Dapat diperbarui dan tidak dapat diperbarui', wrong:'Terbuka dan tertutup'},
        {q:'Dampak positif globalisasi di bidang informasi adalah...', correct:'Pertukaran informasi menjadi lebih mudah', wrong:'Pertukaran informasi menjadi terhambat'},
        {q:'Kegiatan menghasilkan barang dan jasa disebut...', correct:'Produksi', wrong:'Konsumsi'},
        {q:'Indonesia berada pada pertemuan tiga lempeng tektonik, yaitu...', correct:'Eurasia, Indo-Australia, dan Pasifik', wrong:'Eurasia, Afrika, dan Antartika'},
        {q:'Harga barang di pasar dipengaruhi oleh...', correct:'Permintaan dan penawaran', wrong:'Warna dan bentuk barang saja'},
        {q:'Hubungan timbal balik antara individu dengan individu atau kelompok disebut...', correct:'Interaksi sosial', wrong:'Mobilitas sosial'},
        {q:'Perpindahan status sosial seseorang di masyarakat disebut...', correct:'Mobilitas sosial', wrong:'Interaksi sosial'},
        {q:'Revolusi hijau bertujuan...', correct:'Meningkatkan produksi pertanian', wrong:'Meningkatkan produksi industri berat'},
        {q:'Penjajahan Belanda di Indonesia sering disebut berlangsung selama...', correct:'3,5 abad', wrong:'35 tahun'},
        {q:'Perdagangan bebas ditandai dengan...', correct:'Hambatan tarif yang minim', wrong:'Tarif yang sangat tinggi'},
        {q:'Lembaga sosial di masyarakat, selain keluarga, meliputi...', correct:'Lembaga pendidikan, agama, dan ekonomi', wrong:'Hanya lembaga keluarga'},
        {q:'Konflik sosial dapat terjadi karena...', correct:'Perbedaan kepentingan', wrong:'Kerja sama yang erat'},
        {q:'Kepadatan penduduk yang tinggi berdampak pada...', correct:'Berkurangnya ketersediaan lahan', wrong:'Bertambahnya lahan kosong'},
        {q:'Perubahan sosial yang berlangsung lambat dalam waktu lama disebut...', correct:'Evolusi', wrong:'Revolusi'},
        {q:'Sistem ekonomi yang dianut Indonesia adalah...', correct:'Ekonomi Pancasila', wrong:'Ekonomi komando'},
        {q:'Perdagangan antarpulau termasuk kegiatan...', correct:'Distribusi', wrong:'Konsumsi'},
        {q:'Iklim tropis Indonesia dipengaruhi oleh letak...', correct:'Astronomis', wrong:'Geologis'},
        {q:'Industrialisasi dapat mendorong...', correct:'Pertumbuhan ekonomi', wrong:'Penurunan lapangan kerja'},
        {q:'Peta yang menggambarkan ketinggian permukaan bumi disebut peta...', correct:'Topografi', wrong:'Administratif'},
        {q:'ASEAN beranggotakan negara-negara di kawasan...', correct:'Asia Tenggara', wrong:'Eropa Barat'},
        {q:'Dampak perubahan sosial bagi masyarakat dapat berupa...', correct:'Positif maupun negatif', wrong:'Selalu negatif'},
        {q:'Pemerataan distribusi pendapatan dapat...', correct:'Mengurangi kesenjangan sosial', wrong:'Memperlebar kesenjangan sosial'},
        {q:'Bonus demografi terjadi ketika...', correct:'Usia produktif lebih banyak dari usia nonproduktif', wrong:'Usia nonproduktif lebih banyak dari usia produktif'},
        {q:'Mata uang yang digunakan dalam perdagangan antarnegara adalah...', correct:'Berbagai mata uang dengan nilai tukar', wrong:'Satu mata uang untuk seluruh dunia'},
        {q:'Peta persebaran penduduk menunjukkan...', correct:'Kepadatan penduduk suatu wilayah', wrong:'Ketinggian permukaan tanah'},
        {q:'Urbanisasi yang tinggi menyebabkan kepadatan penduduk meningkat di daerah...', correct:'Perkotaan', wrong:'Perdesaan'},
        {q:'Sumber daya manusia yang berkualitas berperan dalam...', correct:'Mendukung pembangunan negara', wrong:'Menghambat pembangunan negara'},
      ],
      sulit: [
        {q:'Letak astronomis Indonesia berada di antara...', correct:'6° LU–11° LS dan 95° BT–141° BT', wrong:'11° LU–6° LS dan 95° BT–141° BT'},
        {q:'Praktik penguasaan wilayah oleh suatu bangsa terhadap bangsa lain disebut...', correct:'Kolonialisme', wrong:'Nasionalisme'},
        {q:'APBN adalah singkatan dari...', correct:'Anggaran Pendapatan dan Belanja Negara', wrong:'Anggaran Perusahaan dan Bisnis Nasional'},
        {q:'Interaksi sosial dapat terjadi antara...', correct:'Individu, kelompok, maupun individu dengan kelompok', wrong:'Individu dengan individu saja'},
        {q:'David Ricardo mengemukakan teori perdagangan internasional yang disebut keunggulan...', correct:'Komparatif', wrong:'Absolut'},
        {q:'Dampak Revolusi Industri bagi perekonomian dunia adalah...', correct:'Produksi barang secara massal dengan mesin', wrong:'Produksi hanya dengan tenaga manual'},
        {q:'Thomas Malthus membahas hubungan antara pertumbuhan penduduk dan...', correct:'Persediaan pangan', wrong:'Teknologi informasi'},
        {q:'Kebijakan moneter dikendalikan oleh...', correct:'Bank sentral', wrong:'Kementerian Perdagangan'},
        {q:'Kebijakan fiskal berkaitan dengan pengaturan...', correct:'Pajak dan belanja negara', wrong:'Suku bunga dan jumlah uang beredar'},
        {q:'Pasar monopoli dikuasai oleh...', correct:'Satu penjual', wrong:'Banyak penjual'},
        {q:'Produk domestik bruto (PDB) mengukur...', correct:'Nilai total produksi barang dan jasa suatu negara', wrong:'Jumlah seluruh utang negara'},
        {q:'Teori dependensi menjelaskan ketergantungan...', correct:'Negara berkembang pada negara maju', wrong:'Negara maju pada negara berkembang'},
        {q:'Perang Dunia II berakhir pada tahun...', correct:'1945', wrong:'1918'},
        {q:'Konferensi Asia Afrika tahun 1955 diselenggarakan di kota...', correct:'Bandung', wrong:'Jakarta'},
        {q:'Zona ekonomi eksklusif Indonesia diukur dari garis pangkal sejauh...', correct:'200 mil laut', wrong:'12 mil laut'},
        {q:'Pelapisan sosial (stratifikasi sosial) dapat ditemukan pada...', correct:'Hampir semua masyarakat, termasuk masyarakat modern', wrong:'Hanya masyarakat tradisional'},
        {q:'Deforestasi (penggundulan hutan) dapat menyebabkan...', correct:'Perubahan iklim dan bencana banjir', wrong:'Bertambahnya penyerapan karbon'},
        {q:'Pembangunan berkelanjutan mempertimbangkan aspek...', correct:'Ekonomi, sosial, dan lingkungan', wrong:'Ekonomi saja'},
        {q:'Perdagangan bilateral melibatkan...', correct:'Dua negara', wrong:'Lebih dari dua negara'},
        {q:'Penyelesaian konflik antarnegara secara damai dapat dilakukan melalui...', correct:'Diplomasi', wrong:'Perang terbuka'},
        {q:'Migrasi internasional dapat memengaruhi...', correct:'Struktur demografi suatu negara', wrong:'Letak astronomis suatu negara'},
        {q:'Kebijakan proteksionisme bertujuan...', correct:'Melindungi produk dalam negeri dengan membatasi impor', wrong:'Membebaskan semua barang impor tanpa bea'},
        {q:'Teori pusat pertumbuhan (Perroux) menjelaskan...', correct:'Konsentrasi kegiatan ekonomi pada wilayah tertentu', wrong:'Penyebaran kegiatan ekonomi sama rata di semua wilayah'},
        {q:'Perang Dingin terjadi antara blok...', correct:'Barat (AS) dan Timur (Uni Soviet)', wrong:'Sekutu dan Poros'},
        {q:'Reformasi agraria bertujuan...', correct:'Mendistribusikan lahan secara lebih adil', wrong:'Memusatkan lahan pada segelintir pemilik'},
        {q:'Bonus demografi menjadi peluang apabila...', correct:'SDM berkualitas dan lapangan kerja tersedia', wrong:'Banyak penduduk usia produktif menganggur'},
        {q:'Teori lokasi industri (Alfred Weber) mempertimbangkan faktor...', correct:'Biaya transportasi dan jarak', wrong:'Warna dan desain produk'},
        {q:'Kerja sama ekonomi regional seperti ASEAN bertujuan...', correct:'Meningkatkan kesejahteraan negara anggota', wrong:'Menguasai wilayah negara anggota'},
        {q:'Krisis moneter 1997–1998 melanda...', correct:'Banyak negara di Asia, termasuk Indonesia', wrong:'Hanya satu negara di Asia'},
        {q:'Pembangunan berkelanjutan memperhatikan kebutuhan...', correct:'Generasi sekarang dan masa depan', wrong:'Generasi sekarang saja'},
      ]
    },
    informatika: {
      mudah: [
        {q:'Komputer adalah alat elektronik yang berfungsi untuk...', correct:'Mengolah data menjadi informasi', wrong:'Menghasilkan listrik'},
        {q:'Perangkat yang digunakan untuk menggerakkan kursor (pointer) di layar adalah...', correct:'Mouse', wrong:'Keyboard'},
        {q:'Perangkat yang digunakan untuk mengetik huruf dan angka adalah...', correct:'Keyboard', wrong:'Monitor'},
        {q:'Jaringan komputer yang saling terhubung di seluruh dunia disebut...', correct:'Internet', wrong:'Intranet'},
        {q:'Kumpulan data yang disimpan dengan nama tertentu disebut...', correct:'File', wrong:'Folder'},
        {q:'Tempat untuk menyimpan dan mengelompokkan file disebut...', correct:'Folder', wrong:'Ikon'},
        {q:'Perangkat yang berfungsi menampilkan gambar dan tulisan dari komputer adalah...', correct:'Monitor', wrong:'Printer'},
        {q:'Perangkat yang digunakan untuk mencetak dokumen ke kertas adalah...', correct:'Printer', wrong:'Scanner'},
        {q:'Bagian komputer yang berfungsi sebagai otak untuk memproses data adalah...', correct:'CPU', wrong:'Monitor'},
        {q:'Fungsi password pada sebuah akun adalah...', correct:'Mengamankan akun dari orang lain', wrong:'Mempercepat koneksi internet'},
        {q:'Sikap yang tepat terhadap password milik kita adalah...', correct:'Merahasiakannya dari orang lain', wrong:'Membagikannya ke teman'},
        {q:'Layanan yang digunakan untuk mengirim surat elektronik disebut...', correct:'Email', wrong:'Browser'},
        {q:'Program yang digunakan untuk menjalankan tugas tertentu di komputer disebut...', correct:'Aplikasi', wrong:'Hardware'},
        {q:'Teknologi untuk menghubungkan perangkat ke internet tanpa kabel adalah...', correct:'Wifi', wrong:'Kabel LAN'},
        {q:'Sikap yang tepat terhadap informasi yang kita temukan di internet adalah...', correct:'Memeriksa kebenarannya terlebih dahulu', wrong:'Langsung mempercayai semuanya'},
        {q:'Salah satu penyebab komputer terkena virus adalah...', correct:'Membuka file yang tidak aman', wrong:'Membersihkan layar monitor'},
        {q:'Perangkat penyimpanan data yang mudah dibawa ke mana-mana adalah...', correct:'Flashdisk', wrong:'Hard disk internal'},
        {q:'Layar yang dapat dioperasikan dengan sentuhan jari disebut...', correct:'Layar sentuh (touchscreen)', wrong:'Layar proyektor'},
        {q:'Kata sandi yang kuat sebaiknya berisi...', correct:'Kombinasi huruf, angka, dan simbol', wrong:'Tanggal lahir sendiri'},
        {q:'Perangkat lunak dalam bahasa Inggris disebut...', correct:'Software', wrong:'Hardware'},
        {q:'Perangkat keras dalam bahasa Inggris disebut...', correct:'Hardware', wrong:'Software'},
        {q:'Aplikasi yang digunakan untuk membuka halaman web disebut...', correct:'Browser', wrong:'Pengolah kata'},
        {q:'Data pribadi seperti alamat dan nomor telepon sebaiknya...', correct:'Tidak dibagikan sembarangan di internet', wrong:'Dibagikan bebas di internet'},
        {q:'Sumber daya pada laptop yang harus diisi ulang ketika habis adalah...', correct:'Baterai', wrong:'Prosesor'},
        {q:'Simbol kecil yang mewakili program atau file di layar komputer disebut...', correct:'Ikon', wrong:'Kursor'},

        {q:'Pintasan keyboard untuk menyalin (copy) teks atau file yang dipilih adalah...', correct:'Ctrl + C', wrong:'Ctrl + V'},
        {q:'Fungsi tombol Restart pada komputer adalah...', correct:'Menyalakan ulang komputer', wrong:'Menghapus semua file'},
        {q:'Ukuran setiap file di komputer pada umumnya...', correct:'Berbeda-beda sesuai isinya', wrong:'Selalu sama'},
        {q:'Port yang umum dipakai untuk menghubungkan flashdisk, mouse, dan keyboard adalah...', correct:'USB', wrong:'HDMI'},
        {q:'Update sistem penting dilakukan untuk...', correct:'Menutup celah keamanan perangkat', wrong:'Memperbesar ukuran layar'},
      ],
      sedang: [
        {q:'Langkah-langkah logis untuk menyelesaikan suatu masalah disebut...', correct:'Algoritma', wrong:'Database'},
        {q:'Bahasa yang digunakan untuk menulis instruksi bagi komputer disebut...', correct:'Bahasa pemrograman', wrong:'Bahasa isyarat'},
        {q:'Satuan terkecil dalam data digital adalah...', correct:'Bit', wrong:'Byte'},
        {q:'1 byte terdiri dari...', correct:'8 bit', wrong:'4 bit'},
        {q:'Diagram yang menggambarkan alur suatu proses atau algoritma disebut...', correct:'Flowchart', wrong:'Diagram batang'},
        {q:'Perangkat lunak yang mengatur kerja perangkat keras dan perangkat lunak komputer adalah...', correct:'Sistem operasi', wrong:'Browser'},
        {q:'Tempat untuk menyimpan dan mengelola data secara terstruktur disebut...', correct:'Basis data (database)', wrong:'Kompiler'},
        {q:'Bahasa yang digunakan untuk membuat struktur halaman web adalah...', correct:'HTML', wrong:'SQL'},
        {q:'Wadah bernama untuk menyimpan nilai yang dapat berubah dalam program disebut...', correct:'Variabel', wrong:'Konstanta'},
        {q:'Struktur untuk mengulang suatu proses beberapa kali dalam program disebut...', correct:'Perulangan (loop)', wrong:'Percabangan (if)'},
        {q:'Kesalahan pada sebuah program disebut...', correct:'Bug', wrong:'Patch'},
        {q:'Jaringan komputer yang mencakup area terbatas, seperti satu gedung, disebut...', correct:'LAN', wrong:'WAN'},
        {q:'Teknik untuk mengamankan data agar tidak mudah dibaca pihak lain disebut...', correct:'Enkripsi', wrong:'Kompresi'},
        {q:'Teknologi penyimpanan data pada server jarak jauh melalui internet disebut...', correct:'Komputasi awan (cloud computing)', wrong:'Penyimpanan lokal'},
        {q:'Teknologi yang meniru kemampuan berpikir manusia disebut...', correct:'Kecerdasan buatan (AI)', wrong:'Realitas virtual (VR)'},
        {q:'Data yang dikirim melalui internet tanpa enkripsi berisiko...', correct:'Mudah dibaca pihak lain', wrong:'Otomatis aman dari peretas'},
        {q:'Proses mencari dan memperbaiki kesalahan pada program disebut...', correct:'Debugging', wrong:'Compiling'},
        {q:'Sistem bilangan yang hanya menggunakan digit 0 dan 1 disebut...', correct:'Biner', wrong:'Desimal'},
        {q:'Sistem yang melindungi jaringan komputer dari serangan luar adalah...', correct:'Firewall', wrong:'Router'},
        {q:'Perintah untuk mengambil data dari basis data disebut...', correct:'Query', wrong:'Backup'},
        {q:'Arsitektur jaringan yang setiap komputernya berkedudukan setara disebut...', correct:'Peer-to-peer', wrong:'Client-server'},
        {q:'Perangkat lunak yang dirancang untuk merusak sistem komputer disebut...', correct:'Malware', wrong:'Freeware'},
        {q:'Program yang mengubah kode program menjadi bahasa mesin disebut...', correct:'Compiler', wrong:'Debugger'},
        {q:'Protokol yang mengamankan komunikasi antara browser dan situs web adalah...', correct:'HTTPS', wrong:'HTTP'},
        {q:'Struktur data yang menyimpan kumpulan nilai dalam satu variabel berindeks disebut...', correct:'Array', wrong:'Loop'},

        {q:'Aplikasi yang digunakan untuk mengolah data dalam bentuk tabel dan rumus adalah...', correct:'Spreadsheet', wrong:'Pengolah presentasi'},
        {q:'Penyimpanan sementara yang mempercepat akses data yang sering digunakan disebut...', correct:'Cache', wrong:'ROM'},
        {q:'Aplikasi yang dibuat untuk satu sistem operasi tertentu biasanya...', correct:'Perlu penyesuaian agar berjalan di sistem operasi lain', wrong:'Otomatis berjalan di semua sistem operasi'},
        {q:'Sistem yang membantu melacak perubahan pada kode program disebut...', correct:'Kontrol versi (version control)', wrong:'Kontrol akses'},
        {q:'Kapasitas transfer data dalam suatu jaringan disebut...', correct:'Bandwidth', wrong:'Latency'},
      ],
      sulit: [
        {q:'Notasi yang sering digunakan untuk menyatakan kompleksitas algoritma adalah...', correct:'Big O', wrong:'Big Data'},
        {q:'Teknik pemrograman di mana suatu fungsi memanggil dirinya sendiri disebut...', correct:'Rekursi', wrong:'Iterasi'},
        {q:'Basis data yang menyimpan data dalam tabel yang saling berelasi disebut...', correct:'Basis data relasional', wrong:'Basis data dokumen (NoSQL)'},
        {q:'Simpul paling atas pada struktur data pohon (tree) disebut...', correct:'Akar (root)', wrong:'Daun (leaf)'},
        {q:'Enkripsi simetris menggunakan...', correct:'Satu kunci yang sama untuk enkripsi dan dekripsi', wrong:'Dua kunci yang berbeda untuk enkripsi dan dekripsi'},
        {q:'Enkripsi asimetris menggunakan...', correct:'Sepasang kunci publik dan privat', wrong:'Satu kunci rahasia yang sama'},
        {q:'Cabang AI yang memungkinkan sistem belajar dari data disebut...', correct:'Machine learning', wrong:'Augmented reality'},
        {q:'Protokol yang menjadi dasar komunikasi data di internet adalah...', correct:'TCP/IP', wrong:'FTP'},
        {q:'Tujuan normalisasi basis data adalah...', correct:'Mengurangi redundansi (pengulangan) data', wrong:'Memperbesar ukuran data'},
        {q:'Proses mengurutkan data berdasarkan kriteria tertentu disebut...', correct:'Sorting', wrong:'Searching'},
        {q:'Salah satu algoritma pengurutan data adalah...', correct:'Quicksort', wrong:'Binary search'},
        {q:'Kompleksitas waktu suatu algoritma menunjukkan...', correct:'Seberapa cepat waktu eksekusi bertambah seiring ukuran data', wrong:'Jumlah baris kode pada program'},
        {q:'Serangan yang membanjiri trafik agar suatu layanan tidak dapat diakses disebut...', correct:'DDoS', wrong:'Phishing'},
        {q:'Paradigma pemrograman yang mengorganisasi kode berdasarkan objek dan kelas disebut...', correct:'Object-oriented programming (OOP)', wrong:'Pemrograman prosedural'},
        {q:'Salah satu contoh sistem kontrol versi dalam pengembangan perangkat lunak adalah...', correct:'Git', wrong:'Photoshop'},
        {q:'Antarmuka yang memungkinkan dua aplikasi berbeda saling berkomunikasi disebut...', correct:'API', wrong:'GUI'},
        {q:'Selain volume yang besar, karakteristik big data juga mencakup...', correct:'Kecepatan (velocity) dan variasi (variety) data', wrong:'Tidak ada, hanya berkaitan dengan volume data'},
        {q:'Model layanan pada cloud computing adalah...', correct:'IaaS, PaaS, dan SaaS', wrong:'HTTP, FTP, dan SMTP'},
        {q:'Penerjemah kode yang mengeksekusi program baris demi baris disebut...', correct:'Interpreter', wrong:'Compiler'},
        {q:'Teknologi pencatatan data yang terdesentralisasi dan sulit diubah adalah...', correct:'Blockchain', wrong:'Basis data terpusat'},
        {q:'Cara suatu bahasa pemrograman dieksekusi (dikompilasi atau diinterpretasi)...', correct:'Berbeda-beda tergantung bahasanya', wrong:'Semuanya sama persis'},
        {q:'Teknologi yang menghubungkan berbagai perangkat fisik melalui internet disebut...', correct:'Internet of Things (IoT)', wrong:'Local Area Network (LAN)'},
        {q:'Tujuan testing perangkat lunak adalah...', correct:'Menemukan kesalahan sebelum program digunakan secara luas', wrong:'Mempercantik tampilan program'},
        {q:'Arsitektur yang memisahkan peran peminta layanan dan penyedia layanan disebut...', correct:'Client-server', wrong:'Peer-to-peer'},
        {q:'Keamanan siber pada suatu sistem yang sudah diluncurkan...', correct:'Tetap diperlukan selama sistem digunakan', wrong:'Tidak diperlukan lagi setelah diluncurkan'},

        {q:'Load balancing digunakan untuk...', correct:'Mendistribusikan trafik ke beberapa server', wrong:'Mengompresi ukuran file'},
        {q:'Microservices adalah pendekatan pengembangan aplikasi dengan memecahnya menjadi...', correct:'Layanan-layanan kecil yang berdiri sendiri', wrong:'Satu program besar yang menyatu (monolit)'},
        {q:'Algoritma pengurutan yang berbeda umumnya memiliki kompleksitas waktu...', correct:'Berbeda-beda', wrong:'Selalu sama'},
        {q:'Teknologi yang mengemas aplikasi beserta lingkungannya, seperti Docker, disebut...', correct:'Containerization', wrong:'Defragmentasi'},
        {q:'Zero-day vulnerability adalah celah keamanan yang...', correct:'Belum diketahui atau diperbaiki oleh pengembang', wrong:'Sudah lama dan sudah tersedia perbaikannya'},
      ]
    },
    sosiologi: {
      mudah: [
        {q:'Ilmu yang mempelajari kehidupan masyarakat disebut...', correct:'Sosiologi', wrong:'Psikologi'},
        {q:'Hubungan timbal balik antarindividu atau antarkelompok disebut...', correct:'Interaksi sosial', wrong:'Stratifikasi sosial'},
        {q:'Salah satu contoh kelompok sosial adalah...', correct:'Keluarga', wrong:'Kerumunan pengunjung mal'},
        {q:'Manusia membutuhkan orang lain dalam hidupnya karena manusia...', correct:'Tidak dapat hidup sendirian', wrong:'Dapat hidup sepenuhnya sendirian'},
        {q:'Aturan yang mengatur perilaku dalam masyarakat disebut...', correct:'Norma', wrong:'Nilai'},
        {q:'Sesuatu yang dianggap baik dan berharga oleh masyarakat disebut...', correct:'Nilai sosial', wrong:'Norma sosial'},
        {q:'Proses belajar menjadi anggota masyarakat disebut...', correct:'Sosialisasi', wrong:'Urbanisasi'},
        {q:'Kedudukan seseorang dalam masyarakat disebut...', correct:'Status sosial', wrong:'Peran sosial'},
        {q:'Perilaku yang diharapkan sesuai dengan status seseorang disebut...', correct:'Peran sosial', wrong:'Status sosial'},
        {q:'Pertentangan antarindividu atau kelompok akibat perbedaan kepentingan disebut...', correct:'Konflik sosial', wrong:'Akomodasi'},
        {q:'Bentuk interaksi sosial yang bersifat positif adalah...', correct:'Kerja sama', wrong:'Pertikaian'},
        {q:'Kebudayaan suatu masyarakat pada umumnya...', correct:'Diwariskan dari generasi ke generasi', wrong:'Muncul dan hilang setiap hari'},
        {q:'Kebudayaan antara satu masyarakat dengan masyarakat lain...', correct:'Berbeda-beda', wrong:'Sama persis'},
        {q:'Lembaga pendidikan seperti sekolah berperan sebagai agen...', correct:'Sosialisasi', wrong:'Konflik'},
        {q:'Salah satu faktor penyebab perubahan sosial adalah...', correct:'Kemajuan teknologi', wrong:'Pergantian siang dan malam'},
        {q:'Manusia disebut sebagai makhluk...', correct:'Sosial', wrong:'Soliter'},
        {q:'Gotong royong merupakan contoh interaksi sosial yang bersifat...', correct:'Asosiatif (positif)', wrong:'Disosiatif (negatif)'},
        {q:'Interaksi sosial dalam masyarakat dapat berupa...', correct:'Kerja sama maupun persaingan', wrong:'Konflik saja'},
        {q:'Adat istiadat merupakan bagian dari...', correct:'Kebudayaan masyarakat', wrong:'Ilmu pengetahuan alam'},
        {q:'Kelompok sosial terbentuk karena adanya...', correct:'Kesamaan tujuan atau kepentingan', wrong:'Kesamaan jumlah anggota'},
        {q:'Salah satu dampak media sosial terhadap masyarakat adalah...', correct:'Berubahnya cara berinteraksi dan berkomunikasi', wrong:'Hilangnya kebutuhan akan norma'},
        {q:'Norma yang bersumber dari ajaran agama disebut...', correct:'Norma agama', wrong:'Norma hukum'},
        {q:'Peran setiap individu dalam masyarakat...', correct:'Berbeda-beda sesuai statusnya', wrong:'Selalu sama'},
        {q:'Toleransi penting untuk...', correct:'Menjaga keharmonisan masyarakat majemuk', wrong:'Menghilangkan perbedaan budaya'},
        {q:'Kecepatan perubahan sosial dalam masyarakat...', correct:'Dapat berlangsung cepat maupun lambat', wrong:'Selalu berlangsung cepat'},

        {q:'Tempat anak belajar berinteraksi dengan guru dan teman sebaya di luar rumah adalah...', correct:'Lingkungan sekolah', wrong:'Lingkungan keluarga'},
        {q:'Cara yang baik untuk menyelesaikan perbedaan pendapat dalam kelompok adalah...', correct:'Musyawarah', wrong:'Pemaksaan kehendak'},
        {q:'Nilai yang dijunjung tinggi di hampir semua masyarakat adalah...', correct:'Kejujuran', wrong:'Kecurangan'},
        {q:'Interaksi sosial melalui surat atau telepon termasuk interaksi...', correct:'Tidak langsung', wrong:'Langsung'},
        {q:'Identitas sosial seseorang terbentuk dari...', correct:'Lingkungan dan kelompok tempat ia hidup', wrong:'Nama yang diberikan sejak lahir saja'},
      ],
      sedang: [
        {q:'Pembagian masyarakat ke dalam tingkatan-tingkatan tertentu disebut...', correct:'Stratifikasi sosial', wrong:'Diferensiasi sosial'},
        {q:'Mobilitas sosial vertikal berarti...', correct:'Perpindahan status ke tingkat yang lebih tinggi atau lebih rendah', wrong:'Perpindahan tempat tinggal dari satu daerah ke daerah lain'},
        {q:'Lembaga sosial dibentuk untuk...', correct:'Memenuhi kebutuhan tertentu dalam masyarakat', wrong:'Menghapus norma yang berlaku'},
        {q:'Penyelesaian konflik sosial dapat dilakukan melalui...', correct:'Negosiasi, mediasi, atau musyawarah', wrong:'Kekerasan sebagai satu-satunya cara'},
        {q:'Proses percampuran dua budaya tanpa menghilangkan ciri aslinya disebut...', correct:'Akulturasi', wrong:'Asimilasi'},
        {q:'Proses peleburan dua kebudayaan menjadi satu kebudayaan baru disebut...', correct:'Asimilasi', wrong:'Akulturasi'},
        {q:'Pembedaan masyarakat secara horizontal, misalnya berdasarkan suku atau profesi, disebut...', correct:'Diferensiasi sosial', wrong:'Stratifikasi sosial'},
        {q:'Integrasi sosial berarti...', correct:'Menyatunya unsur-unsur masyarakat menjadi kesatuan yang serasi', wrong:'Perpecahan antaranggota masyarakat'},
        {q:'Perubahan sosial yang membawa masyarakat ke arah kemajuan disebut perubahan...', correct:'Progresif', wrong:'Regresif'},
        {q:'Agen sosialisasi yang pertama dan utama bagi seorang anak adalah...', correct:'Keluarga', wrong:'Sekolah'},
        {q:'Kesesuaian perilaku individu dengan norma yang berlaku disebut...', correct:'Konformitas', wrong:'Deviasi'},
        {q:'Perilaku yang menyimpang dari norma masyarakat disebut...', correct:'Deviasi sosial', wrong:'Integrasi sosial'},
        {q:'Upaya untuk menjaga ketertiban dalam masyarakat disebut...', correct:'Kontrol sosial', wrong:'Mobilitas sosial'},
        {q:'Penyimpangan sosial dalam masyarakat...', correct:'Tidak semuanya tergolong tindak kriminal', wrong:'Semuanya tergolong tindak kriminal'},
        {q:'Masyarakat multikultural adalah masyarakat yang...', correct:'Terdiri dari berbagai kelompok dengan latar belakang budaya berbeda', wrong:'Hanya terdiri dari satu kelompok budaya'},
        {q:'Interaksi sosial di era digital dapat dilakukan...', correct:'Secara langsung maupun melalui media', wrong:'Hanya secara langsung'},
        {q:'Solidaritas sosial berfungsi untuk...', correct:'Memperkuat rasa kebersamaan dalam kelompok', wrong:'Menimbulkan persaingan antaranggota'},
        {q:'Objek kajian sosiologi adalah...', correct:'Hubungan individu dan kelompok dalam masyarakat', wrong:'Hanya perilaku individu secara terpisah'},
        {q:'Teori fungsionalisme memandang masyarakat sebagai...', correct:'Sistem yang bagian-bagiannya saling berkaitan', wrong:'Arena pertentangan antarkelas'},
        {q:'Teori konflik menekankan...', correct:'Perbedaan kepentingan dalam masyarakat', wrong:'Keseimbangan dan keteraturan sosial'},
        {q:'Salah satu dampak modernisasi terhadap masyarakat adalah...', correct:'Bergesernya nilai dan norma', wrong:'Hilangnya seluruh kebudayaan secara instan'},
        {q:'Kesenjangan sosial dapat muncul akibat...', correct:'Perbedaan akses terhadap sumber daya', wrong:'Kesamaan akses terhadap sumber daya'},
        {q:'Lembaga ekonomi berperan dalam...', correct:'Mengatur produksi dan distribusi barang dan jasa', wrong:'Membina keyakinan dan ibadah masyarakat'},
        {q:'Bencana alam dan peperangan sebagai penyebab perubahan sosial termasuk faktor...', correct:'Eksternal', wrong:'Internal'},
        {q:'Norma yang memiliki sanksi paling tegas dan tertulis adalah...', correct:'Norma hukum', wrong:'Norma kesopanan'},

        {q:'Perpindahan penduduk dari desa ke kota disebut...', correct:'Urbanisasi', wrong:'Transmigrasi'},
        {q:'Kelompok sosial dalam masyarakat...', correct:'Dapat berubah seiring waktu', wrong:'Bersifat permanen dan tidak dapat berubah'},
        {q:'Pola sosialisasi yang menekankan hukuman terhadap perilaku menyimpang disebut...', correct:'Sosialisasi represif', wrong:'Sosialisasi partisipatif'},
        {q:'Pola sosialisasi yang melibatkan penghargaan atas perilaku yang sesuai disebut...', correct:'Sosialisasi partisipatif', wrong:'Sosialisasi represif'},
        {q:'Konflik dengan kelompok lain dapat berfungsi positif karena...', correct:'Mempererat solidaritas dalam kelompok sendiri', wrong:'Melemahkan ikatan antaranggota kelompok'},
      ],
      sulit: [
        {q:'Teori interaksionisme simbolik menekankan pentingnya...', correct:'Makna simbol dalam interaksi sosial', wrong:'Struktur ekonomi masyarakat'},
        {q:'Kondisi ketika norma dalam masyarakat melemah atau kacau disebut...', correct:'Anomie', wrong:'Solidaritas mekanik'},
        {q:'Dominasi suatu kelompok terhadap kelompok lain dalam sosiologi dikenal dengan istilah...', correct:'Hegemoni', wrong:'Akulturasi'},
        {q:'Teori fungsionalisme struktural dikembangkan oleh...', correct:'Talcott Parsons', wrong:'Karl Marx'},
        {q:'Konsep kelas sosial menurut Marx berkaitan dengan...', correct:'Kepemilikan alat produksi', wrong:'Tingkat pendidikan'},
        {q:'Konsep habitus (kebiasaan yang terinternalisasi) dikembangkan oleh...', correct:'Pierre Bourdieu', wrong:'Max Weber'},
        {q:'Modal sosial berkaitan dengan...', correct:'Jaringan, kepercayaan, dan norma yang memudahkan kerja sama', wrong:'Kekayaan materi seseorang saja'},
        {q:'Teori pertukaran sosial memandang interaksi sebagai...', correct:'Proses saling memberi dan menerima', wrong:'Pertentangan antarkelas'},
        {q:'Gerakan sosial pada umumnya muncul sebagai...', correct:'Upaya kolektif menanggapi kondisi tertentu dalam masyarakat', wrong:'Perilaku acak individu tanpa tujuan bersama'},
        {q:'Sikap yang menganggap budaya sendiri lebih unggul daripada budaya lain disebut...', correct:'Etnosentrisme', wrong:'Relativisme budaya'},
        {q:'Relativisme budaya menilai suatu budaya berdasarkan...', correct:'Konteks dan standar budaya itu sendiri', wrong:'Standar budaya lain'},
        {q:'Konsep dramaturgi untuk menjelaskan interaksi sosial dikembangkan oleh...', correct:'Erving Goffman', wrong:'Emile Durkheim'},
        {q:'Pada stratifikasi sosial tertutup, mobilitas sosial...', correct:'Sangat terbatas', wrong:'Berlangsung bebas'},
        {q:'Menurut Marx, pertentangan kelas terjadi antara...', correct:'Kelas borjuis dan proletar', wrong:'Kelompok mayoritas dan minoritas'},
        {q:'Sosialisasi yang terjadi di luar lingkungan keluarga, misalnya di sekolah, disebut...', correct:'Sosialisasi sekunder', wrong:'Sosialisasi primer'},
        {q:'Penyimpangan yang dilakukan sekali dan tidak diulang disebut...', correct:'Deviasi primer', wrong:'Deviasi sekunder'},
        {q:'Teori yang menjelaskan bahwa pemberian cap sosial dapat memengaruhi identitas seseorang adalah...', correct:'Teori labeling', wrong:'Teori fungsionalisme'},
        {q:'Perasaan keterasingan individu dalam masyarakat disebut...', correct:'Alienasi', wrong:'Anomie'},
        {q:'Teori-teori sosiologi memandang masyarakat dari...', correct:'Sudut pandang yang beragam', wrong:'Sudut pandang yang sama'},
        {q:'Lembaga yang mengatur seluruh aspek kehidupan anggotanya, contohnya penjara, disebut...', correct:'Institusi total', wrong:'Institusi sukarela'},
        {q:'Pengetahuan, keterampilan, dan pendidikan yang dimiliki seseorang sebagai sumber daya sosial disebut...', correct:'Kapital budaya', wrong:'Kapital ekonomi'},
        {q:'Postmodernisme dalam sosiologi cenderung menolak...', correct:'Kebenaran tunggal yang mutlak', wrong:'Keberagaman sudut pandang'},
        {q:'Globalisasi budaya dapat menyebabkan...', correct:'Homogenisasi maupun penguatan identitas lokal', wrong:'Hilangnya seluruh budaya lokal tanpa pengecualian'},
        {q:'Pendekatan yang digunakan dalam metode penelitian sosiologi adalah...', correct:'Kuantitatif maupun kualitatif', wrong:'Kuantitatif saja'},
        {q:'Penemuan baru (inovasi) dalam masyarakat sebagai pemicu perubahan sosial tergolong faktor...', correct:'Internal', wrong:'Eksternal'},

        {q:'Teori sosiologi kritis menyoroti...', correct:'Ketidakadilan struktural dalam masyarakat', wrong:'Keteraturan sosial yang harus dipertahankan'},
        {q:'Simulakra berkaitan dengan...', correct:'Representasi realitas dalam masyarakat postmodern', wrong:'Pembagian kerja dalam masyarakat industri'},
        {q:'Penelitian sosiologi yang melibatkan manusia harus memperhatikan...', correct:'Etika penelitian, seperti persetujuan dan kerahasiaan responden', wrong:'Kepentingan peneliti saja tanpa etika'},
        {q:'Tempat masyarakat berdiskusi dan berpartisipasi secara terbuka disebut...', correct:'Ruang publik', wrong:'Ruang privat'},
        {q:'Proses ketimpangan diwariskan dari satu generasi ke generasi berikutnya dijelaskan oleh konsep...', correct:'Reproduksi sosial', wrong:'Mobilitas sosial'},
      ]
    },
    penjas: {
      mudah: [
        {q:'Manfaat utama olahraga bagi tubuh adalah...', correct:'Menjaga kesehatan tubuh', wrong:'Menurunkan kesehatan tubuh'},
        {q:'Pemanasan sebaiknya dilakukan...', correct:'Sebelum berolahraga', wrong:'Setelah berolahraga'},
        {q:'Jumlah pemain sepak bola dalam satu tim adalah...', correct:'11 pemain', wrong:'9 pemain'},
        {q:'Bola basket dimainkan menggunakan...', correct:'Tangan', wrong:'Kaki'},
        {q:'Renang adalah olahraga yang dilakukan di...', correct:'Air', wrong:'Lapangan'},
        {q:'Lari maraton termasuk lari jarak...', correct:'Jauh', wrong:'Pendek'},
        {q:'Push up adalah latihan untuk melatih...', correct:'Kekuatan otot lengan', wrong:'Kelenturan kaki'},
        {q:'Peregangan sebelum olahraga berguna untuk...', correct:'Mengurangi risiko cedera', wrong:'Meningkatkan risiko cedera'},
        {q:'Bulu tangkis dimainkan menggunakan...', correct:'Raket dan kok', wrong:'Tongkat dan bola'},
        {q:'Bola voli dimainkan dengan cara...', correct:'Memukul bola menggunakan tangan', wrong:'Menendang bola menggunakan kaki'},
        {q:'Istirahat yang cukup bagi kesehatan tubuh adalah hal yang...', correct:'Penting', wrong:'Tidak penting'},
        {q:'Makanan sehat bagi performa olahraga bersifat...', correct:'Mendukung performa', wrong:'Menurunkan performa'},
        {q:'Olahraga boleh dilakukan oleh...', correct:'Semua orang', wrong:'Atlet profesional saja'},
        {q:'Senam irama biasanya diiringi dengan...', correct:'Musik', wrong:'Peluit'},
        {q:'Lompat jauh termasuk cabang...', correct:'Atletik', wrong:'Renang'},
        {q:'Sarung tangan tinju digunakan dalam olahraga...', correct:'Tinju', wrong:'Bulu tangkis'},
        {q:'Menjaga kebersihan tubuh setelah berolahraga adalah hal yang...', correct:'Penting', wrong:'Tidak penting'},
        {q:'Untuk berenang dengan baik, teknik pernapasan yang benar...', correct:'Perlu diketahui', wrong:'Tidak perlu diketahui'},
        {q:'Bersepeda termasuk jenis olahraga...', correct:'Kardio', wrong:'Kekuatan'},
        {q:'Wasit dalam pertandingan bertugas...', correct:'Memimpin dan mengawasi jalannya pertandingan', wrong:'Menjadi pemain cadangan'},
        {q:'Aturan permainan pada setiap cabang olahraga...', correct:'Berbeda-beda sesuai cabangnya', wrong:'Selalu sama persis'},
        {q:'Pemanasan terhadap performa olahraga...', correct:'Berpengaruh positif', wrong:'Tidak berpengaruh'},
        {q:'Dalam olahraga beregu seperti sepak bola, kerja sama tim bersifat...', correct:'Penting', wrong:'Tidak penting'},
        {q:'Kelenturan tubuh dapat dilatih melalui...', correct:'Peregangan secara rutin', wrong:'Menahan napas'},
        {q:'Saat berolahraga dalam waktu lama, air minum...', correct:'Tetap diperlukan', wrong:'Tidak diperlukan'},

        {q:'Olahraga tim membutuhkan...', correct:'Komunikasi yang baik antar pemain', wrong:'Persaingan antar pemain satu tim'},
        {q:'Cedera olahraga dapat dicegah dengan...', correct:'Pemanasan dan teknik yang benar', wrong:'Langsung berolahraga tanpa persiapan'},
        {q:'Intensitas olahraga sebaiknya...', correct:'Disesuaikan dengan kondisi tubuh masing-masing', wrong:'Sama untuk semua orang tanpa memperhatikan kondisi tubuh'},
        {q:'Tidur yang cukup bagi pemulihan tubuh setelah berolahraga bersifat...', correct:'Mendukung pemulihan', wrong:'Menghambat pemulihan'},
        {q:'Untuk menjaga kebugaran, senam pagi sebaiknya dilakukan secara...', correct:'Rutin', wrong:'Sesekali saja'},
      ],
      sedang: [
        {q:'Saat melakukan aktivitas fisik yang intens, denyut jantung akan...', correct:'Meningkat', wrong:'Menurun'},
        {q:'VO2 max adalah ukuran kemampuan tubuh dalam...', correct:'Menggunakan oksigen saat berolahraga', wrong:'Menghasilkan asam laktat'},
        {q:'Latihan interval menggabungkan intensitas...', correct:'Tinggi dan rendah secara bergantian', wrong:'Tinggi terus-menerus tanpa jeda'},
        {q:'Penanganan awal cedera otot dapat menggunakan metode...', correct:'RICE (Rest, Ice, Compression, Elevation)', wrong:'HIIT (High Intensity Interval Training)'},
        {q:'Daya tahan kardiorespirasi berkaitan dengan kemampuan...', correct:'Jantung dan paru-paru saat beraktivitas', wrong:'Kekuatan otot lengan saja'},
        {q:'Teknik pernapasan yang salah terhadap performa renang...', correct:'Tetap berpengaruh dan menurunkan performa', wrong:'Tidak berpengaruh sama sekali'},
        {q:'Zona latihan denyut jantung digunakan untuk mengatur...', correct:'Intensitas olahraga', wrong:'Jumlah pemain dalam tim'},
        {q:'Peraturan offside berlaku dalam permainan...', correct:'Sepak bola', wrong:'Bola voli'},
        {q:'Dalam bola voli, jumlah maksimal sentuhan sebelum bola melewati net adalah...', correct:'3 kali sentuhan', wrong:'5 kali sentuhan'},
        {q:'Prinsip overload dalam latihan berarti meningkatkan beban secara...', correct:'Bertahap melebihi kemampuan biasa', wrong:'Tetap sama setiap saat'},
        {q:'Sebelum berolahraga intens, pemanasan yang lebih dianjurkan adalah...', correct:'Pemanasan dinamis', wrong:'Pemanasan statis'},
        {q:'Kekuatan otot dapat ditingkatkan melalui...', correct:'Latihan beban secara teratur', wrong:'Istirahat total tanpa latihan'},
        {q:'Teknik dasar dalam bola basket meliputi...', correct:'Dribbling, passing, dan shooting', wrong:'Servis, smash, dan blocking'},
        {q:'Start jongkok digunakan pada lari jarak...', correct:'Pendek (sprint)', wrong:'Jauh (maraton)'},
        {q:'Jika tidak dilatih secara rutin, fleksibilitas tubuh akan...', correct:'Menurun', wrong:'Meningkat'},
        {q:'Prinsip specificity dalam latihan berarti latihan harus...', correct:'Disesuaikan dengan tujuan tertentu', wrong:'Dilakukan secara acak tanpa tujuan'},
        {q:'Jenis kebugaran fisik yang diperlukan setiap cabang olahraga...', correct:'Berbeda-beda sesuai kebutuhan cabangnya', wrong:'Selalu sama persis untuk semua cabang'},
        {q:'Setelah latihan intensitas tinggi, recovery atau pemulihan bersifat...', correct:'Penting', wrong:'Tidak diperlukan'},
        {q:'Dehidrasi terhadap performa fisik seseorang akan...', correct:'Menurunkan performa', wrong:'Meningkatkan performa'},
        {q:'Teknik smash dalam bulu tangkis digunakan untuk...', correct:'Menyerang lawan', wrong:'Bertahan dari serangan lawan'},
        {q:'Dalam senam lantai, unsur yang sangat diperlukan adalah...', correct:'Keseimbangan tubuh', wrong:'Kecepatan lari'},
        {q:'Latihan aerobik secara rutin dapat...', correct:'Meningkatkan kesehatan jantung', wrong:'Menurunkan kesehatan jantung'},
        {q:'Cedera olahraga sebaiknya ditangani dengan...', correct:'Penanganan khusus yang tepat', wrong:'Dibiarkan tanpa penanganan'},
        {q:'Prinsip progresif dalam latihan berarti peningkatan beban dilakukan secara...', correct:'Bertahap', wrong:'Langsung maksimal sejak awal'},
        {q:'Postur tubuh yang baik dalam berbagai jenis olahraga bersifat...', correct:'Penting', wrong:'Tidak berpengaruh'},

        {q:'Teknik dasar renang meliputi gerakan...', correct:'Kaki, tangan, dan pernapasan', wrong:'Melompat dan berputar di udara'},
        {q:'Latihan fleksibilitas terhadap performa olahraga...', correct:'Tetap berpengaruh dan mendukung performa', wrong:'Tidak berpengaruh sama sekali'},
        {q:'Kebugaran jasmani mencakup unsur...', correct:'Daya tahan, kekuatan, dan kelenturan tubuh', wrong:'Kecerdasan dan hafalan materi'},
        {q:'Agar performa optimal saat bertanding, nutrisi sebelum bertanding perlu...', correct:'Diperhatikan dengan baik', wrong:'Diabaikan'},
        {q:'Peran pemain cadangan dalam olahraga tim...', correct:'Tetap penting sebagai bagian dari strategi tim', wrong:'Tidak memiliki peran penting sama sekali'},
      ],
      sulit: [
        {q:'Sistem energi anaerobik digunakan tubuh pada aktivitas fisik...', correct:'Intensitas tinggi dalam waktu singkat', wrong:'Intensitas rendah dalam waktu lama'},
        {q:'Sistem energi aerobik lebih dominan digunakan pada aktivitas fisik...', correct:'Durasi panjang, intensitas rendah hingga sedang', wrong:'Durasi singkat, intensitas sangat tinggi'},
        {q:'Periodisasi latihan adalah perencanaan latihan yang...', correct:'Dibagi dalam beberapa fase untuk mencapai performa optimal', wrong:'Dilakukan dengan intensitas sama setiap hari tanpa perencanaan'},
        {q:'Overtraining dapat menyebabkan...', correct:'Penurunan performa dan risiko cedera', wrong:'Peningkatan performa secara instan'},
        {q:'Analisis biomekanika digunakan untuk mengevaluasi...', correct:'Teknik gerakan olahraga', wrong:'Jadwal pertandingan'},
        {q:'Ambang laktat adalah titik di mana produksi asam laktat...', correct:'Meningkat tajam saat berolahraga', wrong:'Berhenti sepenuhnya saat berolahraga'},
        {q:'Program latihan bagi atlet sebaiknya...', correct:'Disesuaikan dengan cabang olahraganya masing-masing', wrong:'Dibuat identik untuk semua cabang olahraga'},
        {q:'Pemulihan aktif dilakukan dengan cara...', correct:'Melakukan aktivitas ringan setelah latihan berat', wrong:'Beristirahat total tanpa gerakan sama sekali'},
        {q:'Prinsip individualitas dalam latihan menyatakan bahwa respons tubuh setiap orang terhadap latihan...', correct:'Berbeda-beda', wrong:'Selalu sama persis'},
        {q:'Doping dalam olahraga dilarang karena...', correct:'Memberikan keunggulan yang tidak fair', wrong:'Membuat pertandingan lebih adil'},
        {q:'VO2 max dapat ditingkatkan melalui...', correct:'Latihan aerobik yang terprogram', wrong:'Istirahat total tanpa aktivitas fisik'},
        {q:'Taktik dan strategi dalam olahraga tim terhadap hasil pertandingan...', correct:'Tetap memengaruhi hasil pertandingan', wrong:'Tidak memengaruhi hasil pertandingan sama sekali'},
        {q:'Analisis performa atlet dapat dilakukan menggunakan...', correct:'Data statistik pertandingan', wrong:'Perkiraan tanpa data'},
        {q:'Rehabilitasi cedera olahraga melibatkan program latihan yang...', correct:'Bertahap untuk mengembalikan fungsi tubuh', wrong:'Langsung berat sejak awal pemulihan'},
        {q:'Kelelahan otot terhadap koordinasi dan teknik gerakan atlet akan...', correct:'Memengaruhi dan menurunkan koordinasi', wrong:'Tidak berpengaruh sama sekali'},
        {q:'Manajemen pertandingan berkaitan dengan aspek...', correct:'Fisik dan mental atlet', wrong:'Fisik saja, tanpa aspek mental'},
        {q:'Psikologi olahraga membahas aspek...', correct:'Mental yang memengaruhi performa atlet', wrong:'Peraturan pertandingan resmi'},
        {q:'Nutrisi olahraga berperan penting dalam...', correct:'Pemulihan dan performa atlet', wrong:'Penentuan skor akhir pertandingan'},
        {q:'Latihan pliometrik digunakan untuk meningkatkan...', correct:'Daya ledak otot', wrong:'Kelenturan sendi saja'},
        {q:'Prinsip reversibilitas berarti kebugaran yang telah dicapai akan...', correct:'Menurun jika latihan dihentikan', wrong:'Tetap bertahan meski latihan dihentikan'},
        {q:'Risiko cedera pada setiap cabang olahraga...', correct:'Berbeda-beda besarnya', wrong:'Selalu sama besarnya'},
        {q:'Analisis gerak digunakan pelatih untuk...', correct:'Memperbaiki teknik atlet', wrong:'Menentukan jadwal pertandingan'},
        {q:'Kapasitas paru-paru dapat meningkat melalui...', correct:'Latihan aerobik jangka panjang', wrong:'Istirahat total tanpa latihan'},
        {q:'Dalam program latihan atlet profesional, manajemen waktu istirahat bersifat...', correct:'Penting', wrong:'Tidak diperlukan'},
        {q:'Kesehatan mental atlet terhadap performa olahraga mereka...', correct:'Tetap berkaitan dan berpengaruh', wrong:'Tidak berkaitan sama sekali'},

        {q:'Analisis video pertandingan dapat membantu evaluasi...', correct:'Teknik dan strategi atlet', wrong:'Jumlah penonton pertandingan'},
        {q:'Program latihan jangka panjang perlu disesuaikan dengan...', correct:'Tahap perkembangan atlet', wrong:'Jadwal siaran televisi'},
        {q:'Waktu pemulihan yang dibutuhkan untuk setiap cedera olahraga...', correct:'Berbeda-beda tergantung jenis cederanya', wrong:'Selalu sama untuk semua jenis cedera'},
        {q:'Menjelang pertandingan besar, manajemen stres bagi atlet bersifat...', correct:'Penting', wrong:'Tidak diperlukan'},
        {q:'Dalam penanganan cedera atlet, kerja sama tim medis dan pelatih bersifat...', correct:'Penting', wrong:'Tidak diperlukan'},
      ]
    },
  };
  var MAPEL_LIST = [
    {v:'bindo', label:'Bahasa Indonesia'},
    {v:'binggris', label:'Bahasa Inggris'},
    {v:'pkn', label:'PKn'},
    {v:'agama_islam', label:'Pendidikan Agama Islam'},
    {v:'matematika', label:'Matematika'},
    {v:'informatika', label:'Informatika'},
    {v:'seni_budaya', label:'Seni Budaya'},
    {v:'prakarya', label:'Prakarya'},
    {v:'ipa', label:'IPA'},
    {v:'ips', label:'IPS'},
    {v:'sosiologi', label:'Sosiologi'},
    {v:'koding', label:'Koding dan KA'},
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
      + '    <div class="mxp-field"><label for="mxpJumlah">Jumlah soal (maks. 30)</label>'
      + '      <input type="number" id="mxpJumlah" min="1" max="30" value="25" step="1"></div>'
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
    var jumlahEl = document.getElementById('mxpJumlah');
    var statusEl = document.getElementById('mxpStatus');

    function shuffleArr(arr){
      var a = arr.slice();
      for(var i=a.length-1;i>0;i--){
        var j = Math.floor(Math.random()*(i+1));
        var t = a[i]; a[i]=a[j]; a[j]=t;
      }
      return a;
    }

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
        statusEl.textContent = 'Bank soal untuk "' + findMapelLabel(mapel) + '" tingkat ' + tingkat + ' sedang disiapkan dan belum tersedia. Mapel yang sudah tersedia: Matematika, Bahasa Indonesia, PKn, IPA, Bahasa Inggris, Pendidikan Agama Islam, IPS, Informatika, Sosiologi, Penjas. Mapel lain bisa diisi manual lewat "Edit soal".';
        return;
      }

      var maxAvail = Math.min(30, set.length);
      var jumlah = parseInt(jumlahEl.value, 10);
      if(isNaN(jumlah) || jumlah < 1) jumlah = 1;
      if(jumlah > maxAvail) jumlah = maxAvail;
      jumlahEl.value = jumlah;

      var picked = shuffleArr(set).slice(0, jumlah);

      var current = null;
      try{ current = localStorage.getItem('kuisBK.v1.soal'); }catch(e){}
      var hasCurrent = false;
      try{ hasCurrent = current && JSON.parse(current).length > 0; }catch(e){}

      if(hasCurrent && !confirm('Ganti soal saat ini dengan ' + picked.length + ' soal siap pakai (' + findMapelLabel(mapel) + ' - ' + jenjangLabel + ' Kelas ' + kelas + ' - ' + tingkat + ')? Soal lama akan digantikan (masih bisa diedit lagi setelah dimuat).')){
        return;
      }

      var newBank = picked.map(function(it){ return {q:it.q, correct:it.correct, wrong:it.wrong, side:'acak'}; });
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
    jumlahEl.addEventListener('change', function(){
      var v = parseInt(jumlahEl.value, 10);
      if(isNaN(v) || v < 1) v = 1;
      if(v > 30) v = 30;
      jumlahEl.value = v;
    });

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

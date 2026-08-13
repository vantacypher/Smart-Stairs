<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Smart Stairs — Interactive Exhibition</title>
<!-- Optimization: Preconnect and link Google Fonts for parallel loading (prevents CSS parsing blocks) -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">
<style>
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}

:root{
  --navy:#060813;
  --card:#111632;
  --card-border:rgba(255,255,255,0.08);
  --led:#00f0ff;
  --walnut:#4a3118;
  --walnut-border:#624323;
  --riser:#1a1c29;
  --text:#f0f4ff;
  --muted:#8e9db4;
  --accent:#2563ff;
  --amber:#ff9f1c;
  --green:#2ec4b6;
  --radius:16px;
  --trans:cubic-bezier(.4,0,.2,1);
}

html,body{
  height:100%;overflow:hidden;
  background:var(--navy);
  font-family:'Outfit',-apple-system,BlinkMacSystemFont,sans-serif;
  color:var(--text);
  -webkit-tap-highlight-color:transparent;
}

/* ── SCREENS ─────────────────────────────── */
.screen{
  position:fixed;inset:0;
  display:flex;flex-direction:column;
  overflow:hidden;
  opacity:0;pointer-events:none;
  transform:translate3d(0,18px,0);
  transition:opacity 0.45s var(--trans),transform 0.45s var(--trans);
  z-index:5;
}
.screen.active{opacity:1;pointer-events:all;transform:translate3d(0,0,0)}

/* ── INTRO OVERLAY ──────────────────────── */
#intro-overlay{
  position:fixed;inset:0;
  /* Optimization: Removed expensive backdrop-filter; the scene behind is already blurred via CSS filter */
  background:rgba(6,8,19,0.85);
  display:flex;flex-direction:column;
  justify-content:center;align-items:center;
  gap:2rem;z-index:100;padding:24px;
  text-align:center;
  transition:opacity 0.75s ease;
}
#intro-overlay.fade-out{opacity:0;pointer-events:none}

.intro-brand{font-size:12px;letter-spacing:.25em;text-transform:uppercase;color:var(--muted);font-weight:500}
.intro-title{font-size:clamp(42px,12vw,58px);font-weight:800;letter-spacing:-.02em;line-height:1.05}
.intro-title span{color:var(--led);text-shadow:0 0 24px rgba(0,240,255,0.38)}
.intro-desc{font-size:14px;color:var(--muted);max-width:280px;line-height:1.5}

.play-btn{
  width:96px;height:96px;border-radius:50%;
  background:rgba(0,240,255,0.08);
  border:1.5px solid rgba(0,240,255,0.32);
  display:flex;align-items:center;justify-content:center;
  cursor:pointer;
  /* Optimization: Specific transition properties instead of 'all' */
  transition:transform 0.28s var(--trans), background-color 0.28s var(--trans);
  box-shadow:0 0 30px rgba(0,240,255,0.1);
  position:relative;
}
.play-btn::before{
  content:'';position:absolute;inset:-7px;border-radius:50%;
  border:1px solid rgba(0,240,255,0.14);
  animation:pulsering 2s ease-in-out infinite;
}
@keyframes pulsering{0%,100%{transform:scale(1);opacity:.4}50%{transform:scale(1.08);opacity:1}}
.play-btn:active{background:rgba(0,240,255,0.18);transform:scale(0.95)}
.play-icon{width:0;height:0;border-style:solid;border-width:12px 0 12px 22px;border-color:transparent transparent transparent var(--led);margin-left:6px;filter:drop-shadow(0 0 6px var(--led))}
.play-label{font-size:11px;letter-spacing:.2em;text-transform:uppercase;color:var(--muted)}

/* Ambient roaming lights behind intro glass (GPU-accelerated translates) */
.intro-glow{position:absolute;border-radius:50%;pointer-events:none}
.ig1{width:260px;height:260px;background:radial-gradient(circle,rgba(0,240,255,0.09),transparent 70%);animation:roam1 9s ease-in-out infinite}
.ig2{width:220px;height:220px;background:radial-gradient(circle,rgba(37,99,255,0.08),transparent 70%);animation:roam2 12s ease-in-out infinite}
@keyframes roam1{0%,100%{transform:translate3d(-70px,-90px,0)}40%{transform:translate3d(80px,-30px,0)}70%{transform:translate3d(10px,80px,0)}}
@keyframes roam2{0%,100%{transform:translate3d(70px,80px,0)}40%{transform:translate3d(-80px,40px,0)}70%{transform:translate3d(-10px,-80px,0)}}

/* ── MAIN SCREEN ────────────────────────── */
#main{background:var(--navy);padding:16px 18px 20px}
.main-header{display:flex;justify-content:space-between;align-items:center;margin-bottom:12px;flex-shrink:0}
.brand-title{font-size:18px;font-weight:700;letter-spacing:-.01em}
.brand-title span{color:var(--led)}

.status-pill{
  display:flex;align-items:center;gap:8px;
  background:rgba(255,255,255,0.03);border:1px solid var(--card-border);
  padding:6px 12px;border-radius:20px;
  font-size:10px;font-weight:600;letter-spacing:.05em;text-transform:uppercase;
}
.status-dot{width:7px;height:7px;border-radius:50%}
.status-pill.ready .status-dot{background:var(--green);box-shadow:0 0 8px var(--green);animation:blink 2s infinite}
.status-pill.detected .status-dot{background:var(--amber);box-shadow:0 0 8px var(--amber);animation:blink 0.8s infinite}
.status-pill.processing .status-dot{background:var(--accent);box-shadow:0 0 8px var(--accent);animation:blink 0.4s infinite}
.status-pill.active-state .status-dot{background:var(--led);box-shadow:0 0 8px var(--led);animation:blink 1.2s infinite}
@keyframes blink{0%,100%{opacity:1}50%{opacity:.25}}

/* 3D SCENE (STAIRCASE FIXED FACING FRONT) */
.scene-outer-container{
  flex:1;position:relative;width:100%;
  display:flex;flex-direction:column;justify-content:center;align-items:center;
  overflow:hidden;border-radius:var(--radius);
  background:radial-gradient(circle at 50% 55%,rgba(17,22,50,0.5),transparent 75%);
  border:1px solid rgba(255,255,255,0.03);
  margin-bottom:14px;
}

.staircase-viewport{
  width:100%;height:100%;
  position:relative;
  perspective:1100px;perspective-origin:50% 30%;
  display:flex;justify-content:center;align-items:center;
}
.staircase-scene{
  position:relative;width:180px;height:160px;
  transform-style:preserve-3d;
  transform:rotateX(45deg) rotateY(0deg) rotateZ(0deg);
  will-change:transform;
}
.staircase-scene.intro-blur{filter:blur(14px);transition:filter 0s}
.staircase-scene.intro-blur.unblur{filter:blur(0px);transition:filter 1.1s ease}

/* 3D STEP */
.step-3d{
  position:absolute;left:0;top:0;
  width:160px;height:24px;
  transform-style:preserve-3d;
  will-change:transform;
}
.step-3d .face{position:absolute;backface-visibility:visible}
.step-3d .tread{
  width:180px;height:36px;left:-10px;top:0;
  background:linear-gradient(135deg,var(--walnut) 0%,#301f0e 100%);
  border:1px solid var(--walnut-border);
  transform:translate3d(0,0,4px) rotateX(-90deg);
  transform-origin:top center;
  border-radius:2px;
}
.step-3d .tread::after{
  content:'';position:absolute;inset:0;
  background:radial-gradient(ellipse at 50% 0%,rgba(0,240,255,0),transparent 70%);
  transition:background 0.4s ease;border-radius:2px;
}
.step-3d.lit-below .tread::after{background:radial-gradient(ellipse at 50% 0%,rgba(0,240,255,0.22),transparent 75%)}
.step-3d .riser{
  width:160px;height:24px;left:0;top:0;
  background:var(--riser);
  border:1px solid rgba(255,255,255,0.03);
  transition:background 0.35s ease;
}
.step-3d.lit .riser{background:linear-gradient(180deg,rgba(0,240,255,0.42) 0%,rgba(0,240,255,0.07) 60%,var(--riser) 100%)}
.step-3d .side-left,.step-3d .side-right{
  width:36px;height:24px;top:0;
  background:#12131b;border:1px solid rgba(255,255,255,0.03);
  display:flex;align-items:center;justify-content:center;
}
.step-3d .side-left::after,.step-3d .side-right::after{
  content:'';width:4px;height:4px;border-radius:50%;background:#4a4c5a;
}
.step-3d .side-left{left:0;transform:translate3d(0,0,4px) rotateY(-90deg);transform-origin:left center}
.step-3d .side-right{left:160px;transform:translate3d(0,0,4px) rotateY(-90deg);transform-origin:left center}

/* LED Edge Glow */
.step-3d .led-strip{
  position:absolute;top:0;left:0;width:160px;height:3px;
  background:var(--led);
  /* Optimization: Defined box-shadow statically. We will animate opacity instead of box-shadow to prevent paint cycles. */
  box-shadow:0 0 12px var(--led),0 1px 3px rgba(0, 240, 255, 0.8);
  opacity:0.15; /* Standby opacity */
  transform:translate3d(0,0,1px);
  transition:opacity 0.35s ease;
  will-change:opacity;
}
.step-3d.lit .led-strip{
  opacity:1 !important; /* Forces full brightness, overriding breathing animations */
}

/* Optimization: Ambient LED breathing (only runs when the simulation is not active to prevent transition snaps) */
.staircase-scene:not(.simulation-active) .step-3d:not(.lit) .led-strip{animation:ledbreath 3s ease-in-out infinite}
.staircase-scene:not(.simulation-active) .step-3d:nth-child(2):not(.lit) .led-strip{animation-delay:.3s}
.staircase-scene:not(.simulation-active) .step-3d:nth-child(3):not(.lit) .led-strip{animation-delay:.6s}
.staircase-scene:not(.simulation-active) .step-3d:nth-child(4):not(.lit) .led-strip{animation-delay:.9s}
.staircase-scene:not(.simulation-active) .step-3d:nth-child(5):not(.lit) .led-strip{animation-delay:1.2s}
.staircase-scene:not(.simulation-active) .step-3d:nth-child(6):not(.lit) .led-strip{animation-delay:1.5s}

/* Optimization: Animates opacity (GPU-friendly) instead of box-shadow (main thread CPU repaint) */
@keyframes ledbreath{0%,100%{opacity:0.15}50%{opacity:0.6}}

.floor-reflection{
  position:absolute;width:300px;height:300px;
  background:radial-gradient(circle at 50% 55%,rgba(0,240,255,0),transparent 60%);
  transform:translate3d(-60px,40px,0) rotateX(90deg);
  pointer-events:none;transition:background 0.5s ease;
}
.floor-reflection.lit{background:radial-gradient(circle at 50% 55%,rgba(0,240,255,0.14),transparent 60%)}

/* Ambient particles (GPU-friendly translate3d) */
.ambient-bg{position:absolute;inset:0;pointer-events:none;overflow:hidden;z-index:1}
.ambient-dust{position:absolute;width:5px;height:5px;background:var(--led);border-radius:50%;opacity:0;filter:blur(1px);will-change:transform, opacity}
.dust-1{left:15%;top:80%;animation:rise 15s infinite linear}
.dust-2{left:45%;top:90%;animation:rise 11s infinite linear 2s}
.dust-3{left:75%;top:85%;animation:rise 17s infinite linear 5s}
.dust-4{left:30%;top:75%;animation:rise 14s infinite linear 1s}
.dust-5{left:85%;top:70%;animation:rise 16s infinite linear 3.5s}
@keyframes rise{0%{transform:translate3d(0,0,0);opacity:0}10%{opacity:.22}90%{opacity:.22}100%{transform:translate3d(14px,-100vh,0);opacity:0}}

/* HUD */
.control-hud-panel{
  width:100%;background:var(--card);border:1px solid var(--card-border);
  border-radius:var(--radius);padding:13px 15px;
  display:flex;flex-direction:column;gap:10px;margin-bottom:14px;flex-shrink:0;
}
.hud-title{font-size:10px;color:var(--muted);text-transform:uppercase;letter-spacing:.1em;font-weight:600}
.hud-flow{display:flex;align-items:center;justify-content:space-between;position:relative}
.hud-node{display:flex;flex-direction:column;align-items:center;gap:5px;z-index:2;width:58px}
.hud-node-circle{
  width:38px;height:38px;border-radius:50%;
  background:rgba(255,255,255,0.03);border:1px solid var(--card-border);
  display:flex;align-items:center;justify-content:center;font-size:16px;
  transition:all 0.28s var(--trans);
}
.hud-node-label{font-size:8.5px;color:var(--muted);text-align:center;font-weight:600;text-transform:uppercase;white-space:nowrap}
.hud-node.active .hud-node-circle{background:rgba(0,240,255,0.1);border-color:var(--led);box-shadow:0 0 10px rgba(0,240,255,0.3);transform:scale(1.08)}
.hud-connector{flex:1;height:2px;background:rgba(255,255,255,0.05);margin:0 -6px 14px;position:relative;overflow:hidden}

/* Optimization: Changed width animation (triggers reflow) to transform scaleX (compositor-only) */
.hud-connector-pulse{
  position:absolute;top:0;left:0;height:100%;
  width:100%;background:var(--led);
  box-shadow:0 0 6px var(--led);
  transform:scaleX(0);
  transform-origin:left center;
  will-change:transform;
}

/* ACTIONS & UNIFIED ACTION BUTTON STYLE */
.main-actions{display:flex;flex-direction:column;gap:10px;width:100%;flex-shrink:0}

/* Unified action button style for all main actions and overlays */
.btn-simulate, .btn-explore-tech, .btn-explore {
  width: 100%; padding: 15px; border-radius: 14px;
  background: linear-gradient(135deg,rgba(0,240,255,0.14),rgba(37,99,255,0.1));
  border: 1px solid rgba(0,240,255,0.32);
  color: var(--led); font-size: 15px; font-weight: 700; letter-spacing: .06em; text-transform: uppercase;
  cursor: pointer; display: flex; align-items: center; justify-content: center; gap: 8px;
  /* Optimization: Specific transition properties instead of 'all' */
  transition: transform 0.22s var(--trans), background-color 0.22s var(--trans), opacity 0.22s var(--trans); font-family: inherit;
}
.btn-simulate:active, .btn-explore-tech:active, .btn-explore:active {
  transform: scale(0.97); background: rgba(0,240,255,0.22);
}
.btn-simulate:disabled, .btn-explore-tech:disabled {
  opacity: .45; pointer-events: none;
}
.btn-simulate-icon{font-size:18px}

/* ── WELCOME MODAL ──────────────────────── */
.welcome-overlay{
  position:fixed;inset:0;
  /* Optimization: Removed backdrop-filter to prevent screen layout redraw lag on mobile overlays */
  background:rgba(6,8,19,0.85);
  z-index:150;display:flex;align-items:center;justify-content:center;padding:24px;
  opacity:0;pointer-events:none;
  transition:opacity 0.38s var(--trans);
}
.welcome-overlay.show{opacity:1;pointer-events:all}
.welcome-card{
  background:rgba(14,18,40,0.9);border:1px solid var(--card-border);
  border-radius:24px;padding:32px 24px;
  width:100%;max-width:340px;text-align:center;
  transform:translate3d(0,18px,0);transition:transform 0.38s var(--trans);
  box-shadow:0 20px 50px rgba(0,0,0,0.5);
}
.welcome-overlay.show .welcome-card{transform:translate3d(0,0,0)}
.welcome-status-dot{width:8px;height:8px;border-radius:50%;background:var(--led);box-shadow:0 0 10px var(--led);margin:0 auto 18px}
.welcome-title{font-size:24px;font-weight:700;margin-bottom:10px;letter-spacing:-.02em}
.welcome-text{font-size:14px;color:var(--muted);line-height:1.6;margin-bottom:26px}

/* ── HUB ────────────────────────────────── */
#hub{background:var(--navy);padding:16px 18px 28px;overflow-y:auto}
.hub-header{display:flex;align-items:center;margin-bottom:20px;flex-shrink:0}
.btn-back-main{
  background:none;border:none;color:var(--muted);
  font-size:14px;font-weight:600;cursor:pointer;font-family:inherit;
  display:flex;align-items:center;gap:6px;padding:8px 0;transition:color .2s;
}
.btn-back-main:active{color:var(--text)}
.hub-heading-area{margin-bottom:20px}
.hub-page-title{font-size:30px;font-weight:800;letter-spacing:-.03em;margin-bottom:5px}
.hub-page-subtitle{font-size:13px;color:var(--muted)}
.hub-grid{display:flex;flex-direction:column;gap:10px}
.hub-card-item{
  background:var(--card);border:1px solid var(--card-border);
  border-radius:var(--radius);padding:18px;
  cursor:pointer;display:flex;align-items:center;justify-content:space-between;gap:16px;
  transition:all 0.18s;min-height:76px;
}
.hub-card-item:active{border-color:var(--led);transform:scale(0.985);background:rgba(0,240,255,0.025)}
.hub-card-left{display:flex;align-items:center;gap:15px}
.hub-card-icon{font-size:22px;width:44px;height:44px;border-radius:10px;background:rgba(255,255,255,0.02);display:flex;align-items:center;justify-content:center;border:1px solid rgba(255,255,255,0.05);flex-shrink:0}
.hub-card-title{font-size:15px;font-weight:700;letter-spacing:-.01em}
.hub-card-arrow{color:var(--led);font-size:14px;opacity:.65}

/* ── DETAIL SCREENS ─────────────────────── */
.detail-screen{background:var(--navy);padding:16px 18px 36px;overflow-y:auto}
.detail-nav{display:flex;align-items:center;margin-bottom:14px;flex-shrink:0}
.btn-back-hub{background:none;border:none;color:var(--muted);font-size:14px;font-weight:600;cursor:pointer;font-family:inherit;display:flex;align-items:center;gap:6px;padding:8px 0;transition:color .2s}
.btn-back-hub:active{color:var(--text)}
.detail-title{font-size:26px;font-weight:800;letter-spacing:-.02em;margin-bottom:18px}

/* WHAT */
.what-visual-container{width:100%;background:var(--card);border:1px solid var(--card-border);border-radius:var(--radius);padding:22px;margin-bottom:18px;display:flex;justify-content:center;align-items:center}
.what-tag-row{display:flex;flex-wrap:wrap;gap:8px;margin-bottom:18px}
.what-tag{background:rgba(0,240,255,0.05);border:1px solid rgba(0,240,255,0.2);color:var(--led);font-size:11px;font-weight:600;padding:6px 12px;border-radius:20px;letter-spacing:.02em}
.what-desc-para{font-size:14.5px;line-height:1.65;color:var(--muted);margin-bottom:14px}
.what-desc-para strong{color:var(--text)}

/* COMPONENTS */
.comp-list-container{display:flex;flex-direction:column;gap:10px}
.comp-card-item{background:var(--card);border:1px solid var(--card-border);border-radius:14px;padding:15px;display:flex;gap:14px;align-items:flex-start}
.comp-card-icon-box{width:42px;height:42px;border-radius:10px;background:rgba(0,240,255,0.05);border:1px solid rgba(0,240,255,0.14);display:flex;align-items:center;justify-content:center;font-size:19px;flex-shrink:0}
.comp-card-name{font-size:14px;font-weight:700;display:block;margin-bottom:4px}
.comp-card-job{font-size:12px;color:var(--muted);line-height:1.5;display:block}

/* CIRCUIT */
.circuit-visual-box{width:100%;background:var(--card);border:1px solid var(--card-border);border-radius:var(--radius);padding:8px;margin-bottom:18px;cursor:zoom-in}
.circuit-instructions{font-size:13px;color:var(--muted);line-height:1.6}
.circuit-instructions strong{color:var(--text)}
.circuit-zoom-overlay{position:fixed;inset:0;background:rgba(6,8,19,0.95);z-index:200;display:flex;flex-direction:column;justify-content:center;align-items:center;padding:16px;opacity:0;pointer-events:none;transition:opacity 0.28s}
.circuit-zoom-overlay.open{opacity:1;pointer-events:all}
.circuit-zoom-inner{width:100%;max-width:600px;display:flex;justify-content:center;align-items:center}
.btn-circuit-close{margin-top:20px;background:rgba(255,255,255,0.07);border:1px solid var(--card-border);color:var(--text);font-size:14px;font-weight:600;padding:12px 32px;border-radius:10px;cursor:pointer;font-family:inherit}

/* AIM */
.aim-card-highlight{background:rgba(0,240,255,0.04);border:1px solid rgba(0,240,255,0.22);border-radius:var(--radius);padding:18px;margin-bottom:22px}
.aim-card-label{font-size:10px;font-weight:700;text-transform:uppercase;color:var(--led);letter-spacing:.1em;margin-bottom:6px}
.aim-card-text{font-size:15px;line-height:1.6}
.obj-list-section{display:flex;flex-direction:column;gap:10px}
.obj-list-title{font-size:11px;font-weight:700;text-transform:uppercase;color:var(--muted);letter-spacing:.08em;margin-bottom:2px}
.obj-card-item{background:var(--card);border:1px solid var(--card-border);border-radius:12px;padding:13px 15px;display:flex;gap:14px;align-items:flex-start}
.obj-card-num{width:24px;height:24px;border-radius:50%;background:rgba(255,255,255,0.03);border:1px solid rgba(255,255,255,0.1);display:flex;align-items:center;justify-content:center;font-size:11px;font-weight:700;color:var(--led);flex-shrink:0}
.obj-card-desc{font-size:13px;line-height:1.5;padding-top:2px}

/* FEATURES */
.feats-container{display:flex;flex-direction:column;gap:10px}
.feat-item-card{background:var(--card);border:1px solid var(--card-border);border-radius:var(--radius);padding:17px;display:flex;gap:15px;align-items:flex-start}
.feat-item-icon{font-size:24px;flex-shrink:0}
.feat-item-title{font-size:14px;font-weight:700;margin-bottom:4px;display:block}
.feat-item-desc{font-size:12.5px;color:var(--muted);line-height:1.5;display:block}

/* WORKFLOW */
.workflow-diagram-container{width:100%;background:var(--card);border:1px solid var(--card-border);border-radius:var(--radius);padding:22px 14px;margin-bottom:18px;display:flex;justify-content:center}
.workflow-notes-container{display:flex;flex-direction:column;gap:10px}
.wf-note-card{border-left:2px solid rgba(0,240,255,0.3);padding-left:12px}
.wf-note-title{font-size:13px;font-weight:700;margin-bottom:2px;display:block}
.wf-note-desc{font-size:11.5px;color:var(--muted);line-height:1.45;display:block}

/* ADVANTAGES */
.advs-container{display:flex;flex-direction:column;gap:10px}
.adv-item-card{background:var(--card);border:1px solid var(--card-border);border-radius:var(--radius);padding:17px;display:flex;gap:13px;align-items:flex-start}
.adv-item-dot{width:7px;height:7px;border-radius:50%;background:var(--led);box-shadow:0 0 6px var(--led);margin-top:5px;flex-shrink:0}
.adv-item-title{font-size:14px;font-weight:700;margin-bottom:4px;display:block}
.adv-item-desc{font-size:12.5px;color:var(--muted);line-height:1.5;display:block}

/* SCROLLBAR */
::-webkit-scrollbar{width:4px}::-webkit-scrollbar-track{background:transparent}::-webkit-scrollbar-thumb{background:rgba(255,255,255,0.1);border-radius:2px}

/* ── DESKTOP PHONE FRAME (fixed for overlay positioning) ─── */
@media (min-width:500px){
  body{display:flex;justify-content:center;align-items:center;background:#030408}
  #phone-frame{
    position:relative;width:390px;height:844px;
    border-radius:40px;border:8px solid #1f2235;
    box-shadow:0 25px 60px rgba(0,0,0,0.8);
    overflow:hidden;flex-shrink:0;
    background:var(--navy);
  }
  .screen,#intro-overlay,.welcome-overlay,.circuit-zoom-overlay{
    position:absolute;
    border-radius:32px;
  }
}
</style>
</head>
<body>

<!-- Phone frame wrapper (desktop only) -->
<div id="phone-frame">

<!-- Ambient particles -->
<div class="ambient-bg">
  <div class="ambient-dust dust-1"></div>
  <div class="ambient-dust dust-2"></div>
  <div class="ambient-dust dust-3"></div>
  <div class="ambient-dust dust-4"></div>
  <div class="ambient-dust dust-5"></div>
</div>

<!-- ══ INTRO OVERLAY ══ -->
<div id="intro-overlay">
  <div class="intro-glow ig1"></div>
  <div class="intro-glow ig2"></div>
  <div class="intro-brand">Computer Exhibition 2025</div>
  <div class="intro-title">Smart<br><span>Stairs</span></div>
  <p class="intro-desc">An interactive simulation of sensor-controlled automated staircase illumination.</p>
  <button class="play-btn" id="playBtn" aria-label="Enter Exhibition">
    <div class="play-icon"></div>
  </button>
  <div class="play-label">Tap to begin</div>
</div>

<!-- ══ MAIN SCREEN ══ -->
<div id="main" class="screen active">
  <div class="main-header">
    <div class="brand-title">Smart<span>Stairs</span></div>
    <div class="status-pill ready" id="systemStatus">
      <div class="status-dot"></div>
      <span id="statusText">System Ready</span>
    </div>
  </div>

  <div class="scene-outer-container" id="staircaseOuter">
    <div class="staircase-viewport" id="viewport">
      <div class="staircase-scene intro-blur" id="scene">
        <div class="floor-reflection" id="floorLight"></div>
        <!-- Step 0 (bottom) -->
        <div class="step-3d" style="transform:translate3d(0,0px,0px)">
          <div class="face riser"></div><div class="face tread"></div>
          <div class="face side-left"></div><div class="face side-right"></div>
          <div class="led-strip"></div>
        </div>
        <!-- Step 1 -->
        <div class="step-3d" style="transform:translate3d(0,-24px,-32px)">
          <div class="face riser"></div><div class="face tread"></div>
          <div class="face side-left"></div><div class="face side-right"></div>
          <div class="led-strip"></div>
        </div>
        <!-- Step 2 -->
        <div class="step-3d" style="transform:translate3d(0,-48px,-64px)">
          <div class="face riser"></div><div class="face tread"></div>
          <div class="face side-left"></div><div class="face side-right"></div>
          <div class="led-strip"></div>
        </div>
        <!-- Step 3 -->
        <div class="step-3d" style="transform:translate3d(0,-72px,-96px)">
          <div class="face riser"></div><div class="face tread"></div>
          <div class="face side-left"></div><div class="face side-right"></div>
          <div class="led-strip"></div>
        </div>
        <!-- Step 4 -->
        <div class="step-3d" style="transform:translate3d(0,-96px,-128px)">
          <div class="face riser"></div><div class="face tread"></div>
          <div class="face side-left"></div><div class="face side-right"></div>
          <div class="led-strip"></div>
        </div>
        <!-- Step 5 (top) -->
        <div class="step-3d" style="transform:translate3d(0,-120px,-160px)">
          <div class="face riser"></div><div class="face tread"></div>
          <div class="face side-left"></div><div class="face side-right"></div>
          <div class="led-strip"></div>
        </div>
      </div>
    </div>
  </div>

  <div class="control-hud-panel">
    <div class="hud-title">System Control Chain</div>
    <div class="hud-flow">
      <div class="hud-node" id="node-ir">
        <div class="hud-node-circle">📡</div>
        <div class="hud-node-label">IR Sensor</div>
      </div>
      <div class="hud-connector"><div class="hud-connector-pulse" id="pulse-1"></div></div>
      <div class="hud-node" id="node-arduino">
        <div class="hud-node-circle">🤖</div>
        <div class="hud-node-label">Arduino</div>
      </div>
      <div class="hud-connector"><div class="hud-connector-pulse" id="pulse-2"></div></div>
      <div class="hud-node" id="node-mosfet">
        <div class="hud-node-circle">🔌</div>
        <div class="hud-node-label">MOSFETs</div>
      </div>
      <div class="hud-connector"><div class="hud-connector-pulse" id="pulse-3"></div></div>
      <div class="hud-node" id="node-leds">
        <div class="hud-node-circle">💡</div>
        <div class="hud-node-label">12V LEDs</div>
      </div>
    </div>
  </div>

  <div class="main-actions">
    <button class="btn-simulate" id="btnSimulate">
      <span class="btn-simulate-icon">👣</span>
      Simulate Detection
    </button>
    <button class="btn-explore-tech" id="btnExploreTech">
      📚 Explore Technology
    </button>
  </div>
</div>

<!-- ══ WELCOME OVERLAY ══ -->
<div class="welcome-overlay" id="welcomeOverlay">
  <div class="welcome-card">
    <div class="welcome-status-dot"></div>
    <h2 class="welcome-title">Welcome!</h2>
    <p class="welcome-text">Are you ready to climb the Smart Stairs?<br>Let's see what they're made of.</p>
    <button class="btn-explore" id="exploreBtn">
      Explore the Technology →
    </button>
  </div>
</div>

<!-- ══ HUB ══ -->
<div id="hub" class="screen">
  <div class="hub-header">
    <button class="btn-back-main" onclick="backToMain()">← Back to Simulator</button>
  </div>
  <div class="hub-heading-area">
    <h1 class="hub-page-title">Technology Hub</h1>
    <p class="hub-page-subtitle">Explore the components, circuits, and logic.</p>
  </div>
  <div class="hub-grid">
    <div class="hub-card-item" onclick="openDetail('what')"><div class="hub-card-left"><div class="hub-card-icon">🏗️</div><div class="hub-card-title">What Are Smart Stairs?</div></div><div class="hub-card-arrow">→</div></div>
    <div class="hub-card-item" onclick="openDetail('components')"><div class="hub-card-left"><div class="hub-card-icon">🔧</div><div class="hub-card-title">Components</div></div><div class="hub-card-arrow">→</div></div>
    <div class="hub-card-item" onclick="openDetail('circuit')"><div class="hub-card-left"><div class="hub-card-icon">⚡</div><div class="hub-card-title">Circuit Diagram</div></div><div class="hub-card-arrow">→</div></div>
    <div class="hub-card-item" onclick="openDetail('aim')"><div class="hub-card-left"><div class="hub-card-icon">🎯</div><div class="hub-card-title">Aim & Objectives</div></div><div class="hub-card-arrow">→</div></div>
    <div class="hub-card-item" onclick="openDetail('features')"><div class="hub-card-left"><div class="hub-card-icon">✨</div><div class="hub-card-title">Features</div></div><div class="hub-card-arrow">→</div></div>
    <div class="hub-card-item" onclick="openDetail('workflow')"><div class="hub-card-left"><div class="hub-card-icon">🔄</div><div class="hub-card-title">Workflow</div></div><div class="hub-card-arrow">→</div></div>
    <div class="hub-card-item" onclick="openDetail('advantages')"><div class="hub-card-left"><div class="hub-card-icon">💡</div><div class="hub-card-title">Advantages</div></div><div class="hub-card-arrow">→</div></div>
  </div>
</div>

<!-- ══ WHAT ARE SMART STAIRS ══ -->
<div id="d-what" class="screen detail-screen">
  <div class="detail-nav"><button class="btn-back-hub" onclick="backToHub()">← Back to Hub</button></div>
  <h1 class="detail-title">What Are Smart Stairs?</h1>
  <div class="what-visual-container">
    <svg width="120" height="90" viewBox="0 0 120 90" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path d="M10 80H110" stroke="#f0f4ff" stroke-width="2" stroke-linecap="round"/>
      <path d="M20 80V65H40V50H60V35H80V20H100" stroke="#f0f4ff" stroke-width="2" stroke-linejoin="round"/>
      <circle cx="100" cy="20" r="4" fill="#00f0ff"/>
      <circle cx="15" cy="50" r="5" fill="#ff9f1c"/>
      <path d="M20 50H50" stroke="#ff9f1c" stroke-width="1.5" stroke-dasharray="3 3"/>
      <line x1="40" y1="65" x2="40" y2="64" stroke="#00f0ff" stroke-width="3" opacity=".7"/>
      <line x1="60" y1="50" x2="60" y2="49" stroke="#00f0ff" stroke-width="3" opacity=".6"/>
      <line x1="80" y1="35" x2="80" y2="34" stroke="#00f0ff" stroke-width="3" opacity=".5"/>
    </svg>
  </div>
  <div class="what-tag-row">
    <span class="what-tag">🔦 Automated Response</span>
    <span class="what-tag">📡 Motion Detection</span>
    <span class="what-tag">🧠 Embedded Logic</span>
  </div>
  <p class="what-desc-para"><strong>Smart Stairs</strong> are an automated staircase-lighting system that uses sensors and a microcontroller to detect movement and control LED lighting — without any manual switching.</p>
  <p class="what-desc-para">An infrared sensor at the foot of the stairs continuously monitors for movement. When someone enters the detection zone, it sends a digital trigger to an Arduino Nano.</p>
  <p class="what-desc-para">The Arduino executes its control logic, activating MOSFETs in sequence to illuminate each step's LED strip — lighting the path automatically. Once the sequence completes, the system returns to standby.</p>
</div>

<!-- ══ COMPONENTS ══ -->
<div id="d-components" class="screen detail-screen">
  <div class="detail-nav"><button class="btn-back-hub" onclick="backToHub()">← Back to Hub</button></div>
  <h1 class="detail-title">Components</h1>
  <div class="comp-list-container">
    <div class="comp-card-item"><div class="comp-card-icon-box">🧠</div><div><span class="comp-card-name">Arduino Nano</span><span class="comp-card-job">The microcontroller brain. Reads IR sensor input, runs sequence delay logic, and writes output triggers to MOSFET gates.</span></div></div>
    <div class="comp-card-item"><div class="comp-card-icon-box">📡</div><div><span class="comp-card-name">IR Proximity Sensor</span><span class="comp-card-job">Scans the staircase base. Emits infrared light and detects reflection, sending HIGH to the Arduino when a visitor approaches.</span></div></div>
    <div class="comp-card-item"><div class="comp-card-icon-box">🔌</div><div><span class="comp-card-name">N-Channel MOSFETs (Logic-Level)</span><span class="comp-card-job">Solid-state switch gates. Let the low-power 5V Arduino cleanly control high-power 12V LED loads without relays.</span></div></div>
    <div class="comp-card-item"><div class="comp-card-icon-box">💡</div><div><span class="comp-card-name">12V LED Blocks</span><span class="comp-card-job">High-density LED strips attached to the front lip of each step, projecting light down onto the tread below.</span></div></div>
    <div class="comp-card-item"><div class="comp-card-icon-box">🔋</div><div><span class="comp-card-name">12V DC 2A Power Adapter</span><span class="comp-card-job">Powers the LED blocks via a shared rail. Arduino is powered separately via USB connection.</span></div></div>
    <div class="comp-card-item"><div class="comp-card-icon-box">⚡</div><div><span class="comp-card-name">Resistors (220Ω & 10kΩ)</span><span class="comp-card-job">220Ω limits gate current; 10kΩ pull-downs prevent floating logic states and ensure clean LOW switching.</span></div></div>
    <div class="comp-card-item"><div class="comp-card-icon-box">🧩</div><div><span class="comp-card-name">Breadboard & Jumper Wires</span><span class="comp-card-job">Prototyping surface and connecting wires used to assemble the full circuit without soldering.</span></div></div>
    <div class="comp-card-item"><div class="comp-card-icon-box">🔗</div><div><span class="comp-card-name">USB Cable for Arduino Nano</span><span class="comp-card-job">Used to upload code to the Arduino and powers the microcontroller during operation.</span></div></div>
  </div>
</div>

<!-- ══ CIRCUIT DIAGRAM ══ -->
<div id="d-circuit" class="screen detail-screen">
  <div class="detail-nav"><button class="btn-back-hub" onclick="backToHub()">← Back to Hub</button></div>
  <h1 class="detail-title">Circuit Schematic</h1>
  <div class="circuit-visual-box" onclick="zoomCircuit(true)">
    <svg viewBox="0 0 320 220" width="100%" height="100%" xmlns="http://www.w3.org/2000/svg" style="background:#0b0e22;border-radius:10px">
      <defs><pattern id="cg" width="16" height="16" patternUnits="userSpaceOnUse"><path d="M16 0L0 0 0 16" fill="none" stroke="rgba(255,255,255,0.03)" stroke-width="1"/></pattern></defs>
      <rect width="100%" height="100%" fill="url(#cg)"/>
      <line x1="20" y1="20" x2="300" y2="20" stroke="#ef4444" stroke-width="2" stroke-linecap="round"/>
      <text x="298" y="15" fill="#ef4444" font-size="7" font-weight="700" text-anchor="end">+12V Rail</text>
      <line x1="20" y1="200" x2="300" y2="200" stroke="#10b981" stroke-width="2" stroke-linecap="round"/>
      <text x="298" y="210" fill="#10b981" font-size="7" font-weight="700" text-anchor="end">Common GND</text>
      <rect x="110" y="60" width="70" height="90" rx="3" fill="#111632" stroke="#2563ff" stroke-width="1.5"/>
      <text x="145" y="80" fill="#2563ff" font-size="8" font-weight="800" text-anchor="middle">ARDUINO</text>
      <text x="145" y="90" fill="#f0f4ff" font-size="7" text-anchor="middle">NANO</text>
      <text x="115" y="110" fill="#00f0ff" font-size="6" font-family="monospace">D2 (IN)</text>
      <text x="115" y="122" fill="#00f0ff" font-size="6" font-family="monospace">D3-D8 OUT</text>
      <text x="115" y="134" fill="#10b981" font-size="6" font-family="monospace">GND</text>
      <rect x="25" y="80" width="40" height="40" rx="2" fill="#111632" stroke="#ff9f1c" stroke-width="1"/>
      <text x="45" y="97" fill="#ff9f1c" font-size="7" font-weight="700" text-anchor="middle">IR</text>
      <text x="45" y="107" fill="#ff9f1c" font-size="6" text-anchor="middle">SENSOR</text>
      <path d="M65 100 L110 110" stroke="#ff9f1c" stroke-dasharray="2 2" stroke-width="1" fill="none"/>
      <rect x="220" y="68" width="36" height="30" rx="2" fill="#111632" stroke="#00f0ff" stroke-width="1"/>
      <text x="238" y="82" fill="#00f0ff" font-size="6" font-weight="700" text-anchor="middle">MOSFET</text>
      <text x="238" y="91" fill="#8e9db4" font-size="5.5" text-anchor="middle">x6 Array</text>
      <path d="M180 122 L200 122 L200 83 L220 83" stroke="#00f0ff" stroke-width="1" fill="none"/>
      <rect x="220" y="130" width="36" height="25" rx="2" fill="#111632" stroke="#f0f4ff" stroke-width="1"/>
      <text x="238" y="142" fill="#f0f4ff" font-size="6" font-weight="700" text-anchor="middle">LEDs</text>
      <text x="238" y="150" fill="#8e9db4" font-size="5.5" text-anchor="middle">x6 load</text>
      <path d="M238 130 L238 20" stroke="#ef4444" stroke-width="1" fill="none"/>
      <path d="M238 155 L238 170 L252 170 L252 98" stroke="#f0f4ff" stroke-dasharray="2 1" stroke-width="1" fill="none"/>
      <path d="M224 98 L224 200" stroke="#10b981" stroke-width="1" fill="none"/>
      <path d="M145 150 L145 200" stroke="#10b981" stroke-width="1" fill="none"/>
      <text x="160" y="215" fill="rgba(142,157,180,0.6)" font-size="7" font-family="monospace" text-anchor="middle">Tap to enlarge</text>
    </svg>
  </div>
  <p class="circuit-instructions"><strong>Tap the diagram to enlarge.</strong> The schematic shows the 12V power rail driving LED loads, with MOSFETs isolating the high-voltage loads from the 5V Arduino logic. Pull-down resistors on each gate prevent floating inputs.</p>
</div>
<div class="circuit-zoom-overlay" id="circuitZoom">
  <div class="circuit-zoom-inner" id="zoomInner"></div>
  <button class="btn-circuit-close" onclick="zoomCircuit(false)">Close Diagram</button>
</div>

<!-- ══ AIM & OBJECTIVES ══ -->
<div id="d-aim" class="screen detail-screen">
  <div class="detail-nav"><button class="btn-back-hub" onclick="backToHub()">← Back to Hub</button></div>
  <h1 class="detail-title">Aim & Objectives</h1>
  <div class="aim-card-highlight">
    <div class="aim-card-label">Main Aim</div>
    <p class="aim-card-text">To design and demonstrate an automated staircase-lighting system that detects movement and activates LED lighting without requiring manual switching.</p>
  </div>
  <div class="obj-list-section">
    <div class="obj-list-title">Core Objectives</div>
    <div class="obj-card-item"><div class="obj-card-num">1</div><p class="obj-card-desc">Detect movement using an IR sensor.</p></div>
    <div class="obj-card-item"><div class="obj-card-num">2</div><p class="obj-card-desc">Process the sensor input using an Arduino Nano.</p></div>
    <div class="obj-card-item"><div class="obj-card-num">3</div><p class="obj-card-desc">Control the staircase LEDs using MOSFET switching.</p></div>
    <div class="obj-card-item"><div class="obj-card-num">4</div><p class="obj-card-desc">Illuminate the staircase automatically when movement is detected.</p></div>
    <div class="obj-card-item"><div class="obj-card-num">5</div><p class="obj-card-desc">Demonstrate the practical use of embedded systems and automation.</p></div>
    <div class="obj-card-item"><div class="obj-card-num">6</div><p class="obj-card-desc">Create a more convenient and responsive staircase-lighting system.</p></div>
  </div>
</div>

<!-- ══ FEATURES ══ -->
<div id="d-features" class="screen detail-screen">
  <div class="detail-nav"><button class="btn-back-hub" onclick="backToHub()">← Back to Hub</button></div>
  <h1 class="detail-title">Key Features</h1>
  <div class="feats-container">
    <div class="feat-item-card"><div class="feat-item-icon">⚡</div><div><span class="feat-item-title">Sequential Illumination</span><span class="feat-item-desc">Stairs activate in a flowing sequence, guiding the user up the path and creating a smooth visual transition.</span></div></div>
    <div class="feat-item-card"><div class="feat-item-icon">🎛️</div><div><span class="feat-item-title">Solid State Switching</span><span class="feat-item-desc">MOSFET network runs completely silent with zero mechanical contact wear — no relays needed.</span></div></div>
    <div class="feat-item-card"><div class="feat-item-icon">💡</div><div><span class="feat-item-title">Under-Tread Light Wash</span><span class="feat-item-desc">LED placement projects light downward onto treads, providing visibility exactly where needed.</span></div></div>
    <div class="feat-item-card"><div class="feat-item-icon">🤲</div><div><span class="feat-item-title">Hands-Free Operation</span><span class="feat-item-desc">No switch or button needed. Walking toward the stairs is the only input required.</span></div></div>
    <div class="feat-item-card"><div class="feat-item-icon">🌿</div><div><span class="feat-item-title">Auto-Off Standby</span><span class="feat-item-desc">After the lighting sequence completes, LEDs return to standby — no manual off required.</span></div></div>
  </div>
</div>

<!-- ══ WORKFLOW ══ -->
<div id="d-workflow" class="screen detail-screen">
  <div class="detail-nav"><button class="btn-back-hub" onclick="backToHub()">← Back to Hub</button></div>
  <h1 class="detail-title">Workflow Logic</h1>
  <div class="workflow-diagram-container">
    <svg width="260" height="240" viewBox="0 0 260 240" fill="none" xmlns="http://www.w3.org/2000/svg">
      <style>
        @keyframes wf-drop {
          0%{transform:translateY(-14px);opacity:0}
          30%{opacity:1}70%{opacity:1}
          100%{transform:translateY(14px);opacity:0}
        }
        .wfd1{animation:wf-drop 2.2s infinite linear}
        .wfd2{animation:wf-drop 2.2s infinite linear .55s}
        .wfd3{animation:wf-drop 2.2s infinite linear 1.1s}
      </style>
      <rect x="55" y="5" width="150" height="36" rx="7" fill="#111632" stroke="#ff9f1c" stroke-width="1.5"/>
      <text x="130" y="28" fill="#ff9f1c" font-size="11" font-weight="700" text-anchor="middle">1. IR SENSOR</text>
      <line x1="130" y1="41" x2="130" y2="72" stroke="rgba(255,255,255,0.1)" stroke-width="1.5"/>
      <circle cx="130" cy="57" r="3.5" fill="#ff9f1c" class="wfd1"/>
      <rect x="55" y="72" width="150" height="36" rx="7" fill="#111632" stroke="#2563ff" stroke-width="1.5"/>
      <text x="130" y="95" fill="#f0f4ff" font-size="11" font-weight="700" text-anchor="middle">2. ARDUINO NANO</text>
      <line x1="130" y1="108" x2="130" y2="140" stroke="rgba(255,255,255,0.1)" stroke-width="1.5"/>
      <circle cx="130" cy="124" r="3.5" fill="#2563ff" class="wfd2"/>
      <rect x="55" y="140" width="150" height="36" rx="7" fill="#111632" stroke="#00f0ff" stroke-width="1.5"/>
      <text x="130" y="163" fill="#00f0ff" font-size="11" font-weight="700" text-anchor="middle">3. MOSFETs</text>
      <line x1="130" y1="176" x2="130" y2="207" stroke="rgba(255,255,255,0.1)" stroke-width="1.5"/>
      <circle cx="130" cy="192" r="3.5" fill="#00f0ff" class="wfd3"/>
      <rect x="55" y="207" width="150" height="30" rx="7" fill="#111632" stroke="#f0f4ff" stroke-width="1.5"/>
      <text x="130" y="226" fill="#f0f4ff" font-size="11" font-weight="700" text-anchor="middle">4. LED LIGHTS</text>
    </svg>
  </div>
  <div class="workflow-notes-container">
    <div class="wf-note-card"><span class="wf-note-title">IR Trigger Input</span><span class="wf-note-desc">Digital reflection reads HIGH → microcontroller processes the input transition state.</span></div>
    <div class="wf-note-card"><span class="wf-note-title">Gate Voltage Transfer</span><span class="wf-note-desc">Arduino writes 5V HIGH to MOSFET gate → charges gate capacitance → opens the drain-source channel.</span></div>
    <div class="wf-note-card"><span class="wf-note-title">Load Switch (12V)</span><span class="wf-note-desc">MOSFET channel connects LED ground to Common GND → 12V LED loads run cleanly under isolation.</span></div>
  </div>
</div>

<!-- ══ ADVANTAGES ══ -->
<div id="d-advantages" class="screen detail-screen">
  <div class="detail-nav"><button class="btn-back-hub" onclick="backToHub()">← Back to Hub</button></div>
  <h1 class="detail-title">System Advantages</h1>
  <div class="advs-container">
    <div class="adv-item-card"><div class="adv-item-dot"></div><div><span class="adv-item-title">Convenience</span><span class="adv-item-desc">Automatic lighting removes the need to locate and flip a switch — useful when hands are full or visibility is low.</span></div></div>
    <div class="adv-item-card"><div class="adv-item-dot"></div><div><span class="adv-item-title">Automation</span><span class="adv-item-desc">The system responds automatically to detected movement, demonstrating a fully self-contained automated response loop.</span></div></div>
    <div class="adv-item-card"><div class="adv-item-dot"></div><div><span class="adv-item-title">Visibility</span><span class="adv-item-desc">Illuminated steps improve visibility of the staircase while it is actively being used.</span></div></div>
    <div class="adv-item-card"><div class="adv-item-dot"></div><div><span class="adv-item-title">Silent Switching</span><span class="adv-item-desc">Without mechanical relays, the system operates in total silence — ideal for nighttime use.</span></div></div>
    <div class="adv-item-card"><div class="adv-item-dot"></div><div><span class="adv-item-title">Technology Demonstration</span><span class="adv-item-desc">Illustrates how sensors, microcontrollers, and electronic switching work together in a practical embedded system.</span></div></div>
  </div>
</div>

</div><!-- /phone-frame -->

<script>
// ══════════════════════════════════════════════
// NAVIGATION
// ══════════════════════════════════════════════
function navigateTo(id) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  const t = document.getElementById(id);
  if (t) { t.classList.add('active'); t.scrollTop = 0; }
}
function backToMain() { navigateTo('main'); }
function backToHub()  { navigateTo('hub'); }
function openDetail(id) { navigateTo('d-' + id); }

// ══════════════════════════════════════════════
// INTRO OVERLAY
// ══════════════════════════════════════════════
const introOverlay = document.getElementById('intro-overlay');
const scene = document.getElementById('scene');

document.getElementById('playBtn').addEventListener('click', () => {
  introOverlay.classList.add('fade-out');
  scene.classList.add('unblur');
  setTimeout(() => { introOverlay.style.display = 'none'; }, 800);
});

// ══════════════════════════════════════════════
// WELCOME OVERLAY
// ══════════════════════════════════════════════
const welcomeOverlay = document.getElementById('welcomeOverlay');
const btnExploreTech = document.getElementById('btnExploreTech');
let welcomeShown = false;

// Clicking Explore Technology on main page
btnExploreTech.addEventListener('click', () => {
  if (simulationActive) return;
  if (!welcomeShown) {
    welcomeOverlay.classList.add('show');
    welcomeShown = true;
  } else {
    navigateTo('hub');
  }
});

// Tapping the staircase viewport
document.getElementById('staircaseOuter').addEventListener('click', (e) => {
  if (simulationActive) return;
  if (!welcomeShown) {
    welcomeOverlay.classList.add('show');
    welcomeShown = true;
  }
});

document.getElementById('exploreBtn').addEventListener('click', () => {
  welcomeOverlay.classList.remove('show');
  setTimeout(() => navigateTo('hub'), 350);
});

// ══════════════════════════════════════════════
// SIMULATION
// ══════════════════════════════════════════════
const btnSimulate = document.getElementById('btnSimulate');
const statusPill  = document.getElementById('systemStatus');
const statusText  = document.getElementById('statusText');
const steps       = document.querySelectorAll('.step-3d');
const floorLight  = document.getElementById('floorLight');
const nodeIR      = document.getElementById('node-ir');
const nodeArduino = document.getElementById('node-arduino');
const nodeMosfet  = document.getElementById('node-mosfet');
const nodeLEDs    = document.getElementById('node-leds');
const pulse1      = document.getElementById('pulse-1');
const pulse2      = document.getElementById('pulse-2');
const pulse3      = document.getElementById('pulse-3');

let simulationActive = false;

function setStatus(txt, cls) { statusText.textContent = txt; statusPill.className = 'status-pill ' + cls; }

btnSimulate.addEventListener('click', () => { if (!simulationActive) runSim(); });

function runSim() {
  simulationActive = true;
  btnSimulate.disabled = true;
  btnExploreTech.disabled = true;

  // Add simulating class to pause ambient CSS animations on the GPU
  scene.classList.add('simulation-active');

  // Reset
  steps.forEach(s => s.classList.remove('lit','lit-below'));
  floorLight.classList.remove('lit');
  document.querySelectorAll('.hud-node').forEach(n => n.classList.remove('active'));
  [pulse1,pulse2,pulse3].forEach(p => { p.style.transition = 'none'; p.style.transform = 'scaleX(0)'; });

  // Stage 1: IR
  setStatus('Movement Detected','detected');
  nodeIR.classList.add('active');

  // Optimization: Pulse transitions use compositor scaleX instead of width reflows
  setTimeout(() => { pulse1.style.transition = 'transform 0.4s linear'; pulse1.style.transform = 'scaleX(1)'; }, 400);
  setTimeout(() => { setStatus('Processing','processing'); nodeArduino.classList.add('active'); }, 800);
  setTimeout(() => { pulse2.style.transition = 'transform 0.4s linear'; pulse2.style.transform = 'scaleX(1)'; }, 1200);
  setTimeout(() => { setStatus('Lighting Active','active-state'); nodeMosfet.classList.add('active'); }, 1600);
  setTimeout(() => { pulse3.style.transition = 'transform 0.3s linear'; pulse3.style.transform = 'scaleX(1)'; nodeLEDs.classList.add('active'); }, 1900);

  // Step illumination
  const startLED = 2200, stepGap = 260;
  steps.forEach((step, i) => {
    setTimeout(() => {
      step.classList.add('lit');
      if (i > 0) steps[i-1].classList.add('lit-below');
      else floorLight.classList.add('lit');
    }, startLED + i * stepGap);
  });

  const holdEnd = startLED + steps.length * stepGap + 3200;
  const fadeDur = 1200;

  setTimeout(() => {
    steps.forEach(s => { s.style.transition = `opacity ${fadeDur}ms ease, background ${fadeDur}ms ease`; });
    floorLight.style.transition = `background ${fadeDur}ms ease`;
    
    steps.forEach(s => s.classList.remove('lit','lit-below'));
    floorLight.classList.remove('lit');
    document.querySelectorAll('.hud-node').forEach(n => n.classList.remove('active'));
    
    [pulse1,pulse2,pulse3].forEach(p => { p.style.transition = 'transform 0.8s ease'; p.style.transform = 'scaleX(0)'; });
    setStatus('System Ready','ready');
  }, holdEnd);

  setTimeout(() => {
    steps.forEach(s => { s.style.transition = ''; });
    floorLight.style.transition = '';
    
    // Remove simulating class to resume GPU idle breathing loops cleanly
    scene.classList.remove('simulation-active');
    
    simulationActive = false;
    btnSimulate.disabled = false;
    btnExploreTech.disabled = false;
  }, holdEnd + fadeDur + 200);
}

// ══════════════════════════════════════════════
// CIRCUIT ZOOM
// ══════════════════════════════════════════════
function zoomCircuit(open) {
  const overlay = document.getElementById('circuitZoom');
  if (open) {
    const svg = document.querySelector('.circuit-visual-box svg');
    if (svg) {
      document.getElementById('zoomInner').innerHTML = '';
      const clone = svg.cloneNode(true);
      clone.setAttribute('width','100%');
      clone.style.background = '#0b0e22';
      clone.style.borderRadius = '12px';
      document.getElementById('zoomInner').appendChild(clone);
    }
    overlay.classList.add('open');
  } else {
    overlay.classList.remove('open');
  }
}
</script>
</body>
</html>

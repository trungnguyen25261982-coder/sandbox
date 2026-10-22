# Particle Lab — Tiny Worlds, Big Chaos

A self-contained, browser-based falling-particle sandbox. Pick a material, paint it into the canvas, and watch particles fall, flow, heat up, burn, cool, react, and grow. The implementation is original and dependency-free; it recreates the **kind of play loop** found in falling-sand sandboxes, not Neal.fun/Sandboxels source code, assets, or branding.

> **Important GitHub limitation:** GitHub renders `README.md` as Markdown and does not execute JavaScript embedded in it. This repository can contain the complete source in this one README, but to run it locally you must copy the `html` code block below into a file named `index.html`. To host a playable page with GitHub Pages, commit that `index.html` as well. A README-only repository cannot itself run the simulation on its GitHub page.

## Quick start

1. Create a GitHub repository and add this `README.md`.
2. Copy the full source from the **Single-file app** code block into `index.html` (the code is plain HTML, CSS, and JavaScript; there are no packages to install).
3. Open `index.html` in a modern browser, or commit it and enable **Settings → Pages** to publish it.
4. Start with Sand, Water, Wall, Wood, Fire, and Plant. Try drawing a wall, piling sand, pouring water, then introducing heat.

## Included

- **Materials:** powders, liquids, solids, energy, gases, and life.
- **Simulation:** falling powders, flowing liquids, rising gases, density swaps, local heat, melting/freezing/evaporation, combustion, acid erosion, and simple plant growth.
- **Tools:** draw, erase, heat, cool, sample, replace mode, adjustable brush, line drawing, pause, single-step, undo/redo, and reset.
- **World controls:** speed and ambient temperature sliders, optional grid and color variation, live material counts, PNG capture, browser save/load, and JSON export/import.
- **Controls:** mouse/touch drawing; right-click erases; middle-click samples; Shift-drag draws a line; mouse wheel changes brush size; Space/P pauses; `[`/`]` adjusts brush; Ctrl/Cmd+Z undoes; Ctrl/Cmd+Shift+Z redoes; C captures a PNG; F toggles fullscreen; E or `/` focuses search; R resets.
- **Privacy:** no external scripts, analytics, backend, or network requests. Browser saves stay in local storage on the device.

## Design notes

The simulation uses a compact 180 × 100 cellular grid and a canvas rendered at pixel scale. Each tick updates cells bottom-up and alternates horizontal scan direction to reduce directional bias. Materials have small, explicit behavior rules that are intentionally readable and easy to extend.

### Extend it

In `index.html`, add a material object to `definitions` near the top of the script. Define `key`, `name`, `color`, `category`, `state`, and `description`; optional fields include `density`, `flammable`, `hot`, and `life`. Add special behavior to `simulateParticle()` if the material needs a unique reaction. Keep the material key unique.

## Single-file app

Copy everything in this code block (including the `<!doctype html>` line) into `index.html`.

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="theme-color" content="#111519">
<title>Particle Lab — Tiny Worlds</title>
<style>
:root{color-scheme:dark;--bg:#111519;--panel:#191f24;--panel2:#202830;--line:#303b44;--muted:#8d9aa5;--text:#ecf2f6;--accent:#d4f36a;--accent-ink:#17200a;--danger:#ff7777;--radius:12px;font-family:Inter,ui-sans-serif,system-ui,-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif}
*{box-sizing:border-box}html,body{height:100%;margin:0;background:var(--bg);color:var(--text)}body{min-width:320px}button,input{font:inherit}button{cursor:pointer}button:focus-visible,input:focus-visible{outline:2px solid var(--accent);outline-offset:2px}.app{height:100%;min-height:620px;display:flex;flex-direction:column;max-width:1800px;margin:auto;padding:14px;gap:12px}.topbar{display:flex;align-items:center;gap:18px;min-height:58px;border:1px solid var(--line);background:#171c21;border-radius:var(--radius);padding:10px 14px}.brand{display:flex;align-items:center;gap:10px;min-width:205px}.brand-mark{width:34px;height:34px;border-radius:9px;background:var(--accent);display:grid;place-items:center;color:#202a0b;font-size:19px;font-weight:1000;letter-spacing:-7px;padding-right:6px;box-shadow:0 0 24px #d4f36a22}.brand-title{font-weight:900;letter-spacing:.11em;font-size:14px}.brand-sub{font-size:9px;letter-spacing:.18em;color:var(--muted);margin-top:2px}.world-pulse{display:flex;align-items:center;gap:8px;color:#b9c6ce;font-size:11px;letter-spacing:.09em;white-space:nowrap}.pulse{width:7px;height:7px;background:var(--accent);border-radius:50%;box-shadow:0 0 0 4px #d4f36a18}.top-actions{margin-left:auto;display:flex;gap:7px;align-items:center}.btn{border:1px solid var(--line);background:var(--panel2);color:var(--text);border-radius:8px;padding:8px 10px;font-size:11px;font-weight:700;transition:background .12s,border-color .12s,transform .12s;white-space:nowrap}.btn:hover{border-color:#596a77;background:#2b3540}.btn:active{transform:translateY(1px)}.btn.primary{background:var(--accent);border-color:var(--accent);color:var(--accent-ink)}.btn.active{background:#d4f36a1d;border-color:var(--accent);color:var(--accent)}.btn.danger:hover{border-color:var(--danger);color:var(--danger)}.layout{display:grid;grid-template-columns:232px minmax(0,1fr) 224px;gap:12px;min-height:0;flex:1}.panel{border:1px solid var(--line);border-radius:var(--radius);background:var(--panel);min-height:0;overflow:hidden;display:flex;flex-direction:column}.panel-head{display:flex;align-items:center;justify-content:space-between;padding:15px 14px 10px;font-size:10px;font-weight:900;letter-spacing:.13em;color:#c3cdd4}.eyebrow{font-size:9px;color:var(--muted);letter-spacing:.13em;font-weight:800}.count-badge{border:1px solid var(--line);border-radius:6px;padding:3px 6px;font-size:10px;color:var(--muted);letter-spacing:0}.search-wrap{padding:0 10px 10px}.search{width:100%;background:#11161a;border:1px solid var(--line);border-radius:8px;padding:9px 10px;color:var(--text);font-size:12px}.search::placeholder{color:#71808a}.category-list{display:flex;gap:5px;flex-wrap:wrap;padding:0 10px 10px}.category{flex:1 0 auto;border:1px solid var(--line);background:#1a2025;border-radius:7px;padding:7px 8px;font-size:10px;color:#a9b5bd;font-weight:700}.category.active{background:#d4f36a17;border-color:#a8c94f;color:var(--accent)}.material-list{display:grid;grid-template-columns:1fr 1fr;gap:6px;padding:2px 10px 12px;overflow:auto;align-content:start}.material{display:flex;align-items:center;gap:7px;text-align:left;min-width:0;border:1px solid #303941;background:#1c2328;color:#d8e0e5;border-radius:8px;padding:8px 7px;font-size:11px;transition:all .12s}.material:hover{background:#273139;border-color:#52616d}.material.selected{border-color:var(--accent);background:#d4f36a16;color:#f5ffd9;box-shadow:inset 0 0 0 1px #d4f36a20}.swatch{width:13px;height:13px;flex:0 0 13px;border-radius:3px;box-shadow:inset 0 0 0 1px #ffffff25}.material-name{overflow:hidden;text-overflow:ellipsis;white-space:nowrap}.palette-note{margin-top:auto;border-top:1px solid var(--line);padding:11px 12px;color:#82909a;font-size:10px;line-height:1.5}.workspace{display:flex;flex-direction:column;gap:10px;min-width:0;min-height:0}.tool-strip{display:flex;gap:7px;align-items:center;flex-wrap:wrap;padding:9px;border:1px solid var(--line);border-radius:var(--radius);background:var(--panel)}.tool-group{display:flex;gap:5px;padding-right:8px;margin-right:2px;border-right:1px solid var(--line)}.tool-group:last-of-type{border-right:0}.tool-button{border:1px solid transparent;background:transparent;color:#a9b6bf;border-radius:7px;padding:8px 10px;font-size:11px;font-weight:750}.tool-button:hover{background:#273139;color:var(--text)}.tool-button.active{color:var(--accent);background:#d4f36a18;border-color:#758d36}.brush-control{margin-left:auto;display:flex;align-items:center;gap:8px;color:var(--muted);font-size:10px;white-space:nowrap}.brush-control input{width:78px;accent-color:var(--accent)}.world-shell{position:relative;flex:1;min-height:300px;border:1px solid #35414a;border-radius:var(--radius);overflow:hidden;background:#0c1114;box-shadow:0 20px 60px #0002}.world-shell:before{content:"";position:absolute;inset:0;pointer-events:none;background:radial-gradient(ellipse at 50% 45%,#1d292d55 0%,transparent 65%);z-index:0}#world{position:absolute;inset:0;width:100%;height:100%;image-rendering:pixelated;touch-action:none;cursor:crosshair;z-index:1}.canvas-overlay{position:absolute;z-index:2;left:14px;top:13px;display:flex;align-items:center;gap:7px;pointer-events:none;color:#82919b;font:9px ui-monospace,SFMono-Regular,Menlo,monospace;letter-spacing:.05em;text-shadow:0 1px 3px #000}.key-hint{margin-left:auto;position:absolute;right:13px;top:13px;z-index:2;color:#7f8f99;font:9px ui-monospace,SFMono-Regular,Menlo,monospace;text-align:right;line-height:1.8;pointer-events:none;text-shadow:0 1px 3px #000}.canvas-footer{display:flex;align-items:center;justify-content:space-between;gap:8px;flex-wrap:wrap;font:10px ui-monospace,SFMono-Regular,Menlo,monospace;color:#92a0a9;padding:0 3px}.footer-left,.footer-right{display:flex;gap:13px;align-items:center}.footer-value{color:#e0e8ec}.tiny-dot{display:inline-block;width:5px;height:5px;border-radius:50%;background:var(--accent);margin-right:5px;vertical-align:middle}.inspector-body{padding:0 12px 13px;display:flex;flex-direction:column;gap:13px;overflow:auto}.selected-card{border:1px solid var(--line);border-radius:10px;background:#141a1e;padding:12px}.selected-preview{height:64px;border-radius:7px;margin-bottom:11px;position:relative;overflow:hidden;background:linear-gradient(135deg,#1b2429,#111619);display:grid;place-items:center}.selected-preview:before{content:"";position:absolute;inset:0;background-image:radial-gradient(#354149 .7px,transparent .7px);background-size:7px 7px;opacity:.5}.big-swatch{position:relative;width:28px;height:28px;border-radius:5px;box-shadow:0 0 28px #ffffff10,inset 0 0 0 1px #ffffff40}.selected-title{font-size:16px;font-weight:850;letter-spacing:-.02em}.selected-sub{font-size:10px;color:var(--muted);margin-top:4px;line-height:1.5}.tag-row{display:flex;gap:5px;flex-wrap:wrap;margin-top:10px}.tag{border:1px solid var(--line);border-radius:5px;padding:4px 6px;color:#9cabb5;font-size:9px;text-transform:uppercase;letter-spacing:.06em}.section{border-top:1px solid var(--line);padding-top:12px}.section-title{font-size:9px;color:#9aa8b1;letter-spacing:.14em;font-weight:900;margin-bottom:10px}.setting-row{display:flex;align-items:center;justify-content:space-between;gap:8px;margin:10px 0;color:#b6c1c8;font-size:11px}.setting-row output{font:10px ui-monospace,monospace;color:var(--accent)}.setting-row input[type=range]{width:94px;accent-color:var(--accent)}.setting-row input[type=checkbox]{accent-color:var(--accent)}.button-row{display:flex;gap:6px}.button-row .btn{flex:1}.stat-grid{display:grid;grid-template-columns:1fr 1fr;gap:7px}.stat{border:1px solid var(--line);border-radius:8px;background:#151a1e;padding:9px}.stat-number{font-size:17px;font-weight:850;letter-spacing:-.04em}.stat-label{font-size:9px;color:var(--muted);margin-top:4px;letter-spacing:.08em;text-transform:uppercase}.element-counts{display:flex;flex-direction:column;gap:8px}.count-line{display:flex;align-items:center;gap:7px;font-size:10px;color:#aab5bd}.count-line .count-name{flex:1}.count-line .count-num{font:10px ui-monospace,monospace;color:#e6edf1}.empty-counts{color:#75838d;font-size:10px;line-height:1.6}.footer-note{color:#697780;font-size:9px;line-height:1.6}.toast{position:fixed;left:50%;bottom:22px;transform:translate(-50%,12px);background:#e8f6b5;color:#1e2910;padding:10px 14px;border-radius:8px;font-size:11px;font-weight:800;box-shadow:0 8px 35px #0006;opacity:0;pointer-events:none;transition:all .18s;z-index:10}.toast.show{opacity:1;transform:translate(-50%,0)}.hidden{display:none!important}button:disabled{opacity:.4;cursor:not-allowed}
@media(max-width:1150px){.layout{grid-template-columns:210px minmax(0,1fr)}.inspector{display:none}.brand{min-width:180px}.world-pulse{display:none}}
@media(max-width:760px){.app{padding:7px;gap:7px;min-height:100%;height:auto}.topbar{gap:8px;padding:9px;flex-wrap:wrap}.brand{min-width:0;margin-right:auto}.top-actions{gap:4px}.top-actions .btn{padding:7px;font-size:10px}.layout{display:flex;flex-direction:column}.library{max-height:245px}.material-list{grid-template-columns:repeat(4,minmax(0,1fr))}.material{justify-content:center;padding:7px 4px}.material-name{font-size:10px}.category-list{gap:4px}.category{padding:6px 5px}.world-shell{flex:initial;height:min(57vh,560px);min-height:300px}.tool-strip{gap:4px;padding:6px}.tool-group{padding-right:4px;gap:2px}.tool-button{padding:7px;font-size:10px}.brush-control{margin-left:0}.canvas-footer{font-size:9px}.key-hint{display:none}}
@media(prefers-reduced-motion:reduce){*{transition:none!important;scroll-behavior:auto!important}}
</style>
</head>
<body>
<div class="app">
  <header class="topbar">
    <div class="brand"><div class="brand-mark">••</div><div><div class="brand-title">PARTICLE LAB</div><div class="brand-sub">TINY WORLDS, BIG CHAOS</div></div></div>
    <div class="world-pulse"><span class="pulse" id="pulse"></span><span id="runLabel">WORLD RUNNING</span><span style="color:#62727d">/</span><span id="headerCount">0 particles</span></div>
    <div class="top-actions">
      <button class="btn" id="pauseBtn" title="Pause or resume (Space)">Ⅱ Pause</button>
      <button class="btn" id="stepBtn" title="Advance one tick (>)">▷ Step</button>
      <button class="btn" id="undoBtn" title="Undo (Ctrl+Z)" disabled>↶ Undo</button>
      <button class="btn" id="redoBtn" title="Redo (Ctrl+Shift+Z)" disabled>↷ Redo</button>
    </div>
  </header>
  <main class="layout">
    <aside class="panel library">
      <div class="panel-head"><span>ELEMENT LIBRARY</span><span class="count-badge" id="libraryCount">00</span></div>
      <div class="search-wrap"><input class="search" id="search" type="search" placeholder="⌕  Find a material…" autocomplete="off" aria-label="Search materials"></div>
      <div class="category-list" id="categories" aria-label="Material categories"></div>
      <div class="material-list" id="materials" aria-label="Materials"></div>
      <div class="palette-note">Choose a material, then paint into the world. Mix elements and watch the rules unfold.</div>
    </aside>

    <section class="workspace" aria-label="Simulation workspace">
      <div class="tool-strip">
        <div class="tool-group" aria-label="Drawing tools">
          <button class="tool-button active" data-tool="draw" title="Draw material (left click)">✳ Draw</button>
          <button class="tool-button" data-tool="erase" title="Erase (right click)">⌫ Erase</button>
          <button class="tool-button" data-tool="heat" title="Add heat">♨ Heat</button>
          <button class="tool-button" data-tool="cool" title="Remove heat">❄ Cool</button>
          <button class="tool-button" data-tool="pick" title="Pick material (middle click)">⌖ Pick</button>
        </div>
        <div class="tool-group"><button class="tool-button" id="replaceBtn" title="Allow drawing over existing pixels">▧ Replace: off</button></div>
        <label class="brush-control" for="brushSize">BRUSH <output id="brushLabel">3</output><input id="brushSize" type="range" min="1" max="12" value="3" aria-label="Brush size"></label>
      </div>
      <div class="world-shell" id="worldShell">
        <canvas id="world" width="180" height="100" aria-label="Particle simulation. Draw with the left mouse button, right-click to erase."></canvas>
        <div class="canvas-overlay"><span class="tiny-dot"></span><span id="worldMode">LIVE SIMULATION</span><span style="color:#4b5d66">•</span><span id="selectedStatus">SAND</span></div>
        <div class="key-hint">SHIFT + DRAG&nbsp; DRAW LINE<br>SCROLL&nbsp; CHANGE BRUSH<br>SPACE&nbsp; PAUSE</div>
      </div>
      <div class="canvas-footer">
        <div class="footer-left"><span>XY <span class="footer-value" id="coords">— / —</span></span><span>FPS <span class="footer-value" id="fps">30</span></span><span>CELLS <span class="footer-value" id="cellCount">0</span></span></div>
        <div class="footer-right"><span>TOOL <span class="footer-value" id="toolStatus">DRAW</span></span><span><span class="tiny-dot"></span><span id="selectionFooter">Sand selected</span></span></div>
      </div>
    </section>

    <aside class="panel inspector">
      <div class="panel-head"><span>WORLD INSPECTOR</span><span class="count-badge">LAB / 01</span></div>
      <div class="inspector-body">
        <section>
          <div class="eyebrow" style="margin:1px 0 8px">CURRENT MATERIAL</div>
          <div class="selected-card">
            <div class="selected-preview"><div class="big-swatch" id="bigSwatch"></div></div>
            <div class="selected-title" id="detailName">Sand</div>
            <div class="selected-sub" id="detailDesc">Loose grains that tumble, pile up, and settle under gravity.</div>
            <div class="tag-row"><span class="tag" id="detailState">POWDER</span><span class="tag" id="detailGroup">TERRAIN</span></div>
          </div>
        </section>
        <section class="section">
          <div class="section-title">WORLD SETTINGS</div>
          <div class="setting-row"><label for="ambient">Ambient temperature</label><output id="ambientValue">20°C</output></div>
          <div class="setting-row" style="margin-top:0"><input id="ambient" type="range" min="-30" max="80" value="20" style="width:100%" aria-label="Ambient temperature"></div>
          <div class="setting-row"><label for="speed">Simulation speed</label><output id="speedValue">30 TPS</output></div>
          <div class="setting-row" style="margin-top:0"><input id="speed" type="range" min="1" max="60" value="30" style="width:100%" aria-label="Simulation speed"></div>
          <div class="setting-row"><label for="gridToggle">Show grid</label><input id="gridToggle" type="checkbox"></div>
          <div class="setting-row"><label for="stainToggle">Color variation</label><input id="stainToggle" type="checkbox" checked></div>
        </section>
        <section class="section">
          <div class="section-title">WORLD ACTIONS</div>
          <div class="button-row"><button class="btn" id="saveBtn">Save world</button><button class="btn" id="loadBtn">Load save</button></div>
          <div class="button-row" style="margin-top:7px"><button class="btn" id="exportBtn">Export JSON</button><button class="btn" id="importBtn">Import JSON</button></div>
          <div class="button-row" style="margin-top:7px"><button class="btn" id="shotBtn">Capture PNG</button><button class="btn danger" id="clearBtn">Reset world</button></div>
          <input class="hidden" type="file" id="fileInput" accept=".json,application/json">
        </section>
        <section class="section">
          <div class="section-title">LIVE READOUT</div>
          <div class="stat-grid"><div class="stat"><div class="stat-number" id="statParticles">0</div><div class="stat-label">Particles</div></div><div class="stat"><div class="stat-number" id="statTypes">0</div><div class="stat-label">Material types</div></div></div>
          <div class="element-counts" id="elementCounts" style="margin-top:12px"><div class="empty-counts">Add a few particles to start the readout.</div></div>
        </section>
        <div class="footer-note">A tiny, local-only particle laboratory. Saves stay in this browser unless exported. No external libraries or network calls.</div>
      </div>
    </aside>
  </main>
</div>
<div class="toast" id="toast" role="status" aria-live="polite"></div>
<script>
(() => {
  'use strict';

  // Particle Lab — an original, dependency-free cellular simulation.
  const W = 180, H = 100, N = W * H;
  const definitions = [
    {key:'sand',name:'Sand',color:'#e5c16d',category:'Powders',state:'powder',density:7,description:'Loose grains that tumble, pile up, and settle under gravity.'},
    {key:'dirt',name:'Dirt',color:'#866044',category:'Powders',state:'powder',density:6,description:'Heavy earth. Add water to turn it into mud.'},
    {key:'ash',name:'Ash',color:'#8c9192',category:'Powders',state:'powder',density:3,description:'Light residue left behind by fire.'},
    {key:'salt',name:'Salt',color:'#e7e7dc',category:'Powders',state:'powder',density:5,description:'Pale crystals that dissolve in water.'},
    {key:'water',name:'Water',color:'#429fe5',category:'Liquids',state:'liquid',density:3,description:'Flows downhill, extinguishes fire, and becomes steam when heated.'},
    {key:'saltwater',name:'Salt water',color:'#4e91c2',category:'Liquids',state:'liquid',density:3.2,description:'A denser blue liquid. Evaporates into steam when heated.'},
    {key:'oil',name:'Oil',color:'#655b32',category:'Liquids',state:'liquid',density:2,flammable:true,description:'A light, flammable liquid. Fire can spread across its surface.'},
    {key:'acid',name:'Acid',color:'#c8ef4e',category:'Liquids',state:'liquid',density:3,description:'A reactive liquid that slowly eats through many materials.'},
    {key:'lava',name:'Lava',color:'#ff5b27',category:'Liquids',state:'liquid',density:8,hot:1100,life:260,description:'Molten rock. It radiates heat and eventually cools into stone.'},
    {key:'mud',name:'Mud',color:'#735443',category:'Liquids',state:'liquid',density:4,description:'Heavy wet earth that oozes slowly.'},
    {key:'stone',name:'Stone',color:'#777f85',category:'Solids',state:'solid',density:9,description:'A sturdy, inert solid. Blocks most movement.'},
    {key:'wall',name:'Wall',color:'#414c55',category:'Solids',state:'solid',density:10,description:'An indestructible boundary for shaping your world.'},
    {key:'wood',name:'Wood',color:'#99663f',category:'Solids',state:'solid',density:6,flammable:true,description:'A solid building material that can catch fire.'},
    {key:'glass',name:'Glass',color:'#a5dce0',category:'Solids',state:'solid',density:7,description:'A smooth, translucent solid made when sand meets intense heat.'},
    {key:'metal',name:'Metal',color:'#9ca8b2',category:'Solids',state:'solid',density:9,description:'Dense metal that conducts heat and resists most reactions.'},
    {key:'ice',name:'Ice',color:'#9ce8ef',category:'Solids',state:'solid',density:4,description:'Frozen water. Sufficient heat turns it back into liquid.'},
    {key:'fire',name:'Fire',color:'#ff7a34',category:'Energy',state:'energy',density:0,hot:500,life:42,description:'A short-lived flame that heats and ignites nearby fuel.'},
    {key:'steam',name:'Steam',color:'#bdcbd0',category:'Gases',state:'gas',density:0,hot:115,life:120,description:'Hot water vapor that rises and condenses as it cools.'},
    {key:'smoke',name:'Smoke',color:'#66747b',category:'Gases',state:'gas',density:0,life:130,description:'A drifting cloud of particles created by combustion.'},
    {key:'seed',name:'Seed',color:'#8ca04a',category:'Life',state:'powder',density:4,description:'Give a seed water and soil to help it grow into a plant.'},
    {key:'plant',name:'Plant',color:'#55bd61',category:'Life',state:'solid',density:4,flammable:true,description:'A living sprout that grows upward when water is nearby.'},
    {key:'snow',name:'Snow',color:'#edf5ff',category:'Powders',state:'powder',density:2,description:'Cold flakes that collect in soft piles and melt with heat.'}
  ];
  const materials = definitions;
  const ids = Object.create(null), byId = [null];
  materials.forEach((m,i) => { m.id = i + 1; ids[m.key] = m.id; byId[m.id] = m; });

  let grid = new Uint8Array(N), heat = new Float32Array(N), life = new Uint16Array(N);
  let processed = new Uint8Array(N), selectedKey = 'sand', category = 'All', tool = 'draw';
  let brush = 3, ambient = 20, tps = 30, paused = false, replace = false;
  let showGrid = false, colorVariation = true, drawing = false, lineMode = false, strokeStart = null, lastPoint = null;
  let tick = 0, frames = 0, frameStamp = performance.now(), fps = 30, lastCounts = 0;
  let undoStack = [], redoStack = [], toastTimer = 0, lastPointForUI = {x:0,y:0};

  const canvas = document.getElementById('world');
  const ctx = canvas.getContext('2d', {alpha:false});
  const el = id => document.getElementById(id);
  const categories = ['All','Powders','Liquids','Solids','Energy','Gases','Life'];
  const index = (x,y) => y * W + x;
  const inBounds = (x,y) => x >= 0 && x < W && y >= 0 && y < H;
  const materialAt = (x,y) => inBounds(x,y) ? grid[index(x,y)] : 0;
  const defAt = id => byId[id] || null;
  const random = Math.random;

  function toast(message) {
    const node = el('toast'); node.textContent = message; node.classList.add('show');
    clearTimeout(toastTimer); toastTimer = setTimeout(() => node.classList.remove('show'), 1800);
  }
  function snapshot() {
    undoStack.push({grid:grid.slice(),heat:heat.slice(),life:life.slice()});
    if (undoStack.length > 14) undoStack.shift();
    redoStack = []; updateHistoryButtons();
  }
  function restore(state) {
    grid.set(state.grid); heat.set(state.heat); life.set(state.life); processed.fill(0); render(); updateReadout(true);
  }
  function updateHistoryButtons() { el('undoBtn').disabled = undoStack.length === 0; el('redoBtn').disabled = redoStack.length === 0; }
  function undo() {
    if (!undoStack.length) return;
    redoStack.push({grid:grid.slice(),heat:heat.slice(),life:life.slice()}); restore(undoStack.pop()); updateHistoryButtons(); toast('Undo complete');
  }
  function redo() {
    if (!redoStack.length) return;
    undoStack.push({grid:grid.slice(),heat:heat.slice(),life:life.slice()}); restore(redoStack.pop()); updateHistoryButtons(); toast('Redo complete');
  }
  function put(x,y,id,temp,lifespan) {
    if (!inBounds(x,y)) return;
    const i = index(x,y); grid[i] = id;
    heat[i] = temp == null ? ((defAt(id) && defAt(id).hot) || ambient) : temp;
    life[i] = lifespan == null ? ((defAt(id) && defAt(id).life) || 0) : lifespan;
  }
  function clearCell(i) { grid[i] = 0; heat[i] = ambient; life[i] = 0; }
  function swap(i,j) {
    let v = grid[i]; grid[i] = grid[j]; grid[j] = v;
    let h = heat[i]; heat[i] = heat[j]; heat[j] = h;
    let l = life[i]; life[i] = life[j]; life[j] = l;
    processed[i] = 1; processed[j] = 1;
  }
  function moveTo(i,x,y,nx,ny) {
    if (!inBounds(nx,ny)) return false;
    const j = index(nx,ny);
    if (grid[j] === 0) { swap(i,j); return true; }
    return false;
  }
  function adjacent(x,y,fn) {
    for (let dy=-1;dy<=1;dy++) for (let dx=-1;dx<=1;dx++) {
      if (dx===0 && dy===0) continue;
      const nx=x+dx, ny=y+dy; if (inBounds(nx,ny)) fn(index(nx,ny),nx,ny,dx,dy);
    }
  }
  function turnInto(i,key,temp,lifespan) {
    grid[i] = ids[key]; heat[i] = temp == null ? ((defAt(ids[key]).hot)||ambient) : temp;
    life[i] = lifespan == null ? ((defAt(ids[key]).life)||0) : lifespan;
  }
  function tryFall(i,x,y,def,dir=1) {
    const belowY=y+dir;
    if (moveTo(i,x,y,x,belowY)) return true;
    if (!inBounds(x,belowY)) return false;
    const j=index(x,belowY), target=defAt(grid[j]);
    if (target && target.state==='liquid' && def.density > target.density) { swap(i,j); return true; }
    const first = random() < .5 ? -1 : 1;
    if (moveTo(i,x,y,x+first,belowY) || moveTo(i,x,y,x-first,belowY)) return true;
    return false;
  }
  function flowLiquid(i,x,y,def) {
    if (tryFall(i,x,y,def,1)) return true;
    const first = random() < .5 ? -1 : 1;
    const spread = def.key==='mud' ? 1 : 2;
    for (let step=1;step<=spread;step++) {
      if (moveTo(i,x,y,x+first*step,y) || moveTo(i,x,y,x-first*step,y)) return true;
    }
    return false;
  }
  function flowGas(i,x,y) {
    if (moveTo(i,x,y,x,y-1)) return true;
    const first = random() < .5 ? -1 : 1;
    return moveTo(i,x,y,x+first,y-1) || moveTo(i,x,y,x-first,y-1) || moveTo(i,x,y,x+first,y) || moveTo(i,x,y,x-first,y);
  }
  function burnNearby(x,y,heatBoost=0) {
    adjacent(x,y,(j,nx,ny) => {
      if (!grid[j]) return;
      const d=defAt(grid[j]);
      heat[j]=Math.min(1400,heat[j]+(heatBoost||1.5));
      if (d.flammable && random() < (d.key==='oil'?.18:.035)) turnInto(j,'fire',500,28+Math.floor(random()*28));
      else if (d.key==='sand' && heat[j] > 850) turnInto(j,'glass',heat[j],0);
      else if ((d.key==='water'||d.key==='saltwater') && heat[j] > 95 && random()<.18) turnInto(j,'steam',115,100);
    });
  }
  function simulateParticle(i,x,y,def) {
    // Slow thermal relaxation makes heating/cooling tools and hot materials interact.
    if (def.state !== 'solid' || def.key==='ice' || def.key==='plant') heat[i] += (ambient - heat[i]) * 0.0015;
    if (def.key==='fire') {
      heat[i]=Math.max(heat[i],260); burnNearby(x,y,2.6);
      let soaked=false;
      adjacent(x,y,(j) => { if (grid[j] && ['water','saltwater','ice'].includes(defAt(grid[j]).key)) soaked=true; });
      life[i] = life[i] > 0 ? life[i]-1 : 0;
      if (soaked || life[i]===0) { if (random()<.7 && inBounds(x,y-1) && !grid[index(x,y-1)]) put(x,y-1,ids.smoke,45,70); turnInto(i,'smoke',45,90); return; }
      if (y>0 && random()<.62 && grid[index(x,y-1)]===0) { const j=index(x,y-1); swap(i,j); return; }
      if (random()<.08) { const dx=random()<.5?-1:1; if (moveTo(i,x,y,x+dx,y-1)) return; }
      return;
    }
    if (def.key==='lava') {
      heat[i]=Math.max(heat[i],720); burnNearby(x,y,3.3);
      adjacent(x,y,(j) => {
        if (grid[j]===ids.sand && random()<.09) turnInto(j,'glass',heat[j],0);
        if (grid[j]===ids.water || grid[j]===ids.saltwater) { turnInto(j,'steam',125,95); }
        if (grid[j]===ids.ice) turnInto(j,'water',25,0);
      });
      if (life[i]>0) life[i]--; else life[i]=0;
      if (life[i]===0 && tick%3===0) { turnInto(i,'stone',ambient,0); return; }
      flowLiquid(i,x,y,def); return;
    }
    if (def.key==='sand' && heat[i]>850) { turnInto(i,'glass',heat[i],0); return; }
    if (def.key==='water' || def.key==='saltwater') {
      if (heat[i]>100) { turnInto(i,'steam',Math.max(110,heat[i]),100); return; }
      if (heat[i]<0) { turnInto(i,'ice',Math.max(-10,heat[i]),0); return; }
      // Water dissolves nearby salt and damps fire; saltwater remains salty.
      adjacent(x,y,(j) => { if (grid[j]===ids.salt && def.key==='water' && random()<.025) clearCell(j); if(grid[j]===ids.fire && random()<.28){turnInto(j,'steam',115,80);} });
      flowLiquid(i,x,y,def); return;
    }
    if (def.key==='ice') { if (heat[i]>18) { turnInto(i,'water',Math.min(heat[i],30),0); return; } tryFall(i,x,y,def,1); return; }
    if (def.key==='steam') {
      if (heat[i]<85) { turnInto(i,'water',Math.max(20,heat[i]),0); return; }
      if (life[i]>0) life[i]--; else life[i]=0;
      if (life[i]===0 && random()<.025) { clearCell(i); return; }
      if (flowGas(i,x,y)) return; return;
    }
    if (def.key==='smoke') {
      if (life[i]>0) life[i]--; else life[i]=0;
      if (life[i]===0) { clearCell(i); return; }
      if (flowGas(i,x,y)) return; return;
    }
    if (def.key==='sand' || def.key==='dirt' || def.key==='ash' || def.key==='salt' || def.key==='seed' || def.key==='snow') {
      if (def.key==='dirt' && adjacentHas(x,y,ids.water) && random()<.04) { turnInto(i,'mud',ambient,0); return; }
      if (def.key==='snow' && heat[i]>2) { turnInto(i,'water',Math.max(3,heat[i]),0); return; }
      if (def.key==='seed') {
        const hasWater=adjacentHas(x,y,ids.water)||adjacentHas(x,y,ids.saltwater);
        const grounded = y===H-1 || grid[index(x,y+1)]!==0;
        if (hasWater && grounded && random()<.06) { turnInto(i,'plant',ambient,0); return; }
      }
      tryFall(i,x,y,def,1); return;
    }
    if (def.key==='mud') { if (heat[i]>90) { turnInto(i,'dirt',ambient,0); return; } flowLiquid(i,x,y,def); return; }
    if (def.key==='oil') {
      if (heat[i]>180 || adjacentHas(x,y,ids.fire) || adjacentHas(x,y,ids.lava)) { if(random()<.12) turnInto(i,'fire',450,38); }
      flowLiquid(i,x,y,def); return;
    }
    if (def.key==='acid') {
      adjacent(x,y,(j) => {
        const target=defAt(grid[j]); if (!target) return;
        if (['wood','plant','dirt','sand','ash','metal','seed'].includes(target.key) && random()<.055) { if (target.key==='metal') turnInto(j,'smoke',35,45); else clearCell(j); }
        else if (target.key==='stone' && random()<.006) clearCell(j);
      });
      flowLiquid(i,x,y,def); return;
    }
    if (def.key==='plant') {
      if (adjacentHas(x,y,ids.fire) || adjacentHas(x,y,ids.lava)) { if(random()<.08) turnInto(i,'fire',470,36); return; }
      if (random()<.012 && y>1 && grid[index(x,y-1)]===0 && (adjacentHas(x,y,ids.water)||adjacentHas(x,y,ids.saltwater))) put(x,y-1,ids.plant,ambient,0);
      return;
    }
    // Heat may turn selected materials into a new phase.
    if (def.key==='wood' && (heat[i]>230 || adjacentHas(x,y,ids.fire)||adjacentHas(x,y,ids.lava))) { if(random()<.025) turnInto(i,'fire',480,45); return; }
    if (def.key==='sand' && heat[i]>850) { turnInto(i,'glass',heat[i],0); return; }
    if (def.key==='glass' && heat[i]>1250) { turnInto(i,'lava',1100,200); return; }
  }
  function adjacentHas(x,y,id) { let found=false; adjacent(x,y,(j)=>{if(grid[j]===id)found=true;}); return found; }

  function step() {
    processed.fill(0); tick++;
    const xForward = tick % 2 === 0;
    for (let y=H-1;y>=0;y--) {
      if (xForward) {
        for (let x=0;x<W;x++) { const i=index(x,y); if (!grid[i]||processed[i]) continue; const d=defAt(grid[i]); if(d) simulateParticle(i,x,y,d); processed[i]=1; }
      } else {
        for (let x=W-1;x>=0;x--) { const i=index(x,y); if (!grid[i]||processed[i]) continue; const d=defAt(grid[i]); if(d) simulateParticle(i,x,y,d); processed[i]=1; }
      }
    }
  }
  function render() {
    ctx.fillStyle='#0b1013'; ctx.fillRect(0,0,W,H);
    const countsById = new Uint32Array(byId.length);
    for(let y=0;y<H;y++) for(let x=0;x<W;x++) {
      const i=index(x,y), id=grid[i]; if(!id) continue;
      const d=byId[id]; if(!d) continue;
      let color=d.color;
      if (colorVariation) {
        const variation=((i*17 + tick*3 + id*11)%9)-4;
        color=adjustColor(d.color, variation*1.7);
      }
      if (d.key==='fire') { const f=(life[i]%5); color=f<2?'#ffd15f':(f<4?'#ff9c38':'#ff6230'); }
      if (d.key==='lava') color=(tick%8<4?'#ff5a27':'#ff9b32');
      ctx.fillStyle=color; ctx.fillRect(x,y,1,1); countsById[id]++;
    }
    if(showGrid) {
      ctx.strokeStyle='rgba(182,206,214,.12)'; ctx.lineWidth=.06;
      ctx.beginPath(); for(let x=0;x<=W;x+=5){ctx.moveTo(x,0);ctx.lineTo(x,H);} for(let y=0;y<=H;y+=5){ctx.moveTo(0,y);ctx.lineTo(W,y);} ctx.stroke();
    }
    return countsById;
  }
  function adjustColor(hex, amount) {
    const raw=parseInt(hex.slice(1),16); let r=(raw>>16)+amount,g=((raw>>8)&255)+amount,b=(raw&255)+amount;
    r=Math.max(0,Math.min(255,r));g=Math.max(0,Math.min(255,g));b=Math.max(0,Math.min(255,b));
    return `rgb(${r|0},${g|0},${b|0})`;
  }
  function updateReadout(force=false) {
    if(!force && tick-lastCounts<8) return; lastCounts=tick;
    const counts=render(); let total=0, types=0, rows=[];
    counts.forEach((n,id)=>{if(n){total+=n;types++;rows.push({d:byId[id],n});}});
    rows.sort((a,b)=>b.n-a.n);
    el('headerCount').textContent=total.toLocaleString()+' particles';
    el('cellCount').textContent=total.toLocaleString();
    el('statParticles').textContent=total.toLocaleString(); el('statTypes').textContent=types;
    const box=el('elementCounts'); box.replaceChildren();
    if(!rows.length){box.innerHTML='<div class="empty-counts">Add a few particles to start the readout.</div>';}
    else rows.slice(0,5).forEach(({d,n})=>{const row=document.createElement('div');row.className='count-line';const sw=document.createElement('span');sw.className='swatch';sw.style.background=d.color;const name=document.createElement('span');name.className='count-name';name.textContent=d.name;const num=document.createElement('span');num.className='count-num';num.textContent=n.toLocaleString();row.append(sw,name,num);box.append(row);});
  }
  function renderCategories() {
    const wrap=el('categories'); wrap.replaceChildren();
    categories.forEach(c=>{const b=document.createElement('button');b.className='category'+(category===c?' active':'');b.textContent=c;b.addEventListener('click',()=>{category=c;renderCategories();renderMaterials();});wrap.append(b);});
  }
  function renderMaterials() {
    const query=el('search').value.trim().toLowerCase();
    const filtered=materials.filter(m=>(category==='All'||m.category===category) && (!query||m.name.toLowerCase().includes(query)||m.description.toLowerCase().includes(query)));
    const wrap=el('materials'); wrap.replaceChildren();
    filtered.forEach(m=>{const b=document.createElement('button');b.className='material'+(selectedKey===m.key?' selected':'');b.dataset.key=m.key;b.setAttribute('aria-pressed',selectedKey===m.key?'true':'false');const sw=document.createElement('span');sw.className='swatch';sw.style.background=m.color;const name=document.createElement('span');name.className='material-name';name.textContent=m.name;b.append(sw,name);b.addEventListener('click',()=>selectMaterial(m.key));wrap.append(b);});
    el('libraryCount').textContent=String(filtered.length).padStart(2,'0');
  }
  function selectMaterial(key) {
    if(!ids[key])return; selectedKey=key; tool='draw';
    document.querySelectorAll('[data-tool]').forEach(b=>b.classList.toggle('active',b.dataset.tool===tool));
    renderMaterials(); updateToolStatus(); updateDetails();
  }
  function updateDetails() {
    const d=materials.find(m=>m.key===selectedKey); if(!d)return;
    el('bigSwatch').style.background=d.color;el('detailName').textContent=d.name;el('detailDesc').textContent=d.description;
    el('detailState').textContent=d.state.toUpperCase();el('detailGroup').textContent=d.category.toUpperCase();
    el('selectedStatus').textContent=d.name.toUpperCase();el('selectionFooter').textContent=d.name+' selected';
  }
  function updateToolStatus() { el('toolStatus').textContent=tool.toUpperCase(); document.querySelectorAll('[data-tool]').forEach(b=>b.classList.toggle('active',b.dataset.tool===tool)); }
  function setTool(next) { tool=next;updateToolStatus(); if(tool==='erase')toast('Erase tool selected'); }
  function canvasPoint(event) {
    const rect=canvas.getBoundingClientRect();
    return {x:Math.max(0,Math.min(W-1,Math.floor((event.clientX-rect.left)/rect.width*W))),y:Math.max(0,Math.min(H-1,Math.floor((event.clientY-rect.top)/rect.height*H)))};
  }
  function stamp(cx,cy,whichTool=tool,button=0) {
    const radius=brush-1;
    for(let dy=-radius;dy<=radius;dy++) for(let dx=-radius;dx<=radius;dx++) {
      if(dx*dx+dy*dy>radius*radius+1)continue;
      const x=cx+dx,y=cy+dy;if(!inBounds(x,y))continue;const i=index(x,y);
      const activeTool=(button===2)?'erase':whichTool;
      if(activeTool==='pick') { if(grid[i])selectMaterial(defAt(grid[i]).key); return; }
      if(activeTool==='erase') { clearCell(i);continue; }
      if(activeTool==='heat') { if(grid[i])heat[i]=Math.min(1400,heat[i]+90);continue; }
      if(activeTool==='cool') { if(grid[i])heat[i]=Math.max(-80,heat[i]-90);continue; }
      if(activeTool==='draw') {
        if(replace||grid[i]===0) put(x,y,ids[selectedKey]);
      }
    }
  }
  function stampLine(a,b,whichTool=tool,button=0) {
    const dx=b.x-a.x,dy=b.y-a.y,steps=Math.max(Math.abs(dx),Math.abs(dy));
    for(let s=0;s<=steps;s++) {const t=steps===0?0:s/steps;stamp(Math.round(a.x+dx*t),Math.round(a.y+dy*t),whichTool,button);}
  }
  canvas.addEventListener('contextmenu',e=>e.preventDefault());
  canvas.addEventListener('pointerdown',e=>{
    e.preventDefault(); canvas.setPointerCapture(e.pointerId);
    const p=canvasPoint(e);lastPointForUI=p;el('coords').textContent=p.x+' / '+p.y;
    if(e.button===1){stamp(p.x,p.y,'pick');return;}
    if(e.button!==0&&e.button!==2)return;
    snapshot(); drawing=true;lineMode=!!e.shiftKey;strokeStart=p;lastPoint=p;
    if(!lineMode)stamp(p.x,p.y,tool,e.button);
    render();updateReadout(true);
  });
  canvas.addEventListener('pointermove',e=>{
    const p=canvasPoint(e);lastPointForUI=p;el('coords').textContent=p.x+' / '+p.y;
    if(!drawing)return;
    if(lineMode){renderLinePreview(strokeStart,p);return;}
    const button=e.buttons&2?2:0;
    if(lastPoint)stampLine(lastPoint,p,tool,button);else stamp(p.x,p.y,tool,button);
    lastPoint=p;
  });
  canvas.addEventListener('pointerup',e=>{
    if(!drawing)return;const p=canvasPoint(e);const button=e.button===2?2:0;
    if(lineMode&&strokeStart)stampLine(strokeStart,p,tool,button);
    drawing=false;lineMode=false;strokeStart=null;lastPoint=null;render();updateReadout(true);
  });
  canvas.addEventListener('pointercancel',()=>{drawing=false;lineMode=false;strokeStart=null;lastPoint=null;render();});
  function renderLinePreview(a,b){render();ctx.save();ctx.strokeStyle='rgba(212,243,106,.8)';ctx.lineWidth=.45;ctx.setLineDash([1,1]);ctx.beginPath();ctx.moveTo(a.x+.5,a.y+.5);ctx.lineTo(b.x+.5,b.y+.5);ctx.stroke();ctx.restore();}
  canvas.addEventListener('wheel',e=>{e.preventDefault();brush=Math.max(1,Math.min(12,brush+(e.deltaY<0?1:-1)));el('brushSize').value=brush;el('brushLabel').textContent=brush;},{passive:false});

  function loop(now) {
    requestAnimationFrame(loop);
    const interval=1000/tps;
    if(!paused && now-frameStamp>=interval) {
      const batches=Math.min(3,Math.floor((now-frameStamp)/interval));
      for(let i=0;i<Math.max(1,batches);i++)step();
      frameStamp=now;render();
    }
    frames++;
    if(now-(window.__fpsStamp||now)>=1000){fps=frames;frames=0;window.__fpsStamp=now;el('fps').textContent=String(fps);}
    if(now-(window.__readoutStamp||0)>650){updateReadout();window.__readoutStamp=now;}
  }
  function togglePause() {
    paused=!paused;el('pauseBtn').textContent=paused?'▶ Resume':'Ⅱ Pause';el('runLabel').textContent=paused?'WORLD PAUSED':'WORLD RUNNING';el('worldMode').textContent=paused?'PAUSED':'LIVE SIMULATION';el('pulse').style.background=paused?'#75838c':'var(--accent)';el('pulse').style.boxShadow=paused?'none':'0 0 0 4px #d4f36a18';
  }
  function clearWorld() {snapshot();grid.fill(0);heat.fill(ambient);life.fill(0);processed.fill(0);render();updateReadout(true);toast('World reset');}
  function exportWorld() {
    const payload={format:'particle-lab-save',version:1,width:W,height:H,ambient,tps,grid:Array.from(grid),heat:Array.from(heat),life:Array.from(life),selectedKey,tick};
    const blob=new Blob([JSON.stringify(payload)],{type:'application/json'});downloadBlob(blob,'particle-lab-world.json');toast('World exported as JSON');
  }
  function downloadBlob(blob,name) {const url=URL.createObjectURL(blob);const a=document.createElement('a');a.href=url;a.download=name;document.body.append(a);a.click();a.remove();setTimeout(()=>URL.revokeObjectURL(url),1200);}
  function importPayload(payload) {
    if(!payload||payload.format!=='particle-lab-save'||payload.width!==W||payload.height!==H||!Array.isArray(payload.grid)||payload.grid.length!==N)throw new Error('That file is not a compatible Particle Lab save.');
    snapshot();
    payload.grid.forEach((value,i)=>{const id=Number(value);grid[i]=(Number.isInteger(id)&&id>=0&&id<byId.length)?id:0;});
    if(Array.isArray(payload.heat)&&payload.heat.length===N)payload.heat.forEach((v,i)=>heat[i]=Number.isFinite(Number(v))?Number(v):ambient);else heat.fill(ambient);
    if(Array.isArray(payload.life)&&payload.life.length===N)payload.life.forEach((v,i)=>life[i]=Math.max(0,Math.min(65535,Number(v)||0)));else life.fill(0);
    if(Number.isFinite(payload.ambient)){ambient=Math.max(-30,Math.min(80,payload.ambient));el('ambient').value=ambient;el('ambientValue').textContent=ambient+'°C';}
    if(Number.isFinite(payload.tps)){tps=Math.max(1,Math.min(60,payload.tps));el('speed').value=tps;el('speedValue').textContent=tps+' TPS';}
    if(ids[payload.selectedKey])selectMaterial(payload.selectedKey);tick=Number(payload.tick)||0;processed.fill(0);render();updateReadout(true);toast('Save loaded');
  }
  function saveToBrowser() {
    const payload={format:'particle-lab-save',version:1,width:W,height:H,ambient,tps,grid:Array.from(grid),heat:Array.from(heat),life:Array.from(life),selectedKey,tick};
    try{localStorage.setItem('particle-lab-save-v1',JSON.stringify(payload));toast('Saved in this browser');}catch(e){toast('Browser storage is unavailable — export JSON instead');}
  }
  function loadFromBrowser() {
    try{const raw=localStorage.getItem('particle-lab-save-v1');if(!raw){toast('No browser save found yet');return;}importPayload(JSON.parse(raw));}catch(e){toast('Could not load the browser save');}
  }

  document.querySelectorAll('[data-tool]').forEach(b=>b.addEventListener('click',()=>setTool(b.dataset.tool)));
  el('replaceBtn').addEventListener('click',()=>{replace=!replace;el('replaceBtn').textContent=replace?'▧ Replace: on':'▧ Replace: off';el('replaceBtn').classList.toggle('active',replace);});
  el('brushSize').addEventListener('input',e=>{brush=Number(e.target.value);el('brushLabel').textContent=brush;});
  el('search').addEventListener('input',renderMaterials);
  el('pauseBtn').addEventListener('click',togglePause);
  el('stepBtn').addEventListener('click',()=>{if(!paused){togglePause();}step();render();updateReadout(true);});
  el('undoBtn').addEventListener('click',undo);el('redoBtn').addEventListener('click',redo);
  el('clearBtn').addEventListener('click',()=>{if(confirm('Reset the entire world? This can be undone.'))clearWorld();});
  el('ambient').addEventListener('input',e=>{ambient=Number(e.target.value);el('ambientValue').textContent=ambient+'°C';});
  el('speed').addEventListener('input',e=>{tps=Number(e.target.value);el('speedValue').textContent=tps+' TPS';});
  el('gridToggle').addEventListener('change',e=>{showGrid=e.target.checked;render();});
  el('stainToggle').addEventListener('change',e=>{colorVariation=e.target.checked;render();});
  el('saveBtn').addEventListener('click',saveToBrowser);el('loadBtn').addEventListener('click',loadFromBrowser);
  el('exportBtn').addEventListener('click',exportWorld);
  el('importBtn').addEventListener('click',()=>{el('fileInput').dataset.intent='import';el('fileInput').click();});
  el('fileInput').addEventListener('change',async e=>{const file=e.target.files&&e.target.files[0];if(!file)return;try{const payload=JSON.parse(await file.text());importPayload(payload);}catch(err){toast(err.message||'Could not read this file');}e.target.value='';});
  el('shotBtn').addEventListener('click',()=>{const a=document.createElement('a');a.href=canvas.toDataURL('image/png');a.download='particle-lab.png';a.click();toast('PNG captured');});
  document.addEventListener('keydown',e=>{
    const tag=(e.target&&e.target.tagName||'').toLowerCase();const typing=['input','textarea','select'].includes(tag);
    if((e.ctrlKey||e.metaKey)&&e.key.toLowerCase()==='z'){e.preventDefault();if(e.shiftKey)redo();else undo();return;}
    if(typing)return;
    if(e.code==='Space'||e.key.toLowerCase()==='p'){e.preventDefault();togglePause();}
    else if(e.key==='>'){if(!paused)togglePause();step();render();updateReadout(true);}
    else if(e.key==='['||e.key==='-'){brush=Math.max(1,brush-1);el('brushSize').value=brush;el('brushLabel').textContent=brush;}
    else if(e.key===']'||e.key==='+'){brush=Math.min(12,brush+1);el('brushSize').value=brush;el('brushLabel').textContent=brush;}
    else if(e.key.toLowerCase()==='e'||e.key==='/'){e.preventDefault();el('search').focus();}
    else if(e.key.toLowerCase()==='r'){clearWorld();}
    else if(e.key.toLowerCase()==='c'||e.key==='F2'){el('shotBtn').click();}
    else if(e.key.toLowerCase()==='f'||e.key==='F11'){e.preventDefault();const frame=el('worldShell');if(!document.fullscreenElement&&frame.requestFullscreen)frame.requestFullscreen();else if(document.exitFullscreen)document.exitFullscreen();}
    else if(e.key==='Escape'){el('search').blur();}
  });
  window.addEventListener('blur',()=>{drawing=false;lineMode=false;strokeStart=null;lastPoint=null;});

  renderCategories();renderMaterials();updateDetails();updateToolStatus();
  heat.fill(ambient);render();updateReadout(true);requestAnimationFrame(loop);
})();
</script>
</body>
</html>

```

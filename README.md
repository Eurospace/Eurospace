<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>ORBITEX</title>
  <script src="https://accounts.google.com/gsi/client" async></script>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      background: #000008;
      font-family: 'Courier New', monospace;
      overflow: hidden;
      color: #00ffff;
      user-select: none;
    }

    #canvas {
      display: block;
      cursor: crosshair;
    }

    #loader {
      position: fixed;
      inset: 0;
      background: #000008;
      z-index: 1000;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      transition: opacity 1s ease;
    }

    #loader.fade {
      opacity: 0;
      pointer-events: none;
    }

    .loader-title {
      font-size: 38px;
      letter-spacing: 10px;
      color: #00ffff;
      text-shadow: 0 0 30px #00ffff;
      margin-bottom: 8px;
    }

    .loader-sub {
      font-size: 14px;
      letter-spacing: 4px;
      color: rgba(0, 200, 255, .4);
      margin-bottom: 32px;
    }

    .loader-bar-wrap {
      width: 260px;
      height: 3px;
      background: rgba(0, 255, 255, .1);
      border-radius: 2px;
      margin-top: 24px;
    }

    .loader-bar {
      height: 3px;
      background: linear-gradient(to right, #4400ff, #00ffff);
      border-radius: 2px;
      width: 0%;
      transition: width .15s;
    }

    .loader-status {
      font-size: 13px;
      letter-spacing: 2px;
      color: rgba(0, 200, 255, .4);
      margin-top: 10px;
    }

    #hud {
      position: fixed;
      top: 0;
      left: 0;
      right: 0;
      padding: 12px 22px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      background: linear-gradient(to bottom, rgba(0, 0, 18, .97), transparent);
      pointer-events: none;
      z-index: 100;
    }

    .hud-center {
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 10px;
      pointer-events: auto;
    }

    .btn-unlock {
      background: linear-gradient(135deg, #ffcc00, #ff8800);
      color: #000;
      font-weight: bold;
      font-size: 11px;
      letter-spacing: 1px;
      padding: 6px 14px;
      border-radius: 15px;
      border: none;
      cursor: pointer;
      box-shadow: 0 0 15px rgba(255, 200, 0, .3);
      transition: all .2s;
    }

    .btn-unlock:hover {
      transform: scale(1.05);
      box-shadow: 0 0 25px rgba(255, 200, 0, .5);
    }

    #mode-tabs {
      position: fixed;
      top: 52px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 4px;
      z-index: 12;
      background: rgba(0, 5, 20, .8);
      border: 1px solid rgba(0, 255, 255, .15);
      border-radius: 25px;
      padding: 4px;
    }

    .mtab {
      font-size: 12px;
      letter-spacing: 2px;
      padding: 6px 18px;
      border-radius: 20px;
      cursor: pointer;
      transition: all .25s;
      color: rgba(0, 200, 255, .5);
      border: none;
      background: none;
    }

    .mtab.active {
      background: rgba(0, 200, 255, .18);
      color: #00ffff;
      box-shadow: 0 0 10px rgba(0, 200, 255, .3);
    }

    #sub-modes {
      position: fixed;
      top: 96px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 5px;
      z-index: 11;
      flex-wrap: wrap;
      justify-content: center;
      max-width: 92vw;
    }

    .mpill {
      font-size: 11px;
      letter-spacing: 1px;
      padding: 5px 12px;
      border-radius: 20px;
      border: 1px solid rgba(0, 255, 255, .22);
      background: rgba(0, 10, 40, .75);
      color: rgba(0, 200, 255, .55);
      cursor: pointer;
      transition: all .2s;
      white-space: nowrap;
    }

    .mpill.active {
      background: rgba(0, 200, 255, .15);
      border-color: rgba(0, 255, 255, .7);
      color: #00ffff;
    }

    .mpill.locked {
      opacity: .4;
      cursor: not-allowed;
      position: relative;
    }

    #search-bar {
      position: fixed;
      top: 136px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 15;
    }

    #search-input {
      background: rgba(0, 8, 32, .88);
      border: 1px solid rgba(0, 255, 255, .25);
      color: #00ffff;
      font-family: 'Courier New', monospace;
      font-size: 14px;
      padding: 8px 14px;
      border-radius: 4px;
      outline: none;
      width: 260px;
      letter-spacing: 1px;
    }

    #search-input::placeholder {
      color: rgba(0, 200, 255, .28);
    }

    #search-results {
      position: fixed;
      top: 174px;
      left: 50%;
      transform: translateX(-50%);
      width: 274px;
      background: rgba(0, 5, 25, .96);
      border: 1px solid rgba(0, 255, 255, .2);
      border-radius: 4px;
      z-index: 20;
      display: none;
      max-height: 200px;
      overflow-y: auto;
    }

    .sr-item {
      padding: 9px 14px;
      font-size: 13px;
      cursor: pointer;
      letter-spacing: 1px;
      border-bottom: 1px solid rgba(0, 255, 255, .06);
    }

    .sr-item:hover {
      background: rgba(0, 200, 255, .12);
    }

    #sidebar {
      position: fixed;
      top: 50%;
      right: 18px;
      transform: translateY(-50%);
      display: flex;
      flex-direction: column;
      gap: 6px;
      z-index: 10;
    }

    .cbtn {
      width: 42px;
      height: 42px;
      background: rgba(0, 255, 255, .07);
      border: 1px solid rgba(0, 255, 255, .3);
      color: #00ffff;
      font-size: 18px;
      cursor: pointer;
      border-radius: 4px;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: all .2s;
      position: relative;
    }

    .cbtn:hover {
      background: rgba(0, 255, 255, .18);
    }

    .cbtn.on {
      background: rgba(0, 255, 255, .2);
      border-color: #00ffff;
    }

    .cbtn.locked-btn {
      opacity: .4;
      cursor: not-allowed;
    }

    .cbtn-tip {
      position: absolute;
      right: 52px;
      top: 50%;
      transform: translateY(-50%);
      background: rgba(0, 5, 25, .95);
      border: 1px solid rgba(0, 255, 255, .2);
      color: #00ffff;
      font-size: 11px;
      padding: 4px 10px;
      border-radius: 4px;
      white-space: nowrap;
      display: none;
      letter-spacing: 1px;
    }

    .cbtn:hover .cbtn-tip {
      display: block;
    }

    #speed-wrap {
      position: fixed;
      bottom: 22px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 10;
      display: flex;
      align-items: center;
      gap: 12px;
      background: rgba(0, 5, 25, .85);
      border: 1px solid rgba(0, 255, 255, .18);
      border-radius: 20px;
      padding: 8px 18px;
    }

    #speed-wrap label {
      font-size: 13px;
      letter-spacing: 2px;
      color: rgba(0, 200, 255, .6);
    }

    #speed-slider {
      -webkit-appearance: none;
      appearance: none;
      width: 110px;
      height: 4px;
      background: rgba(0, 255, 255, .2);
      outline: none;
      border-radius: 2px;
    }

    #speed-slider::-webkit-slider-thumb {
      -webkit-appearance: none;
      appearance: none;
      width: 14px;
      height: 14px;
      border-radius: 50%;
      background: #00ffff;
      cursor: pointer;
    }

    #speed-val {
      font-size: 13px;
      color: #00ffff;
      width: 36px;
    }

    #pause-btn {
      font-size: 16px;
      cursor: pointer;
      color: #00ffff;
      background: none;
      border: none;
    }

    #panel {
      position: fixed;
      top: 50%;
      right: 70px;
      transform: translateY(-50%);
      width: 310px;
      background: rgba(0, 3, 20, .95);
      border: 1px solid rgba(0, 255, 255, .2);
      border-radius: 8px;
      padding: 18px;
      z-index: 20;
      display: none;
      max-height: 80vh;
      overflow-y: auto;
    }

    #panel.on {
      display: block;
      animation: pIn .2s ease;
    }

    @keyframes pIn {
      from {
        opacity: 0;
        transform: translateY(-50%) translateX(10px)
      }

      to {
        opacity: 1;
        transform: translateY(-50%) translateX(0)
      }
    }

    .ph {
      display: flex;
      justify-content: space-between;
      margin-bottom: 12px;
      border-bottom: 1px solid rgba(0, 255, 255, .1);
      padding-bottom: 12px;
    }

    .pname {
      font-size: 18px;
      font-weight: bold;
      color: #fff;
      letter-spacing: 2px;
    }

    .ptype {
      font-size: 11px;
      letter-spacing: 2px;
      padding: 3px 9px;
      border-radius: 20px;
      border: 1px solid;
      margin-top: 5px;
      display: inline-block;
    }

    .pcls {
      background: none;
      border: none;
      color: rgba(0, 255, 255, .38);
      font-size: 18px;
      cursor: pointer;
    }

    .pvis {
      width: 56px;
      height: 56px;
      border-radius: 50%;
      margin: 12px auto;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 28px;
    }

    .dr {
      display: flex;
      justify-content: space-between;
      padding: 5px 0;
      border-bottom: 1px solid rgba(255, 255, 255, .04);
      font-size: 12px;
    }

    .dl {
      color: rgba(0, 190, 255, .5);
      padding-right: 8px;
      flex-shrink: 0;
    }

    .dv {
      color: #d5eeff;
      text-align: right;
    }

    .tool-panel {
      position: fixed;
      top: 50%;
      left: 18px;
      transform: translateY(-50%);
      width: 300px;
      background: rgba(0, 3, 20, .95);
      border: 1px solid rgba(0, 255, 255, .2);
      border-radius: 8px;
      padding: 18px;
      z-index: 20;
      display: none;
      max-height: 85vh;
      overflow-y: auto;
    }

    .tool-panel.on {
      display: block;
      animation: tIn .2s ease;
    }

    @keyframes tIn {
      from {
        opacity: 0;
        transform: translateY(-50%) translateX(-10px)
      }

      to {
        opacity: 1;
        transform: translateY(-50%) translateX(0)
      }
    }

    .tool-title {
      font-size: 13px;
      letter-spacing: 3px;
      color: rgba(0, 200, 255, .8);
      margin-bottom: 14px;
      border-bottom: 1px solid rgba(0, 255, 255, .1);
      padding-bottom: 10px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .tcls {
      background: none;
      border: none;
      color: rgba(0, 255, 255, .38);
      font-size: 16px;
      cursor: pointer;
    }

    .tinput {
      background: rgba(0, 8, 32, .8);
      border: 1px solid rgba(0, 255, 255, .25);
      color: #00ffff;
      font-family: 'Courier New', monospace;
      font-size: 13px;
      padding: 7px 11px;
      border-radius: 4px;
      outline: none;
      width: 100%;
      margin-bottom: 9px;
    }

    .tbtn {
      background: rgba(0, 255, 255, .08);
      border: 1px solid rgba(0, 255, 255, .28);
      color: #00ffff;
      font-family: 'Courier New', monospace;
      font-size: 12px;
      letter-spacing: 1px;
      padding: 9px;
      border-radius: 4px;
      cursor: pointer;
      width: 100%;
      margin-bottom: 7px;
      transition: all .2s;
      text-align: center;
    }

    .tbtn:hover {
      background: rgba(0, 255, 255, .2);
    }

    .tbtn.active {
      background: rgba(0, 255, 200, .2);
      border-color: rgba(0, 255, 200, .6);
      color: #00ffcc;
    }

    .tbtn-red {
      background: rgba(255, 60, 60, .08);
      border: 1px solid rgba(255, 60, 60, .28);
      color: #ff7777;
    }

    .tbtn-gold {
      background: rgba(255, 200, 0, .08);
      border: 1px solid rgba(255, 200, 0, .28);
      color: #ffcc44;
    }

    .tbtn-purple {
      background: rgba(180, 100, 255, .08);
      border: 1px solid rgba(180, 100, 255, .28);
      color: #cc88ff;
    }

    .tbtn-green {
      background: rgba(0, 255, 100, .08);
      border: 1px solid rgba(0, 255, 100, .28);
      color: #00ff88;
    }

    .slbl {
      font-size: 12px;
      color: rgba(0, 200, 255, .5);
      letter-spacing: 1px;
      margin-bottom: 5px;
      display: flex;
      justify-content: space-between;
    }

    .planet-row {
      display: flex;
      align-items: center;
      gap: 10px;
      padding: 6px 0;
      border-bottom: 1px solid rgba(0, 255, 255, .05);
      font-size: 12px;
    }

    .vis-item {
      display: flex;
      align-items: center;
      gap: 9px;
      padding: 7px 0;
      border-bottom: 1px solid rgba(0, 255, 255, .05);
    }

    .vis-dot {
      width: 10px;
      height: 10px;
      border-radius: 50%;
      flex-shrink: 0;
    }

    .vis-name {
      font-size: 12px;
      color: #d5eeff;
      width: 76px;
    }

    .vis-info {
      font-size: 11px;
      color: rgba(0, 200, 255, .5);
      flex: 1;
    }

    .vis-status {
      font-size: 11px;
      padding: 3px 7px;
      border-radius: 10px;
    }

    .v-yes {
      background: rgba(0, 200, 100, .15);
      color: #00ff88;
      border: 1px solid rgba(0, 200, 100, .3);
    }

    .v-no {
      background: rgba(200, 50, 50, .1);
      color: #ff6666;
      border: 1px solid rgba(200, 50, 50, .2);
    }

    .v-may {
      background: rgba(200, 150, 0, .1);
      color: #ffcc44;
      border: 1px solid rgba(200, 150, 0, .2);
    }

    #funfact {
      position: fixed;
      bottom: 70px;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(0, 5, 28, .92);
      border: 1px solid rgba(0, 255, 255, .15);
      border-radius: 6px;
      padding: 10px 16px;
      font-size: 12px;
      color: rgba(0, 200, 255, .85);
      letter-spacing: 1px;
      max-width: 80vw;
      text-align: center;
      z-index: 8;
      pointer-events: none;
      transition: opacity .8s;
      line-height: 1.8;
    }

    #grav-hint {
      position: fixed;
      bottom: 120px;
      left: 50%;
      transform: translateX(-50%);
      font-size: 13px;
      letter-spacing: 2px;
      color: rgba(0, 255, 200, .8);
      background: rgba(0, 5, 25, .88);
      border: 1px solid rgba(0, 255, 200, .2);
      border-radius: 20px;
      padding: 8px 20px;
      display: none;
      pointer-events: none;
      z-index: 15;
    }

    #tour-bar {
      position: fixed;
      bottom: 70px;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(0, 5, 28, .92);
      border: 1px solid rgba(0, 255, 255, .2);
      border-radius: 8px;
      padding: 12px 20px;
      font-size: 13px;
      color: #00ffff;
      letter-spacing: 1px;
      max-width: 80vw;
      text-align: center;
      z-index: 15;
      display: none;
      line-height: 1.8;
    }

    #kbd-overlay {
      position: fixed;
      inset: 0;
      background: rgba(0, 0, 0, .85);
      z-index: 500;
      display: none;
      align-items: center;
      justify-content: center;
    }

    #kbd-overlay.on {
      display: flex;
    }

    .kbd-box {
      background: rgba(0, 5, 25, .98);
      border: 1px solid rgba(0, 255, 255, .3);
      border-radius: 12px;
      padding: 28px 36px;
      width: 480px;
      max-height: 80vh;
      overflow-y: auto;
    }

    .kbd-title {
      font-size: 18px;
      letter-spacing: 4px;
      color: #00ffff;
      margin-bottom: 20px;
      text-align: center;
    }

    .kbd-row {
      display: flex;
      justify-content: space-between;
      padding: 8px 0;
      border-bottom: 1px solid rgba(0, 255, 255, .07);
      font-size: 13px;
    }

    .kbd-key {
      background: rgba(0, 255, 255, .1);
      border: 1px solid rgba(0, 255, 255, .3);
      color: #00ffff;
      padding: 3px 10px;
      border-radius: 4px;
      font-size: 12px;
    }

    .kbd-desc {
      color: rgba(0, 200, 255, .7);
    }

    #quiz-overlay {
      position: fixed;
      inset: 0;
      background: rgba(0, 0, 0, .9);
      z-index: 400;
      display: none;
      align-items: center;
      justify-content: center;
      flex-direction: column;
    }

    #quiz-overlay.on {
      display: flex;
    }

    .quiz-box {
      background: rgba(0, 5, 25, .98);
      border: 1px solid rgba(0, 255, 255, .3);
      border-radius: 12px;
      padding: 28px 36px;
      width: 420px;
      text-align: center;
    }

    .quiz-title {
      font-size: 20px;
      letter-spacing: 4px;
      color: #00ffff;
      margin-bottom: 6px;
    }

    .quiz-score {
      font-size: 13px;
      color: rgba(0, 200, 255, .5);
      margin-bottom: 20px;
    }

    .quiz-canvas {
      border-radius: 50%;
      margin: 0 auto 20px;
      display: block;
    }

    .quiz-opts {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-top: 16px;
    }

    .quiz-opt {
      background: rgba(0, 255, 255, .07);
      border: 1px solid rgba(0, 255, 255, .25);
      color: #00ffff;
      font-family: 'Courier New', monospace;
      font-size: 13px;
      padding: 10px;
      border-radius: 6px;
      cursor: pointer;
      transition: all .2s;
    }

    .quiz-opt.correct {
      background: rgba(0, 200, 100, .2);
      border-color: #00ff88;
      color: #00ff88;
    }

    .quiz-opt.wrong {
      background: rgba(200, 50, 50, .2);
      border-color: #ff6666;
      color: #ff6666;
    }

    .quiz-feedback {
      font-size: 13px;
      min-height: 20px;
      margin-top: 10px;
    }

    #countdown-bar {
      position: fixed;
      top: 0;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(0, 5, 25, .9);
      border-bottom: 1px solid rgba(0, 255, 255, .15);
      padding: 5px 20px;
      font-size: 11px;
      color: rgba(0, 200, 255, .7);
      letter-spacing: 1px;
      z-index: 11;
      white-space: nowrap;
      pointer-events: none;
    }

    #res-legend {
      position: fixed;
      bottom: 200px;
      left: 18px;
      z-index: 9;
      display: none;
      background: rgba(0, 3, 20, .88);
      border: 1px solid rgba(0, 255, 255, .15);
      border-radius: 6px;
      padding: 10px 14px;
      font-size: 11px;
      letter-spacing: 1px;
      line-height: 2;
    }

    #res-legend.on {
      display: block;
    }

    .res-row {
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .res-dot {
      width: 8px;
      height: 8px;
      border-radius: 50%;
      flex-shrink: 0;
    }

    #oort-legend {
      position: fixed;
      bottom: 70px;
      left: 18px;
      z-index: 9;
      display: none;
      font-size: 11px;
      color: #0088bb;
      letter-spacing: 1px;
      line-height: 2.2;
      background: rgba(0, 3, 20, .85);
      border: 1px solid rgba(0, 255, 255, .12);
      border-radius: 6px;
      padding: 10px 14px;
    }

    #oort-legend.on {
      display: block;
    }

    .oleg {
      display: flex;
      align-items: center;
      gap: 7px;
    }

    .oleg-dot {
      width: 9px;
      height: 9px;
      border-radius: 50%;
      flex-shrink: 0;
    }

    #scale-bar {
      position: fixed;
      bottom: 70px;
      right: 70px;
      z-index: 9;
      font-size: 10px;
      color: rgba(0, 200, 255, .5);
      letter-spacing: 1px;
      text-align: right;
      display: none;
    }

    #scale-bar.on {
      display: block;
    }

    #solar-legend {
      position: fixed;
      bottom: 70px;
      left: 18px;
      z-index: 9;
      font-size: 12px;
      color: #0088bb;
      letter-spacing: 1px;
      line-height: 2.2;
    }

    .leg {
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .leg-dot {
      width: 10px;
      height: 10px;
      border-radius: 50%;
      flex-shrink: 0;
    }

    #tip {
      position: fixed;
      background: rgba(0, 6, 28, .95);
      border: 1px solid rgba(0, 255, 255, .28);
      color: #00ffff;
      font-size: 13px;
      padding: 6px 12px;
      border-radius: 4px;
      pointer-events: none;
      z-index: 30;
      display: none;
      letter-spacing: 1px;
      white-space: nowrap;
    }

    #oort-indicator {
      position: fixed;
      top: 52px;
      left: 20px;
      font-size: 10px;
      letter-spacing: 3px;
      color: rgba(0, 200, 255, .4);
      z-index: 10;
      display: none;
    }

    #oort-indicator.on {
      display: block;
    }

    body::after {
      content: '';
      position: fixed;
      inset: 0;
      background: repeating-linear-gradient(0deg, transparent, transparent 2px, rgba(0, 0, 0, .018) 2px, rgba(0, 0, 0, .018) 4px);
      pointer-events: none;
      z-index: 100;
    }

    #panel::-webkit-scrollbar,
    .tool-panel::-webkit-scrollbar,
    .kbd-box::-webkit-scrollbar {
      width: 4px;
    }

    #panel::-webkit-scrollbar-thumb,
    .tool-panel::-webkit-scrollbar-thumb,
    .kbd-box::-webkit-scrollbar-thumb {
      background: rgba(0, 200, 255, .2);
    }

    .section-divider {
      font-size: 10px;
      letter-spacing: 3px;
      color: rgba(0, 200, 255, .3);
      margin: 10px 0 6px;
      padding-top: 8px;
      border-top: 1px solid rgba(0, 255, 255, .08);
    }
  </style>
</head>

<body>


  <div id="countdown-bar"></div>
  <canvas id="canvas"></canvas>
  <div id="hud">
    <img id="eu_logo" src="eurospace_horizon.png" alt="EuroSpace Horizon Logo"
      style="position:absolute;top:5px;left:5px;width:40px;height:auto;">
    <div class="title">✦ ORBITEX</div>
    <div class="title">✦ ORBITEX</div>
    <div class="hud-center">
    </div>
    <div class="hud-r" id="hud-r">DRAG · PINCH · TAP<br>ZOOM: 1.00x</div>
  </div>

  <div id="mode-tabs">
    <button class="mtab active" id="tab-solar" onclick="switchAppMode('solar')">☀ SOLAR SYSTEM</button>
    <button class="mtab" id="tab-oort" onclick="tryFeature('oort','pro',()=>switchAppMode('oort'))">❄OORT
      CLOUD</button>
  </div>
  <div id="sub-modes">
    <div id="solar-pills" style="display:flex;gap:5px;flex-wrap:wrap;justify-content:center">
      <div class="mpill active" onclick="setSolarView('system')" id="m-system">◎ FULL</div>
      <div class="mpill" onclick="setSolarView('inner')" id="m-inner">⊙ INNER</div>
      <div class="mpill" onclick="setSolarView('outer')" id="m-outer">◯ OUTER</div>
      <div class="mpill" onclick="tryFeature('kuiper','pro',()=>setSolarView('kuiper'))" id="m-kuiper">🪐 KUIPER
      </div>
      <div class="mpill" onclick="setSolarView('jupiter')" id="m-jupiter">🟠 JUPITER</div>
      <div class="mpill" onclick="setSolarView('saturn')" id="m-saturn">🪐 SATURN</div>
      <div class="mpill" onclick="setSolarView('uranus')" id="m-uranus">🔵 URANUS</div>
    </div>
    <div id="oort-pills" style="display:none;gap:5px;flex-wrap:wrap;justify-content:center">
      <div class="mpill active" onclick="setOortView('full')" id="o-full">◎ FULL CLOUD</div>
      <div class="mpill" onclick="setOortView('inner')" id="o-inner">⊙ INNER OORT</div>
      <div class="mpill" onclick="setOortView('outer')" id="o-outer">◯ OUTER OORT</div>
      <div class="mpill" onclick="setOortView('hills')" id="o-hills">🌊 HILLS CLOUD</div>
      <div class="mpill" onclick="setOortView('solar')" id="o-solar">☀ SOLAR REF</div>
      <div class="mpill" onclick="setOortView('comets')" id="o-comets">☄ COMETS</div>
    </div>
  </div>
  <div id="search-bar"><input id="search-input" placeholder="🔍  SEARCH..." autocomplete="off" spellcheck="false"></div>
  <div id="search-results"></div>
  <div id="panel">
    <div class="ph">
      <div>
        <div class="pname" id="p-name"></div>
        <div class="ptype" id="p-type"></div>
      </div>
      <button class="pcls" onclick="closePanel()">✕</button>
    </div>
    <div class="pvis" id="p-vis"></div>
    <div id="p-rows"></div>
  </div>
  <div class="tool-panel" id="age-panel">
    <div class="tool-title">🎂 AGE CALCULATOR <button class="tcls" onclick="closeTool('age-panel')">✕</button></div>
    <input type="date" class="tinput" id="age-input">
    <button class="tbtn" onclick="calcAge()">CALCULATE MY AGES</button>
    <div id="age-result"></div>
  </div>
  <div class="tool-panel" id="vis-panel">
    <div class="tool-title">🌙 VISIBLE TONIGHT <button class="tcls" onclick="closeTool('vis-panel')">✕</button></div>
    <div id="vis-result"></div>
  </div>
  <div class="tool-panel" id="grav-panel">
    <div class="tool-title">🌀 GRAVITY SIMULATOR <button class="tcls" onclick="closeTool('grav-panel')">✕</button></div>
    <div style="font-size:12px;color:rgba(0,200,255,.45);line-height:1.8;margin-bottom:12px">Tap space to launch
      objects. Watch gravity in action!</div>
    <div class="slbl"><span>VELOCITY</span><span id="gv-vel-val">40</span></div>
    <input type="range" class="tinput" id="gv-vel" min="1" max="100" value="40"
      style="padding:0;height:4px;margin-bottom:14px"
      oninput="document.getElementById('gv-vel-val').textContent=this.value">
    <div class="slbl"><span>TRAIL LENGTH</span><span id="gv-trail-val">80</span></div>
    <input type="range" class="tinput" id="gv-trail" min="20" max="300" value="80"
      style="padding:0;height:4px;margin-bottom:14px"
      oninput="document.getElementById('gv-trail-val').textContent=this.value">
    <button class="tbtn" id="gv-launch-btn" onclick="toggleGravLaunch()">🚀 TAP SPACE TO LAUNCH</button>
    <button class="tbtn tbtn-red" onclick="clearProjectiles()">✕ CLEAR ALL</button>
    <div id="gv-status"
      style="font-size:12px;color:rgba(0,255,180,.5);margin-top:8px;text-align:center;min-height:16px"></div>
  </div>
  <div class="tool-panel" id="mission-panel">
    <div class="tool-title">🛰 MISSION SIMULATOR <button class="tcls" onclick="closeTool('mission-panel')">✕</button>
    </div>
    <button class="tbtn tbtn-gold" onclick="launchMission('apollo11')">🌕 APOLLO 11 — Moon Landing</button>
    <button class="tbtn tbtn-gold" onclick="launchMission('voyager1')">🛸 VOYAGER 1 — Grand Tour</button>
    <button class="tbtn tbtn-gold" onclick="launchMission('voyager2')">🛸 VOYAGER 2 — Outer Planets</button>
    <button class="tbtn tbtn-gold" onclick="launchMission('cassini')">🪐 CASSINI — Saturn Orbit</button>
    <button class="tbtn tbtn-gold" onclick="launchMission('newhorizons')">❄ NEW HORIZONS — Pluto</button>
    <button class="tbtn tbtn-gold" onclick="launchMission('jwst')">🔭 JAMES WEBB — L2 Point</button>
    <button class="tbtn tbtn-purple" onclick="launchMission('slingshot')">🎯 ORBITAL SLINGSHOT</button>
    <button class="tbtn tbtn-red" onclick="stopMission()">✕ STOP MISSION</button>
    <div id="mission-status"
      style="font-size:12px;color:rgba(0,255,180,.5);margin-top:8px;min-height:40px;line-height:1.8"></div>
  </div>
  <div class="tool-panel" id="collision-panel">
    <div class="tool-title">💥 COLLISION MODE <button class="tcls" onclick="closeTool('collision-panel')">✕</button>
    </div>
    <div style="font-size:12px;color:rgba(0,200,255,.45);line-height:1.8;margin-bottom:12px">Tap two planets to smash
      them together!</div>
    <div id="collision-status"
      style="font-size:13px;color:rgba(0,255,180,.6);min-height:40px;line-height:1.8;margin-bottom:10px">Select first
      planet...</div>
    <button class="tbtn tbtn-red" onclick="resetCollision()">✕ RESET</button>
  </div>
  <div class="tool-panel" id="alien-panel">
    <div class="tool-title">👽 ALIEN PLANET GENERATOR <button class="tcls" onclick="closeTool('alien-panel')">✕</button>
    </div>
    <button class="tbtn tbtn-purple" onclick="generateAlien()">✦ GENERATE ALIEN WORLD</button>
    <button class="tbtn" onclick="addAlienToMap()" id="btn-add-alien" style="display:none">🌍 ADD TO MAP</button>
    <div id="alien-result"></div>
  </div>
  <div class="tool-panel" id="timemachine-panel">
    <div class="tool-title">⏱ TIME MACHINE <button class="tcls" onclick="closeTool('timemachine-panel')">✕</button>
    </div>
    <input type="date" class="tinput" id="tm-input">
    <button class="tbtn" onclick="setTimeMachine()">⏱ JUMP TO DATE</button>
    <button class="tbtn tbtn-red" onclick="resetTimeMachine()">↺ RETURN TO NOW</button>
    <div id="tm-status" style="font-size:12px;color:rgba(0,255,180,.5);margin-top:8px;min-height:20px"></div>
    <div class="section-divider">HISTORIC DATES</div>
    <button class="tbtn" onclick="jumpToDate('1969-07-20')">🌕 Moon Landing (1969)</button>
    <button class="tbtn" onclick="jumpToDate('1977-09-05')">🛸 Voyager 1 Launch (1977)</button>
    <button class="tbtn" onclick="jumpToDate('2006-01-19')">❄ New Horizons Launch (2006)</button>
    <button class="tbtn" onclick="jumpToDate('2015-07-14')">🌑 Pluto Flyby (2015)</button>
  </div>
  <div class="tool-panel" id="compare-tool-panel">
    <div class="tool-title">⚖ SIZE COMPARISON <button class="tcls" onclick="closeTool('compare-tool-panel')">✕</button>
    </div>
    <div id="cmp-status" style="font-size:13px;color:rgba(0,255,180,.6);min-height:20px;margin-bottom:10px">Tap first
      object on map...</div>
    <div id="cmp-content"></div>
    <button class="tbtn tbtn-red" onclick="resetCompare()">✕ RESET</button>
  </div>
  <div class="tool-panel" id="resonance-panel">
    <div class="tool-title">🔗 ORBITAL RESONANCES <button class="tcls" onclick="closeTool('resonance-panel')">✕</button>
    </div>
    <div style="font-size:12px;color:rgba(0,200,255,.45);line-height:1.8;margin-bottom:12px">Two bodies with orbital
      period ratios of small integers.</div>
    <div id="resonance-list"></div>
    <button class="tbtn" onclick="toggleResonance()" id="btn-res-toggle">SHOW ON MAP</button>
  </div>
  <div class="tool-panel" id="oort-info-panel">
    <div class="tool-title">❄ OORT CLOUD INFO <button class="tcls" onclick="closeTool('oort-info-panel')">✕</button>
    </div>
    <div style="font-size:12px;color:rgba(0,200,255,.5);line-height:1.9;margin-bottom:8px">The <span
        style="color:#00ffff">Oort Cloud</span> is a vast spherical shell of icy bodies surrounding our Solar System.
    </div>
    <div id="oort-info-rows"></div>
  </div>
  <div id="grav-hint">🎯 TAP ANYWHERE TO LAUNCH</div>
  <div id="funfact"></div>
  <div id="tour-bar"></div>
  <div id="tip"></div>
  <div id="res-legend"></div>
  <div id="solar-legend">
    <div class="leg">
      <div class="leg-dot" style="background:#ffcc44"></div>PLANET
    </div>
    <div class="leg">
      <div class="leg-dot" style="background:#88aaff"></div>MOON
    </div>
    <div class="leg">
      <div class="leg-dot" style="background:#bbbbbb"></div>DWARF PLANET
    </div>
    <div class="leg">
      <div class="leg-dot" style="background:#44ffcc"></div>COMET
    </div>
    <div class="leg">
      <div class="leg-dot" style="background:#ff88ff"></div>SPACECRAFT
    </div>
    <div class="leg">
      <div class="leg-dot" style="background:#cc88ff"></div>ALIEN WORLD
    </div>
  </div>
  <div id="oort-legend">
    <div class="oleg">
      <div class="oleg-dot" style="background:#44ffcc"></div>LONG-PERIOD COMET
    </div>
    <div class="oleg">
      <div class="oleg-dot" style="background:#ffaa44"></div>SHORT-PERIOD COMET
    </div>
    <div class="oleg">
      <div class="oleg-dot" style="background:#ff88ff"></div>INTERSTELLAR OBJECT
    </div>
    <div class="oleg">
      <div class="oleg-dot" style="background:#4488ff"></div>INNER OORT CANDIDATE
    </div>
    <div class="oleg">
      <div class="oleg-dot" style="background:#ffaaaa"></div>NEARBY STAR
    </div>
  </div>
  <div id="scale-bar"></div>
  <div id="oort-indicator">❄ OORT CLOUD MODE</div>
  <div id="speed-wrap">
    <label>TIME</label>
    <button id="pause-btn" onclick="togglePause()">⏸</button>
    <input type="range" id="speed-slider" min="0" max="100" value="30">
    <div id="speed-val">1×</div>
  </div>
  <div id="sidebar">
    <button class="cbtn" onclick="zoomBy(1.3)"><span>＋</span><span class="cbtn-tip">ZOOM IN</span></button>
    <button class="cbtn" onclick="zoomBy(0.77)"><span>－</span><span class="cbtn-tip">ZOOM OUT</span></button>
    <button class="cbtn" onclick="resetCurrentView()"><span>⌂</span><span class="cbtn-tip">RESET</span></button>
    <button class="cbtn solar-only" id="btn-age"
      onclick="tryFeature('age','pro',()=>toggleTool('age-panel'))"><span>🎂</span><span class="cbtn-tip">AGE
        CALC</span></button>
    <button class="cbtn solar-only" id="btn-vis"
      onclick="tryFeature('vis','pro',()=>toggleTool('vis-panel'))"><span>🌙</span><span class="cbtn-tip">VISIBLE
        TONIGHT</span></button>
    <button class="cbtn solar-only" id="btn-grav"
      onclick="tryFeature('grav','max',()=>toggleTool('grav-panel'))"><span>🌀</span><span class="cbtn-tip">GRAVITY
        SIM</span></button>
    <button class="cbtn solar-only" id="btn-mission"
      onclick="tryFeature('max','max',()=>toggleTool('mission-panel'))"><span>🛰</span><span
        class="cbtn-tip">MISSIONS</span></button>
    <button class="cbtn solar-only" id="btn-collision"
      onclick="tryFeature('collision','max',()=>toggleTool('collision-panel'))"><span>💥</span><span
        class="cbtn-tip">COLLISION</span></button>
    <button class="cbtn solar-only" id="btn-alien"
      onclick="tryFeature('alien','max',()=>toggleTool('alien-panel'))"><span>👽</span><span class="cbtn-tip">ALIEN
        WORLD</span></button>
    <button class="cbtn solar-only" id="btn-tm"
      onclick="tryFeature('tm','max',()=>toggleTool('timemachine-panel'))"><span>⏱</span><span class="cbtn-tip">TIME
        MACHINE</span></button>
    <button class="cbtn solar-only" id="btn-cmp"
      onclick="tryFeature('cmp','max',()=>toggleTool('compare-tool-panel'))"><span>⚖</span><span
        class="cbtn-tip">COMPARE</span></button>
    <button class="cbtn solar-only" id="btn-scale"
      onclick="tryFeature('scale','max',()=>toggleScaleMode())"><span>📐</span><span class="cbtn-tip">TRUE
        SCALE</span></button>
    <button class="cbtn solar-only" id="btn-solar-wind"
      onclick="tryFeature('solarwind','max',()=>toggleSolarWind())"><span>☀</span><span class="cbtn-tip">SOLAR
        WIND</span></button>
    <button class="cbtn solar-only" id="btn-res"
      onclick="tryFeature('res','max',()=>toggleTool('resonance-panel'))"><span>🔗</span><span
        class="cbtn-tip">RESONANCES</span></button>
    <button class="cbtn solar-only" id="btn-quiz" onclick="tryFeature('quiz','pro',()=>startQuiz())"><span>❓</span><span
        class="cbtn-tip">QUIZ
        MODE</span></button>
    <button class="cbtn solar-only" id="btn-tour" onclick="tryFeature('tour','max',()=>startTour())"><span>▶</span><span
        class="cbtn-tip">AUTO
        TOUR</span></button>
    <button class="cbtn oort-only" id="btn-oort-density"
      onclick="tryFeature('oortdensity','max',()=>toggleOortDensity())" style="display:none"><span>🌫</span><span
        class="cbtn-tip">DENSITY MAP</span></button>
    <button class="cbtn oort-only" id="btn-oort-3d" onclick="tryFeature('oort3d','max',()=>toggleOort3D())"
      style="display:none"><span style="font-size:11px">3D</span><span class="cbtn-tip">3D TILT VIEW</span></button>
    <button class="cbtn oort-only" id="btn-oort-info" onclick="showOortInfo()" style="display:none"><span>ℹ</span><span
        class="cbtn-tip">OORT INFO</span></button>
    <button class="cbtn" id="btn-orb" onclick="toggleOrbits()"><span>◎</span><span
        class="cbtn-tip">ORBITS</span></button>
    <button class="cbtn" id="btn-lbl" onclick="toggleLabels()"><span>🏷</span><span
        class="cbtn-tip">LABELS</span></button>

    <button class="cbtn" onclick="showKbd()"><span>?</span><span class="cbtn-tip">SHORTCUTS</span></button>
  </div>
  <div id="kbd-overlay">
    <div class="kbd-box">
      <div class="kbd-title">⌨ SHORTCUTS</div>
      <div class="kbd-row"><span class="kbd-key">TAB</span><span class="kbd-desc">Switch Solar / Oort Cloud</span></div>
      <div class="kbd-row"><span class="kbd-key">P</span><span class="kbd-desc">Pause / Resume</span></div>
      <div class="kbd-row"><span class="kbd-key">R</span><span class="kbd-desc">Reset view</span></div>
      <div class="kbd-row"><span class="kbd-key">G</span><span class="kbd-desc">Gravity simulator</span></div>
      <div class="kbd-row"><span class="kbd-key">T</span><span class="kbd-desc">Auto tour</span></div>
      <div class="kbd-row"><span class="kbd-key">Q</span><span class="kbd-desc">Quiz mode</span></div>
      <div class="kbd-row"><span class="kbd-key">O</span><span class="kbd-desc">Toggle orbits</span></div>
      <div class="kbd-row"><span class="kbd-key">L</span><span class="kbd-desc">Toggle labels</span></div>
      <div class="kbd-row"><span class="kbd-key">1–9, 0</span><span class="kbd-desc">Jump to planets</span></div>
      <div class="kbd-row"><span class="kbd-key">ESC</span><span class="kbd-desc">Close panels</span></div>
      <br><button class="tbtn" onclick="hideKbd()">CLOSE</button>
    </div>
  </div>
  <div id="quiz-overlay">
    <div class="quiz-box">
      <div class="quiz-title">🪐 PLANET QUIZ</div>
      <div class="quiz-score" id="quiz-score">Score: 0 / 0</div>
      <canvas class="quiz-canvas" id="quiz-canvas" width="140" height="140"></canvas>
      <div style="font-size:13px;color:rgba(0,200,255,.6);margin-bottom:8px;letter-spacing:1px">WHICH PLANET IS THIS?
      </div>
      <div class="quiz-opts" id="quiz-opts"></div>
      <div class="quiz-feedback" id="quiz-feedback"></div>
      <button class="tbtn tbtn-red" style="margin-top:14px" onclick="stopQuiz()">EXIT QUIZ</button>
    </div>
  </div>

  <!-- ══════════ LOADER ══════════ -->
  <div id="loader">
    <div class="loader-title">ORBITEX</div>
    <div class="loader-sub">SOLAR SYSTEM & OORT CLOUD</div>
    <canvas id="loader-canvas" width="200" height="200"></canvas>
    <div class="loader-bar-wrap">
      <div class="loader-bar" id="loader-bar"></div>
    </div>
    <div class="loader-status" id="loader-status">INITIALIZING...</div>
  </div>

  <script>
    // ══════════ APP CONFIGURATION ══════════
    let currentTier = 'max';

    function tryFeature(featureKey, reqTier, fn) {
      fn();
    }

    function hasAccess(feature) {
      return true;
    }

    // ══════════ CORE ENGINE (original code below) ══════════
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    let W = canvas.width = innerWidth, H = canvas.height = innerHeight;
    window.addEventListener('resize', () => { W = canvas.width = innerWidth; H = canvas.height = innerHeight; });
    let camX = 0, camY = 0, scale = 1, tCamX = 0, tCamY = 0, tScale = 1;
    let isDrag = false, dx0 = 0, dy0 = 0, cx0 = 0, cy0 = 0;
    let tick = 0, speed = 0.3, paused = false;
    let hovObj = null, selObj = null;
    let showLabels = true, showOrbits = true;
    let mouseScreenX = 0, mouseScreenY = 0;
    let appMode = 'solar';
    let showTempMap = false, showSolarWind = false, showResonance = false, scaleMode = false;
    let gravLaunching = false, collisionMode = false, collisionObj1 = null;
    let compareMode = false, compareObj1 = null, compareObj2 = null;
    let tourMode = false, tourTimer = null, tourIdx = 0;
    let activeMission = null, missionTick = 0, missionPath = [];
    let explosions = [], alienPlanets = [], solarWindParticles = [];
    let lastAlienData = null, projectiles = [], pidCount = 0;
    let oortDensity = false, oort3D = false;
    let quizScore = 0, quizTotal = 0, quizActive = false;

    function switchAppMode(mode) {
      appMode = mode; closePanel();
      document.querySelectorAll('.tool-panel').forEach(p => p.classList.remove('on'));
      document.getElementById('tab-solar').classList.toggle('active', mode === 'solar');
      document.getElementById('tab-oort').classList.toggle('active', mode === 'oort');
      document.getElementById('solar-pills').style.display = mode === 'solar' ? 'flex' : 'none';
      document.getElementById('oort-pills').style.display = mode === 'oort' ? 'flex' : 'none';
      document.getElementById('solar-legend').style.display = mode === 'solar' ? 'block' : 'none';
      document.getElementById('oort-legend').classList.toggle('on', mode === 'oort');
      document.getElementById('scale-bar').classList.toggle('on', mode === 'oort');
      document.getElementById('oort-indicator').classList.toggle('on', mode === 'oort');
      document.querySelectorAll('.solar-only').forEach(b => b.style.display = mode === 'solar' ? 'flex' : 'none');
      document.querySelectorAll('.oort-only').forEach(b => b.style.display = mode === 'oort' ? 'flex' : 'none');
      if (mode === 'solar') setSolarView('system'); else setOortView('full');
    }
    function smoothCam() { camX += (tCamX - camX) * .09; camY += (tCamY - camY) * .09; scale += (tScale - scale) * .09; }
    function zoomBy(f) { tScale = Math.max(.005, Math.min(30, scale * f)); }
    function resetCurrentView() { if (appMode === 'solar') setSolarView('system'); else setOortView('full'); }

    const SOLAR_FACTS = ["☀️ The Sun contains 99.86% of all mass in the Solar System.", "🪐 Saturn's rings are only ~10 metres thick in places.", "🌍 Earth is the densest planet in the Solar System.", "🌕 The Moon drifts away from Earth at 3.8 cm per year.", "💨 Neptune's winds reach 2,100 km/h.", "🌡️ Venus is hotter than Mercury despite being farther from the Sun.", "⭐ Sunlight takes 8 minutes 20 seconds to reach Earth.", "🌀 Jupiter's Great Red Spot has lasted over 350 years.", "🧊 Europa has more water than all of Earth's oceans combined.", "🔭 Uranus rotates on its side — axial tilt is 97.77°.", "🛸 Voyager 1 is the most distant human-made object ever.", "🌌 Pluto's heart-shaped region is made of nitrogen ice.", "🔥 The Sun's core reaches 15 million degrees Celsius.", "🌊 Enceladus shoots water geysers 500 km into space."];
    const OORT_FACTS = ["☄️ The Oort Cloud may contain up to 2 trillion icy objects.", "🌌 The Oort Cloud extends 1/4 of the way to the nearest star.", "❄️ Objects in the Oort Cloud are mostly water ice, ammonia and methane.", "🔭 No Oort Cloud object has ever been directly observed.", "💫 Galactic tides from the Milky Way shape the Oort Cloud.", "⭐ Passing stars can perturb Oort Cloud objects into the inner Solar System.", "☄️ Long-period comets like Hale-Bopp originate from the Oort Cloud.", "🌍 The Oort Cloud is the largest structure associated with our Solar System.", "❄️ Sedna may be the first known inner Oort Cloud object.", "🌠 The Oort Cloud formed from debris ejected by giant planets 4.6 billion years ago."];
    const factEl = document.getElementById('funfact');
    function showFact() { const pool = appMode === 'oort' ? OORT_FACTS : SOLAR_FACTS; factEl.textContent = pool[Math.floor(Math.random() * pool.length)]; factEl.style.opacity = '1'; setTimeout(() => factEl.style.opacity = '0', 5500); }
    setInterval(showFact, 10000); setTimeout(showFact, 4000);

    const EVENTS = [{ name: "Saturn Opposition", date: new Date('2025-09-21') }, { name: "Jupiter Opposition", date: new Date('2025-12-07') }, { name: "Apophis Flyby", date: new Date('2029-04-13') }, { name: "Halley's Comet Perihelion", date: new Date('2061-07-28') }];
    function updateCountdown() { const now = new Date(); const up = EVENTS.filter(e => e.date > now).sort((a, b) => a.date - b.date)[0]; if (up) { const d = Math.ceil((up.date - now) / (1000 * 60 * 60 * 24)); document.getElementById('countdown-bar').textContent = '⏳ NEXT: ' + up.name + ' — ' + d + ' days'; } }
    updateCountdown(); setInterval(updateCountdown, 60000);

    function makeTex(fn) { const c = document.createElement('canvas'); c.width = c.height = 128; fn(c.getContext('2d'), 64, 64, 60); return c; }
    const TEX = {
      sun: makeTex((c, cx, cy, r) => { const g = c.createRadialGradient(cx - 15, cy - 15, 0, cx, cy, r); g.addColorStop(0, '#fffde0'); g.addColorStop(.3, '#ffdd00'); g.addColorStop(.7, '#ff8800'); g.addColorStop(1, '#cc4400'); c.fillStyle = g; c.beginPath(); c.arc(cx, cy, r, 0, Math.PI * 2); c.fill(); }),
      mercury: makeTex((c, cx, cy, r) => { const g = c.createRadialGradient(cx - 10, cy - 10, 0, cx, cy, r); g.addColorStop(0, '#cccccc'); g.addColorStop(.5, '#999999'); g.addColorStop(1, '#555555'); c.fillStyle = g; c.beginPath(); c.arc(cx, cy, r, 0, Math.PI * 2); c.fill(); for (let i = 0; i < 12; i++) { c.fillStyle = 'rgba(70,70,70,.35)'; c.beginPath(); c.arc(cx + Math.cos(i * 1.3) * 38, cy + Math.sin(i * 1.3) * 38, 3 + Math.random() * 5, 0, Math.PI * 2); c.fill(); } }),
      venus: makeTex((c, cx, cy, r) => { const g = c.createRadialGradient(cx - 12, cy - 12, 0, cx, cy, r); g.addColorStop(0, '#fff0b0'); g.addColorStop(.4, '#ffcc55'); g.addColorStop(.8, '#dd8800'); g.addColorStop(1, '#aa5500'); c.fillStyle = g; c.beginPath(); c.arc(cx, cy, r, 0, Math.PI * 2); c.fill(); }),
      earth: makeTex((c, cx, cy, r) => { const g = c.createRadialGradient(cx - 14, cy - 14, 0, cx, cy, r); g.addColorStop(0, '#88ccff'); g.addColorStop(.35, '#2266cc'); g.addColorStop(.7, '#224499'); g.addColorStop(1, '#001144'); c.fillStyle = g; c.beginPath(); c.arc(cx, cy, r, 0, Math.PI * 2); c.fill(); c.fillStyle = '#3a9e3a'; c.beginPath(); c.ellipse(cx + 6, cy - 10, 20, 13, .5, 0, Math.PI * 2); c.fill(); c.fillStyle = '#4ab84a'; c.beginPath(); c.ellipse(cx - 15, cy + 16, 15, 9, -.3, 0, Math.PI * 2); c.fill(); }),
      mars: makeTex((c, cx, cy, r) => { const g = c.createRadialGradient(cx - 12, cy - 12, 0, cx, cy, r); g.addColorStop(0, '#ff9966'); g.addColorStop(.4, '#cc4422'); g.addColorStop(.8, '#992211'); g.addColorStop(1, '#661100'); c.fillStyle = g; c.beginPath(); c.arc(cx, cy, r, 0, Math.PI * 2); c.fill(); }),
      jupiter: makeTex((c, cx, cy, r) => { const g = c.createRadialGradient(cx - 15, cy - 15, 0, cx, cy, r); g.addColorStop(0, '#ffeecc'); g.addColorStop(.3, '#ddaa66'); g.addColorStop(.7, '#cc8833'); g.addColorStop(1, '#884411'); c.fillStyle = g; c.beginPath(); c.arc(cx, cy, r, 0, Math.PI * 2); c.fill();[[14, 'rgba(170,90,35,.38)'], [28, 'rgba(240,200,140,.22)'], [42, 'rgba(150,70,25,.32)'], [54, 'rgba(220,160,75,.24)']].forEach(([y, col]) => { c.strokeStyle = col; c.lineWidth = 11; c.beginPath(); c.ellipse(cx, cy + (y - 34), r * .97, 4.5, 0, 0, Math.PI * 2); c.stroke(); }); c.fillStyle = 'rgba(160,70,35,.55)'; c.beginPath(); c.ellipse(cx + 10, cy + 12, 14, 9, 0, 0, Math.PI * 2); c.fill(); }),
      saturn: makeTex((c, cx, cy, r) => { const g = c.createRadialGradient(cx - 12, cy - 12, 0, cx, cy, r); g.addColorStop(0, '#fff0cc'); g.addColorStop(.35, '#ddcc88'); g.addColorStop(.7, '#bbaa55'); g.addColorStop(1, '#887733'); c.fillStyle = g; c.beginPath(); c.arc(cx, cy, r, 0, Math.PI * 2); c.fill(); }),
      uranus: makeTex((c, cx, cy, r) => { const g = c.createRadialGradient(cx - 12, cy - 12, 0, cx, cy, r); g.addColorStop(0, '#ccffff'); g.addColorStop(.4, '#66ddee'); g.addColorStop(.8, '#2299aa'); g.addColorStop(1, '#115566'); c.fillStyle = g; c.beginPath(); c.arc(cx, cy, r, 0, Math.PI * 2); c.fill(); }),
      neptune: makeTex((c, cx, cy, r) => { const g = c.createRadialGradient(cx - 12, cy - 12, 0, cx, cy, r); g.addColorStop(0, '#8899ff'); g.addColorStop(.35, '#3344dd'); g.addColorStop(.7, '#1122bb'); g.addColorStop(1, '#001188'); c.fillStyle = g; c.beginPath(); c.arc(cx, cy, r, 0, Math.PI * 2); c.fill(); })
    };

    const BODIES = [
      { id: 'sun', name: 'The Sun', type: 'Star', emoji: '☀️', color: '#fff7aa', glowColor: '#ff9900', radius: 28, orbitR: 0, orbitalPeriod: 0, parent: null, tex: 'sun', gravMass: 30000, tempK: 5778, realRadiusKm: 696000, details: { 'Classification': 'G-type Main Sequence', 'Age': '4.603 Gyr', 'Diameter': '1,392,700 km', 'Surface Temp': '5,778 K', 'Core Temp': '~15,000,000 K' } },
      { id: 'mercury', name: 'Mercury', type: 'Terrestrial Planet', emoji: '⚫', color: '#aaaaaa', glowColor: '#888888', radius: 5, orbitR: 80, orbitalPeriod: 0.241, parent: 'sun', angleOffset: 0.8, tex: 'mercury', gravMass: 100, tempK: 440, realRadiusKm: 2440, details: { 'Distance from Sun': '0.387 AU', 'Orbital Period': '87.97 days', 'Diameter': '4,879 km' } },
      { id: 'venus', name: 'Venus', type: 'Terrestrial Planet', emoji: '🟡', color: '#ffcc77', glowColor: '#ffaa33', radius: 8, orbitR: 130, orbitalPeriod: 0.615, parent: 'sun', angleOffset: 2.1, tex: 'venus', gravMass: 800, tempK: 737, realRadiusKm: 6052, details: { 'Distance from Sun': '0.723 AU', 'Orbital Period': '224.7 days', 'Surface Temp': '465°C' } },
      { id: 'earth', name: 'Earth', type: 'Terrestrial Planet', emoji: '🌍', color: '#4488ff', glowColor: '#2266dd', radius: 9, orbitR: 190, orbitalPeriod: 1.0, parent: 'sun', angleOffset: 4.2, tex: 'earth', gravMass: 1000, tempK: 288, realRadiusKm: 6371, details: { 'Distance from Sun': '1.0 AU', 'Orbital Period': '365.25 days', 'Diameter': '12,742 km', 'Notable': 'Only known planet with life' } },
      { id: 'moon', name: 'The Moon', type: 'Natural Satellite', emoji: '🌕', color: '#ccccaa', glowColor: '#aaaaaa', radius: 4, orbitR: 28, orbitalPeriod: 0.0748, parent: 'earth', angleOffset: 1.0, gravMass: 12, tempK: 250, realRadiusKm: 1737, details: { 'Distance from Earth': '384,400 km', 'Orbital Period': '27.32 days', 'First Landing': 'Apollo 11, Jul 20 1969' } },
      { id: 'mars', name: 'Mars', type: 'Terrestrial Planet', emoji: '🔴', color: '#dd5533', glowColor: '#cc3311', radius: 7, orbitR: 260, orbitalPeriod: 1.881, parent: 'sun', angleOffset: 0.5, tex: 'mars', gravMass: 500, tempK: 210, realRadiusKm: 3390, details: { 'Distance from Sun': '1.524 AU', 'Orbital Period': '686.97 days', 'Diameter': '6,779 km', 'Moons': '2' } },
      { id: 'phobos', name: 'Phobos', type: 'Natural Satellite', emoji: '🪨', color: '#997755', glowColor: '#775533', radius: 3, orbitR: 20, orbitalPeriod: 0.00876, parent: 'mars', angleOffset: 0.3, gravMass: 2, tempK: 233, realRadiusKm: 11, details: { 'Discoverer': 'Asaph Hall', 'Discovery': '1877' } },
      { id: 'deimos', name: 'Deimos', type: 'Natural Satellite', emoji: '🪨', color: '#886644', glowColor: '#664422', radius: 2, orbitR: 30, orbitalPeriod: 0.0347, parent: 'mars', angleOffset: 2.5, gravMass: 1, tempK: 233, realRadiusKm: 6, details: { 'Discoverer': 'Asaph Hall', 'Discovery': '1877' } },
      { id: 'ceres', name: 'Ceres', type: 'Dwarf Planet', emoji: '⬜', color: '#bbbbbb', glowColor: '#999999', radius: 5, orbitR: 330, orbitalPeriod: 4.6, parent: 'sun', angleOffset: 1.2, gravMass: 50, tempK: 167, realRadiusKm: 470, details: { 'Discoverer': 'Giuseppe Piazzi', 'Discovery': '1801', 'Diameter': '939.4 km' } },
      { id: 'jupiter', name: 'Jupiter', type: 'Gas Giant', emoji: '🟠', color: '#cc8844', glowColor: '#aa6622', radius: 20, orbitR: 430, orbitalPeriod: 11.86, parent: 'sun', angleOffset: 3.3, tex: 'jupiter', gravMass: 15000, tempK: 165, realRadiusKm: 71492, details: { 'Distance from Sun': '5.2 AU', 'Orbital Period': '11.86 years', 'Diameter': '139,820 km', 'Moons': '95' } },
      { id: 'io', name: 'Io', type: 'Natural Satellite', emoji: '🟡', color: '#ffdd44', glowColor: '#ddaa00', radius: 4, orbitR: 36, orbitalPeriod: 0.00484, parent: 'jupiter', angleOffset: 0.5, gravMass: 20, tempK: 130, realRadiusKm: 1822, details: { 'Notable': 'Most volcanically active body in Solar System' } },
      { id: 'europa', name: 'Europa', type: 'Natural Satellite', emoji: '⚪', color: '#aaccff', glowColor: '#6699cc', radius: 4, orbitR: 52, orbitalPeriod: 0.00971, parent: 'jupiter', angleOffset: 1.8, gravMass: 15, tempK: 102, realRadiusKm: 1561, details: { 'Notable': 'Top life candidate — subsurface ocean' } },
      { id: 'ganymede', name: 'Ganymede', type: 'Natural Satellite', emoji: '🔵', color: '#7799cc', glowColor: '#557799', radius: 5, orbitR: 68, orbitalPeriod: 0.0194, parent: 'jupiter', angleOffset: 3.5, gravMass: 25, tempK: 110, realRadiusKm: 2634, details: { 'Notable': 'Largest moon in the Solar System' } },
      { id: 'callisto', name: 'Callisto', type: 'Natural Satellite', emoji: '⚫', color: '#667788', glowColor: '#445566', radius: 5, orbitR: 85, orbitalPeriod: 0.0457, parent: 'jupiter', angleOffset: 5.0, gravMass: 20, tempK: 134, realRadiusKm: 2410, details: { 'Discoverer': 'Galileo', 'Discovery': '1610' } },
      { id: 'saturn', name: 'Saturn', type: 'Gas Giant', emoji: '🪐', color: '#ddcc88', glowColor: '#bbaa66', radius: 17, orbitR: 560, orbitalPeriod: 29.46, parent: 'sun', angleOffset: 1.7, hasRings: true, tex: 'saturn', gravMass: 8000, tempK: 134, realRadiusKm: 60268, details: { 'Distance from Sun': '9.58 AU', 'Orbital Period': '29.46 years', 'Diameter': '116,460 km', 'Moons': '146' } },
      { id: 'titan', name: 'Titan', type: 'Natural Satellite', emoji: '🟤', color: '#cc9944', glowColor: '#aa7722', radius: 5, orbitR: 46, orbitalPeriod: 0.0437, parent: 'saturn', angleOffset: 2.2, gravMass: 22, tempK: 94, realRadiusKm: 2575, details: { 'Notable': 'Only moon with a dense atmosphere' } },
      { id: 'enceladus', name: 'Enceladus', type: 'Natural Satellite', emoji: '⚪', color: '#eeeeff', glowColor: '#aaaadd', radius: 3, orbitR: 30, orbitalPeriod: 0.013, parent: 'saturn', angleOffset: 4.5, gravMass: 5, tempK: 75, realRadiusKm: 252, details: { 'Notable': 'Active water geysers' } },
      { id: 'mimas', name: 'Mimas', type: 'Natural Satellite', emoji: '⚫', color: '#bbbbcc', glowColor: '#888899', radius: 3, orbitR: 22, orbitalPeriod: 0.00944, parent: 'saturn', angleOffset: 1.1, gravMass: 3, tempK: 64, realRadiusKm: 198, details: { 'Diameter': '396 km' } },
      { id: 'rhea', name: 'Rhea', type: 'Natural Satellite', emoji: '⚪', color: '#ddddee', glowColor: '#aaaacc', radius: 4, orbitR: 42, orbitalPeriod: 0.0122, parent: 'saturn', angleOffset: 5.2, gravMass: 8, tempK: 76, realRadiusKm: 764, details: { 'Diameter': '1,527 km' } },
      { id: 'dione', name: 'Dione', type: 'Natural Satellite', emoji: '⚪', color: '#ccccdd', glowColor: '#aaaaaa', radius: 3, orbitR: 36, orbitalPeriod: 0.00655, parent: 'saturn', angleOffset: 3.3, gravMass: 6, tempK: 87, realRadiusKm: 562, details: { 'Diameter': '1,123 km' } },
      { id: 'uranus', name: 'Uranus', type: 'Ice Giant', emoji: '🔵', color: '#88ddee', glowColor: '#44aacc', radius: 13, orbitR: 680, orbitalPeriod: 84.01, parent: 'sun', angleOffset: 5.5, hasRings: true, ringThin: true, tex: 'uranus', gravMass: 2000, tempK: 76, realRadiusKm: 25362, details: { 'Distance from Sun': '19.2 AU', 'Axial Tilt': '97.77°', 'Diameter': '50,724 km' } },
      { id: 'miranda', name: 'Miranda', type: 'Natural Satellite', emoji: '⚫', color: '#aabbcc', glowColor: '#778899', radius: 3, orbitR: 22, orbitalPeriod: 0.00513, parent: 'uranus', angleOffset: 0.7, gravMass: 2, tempK: 60, realRadiusKm: 236, details: { 'Diameter': '471 km' } },
      { id: 'ariel', name: 'Ariel', type: 'Natural Satellite', emoji: '⚪', color: '#bbccdd', glowColor: '#889aaa', radius: 3, orbitR: 30, orbitalPeriod: 0.011, parent: 'uranus', angleOffset: 2.2, gravMass: 5, tempK: 60, realRadiusKm: 579, details: { 'Diameter': '1,158 km' } },
      { id: 'umbriel', name: 'Umbriel', type: 'Natural Satellite', emoji: '⚫', color: '#556677', glowColor: '#334455', radius: 3, orbitR: 38, orbitalPeriod: 0.0175, parent: 'uranus', angleOffset: 4.0, gravMass: 5, tempK: 60, realRadiusKm: 585, details: { 'Diameter': '1,169 km' } },
      { id: 'titania', name: 'Titania', type: 'Natural Satellite', emoji: '⚪', color: '#9aabbc', glowColor: '#778899', radius: 4, orbitR: 47, orbitalPeriod: 0.0238, parent: 'uranus', angleOffset: 1.5, gravMass: 10, tempK: 60, realRadiusKm: 789, details: { 'Diameter': '1,578 km' } },
      { id: 'oberon', name: 'Oberon', type: 'Natural Satellite', emoji: '⚫', color: '#8899aa', glowColor: '#556677', radius: 4, orbitR: 57, orbitalPeriod: 0.0369, parent: 'uranus', angleOffset: 3.8, gravMass: 9, tempK: 60, realRadiusKm: 762, details: { 'Diameter': '1,523 km' } },
      { id: 'neptune', name: 'Neptune', type: 'Ice Giant', emoji: '🔵', color: '#4455ff', glowColor: '#2233dd', radius: 12, orbitR: 790, orbitalPeriod: 164.8, parent: 'sun', angleOffset: 3.8, tex: 'neptune', gravMass: 2000, tempK: 72, realRadiusKm: 24622, details: { 'Distance from Sun': '30.1 AU', 'Diameter': '49,244 km', 'Wind Speed': '2,100 km/h' } },
      { id: 'triton', name: 'Triton', type: 'Natural Satellite', emoji: '⚫', color: '#6688aa', glowColor: '#445577', radius: 4, orbitR: 32, orbitalPeriod: 0.0163, parent: 'neptune', angleOffset: 1.4, gravMass: 15, tempK: 38, realRadiusKm: 1354, details: { 'Notable': 'Retrograde orbit — likely captured KBO' } },
      { id: 'pluto', name: 'Pluto', type: 'Dwarf Planet', emoji: '🟤', color: '#cc9966', glowColor: '#aa7744', radius: 5, orbitR: 890, orbitalPeriod: 248, parent: 'sun', angleOffset: 2.9, gravMass: 5, tempK: 44, realRadiusKm: 1188, details: { 'Distance from Sun': '39.5 AU', 'Diameter': '2,377 km' } },
      { id: 'charon', name: 'Charon', type: 'Natural Satellite', emoji: '⚫', color: '#998877', glowColor: '#776655', radius: 3, orbitR: 24, orbitalPeriod: 0.0174, parent: 'pluto', angleOffset: 0.5, gravMass: 3, tempK: 53, realRadiusKm: 606, details: { 'Diameter': '1,212 km' } },
      { id: 'eris', name: 'Eris', type: 'Dwarf Planet', emoji: '⚪', color: '#dddddd', glowColor: '#bbbbbb', radius: 5, orbitR: 980, orbitalPeriod: 559, parent: 'sun', angleOffset: 5.1, gravMass: 5, tempK: 30, realRadiusKm: 1163, details: { 'Distance from Sun': '67.7 AU', 'Notable': 'Triggered Pluto reclassification' } },
      { id: 'makemake', name: 'Makemake', type: 'Dwarf Planet', emoji: '🟥', color: '#cc6644', glowColor: '#aa4422', radius: 4, orbitR: 1040, orbitalPeriod: 310, parent: 'sun', angleOffset: 1.5, gravMass: 3, tempK: 32, realRadiusKm: 715, details: { 'Discovery': '2005', 'Diameter': '~1,430 km' } },
      { id: 'haumea', name: 'Haumea', type: 'Dwarf Planet', emoji: '🥚', color: '#ddccaa', glowColor: '#bbaa88', radius: 4, orbitR: 1000, orbitalPeriod: 285, parent: 'sun', angleOffset: 3.7, gravMass: 3, tempK: 32, realRadiusKm: 780, details: { 'Discovery': '2004', 'Notable': 'Egg-shaped; has ring' } },
      { id: 'sedna', name: 'Sedna', type: 'Dwarf Planet Candidate', emoji: '🔴', color: '#993322', glowColor: '#771100', radius: 4, orbitR: 1130, orbitalPeriod: 11400, parent: 'sun', angleOffset: 0.7, gravMass: 2, tempK: 12, realRadiusKm: 498, details: { 'Distance from Sun': '84-937 AU', 'Notable': 'First known inner Oort Cloud candidate' } },
      { id: 'halley', name: "Halley's Comet", type: 'Comet', emoji: '☄️', color: '#44ffcc', glowColor: '#22ddaa', radius: 5, orbitR: 580, orbitalPeriod: 75.3, parent: 'sun', angleOffset: 1.9, isComet: true, gravMass: 1, tempK: 180, realRadiusKm: 7.5, details: { 'Orbital Period': '75-76 years', 'Last Perihelion': 'Feb 9, 1986', 'Next': '~2061' } },
      { id: 'hale-bopp', name: 'Hale-Bopp', type: 'Comet', emoji: '☄️', color: '#88ffee', glowColor: '#44ddcc', radius: 4, orbitR: 850, orbitalPeriod: 2520, parent: 'sun', angleOffset: 4.3, isComet: true, gravMass: 1, tempK: 150, realRadiusKm: 30, details: { 'Discovery': 'Jul 23, 1995', 'Orbital Period': '~2,520 years' } },
      { id: 'voyager1', name: 'Voyager 1', type: 'Spacecraft', emoji: '🛸', color: '#ff88ff', glowColor: '#dd44dd', radius: 4, orbitR: 1350, orbitalPeriod: 0, parent: 'sun', angleOffset: 0.4, isSpacecraft: true, gravMass: 0, tempK: 3, realRadiusKm: 0, details: { 'Launched': 'Sep 5, 1977', 'Status': 'Interstellar space (2012)' } },
      { id: 'voyager2', name: 'Voyager 2', type: 'Spacecraft', emoji: '🛸', color: '#ff66ff', glowColor: '#cc22cc', radius: 4, orbitR: 1200, orbitalPeriod: 0, parent: 'sun', angleOffset: 5.9, isSpacecraft: true, gravMass: 0, tempK: 3, realRadiusKm: 0, details: { 'Launched': 'Aug 20, 1977', 'Notable': 'Only craft to visit all 4 outer planets' } },
      { id: 'newhorizons', name: 'New Horizons', type: 'Spacecraft', emoji: '🛸', color: '#ffaaff', glowColor: '#ee88ee', radius: 4, orbitR: 1050, orbitalPeriod: 0, parent: 'sun', angleOffset: 3.2, isSpacecraft: true, gravMass: 0, tempK: 3, realRadiusKm: 0, details: { 'Launched': 'Jan 19, 2006', 'Pluto Flyby': 'Jul 14, 2015' } },
      { id: 'oumuamua_solar', name: 'Oumuamua', type: 'Interstellar Object', emoji: '🛸', color: '#ff44ff', glowColor: '#cc22cc', radius: 3, orbitR: 600, orbitalPeriod: 0, parent: 'sun', angleOffset: 4.5, isComet: false, gravMass: 0, tempK: 40, realRadiusKm: 0.1, details: { 'Discovery': 'Oct 2017', 'Note': 'First confirmed interstellar object' } },
      { id: 'borisov_solar', name: '2I/Borisov', type: 'Interstellar Comet', emoji: '☄️', color: '#ff88ff', glowColor: '#dd44dd', radius: 3, orbitR: 700, orbitalPeriod: 0, parent: 'sun', angleOffset: 2.0, isComet: true, gravMass: 0, tempK: 40, realRadiusKm: 0.5, details: { 'Discovery': 'Aug 2019', 'Note': 'Second interstellar object' } },
      { id: 'atlas_3i', name: '3I/Atlas', type: 'Interstellar Comet', emoji: '☄️', color: '#44ffcc', glowColor: '#22ddaa', radius: 3, orbitR: 800, orbitalPeriod: 0, parent: 'sun', angleOffset: 1.2, isComet: true, gravMass: 0, tempK: 40, realRadiusKm: 0.5, details: { 'Note': 'Third interstellar candidate' } },
    ];
    const BM = {}; BODIES.forEach(b => BM[b.id] = b);

    const TEMP_COLORS = {};
    (function () { BODIES.forEach(b => { if (!b.tempK) return; const t = Math.max(0, Math.min(1, (b.tempK - 10) / (6000 - 10))); let r, g, bl; if (t < .25) { r = Math.floor(t * 4 * 40); g = Math.floor(t * 4 * 170); bl = 255; } else if (t < .5) { const s = (t - .25) * 4; r = Math.floor(40 + s * 215); g = Math.floor(170 + s * 85); bl = Math.floor(255 - s * 255); } else if (t < .75) { const s = (t - .5) * 4; r = 255; g = Math.floor(255 - s * 125); bl = 0; } else { const s = (t - .75) * 4; r = 255; g = Math.floor(130 + s * 125); bl = Math.floor(s * 100); } r = Math.min(255, Math.max(0, r)); g = Math.min(255, Math.max(0, g)); bl = Math.min(255, Math.max(0, bl)); TEMP_COLORS[b.id] = { r, g, b: bl, css: 'rgb(' + r + ',' + g + ',' + bl + ')' }; }); })();

    const ASTEROIDS = Array.from({ length: 440 }, () => { const a = Math.random() * Math.PI * 2, r = 295 + Math.random() * 60; return { angle: a, r, speed: .0001 + Math.random() * .0002, size: .6 + Math.random() * 1.8, bright: Math.random() }; });
    const STARS = Array.from({ length: 800 }, () => ({ x: Math.random(), y: Math.random(), r: Math.random() * 1.4 + .2, b: Math.random(), t: Math.random() * Math.PI * 2, c: ['#ffffff', '#aaccff', '#ffddcc', '#ccffff'][Math.floor(Math.random() * 4)] }));

    function initSolarWind() { solarWindParticles = Array.from({ length: 200 }, () => { const a = Math.random() * Math.PI * 2, d = 30 + Math.random() * 20; return { x: Math.cos(a) * d, y: Math.sin(a) * d, vx: Math.cos(a) * (.5 + Math.random()), vy: Math.sin(a) * (.5 + Math.random()), life: 1 }; }); }
    initSolarWind();
    function updateSolarWind() { solarWindParticles.forEach(p => { p.x += p.vx * speed; p.y += p.vy * speed; const d = Math.sqrt(p.x * p.x + p.y * p.y); p.life = 1 - Math.min(1, d / 1200); if (d > 1400 || p.life <= 0) { const a = Math.random() * Math.PI * 2, dist = 30 + Math.random() * 20; p.x = Math.cos(a) * dist; p.y = Math.sin(a) * dist; p.vx = Math.cos(a) * (.5 + Math.random()); p.vy = Math.sin(a) * (.5 + Math.random()); p.life = 1; } }); }
    function drawSolarWind() { if (!showSolarWind) return; solarWindParticles.forEach(p => { const [sx, sy] = w2s(p.x, p.y); if (sx < 0 || sx > W || sy < 0 || sy > H) return; ctx.globalAlpha = p.life * .4; ctx.fillStyle = '#ffaa44'; ctx.beginPath(); ctx.arc(sx, sy, 1.5 * Math.min(scale, 2), 0, Math.PI * 2); ctx.fill(); }); ctx.globalAlpha = 1; }

    const G = 0.006;
    const PROJ_COLORS = ['#ff4444', '#ff8844', '#ffcc44', '#44ff88', '#44ffcc', '#44aaff', '#8844ff', '#ff44aa', '#ff44ff', '#00ffff', '#ffff44', '#88ff44'];
    function spawnProjectile(wx, wy) { const vel = parseFloat(document.getElementById('gv-vel').value) / 400; const ang = Math.random() * Math.PI * 2; const col = PROJ_COLORS[pidCount % PROJ_COLORS.length]; projectiles.push({ id: pidCount++, x: wx, y: wy, vx: Math.cos(ang) * vel, vy: Math.sin(ang) * vel, trail: [], dead: false, r: parseInt(col.slice(1, 3), 16), g: parseInt(col.slice(3, 5), 16), b: parseInt(col.slice(5, 7), 16) }); updateGravStatus(); }
    function updateProjectiles() { if (paused) return; projectiles.forEach(p => { if (p.dead) return; p.trail.push({ x: p.x, y: p.y }); const maxT = parseInt(document.getElementById('gv-trail').value) || 80; if (p.trail.length > maxT) p.trail.shift(); for (let i = 0; i < BODIES.length; i++) { const b = BODIES[i]; if (!b.gravMass || b.gravMass < 1) continue; const bp = getBodyPos(b); const dx = bp.x - p.x, dy = bp.y - p.y; const d2 = dx * dx + dy * dy, d = Math.sqrt(d2); if (d < b.radius + 1) { p.dead = true; break; } const f = G * b.gravMass / d2; p.vx += f * (dx / d) * speed; p.vy += f * (dy / d) * speed; } if (p.dead) return; p.x += p.vx * speed; p.y += p.vy * speed; if (Math.abs(p.x) > 5000 || Math.abs(p.y) > 5000) p.dead = true; }); projectiles.forEach(p => { if (p.dead && p.trail.length > 0) p.trail.splice(0, 3); }); const before = projectiles.length; projectiles = projectiles.filter(p => !(p.dead && p.trail.length === 0)); if (projectiles.length !== before) updateGravStatus(); }
    function drawProjectiles() { projectiles.forEach(p => { if (p.trail.length < 2) return; for (let i = 1; i < p.trail.length; i++) { const [sx1, sy1] = w2s(p.trail[i - 1].x, p.trail[i - 1].y); const [sx2, sy2] = w2s(p.trail[i].x, p.trail[i].y); const t = i / p.trail.length, alpha = t * (p.dead ? .4 : .9); ctx.strokeStyle = 'rgba(' + p.r + ',' + p.g + ',' + p.b + ',' + alpha.toFixed(2) + ')'; ctx.lineWidth = t * 2.5; ctx.lineCap = 'round'; ctx.beginPath(); ctx.moveTo(sx1, sy1); ctx.lineTo(sx2, sy2); ctx.stroke(); } if (!p.dead) { const [sx, sy] = w2s(p.x, p.y); const gs = Math.max(5, 8 * Math.min(scale, 2)); const grd = ctx.createRadialGradient(sx, sy, 0, sx, sy, gs); grd.addColorStop(0, 'rgba(255,255,255,1)'); grd.addColorStop(.3, 'rgba(' + p.r + ',' + p.g + ',' + p.b + ',0.9)'); grd.addColorStop(1, 'rgba(' + p.r + ',' + p.g + ',' + p.b + ',0)'); ctx.fillStyle = grd; ctx.beginPath(); ctx.arc(sx, sy, gs, 0, Math.PI * 2); ctx.fill(); } }); }
    function clearProjectiles() { projectiles = []; pidCount = 0; updateGravStatus(); }
    function updateGravStatus() { const alive = projectiles.filter(p => !p.dead).length; document.getElementById('gv-status').textContent = projectiles.length ? (alive + ' active · ' + (projectiles.length - alive) + ' fading') : ''; }
    function toggleGravLaunch() { gravLaunching = !gravLaunching; const btn = document.getElementById('gv-launch-btn'), hint = document.getElementById('grav-hint'); if (gravLaunching) { btn.classList.add('active'); btn.textContent = '🎯 LAUNCHING...'; hint.style.display = 'block'; } else { btn.classList.remove('active'); btn.textContent = '🚀 TAP SPACE TO LAUNCH'; hint.style.display = 'none'; } }
    function drawLaunchCursor() { if (!gravLaunching) return; const sx = mouseScreenX, sy = mouseScreenY; ctx.strokeStyle = 'rgba(0,255,200,.6)'; ctx.lineWidth = 1; ctx.setLineDash([4, 4]); ctx.beginPath(); ctx.arc(sx, sy, 14, 0, Math.PI * 2); ctx.stroke(); ctx.setLineDash([]); ctx.fillStyle = 'rgba(0,255,200,1)'; ctx.beginPath(); ctx.arc(sx, sy, 3, 0, Math.PI * 2); ctx.fill(); }

    function spawnExplosion(wx, wy, size) { const parts = Array.from({ length: 80 }, () => { const a = Math.random() * Math.PI * 2, s = 1 + Math.random() * size * .02; return { x: wx, y: wy, vx: Math.cos(a) * s, vy: Math.sin(a) * s, life: 1, maxR: 2 + Math.random() * size * .1, color: ['#ff4400', '#ff8800', '#ffcc00', '#ffffff', '#ff2200'][Math.floor(Math.random() * 5)] }; }); explosions.push({ parts, age: 0, maxAge: 120 }); }
    function updateExplosions() { explosions.forEach(e => { e.age++; e.parts.forEach(p => { p.x += p.vx * speed; p.y += p.vy * speed; p.vx *= .97; p.vy *= .97; p.life = 1 - e.age / e.maxAge; }); }); explosions = explosions.filter(e => e.age < e.maxAge); }
    function drawExplosions() { explosions.forEach(e => { e.parts.forEach(p => { const [sx, sy] = w2s(p.x, p.y); ctx.globalAlpha = p.life * .9; ctx.fillStyle = p.color; ctx.beginPath(); ctx.arc(sx, sy, p.maxR * p.life * Math.min(scale, 3), 0, Math.PI * 2); ctx.fill(); }); }); ctx.globalAlpha = 1; }
    function handleCollisionClick(obj) { if (!obj || obj.type === 'Natural Satellite' || obj.type === 'Spacecraft') return; if (!collisionObj1) { collisionObj1 = obj; document.getElementById('collision-status').textContent = '✓ Selected: ' + obj.name + '\nNow tap a second planet...'; } else if (obj !== collisionObj1) { document.getElementById('collision-status').textContent = '💥 COLLISION: ' + collisionObj1.name + ' + ' + obj.name; const p1 = getBodyPos(collisionObj1), p2 = getBodyPos(obj); spawnExplosion((p1.x + p2.x) / 2, (p1.y + p2.y) / 2, Math.max(collisionObj1.radius, obj.radius) * 2); setTimeout(() => { collisionObj1 = null; document.getElementById('collision-status').textContent = 'Select first planet...'; }, 3000); } }
    function resetCollision() { collisionObj1 = null; document.getElementById('collision-status').textContent = 'Select first planet...'; }

    const ALIEN_NAMES = ['Zorveth', 'Xandria', 'Krython', 'Peluvos', 'Nimara', 'Quelox', 'Draveth', 'Sylonia', 'Threxus', 'Velmara', 'Corenth', 'Dralix'];
    const ALIEN_TYPES = ['Super-Earth', 'Hot Jupiter', 'Ice World', 'Lava World', 'Ocean World', 'Desert World', 'Gas Dwarf', 'Hycean World'];
    const ALIEN_ATMOS = ['Methane/Ammonia', 'Hydrogen/Helium', 'CO₂/Sulfur', 'Nitrogen/Oxygen', 'None (bare rock)', 'Water Vapor'];
    const ALIEN_LIFE = ['None detected', 'Possible microbial', 'Confirmed microbial', 'Possible complex', 'Intelligent signals?'];
    const ALIEN_PALETTES = [[[255, 80, 40], [200, 40, 20], [120, 20, 10]], [[40, 160, 255], [20, 80, 200], [10, 40, 120]], [[80, 220, 120], [40, 160, 80], [20, 80, 40]], [[200, 100, 255], [140, 60, 200], [80, 30, 120]], [[255, 200, 60], [200, 140, 20], [120, 80, 10]], [[60, 220, 220], [30, 160, 180], [15, 80, 100]]];
    function generateAlienTexture(pal) { const c = document.createElement('canvas'); c.width = c.height = 120; const x = c.getContext('2d'); const [c1, c2, c3] = pal; const g = x.createRadialGradient(38, 38, 0, 60, 60, 58); g.addColorStop(0, 'rgba(' + Math.min(255, c1[0] + 60) + ',' + Math.min(255, c1[1] + 60) + ',' + Math.min(255, c1[2] + 60) + ',1)'); g.addColorStop(.5, 'rgba(' + c1[0] + ',' + c1[1] + ',' + c1[2] + ',1)'); g.addColorStop(.8, 'rgba(' + c2[0] + ',' + c2[1] + ',' + c2[2] + ',1)'); g.addColorStop(1, 'rgba(' + c3[0] + ',' + c3[1] + ',' + c3[2] + ',1)'); x.fillStyle = g; x.beginPath(); x.arc(60, 60, 58, 0, Math.PI * 2); x.fill(); for (let i = 0; i < 5; i++) { x.fillStyle = 'rgba(255,255,255,' + (0.05 + Math.random() * .1) + ')'; x.beginPath(); x.ellipse(60, 18 + i * 18, 42 + Math.random() * 8, 3 + Math.random() * 4, 0, 0, Math.PI * 2); x.fill(); } const rim = x.createRadialGradient(60, 60, 50, 60, 60, 62); rim.addColorStop(0, 'rgba(' + c1[0] + ',' + c1[1] + ',' + c1[2] + ',0)'); rim.addColorStop(1, 'rgba(' + Math.min(255, c1[0] + 80) + ',' + Math.min(255, c1[1] + 80) + ',' + Math.min(255, c1[2] + 80) + ',.35)'); x.fillStyle = rim; x.beginPath(); x.arc(60, 60, 62, 0, Math.PI * 2); x.fill(); x.fillStyle = 'rgba(255,255,255,0.16)'; x.beginPath(); x.ellipse(40, 36, 15, 9, -.5, 0, Math.PI * 2); x.fill(); return c; }
    function generateAlien() { const pal = ALIEN_PALETTES[Math.floor(Math.random() * ALIEN_PALETTES.length)]; const name = ALIEN_NAMES[Math.floor(Math.random() * ALIEN_NAMES.length)] + '-' + (Math.floor(Math.random() * 900) + 100); const type = ALIEN_TYPES[Math.floor(Math.random() * ALIEN_TYPES.length)]; const atmos = ALIEN_ATMOS[Math.floor(Math.random() * ALIEN_ATMOS.length)]; const life = ALIEN_LIFE[Math.floor(Math.random() * ALIEN_LIFE.length)]; const c = pal[0]; const hex = '#' + c[0].toString(16).padStart(2, '0') + c[1].toString(16).padStart(2, '0') + c[2].toString(16).padStart(2, '0'); const tex = generateAlienTexture(pal); lastAlienData = { name, type, tex, color: hex, orbitR: 250 + Math.random() * 700, angleOffset: Math.random() * Math.PI * 2 }; const stats = [['Distance', '~' + (1 + Math.random() * 500).toFixed(1) + ' light-years'], ['Mass', (0.1 + Math.random() * 15).toFixed(2) + '× Earth'], ['Radius', (0.3 + Math.random() * 12).toFixed(1) + '× Earth'], ['Surface Temp', Math.floor(-200 + Math.random() * 1200) + '°C'], ['Atmosphere', atmos], ['Year Length', (0.01 + Math.random() * 5000).toFixed(2) + ' Earth years'], ['Moons', Math.floor(Math.random() * 60)], ['Life Signs', life]]; const prev = document.createElement('canvas'); prev.width = prev.height = 90; prev.style.cssText = 'border-radius:50%;display:block;margin:12px auto;border:1px solid rgba(180,100,255,.4)'; const px = prev.getContext('2d'); px.save(); px.beginPath(); px.arc(45, 45, 44, 0, Math.PI * 2); px.clip(); px.drawImage(tex, 0, 0, 120, 120, 1, 1, 88, 88); px.restore(); const result = document.getElementById('alien-result'); result.innerHTML = '<div style="font-size:14px;color:#fff;font-weight:bold;text-align:center;letter-spacing:2px;margin-bottom:4px">' + name + '</div><div style="font-size:11px;color:rgba(180,100,255,.7);text-align:center;margin-bottom:10px;letter-spacing:1px">' + type + '</div>' + stats.map(([k, v]) => '<div class="dr"><span class="dl">' + k.toUpperCase() + '</span><span class="dv">' + v + '</span></div>').join('') + '<div style="margin-top:8px;font-size:11px;color:rgba(0,200,255,.3)">Procedurally generated</div>'; result.insertBefore(prev, result.firstChild); document.getElementById('btn-add-alien').style.display = 'block'; }
    function addAlienToMap() { if (!lastAlienData) return; alienPlanets.push({ ...lastAlienData, id: 'alien_' + Date.now() }); document.getElementById('btn-add-alien').style.display = 'none'; document.getElementById('alien-result').innerHTML += '<div style="color:#00ff88;font-size:12px;margin-top:8px;text-align:center">✓ Added to map!</div>'; }

    const RESONANCES = [{ a: 'io', b: 'europa', ratio: '1:2', color: '#ffaa44', desc: "Io 2 orbits per Europa 1" }, { a: 'europa', b: 'ganymede', ratio: '1:2', color: '#44ffaa', desc: "Europa 2 orbits per Ganymede 1" }, { a: 'io', b: 'ganymede', ratio: '1:4', color: '#ff44cc', desc: "Io 4 orbits per Ganymede 1" }, { a: 'titan', b: 'dione', ratio: '4:1', color: '#44ccff', desc: 'Titan 1 orbit per Dione 4' }, { a: 'titan', b: 'rhea', ratio: '2:1', color: '#ffcc44', desc: 'Titan 1 orbit per Rhea 2' }];
    function buildResonancePanel() { document.getElementById('resonance-list').innerHTML = RESONANCES.map(r => '<div style="padding:8px 0;border-bottom:1px solid rgba(0,255,255,.07)"><div style="display:flex;align-items:center;gap:8px;margin-bottom:4px"><div style="width:10px;height:10px;border-radius:50%;background:' + r.color + ';flex-shrink:0"></div><span style="color:#fff;font-size:13px;font-weight:bold">' + (BM[r.a] ? BM[r.a].name : '?') + ' : ' + (BM[r.b] ? BM[r.b].name : '?') + '</span><span style="background:rgba(0,255,255,.1);border:1px solid rgba(0,255,255,.2);color:#00ffff;font-size:10px;padding:2px 6px;border-radius:10px">' + r.ratio + '</span></div><div style="font-size:11px;color:rgba(0,200,255,.5);padding-left:18px">' + r.desc + '</div></div>').join(''); }
    function toggleResonance() { showResonance = !showResonance; document.getElementById('btn-res').classList.toggle('on', showResonance); document.getElementById('btn-res-toggle').textContent = showResonance ? 'HIDE FROM MAP' : 'SHOW ON MAP'; document.getElementById('btn-res-toggle').classList.toggle('active', showResonance); const leg = document.getElementById('res-legend'); leg.classList.toggle('on', showResonance); if (showResonance) { leg.innerHTML = '<div style="font-size:10px;letter-spacing:2px;margin-bottom:6px;color:rgba(0,200,255,.7)">RESONANCES</div>' + RESONANCES.map(r => '<div class="res-row"><div class="res-dot" style="background:' + r.color + '"></div><span style="font-size:10px;color:#d5eeff">' + (BM[r.a] ? BM[r.a].name : '?') + ' — ' + r.ratio + ' — ' + (BM[r.b] ? BM[r.b].name : '?') + '</span></div>').join(''); } }
    function drawResonance() { if (!showResonance) return; const dashOff = (tick * .5) % 20; RESONANCES.forEach(res => { const a = BM[res.a], b = BM[res.b]; if (!a || !b) return; const pa = getBodyPos(a), pb = getBodyPos(b); const [sx1, sy1] = w2s(pa.x, pa.y), [sx2, sy2] = w2s(pb.x, pb.y); const on = (sx1 > -50 && sx1 < W + 50 && sy1 > -50 && sy1 < H + 50) || (sx2 > -50 && sx2 < W + 50 && sy2 > -50 && sy2 < H + 50); if (!on) return; ctx.strokeStyle = res.color + '99'; ctx.lineWidth = 1.5; ctx.setLineDash([6, 8]); ctx.lineDashOffset = -dashOff; ctx.beginPath(); ctx.moveTo(sx1, sy1); ctx.lineTo(sx2, sy2); ctx.stroke(); ctx.setLineDash([]); ctx.lineDashOffset = 0; const mx = (sx1 + sx2) / 2, my = (sy1 + sy2) / 2; ctx.font = 'bold 11px Courier New'; ctx.textAlign = 'center'; const tw = ctx.measureText(res.ratio).width; ctx.fillStyle = 'rgba(0,3,18,.9)'; ctx.fillRect(mx - tw / 2 - 5, my - 11, tw + 10, 18); ctx.fillStyle = res.color; ctx.fillText(res.ratio, mx, my + 1); }); }

    function setTimeMachine() { const v = document.getElementById('tm-input').value; if (!v) return; const d = new Date(v); tick = ((d - new Date('2000-01-01')) / (1000 * 60 * 60 * 24)) * 10; document.getElementById('tm-status').textContent = 'Viewing: ' + d.toDateString(); paused = true; document.getElementById('pause-btn').textContent = '▶'; }
    function resetTimeMachine() { tick = 0; paused = false; document.getElementById('pause-btn').textContent = '⏸'; document.getElementById('tm-status').textContent = 'Returned to present.'; }
    function jumpToDate(d) { document.getElementById('tm-input').value = d; setTimeMachine(); }

    const MISSIONS = { apollo11: { name: 'Apollo 11', color: '#ffcc44', desc: 'Earth to Moon 1969', waypoints: [{ id: 'earth', label: 'Launch' }, { id: 'moon', label: 'Lunar Orbit' }, { id: 'moon', label: 'LANDING' }, { id: 'earth', label: 'Splashdown' }] }, voyager1: { name: 'Voyager 1', color: '#ff88ff', desc: 'Grand Tour', waypoints: [{ id: 'earth', label: 'Launch 1977' }, { id: 'jupiter', label: 'Jupiter Flyby' }, { id: 'saturn', label: 'Saturn Flyby' }, { id: 'voyager1', label: 'Interstellar Space' }] }, voyager2: { name: 'Voyager 2', color: '#ff66ff', desc: 'All 4 outer planets', waypoints: [{ id: 'earth', label: 'Launch 1977' }, { id: 'jupiter', label: 'Jupiter Flyby' }, { id: 'saturn', label: 'Saturn Flyby' }, { id: 'uranus', label: 'Uranus Flyby' }, { id: 'neptune', label: 'Neptune Flyby' }, { id: 'voyager2', label: 'Interstellar Space' }] }, cassini: { name: 'Cassini', color: '#ddaaff', desc: 'Saturn Orbital Mission', waypoints: [{ id: 'earth', label: 'Launch 1997' }, { id: 'jupiter', label: 'Jupiter Assist' }, { id: 'saturn', label: 'Saturn Orbit' }, { id: 'titan', label: 'Huygens on Titan' }, { id: 'saturn', label: 'Grand Finale 2017' }] }, newhorizons: { name: 'New Horizons', color: '#ffaaff', desc: 'First mission to Pluto', waypoints: [{ id: 'earth', label: 'Launch 2006' }, { id: 'jupiter', label: 'Jupiter Assist' }, { id: 'pluto', label: 'Pluto Flyby 2015' }, { id: 'newhorizons', label: 'Arrokoth 2019' }] }, jwst: { name: 'James Webb', color: '#88ffaa', desc: 'L2 Observatory', waypoints: [{ id: 'earth', label: 'Launch Dec 2021' }, { id: 'moon', label: 'Moon Distance' }, { id: 'earth', label: 'L2 Point 2022' }] }, slingshot: { name: 'Orbital Slingshot', color: '#44ffcc', desc: 'Jupiter to Saturn!', waypoints: [{ id: 'earth', label: 'Launch' }, { id: 'jupiter', label: 'Jupiter Assist' }, { id: 'saturn', label: 'Saturn Arrival' }] } };
    function launchMission(id) { stopMission(); activeMission = MISSIONS[id]; missionTick = 0; missionPath = []; document.getElementById('mission-status').textContent = '▶ ' + activeMission.name + '\n' + activeMission.desc; setSolarView('system'); }
    function animateMission() { if (!activeMission) return; const wp = activeMission.waypoints; const step = Math.floor(missionTick / 60) % wp.length; const t = (missionTick % 60) / 60; const curr = BM[wp[step].id], next = BM[wp[(step + 1) % wp.length].id]; if (curr && next) { const p1 = getBodyPos(curr), p2 = getBodyPos(next); const mx = p1.x + (p2.x - p1.x) * t, my = p1.y + (p2.y - p1.y) * t; missionPath.push({ x: mx, y: my }); if (missionPath.length > 300) missionPath.shift(); tCamX = mx; tCamY = my; if (missionTick % 60 === 0) document.getElementById('mission-status').textContent = '▶ ' + activeMission.name + '\n' + wp[step].label; } missionTick++; }
    function stopMission() { activeMission = null; missionPath = []; missionTick = 0; document.getElementById('mission-status').textContent = ''; }
    function drawMission() { if (!activeMission || missionPath.length < 2) return; for (let i = 1; i < missionPath.length; i++) { const [sx1, sy1] = w2s(missionPath[i - 1].x, missionPath[i - 1].y); const [sx2, sy2] = w2s(missionPath[i].x, missionPath[i].y); const alpha = (i / missionPath.length) * .85; const hex = Math.floor(alpha * 255).toString(16).padStart(2, '00'); ctx.strokeStyle = activeMission.color + hex; ctx.lineWidth = 2; ctx.beginPath(); ctx.moveTo(sx1, sy1); ctx.lineTo(sx2, sy2); ctx.stroke(); } if (missionPath.length > 0) { const last = missionPath[missionPath.length - 1]; const [sx, sy] = w2s(last.x, last.y); ctx.fillStyle = activeMission.color; ctx.beginPath(); ctx.arc(sx, sy, 6, 0, Math.PI * 2); ctx.fill(); } }

    const TOUR_STOPS = [{ target: 'sun', scale: .8, text: 'The Sun — 99.86% of all Solar System mass.' }, { target: 'mercury', scale: 5, text: 'Mercury — swings from -180°C to +430°C.' }, { target: 'venus', scale: 4, text: 'Venus — hottest planet at 465°C.' }, { target: 'earth', scale: 5, text: 'Earth — only known planet with life.' }, { target: 'moon', scale: 8, text: 'The Moon — first visited July 20, 1969.' }, { target: 'mars', scale: 4, text: 'Mars — Olympus Mons, tallest volcano.' }, { target: 'jupiter', scale: 3, text: 'Jupiter — Great Red Spot 350+ years.' }, { target: 'europa', scale: 8, text: "Europa — ocean twice Earth's size." }, { target: 'saturn', scale: 2.5, text: 'Saturn — rings only 10 metres thick!' }, { target: 'uranus', scale: 3, text: 'Uranus — 97.77° axial tilt.' }, { target: 'neptune', scale: 3, text: 'Neptune — winds 2,100 km/h.' }, { target: 'pluto', scale: 5, text: 'Pluto — heart-shaped nitrogen ice plain.' }, { target: 'system', scale: .3, text: 'The Solar System — 4.6 billion years old.' }];
    function startTour() { tourMode = true; tourIdx = 0; document.getElementById('btn-tour').classList.add('on'); document.getElementById('tour-bar').style.display = 'block'; advanceTour(); }
    function advanceTour() { if (!tourMode || tourIdx >= TOUR_STOPS.length) { stopTour(); return; } const s = TOUR_STOPS[tourIdx]; document.getElementById('tour-bar').innerHTML = '🔭 TOUR (' + (tourIdx + 1) + '/' + TOUR_STOPS.length + ') &nbsp;·&nbsp; ' + s.text; if (s.target === 'system') setSolarView('system'); else { const b = BM[s.target]; if (b) { const p = getBodyPos(b); tCamX = p.x; tCamY = p.y; tScale = s.scale; } } tourIdx++; tourTimer = setTimeout(advanceTour, 5500); }
    function stopTour() { tourMode = false; clearTimeout(tourTimer); document.getElementById('tour-bar').style.display = 'none'; document.getElementById('btn-tour').classList.remove('on'); }

    function handleCompareClick(obj) { if (!obj) return; if (!compareObj1) { compareObj1 = obj; document.getElementById('cmp-status').textContent = '✓ ' + obj.name + ' — tap second...'; } else if (obj !== compareObj1) { compareObj2 = obj; const a = compareObj1, b = compareObj2; const maxR = Math.max(a.realRadiusKm, b.realRadiusKm) || 1; const ra = Math.max(8, (a.realRadiusKm / maxR) * 48), rb = Math.max(8, (b.realRadiusKm / maxR) * 48); document.getElementById('cmp-status').textContent = ''; document.getElementById('cmp-content').innerHTML = '<div style="display:flex;justify-content:space-around;align-items:flex-end;margin-bottom:14px;height:64px"><div style="text-align:center"><div style="width:' + (ra * 2) + 'px;height:' + (ra * 2) + 'px;border-radius:50%;background:' + a.color + ';margin:0 auto;display:flex;align-items:center;justify-content:center;font-size:' + Math.max(10, ra * .5) + 'px">' + a.emoji + '</div><div style="font-size:11px;color:rgba(0,200,255,.6);margin-top:5px">' + a.name + '</div></div><div style="text-align:center"><div style="width:' + (rb * 2) + 'px;height:' + (rb * 2) + 'px;border-radius:50%;background:' + b.color + ';margin:0 auto;display:flex;align-items:center;justify-content:center;font-size:' + Math.max(10, rb * .5) + 'px">' + b.emoji + '</div><div style="font-size:11px;color:rgba(0,200,255,.6);margin-top:5px">' + b.name + '</div></div></div>' + [['Radius', a.realRadiusKm ? a.realRadiusKm.toLocaleString() + ' km' : '?', b.realRadiusKm ? b.realRadiusKm.toLocaleString() + ' km' : '?'], ['Type', a.type, b.type], ['Temp', a.tempK ? a.tempK + 'K' : '?', b.tempK ? b.tempK + 'K' : '?']].map(([k, av, bv]) => '<div class="dr"><span class="dl">' + k + '</span><span style="color:#d5eeff;font-size:11px">' + av + ' vs ' + bv + '</span></div>').join(''); } }
    function resetCompare() { compareObj1 = null; compareObj2 = null; document.getElementById('cmp-status').textContent = 'Tap first object on map...'; document.getElementById('cmp-content').innerHTML = ''; }

    const QUIZ_PLANETS = ['sun', 'mercury', 'venus', 'earth', 'mars', 'jupiter', 'saturn', 'uranus', 'neptune', 'pluto'];
    function startQuiz() { quizScore = 0; quizTotal = 0; quizActive = true; document.getElementById('quiz-overlay').classList.add('on'); nextQuizQ(); }
    function stopQuiz() { quizActive = false; document.getElementById('quiz-overlay').classList.remove('on'); }
    function nextQuizQ() { const all = QUIZ_PLANETS.filter(id => BM[id]); const correct = all[Math.floor(Math.random() * all.length)]; const b = BM[correct]; if (!b) return; const qc = document.getElementById('quiz-canvas'); const qx = qc.getContext('2d'); qx.clearRect(0, 0, 140, 140); if (TEX[correct]) { qx.save(); qx.beginPath(); qx.arc(70, 70, 60, 0, Math.PI * 2); qx.clip(); qx.drawImage(TEX[correct], 10, 10, 120, 120); qx.restore(); } else { const g = qx.createRadialGradient(55, 55, 0, 70, 70, 60); g.addColorStop(0, '#ffffff'); g.addColorStop(.4, b.color); g.addColorStop(1, b.color + '66'); qx.fillStyle = g; qx.beginPath(); qx.arc(70, 70, 60, 0, Math.PI * 2); qx.fill(); } const wrong = all.filter(id => id !== correct).sort(() => Math.random() - .5).slice(0, 3); const opts = [correct, ...wrong].sort(() => Math.random() - .5); document.getElementById('quiz-score').textContent = 'Score: ' + quizScore + ' / ' + quizTotal; document.getElementById('quiz-feedback').textContent = ''; document.getElementById('quiz-opts').innerHTML = opts.map(id => '<button class="quiz-opt" onclick="answerQuiz(\'' + id + '\',\'' + correct + '\')">' + (BM[id] ? BM[id].name : id) + '</button>').join(''); }
    function answerQuiz(chosen, correct) { quizTotal++; const btns = document.querySelectorAll('.quiz-opt'); btns.forEach(b => { if (b.textContent === (BM[chosen] ? BM[chosen].name : chosen)) b.classList.add(chosen === correct ? 'correct' : 'wrong'); if (b.textContent === (BM[correct] ? BM[correct].name : correct)) b.classList.add('correct'); b.disabled = true; }); if (chosen === correct) { quizScore++; document.getElementById('quiz-feedback').style.color = '#00ff88'; document.getElementById('quiz-feedback').textContent = '✓ CORRECT!'; } else { document.getElementById('quiz-feedback').style.color = '#ff6666'; document.getElementById('quiz-feedback').textContent = '✗ It was ' + BM[correct].name + '.'; } document.getElementById('quiz-score').textContent = 'Score: ' + quizScore + ' / ' + quizTotal; setTimeout(() => { if (quizActive) nextQuizQ(); }, 1800); }

    function getBodyPos(b) { if (!b.parent) return { x: 0, y: 0 }; const par = BM[b.parent], pp = getBodyPos(par); if (b.isSpacecraft && b.orbitalPeriod === 0) return { x: pp.x + b.orbitR * Math.cos(b.angleOffset), y: pp.y + b.orbitR * Math.sin(b.angleOffset) }; const angle = b.angleOffset + (tick * .001 * speed) / (b.orbitalPeriod || 1); return { x: pp.x + b.orbitR * Math.cos(angle), y: pp.y + b.orbitR * Math.sin(angle) }; }
    function w2s(wx, wy) { if (appMode === 'oort' && oort3D) { const tilt = 0.45; return [W / 2 + (wx - camX) * scale, H / 2 + (wy * Math.cos(tilt) - wx * .1 * Math.sin(tilt) - camY) * scale]; } return [W / 2 + (wx - camX) * scale, H / 2 + (wy - camY) * scale]; }
    function s2w(sx, sy) { return [camX + (sx - W / 2) / scale, camY + (sy - H / 2) / scale]; }
    const TYPE_COLORS = { 'Star': '#ffcc44', 'Terrestrial Planet': '#4488ff', 'Gas Giant': '#ffaa44', 'Ice Giant': '#44aaff', 'Natural Satellite': '#88aaff', 'Dwarf Planet': '#cccccc', 'Dwarf Planet Candidate': '#667788', 'Comet': '#44ffcc', 'Spacecraft': '#ff88ff', 'Long-Period Comet': '#44ffcc', 'Short-Period Comet': '#ffaa44', 'Oort Cloud Comet': '#44ffcc', 'Inner Oort Candidate': '#4466ff', 'Interstellar Comet': '#ff88ff', 'Interstellar Object': '#ff44ff', 'Nearest Star': '#ffaaaa', 'Nearby Star': '#ffcc88', 'Theoretical Region': '#224488' };
    function typeColor(t) { return TYPE_COLORS[t] || '#00ffff'; }
    function getDisplayRadius(b) { if (!scaleMode) return b.radius; return Math.max(3, Math.min(60, (b.realRadiusKm || b.radius * 100) / 6371 * 8)); }

    function drawBody(b) {
      const pos = getBodyPos(b); const [sx, sy] = w2s(pos.x, pos.y);
      if (sx < -120 || sx > W + 120 || sy < -120 || sy > H + 120) return;
      const isHov = hovObj === b, isSel = selObj === b;
      const r = Math.max(getDisplayRadius(b) * Math.min(scale, 3), b.id === 'sun' ? 12 : b.isComet || b.isSpacecraft ? 2 : 3);
      const dr = r * (isSel ? 1 + .1 * Math.sin(tick * .09) : 1);
      const bodyColor = showTempMap && TEMP_COLORS[b.id] ? TEMP_COLORS[b.id].css : b.color;
      const gc = b.glowColor || b.color; const gs = isHov || isSel ? dr * 5 : dr * 3;
      const grd = ctx.createRadialGradient(sx, sy, 0, sx, sy, gs); grd.addColorStop(0, gc + '99'); grd.addColorStop(.4, gc + '22'); grd.addColorStop(1, gc + '00'); ctx.fillStyle = grd; ctx.beginPath(); ctx.arc(sx, sy, gs, 0, Math.PI * 2); ctx.fill();
      if (b.isComet && dr > 3) { const dx = pos.x, dy = pos.y; const d = Math.sqrt(dx * dx + dy * dy) || 1; const tl = dr * 12; const tx = (dx / d) * tl, ty = (dy / d) * tl; const tg = ctx.createLinearGradient(sx, sy, sx + tx, sy + ty); tg.addColorStop(0, b.color + 'cc'); tg.addColorStop(1, b.color + '00'); ctx.strokeStyle = tg; ctx.lineWidth = dr * 1.6; ctx.beginPath(); ctx.moveTo(sx, sy); ctx.lineTo(sx + tx, sy + ty); ctx.stroke(); }
      if (b.isSpacecraft) ctx.globalAlpha = Math.sin(tick * .12) > .5 ? 1 : .45;
      if (!showTempMap && b.tex && TEX[b.tex] && dr > 5) { ctx.save(); ctx.beginPath(); ctx.arc(sx, sy, dr, 0, Math.PI * 2); ctx.clip(); ctx.drawImage(TEX[b.tex], sx - dr, sy - dr, dr * 2, dr * 2); ctx.restore(); }
      else { const bg = ctx.createRadialGradient(sx - dr * .3, sy - dr * .3, 0, sx, sy, dr); bg.addColorStop(0, '#ffffff'); bg.addColorStop(.35, bodyColor); bg.addColorStop(1, bodyColor + '88'); ctx.fillStyle = bg; ctx.beginPath(); ctx.arc(sx, sy, dr, 0, Math.PI * 2); ctx.fill(); }
      if (b.id === 'earth' && dr > 8) { const ta = tick * .01; ctx.save(); ctx.beginPath(); ctx.arc(sx, sy, dr, 0, Math.PI * 2); ctx.clip(); ctx.fillStyle = 'rgba(0,0,20,.45)'; ctx.beginPath(); ctx.arc(sx + Math.cos(ta) * dr * .3, sy + Math.sin(ta) * dr * .3, dr, 0, Math.PI * 2); ctx.fill(); ctx.restore(); }
      ctx.globalAlpha = 1;
      if (collisionMode && collisionObj1 === b) { ctx.strokeStyle = '#ff4444'; ctx.lineWidth = 3; ctx.beginPath(); ctx.arc(sx, sy, dr + 7, 0, Math.PI * 2); ctx.stroke(); }
      if (compareMode && (compareObj1 === b || compareObj2 === b)) { ctx.strokeStyle = '#ffcc44'; ctx.lineWidth = 2; ctx.beginPath(); ctx.arc(sx, sy, dr + 6, 0, Math.PI * 2); ctx.stroke(); }
      if (isSel || isHov) { ctx.strokeStyle = b.color; ctx.lineWidth = isSel ? 2 : 1; ctx.globalAlpha = isSel ? .9 : .45; ctx.beginPath(); ctx.arc(sx, sy, dr + 5, 0, Math.PI * 2); ctx.stroke(); ctx.globalAlpha = 1; }
      if (showLabels) { const ms = b.parent && b.parent !== 'sun' ? 3 : b.isComet || b.isSpacecraft ? .8 : .5; if (scale > ms || isHov) { ctx.font = (isHov ? 'bold ' : '') + Math.min(13 + scale * .5, 16) + 'px Courier New'; ctx.fillStyle = isHov ? '#ffffff' : 'rgba(140,210,255,.82)'; ctx.textAlign = 'center'; ctx.shadowColor = b.color; ctx.shadowBlur = isHov ? 12 : 5; ctx.fillText(b.name, sx, sy + dr + 16); ctx.shadowBlur = 0; } }
    }

    function drawSolar() {
      const bg = ctx.createRadialGradient(W / 2, H / 2, 0, W / 2, H / 2, Math.max(W, H) * .75); bg.addColorStop(0, '#000018'); bg.addColorStop(1, '#000004'); ctx.fillStyle = bg; ctx.fillRect(0, 0, W, H);
      STARS.forEach(s => { const tw = .5 + .5 * Math.sin(tick * .016 + s.t); ctx.globalAlpha = .2 + .7 * s.b * tw; ctx.fillStyle = s.c; ctx.beginPath(); ctx.arc(s.x * W, s.y * H, s.r, 0, Math.PI * 2); ctx.fill(); }); ctx.globalAlpha = 1;
      if (showSolarWind) { updateSolarWind(); drawSolarWind(); }
      if (showOrbits) { BODIES.forEach(b => { if (!b.parent || b.parent !== 'sun' || b.isSpacecraft) return; const [ox, oy] = w2s(0, 0), isDwarf = b.type.includes('Dwarf') || b.isComet; ctx.strokeStyle = isDwarf ? 'rgba(80,120,80,.07)' : 'rgba(0,100,200,.09)'; ctx.lineWidth = 1; ctx.setLineDash(isDwarf ? [2, 10] : [4, 8]); ctx.beginPath(); ctx.arc(ox, oy, b.orbitR * scale, 0, Math.PI * 2); ctx.stroke(); ctx.setLineDash([]); }); if (scale > 2) BODIES.forEach(b => { if (!b.parent || b.parent === 'sun' || b.isSpacecraft) return; const pp = getBodyPos(BM[b.parent]); const [px, py] = w2s(pp.x, pp.y); ctx.strokeStyle = 'rgba(0,160,220,.07)'; ctx.lineWidth = 1; ctx.setLineDash([2, 6]); ctx.beginPath(); ctx.arc(px, py, b.orbitR * scale, 0, Math.PI * 2); ctx.stroke(); ctx.setLineDash([]); }); }
      ASTEROIDS.forEach(a => { a.angle += a.speed * (paused ? 0 : speed * .1); const ax = a.r * Math.cos(a.angle), ay = a.r * Math.sin(a.angle); const [sx, sy] = w2s(ax, ay); if (sx < 0 || sx > W || sy < 0 || sy > H) return; ctx.globalAlpha = .15 + a.bright * .4; ctx.fillStyle = '#aaaaaa'; ctx.beginPath(); ctx.arc(sx, sy, a.size * (scale * .3 + .2), 0, Math.PI * 2); ctx.fill(); }); ctx.globalAlpha = 1;
      BODIES.forEach(b => { if (!b.hasRings) return; const pos = getBodyPos(b); const [sx, sy] = w2s(pos.x, pos.y); const inner = (b.radius + 5) * scale, outer = (b.radius + (b.ringThin ? 10 : 22)) * scale; if (outer < 2) return; ctx.save(); ctx.translate(sx, sy); ctx.scale(1, .32); for (let ri = inner; ri < outer; ri += 2) { const alpha = b.ringThin ? .13 : (.3 * (1 - (ri - inner) / (outer - inner))); ctx.strokeStyle = b.ringThin ? 'rgba(100,200,255,' + alpha + ')' : 'rgba(210,180,100,' + alpha + ')'; ctx.lineWidth = 2; ctx.beginPath(); ctx.arc(0, 0, ri, 0, Math.PI * 2); ctx.stroke(); } ctx.restore(); });
      drawResonance(); BODIES.forEach(drawBody);
      alienPlanets.forEach(ap => { const angle = ap.angleOffset + (tick * .001 * speed) / 200; const wx = ap.orbitR * Math.cos(angle), wy = ap.orbitR * Math.sin(angle); const [sx, sy] = w2s(wx, wy); const r = 8 * Math.min(scale, 3); if (ap.tex) { ctx.save(); ctx.beginPath(); ctx.arc(sx, sy, r, 0, Math.PI * 2); ctx.clip(); ctx.drawImage(ap.tex, sx - r, sy - r, r * 2, r * 2); ctx.restore(); } else { ctx.fillStyle = ap.color; ctx.beginPath(); ctx.arc(sx, sy, r, 0, Math.PI * 2); ctx.fill(); } if (showLabels && scale > .3) { ctx.font = '11px Courier New'; ctx.fillStyle = 'rgba(180,100,255,.8)'; ctx.textAlign = 'center'; ctx.shadowColor = '#cc88ff'; ctx.shadowBlur = 4; ctx.fillText(ap.name, sx, sy + r + 14); ctx.shadowBlur = 0; } });
      if (activeMission) { animateMission(); drawMission(); }
      updateExplosions(); drawExplosions(); if (!paused) updateProjectiles(); drawProjectiles(); drawLaunchCursor();
    }

    const SOLAR_VIEWS = { system: { cx: 0, cy: 0, s: .3 }, inner: { cx: 0, cy: 0, s: 2.0 }, outer: { cx: 150, cy: 0, s: .7 }, kuiper: { cx: 350, cy: 0, s: .22 }, jupiter: { target: 'jupiter', s: 4 }, saturn: { target: 'saturn', s: 3.5 }, uranus: { target: 'uranus', s: 5 } };
    function setSolarView(v) { const t = SOLAR_VIEWS[v]; if (t && t.target) { const pos = getBodyPos(BM[t.target]); tCamX = pos.x; tCamY = pos.y; } else if (t) { tCamX = t.cx; tCamY = t.cy; } if (t) tScale = t.s; document.querySelectorAll('#solar-pills .mpill').forEach(p => p.classList.remove('active')); const el = document.getElementById('m-' + v); if (el) el.classList.add('active'); closePanel(); }
    function hitTestSolar(mx, my) { for (let i = BODIES.length - 1; i >= 0; i--) { const b = BODIES[i], pos = getBodyPos(b); const [sx, sy] = w2s(pos.x, pos.y); const r = Math.max(getDisplayRadius(b) * Math.min(scale, 3), b.id === 'sun' ? 12 : 3) + 14; if ((mx - sx) ** 2 + (my - sy) ** 2 < r * r) return b; } return null; }

    const OORT_OBJECTS = [
      { id: 'sun_ref', name: 'The Sun', type: 'Star', emoji: '☀️', color: '#fff7aa', glowColor: '#ff9900', x: 0, y: 0, radius: 4, details: { 'Note': 'At this scale the entire Solar System is one pixel', 'Inner Oort Edge': '~2,000 AU', 'Outer Oort Edge': '~200,000 AU' } },
      { id: 'sedna_o', name: 'Sedna', type: 'Inner Oort Candidate', emoji: '🔴', color: '#cc3322', glowColor: '#aa1100', x: 0, y: 0, radius: 3, orbitR: 0.52, orbitalPeriod: 11400, angleOffset: 1.2, details: { 'Distance': '76–937 AU', 'Period': '~11,400 years', 'Significance': 'First known inner Oort object' } },
      { id: 'vp113', name: '2012 VP₁₁₃', type: 'Inner Oort Candidate', emoji: '🔵', color: '#4466cc', glowColor: '#2244aa', x: 0, y: 0, radius: 2.5, orbitR: 0.83, orbitalPeriod: 4300, angleOffset: 3.7, details: { 'Distance': '80–452 AU', 'Period': '~4,300 years' } },
      { id: 'leleakuhonua', name: 'Leleakūhonua', type: 'Inner Oort Candidate', emoji: '🟣', color: '#8855cc', glowColor: '#6633aa', x: 0, y: 0, radius: 2, orbitR: 1.1, orbitalPeriod: 32000, angleOffset: 5.1, details: { 'Distance': '65–2,300 AU', 'Period': '~32,000 years' } },
      { id: 'bernardinelli', name: 'Bernardinelli-Bernstein', type: 'Oort Cloud Comet', emoji: '☄️', color: '#44ffcc', glowColor: '#22ddaa', x: 0, y: 0, radius: 3.5, orbitR: 2.1, orbitalPeriod: 3000000, angleOffset: 2.3, isComet: true, details: { 'Diameter': '~137 km — largest known Oort comet', 'Discovery': 'Jun 19, 2021' } },
      { id: 'hale_bopp_o', name: 'Hale-Bopp', type: 'Long-Period Comet', emoji: '☄️', color: '#88ffee', glowColor: '#44ddcc', x: 0, y: 0, radius: 3, orbitR: 185, orbitalPeriod: 2520, angleOffset: 4.8, isComet: true, details: { 'Perihelion': 'Apr 1, 1997', 'Period': '~2,520 years' } },
      { id: 'halley_o', name: "Halley's Comet", type: 'Short-Period Comet', emoji: '☄️', color: '#ffaa44', glowColor: '#dd8822', x: 0, y: 0, radius: 3, orbitR: 0.17, orbitalPeriod: 75, angleOffset: 0.9, isComet: true, details: { 'Perihelion': 'next ~2061', 'Period': '75–76 years' } },
      { id: 'west', name: 'Comet West', type: 'Long-Period Comet', emoji: '☄️', color: '#44ffcc', glowColor: '#22ddaa', x: 0, y: 0, radius: 2.5, orbitR: 70, orbitalPeriod: 500000, angleOffset: 1.5, isComet: true, details: { 'Perihelion': 'Feb 1976', 'Note': 'One of brightest 20th century comets' } },
      { id: 'mcnaught', name: 'Comet McNaught', type: 'Long-Period Comet', emoji: '☄️', color: '#44ffcc', glowColor: '#22ddaa', x: 0, y: 0, radius: 2.5, orbitR: 90, orbitalPeriod: 92600, angleOffset: 3.2, isComet: true, details: { 'Perihelion': 'Jan 2007', 'Note': 'Visible in daylight' } },
      { id: 'hyakutake', name: 'Comet Hyakutake', type: 'Long-Period Comet', emoji: '☄️', color: '#44ffcc', glowColor: '#22ddaa', x: 0, y: 0, radius: 2.5, orbitR: 120, orbitalPeriod: 70000, angleOffset: 5.5, isComet: true, details: { 'Perihelion': 'May 1996', 'Note': 'First comet seen in X-rays' } },
      { id: 'borisov', name: 'Comet Borisov', type: 'Interstellar Comet', emoji: '🛸', color: '#ff88ff', glowColor: '#dd44dd', x: 0, y: 0, radius: 3, orbitR: 150, orbitalPeriod: 0, angleOffset: 2.0, isComet: true, details: { 'Discovery': 'Aug 2019', 'Note': 'Only confirmed interstellar comet' } },
      { id: 'oumuamua', name: 'ʻOumuamua', type: 'Interstellar Object', emoji: '🛸', color: '#ff44ff', glowColor: '#cc22cc', x: 0, y: 0, radius: 3, orbitR: 160, orbitalPeriod: 0, angleOffset: 4.5, details: { 'Discovery': 'Oct 2017', 'Note': 'First confirmed interstellar object' } },
      { id: 'ikeya_seki', name: 'Ikeya-Seki', type: 'Long-Period Comet', emoji: '☄️', color: '#44ffcc', glowColor: '#22ddaa', x: 0, y: 0, radius: 2.5, orbitR: 165, orbitalPeriod: 880, angleOffset: 1.1, isComet: true, details: { 'Perihelion': 'Oct 1965', 'Note': 'One of brightest comets in 1,000 years' } },
      { id: 'shoemaker_levy', name: 'Shoemaker-Levy 9', type: 'Short-Period Comet', emoji: '💥', color: '#ffaa44', glowColor: '#dd8822', x: 0, y: 0, radius: 2.5, orbitR: 0.04, orbitalPeriod: 2, angleOffset: 3.3, isComet: true, details: { 'Fate': 'Crashed into Jupiter Jul 1994' } },
      { id: 'proxima_o', name: 'Proxima Centauri', type: 'Nearest Star', emoji: '⭐', color: '#ffaaaa', glowColor: '#ff6666', x: 0, y: 0, radius: 4, orbitR: 268.77, orbitalPeriod: 0, angleOffset: 0.7, isFixed: true, details: { 'Distance': '4.2465 light-years (268,770 AU)', 'Type': 'M5.5Ve Red Dwarf' } },
      { id: 'alphaCen', name: 'Alpha Centauri A+B', type: 'Nearby Star', emoji: '⭐', color: '#ffffaa', glowColor: '#ffcc44', x: 0, y: 0, radius: 3.5, orbitR: 277, orbitalPeriod: 0, angleOffset: 0.72, isFixed: true, details: { 'Distance': '4.365 light-years', 'Type': 'G2V + K1V binary' } },
      { id: 'oort_hills_inner', name: 'Hills Cloud Inner Edge', type: 'Theoretical Region', emoji: '🌊', color: '#4488ff', glowColor: '#2266cc', x: 0, y: 0, radius: 0, isRegion: true, regionR: 2, details: { 'Distance': '~2,000 AU' } },
      { id: 'oort_hills_outer', name: 'Hills Cloud Outer Edge', type: 'Theoretical Region', emoji: '🌊', color: '#3366cc', glowColor: '#1144aa', x: 0, y: 0, radius: 0, isRegion: true, regionR: 20, details: { 'Distance': '~20,000 AU' } },
      { id: 'oort_outer_boundary', name: 'Outer Oort Cloud Edge', type: 'Theoretical Region', emoji: '🌐', color: '#1144aa', glowColor: '#003388', x: 0, y: 0, radius: 0, isRegion: true, regionR: 200, details: { 'Distance': '~200,000 AU', 'Contents': '~2 trillion icy bodies' } },
    ];
    const OM = {}; OORT_OBJECTS.forEach(o => OM[o.id] = o);

    const OORT_PARTICLES = [];
    (function () { for (let i = 0; i < 600; i++) { const r = 2 + Math.random() * 18, theta = Math.random() * Math.PI * 2; OORT_PARTICLES.push({ r, theta, size: .8 + Math.random() * 1.4, bright: .3 + Math.random() * .7, type: 'hills', speed: (Math.random() - .5) * .0001, color: '#4488ff' }); } for (let i = 0; i < 1200; i++) { const r = 20 + Math.random() * 180, theta = Math.random() * Math.PI * 2; OORT_PARTICLES.push({ r, theta, size: .4 + Math.random() * .9, bright: .15 + Math.random() * .5, type: 'outer', speed: (Math.random() - .5) * .00002, color: '#88aacc' }); } })();

    function getOortPos(o) { if (o.isFixed || o.isRegion) { if (o.orbitR) { const a = o.angleOffset || 0; return { x: o.orbitR * Math.cos(a), y: o.orbitR * Math.sin(a) }; } return { x: 0, y: 0 }; } if (!o.orbitR) return { x: 0, y: 0 }; const a = o.angleOffset + (o.orbitalPeriod && o.orbitalPeriod > 0 ? (tick * .00005 * speed) / o.orbitalPeriod : 0); return { x: o.orbitR * Math.cos(a), y: o.orbitR * Math.sin(a) }; }

    function drawOortParticles() { OORT_PARTICLES.forEach(p => { if (!paused) p.theta += p.speed * speed; const wx = p.r * Math.cos(p.theta), wy = p.r * Math.sin(p.theta); const [sx, sy] = w2s(wx, wy); if (sx < -5 || sx > W + 5 || sy < -5 || sy > H + 5) return; const tw = .4 + .6 * Math.sin(tick * .02 + p.r); ctx.globalAlpha = (p.bright * tw) * (p.type === 'hills' ? .6 : .3); ctx.fillStyle = p.color; ctx.beginPath(); ctx.arc(sx, sy, Math.max(.4, p.size * (Math.min(scale, .8) + .2)), 0, Math.PI * 2); ctx.fill(); }); ctx.globalAlpha = 1; }
    function drawOortDensityCloud() { if (!oortDensity) return; const gc1 = ctx.createRadialGradient(W / 2, H / 2, 2 * scale, W / 2, H / 2, 20 * scale); gc1.addColorStop(0, 'rgba(0,50,200,0)'); gc1.addColorStop(.3, 'rgba(0,80,200,.06)'); gc1.addColorStop(.7, 'rgba(0,100,255,.04)'); gc1.addColorStop(1, 'rgba(0,50,150,0)'); ctx.fillStyle = gc1; ctx.fillRect(0, 0, W, H); const gc2 = ctx.createRadialGradient(W / 2, H / 2, 20 * scale, W / 2, H / 2, 200 * scale); gc2.addColorStop(0, 'rgba(0,30,80,0)'); gc2.addColorStop(.2, 'rgba(0,60,120,.04)'); gc2.addColorStop(.6, 'rgba(0,80,160,.06)'); gc2.addColorStop(1, 'rgba(0,30,80,0)'); ctx.fillStyle = gc2; ctx.fillRect(0, 0, W, H); }

    function drawOortObject(o) {
      if (o.isRegion) { if (!o.regionR) return; const [cx2, cy2] = w2s(0, 0); const r = o.regionR * scale; if (r < 2) return; ctx.strokeStyle = 'rgba(0,80,180,.14)'; ctx.lineWidth = 1; ctx.setLineDash([6, 10]); ctx.beginPath(); ctx.arc(cx2, cy2, r, 0, Math.PI * 2); ctx.stroke(); ctx.setLineDash([]); if (showLabels && r > 30) { ctx.font = '10px Courier New'; ctx.textAlign = 'center'; ctx.fillStyle = 'rgba(0,150,255,.4)'; ctx.fillText(o.name, cx2, cy2 - r - 6); } return; }
      const pos = getOortPos(o); const [sx, sy] = w2s(pos.x, pos.y); if (sx < -80 || sx > W + 80 || sy < -80 || sy > H + 80) return;
      const isHov = hovObj === o, isSel = selObj === o; const r = Math.max(o.radius * Math.min(scale * 2, 4), o.id === 'sun_ref' ? 8 : 3); const dr = r * (isSel ? 1 + .1 * Math.sin(tick * .09) : 1);
      const gc = o.glowColor || o.color || '#00ffff'; const gs = isHov || isSel ? dr * 5 : dr * 3;
      const grd = ctx.createRadialGradient(sx, sy, 0, sx, sy, gs); grd.addColorStop(0, gc + '99'); grd.addColorStop(.4, gc + '22'); grd.addColorStop(1, gc + '00'); ctx.fillStyle = grd; ctx.beginPath(); ctx.arc(sx, sy, gs, 0, Math.PI * 2); ctx.fill();
      if (o.isComet && dr > 2 && o.orbitR) { const dx = pos.x, dy = pos.y; const d = Math.sqrt(dx * dx + dy * dy) || 1; const tl = dr * 10; const tx = (dx / d) * tl, ty = (dy / d) * tl; const tg = ctx.createLinearGradient(sx, sy, sx + tx, sy + ty); tg.addColorStop(0, o.color + 'bb'); tg.addColorStop(1, o.color + '00'); ctx.strokeStyle = tg; ctx.lineWidth = dr * 1.5; ctx.beginPath(); ctx.moveTo(sx, sy); ctx.lineTo(sx + tx, sy + ty); ctx.stroke(); }
      if (o.type === 'Interstellar Comet' || o.type === 'Interstellar Object') ctx.globalAlpha = .7 + .3 * Math.sin(tick * .08);
      const bg = ctx.createRadialGradient(sx - dr * .3, sy - dr * .3, 0, sx, sy, dr); bg.addColorStop(0, '#ffffff'); bg.addColorStop(.35, o.color || '#00ffff'); bg.addColorStop(1, (o.color || '#00ffff') + '88'); ctx.fillStyle = bg; ctx.beginPath(); ctx.arc(sx, sy, dr, 0, Math.PI * 2); ctx.fill(); ctx.globalAlpha = 1;
      if (isSel || isHov) { ctx.strokeStyle = o.color || '#00ffff'; ctx.lineWidth = isSel ? 2 : 1; ctx.globalAlpha = isSel ? .9 : .4; ctx.beginPath(); ctx.arc(sx, sy, dr + 4, 0, Math.PI * 2); ctx.stroke(); ctx.globalAlpha = 1; }
      if (showLabels) { const ms = o.id === 'sun_ref' ? .05 : o.isFixed ? .3 : o.orbitR > 50 ? .8 : o.orbitR > 10 ? .4 : .15; if (scale > ms || isHov) { ctx.font = (isHov ? 'bold ' : '') + Math.min(11 + scale * .3, 14) + 'px Courier New'; ctx.fillStyle = isHov ? '#ffffff' : 'rgba(140,210,255,.75)'; ctx.textAlign = 'center'; ctx.shadowColor = o.color || '#00ffff'; ctx.shadowBlur = isHov ? 10 : 4; ctx.fillText(o.name, sx, sy + dr + 14); ctx.shadowBlur = 0; } }
    }

    function drawOortOrbits() { if (!showOrbits) return; OORT_OBJECTS.forEach(o => { if (!o.orbitR || o.isFixed || o.isRegion) return; const [cx2, cy2] = w2s(0, 0); const r = o.orbitR * scale; if (r < 1) return; ctx.strokeStyle = o.isComet ? 'rgba(0,200,150,.06)' : 'rgba(0,100,200,.06)'; ctx.lineWidth = 1; ctx.setLineDash(o.isComet ? [3, 8] : [2, 6]); ctx.beginPath(); ctx.arc(cx2, cy2, r, 0, Math.PI * 2); ctx.stroke(); ctx.setLineDash([]); }); if (scale > 0.5) { const [cx2, cy2] = w2s(0, 0); ctx.strokeStyle = 'rgba(0,80,180,.1)'; ctx.lineWidth = 1; ctx.setLineDash([4, 8]); ctx.beginPath(); ctx.arc(cx2, cy2, 2 * scale, 0, Math.PI * 2); ctx.stroke(); ctx.beginPath(); ctx.arc(cx2, cy2, 20 * scale, 0, Math.PI * 2); ctx.stroke(); ctx.strokeStyle = 'rgba(0,50,120,.08)'; ctx.beginPath(); ctx.arc(cx2, cy2, 200 * scale, 0, Math.PI * 2); ctx.stroke(); ctx.setLineDash([]); if (scale > 1) { ctx.font = '10px Courier New'; ctx.textAlign = 'center'; ctx.fillStyle = 'rgba(0,150,255,.3)'; ctx.fillText('HILLS CLOUD', cx2, cy2 - 2 * scale - 5); ctx.fillText('INNER OORT', cx2, cy2 - 20 * scale - 5); ctx.fillText('OUTER OORT BOUNDARY', cx2, cy2 - 200 * scale - 5); } } }

    function updateScaleBar() { const sb = document.getElementById('scale-bar'); if (appMode !== 'oort') { sb.innerHTML = ''; return; } const barAU = scale > 2 ? 1000 : scale > .5 ? 10000 : scale > .1 ? 50000 : 100000; const barPx = barAU / 1000 * scale; sb.innerHTML = barPx > 20 && barPx < 300 ? '<span>━ ' + barAU.toLocaleString() + ' AU ━</span>' : ''; }

    function drawOort() { const bg = ctx.createRadialGradient(W / 2, H / 2, 0, W / 2, H / 2, Math.max(W, H) * .75); bg.addColorStop(0, '#000014'); bg.addColorStop(1, '#000004'); ctx.fillStyle = bg; ctx.fillRect(0, 0, W, H); STARS.forEach(s => { const tw = .5 + .5 * Math.sin(tick * .014 + s.t); ctx.globalAlpha = .18 + .6 * s.b * tw; ctx.fillStyle = s.c; ctx.beginPath(); ctx.arc(s.x * W, s.y * H, s.r, 0, Math.PI * 2); ctx.fill(); }); ctx.globalAlpha = 1; if (oort3D) { ctx.fillStyle = 'rgba(0,200,255,.15)'; ctx.font = '11px Courier New'; ctx.textAlign = 'left'; ctx.fillText('❄ 3D VIEW — 45° TILT', 20, H - 52); } drawOortDensityCloud(); drawOortOrbits(); drawOortParticles(); OORT_OBJECTS.forEach(drawOortObject); updateScaleBar(); }
    function hitTestOort(mx, my) { for (let i = OORT_OBJECTS.length - 1; i >= 0; i--) { const o = OORT_OBJECTS[i]; if (o.isRegion) continue; const pos = getOortPos(o); const [sx, sy] = w2s(pos.x, pos.y); const r = Math.max(o.radius * Math.min(scale * 2, 4), o.id === 'sun_ref' ? 8 : 3) + 14; if ((mx - sx) ** 2 + (my - sy) ** 2 < r * r) return o; } return null; }

    const OORT_VIEWS = { full: { cx: 0, cy: 0, s: .012 }, inner: { cx: 0, cy: 0, s: .08 }, outer: { cx: 0, cy: 0, s: .006 }, hills: { cx: 0, cy: 0, s: .15 }, solar: { cx: 0, cy: 0, s: 2.5 }, comets: { cx: 0, cy: 0, s: .025 } };
    function setOortView(v) { const t = OORT_VIEWS[v]; tCamX = t.cx; tCamY = t.cy; tScale = t.s; document.querySelectorAll('#oort-pills .mpill').forEach(p => p.classList.remove('active')); const el = document.getElementById('o-' + v); if (el) el.classList.add('active'); closePanel(); }
    function toggleOortDensity() { oortDensity = !oortDensity; document.getElementById('btn-oort-density').classList.toggle('on', oortDensity); }
    function toggleOort3D() { oort3D = !oort3D; document.getElementById('btn-oort-3d').classList.toggle('on', oort3D); }
    function showOortInfo() { document.getElementById('oort-info-rows').innerHTML = [['What is it?', 'Spherical shell of icy bodies surrounding Solar System'], ['Inner Edge', '~2,000 AU from Sun'], ['Outer Edge', '~200,000 AU (~3.2 light-years)'], ['Objects', 'Up to 2 trillion icy bodies'], ['Composition', 'Water ice, ammonia, methane'], ['Proposed by', 'Jan Oort (1950)'], ['Hills Cloud', 'Inner region 2,000–20,000 AU'], ['Nearest Star', 'Proxima Centauri at 268,770 AU — inside outer Oort!']].map(([k, v]) => '<div class="dr"><span class="dl">' + k.toUpperCase() + '</span><span class="dv">' + v + '</span></div>').join(''); document.getElementById('oort-info-panel').classList.add('on'); }

    function draw() { if (!paused) tick++; smoothCam(); ctx.clearRect(0, 0, W, H); if (appMode === 'solar') drawSolar(); else drawOort(); document.getElementById('hud-r').innerHTML = 'ZOOM: ' + scale.toFixed(appMode === 'oort' ? 4 : 2) + 'x' + (appMode === 'oort' ? '  ❄' : '') + (paused ? '  ⏸' : '') + '\nDRAG · PINCH · TAP'; requestAnimationFrame(draw); }
    function hitTestCurrent(mx, my) { return appMode === 'solar' ? hitTestSolar(mx, my) : hitTestOort(mx, my); }

    const tip = document.getElementById('tip');
    canvas.addEventListener('mousemove', e => { mouseScreenX = e.clientX; mouseScreenY = e.clientY; const obj = hitTestCurrent(e.clientX, e.clientY); hovObj = obj; canvas.style.cursor = obj ? 'pointer' : isDrag ? 'grabbing' : 'crosshair'; if (obj && !isDrag) { tip.style.display = 'block'; tip.style.left = (e.clientX + 14) + 'px'; tip.style.top = (e.clientY - 10) + 'px'; tip.textContent = obj.emoji + '  ' + obj.name.toUpperCase(); } else tip.style.display = 'none'; if (isDrag) { tCamX = cx0 - (e.clientX - dx0) / scale; tCamY = cy0 - (e.clientY - dy0) / scale; camX = tCamX; camY = tCamY; } });
    canvas.addEventListener('mousedown', e => { if (e.button !== 0) return; isDrag = true; dx0 = e.clientX; dy0 = e.clientY; cx0 = camX; cy0 = camY; });
    canvas.addEventListener('mouseup', e => { if (e.button !== 0) return; const moved = Math.abs(e.clientX - dx0) > 5 || Math.abs(e.clientY - dy0) > 5; isDrag = false; if (!moved) { const obj = hitTestCurrent(e.clientX, e.clientY); if (gravLaunching && appMode === 'solar') { const [wx, wy] = s2w(e.clientX, e.clientY); spawnProjectile(wx, wy); } else if (collisionMode && appMode === 'solar') { handleCollisionClick(obj); } else if (compareMode && appMode === 'solar') { handleCompareClick(obj); } else { if (obj) openPanel(obj); else closePanel(); } } });
    canvas.addEventListener('contextmenu', e => { e.preventDefault(); if (gravLaunching) { gravLaunching = false; document.getElementById('gv-launch-btn').classList.remove('active'); document.getElementById('gv-launch-btn').textContent = '🚀 TAP SPACE TO LAUNCH'; document.getElementById('grav-hint').style.display = 'none'; } });
    canvas.addEventListener('wheel', e => { e.preventDefault(); const minZ = appMode === 'oort' ? .003 : .08; const maxZ = appMode === 'oort' ? 20 : 25; const f = e.deltaY < 0 ? 1.12 : .89, prev = scale; tScale = Math.max(minZ, Math.min(maxZ, scale * f)); const wx = camX + (e.clientX - W / 2) / prev, wy = camY + (e.clientY - H / 2) / prev; tCamX = wx - (e.clientX - W / 2) / tScale; tCamY = wy - (e.clientY - H / 2) / tScale; }, { passive: false });
    let lTD = null;
    canvas.addEventListener('touchstart', e => { e.preventDefault(); if (e.touches.length === 1) { isDrag = true; dx0 = e.touches[0].clientX; dy0 = e.touches[0].clientY; cx0 = camX; cy0 = camY; } }, { passive: false });
    canvas.addEventListener('touchmove', e => { e.preventDefault(); if (e.touches.length === 2) { const d = Math.hypot(e.touches[0].clientX - e.touches[1].clientX, e.touches[0].clientY - e.touches[1].clientY); if (lTD) tScale = Math.max(.003, Math.min(25, scale * (d / lTD))); lTD = d; } else if (e.touches.length === 1 && isDrag) { tCamX = cx0 - (e.touches[0].clientX - dx0) / scale; tCamY = cy0 - (e.touches[0].clientY - dy0) / scale; camX = tCamX; camY = tCamY; } }, { passive: false });
    canvas.addEventListener('touchend', e => { e.preventDefault(); lTD = null; isDrag = false; if (e.changedTouches.length === 1) { const mx = e.changedTouches[0].clientX, my = e.changedTouches[0].clientY; if (Math.abs(mx - dx0) < 12 && Math.abs(my - dy0) < 12) { if (gravLaunching && appMode === 'solar') { const [wx, wy] = s2w(mx, my); spawnProjectile(wx, wy); } else if (collisionMode && appMode === 'solar') { const o = hitTestCurrent(mx, my); handleCollisionClick(o); } else if (compareMode && appMode === 'solar') { const o = hitTestCurrent(mx, my); handleCompareClick(o); } else { const o = hitTestCurrent(mx, my); if (o) openPanel(o); else closePanel(); } } } }, { passive: false });

    document.getElementById('speed-slider').addEventListener('input', function () { const v = this.value; speed = v == 0 ? 0 : Math.pow(v / 30, 2.5) * .3; document.getElementById('speed-val').textContent = speed < .01 ? '0×' : speed.toFixed(1) + '×'; });
    function togglePause() { paused = !paused; document.getElementById('pause-btn').textContent = paused ? '▶' : '⏸'; }

    function openPanel(obj) { selObj = obj; document.getElementById('p-name').textContent = obj.name; const pt = document.getElementById('p-type'); pt.textContent = obj.type.toUpperCase(); const c = typeColor(obj.type); pt.style.color = c; pt.style.borderColor = c; const pv = document.getElementById('p-vis'); pv.textContent = obj.emoji; pv.style.boxShadow = '0 0 24px ' + (obj.glowColor || obj.color || '#00ffff') + '66'; pv.style.background = 'radial-gradient(circle,' + (obj.color || '#00ffff') + '22,transparent)'; document.getElementById('p-rows').innerHTML = Object.entries(obj.details || {}).map(([k, v]) => '<div class="dr"><span class="dl">' + k.toUpperCase() + '</span><span class="dv">' + v + '</span></div>').join(''); document.getElementById('panel').classList.add('on'); }
    function closePanel() { selObj = null; document.getElementById('panel').classList.remove('on'); }

    const TBMAP = { 'age-panel': 'btn-age', 'vis-panel': 'btn-vis', 'grav-panel': 'btn-grav', 'mission-panel': 'btn-mission', 'collision-panel': 'btn-collision', 'alien-panel': 'btn-alien', 'timemachine-panel': 'btn-tm', 'compare-tool-panel': 'btn-cmp', 'resonance-panel': 'btn-res', 'oort-info-panel': 'btn-oort-info' };
    function toggleTool(id) { const el = document.getElementById(id); const wasOn = el.classList.contains('on'); document.querySelectorAll('.tool-panel').forEach(p => p.classList.remove('on')); Object.keys(TBMAP).forEach(k => { if (TBMAP[k]) { const btn = document.getElementById(TBMAP[k]); if (btn) btn.classList.remove('on'); } }); collisionMode = false; compareMode = false; if (!wasOn) { el.classList.add('on'); if (TBMAP[id]) { const btn = document.getElementById(TBMAP[id]); if (btn) btn.classList.add('on'); } if (id === 'vis-panel') buildVis(); if (id === 'collision-panel') collisionMode = true; if (id === 'compare-tool-panel') compareMode = true; if (id === 'resonance-panel') buildResonancePanel(); } }
    function closeTool(id) { document.getElementById(id).classList.remove('on'); if (TBMAP[id]) { const btn = document.getElementById(TBMAP[id]); if (btn) btn.classList.remove('on'); } collisionMode = false; compareMode = false; }
    function toggleScaleMode() { scaleMode = !scaleMode; document.getElementById('btn-scale').classList.toggle('on', scaleMode); }
    function toggleSolarWind() { showSolarWind = !showSolarWind; document.getElementById('btn-solar-wind').classList.toggle('on', showSolarWind); }
    function toggleOrbits() { showOrbits = !showOrbits; document.getElementById('btn-orb').classList.toggle('on', showOrbits); }
    function toggleLabels() { showLabels = !showLabels; document.getElementById('btn-lbl').classList.toggle('on', showLabels); }
    function showKbd() { document.getElementById('kbd-overlay').classList.add('on'); }
    function hideKbd() { document.getElementById('kbd-overlay').classList.remove('on'); }

    const PY = { Mercury: { p: 0.2408, e: '⚫' }, Venus: { p: 0.6152, e: '🟡' }, Earth: { p: 1, e: '🌍' }, Mars: { p: 1.8808, e: '🔴' }, Jupiter: { p: 11.862, e: '🟠' }, Saturn: { p: 29.457, e: '🪐' }, Uranus: { p: 84.011, e: '🔵' }, Neptune: { p: 164.8, e: '🔵' }, Pluto: { p: 248.09, e: '🟤' } };
    function calcAge() { const v = document.getElementById('age-input').value; if (!v) { document.getElementById('age-result').innerHTML = '<div style="color:#ff6666;font-size:13px">Please enter your birthday.</div>'; return; } const born = new Date(v), now = new Date(); const yrs = (now - born) / (365.25 * 24 * 3600 * 1000); if (yrs < 0) { document.getElementById('age-result').innerHTML = '<div style="color:#ff6666;font-size:13px">Birthday cannot be in the future!</div>'; return; } let html = '<div style="font-size:13px;color:rgba(0,200,255,.5);margin-bottom:10px">EARTH AGE: <span style="color:#fff;font-weight:bold">' + yrs.toFixed(2) + ' years</span></div>'; Object.entries(PY).forEach(([name, p]) => { const age = yrs / p.p, y = Math.floor(age), m = Math.floor((age - y) * 12); html += '<div class="planet-row"><span style="font-size:16px;width:22px">' + p.e + '</span><span style="color:rgba(0,200,255,.55);width:72px;font-size:12px">' + name + '</span><span style="color:#fff;font-size:12px;font-weight:bold">' + (age < 1 ? Math.round(age * 365) + ' days' : y + 'y ' + m + 'm') + '</span></div>'; }); const nb = new Date(born); nb.setFullYear(now.getFullYear()); if (nb < now) nb.setFullYear(now.getFullYear() + 1); const dl = Math.ceil((nb - now) / (24 * 3600 * 1000)); html += '<div style="margin-top:12px;font-size:12px;color:rgba(0,200,255,.5);padding-top:10px;border-top:1px solid rgba(0,255,255,.08)">🎂 Next birthday in <span style="color:#ffcc44">' + dl + ' days</span></div>'; document.getElementById('age-result').innerHTML = html; }

    function buildVis() { const now = new Date(), dy = Math.floor((now - (new Date(now.getFullYear(), 0, 0))) / (1000 * 60 * 60 * 24)); const PL = [{ n: 'Mercury', e: '⚫', c: '#aaaaaa', pr: 87.97, desc: 'Near dawn/dusk', t: 35 }, { n: 'Venus', e: '🟡', c: '#ffcc77', pr: 224.7, desc: 'Morning/evening star', t: 70 }, { n: 'Mars', e: '🔴', c: '#dd5533', pr: 686.97, desc: 'Visible near opposition', t: 80 }, { n: 'Jupiter', e: '🟠', c: '#cc8844', pr: 4332, desc: 'Very bright most nights', t: 85 }, { n: 'Saturn', e: '🪐', c: '#ddcc88', pr: 10759, desc: 'Rings with binoculars', t: 78 }, { n: 'Uranus', e: '🔵', c: '#88ddee', pr: 30589, desc: 'Barely naked-eye', t: 40 }, { n: 'Neptune', e: '🔵', c: '#4455ff', pr: 59800, desc: 'Binoculars required', t: 30 }]; const mn = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']; let html = '<div style="font-size:12px;color:rgba(0,200,255,.35);margin-bottom:12px">' + mn[now.getMonth()] + ' ' + now.getDate() + ', ' + now.getFullYear() + '</div>'; PL.forEach(p => { const phase = (dy / p.pr * Math.PI * 2) % (Math.PI * 2); const score = Math.floor(Math.abs(Math.sin(phase + p.pr * .01)) * 100); let st, cl; if (score > p.t * .8) { st = 'VISIBLE'; cl = 'v-yes'; } else if (score > p.t * .4) { st = 'POSSIBLE'; cl = 'v-may'; } else { st = 'NOT VISIBLE'; cl = 'v-no'; } html += '<div class="vis-item"><div class="vis-dot" style="background:' + p.c + '"></div><div class="vis-name">' + p.e + ' ' + p.n + '</div><div class="vis-info">' + p.desc + '</div><div class="vis-status ' + cl + '">' + st + '</div></div>'; }); document.getElementById('vis-result').innerHTML = html; }

    const si = document.getElementById('search-input'), sres = document.getElementById('search-results');
    si.addEventListener('input', () => { const q = si.value.toLowerCase().trim(); if (!q) { sres.style.display = 'none'; return; } const sm = BODIES.filter(b => b.name.toLowerCase().includes(q)).map(b => ({ ...b, _src: 'solar' })); const om = OORT_OBJECTS.filter(o => !o.isRegion && o.name.toLowerCase().includes(q)).map(o => ({ ...o, _src: 'oort' })); const m = [...sm, ...om].slice(0, 10); if (!m.length) { sres.style.display = 'none'; return; } sres.style.display = 'block'; sres.innerHTML = m.map(o => '<div class="sr-item" onclick="jumpToObj(\'' + o.id + '\',\'' + o._src + '\')">' + o.emoji + ' ' + o.name + '<span style="color:rgba(0,200,255,.4);font-size:10px;margin-left:6px">' + o.type + (o._src === 'oort' ? ' ❄' : '') + '</span></div>').join(''); });
    si.addEventListener('keydown', e => { if (e.key === 'Escape') { sres.style.display = 'none'; si.value = ''; } });
    document.addEventListener('click', e => { if (!e.target.closest('#search-bar') && !e.target.closest('#search-results')) sres.style.display = 'none'; });
    function jumpToObj(id, src) { sres.style.display = 'none'; si.value = ''; if (src === 'oort' && appMode !== 'oort') switchAppMode('oort'); if (src === 'solar' && appMode !== 'solar') switchAppMode('solar'); setTimeout(() => { if (src === 'solar') { const b = BM[id]; if (!b) return; const pos = getBodyPos(b); tCamX = pos.x; tCamY = pos.y; tScale = b.parent && b.parent !== 'sun' ? 8 : b.parent === 'sun' ? 2.5 : 1; setTimeout(() => openPanel(b), 400); } else { const o = OM[id]; if (!o) return; const pos = getOortPos(o); tCamX = pos.x; tCamY = pos.y; tScale = o.id === 'sun_ref' ? 2 : o.orbitR > 100 ? .02 : o.orbitR > 20 ? .06 : o.orbitR > .5 ? .3 : 1; setTimeout(() => openPanel(o), 400); } }, 200); }

    const KB_MAP = { '1': 'mercury', '2': 'venus', '3': 'earth', '4': 'mars', '5': 'jupiter', '6': 'saturn', '7': 'uranus', '8': 'neptune', '9': 'pluto', '0': 'sun' };
    document.addEventListener('keydown', e => { if (e.target.tagName === 'INPUT') return; const k = e.key.toLowerCase(); if (k === 'tab') { e.preventDefault(); const nextMode = appMode === 'solar' ? 'oort' : 'solar'; switchAppMode(nextMode); } else if (k === 'p') togglePause(); else if (k === 'r') resetCurrentView(); else if (k === 'g' && appMode === 'solar') toggleTool('grav-panel'); else if (k === 't' && appMode === 'solar') startTour(); else if (k === 'q' && appMode === 'solar') startQuiz(); else if (k === 'o') toggleOrbits(); else if (k === 'l') toggleLabels(); else if (k === 'w' && appMode === 'solar') toggleSolarWind(); else if (k === '+' || k === '=') zoomBy(1.3); else if (k === '-') zoomBy(.77); else if (k === 'escape') { closePanel(); document.querySelectorAll('.tool-panel').forEach(p => p.classList.remove('on')); Object.keys(TBMAP).forEach(id => { if (TBMAP[id]) { const btn = document.getElementById(TBMAP[id]); if (btn) btn.classList.remove('on'); } }); if (tourMode) stopTour(); hideKbd(); } else if (k === '?') showKbd(); else if (KB_MAP[e.key] && appMode === 'solar') { const b = BM[KB_MAP[e.key]]; if (b) { const p = getBodyPos(b); tCamX = p.x; tCamY = p.y; tScale = b.id === 'sun' ? 1.5 : 3; } } });

    // ══════════ LOADER ══════════
    const lc = document.getElementById('loader-canvas'); const lx = lc.getContext('2d'); let lt = 0;
    const lp = [{ r: 18, sp: .04, c: '#aaaaaa', s: 4 }, { r: 30, sp: .025, c: '#ffcc77', s: 6 }, { r: 44, sp: .018, c: '#4488ff', s: 7 }, { r: 58, sp: .012, c: '#dd5533', s: 5 }, { r: 74, sp: .006, c: '#cc8844', s: 12 }];
    function animLoader() { lt++; lx.clearRect(0, 0, 200, 200); for (let i = 0; i < 5; i++) { lx.strokeStyle = 'rgba(0,' + (60 + i * 20) + ',' + (140 + i * 15) + ',' + (0.04 + i * .02) + ')'; lx.lineWidth = 1; lx.setLineDash([2, 5]); lx.beginPath(); lx.arc(100, 100, 20 + i * 18, 0, Math.PI * 2); lx.stroke(); lx.setLineDash([]); } const sg = lx.createRadialGradient(100, 100, 0, 100, 100, 14); sg.addColorStop(0, '#fffde0'); sg.addColorStop(.5, '#ffaa00'); sg.addColorStop(1, '#ff5500'); lx.fillStyle = sg; lx.beginPath(); lx.arc(100, 100, 14, 0, Math.PI * 2); lx.fill(); lp.forEach(p => { const a = lt * p.sp, px = 100 + p.r * Math.cos(a), py = 100 + p.r * Math.sin(a); lx.strokeStyle = 'rgba(0,100,200,.08)'; lx.lineWidth = 1; lx.beginPath(); lx.arc(100, 100, p.r, 0, Math.PI * 2); lx.stroke(); const g = lx.createRadialGradient(px, py, 0, px, py, p.s); g.addColorStop(0, '#fff'); g.addColorStop(.5, p.c); g.addColorStop(1, p.c + '66'); lx.fillStyle = g; lx.beginPath(); lx.arc(px, py, p.s, 0, Math.PI * 2); lx.fill(); }); if (lt < 200) requestAnimationFrame(animLoader); }
    animLoader();
    const LSTEPS = ['LOADING SOLAR SYSTEM...', 'CALCULATING ORBITS...', 'MAPPING OORT CLOUD...', 'PLACING COMETS...', 'BUILDING PARTICLES...', 'READY ✓'];
    let lstep = 0;
    function stepLoader() { if (lstep < LSTEPS.length) { document.getElementById('loader-status').textContent = LSTEPS[lstep]; document.getElementById('loader-bar').style.width = ((lstep + 1) / LSTEPS.length * 100) + '%'; lstep++; setTimeout(stepLoader, 340); } else { setTimeout(() => { document.getElementById('loader').classList.add('fade'); setTimeout(() => { document.getElementById('loader').style.display = 'none'; draw(); }, 1000); }, 300); } }
    setTimeout(stepLoader, 300);
  </script>
</body>

</html>

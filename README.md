<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Minecraft Bedrock - İnek Can Sistemi</title>
    <style>
        * { box-sizing: border-box; }
        body {
            margin: 0; padding: 0;
            width: 100vw; height: 100vh;
            font-family: 'Courier New', Courier, monospace;
            color: #ffffff; overflow: hidden;
            user-select: none; touch-action: none;
            background-color: #78a7ff;
        }

        #game-viewport {
            position: fixed; top: 0; left: 0;
            width: 100vw; height: 100vh;
            overflow: hidden; cursor: grab; z-index: 1;
            background: linear-gradient(to bottom, #78a7ff 0%, #a4c2f4 100%);
        }
        #game-viewport:active { cursor: grabbing; }

        /* Nişangah (Nokta) +50px aşağıda */
        #crosshair {
            position: absolute; top: calc(50% + 50px); left: 50%;
            width: 8px; height: 8px; background-color: rgba(255, 255, 255, 0.95);
            border: 1px solid black; border-radius: 50%;
            transform: translate(-50%, -50%); z-index: 15; pointer-events: none;
            box-shadow: 0 0 4px rgba(0,0,0,0.8);
        }

        #sky {
            position: absolute; top: 0; left: 0;
            width: 100vw; height: 100vh;
            background: linear-gradient(to bottom, #78a7ff 0%, #a4c2f4 100%);
            z-index: 2; pointer-events: none; transition: background 1s ease;
        }

        .sun {
            position: absolute; width: 60px; height: 60px;
            background-color: #ffee00; border-radius: 50%;
            z-index: 3; pointer-events: none; box-shadow: 0 0 25px #ffaa00;
        }

        .moon {
            position: absolute; width: 50px; height: 50px;
            background-color: #f0f0f0; border-radius: 50%;
            z-index: 3; pointer-events: none; box-shadow: 0 0 15px rgba(255,255,255,0.8);
        }

        .cloud {
            position: absolute; width: 120px; height: 40px;
            background: rgba(255, 255, 255, 0.85); border-radius: 20px;
            z-index: 3; pointer-events: none; filter: blur(2px);
        }

        #ground-strip {
            position: absolute; bottom: 0; left: 0; width: 100vw; height: 38vh;
            z-index: 3; pointer-events: none; transition: background 0.3s;
            border-top: 5px solid #3b5e1b;
            background: #5b8731;
        }

        #world {
            position: absolute; width: 100%; height: 100%;
            z-index: 4; pointer-events: none;
        }

        .tree-3d { position: absolute; }
        .block-log {
            position: absolute; width: 35px; height: 120px;
            background: linear-gradient(90deg, #6a4e27, #503920); border: 2px solid #332211;
        }
        .block-leaves {
            position: absolute; width: 90px; height: 90px;
            background: linear-gradient(90deg, #2d6b20, #1b4213); border: 2px solid #10290b; border-radius: 12px;
        }

        .cow-3d {
            position: absolute; width: 60px; height: 35px;
            background-color: #dcdcdc; border: 2px solid #111; border-radius: 8px;
            box-shadow: 3px 3px 0px rgba(0,0,0,0.3);
        }
        .cow-head-3d {
            position: absolute; top: -18px; left: -12px;
            width: 25px; height: 22px; background-color: #bcbcbc; border: 2px solid #111; border-radius: 4px;
        }
        .cow-tail-3d {
            position: absolute; top: 5px; right: -8px;
            width: 4px; height: 18px; background-color: #555555; border: 1px solid #111; border-radius: 2px;
        }
        .cow-hp-badge {
            position: absolute; top: -35px; left: 50%; transform: translateX(-50%);
            background: rgba(0,0,0,0.8); color: #ff5555; font-size: 10px; padding: 2px 4px;
            border-radius: 4px; font-weight: bold; white-space: nowrap; border: 1px solid #ff3333;
        }

        #hearts-container {
            position: fixed; top: 20px; right: 20px; z-index: 30;
            display: flex; gap: 2px; background: rgba(0,0,0,0.5); padding: 8px; border-radius: 6px; pointer-events: none;
        }
        .heart { width: 18px; height: 18px; position: relative; display: inline-block; }
        .heart::before, .heart::after {
            content: ""; position: absolute; top: 0; width: 10px; height: 16px;
            border-radius: 10px 10px 0 0; background: #ff2222;
        }
        .heart::before { left: 9px; transform: rotate(-45deg); transform-origin: 0 100%; }
        .heart::after { left: 0; transform: rotate(45deg); transform-origin: 100% 100%; }
        .heart.empty::before, .heart.empty::after { background: #444; }
        .heart.half::after { background: #444; }

        #info-panel {
            position: fixed; top: 20px; left: 20px; z-index: 30;
            background-color: rgba(0, 0, 0, 0.75); padding: 12px; border-radius: 8px;
            border: 2px solid #3c3c3c; line-height: 1.6; font-size: 13px; pointer-events: none;
        }
        #info-panel b { color: #55ffff; }

        #inventory-bar {
            position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%);
            z-index: 30; display: flex; background-color: rgba(0, 0, 0, 0.85);
            padding: 6px; border: 3px solid #3c3c3c; border-radius: 8px; gap: 4px;
        }
        .inv-slot {
            width: 45px; height: 45px; background-color: #8b8b8b;
            border: 3px solid #373737; border-radius: 4px;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            font-size: 8px; color: #fff; position: relative; font-weight: bold; text-align: center; cursor: pointer;
        }
        .inv-slot.active { border-color: #fff; box-shadow: 0 0 8px #ffcc00; }
        .inv-count { position: absolute; bottom: 2px; right: 4px; font-size: 10px; color: #fff; text-shadow: 1px 1px 0 #000; }

        #game-controls {
            position: fixed; bottom: 85px; left: 20px; z-index: 30;
            background-color: rgba(35, 35, 35, 0.85); border: 3px solid #3c3c3c;
            padding: 10px; border-radius: 8px; text-align: center;
        }
        .control-grid {
            display: grid; grid-template-columns: repeat(3, 45px); grid-template-rows: repeat(2, 40px);
            gap: 5px; justify-content: center;
        }
        .ctrl-btn {
            background-color: #444; color: white; border: 2px solid #222;
            font-size: 11px; cursor: pointer; border-radius: 5px; font-weight: bold;
        }
        .ctrl-btn:hover { background-color: #666; }
        .ctrl-btn:active { background-color: #888; transform: scale(0.95); }

        .btn-ileri { grid-column: 2; grid-row: 1; }
        .btn-sol { grid-column: 1; grid-row: 2; }
        .btn-geri { grid-column: 2; grid-row: 2; }
        .btn-sag { grid-column: 3; grid-row: 2; }

        #action-btn-panel {
            position: fixed; bottom: 85px; right: 20px; z-index: 30;
            display: flex; flex-direction: column; gap: 8px;
        }
        .action-btn {
            background-color: #8b5a2b; color: white; border: 2px solid #3c2415;
            padding: 10px 14px; font-size: 12px; font-weight: bold; cursor: pointer; border-radius: 8px;
            box-shadow: 2px 2px 0px rgba(0,0,0,0.5); text-align: center;
        }
        .action-btn:hover { background-color: #a06835; }
        .action-btn:active { transform: scale(0.95); }

        #win-screen {
            display: none; position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
            background: rgba(0, 0, 0, 0.85); z-index: 100;
            justify-content: center; align-items: center; flex-direction: column; text-align: center;
        }
        #win-screen h1 { color: #55ff55; font-size: 40px; text-shadow: 0 0 10px #00ff00; }
        .tip { font-size: 9px; color: #ffcc00; margin-top: 6px; pointer-events: none; }
    </style>
</head>
<body 
    onmousedown="baslangicVurmaBaslat(event)" onmouseup="vurmaDurdur()" onmouseleave="vurmaDurdur()"
    ontouchstart="baslangicVurmaBaslat(event)" ontouchend="vurmaDurdur()">

    <div id="game-viewport">
        <div id="sky"></div>
        <div id="sun" class="sun"></div>
        <div id="moon" class="moon"></div>
        <div id="clouds-container"></div>
        <div id="ground-strip"></div>
        <div id="crosshair"></div>
        <div id="world"></div>
    </div>

    <div id="hearts-container">
        <div class="heart" id="h-0"></div><div class="heart" id="h-1"></div><div class="heart" id="h-2"></div><div class="heart" id="h-3"></div><div class="heart" id="h-4"></div>
        <div class="heart" id="h-5"></div><div class="heart" id="h-6"></div><div class="heart" id="h-7"></div><div class="heart" id="h-8"></div><div class="heart" id="h-9"></div>
    </div>

    <div id="inventory-bar" onclick="event.stopPropagation()">
        <div class="inv-slot active" id="slot-0" onclick="aktifSlotSec(0)"><span id="name-0">İnek Eti</span><span class="inv-count" id="cnt-0">0</span></div>
        <div class="inv-slot" id="slot-1" onclick="aktifSlotSec(1)"><span id="name-1">Deri</span><span class="inv-count" id="cnt-1">0</span></div>
        <div class="inv-slot" id="slot-2" onclick="aktifSlotSec(2)"><span id="name-2">Odun</span><span class="inv-count" id="cnt-2">0</span></div>
        <div class="inv-slot" id="slot-3" onclick="aktifSlotSec(3)"></div>
        <div class="inv-slot" id="slot-4" onclick="aktifSlotSec(4)"></div>
    </div>

    <div id="win-screen">
        <h1>🎉 DÜNYAYI TEMİZLEDİN! 🎉</h1>
        <p>Bütün nesneleri yok ettin!</p>
    </div>

    <div id="info-panel">
        🌍 <b>Minecraft Bedrock</b><br>
        📍 Konum (X: <span id="pos-x">0</span>, Y: <span id="pos-y">0</span>, Z: <span id="pos-z">0</span>)<br>
        ⏱️ Durum: <span id="time-status">Gündüz ☀️</span>
    </div>

    <div id="game-controls" onclick="event.stopPropagation()">
        <div class="control-grid">
            <button class="ctrl-btn btn-ileri" onclick="hareketEt('ileri')">İleri</button>
            <button class="ctrl-btn btn-sol" onclick="hareketEt('sol')">Sol</button>
            <button class="ctrl-btn btn-geri" onclick="hareketEt('geri')">Geri</button>
            <button class="ctrl-btn btn-sag" onclick="hareketEt('sag')">Sağ</button>
        </div>
        <div class="tip">💡 Nişangah +50px aşağıda!</div>
    </div>

    <div id="action-btn-panel" onclick="event.stopPropagation()">
        <button class="action-btn" style="background-color: #c62828;" onclick="inegiKes()">🗡️ İnek Kes</button>
        <button class="action-btn" style="background-color: #5d4037;" onclick="odunKes()">🪵 Odun Kes</button>
        <button class="action-btn" style="background-color: #2e7d32;" onclick="etYee()">🍖 İnek Eti Ye</button>
    </div>

    <script>
        let hp = 20; 
        let beefCount = 0, leatherCount = 0, woodCount = 0;
        
        let posX = 0, posZ = 0, posY = 0; 
        let cameraAngle = 0;   

        let timeCycle = 0; 
        let isNight = false;
        let selectedSlotIndex = 0;

        let treeStates = {};
        let cowOffsets = {};
        let cowHealths = {};
        let destroyedItems = new Set();
        let activeElements = [];

        let isDragging = false;
        let startX = 0, startY = 0;
        let vurmaInterval = null;

        const viewport = document.getElementById("game-viewport");

        viewport.addEventListener("mousedown", (e) => { isDragging = true; startX = e.clientX; startY = e.clientY; });
        window.addEventListener("mousemove", (e) => {
            if (!isDragging) return;
            let deltaX = e.clientX - startX;
            startX = e.clientX; startY = e.clientY;
            cameraAngle += deltaX * 0.6;
            sahneyiGuncelle();
        });
        window.addEventListener("mouseup", () => { isDragging = false; });

        viewport.addEventListener("touchstart", (e) => { isDragging = true; startX = e.touches[0].clientX; startY = e.touches[0].clientY; });
        window.addEventListener("touchmove", (e) => {
            if (!isDragging) return;
            let deltaX = e.touches[0].clientX - startX;
            startX = e.touches[0].clientX; startY = e.touches[0].clientY;
            cameraAngle += deltaX * 0.6;
            sahneyiGuncelle();
        });
        window.addEventListener("touchend", () => { isDragging = false; });

        function pseudoRandom(seed) {
            let x = Math.sin(seed) * 10000;
            return x - Math.floor(x);
        }

        function aktifSlotSec(index) {
            selectedSlotIndex = index;
            for(let i=0; i<5; i++) {
                document.getElementById(`slot-${i}`).classList.remove('active');
            }
            document.getElementById(`slot-${index}`).classList.add('active');
        }

        function etYee() {
            if (beefCount > 0) {
                beefCount--;
                hp = Math.min(20, hp + 4);
                sahneyiGuncelle();
            } else {
                alert("Envanterinde hiç İnek Eti yok!");
            }
        }

        setInterval(() => {
            for (let id in cowOffsets) {
                cowOffsets[id].angle += 0.02;
                cowOffsets[id].xOff = Math.sin(cowOffsets[id].angle) * 35;
                cowOffsets[id].zOff = Math.cos(cowOffsets[id].angle) * 35;
            }
            sahneyiGuncelle();
        }, 400);

        setInterval(() => {
            timeCycle += 1;
            if (timeCycle >= 1200) timeCycle = 0;
            isNight = timeCycle >= 600;
            sahneyiGuncelle();
        }, 1000);

        function kalpleriGuncelle() {
            for (let i = 0; i < 10; i++) {
                let heartEl = document.getElementById(`h-${i}`);
                heartEl.className = "heart";
                let kalpCanSiniri = i * 2;
                if (hp <= kalpCanSiniri) heartEl.classList.add("empty");
                else if (hp === kalpCanSiniri + 1) heartEl.classList.add("half");
            }
        }

        function sahneyiGuncelle() {
            kalpleriGuncelle();
            let worldDiv = document.getElementById("world");
            worldDiv.innerHTML = "";
            activeElements = [];

            let rad = (cameraAngle * Math.PI) / 180;
            let cos = Math.cos(rad);
            let sin = Math.sin(rad);

            let skyEl = document.getElementById("sky");
            let sunEl = document.getElementById("sun");
            let moonEl = document.getElementById("moon");
            let cloudsContainer = document.getElementById("clouds-container");
            cloudsContainer.innerHTML = "";

            let sunAngle = (timeCycle / 1200) * 360;
            let sunRad = (sunAngle * Math.PI) / 180;
            let sunX = window.innerWidth / 2 + Math.cos(sunRad) * 450;
            let sunY = window.innerHeight / 2 - Math.sin(sunRad) * 250;
            
            let moonRad = ((sunAngle + 180) * Math.PI) / 180;
            let moonX = window.innerWidth / 2 + Math.cos(moonRad) * 450;
            let moonY = window.innerHeight / 2 - Math.sin(moonRad) * 250;

            sunEl.style.left = sunX + "px";
            sunEl.style.top = sunY + "px";
            moonEl.style.left = moonX + "px";
            moonEl.style.top = moonY + "px";

            if (!isNight) {
                for (let c = 0; c < 4; c++) {
                    let cloudX = ((timeCycle * 0.5 + c * 300) % (window.innerWidth + 300)) - 150;
                    let cloudEl = document.createElement("div");
                    cloudEl.className = "cloud";
                    cloudEl.style.left = cloudX + "px";
                    cloudEl.style.top = (80 + c * 40) + "px";
                    cloudsContainer.appendChild(cloudEl);
                }
            }

            let kalanSaniye = isNight ? (1200 - timeCycle) : (600 - timeCycle);
            let kalanDakika = Math.ceil(kalanSaniye / 60);
            document.getElementById("time-status").innerText = isNight ? `Gece 🌙 (${kalanDakika} dk)` : `Gündüz ☀️ (${kalanDakika} dk)`;

            if (isNight) {
                skyEl.style.background = "linear-gradient(to bottom, #020408 0%, #0d1629 100%)";
                sunEl.style.display = "none";
                moonEl.style.display = "block";
            } else {
                skyEl.style.background = "linear-gradient(to bottom, #78a7ff 0%, #a4c2f4 100%)";
                sunEl.style.display = "block";
                moonEl.style.display = "none";
            }

            let blokX = Math.round(posX / 40);
            let blokZ = Math.round(posZ / 40);

            let olusturulanToplamSayac = 0;
            let yokEdilenSayac = destroyedItems.size;

            let gorusMesafesi = 8; 
            let merkezChunkX = Math.floor(posX / 200);
            let merkezChunkZ = Math.floor(posZ / 200);

            for (let cx = merkezChunkX - gorusMesafesi; cx <= merkezChunkX + gorusMesafesi; cx++) {
                for (let cz = merkezChunkZ - gorusMesafesi; cz <= merkezChunkZ + gorusMesafesi; cz++) {
                    
                    let seedX = cx * 100 + 17;
                    let seedZ = cz * 100 + 31;
                    let itemX = (cx * 200) + (pseudoRandom(seedX) * 160 - 80);
                    let itemZ = (cz * 200) + (pseudoRandom(seedZ) * 160 - 80);
                    
                    let itemId = `item_${cx}_${cz}`;
                    let spawnChance = pseudoRandom(seedX * seedZ);

                    let spawnLimit = 0.22; 
                    if (spawnChance < spawnLimit) {
                        olusturulanToplamSayac++;
                        if (!destroyedItems.has(itemId)) {
                            let tipRnd = pseudoRandom(seedX + seedZ);
                            let type = 'tree';

                            if (!isNight) {
                                type = tipRnd > 0.40 ? 'tree' : 'cow'; 
                            } else {
                                if (tipRnd < 0.15) type = 'zombie';
                                else if (tipRnd < 0.30) type = 'skeleton';
                                else if (tipRnd < 0.45) type = 'spider';
                                else type = 'cow';
                            }
                            
                            let currentX = itemX;
                            let currentZ = itemZ;
                            if (type === 'cow') {
                                if (!cowOffsets[itemId]) {
                                    cowOffsets[itemId] = { xOff: 0, zOff: 0, angle: pseudoRandom(seedX) * 10 };
                                }
                                if (cowHealths[itemId] === undefined) {
                                    cowHealths[itemId] = 3; 
                                }
                                currentX += cowOffsets[itemId].xOff;
                                currentZ += cowOffsets[itemId].zOff;
                            }

                            let relX = currentX - posX;
                            let relZ = currentZ - posZ;

                            let rotX = relX * cos - relZ * sin;
                            let rotZ = relX * sin + relZ * cos;

                            let maxBlokMesafeSiniri = 65; 
                            let blokMesafe = rotZ / 40;

                            if (rotZ > 10 && blokMesafe <= maxBlokMesafeSiniri) {
                                let elem = document.createElement("div");
                                let ekranMerkezX = window.innerWidth / 2;
                                let ekranMerkezY = window.innerHeight * 0.62;
                                
                                let renderScale = Math.max(0.25, 1 - (blokMesafe / maxBlokMesafeSiniri)); 
                                let screenX = ekranMerkezX + rotX * (320 / rotZ);
                                let screenY = ekranMerkezY + (posY * 35);

                                elem.style.position = "absolute";
                                elem.style.left = screenX + "px";
                                elem.style.top = screenY + "px";

                                if (type === 'tree') {
                                    if (!treeStates[itemId]) treeStates[itemId] = { leaves: 3, logs: 1 };
                                    let state = treeStates[itemId];

                                    elem.className = "tree-3d";
                                    elem.style.transform = `translate(-50%, -100%) scale(${renderScale})`;
                                    
                                    let leavesHtml = state.leaves > 0 ? `<div class="block-leaves" style="bottom: 80px; left: -45px; opacity: ${state.leaves / 3};"></div>` : '';
                                    let logHtml = state.logs > 0 ? `<div class="block-log" style="bottom: 0; left: -17px;"></div>` : '';
                                    elem.innerHTML = `${leavesHtml}${logHtml}`;
                                    
                                    activeElements.push({ id: itemId, type: 'tree', x: screenX, y: screenY - (renderScale*50), radius: 65 * renderScale, distance: blokMesafe });
                                } 
                                else if (type === 'cow') {
                                    let cowHp = cowHealths[itemId];
                                    elem.className = "cow-3d";
                                    elem.style.transform = `translate(-50%, -100%) scale(${renderScale})`;
                                    elem.innerHTML = `<div class="cow-hp-badge">❤️ ${cowHp}/3</div><div class="cow-head-3d"></div><div class="cow-tail-3d"></div>`;
                                    
                                    let cowCenterY = screenY - (renderScale * 25);
                                    activeElements.push({ id: itemId, type: 'cow', x: screenX, y: cowCenterY, radius: 60 * renderScale, distance: blokMesafe });
                                }
                                else if (type === 'zombie') {
                                    elem.className = "zombie-3d";
                                    elem.style.transform = `translate(-50%, -100%) scale(${renderScale})`;
                                    activeElements.push({ id: itemId, type: 'zombie', x: screenX, y: screenY - (renderScale*40), radius: 50 * renderScale, distance: blokMesafe });
                                }
                                else if (type === 'skeleton') {
                                    elem.className = "skeleton-3d";
                                    elem.style.transform = `translate(-50%, -100%) scale(${renderScale})`;
                                    activeElements.push({ id: itemId, type: 'skeleton', x: screenX, y: screenY - (renderScale*40), radius: 50 * renderScale, distance: blokMesafe });
                                }
                                else if (type === 'spider') {
                                    elem.className = "spider-3d";
                                    elem.style.transform = `translate(-50%, -100%) scale(${renderScale})`;
                                    activeElements.push({ id: itemId, type: 'spider', x: screenX, y: screenY - (renderScale*15), radius: 60 * renderScale, distance: blokMesafe });
                                }

                                worldDiv.appendChild(elem);
                            }
                        }
                    }
                }
            }

            if (olusturulanToplamSayac > 0 && yokEdilenSayac >= olusturulanToplamSayac) {
                document.getElementById("win-screen").style.display = "flex";
            }

            document.getElementById("cnt-0").innerText = beefCount;
            document.getElementById("cnt-1").innerText = leatherCount;
            document.getElementById("cnt-2").innerText = woodCount;

            document.getElementById("pos-x").innerText = blokX;
            document.getElementById("pos-z").innerText = blokZ;
        }

        // İnek Kes tuşu (Uyarısız, 33 blok mesafe sınırı)
        function inegiKes() {
            let centerX = window.innerWidth / 2;
            let centerY = window.innerHeight / 2 + 50; 

            for (let el of activeElements) {
                if (el.type === 'cow') {
                    let mesafe = Math.hypot(el.x - centerX, el.y - centerY);
                    if (mesafe <= el.radius && el.distance <= 33.0) {
                        cowHealths[el.id]--; 
                        if (cowHealths[el.id] <= 0) {
                            destroyedItems.add(el.id);
                            beefCount++;      
                            leatherCount++;   
                        }
                        sahneyiGuncelle();
                        break;
                    }
                }
            }
        }

        // Odun Kes tuşu (Doğrudan odunu keser, 33 blok mesafe sınırı)
        function odunKes() {
            let centerX = window.innerWidth / 2;
            let centerY = window.innerHeight / 2 + 50;

            for (let el of activeElements) {
                if (el.type === 'tree') {
                    let mesafe = Math.hypot(el.x - centerX, el.y - centerY);
                    if (mesafe <= el.radius && el.distance <= 33.0) {
                        destroyedItems.add(el.id);
                        woodCount++;
                        sahneyiGuncelle();
                        break;
                    }
                }
            }
        }

        function hedefiVurVeKkir() {
            let centerX = window.innerWidth / 2;
            let centerY = window.innerHeight / 2 + 50;

            for (let el of activeElements) {
                let mesafe = Math.hypot(el.x - centerX, el.y - centerY);
                if (mesafe < el.radius && el.distance <= 65.0) {
                    if (el.type === 'tree') {
                        destroyedItems.add(el.id);
                        woodCount++;
                    } else if (el.type !== 'cow') {
                        destroyedItems.add(el.id);
                    }
                    sahneyiGuncelle();
                    break;
                }
            }
        }

        function baslangicVurmaBaslat(event) {
            if(event && (event.target.closest('#game-controls') || event.target.closest('#inventory-bar') || event.target.closest('#hearts-container') || event.target.closest('#action-btn-panel') || event.target.closest('#game-viewport'))) return;
            hedefiVurVeKkir();
            if (!vurmaInterval) vurmaInterval = setInterval(hedefiVurVeKkir, 250);
        }

        function vurmaDurdur() {
            if (vurmaInterval) { clearInterval(vurmaInterval); vurmaInterval = null; }
        }

        function hareketEt(yon) {
            let adimHizi = 30;
            let rad = (cameraAngle * Math.PI) / 180;
            if (yon === 'ileri') { posX += adimHizi * Math.sin(rad); posZ += adimHizi * Math.cos(rad); }
            if (yon === 'geri') { posX -= adimHizi * Math.sin(rad); posZ -= adimHizi * Math.cos(rad); }
            if (yon === 'sol') { posX -= adimHizi * Math.cos(rad); posZ += adimHizi * Math.sin(rad); }
            if (yon === 'sag') { posX += adimHizi * Math.cos(rad); posZ -= adimHizi * Math.sin(rad); }
            sahneyiGuncelle();
        }

        sahneyiGuncelle();
    </script>
</body>
</html>

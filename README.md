# map
진주제일여고
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>진주제일여고 무장애 통합지도</title>
    <style>
        :root {
            --bg-map: #e8eaed;
            --road: #ffffff;
            --visual: #4285f4;
            --wheel: #ea4335;
            --elder: #fbbc05;
        }

        body, html { margin: 0; padding: 0; width: 100%; height: 100%; overflow: hidden; font-family: 'Pretendard', sans-serif; }
        
        #header {
            position: absolute; top: 20px; left: 20px; z-index: 1000;
            background: white; padding: 20px; border-radius: 16px;
            box-shadow: 0 8px 30px rgba(0,0,0,0.12); width: 280px;
        }
        h1 { font-size: 18px; margin: 0 0 15px 0; color: #202124; }
        .filter-btn {
            display: flex; align-items: center; gap: 10px; margin-bottom: 8px;
            cursor: pointer; font-size: 14px; padding: 5px; border-radius: 8px; transition: 0.2s;
        }
        .filter-btn:hover { background: #f1f3f4; }
        
        #map-viewport {
            width: 100vw; height: 100vh; background: var(--bg-map);
            cursor: grab; overflow: hidden; position: relative;
        }
        #map-content {
            position: absolute; width: 4000px; height: 4000px;
            transform-origin: center; transition: transform 0.1s ease-out;
        }

        .landmark-school {
            position: absolute; top: 1900px; left: 1850px;
            width: 300px; height: 150px; background: #fff;
            border: 4px solid #4285f4; border-radius: 12px;
            display: flex; align-items: center; justify-content: center;
            font-weight: bold; font-size: 20px; color: #4285f4; box-shadow: 0 4px 10px rgba(0,0,0,0.1);
        }
        .road-main { position: absolute; background: var(--road); }

        .marker {
            position: absolute; width: 32px; height: 32px;
            border-radius: 50% 50% 50% 0; transform: rotate(-45deg);
            border: 2px solid white; cursor: pointer; z-index: 10;
            display: flex; align-items: center; justify-content: center;
            box-shadow: 0 4px 6px rgba(0,0,0,0.2);
        }
        .marker span { transform: rotate(45deg); font-size: 16px; }

        #detail-panel {
            position: absolute; bottom: -400px; left: 50%; transform: translateX(-50%);
            width: 90%; max-width: 600px; background: white;
            padding: 25px; border-radius: 24px 24px 0 0; z-index: 2000;
            box-shadow: 0 -10px 40px rgba(0,0,0,0.15); transition: bottom 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            max-height: 50vh; overflow-y: auto;
        }
        #detail-panel.active { bottom: 0; }
        .close-btn { float: right; border: none; background: #eee; border-radius: 50%; width: 30px; height: 30px; cursor: pointer; font-size: 18px; }

        .danger { background: #ea4335 !important; }
        .warning { background: #fbbc05 !important; }
        .safe { background: #34a853 !important; }
        
        .info-row { margin: 10px 0; padding: 10px; background: #f5f5f5; border-radius: 8px; }
        .info-label { font-weight: bold; color: #202124; }
        .info-value { color: #666; margin-top: 5px; }
    </style>
</head>
<body>

<div id="header">
    <h1>진주제일여고 Barrier-Free</h1>
    <div class="filter-btn"><input type="checkbox" id="check-visual" checked onchange="updateMarkers()"> 👁️ 시각장애</div>
    <div class="filter-btn"><input type="checkbox" id="check-wheel" checked onchange="updateMarkers()"> ♿ 지체장애</div>
    <div class="filter-btn"><input type="checkbox" id="check-elder" checked onchange="updateMarkers()"> 🧓 고령자</div>
    <div style="margin-top:10px; font-size:11px; color:#666; border-top:1px solid #eee; padding-top:10px;">
        * 마우스로 드래그하여 이동<br>
        * 휠로 확대/축소 가능
    </div>
</div>

<div id="map-viewport">
    <div id="map-content">
        <div class="road-main" style="top:2070px; left:0; width:4000px; height:80px;"></div>
        <div class="road-main" style="top:0; left:2150px; width:80px; height:4000px;"></div>
        <div class="landmark-school">진주제일여자고등학교</div>

        <!-- 시각장애 시설 -->
        <div class="marker visual" data-type="visual" style="top:2060px; left:2140px; background:var(--visual);" 
             onclick="showInfo('교차로 횡단보도', '음성 신호기 및 점자 블록 설치. 신호등 진동 안내기 포함. 버스 110번, 115번 정류장 근처', '시각장애')"><span>🔔</span></div>

        <div class="marker visual" data-type="visual" style="top:2150px; left:2250px; background:var(--visual);" 
             onclick="showInfo('점자 블록 경로', '학교 정문에서 상가 방향 완전한 점자 블록 설치. 길이 150m', '시각장애')"><span>📍</span></div>

        <div class="marker visual" data-type="visual" style="top:1950px; left:2050px; background:var(--visual);" 
             onclick="showInfo('버스 정류장 음성 안내', '버스 108번, 112번 정류장. 음성 신호기 설치됨', '시각장애')"><span>🚌</span></div>

        <!-- 지체장애 시설 -->
        <div class="marker wheel safe" data-type="wheel" style="top:2060px; left:1900px;" 
             onclick="showInfo('정문 경사로', '회전폭: 135cm (여유)\n경사도: 2.8도 (매우 완만)\n턱: 없음\n통행상태: 여유', '지체장애', 135, 2.8, 0)"><span>♿</span></div>

        <div class="marker wheel danger" data-type="wheel" style="top:2000px; left:2200px;" 
             onclick="showInfo('측면 보도구간', '폭: 58cm (위험)\n높이차: 5cm\n휠체어 회전 불가\n우회 권장', '지체장애', 58, 8, 5)"><span>⚠️</span></div>

        <div class="marker wheel warning" data-type="wheel" style="top:2100px; left:1950px;" 
             onclick="showInfo('편의점 입구 경사로', '폭: 90cm (통행가능)\n경사도: 6도\n턱: 2cm\n휠체어 진입 가능', '지체장애', 90, 6, 2)"><span>🛗</span></div>

        <div class="marker wheel" data-type="wheel" style="top:1950px; left:2300px; background:#9c27b0;" 
             onclick="showInfo('엘레베이터 위치', '학교 동관 1층\n폭: 110cm\n길이: 140cm\n높이차: 4m 대응', '지체장애')"><span>🛗</span></div>

        <div class="marker wheel" data-type="wheel" style="top:2150px; left:2050px; background:#ff9800;" 
             onclick="showInfo('계단 위치', '학교 주출입구\n높이: 120cm\n계단 수: 15개\n핸드레일: 양쪽 설치', '지체장애')"><span>🪜</span></div>

        <!-- 고령자 시설 -->
        <div class="marker elder" data-type="elder" style="top:1950px; left:2250px; background:var(--elder);" 
             onclick="showInfo('학교 숲 쉼터', '벤치 4개소\n완만한 산책로\n높이차 없음\n음지 제공', '고령자')"><span>🪑</span></div>

        <div class="marker elder" data-type="elder" style="top:2050px; left:2300px; background:var(--elder);" 
             onclick="showInfo('중앙 광장 휴식구역', '벤치 6개소\n평탄한 경로\n조명 설치\n음료수 자판기 근처', '고령자')"><span>☕</span></div>

        <div class="marker elder" data-type="elder" style="top:2200px; left:1900px; background:var(--elder);" 
             onclick="showInfo('완만한 산책로', '경사도 2도 이하\n길이 500m\n중간에 벤치 3개소\n보안 초소 근처', '고령자')"><span>🚶</span></div>
    </div>
</div>

<div id="detail-panel">
    <button class="close-btn" onclick="closeInfo()">✕</button>
    <h2 id="p-title" style="margin-top:0; color:#202124;">시설 정보를 선택하세요</h2>
    <div id="p-tag" style="margin-bottom:15px;"></div>
    <div id="p-desc" style="color:#555; line-height:1.8; white-space: pre-wrap;"></div>
</div>

<script>
    const content = document.getElementById('map-content');
    const viewport = document.getElementById('map-viewport');
    let isDragging = false;
    let startX, startY;
    let translateX = -1800, translateY = -1850;
    let scale = 1;

    content.style.transform = `translate(${translateX}px, ${translateY}px) scale(${scale})`;

    viewport.onmousedown = (e) => {
        isDragging = true;
        viewport.style.cursor = 'grabbing';
        startX = e.clientX - translateX;
        startY = e.clientY - translateY;
    };

    window.onmousemove = (e) => {
        if (!isDragging) return;
        translateX = e.clientX - startX;
        translateY = e.clientY - startY;
        content.style.transform = `translate(${translateX}px, ${translateY}px) scale(${scale})`;
    };

    window.onmouseup = () => {
        isDragging = false;
        viewport.style.cursor = 'grab';
    };

    viewport.onwheel = (e) => {
        e.preventDefault();
        const zoomSpeed = 0.1;
        if (e.deltaY < 0) scale = Math.min(scale + zoomSpeed, 2);
        else scale = Math.max(scale - zoomSpeed, 0.5);
        content.style.transform = `translate(${translateX}px, ${translateY}px) scale(${scale})`;
    };

    function showInfo(title, desc, type, width, angle, height) {
        const panel = document.getElementById('detail-panel');
        document.getElementById('p-title').innerText = title;
        document.getElementById('p-desc').innerText = desc;
        
        let tagHtml = `<span style="background:#eee; padding:6px 12px; border-radius:20px; font-size:12px; margin-right:8px; display:inline-block;">${type}</span>`;
        
        if(width) {
            let cls = width >= 120 ? 'safe' : (width >= 90 ? 'warning' : 'danger');
            let txt = width >= 120 ? '여유' : (width >= 90 ? '통행가능' : '위험');
            tagHtml += `<span class="${cls}" style="color:white; padding:6px 12px; border-radius:20px; font-size:12px; display:inline-block;">${width}cm (${txt})</span>`;
        }
        
        document.getElementById('p-tag').innerHTML = tagHtml;
        panel.classList.add('active');
    }

    function closeInfo() {
        document.getElementById('detail-panel').classList.remove('active');
    }

    function updateMarkers() {
        const v = document.getElementById('check-visual').checked;
        const w = document.getElementById('check-wheel').checked;
        const e = document.getElementById('check-elder').checked;

        document.querySelectorAll('.visual').forEach(m => m.style.display = v ? 'flex' : 'none');
        document.querySelectorAll('.wheel').forEach(m => m.style.display = w ? 'flex' : 'none');
        document.querySelectorAll('.elder').forEach(m => m.style.display = e ? 'flex' : 'none');
    }
</script>

</body>
</html>

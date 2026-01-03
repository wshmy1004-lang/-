<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>제20회 전국 교사대회 경품 추첨</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        :root {
            --primary: #ff4757;
            --gold: #ffa502;
            --dark: #2f3542;
            --sky: #87ceeb;
        }
        body { font-family: 'Pretendard', sans-serif; background: #2f3542; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; }
       
        .container {
            background: white; padding: 40px; border-radius: 30px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.3); width: 100%; max-width: 550px;
            text-align: center; position: relative;
        }

        .header-title {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
            margin-bottom: 20px;
        }

        #display-box {
            height: 180px; margin: 20px 0; border: 8px solid var(--sky);
            border-radius: 25px; display: flex; align-items: center; justify-content: center;
            background: #fff; position: relative; overflow: hidden;
        }

        #display-area {
            font-size: 2.2rem; font-weight: 900; color: var(--dark);
            transition: all 0.1s; white-space: nowrap;
        }

        .controls { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 20px; }
       
        select, .status-box {
            padding: 15px; border-radius: 12px; border: 1px solid #ddd;
            font-size: 16px; font-weight: bold; outline: none;
        }
        .status-box {
            display: flex; align-items: center; justify-content: center;
            background: #f1f2f6; border: 1px solid #ddd;
        }
       
        #draw-btn {
            background: var(--primary); color: white; border: none; font-weight: bold;
            cursor: pointer; grid-column: span 2; font-size: 1.5rem;
            box-shadow: 0 5px 0 #b33939; transition: 0.1s; padding: 15px; border-radius: 12px;
        }
        #draw-btn:active { transform: translateY(3px); box-shadow: 0 2px 0 #b33939; }
        #draw-btn:disabled { background: #ced4da; box-shadow: none; cursor: not-allowed; }

        /* 하단 문구 영역 */
        .event-info {
            background: #fff9e6; border: 1px solid #ffeaa7; padding: 15px;
            border-radius: 12px; margin-bottom: 20px; color: #d35400;
            font-weight: 800; line-height: 1.6; font-size: 18px; text-align: center;
        }

        .winner-pop { animation: pop 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275); color: #000000 !important; }
        .shake { animation: shake 0.1s infinite; }
        @keyframes shake {
            0% { transform: translate(2px, 2px); }
            50% { transform: translate(-2px, -2px); }
            100% { transform: translate(2px, -2px); }
        }
        @keyframes pop { 0% { transform: scale(0.5); opacity: 0; } 100% { transform: scale(1.1); opacity: 1; } }
       
        .winner-log { margin-top: 30px; text-align: left; max-height: 200px; overflow-y: auto; }
        .winner-item {
            background: #f8f9fa; margin-bottom: 8px; padding: 12px 20px;
            border-radius: 10px; display: flex; justify-content: space-between;
            border-left: 5px solid var(--gold);
        }
    </style>
</head>
<body>

<div class="container">
    <div class="header-title">
        <span>✨</span>
        <h2 style="margin:0; color: var(--dark);">제20회 전국 교사대회 경품 추첨</h2>
        <span>✨</span>
    </div>

    <div class="file-upload" style="margin-bottom: 20px;">
        <input type="file" id="excel-file" accept=".xlsx, .xls, .csv">
    </div>

    <div id="display-box">
        <div id="display-area">행운의 주인공은 누구?</div>
    </div>

    <div class="controls">
        <select id="region-select">
            <option value="all">🌏 전체 연회</option>
        </select>
        <div class="status-box">
            잔여: <span id="count" style="margin-left:8px; color:var(--primary);">0</span>명
        </div>
        <button id="draw-btn" onclick="startDraw()" disabled>추첨</button>
    </div>

    <div class="event-info">
        ⛪ 기독교대한감리회 교회학교전국연합회
    </div>

    <div class="winner-log" id="winners"></div>
</div>

<script>
    let participants = [];
    const displayArea = document.getElementById('display-area');
    const drawBtn = document.getElementById('draw-btn');
   
    document.getElementById('excel-file').addEventListener('change', function(e) {
        const reader = new FileReader();
        reader.onload = (e) => {
            const data = new Uint8Array(e.target.result);
            const workbook = XLSX.read(data, {type: 'array'});
            const json = XLSX.utils.sheet_to_json(workbook.Sheets[workbook.SheetNames[0]]);
            participants = json.map(row => ({
                name: row['성함'] || row['이름'] || '이름없음',
                region: row['지역'] || '기타'
            }));
            initRegions();
            updateDisplay();
            drawBtn.disabled = false;
            displayArea.innerText = "준비완료!";
        };
        reader.readAsArrayBuffer(e.target.files[0]);
    });

    function initRegions() {
        const regions = [...new Set(participants.map(p => p.region))];
        const select = document.getElementById('region-select');
        select.innerHTML = '<option value="all">🌏 전체 연회</option>';
        regions.forEach(r => {
            const opt = document.createElement('option');
            opt.value = r; opt.innerText = r;
            select.appendChild(opt);
        });
    }

    function updateDisplay() {
        const region = document.getElementById('region-select').value;
        const pool = participants.filter(p => region === 'all' || p.region === region);
        document.getElementById('count').innerText = pool.length;
    }

    document.getElementById('region-select').onchange = updateDisplay;

    function startDraw() {
        const region = document.getElementById('region-select').value;
        const pool = participants.filter(p => region === 'all' || p.region === region);
        if (pool.length === 0) return alert("추첨 대상자가 없습니다.");

        drawBtn.disabled = true;
        displayArea.classList.remove('winner-pop');
        displayArea.classList.add('shake');

        let duration = 2000;
        let startTime = Date.now();

        const rolling = setInterval(() => {
            const elapsed = Date.now() - startTime;
            const randomPerson = pool[Math.floor(Math.random() * pool.length)];
            displayArea.innerText = randomPerson.name;
            if (elapsed >= duration) {
                clearInterval(rolling);
                displayArea.classList.remove('shake');
                finalize(pool);
            }
        }, 60);
    }

    function finalize(pool) {
        const winner = pool[Math.floor(Math.random() * pool.length)];
        participants = participants.filter(p => p !== winner);
        displayArea.innerText = winner.name;
        displayArea.classList.add('winner-pop');
        confetti({ particleCount: 150, spread: 70, origin: { y: 0.6 } });
        const log = document.createElement('div');
        log.className = 'winner-item';
        log.innerHTML = `<span><strong>${winner.name}</strong></span> <span>${winner.region}</span>`;
        document.getElementById('winners').prepend(log);
        updateDisplay();
        drawBtn.disabled = false;
    }
</script>
</body>
</html>

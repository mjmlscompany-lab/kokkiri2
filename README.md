<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>🐘 콕끼리 전적판</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        :root { --primary: #3498db; --win: #2ecc71; --dark: #2c3e50; --bg: #f4f7f9; --result: #9b59b6; --excel: #27ae60; }
        body { font-family: 'Pretendard', sans-serif; background: var(--bg); margin: 0; padding: 10px; color: var(--dark); padding-bottom: 90px; }
        .round-tabs { display: flex; gap: 5px; overflow-x: auto; padding: 5px 0 10px; position: sticky; top: 0; background: var(--bg); z-index: 100; }
        .r-btn { padding: 12px 15px; border-radius: 12px; border: 1px solid #ddd; background: white; white-space: nowrap; font-weight: bold; flex: 1; min-width: 80px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
        .r-btn.active { background: var(--dark); color: white; border-color: var(--dark); }
        .r-btn.res-tab.active { background: var(--result); border-color: var(--result); }
        .page-content { display: none; }
        .page-content.active { display: block; }
        .court-grid { display: grid; grid-template-columns: 1fr; gap: 15px; }
        @media (min-width: 768px) { .court-grid { grid-template-columns: 1fr 1fr; } }
        .court-card { background: white; border-radius: 18px; padding: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.06); border: 1px solid #eee; }
        .court-title { font-size: 1rem; font-weight: 800; color: var(--primary); margin-bottom: 12px; border-bottom: 2px solid #f0f7ff; padding-bottom: 8px; display: flex; align-items: center; gap: 5px; }
        .team-container { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .team-area { padding: 10px; border-radius: 12px; background: #fafafa; border: 2px solid #f0f0f0; }
        .team-area.selected { background: #eafaf1; border-color: var(--win); }
        .player-input { width: 100%; padding: 10px 5px; border-radius: 8px; border: 1px solid #ddd; margin-bottom: 5px; font-size: 14px; box-sizing: border-box; }
        .record-btn { width: 100%; padding: 12px 0; border: 1px solid #ccc; border-radius: 10px; background: white; font-size: 0.85rem; font-weight: bold; color: #888; margin-top: 8px; cursor: pointer; }
        .record-btn.active { background: var(--win); color: white; border-color: var(--win); }
        .apply-all { width: calc(100% - 20px); max-width: 500px; padding: 20px; background: var(--dark); color: white; border: none; border-radius: 15px; font-size: 1.2rem; font-weight: bold; position: fixed; bottom: 10px; left: 50%; transform: translateX(-50%); z-index: 200; box-shadow: 0 5px 20px rgba(0,0,0,0.3); }
        .rank-table { width: 100%; border-collapse: collapse; font-size: 1.05rem; background: white; border-radius: 15px; overflow: hidden; }
        .rank-table th { padding: 15px; border-bottom: 2px solid #eee; background: #fafafa; }
        .rank-table td { padding: 15px 5px; text-align: center; border-bottom: 1px solid #f2f2f2; }
        .excel-btn { background: var(--excel); color: white; border: none; border-radius: 12px; padding: 15px; width: 100%; font-weight: bold; font-size: 1rem; margin-top: 15px; cursor: pointer; display: flex; align-items: center; justify-content: center; gap: 8px; }
    </style>
</head>
<body>

<div class="round-tabs" id="roundTabs"></div>
<div id="mainContainer"></div>

<div id="page-6" class="page-content">
    <div style="background: white; padding: 20px; border-radius: 25px; box-shadow: 0 4px 15px rgba(0,0,0,0.1);">
        <h2 style="color:var(--result); margin-top:0;">🏆 실시간 순위 현황 🐘</h2>
        <table class="rank-table" id="rankTable">
            <thead><tr><th>순위</th><th>이름</th><th>승</th><th>패</th></tr></thead>
            <tbody id="rankBody"></tbody>
        </table>
        <button class="excel-btn" onclick="downloadExcel()">📊 엑셀 파일로 저장하기</button>
        <button onclick="if(confirm('데이터를 초기화할까요?')) {localStorage.removeItem('kok-fixed-match'); location.reload();}" style="margin-top:25px; width:100%; padding:15px; border:none; border-radius:12px; color:#999; background:#eee; font-weight:bold; cursor:pointer;">🐘 데이터 초기화</button>
    </div>
</div>

<button class="apply-all" id="saveBtn" onclick="confirmAndSave()">전적 합산하기 🐘</button>

<script>
    const schedule = {
        1: [ // 1대진
            {t0:["시오","로토","현이","곽동칠"], t1:["구구","백구","대식","동구"]},
            {t0:["C코","구름","따순","우민"], t1:["아몬드","이름","제리","란지"]},
            {t0:["우지","뉴키","후니","리버"], t1:["덕자","단우","쿠쿠","무지"]},
            {t0:["슝슝이","죠스","솔찬","루피"], t1:["새로","완태","동이",""]},
            {t0:[], t1:[]}, {t0:[], t1:[]}
        ],
        2: [ // 2대진
            {t0:["시오","백구","제리","동구"], t1:["구구","구름","따순","우민"]},
            {t0:["C코","이름","현이","곽동칠"], t1:["아몬드","로토","대식","란지"]},
            {t0:["우지","단우","쿠쿠","무지"], t1:["덕자","뉴키","후니","리버"]},
            {t0:["슝슝이","완태","솔찬","루피"], t1:["새로","죠스","동이",""]},
            {t0:[], t1:[]}, {t0:[], t1:[]}
        ],
        3: [ // 3대진
            {t0:["시오","구름","대식","란지"], t1:["구구","이름","현이","곽동칠"]},
            {t0:["C코","로토","따순","우민"], t1:["아몬드","백구","제리","동구"]},
            {t0:["우지","뉴키","쿠쿠","무지"], t1:["덕자","단우","후니","리버"]},
            {t0:["슝슝이","죠스","완태","루피"], t1:["새로","솔찬","동이",""]},
            {t0:[], t1:[]}, {t0:[], t1:[]}
        ],
        4: [ // 4대진
            {t0:["시오","이름","따순","곽동칠"], t1:["구구","로토","제리","우민"]},
            {t0:["C코","백구","현이","동구"], t1:["아몬드","구름","대식","란지"]},
            {t0:["우지","단우","후니","리버"], t1:["덕자","뉴키","쿠쿠","무지"]},
            {t0:["슝슝이","솔찬","동이","루피"], t1:["새로","죠스","완태",""]},
            {t0:[], t1:[]}, {t0:[], t1:[]}
        ],
        5: [ // 5대진
            {t0:["시오","구름","현이","우민"], t1:["구구","백구","따순","곽동칠"]},
            {t0:["C코","로토","대식","동구"], t1:["아몬드","이름","제리","란지"]},
            {t0:["우지","뉴키","쿠쿠","리버"], t1:["덕자","단우","후니","무지"]},
            {t0:["슝슝이","죠스","동이","루피"], t1:["새로","솔찬","완태",""]},
            {t0:[], t1:[]}, {t0:[], t1:[]}
        ]
    };

    const initialPlayers = [
        "시오", "구구", "C코", "로토", "백구", "구름", "이름", "뉴키", "단우", 
        "슝슝이", "솔찬", "새로", "현이", "대식", "따순", "제리", "후니", "쿠쿠", 
        "곽동칠", "동구", "우민", "란지", "리버", "무지", "죠스", "루피", "완태", 
        "동이", "아몬드", "우지", "덕자"
    ].map(name => ({name, wins: 0, losses: 0}));

    let players = JSON.parse(localStorage.getItem('kok-fixed-match')) || initialPlayers;
    let winners = {}; 

    function init() {
        const rTabs = document.getElementById('roundTabs');
        const main = document.getElementById('mainContainer');

        for(let r=1; r<=6; r++) {
            const btn = document.createElement('button');
            btn.className = `r-btn ${r===1?'active':''} ${r===6?'res-tab':''}`;
            btn.id = `rbtn${r}`;
            btn.innerText = r === 6 ? '📊 결과' : `${r}대진`;
            btn.onclick = () => showRound(r);
            rTabs.appendChild(btn);

            if(r < 6) {
                const rPage = document.createElement('div');
                rPage.className = `page-content ${r===1?'active':''}`;
                rPage.id = `page-${r}`;
                let grid = `<div class="court-grid">`;
                for(let c=1; c<=6; c++) {
                    const match = schedule[r][c-1] || {t0:[], t1:[]};
                    if(match.t0.length === 0 && match.t1.length === 0) continue; 

                    grid += `
                        <div class="court-card">
                            <div class="court-title">${c}번 코트</div>
                            <div class="team-container">
                                <div class="team-area" id="area-${r}-${c}-t0">
                                    ${match.t0.map(p => `<div style="padding:5px; font-weight:bold; font-size:14px;">${p}</div>`).join('')}
                                    <button class="record-btn" id="btn-${r}-${c}-0" onclick="markWinner(${r},${c},0)">A팀 승</button>
                                </div>
                                <div class="team-area" id="area-${r}-${c}-t1">
                                    ${match.t1.map(p => `<div style="padding:5px; font-weight:bold; font-size:14px;">${p}</div>`).join('')}
                                    <button class="record-btn" id="btn-${r}-${c}-1" onclick="markWinner(${r},${c},1)">B팀 승</button>
                                </div>
                            </div>
                        </div>`;
                }
                grid += `</div>`;
                rPage.innerHTML = grid;
                main.appendChild(rPage);
            }
        }
        renderRank();
    }

    function showRound(r) {
        document.querySelectorAll('.r-btn').forEach(b => b.classList.remove('active'));
        document.querySelectorAll('.page-content').forEach(p => p.classList.remove('active'));
        document.getElementById(`rbtn${r}`).classList.add('active');
        document.getElementById(`page-${r}`).classList.add('active');
        document.getElementById('saveBtn').style.display = r === 6 ? 'none' : 'block';
        window.scrollTo(0,0);
    }

    function markWinner(r, c, team) {
        winners[`${r}-${c}`] = team;
        document.getElementById(`btn-${r}-${c}-0`).classList.toggle('active', team === 0);
        document.getElementById(`btn-${r}-${c}-1`).classList.toggle('active', team === 1);
        document.getElementById(`area-${r}-${c}-t0`).classList.toggle('selected', team === 0);
        document.getElementById(`area-${r}-${c}-t1`).classList.toggle('selected', team === 1);
    }

    function confirmAndSave() {
        if(Object.keys(winners).length === 0) return alert("결과가 없습니다.");
        if(!confirm("합산하시겠습니까?")) return;
        for(let key in winners) {
            const [r, c] = key.split('-');
            const winIdx = winners[key];
            const match = schedule[r][c-1];
            const winTeam = winIdx === 0 ? match.t0 : match.t1;
            const lossTeam = winIdx === 0 ? match.t1 : match.t0;
            winTeam.forEach(n => { const p = players.find(m => m.name === n); if(p) p.wins++; });
            lossTeam.forEach(n => { const p = players.find(m => m.name === n); if(p) p.losses++; });
        }
        localStorage.setItem('kok-fixed-match', JSON.stringify(players));
        alert("합산 완료!");
        showRound(6);
        renderRank();
    }

    function renderRank() {
        const sorted = [...players].sort((a, b) => b.wins - a.wins || a.losses - b.losses);
        document.getElementById('rankBody').innerHTML = sorted.map((p, i) => `
            <tr><td>${i+1}</td><td><strong>${p.name}</strong></td><td style="color:#2ecc71; font-weight:bold;">${p.wins}</td><td>${p.losses}</td></tr>
        `).join('');
    }

    function downloadExcel() {
        const table = document.getElementById("rankTable");
        const wb = XLSX.utils.table_to_book(table, {sheet: "전적결과"});
        const today = new Date().toISOString().slice(0, 10);
        XLSX.writeFile(wb, `콕끼리_결과_${today}.xlsx`);
    }

    init();
</script>
</body>
</html>

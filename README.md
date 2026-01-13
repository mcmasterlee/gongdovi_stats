<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gongdobi AI Analytics</title>
    
    <!-- Libraries -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="stylesheet" as="style" crossorigin href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.8/dist/web/static/pretendard.css" />
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">

    <style>
        :root {
            /* Palette */
            --bg-body: #F8FAFC;
            --bg-card: #FFFFFF;
            --primary: #1E293B;  /* Deep Navy */
            --secondary: #64748B;
            --gold: #D97706;
            
            --radius: 16px;
            --shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05), 0 2px 4px -1px rgba(0, 0, 0, 0.03);
            --border: 1px solid #E2E8F0;
        }

        /* [중요] 전체 가로 스크롤 방지 */
        body {
            font-family: 'Pretendard', sans-serif;
            background-color: var(--bg-body);
            color: var(--primary);
            margin: 0;
            padding: 30px;
            overflow-y: scroll; 
            overflow-x: hidden; 
        }

        .container { max-width: 1400px; margin: 0 auto; width: 100%; box-sizing: border-box; }

        /* --- Header --- */
        header {
            display: flex; justify-content: space-between; align-items: center;
            margin-bottom: 25px; 
            background: transparent; 
            padding: 0;
        }

        .brand-box {
            background-color: #000000;
            padding: 15px 30px;
            border-radius: 12px;
            display: flex;
            align-items: center;
            gap: 12px;
            box-shadow: var(--shadow);
        }

        .brand-box h1 { margin: 0; font-size: 20px; font-weight: 700; color: #FFFFFF; letter-spacing: -0.5px; }
        .ai-badge { background: #333; color: #00E096; font-size: 12px; padding: 4px 8px; border-radius: 4px; font-weight: 800; }

        .date-control {
            display: flex; align-items: center; gap: 15px;
            background: #FFFFFF; padding: 10px 25px; border-radius: 30px;
            border: var(--border);
            box-shadow: var(--shadow);
        }
        .nav-btn { background: none; border: none; cursor: pointer; color: var(--secondary); font-size: 16px; }
        .nav-btn:hover { color: var(--primary); transform: scale(1.1); transition: 0.2s; }
        .current-date { font-weight: 800; font-size: 18px; min-width: 80px; text-align: center; color: var(--primary); }

        /* --- Grid Layout --- */
        .grid-container {
            display: grid; grid-template-columns: repeat(4, 1fr); gap: 20px; margin-bottom: 20px;
            width: 100%; box-sizing: border-box;
        }
        .span-1 { grid-column: span 1; }
        .span-3 { grid-column: span 3; }

        .card {
            background: var(--bg-card); border-radius: var(--radius); padding: 20px;
            box-shadow: var(--shadow); border: var(--border);
            display: flex; flex-direction: column;
            position: relative;
            width: 100%; 
            box-sizing: border-box; /* 패딩 포함 너비 계산 */
            overflow: hidden; /* 카드 밖으로 내용 나가는 것 방지 */
        }

        /* --- KPI --- */
        .card-title { font-size: 14px; font-weight: 600; color: var(--secondary); margin-bottom: 8px; display: flex; align-items: center; gap: 6px;}
        .kpi-value { font-size: 26px; font-weight: 800; color: var(--primary); letter-spacing: -1px; margin-bottom: 4px; }
        .kpi-diff { font-size: 13px; font-weight: 600; padding: 3px 8px; border-radius: 4px; display: inline-block; }
        .up { background: #ECFDF5; color: #047857; }
        .down { background: #FEF2F2; color: #DC2626; }

        /* --- Chart & Table Switcher --- */
        .chart-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
        .chart-title { font-size: 16px; font-weight: 700; color: var(--primary); display: flex; align-items: center; gap: 8px; }
        
        .view-switch {
            background: #F1F5F9;
            padding: 4px;
            border-radius: 8px;
            display: flex;
            gap: 4px;
        }
        .switch-btn {
            border: none;
            background: transparent;
            padding: 6px 12px;
            border-radius: 6px;
            font-size: 12px;
            font-weight: 600;
            color: var(--secondary);
            cursor: pointer;
            transition: all 0.2s;
        }
        .switch-btn.active {
            background: #FFFFFF;
            color: var(--primary);
            box-shadow: 0 1px 2px rgba(0,0,0,0.1);
        }

        /* --- FIXED CONTAINER (Horizontal & Vertical Lock) --- */
        .fixed-height-box {
            height: 320px; 
            width: 100%; 
            position: relative;
            overflow: hidden; /* 영역 밖 내용 숨김 (중요) */
        }

        .chart-view {
            width: 100%;
            height: 100%;
            display: block; 
            position: relative;
        }
        
        .table-view {
            width: 100%;
            height: 100%;
            overflow-y: auto; /* 세로 스크롤 허용 */
            overflow-x: hidden; /* [핵심] 가로 스크롤 차단 */
            display: none; 
            border: 1px solid #F1F5F9;
            border-radius: 8px;
            box-sizing: border-box; 
        }
        .table-view.active { display: block; }

        .target-stack { display: flex; flex-direction: column; height: 100%; }

        /* [중요] 표 레이아웃 강력 고정 */
        .analysis-table { 
            width: 100% !important; 
            table-layout: fixed !important; /* 강제 고정 */
            border-collapse: collapse; 
            font-size: 13px; 
        }
        
        .analysis-table th { 
            position: sticky; top: 0; background: #F8FAFC; 
            padding: 10px 2px; /* 패딩 축소 */
            text-align: center; border-bottom: 1px solid #E2E8F0; color: var(--secondary); z-index: 5;
            white-space: nowrap; 
            overflow: hidden;
            text-overflow: ellipsis; 
        }
        
        .analysis-table td { 
            padding: 10px 2px; /* 패딩 축소 */
            text-align: center; border-bottom: 1px solid #F1F5F9; 
            white-space: nowrap; 
            overflow: hidden;
            text-overflow: ellipsis; 
        }
        .analysis-table tr.current-month-row { background-color: #ECFDF5; font-weight: 700; color: #047857; } 

        /* --- Bottom Row --- */
        .bottom-row { display: grid; grid-template-columns: repeat(2, 1fr); gap: 20px; width: 100%; box-sizing: border-box;}
        .bottom-card { 
            height: 320px; 
            width: 100%;
            box-sizing: border-box;
            overflow: hidden; 
            display: flex; flex-direction: column; 
        }

        .table-wrap { 
            overflow-y: auto; 
            overflow-x: hidden; 
            flex-grow: 1; 
            padding-right: 2px; 
            width: 100%;
        }
        .table-wrap::-webkit-scrollbar { width: 5px; }
        .table-wrap::-webkit-scrollbar-thumb { background: #E2E8F0; border-radius: 10px; }

        .event-table {
            width: 100%;
            table-layout: fixed;
            border-collapse: collapse;
            font-size: 13px;
        }
        .event-table th { text-align: left; padding: 8px 5px; color: var(--secondary); background: #fff; position: sticky; top: 0; border-bottom: 1px solid #E2E8F0; z-index: 10;}
        .event-table td { padding: 10px 5px; border-bottom: 1px solid #F1F5F9; color: var(--primary); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
        .rank-circle { width: 18px; height: 18px; background: var(--primary); color: #fff; border-radius: 50%; display: inline-flex; align-items: center; justify-content: center; font-size: 11px; margin-right: 8px; flex-shrink: 0; }

        /* AI Box */
        .ai-box { background: linear-gradient(135deg, #1E293B 0%, #0F172A 100%); color: #fff; border: 1px solid #334155; }
        .ai-title { color: #fff; border-bottom: 1px solid #334155; padding-bottom: 12px; margin-bottom: 12px; font-size: 15px; font-weight: 700; display: flex; align-items: center; gap: 8px;}
        .ai-scroll { overflow-y: auto; flex-grow: 1; padding-right: 5px; overflow-x: hidden;}
        .ai-scroll::-webkit-scrollbar { width: 5px; }
        .ai-scroll::-webkit-scrollbar-thumb { background: #475569; border-radius: 10px; }
        .ai-text { font-size: 13px; line-height: 1.5; color: #E2E8F0; margin-bottom: 15px; }
        .ai-highlight { color: #38BDF8; font-weight: 700; }
        .strategy-box { background: rgba(255,255,255,0.08); padding: 12px; border-radius: 8px; margin-bottom: 8px; border-left: 3px solid var(--gold); }
        .strategy-head { font-weight: 700; color: var(--gold); font-size: 12px; margin-bottom: 4px; display: block; }
        .strategy-body { font-size: 12px; color: #F1F5F9; line-height: 1.4; }

        @media (max-width: 1024px) {
            .grid-container, .bottom-row { grid-template-columns: 1fr; }
            .span-1, .span-3 { grid-column: span 1; }
            header { flex-direction: column; align-items: flex-start; gap: 15px; }
            .brand-box { width: 100%; justify-content: center; }
            .date-control { width: 100%; justify-content: space-between; }
        }
    </style>
</head>
<body>

<div class="container">
    
    <header>
        <div class="brand-box">
            <h1>공도비 운영 현황</h1>
            <span class="ai-badge">AI Report</span>
        </div>

        <div class="date-control">
            <button class="nav-btn" onclick="changeMonth(-1)"><i class="fa-solid fa-chevron-left"></i></button>
            <span id="currentDate" class="current-date">2025.12</span>
            <button class="nav-btn" onclick="changeMonth(1)"><i class="fa-solid fa-chevron-right"></i></button>
        </div>
    </header>

    <!-- KPI Row -->
    <div class="grid-container">
        <div class="card span-1">
            <div class="card-title">전체 회원 수</div>
            <div class="kpi-value" id="kpi-total">-</div>
            <div id="diff-total">-</div>
        </div>
        <div class="card span-1">
            <div class="card-title">월간 방문 횟수</div>
            <div class="kpi-value" id="kpi-visit">-</div>
            <div id="diff-visit">-</div>
        </div>
        <div class="card span-1">
            <div class="card-title">신규 가입자</div>
            <div class="kpi-value" id="kpi-new">-</div>
            <div id="diff-new">-</div>
        </div>
        <div class="card span-1" style="border: 1px solid var(--gold); background: #FFFBEB;">
            <div class="card-title" style="color:#92400E;"><i class="fa-solid fa-crown"></i> 현재 카페 등급</div>
            <div class="kpi-value" id="kpi-rank" style="color:#B45309;">-</div>
            <div style="font-size:13px; color:#92400E; font-weight:600;">활동점수 <span id="kpi-score">0</span>점</div>
        </div>
    </div>

    <!-- Charts Row -->
    <div class="grid-container">
        <!-- Main Chart -->
        <div class="card span-3">
            <div class="chart-header">
                <div class="chart-title"><i class="fa-solid fa-chart-line"></i> 월간 트래픽 추이 (YoY)</div>
                <!-- Main Toggle -->
                <div class="view-switch">
                    <button class="switch-btn active" id="btn-main-chart" onclick="toggleView('main', 'chart')">차트</button>
                    <button class="switch-btn" id="btn-main-table" onclick="toggleView('main', 'table')">월간 분석표</button>
                </div>
            </div>
            
            <!-- Fixed Height Container -->
            <div class="fixed-height-box">
                <!-- Chart View -->
                <div id="mainChartWrapper" class="chart-view">
                    <canvas id="mainChart"></canvas>
                </div>
                <!-- Table View -->
                <div id="mainTableWrapper" class="table-view">
                    <table class="analysis-table">
                        <colgroup>
                            <col style="width: 20%">
                            <col style="width: 20%">
                            <col style="width: 20%">
                            <col style="width: 20%">
                            <col style="width: 20%">
                        </colgroup>
                        <thead>
                            <tr>
                                <th>월</th>
                                <th>2024</th>
                                <th>2025</th>
                                <th>증감</th>
                                <th>증감률</th>
                            </tr>
                        </thead>
                        <tbody id="mainTableBody"></tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- Demographics (Target) -->
        <div class="card span-1">
            <div class="chart-header">
                <div class="chart-title"><i class="fa-solid fa-users"></i> 타겟 분석</div>
                <!-- Target Toggle -->
                <div class="view-switch">
                    <button class="switch-btn active" id="btn-target-chart" onclick="toggleView('target', 'chart')">차트</button>
                    <button class="switch-btn" id="btn-target-table" onclick="toggleView('target', 'table')">표</button>
                </div>
            </div>

            <!-- Fixed Height Container -->
            <div class="fixed-height-box">
                <!-- Chart View (Stacked) -->
                <div id="targetChartWrapper" class="chart-view target-stack">
                    <div style="height: 140px; position:relative; margin-bottom: 10px;">
                        <canvas id="visitorChart"></canvas>
                    </div>
                    <div style="flex-grow:1; position:relative;">
                        <canvas id="ageChart"></canvas>
                    </div>
                </div>
                <!-- Table View -->
                <div id="targetTableWrapper" class="table-view">
                    <table class="analysis-table">
                         <colgroup>
                            <col style="width: 25%">
                            <col style="width: 25%">
                            <col style="width: 25%">
                            <col style="width: 25%">
                        </colgroup>
                        <thead>
                            <tr>
                                <th>월</th>
                                <th>여성</th>
                                <th>30대</th>
                                <th>40대</th>
                            </tr>
                        </thead>
                        <tbody id="targetTableBody"></tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>

    <!-- Bottom Row (Compact) -->
    <div class="bottom-row">
        <!-- Event Table -->
        <div class="card bottom-card">
            <div class="chart-header">
                <div class="chart-title"><i class="fa-solid fa-list-check"></i> 진행 이벤트 성과</div>
            </div>
            <div class="table-wrap">
                <table class="event-table">
                    <colgroup>
                        <col style="width: 20%">
                        <col style="width: 50%">
                        <col style="width: 15%">
                        <col style="width: 15%">
                    </colgroup>
                    <thead>
                        <tr>
                            <th>날짜</th>
                            <th>이벤트명</th>
                            <th>조회</th>
                            <th>반응</th>
                        </tr>
                    </thead>
                    <tbody id="eventTable"></tbody>
                </table>
            </div>
        </div>

        <!-- AI Insight -->
        <div class="card bottom-card ai-box">
            <div class="ai-title"><i class="fa-solid fa-robot"></i> AI 운영 인사이트</div>
            <div class="ai-scroll">
                <div id="ai-analysis" class="ai-text"></div>
                <div style="margin-top: 15px;">
                    <div style="color: #94A3B8; font-size: 11px; margin-bottom: 8px; font-weight: 700; text-transform:uppercase;">Next Month Strategy (<span id="nextMonthLabel"></span>)</div>
                    <div id="ai-strategy"></div>
                </div>
            </div>
        </div>
    </div>
</div>

<script>
    // --- Data ---
    const statsDB = {
        '2024': {
            '01': { total:9068, new:406, visit:36873, mem:2489, non:7776, f:89.7, age30:4.1, age40:33.7 },
            '02': { total:9600, new:532, visit:33126, mem:2529, non:6789, f:90.5, age30:3.7, age40:35.0 },
            '03': { total:10092, new:492, visit:33905, mem:2684, non:6695, f:88.4, age30:4.9, age40:29.7 },
            '04': { total:10525, new:433, visit:37861, mem:2821, non:7819, f:86.9, age30:3.8, age40:31.5 },
            '05': { total:10813, new:288, visit:40666, mem:2618, non:11432, f:87.1, age30:3.8, age40:28.4 },
            '06': { total:11097, new:278, visit:40894, mem:2538, non:13788, f:86.7, age30:3.7, age40:28.3 },
            '07': { total:11563, new:466, visit:45315, mem:2958, non:9544, f:87.8, age30:3.5, age40:30.0 },
            '08': { total:11864, new:301, visit:37742, mem:2577, non:8954, f:87.3, age30:3.5, age40:28.7 },
            '09': { total:12088, new:224, visit:36572, mem:2626, non:8461, f:88.3, age30:3.5, age40:28.3 },
            '10': { total:12537, new:449, visit:40359, mem:2883, non:11165, f:87.9, age30:3.4, age40:27.6 },
            '11': { total:12971, new:434, visit:49004, mem:3110, non:15538, f:88.0, age30:3.3, age40:27.8 },
            '12': { total:13567, new:596, visit:65466, mem:3624, non:19258, f:89.5, age30:4.2, age40:26.3 }
        },
        '2025': {
            '01': { total:14121, new:554, visit:56610, mem:3501, non:20625, f:87.6, age30:4.1, age40:24.7 },
            '02': { total:14676, new:555, visit:50112, mem:3466, non:17792, f:87.6, age30:3.7, age40:25.0 },
            '03': { total:15285, new:609, visit:56907, mem:3996, non:22214, f:87.6, age30:3.3, age40:24.1 },
            '04': { total:15941, new:656, visit:55494, mem:4071, non:20946, f:87.5, age30:2.8, age40:22.1 },
            '05': { total:16364, new:423, visit:45501, mem:3391, non:16024, f:89.0, age30:2.8, age40:23.1 },
            '06': { total:17045, new:681, visit:55586, mem:3697, non:22546, f:87.8, age30:3.5, age40:22.8 },
            '07': { total:17685, new:640, visit:63665, mem:4115, non:26099, f:86.9, age30:3.3, age40:22.9 },
            '08': { total:18323, new:638, visit:52592, mem:3772, non:23335, f:86.1, age30:3.3, age40:22.1 },
            '09': { total:19274, new:951, visit:54603, mem:4293, non:25456, f:86.7, age30:4.0, age40:20.8 },
            '10': { total:19897, new:623, visit:57434, mem:3826, non:31433, f:85.3, age30:3.5, age40:19.5 },
            '11': { total:20765, new:868, visit:62379, mem:4562, non:28395, f:86.6, age30:3.5, age40:20.2 },
            '12': { total:22244, new:1479, visit:66721, mem:5616, non:28007, f:83.3, age30:3.6, age40:18.8 }
        }
    };
    const rankDB = { '01':'열매2','02':'열매1','03':'열매1','04':'열매1','05':'열매3','06':'열매5','07':'열매5','08':'열매5','09':'열매5','10':'열매5','11':'열매5','12':'열매5' };
    const scoreDB = { '01':'33,639','02':'30,352','03':'30,412','04':'29,471','05':'67,054','06':'98,294','07':'97,307','08':'87,130','09':'95,550','10':'92,313','11':'95,466','12':'105,705' };
    const eventDB = {
        '01': [{d:'1/07', t:'22년도 초등 1학기 교사용 교재 증정', v:664, c:35, l:30}, {d:'1/10', t:'고등 교사용 교재 증정', v:1453, c:122, l:35}, {d:'1/14', t:'22개정 고등 개념+유형 세미나 사전 설문', v:639, c:30, l:16}, {d:'1/20', t:'개념루트&유형만렙 교재 체험단', v:626, c:13, l:10}, {d:'1/24', t:'22년도 초중등 1학기 교재 증정 2탄', v:1241, c:106, l:42}],
        '02': [{d:'2/10', t:'2025 새학기 맞이 간식박스', v:1919, c:242, l:79}, {d:'2/17', t:'완자공부력 전과목 어휘 교사용 체험단', v:889, c:30, l:23}, {d:'2/18', t:'고등 수학 유튜브 시청 교재 받기', v:592, c:21, l:14}, {d:'2/21', t:'25년도 초중등 1학기 교재 증정 3탄', v:1776, c:176, l:59}, {d:'2/25', t:'귀염뽀짝 완공이 스티커 인증', v:1048, c:155, l:42}],
        '03': [{d:'3/14', t:'22회 테솜 수학학력평가 교사용 교재', v:802, c:187, l:33}, {d:'3/17', t:'22개정 초등 수학 학습 가이드북', v:828, c:29, l:34}, {d:'3/19', t:'25년도 고등 과학/사회 교사용 교재', v:632, c:82, l:22}, {d:'3/26', t:'중고등 완자 기출픽 체험단', v:779, c:62, l:26}, {d:'3/31', t:'개념+유형X별나 스터디 플래너', v:389, c:5, l:14}],
        '04': [{d:'4/07', t:'25년도 1학기 마지막 교재 증정', v:1387, c:135, l:42}, {d:'4/15', t:'선생님들의 어린이날 아이디어 공모전', v:765, c:75, l:33}, {d:'4/18', t:'예스24X비상교육 스승의 날 이벤트', v:1464, c:98, l:46}, {d:'4/28', t:'어린이날 선물 뽑기 이벤트', v:1385, c:51, l:28}],
        '05': [{d:'5/02', t:'스승의 날 기념 공도비 굿즈 증정', v:1375, c:63, l:37}, {d:'5/12', t:'25년도 초등 수학 2학기 교재 증정', v:1163, c:156, l:85}, {d:'5/19', t:'완자 스무살 생일 축하', v:951, c:156, l:39}, {d:'5/26', t:'초, 중, 고등 오투 표지 디자인 선호도', v:748, c:145, l:43}],
        '06': [{d:'6/05', t:'공도비 월간 미션제 이벤트', v:1812, c:136, l:75}, {d:'6/10', t:'25년도 중학 수학 2학기 교재 증정', v:1819, c:258, l:73}, {d:'6/13', t:'완자공부력 표지 디자인 선호도', v:996, c:148, l:45}, {d:'6/16', t:'여름방학 대비 개념+유형 지우개 박스', v:1405, c:237, l:57}],
        '07': [{d:'7/02', t:'25년도 초중등 사회/과학 2학기 교재', v:1262, c:134, l:42}, {d:'7/14', t:'25년도 초중등 전과목 2학기 교재 증정', v:1733, c:235, l:67}, {d:'7/16', t:'초등+고등 수학 수학의 신 교재 체험단', v:725, c:44, l:37}, {d:'7/28', t:'25년도 초중등 전과목 2학기 증정 2차', v:1404, c:162, l:63}],
        '08': [{d:'8/01', t:'공도비 굿즈 받기 자랑 이벤트', v:847, c:54, l:27}, {d:'8/05', t:'비상교육 꼼꼼체 레드닷 어워드 수상 기념', v:1373, c:135, l:40}, {d:'8/22', t:'25년도 초중등 전과목 2학기 교재 증정 3차', v:709, c:47, l:41}, {d:'8/27', t:'22개정 완자 기출픽 체험단 모집', v:1362, c:61, l:45}],
        '09': [{d:'9/03', t:'22개정 중학 수학만 중간고사 교재 증정', v:1107, c:117, l:45}, {d:'9/04', t:'개념루트 9월 모의고사 풀이 영상 댓글', v:1819, c:14, l:20}, {d:'9/23', t:'25년도 초중등 2학기 교사용 교재 증정', v:855, c:68, l:33}, {d:'9/29', t:'리더스뱅크 교사용 교재 체험단', v:454, c:4, l:11}],
        '10': [{d:'10/02', t:'2025 결실 공유 한가위 이벤트', v:1248, c:86, l:55}, {d:'10/02', t:'22개정 고등 교사용 교재 증정', v:1638, c:188, l:57}, {d:'10/22', t:'수업은 개념+유형 x 숙제는 개념+연산', v:1036, c:44, l:37}, {d:'10/23', t:'비상교육 베스트 교재 표지 디자인 추천', v:1143, c:105, l:49}, {d:'10/31', t:'26년 중학 1학기 교사용 교재 증정', v:1761, c:212, l:71}],
        '11': [{d:'11/01', t:'공도비 회원 20,000명 달성 기원 칭찬', v:1365, c:120, l:68}, {d:'11/06', t:'국어 선생님 의견 설문조사', v:552, c:45, l:35}, {d:'11/07', t:'비상교육 베스트 교재 표지 디자인 투표', v:1063, c:173, l:66}, {d:'11/11', t:'2026 비상교육 탁상용/벽걸이 달력 증정', v:1773, c:365, l:98}, {d:'11/14', t:'26년 초등/중학 1학기 교사용 교재 증정', v:2089, c:217, l:76}],
        '12': [{d:'12/01', t:'개념루트&유형만렙 고등 수학 체험단', v:1101, c:51, l:34}, {d:'12/04', t:'완자공부력 사회 교과서 디자인 조사', v:664, c:78, l:41}, {d:'12/09', t:'초중등 수능독해 교사용 교재 증정', v:767, c:62, l:34}, {d:'12/17', t:'2026 선생님 수업 지원! 수학교구 드림', v:1788, c:162, l:63}, {d:'12/18', t:'25년 효자 교재 소개 이벤트', v:1694, c:104, l:66}, {d:'12/19', t:'26년 초등/중학 1학기 교사용 교재 증정 2차', v:2286, c:212, l:73}]
    };

    // Global
    Chart.defaults.font.family = "'Pretendard', sans-serif";
    Chart.defaults.color = '#64748B';

    let mainChart, visitorChart, ageChart;
    let blinkInterval;
    let currentDate = new Date(2025, 11); // Dec 2025

    function init() { 
        renderReport(); 
        renderMainTable();
        renderTargetTable();
    }
    window.onload = init;

    function changeMonth(delta) {
        const next = new Date(currentDate);
        next.setMonth(next.getMonth() + delta);
        if(next.getFullYear() !== 2025) {
            alert("2025년 데이터만 조회 가능합니다.");
            return;
        }
        currentDate = next;
        renderReport();
        renderMainTable();
        renderTargetTable();
        
        // Auto Scroll for visible tables
        scrollToActiveRow('mainTableWrapper');
        scrollToActiveRow('targetTableWrapper');
    }

    function toggleView(section, mode) {
        const wrapperChart = document.getElementById(section + 'ChartWrapper');
        const wrapperTable = document.getElementById(section + 'TableWrapper');
        const btnChart = document.getElementById('btn-' + section + '-chart');
        const btnTable = document.getElementById('btn-' + section + '-table');

        if(mode === 'chart') {
            wrapperChart.style.display = (section === 'target') ? 'flex' : 'block';
            wrapperTable.classList.remove('active');
            btnChart.classList.add('active');
            btnTable.classList.remove('active');
        } else {
            wrapperChart.style.display = 'none';
            wrapperTable.classList.add('active');
            btnChart.classList.remove('active');
            btnTable.classList.add('active');
            
            // Auto scroll whenever toggled to table
            scrollToActiveRow(section + 'TableWrapper');
        }
    }

    // --- Auto Scroll Function ---
    function scrollToActiveRow(wrapperId) {
        setTimeout(() => {
            const container = document.getElementById(wrapperId);
            if(!container.classList.contains('active')) return;
            
            const activeRow = container.querySelector('.current-month-row');
            if (activeRow) {
                const containerHeight = container.clientHeight;
                const rowTop = activeRow.offsetTop;
                const rowHeight = activeRow.clientHeight;
                container.scrollTop = rowTop - (containerHeight / 2) + (rowHeight / 2);
            }
        }, 50); 
    }

    function renderMainTable() {
        const tbody = document.getElementById('mainTableBody');
        const currentMonthIdx = currentDate.getMonth(); 
        let html = '';

        for (let i = 0; i < 12; i++) {
            const mStr = String(i + 1).padStart(2, '0');
            const d24 = statsDB['2024'][mStr].visit;
            const d25 = statsDB['2025'][mStr].visit;
            const diff = d25 - d24;
            const pct = ((diff/d24)*100).toFixed(1);
            
            const colorClass = diff > 0 ? '#047857' : (diff < 0 ? '#DC2626' : '#64748B');
            const bgClass = (i === currentMonthIdx) ? 'current-month-row' : '';

            html += `
                <tr class="${bgClass}">
                    <td>${i + 1}월</td>
                    <td>${d24.toLocaleString()}</td>
                    <td>${d25.toLocaleString()}</td>
                    <td style="color:${colorClass}">${diff > 0 ? '+' : ''}${diff.toLocaleString()}</td>
                    <td style="color:${colorClass}">${diff > 0 ? '+' : ''}${pct}%</td>
                </tr>
            `;
        }
        tbody.innerHTML = html;
    }

    function renderTargetTable() {
        const tbody = document.getElementById('targetTableBody');
        const currentMonthIdx = currentDate.getMonth(); 
        let html = '';

        for (let i = 0; i < 12; i++) {
            const mStr = String(i + 1).padStart(2, '0');
            const data = statsDB['2025'][mStr];
            const bgClass = (i === currentMonthIdx) ? 'current-month-row' : '';

            html += `
                <tr class="${bgClass}">
                    <td>${i + 1}월</td>
                    <td>${data.f}%</td>
                    <td>${data.age30}%</td>
                    <td>${data.age40}%</td>
                </tr>
            `;
        }
        tbody.innerHTML = html;
    }

    // --- Render Logic ---
    function renderReport() {
        const year = currentDate.getFullYear();
        const monthIndex = currentDate.getMonth(); 
        const monthStr = String(monthIndex + 1).padStart(2, '0');
        
        const nextD = new Date(currentDate);
        nextD.setMonth(nextD.getMonth() + 1);
        const nextMonthStr = String(nextD.getMonth() + 1).padStart(2, '0');

        document.getElementById('currentDate').innerText = `${year}.${monthStr}`;
        document.getElementById('nextMonthLabel').innerText = `${nextMonthStr}월`;

        const cur = statsDB[year][monthStr];
        const prev = statsDB[year-1][monthStr];
        const prevNext = statsDB[year-1][nextMonthStr];
        const rank = rankDB[monthStr];
        const score = scoreDB[monthStr];
        const events = eventDB[monthStr] || [];
        const nextEvents = eventDB[nextMonthStr] || [];

        updateKPI('total', cur.total, prev.total);
        updateKPI('visit', cur.visit, prev.visit);
        updateKPI('new', cur.new, prev.new);
        document.getElementById('kpi-rank').innerText = rank;
        document.getElementById('kpi-score').innerText = score;

        renderMainChart(statsDB['2024'], statsDB['2025'], monthIndex);
        renderVisitorChart(cur);
        renderAgeChart(cur);
        renderEventTable(events);
        renderAIInsight(cur, prev, prevNext, nextEvents, monthStr, nextMonthStr);
    }

    function updateKPI(id, curr, prev) {
        const diff = curr - prev;
        const pct = ((diff/prev)*100).toFixed(1);
        const elVal = document.getElementById(`kpi-${id}`);
        const elDiff = document.getElementById(`diff-${id}`);
        elVal.innerText = curr.toLocaleString();
        const cls = diff > 0 ? 'up' : 'down';
        const icon = diff > 0 ? '▲' : '▼';
        elDiff.className = `kpi-diff ${cls}`;
        elDiff.innerHTML = `${icon} ${Math.abs(diff).toLocaleString()} (${pct}%) YoY`;
    }

    function renderMainChart(db24, db25, activeIndex) {
        const ctx = document.getElementById('mainChart').getContext('2d');
        if(mainChart) mainChart.destroy();
        if(blinkInterval) clearInterval(blinkInterval);

        const pointRadiuses = new Array(12).fill(3);
        pointRadiuses[activeIndex] = 8; 
        const pointColors = new Array(12).fill('#1E293B');
        pointColors[activeIndex] = '#D97706'; 

        mainChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: Array.from({length:12}, (_,i)=>`${i+1}월`),
                datasets: [
                    { 
                        label:'2025', 
                        data: Object.values(db25).map(d=>d.visit), 
                        borderColor:'#1E293B', 
                        backgroundColor:'rgba(30, 41, 59, 0.1)',
                        borderWidth:2, 
                        pointRadius: pointRadiuses,
                        pointBackgroundColor: pointColors,
                        pointBorderColor: '#fff',
                        fill:true, 
                        tension:0.4,
                        order: 1
                    },
                    { 
                        label:'2024', 
                        data: Object.values(db24).map(d=>d.visit), 
                        borderColor:'#94A3B8', 
                        borderDash:[5,5], 
                        borderWidth:2, 
                        pointRadius: 4, 
                        pointHoverRadius: 6, 
                        fill:false, 
                        tension:0.4,
                        order: 2
                    }
                ]
            },
            options: { 
                responsive:true, 
                maintainAspectRatio:false, 
                interaction: {
                    mode: 'index',
                    intersect: false,
                },
                plugins:{legend:{position:'top', align:'end'}}, 
                scales:{x:{grid:{display:false}}, y:{grid:{color:'#F1F5F9'}}} 
            }
        });

        let isBig = true;
        blinkInterval = setInterval(() => {
            if(!mainChart) return;
            const ds = mainChart.data.datasets[0];
            ds.pointRadius[activeIndex] = isBig ? 4 : 9;
            ds.pointBackgroundColor[activeIndex] = isBig ? '#1E293B' : '#D97706';
            mainChart.update('none'); 
            isBig = !isBig;
        }, 600);
    }

    function renderVisitorChart(data) {
        const ctx = document.getElementById('visitorChart').getContext('2d');
        if(visitorChart) visitorChart.destroy();
        visitorChart = new Chart(ctx, {
            type: 'doughnut',
            data: {
                labels: ['멤버', '검색(비멤버)'],
                datasets: [{ data: [data.mem, data.non], backgroundColor: ['#1E293B', '#3B82F6'], borderWidth:0 }]
            },
            options: { responsive:true, maintainAspectRatio:false, cutout:'70%', plugins:{legend:{position:'right', labels:{boxWidth:10}}} }
        });
    }

    function renderAgeChart(data) {
        const ctx = document.getElementById('ageChart').getContext('2d');
        if(ageChart) ageChart.destroy();
        const others = 100 - (data.age30 + data.age40);
        ageChart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: ['30대', '40대', '기타'],
                datasets: [{ label:'여성', data: [data.age30, data.age40, others], backgroundColor: ['#FCD34D', '#1E293B', '#E2E8F0'], borderRadius:4, barThickness: 20 }]
            },
            options: { responsive:true, maintainAspectRatio:false, indexAxis:'y', plugins:{legend:{display:false}}, scales:{x:{display:false}, y:{grid:{display:false}}} }
        });
    }

    function renderEventTable(events) {
        const tbody = document.getElementById('eventTable');
        const sorted = [...events].sort((a,b) => b.v - a.v);
        if(!sorted.length) { tbody.innerHTML = '<tr><td colspan="4">데이터 없음</td></tr>'; return; }
        
        tbody.innerHTML = sorted.map((e,i) => `
            <tr>
                <td style="color:#64748B" title="${e.d}">${e.d}</td>
                <td style="font-weight:600; color:#1E293B;" title="${e.t}">
                    ${i<3 ? `<span class="rank-circle">${i+1}</span>` : ''} ${e.t}
                </td>
                <td>${e.v.toLocaleString()}</td>
                <td style="color:#3B82F6; font-weight:600;">${(e.c+e.l).toLocaleString()}</td>
            </tr>
        `).join('');
    }

    function renderAIInsight(cur, prev, prevNext, nextEvents, month, nextMonth) {
        const vDiff = cur.visit - prev.visit;
        let text = `<p>작년 ${month}월 대비 방문 횟수가 <span class="ai-highlight">${Math.abs(vDiff).toLocaleString()}회 ${vDiff>0?'증가':'감소'}</span>했습니다.`;
        if(cur.non > cur.mem) text += ` 검색 유입 비중이 높아 <span class="ai-highlight">신규 트래픽</span> 확보에 유리한 구조입니다.</p>`;
        else text += ` 멤버 재방문 비중이 높아 <span class="ai-highlight">충성 유저</span> 기반이 탄탄합니다.</p>`;
        document.getElementById('ai-analysis').innerHTML = text;

        let html = '';
        if(prevNext) {
            const trend = prevNext.visit - prev.visit;
            if(trend > 0) html += `<div class="strategy-box"><span class="strategy-head">📈 상승 기류 포착</span>작년 데이터 분석 결과, ${nextMonth}월은 트래픽이 자연 증가하는 시기입니다.</div>`;
            else html += `<div class="strategy-box"><span class="strategy-head">🛡️ 비수기 방어</span>작년 동월 트래픽 감소가 관측되었습니다. 체류 시간을 늘릴 참여형 이벤트가 필수입니다.</div>`;
        }
        if(nextEvents.length) {
            const best = nextEvents.sort((a,b)=>b.v-a.v)[0];
            html += `<div class="strategy-box"><span class="strategy-head">🔥 킬러 콘텐츠 추천</span>익월 예정된 <strong>'${best.t}'</strong> (예상 조회수 ${best.v.toLocaleString()})를 메인 배너로 집중 홍보하세요.</div>`;
        }
        document.getElementById('ai-strategy').innerHTML = html;
    }
</script>
</body>
</html>

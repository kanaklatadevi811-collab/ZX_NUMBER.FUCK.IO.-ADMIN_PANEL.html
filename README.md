<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Color & Number Prediction Panel | Auto IST Sync + Admin Override</title>
    <!-- Tailwind CSS + Google Fonts + Font Awesome -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:opsz,wght@14..32,300;400;500;600;700;800&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <style>
        * { font-family: 'Inter', sans-serif; }
        body {
            background: radial-gradient(circle at 10% 20%, rgba(10, 20, 35, 1) 0%, rgba(2, 8, 18, 1) 100%);
            min-height: 100vh;
        }
        .glass-card {
            background: rgba(15, 25, 45, 0.55);
            backdrop-filter: blur(12px);
            border-radius: 2rem;
            border: 1px solid rgba(80, 140, 210, 0.25);
            box-shadow: 0 20px 35px -12px rgba(0,0,0,0.5);
        }
        .glass-card-dark {
            background: rgba(8, 12, 24, 0.7);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 215, 0, 0.2);
        }
        .confidence-fill {
            background: linear-gradient(90deg, #3b82f6, #a855f7);
            border-radius: 20px;
            transition: width 0.4s ease;
        }
        .countdown-timer {
            font-feature-settings: "tnum";
            font-variant-numeric: tabular-nums;
        }
        @keyframes pulse-gentle {
            0% { opacity: 0.7; transform: scale(1);}
            100% { opacity: 1; transform: scale(1.02);}
        }
        .history-row { transition: background 0.1s; }
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-track { background: #0f172a; }
        ::-webkit-scrollbar-thumb { background: #3b82f6; border-radius: 10px; }
        .admin-badge { background: rgba(255, 193, 7, 0.2); border-left: 3px solid #fbbf24; }
        .manual-override { animation: glow 0.5s ease-out; }
        @keyframes glow {
            0% { box-shadow: 0 0 0 0 rgba(251, 191, 36, 0.7); }
            100% { box-shadow: 0 0 0 8px rgba(251, 191, 36, 0); }
        }
        .period-badge {
            background: linear-gradient(135deg, #1e293b, #0f172a);
            border-radius: 12px;
            padding: 2px 8px;
            font-family: monospace;
        }
    </style>
</head>
<body class="text-white p-4 md:p-6">

    <div class="max-w-6xl mx-auto">
        <!-- Header -->
        <div class="flex justify-between items-center mb-6 flex-wrap gap-3">
            <div class="flex items-center gap-3">
                <i class="fas fa-brain text-3xl text-cyan-400"></i>
                <h1 class="text-2xl md:text-3xl font-bold bg-gradient-to-r from-cyan-300 to-purple-400 bg-clip-text text-transparent">PREDICTION ENGINE</h1>
                <span id="admin-mode-indicator" class="hidden text-xs bg-yellow-600/80 px-2 py-1 rounded-full ml-2"><i class="fas fa-crown"></i> ADMIN ACTIVE</span>
            </div>
            <div class="glass-card px-5 py-2 rounded-2xl flex items-center gap-2 text-sm font-mono">
                <i class="fas fa-clock text-emerald-400"></i>
                <span id="ist-time-label" class="font-semibold tracking-wide">--:--:-- IST</span>
                <span class="text-xs text-gray-300 ml-1">(Synchronized)</span>
            </div>
        </div>

        <!-- Main Prediction Panel -->
        <div class="glass-card p-5 md:p-8 mb-6">
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6 items-center">
                <!-- Removed Period Block -->
                <!-- Predicted Number + Color - Centered and larger -->
                <div class="text-center">
                    <div class="text-sm uppercase tracking-wider text-gray-300 mb-2"><i class="fas fa-chart-line mr-1"></i> AI PREDICTION</div>
                    <div class="flex flex-col items-center gap-3">
                        <div class="flex items-baseline gap-4">
                            <span id="predicted-number" class="text-7xl md:text-8xl font-black number-badge bg-black/30 px-6 py-3 rounded-3xl inline-block">?</span>
                            <span id="predicted-color-badge" class="text-3xl font-bold uppercase px-5 py-2 rounded-full bg-opacity-30 backdrop-blur-sm">---</span>
                        </div>
                        <div class="w-full max-w-[240px] mt-2">
                            <div class="flex justify-between text-xs text-gray-300 mb-1"><span>Confidence Meter</span><span id="confidence-percent" class="font-mono">0%</span></div>
                            <div class="h-3 w-full bg-gray-800 rounded-full overflow-hidden">
                                <div id="confidence-bar" class="confidence-fill h-full w-0 rounded-full"></div>
                            </div>
                        </div>
                    </div>
                </div>
                <!-- Countdown Timer -->
                <div class="text-center">
                    <div class="text-sm uppercase tracking-wider text-gray-300 mb-1"><i class="fas fa-hourglass-half mr-1"></i> NEXT UPDATE IN</div>
                    <div class="text-5xl md:text-6xl font-black font-mono countdown-timer text-purple-300" id="countdown-seconds">00</div>
                    <div class="text-xs text-gray-400 mt-1">auto prediction at :00 seconds</div>
                </div>
            </div>
            <!-- Hidden admin override badge -->
            <div id="admin-override-badge" class="text-xs mt-3 text-center hidden text-yellow-400"><i class="fas fa-hand-sparkles"></i> Admin Override Active</div>
        </div>

        <!-- AI Insight & Admin Panel -->
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-5 mb-6">
            <div class="glass-card p-4 col-span-1 lg:col-span-2">
                <div class="flex items-center gap-2 mb-3 border-b border-white/10 pb-2">
                    <i class="fas fa-robot text-indigo-300"></i>
                    <h3 class="font-semibold">AI Pattern Analysis</h3>
                    <span class="text-xs bg-blue-900/50 px-2 py-0.5 rounded-full ml-2">Odd=Green | Even=Red | 0=Violet</span>
                </div>
                <div id="pattern-insight" class="text-gray-200 text-sm leading-relaxed">Initializing pattern recognition engine...</div>
                <div class="flex flex-wrap gap-3 mt-3 text-xs">
                    <div class="bg-gray-800/50 px-3 py-1 rounded-full"><span class="text-cyan-300">Streak:</span> <span id="streak-indicator">-</span></div>
                    <div class="bg-gray-800/50 px-3 py-1 rounded-full"><span class="text-purple-300">Top Pattern:</span> <span id="top-pattern">-</span></div>
                    <div class="bg-gray-800/50 px-3 py-1 rounded-full"><span class="text-emerald-300">Weighted Score:</span> <span id="weighted-score">-</span></div>
                </div>
            </div>
            
            <!-- ADMIN PANEL -->
            <div class="glass-card-dark p-4 border-yellow-500/30 relative">
                <div class="flex items-center justify-between mb-3 border-b border-yellow-500/30 pb-2">
                    <div class="flex items-center gap-2">
                        <i class="fas fa-lock text-yellow-400"></i>
                        <h3 class="font-bold text-yellow-300">ADMIN PANEL</h3>
                    </div>
                    <i class="fas fa-shield-alt text-gray-400 text-xs"></i>
                </div>
                <div id="admin-login-area">
                    <div class="flex gap-2 mb-3">
                        <input type="password" id="admin-key-input" placeholder="Enter Master Key" class="w-full bg-black/50 border border-gray-600 rounded-xl px-3 py-2 text-sm text-white focus:outline-none focus:border-yellow-500">
                        <button id="admin-login-btn" class="bg-yellow-600/80 hover:bg-yellow-500 px-4 py-2 rounded-xl text-sm font-semibold transition"><i class="fas fa-key"></i> Unlock</button>
                    </div>
                </div>
                <div id="admin-controls" class="hidden">
                    <div class="text-xs text-yellow-400 mb-2 flex items-center gap-2"><i class="fas fa-crown"></i> Live Manual Override</div>
                    <div class="flex gap-2 mb-3">
                        <select id="admin-number" class="bg-black/70 border border-yellow-600 rounded-xl px-3 py-2 text-white text-center">
                            <option value="0">0 (Violet)</option><option value="1">1 (Green)</option><option value="2">2 (Red)</option>
                            <option value="3">3 (Green)</option><option value="4">4 (Red)</option><option value="5">5 (Violet)</option>
                            <option value="6">6 (Red)</option><option value="7">7 (Green)</option><option value="8">8 (Red)</option>
                            <option value="9">9 (Green)</option>
                        </select>
                        <button id="admin-push-btn" class="bg-yellow-600 hover:bg-yellow-500 px-4 py-2 rounded-xl font-bold text-sm flex-1 transition"><i class="fas fa-bullhorn"></i> PUSH PREDICTION</button>
                    </div>
                    <button id="admin-reset-history" class="text-xs bg-red-800/40 hover:bg-red-700/60 w-full py-1.5 rounded-lg transition"><i class="fas fa-trash-alt"></i> Reset History (Clear & Fresh)</button>
                    <div class="text-[10px] text-gray-400 mt-2 text-center">Override lasts 1 minute | Auto-sync next minute</div>
                </div>
            </div>
        </div>

        <!-- Historical Chart -->
        <div class="glass-card p-5">
            <div class="flex justify-between items-center mb-4 flex-wrap gap-2">
                <h3 class="font-semibold flex items-center gap-2"><i class="fas fa-table-list text-blue-300"></i> HISTORICAL CHART (last 15 periods)</h3>
                <div class="text-xs text-gray-400 bg-black/30 px-3 py-1 rounded-full">Odd=Green | Even=Red | 0=Violet</div>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-center text-sm">
                    <thead class="border-b border-white/20 text-gray-300">
                        <tr><th class="pb-2 font-medium">Period (IST)</th><th class="pb-2 font-medium">Number</th><th class="pb-2 font-medium">Color</th><th class="pb-2 font-medium hidden md:table-cell">Pattern</th></tr>
                    </thead>
                    <tbody id="history-tbody" class="divide-y divide-white/5"><tr><td colspan="4" class="py-5 text-gray-400">Loading...</td></tr>
                </tbody>
            </table>
            </div>
            <div class="text-xs text-gray-500 mt-3 text-center italic">* Weighted probability: P(x) = Freq(x) * Recency(x) / ΣPatternStrength</div>
        </div>
    </div>

    <script>
        // ======================= COLOR RULE =======================
        // Odd numbers (1,3,5,7,9) -> GREEN | Even (2,4,6,8) -> RED | 0 -> VIOLET
        function getColorFromNumber(num) {
            if (num === 0) return "Violet";
            return (num % 2 === 0) ? "Red" : "Green";
        }
        
        // ======================= PERIOD PARSING =======================
        function parsePeriodToDate(periodStr) {
            if (!periodStr || periodStr.length !== 12) return null;
            const year = parseInt(periodStr.substring(0,4));
            const month = parseInt(periodStr.substring(4,6)) - 1;
            const day = parseInt(periodStr.substring(6,8));
            const hour = parseInt(periodStr.substring(8,10));
            const minute = parseInt(periodStr.substring(10,12));
            return new Date(Date.UTC(year, month, day, hour, minute, 0, 0));
        }
        
        function formatPeriodFromDate(date) {
            const year = date.getFullYear();
            const month = String(date.getMonth() + 1).padStart(2,'0');
            const day = String(date.getDate()).padStart(2,'0');
            const hours = String(date.getHours()).padStart(2,'0');
            const minutes = String(date.getMinutes()).padStart(2,'0');
            return `${year}${month}${day}${hours}${minutes}`;
        }
        
        function getSeedFromPeriod(periodStr) {
            let hash = 0;
            for (let i = 0; i < periodStr.length; i++) hash = ((hash << 5) - hash) + periodStr.charCodeAt(i);
            return Math.abs(hash);
        }
        
        function getHistoricalResultForPeriod(periodDate) {
            const periodStr = formatPeriodFromDate(periodDate);
            const seed = getSeedFromPeriod(periodStr);
            let rng = (seed * 9301 + 49297) % 233280;
            let randomVal = rng / 233280;
            let num = Math.floor(randomVal * 10);
            return {
                periodStr: periodStr,
                number: num,
                color: getColorFromNumber(num),
                timestamp: periodDate.getTime()
            };
        }
        
        function buildFreshHistoryFromNow(nowIST) {
            let history = [];
            for (let i = 14; i >= 0; i--) {
                let pastDate = new Date(nowIST.getTime() - (i * 60000));
                let result = getHistoricalResultForPeriod(pastDate);
                history.push(result);
            }
            return history.sort((a,b) => a.timestamp - b.timestamp);
        }
        
        function generateNextMinuteResult(periodDate, previousHistory) {
            let baseResult = getHistoricalResultForPeriod(periodDate);
            if (previousHistory.length >= 3) {
                let last = previousHistory[previousHistory.length-1].number;
                let secondLast = previousHistory[previousHistory.length-2].number;
                if (last === secondLast && Math.random() < 0.6) {
                    baseResult.number = last;
                    baseResult.color = getColorFromNumber(last);
                }
            }
            return baseResult;
        }
        
        // ======================= AI PATTERN RECOGNITION =======================
        function analyzePatterns(history) {
            if (history.length < 4) return { patternStrength: 0.5, patternName: "Insufficient data", streakColor: null, currentStreak: 0, abab: false, aabb: false };
            let colorsSeq = history.map(h => h.color);
            let numbersSeq = history.map(h => h.number);
            let currentStreak = 1;
            let streakColor = colorsSeq[colorsSeq.length-1];
            for (let i=colorsSeq.length-2; i>=0; i--) {
                if (colorsSeq[i] === colorsSeq[i+1]) currentStreak++;
                else break;
            }
            let abab = false;
            if (numbersSeq.length >=4) {
                let a = numbersSeq[numbersSeq.length-4], b = numbersSeq[numbersSeq.length-3];
                if (a === numbersSeq[numbersSeq.length-2] && b === numbersSeq[numbersSeq.length-1]) abab = true;
            }
            let aabb = false;
            if (numbersSeq.length >=4) {
                if (numbersSeq[numbersSeq.length-4] === numbersSeq[numbersSeq.length-3] && 
                    numbersSeq[numbersSeq.length-2] === numbersSeq[numbersSeq.length-1] &&
                    numbersSeq[numbersSeq.length-4] !== numbersSeq[numbersSeq.length-2]) aabb = true;
            }
            let patternStrength = 0.5;
            let patternName = "Random walk";
            if (abab) { patternStrength = 0.82; patternName = "AB-AB Pattern"; }
            else if (aabb) { patternStrength = 0.78; patternName = "AABB Pattern"; }
            else if (currentStreak >= 3) { patternStrength = 0.7 + (currentStreak * 0.05); patternName = `Long Dragon (${currentStreak}x ${streakColor})`; }
            else if (currentStreak === 2) { patternStrength = 0.62; patternName = "Color Repeat"; }
            return { patternStrength, patternName, streakColor, currentStreak, abab, aabb };
        }
        
        function computeWeightedPrediction(history, patternMeta) {
            if (history.length === 0) return { number: 5, confidence: 40, weightedScore: 0 };
            let lastIndexWeight = {};
            for (let i=0; i<history.length; i++) {
                let num = history[i].number;
                let recency = Math.pow(1.2, i);
                if (!lastIndexWeight[num]) lastIndexWeight[num] = 0;
                lastIndexWeight[num] += recency;
            }
            let freqMap = {};
            history.forEach(h => { freqMap[h.number] = (freqMap[h.number]||0)+1; });
            let weightedScores = {};
            for (let num=0; num<=9; num++) {
                let freq = freqMap[num] || 0.2;
                let recencyVal = lastIndexWeight[num] || 0.5;
                weightedScores[num] = (freq + 0.5) * (recencyVal + 0.8);
            }
            let adjustedWeights = {};
            let sumAdj = 0;
            for (let num=0; num<=9; num++) {
                let boost = 1.0;
                if (patternMeta.currentStreak >= 2 && patternMeta.streakColor) {
                    if (getColorFromNumber(num) === patternMeta.streakColor) boost = 1.4;
                }
                if (patternMeta.abab && history.length>=2 && num === history[history.length-3]?.number) boost = 1.35;
                if (patternMeta.aabb && history.length>=2 && num === history[history.length-2]?.number) boost = 1.3;
                adjustedWeights[num] = weightedScores[num] * boost * patternMeta.patternStrength;
                sumAdj += adjustedWeights[num];
            }
            let rand = Math.random() * sumAdj;
            let accum = 0, selectedNum = 0;
            for (let num=0; num<=9; num++) {
                accum += adjustedWeights[num];
                if (rand <= accum) { selectedNum = num; break; }
            }
            let maxWeight = Math.max(...Object.values(adjustedWeights));
            let confidenceRaw = (adjustedWeights[selectedNum] / maxWeight) * 100;
            let confidence = Math.min(96, Math.max(55, Math.floor(confidenceRaw * 0.9 + patternMeta.patternStrength * 25)));
            return { number: selectedNum, confidence, weightedScore: adjustedWeights[selectedNum].toFixed(2) };
        }
        
        function runAIPrediction(history) {
            if (!history.length) return { number: 5, color: "Violet", confidence: 50, patternInsight: "No data", topPattern: "None", streakInfo: "-", weightedScore: "0" };
            let patternMeta = analyzePatterns(history);
            let { number, confidence, weightedScore } = computeWeightedPrediction(history, patternMeta);
            let predictedColor = getColorFromNumber(number);
            let insight = `📊 Pattern: ${patternMeta.patternName} (strength ${(patternMeta.patternStrength*100).toFixed(0)}%). Weighted probability favors ${number} (${predictedColor}). ${patternMeta.currentStreak>=2 ? `🔥 ${patternMeta.currentStreak}-color streak detected.` : "Recency & frequency combined."}`;
            return { number, color: predictedColor, confidence, patternInsight: insight, topPattern: patternMeta.patternName, streakInfo: patternMeta.currentStreak ? `${patternMeta.currentStreak}x ${patternMeta.streakColor || ''}` : "No streak", weightedScore };
        }
        
        // ======================= GLOBAL STATE =======================
        let historicalData = [];
        let currentPrediction = {};
        let serverTimeOffset = 0;
        let countdownInterval = null;
        let isAdminAuthenticated = false;
        let adminOverrideActive = false;
        let adminManualPrediction = null;
        const MASTER_KEY = "ZX_BHAIYA_JI";
        
        // UI Updates - No period display anymore
        function updateUIWithPrediction(pred, isOverride = false) {
            document.getElementById('predicted-number').innerText = pred.number !== undefined ? pred.number : '?';
            let colorSpan = document.getElementById('predicted-color-badge');
            let color = pred.color || 'Gray';
            colorSpan.innerText = color;
            let gradientClass = color === 'Red' ? 'bg-red-600/40 text-red-200 border-red-400/50' : (color === 'Green' ? 'bg-green-600/40 text-green-200 border-green-400/50' : 'bg-purple-600/40 text-purple-200 border-purple-400/50');
            colorSpan.className = `text-2xl font-bold uppercase px-5 py-2 rounded-full backdrop-blur-sm ${gradientClass}`;
            document.getElementById('confidence-percent').innerText = `${pred.confidence}%`;
            document.getElementById('confidence-bar').style.width = `${pred.confidence}%`;
            document.getElementById('pattern-insight').innerHTML = pred.patternInsight || "Analyzing...";
            document.getElementById('streak-indicator').innerText = pred.streakInfo || '-';
            document.getElementById('top-pattern').innerText = pred.topPattern || '-';
            document.getElementById('weighted-score').innerText = pred.weightedScore || '-';
            if (isOverride) {
                let badge = document.getElementById('admin-override-badge');
                badge.classList.remove('hidden');
                setTimeout(() => { if(!adminOverrideActive) badge.classList.add('hidden'); }, 3000);
            }
        }
        
        function renderHistoryTable(history) {
            let tbody = document.getElementById('history-tbody');
            if (!history.length) { tbody.innerHTML = '<tr><td colspan="4" class="py-4 text-gray-400">No data</td></tr>'; return; }
            let html = '';
            let recentList = [];
            for (let i = history.length-1; i >= Math.max(0, history.length-15); i--) {
                let h = history[i];
                let colorClass = h.color === 'Red' ? 'text-red-400 font-bold' : (h.color === 'Green' ? 'text-green-400 font-bold' : 'text-purple-400 font-bold');
                let patternTag = (i>0 && history[i].number === history[i-1].number) ? '🔁 Repeat' : ((i>1 && history[i].color === history[i-1].color && history[i].color === history[i-2].color) ? '🔥 Streak' : '─');
                html += `<tr class="history-row border-b border-white/5">
                            <td class="py-2 font-mono text-xs">${h.periodStr}</td>
                            <td class="py-2 text-xl font-bold">${h.number}</td>
                            <td class="py-2 ${colorClass} font-semibold">${h.color}</td>
                            <td class="py-2 text-gray-400 text-xs hidden md:table-cell">${patternTag}</td>
                          </tr>`;
                if (i >= history.length-5) recentList.push(`${h.periodStr.slice(-4)}: ${h.number}${h.color[0]}`);
            }
            tbody.innerHTML = html;
            let triggersDiv = document.getElementById('recent-triggers');
            if(triggersDiv) triggersDiv.innerHTML = recentList.map(t => `<div class="flex items-center gap-1"><i class="fas fa-chart-line text-[10px]"></i><span>${t}</span></div>`).join('') || "No recent";
        }
        
        // Core refresh
        function refreshPredictionCycle() {
            let nowIST = new Date(Date.now() + serverTimeOffset);
            
            if (adminOverrideActive && adminManualPrediction) {
                let overridePred = { ...adminManualPrediction, confidence: 98, patternInsight: "🔴 ADMIN OVERRIDE: Manual prediction active", topPattern: "Admin", streakInfo: "Manual", weightedScore: "99" };
                updateUIWithPrediction(overridePred, true);
                renderHistoryTable(historicalData);
                return;
            }
            
            if (historicalData.length === 0) {
                historicalData = buildFreshHistoryFromNow(nowIST);
            } else {
                let currentPeriodStr = formatPeriodFromDate(nowIST);
                let lastPeriod = historicalData[historicalData.length-1].periodStr;
                if (lastPeriod !== currentPeriodStr) {
                    let newResult = generateNextMinuteResult(nowIST, historicalData);
                    historicalData.push(newResult);
                    if (historicalData.length > 40) historicalData = historicalData.slice(-30);
                    renderHistoryTable(historicalData);
                }
            }
            let prediction = runAIPrediction(historicalData);
            currentPrediction = prediction;
            updateUIWithPrediction(prediction, false);
            renderHistoryTable(historicalData);
            
            if (adminOverrideActive) {
                adminOverrideActive = false;
                adminManualPrediction = null;
                document.getElementById('admin-override-badge')?.classList.add('hidden');
            }
        }
        
        function resetHistoryAndClear() {
            let nowRef = new Date(Date.now() + serverTimeOffset);
            historicalData = buildFreshHistoryFromNow(nowRef);
            renderHistoryTable(historicalData);
            let pred = runAIPrediction(historicalData);
            updateUIWithPrediction(pred, false);
            const toast = document.createElement('div');
            toast.className = 'fixed bottom-4 right-4 bg-green-800/90 text-white px-4 py-2 rounded-xl text-sm z-50';
            toast.innerHTML = '<i class="fas fa-check-circle"></i> History cleared & reset!';
            document.body.appendChild(toast);
            setTimeout(() => toast.remove(), 2000);
        }
        
        function adminPushManual(numberVal) {
            if (!isAdminAuthenticated) { alert("Admin not authenticated!"); return; }
            let num = parseInt(numberVal);
            let color = getColorFromNumber(num);
            adminManualPrediction = { number: num, color: color, confidence: 98 };
            adminOverrideActive = true;
            let overridePred = { ...adminManualPrediction, confidence: 98, patternInsight: "🔐 ADMIN MANUAL OVERRIDE", topPattern: "Manual Command", streakInfo: "Admin", weightedScore: "100" };
            updateUIWithPrediction(overridePred, true);
            document.querySelector('.glass-card').classList.add('manual-override');
            setTimeout(() => document.querySelector('.glass-card').classList.remove('manual-override'), 600);
        }
        
        function setupAdmin() {
            document.getElementById('admin-login-btn').addEventListener('click', () => {
                let key = document.getElementById('admin-key-input').value;
                if (key === MASTER_KEY) {
                    isAdminAuthenticated = true;
                    document.getElementById('admin-login-area').classList.add('hidden');
                    document.getElementById('admin-controls').classList.remove('hidden');
                    document.getElementById('admin-mode-indicator').classList.remove('hidden');
                } else { alert('Invalid Key!'); }
            });
            document.getElementById('admin-push-btn').addEventListener('click', () => {
                let val = document.getElementById('admin-number').value;
                adminPushManual(val);
            });
            document.getElementById('admin-reset-history').addEventListener('click', () => {
                if(confirm('Reset all historical chart data?')) resetHistoryAndClear();
            });
        }
        
        async function syncIST() {
            try {
                const res = await fetch('https://worldtimeapi.org/api/timezone/Asia/Kolkata');
                const data = await res.json();
                let serverIST = new Date(data.utc_datetime);
                let localNow = new Date();
                serverTimeOffset = serverIST.getTime() - localNow.getTime();
            } catch(e) {
                serverTimeOffset = 5.5 * 60 * 60 * 1000;
            }
            updateISTclock();
        }
        
        function updateISTclock() {
            let ist = new Date(Date.now() + serverTimeOffset);
            document.getElementById('ist-time-label').innerHTML = `${ist.toLocaleTimeString('en-IN', { hour12: false })} IST`;
        }
        
        function scheduleCountdown() {
            if (countdownInterval) clearInterval(countdownInterval);
            countdownInterval = setInterval(() => {
                let nowIST = new Date(Date.now() + serverTimeOffset);
                let remaining = 60 - nowIST.getSeconds();
                if (remaining === 60) remaining = 0;
                document.getElementById('countdown-seconds').innerText = String(remaining).padStart(2,'0');
                if (remaining === 0) refreshPredictionCycle();
                updateISTclock();
            }, 200);
        }
        
        async function init() {
            await syncIST();
            let nowIST = new Date(Date.now() + serverTimeOffset);
            historicalData = buildFreshHistoryFromNow(nowIST);
            renderHistoryTable(historicalData);
            refreshPredictionCycle();
            scheduleCountdown();
            setupAdmin();
        }
        init();
    </script>
</body>
</html>

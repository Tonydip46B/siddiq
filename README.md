<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Creator's Compass</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
</head>
<body class="bg-slate-900 text-white font-sans selection:bg-blue-500 selection:text-white min-h-screen">

    <header class="border-b border-slate-800 p-6 flex justify-between items-center">
        <div class="flex items-center gap-2">
            <div class="bg-blue-600 p-2 rounded-lg">
                <i data-lucide="zap" class="text-white"></i>
            </div>
            <h1 class="text-2xl font-bold tracking-tight">Creator's Compass</h1>
        </div>
        <div class="text-sm text-slate-400">Beta v1.0</div>
    </header>

    <main class="max-w-4xl mx-auto p-6 mt-8">
        <div class="flex gap-4 mb-8">
            <button onclick="switchTab('shield')" id="btn-shield" class="flex items-center gap-2 px-6 py-3 rounded-full font-medium transition-all bg-blue-600 text-white shadow-lg shadow-blue-900/50">
                <i data-lucide="shield-check"></i> Algo-Shield
            </button>
            <button onclick="switchTab('pulse')" id="btn-pulse" class="flex items-center gap-2 px-6 py-3 rounded-full font-medium transition-all bg-slate-800 text-slate-400 hover:bg-slate-700">
                <i data-lucide="activity"></i> Pulse Monitor
            </button>
        </div>

        <div id="tab-shield" class="grid md:grid-cols-2 gap-8">
            <div class="space-y-4">
                <label class="block text-slate-400 text-sm font-semibold uppercase tracking-wider">Paste Caption / Script</label>
                <textarea 
                    id="input-text"
                    oninput="analyzeText()"
                    class="w-full h-96 bg-slate-800 border border-slate-700 rounded-xl p-4 text-slate-200 focus:ring-2 focus:ring-blue-500 focus:outline-none resize-none text-lg leading-relaxed"
                    placeholder="Type here... e.g. 'I want to kill this workout!'"
                ></textarea>
            </div>

            <div class="bg-slate-800 rounded-xl border border-slate-700 p-6 h-96 overflow-y-auto">
                <div id="empty-state" class="h-full flex flex-col items-center justify-center text-slate-500 space-y-4">
                    <i data-lucide="shield-check" class="w-12 h-12 opacity-20"></i>
                    <p>Waiting for text...</p>
                </div>

                <div id="results-state" class="hidden space-y-6">
                    <div class="flex items-center justify-between bg-slate-900 p-4 rounded-lg border border-slate-700">
                        <div>
                            <p class="text-slate-400 text-xs uppercase font-bold">Reach Score</p>
                            <p id="score-display" class="text-3xl font-bold text-green-400">100%</p>
                        </div>
                        <div id="score-icon"></div>
                    </div>

                    <div>
                        <h3 class="text-sm font-bold text-slate-300 mb-3 uppercase">Detected Risks</h3>
                        <ul id="risk-list" class="space-y-2"></ul>
                    </div>
                </div>
            </div>
        </div>

        <div id="tab-pulse" class="hidden max-w-2xl mx-auto text-center space-y-8 py-12">
            <div class="space-y-2">
                <h2 class="text-3xl font-bold">Check Political Heat</h2>
                <p class="text-slate-400">Is this topic safe to post about right now?</p>
            </div>
            
            <div class="relative">
                <input 
                    type="text" 
                    id="pulse-topic"
                    placeholder="Enter topic (e.g. Gas Stoves, Nike)"
                    class="w-full bg-slate-800 border border-slate-700 rounded-full py-4 pl-6 pr-16 text-lg text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                >
                <button 
                    onclick="checkPulse()"
                    class="absolute right-2 top-2 bg-blue-600 hover:bg-blue-500 text-white p-2 rounded-full transition-colors"
                >
                    <i data-lucide="search"></i>
                </button>
            </div>

            <div id="pulse-loading" class="hidden animate-pulse text-blue-400 font-mono">Scanning Google Trends...</div>
            <div id="pulse-result" class="hidden p-6 rounded-xl border-l-4 text-left"></div>
        </div>
    </main>

    <script>
        lucide.createIcons();

        const BANNED_WORDS = {
            highRisk: ["kill", "shoot", "suicide", "murder", "gun", "weapon", "bomb", "terrorist", "isis", "child abuse"],
            algoSpeak: ["dead", "died", "sexual", "assault", "rape", "porn", "naked", "white", "black", "racist"],
            salesTrigger: ["buy now", "click here", "giveaway", "free money", "crypto", "bitcoin", "guaranteed", "winner"],
            politicalHeat: ["vaccine", "agenda", "woke", "patriot", "border", "election", "climate hoax", "boycott", "trump", "biden", "kamala"]
        };

        function switchTab(tab) {
            const shieldTab = document.getElementById('tab-shield');
            const pulseTab = document.getElementById('tab-pulse');
            const btnShield = document.getElementById('btn-shield');
            const btnPulse = document.getElementById('btn-pulse');

            if (tab === 'shield') {
                shieldTab.classList.remove('hidden');
                pulseTab.classList.add('hidden');
                btnShield.className = "flex items-center gap-2 px-6 py-3 rounded-full font-medium transition-all bg-blue-600 text-white shadow-lg shadow-blue-900/50";
                btnPulse.className = "flex items-center gap-2 px-6 py-3 rounded-full font-medium transition-all bg-slate-800 text-slate-400 hover:bg-slate-700";
            } else {
                shieldTab.classList.add('hidden');
                pulseTab.classList.remove('hidden');
                btnPulse.className = "flex items-center gap-2 px-6 py-3 rounded-full font-medium transition-all bg-blue-600 text-white shadow-lg shadow-blue-900/50";
                btnShield.className = "flex items-center gap-2 px-6 py-3 rounded-full font-medium transition-all bg-slate-800 text-slate-400 hover:bg-slate-700";
            }
        }

        function analyzeText() {
            const text = document.getElementById('input-text').value;
            const emptyState = document.getElementById('empty-state');
            const resultsState = document.getElementById('results-state');
            
            if (!text) {
                emptyState.classList.remove('hidden');
                resultsState.classList.add('hidden');
                return;
            }

            emptyState.classList.add('hidden');
            resultsState.classList.remove('hidden');

            let score = 100;
            let flaggedItems = [];
            const words = text.toLowerCase().split(/\s+/);

            words.forEach(word => {
                const cleanWord = word.replace(/[^a-zA-Z0-9]/g, "");
                if (BANNED_WORDS.highRisk.includes(cleanWord)) { flaggedItems.push({ word: cleanWord, category: 'CRITICAL (Ban Risk)' }); score -= 20; }
                else if (BANNED_WORDS.algoSpeak.includes(cleanWord)) { flaggedItems.push({ word: cleanWord, category: 'Reach Killer (Use Algo-speak)' }); score -= 10; }
                else if (BANNED_WORDS.salesTrigger.includes(cleanWord)) { flaggedItems.push({ word: cleanWord, category: 'Spam Filter' }); score -= 5; }
                else if (BANNED_WORDS.politicalHeat.includes(cleanWord)) { flaggedItems.push({ word: cleanWord, category: 'Political Hot Button' }); score -= 15; }
            });

            score = Math.max(0, score);
            
            const scoreDisplay = document.getElementById('score-display');
            const scoreIcon = document.getElementById('score-icon');
            const riskList = document.getElementById('risk-list');

            scoreDisplay.innerText = score + "%";
            
            if (score > 80) {
                scoreDisplay.className = "text-3xl font-bold text-green-400";
                scoreIcon.innerHTML = `<i data-lucide="check-circle" class="text-green-500 w-8 h-8"></i>`;
            } else if (score > 50) {
                scoreDisplay.className = "text-3xl font-bold text-yellow-400";
                scoreIcon.innerHTML = `<i data-lucide="alert-triangle" class="text-yellow-500 w-8 h-8"></i>`;
            } else {
                scoreDisplay.className = "text-3xl font-bold text-red-500";
                scoreIcon.innerHTML = `<i data-lucide="alert-triangle" class="text-red-500 w-8 h-8"></i>`;
            }

            riskList.innerHTML = "";
            if (flaggedItems.length === 0) {
                riskList.innerHTML = `<p class="text-green-400 text-sm flex items-center gap-2"><i data-lucide="check-circle" class="w-4 h-4"></i> No risky words detected.</p>`;
            } else {
                flaggedItems.forEach(item => {
                    riskList.innerHTML += `
                        <li class="bg-red-500/10 border border-red-500/20 p-3 rounded-md flex items-start justify-between">
                            <span class="text-red-400 font-mono font-bold">"${item.word}"</span>
                            <span class="text-xs text-red-300 bg-red-900/50 px-2 py-1 rounded">${item.category}</span>
                        </li>`;
                });
            }
            lucide.createIcons();
        }

        function checkPulse() {
            const topic = document.getElementById('pulse-topic').value;
            if (!topic) return;

            document.getElementById('pulse-loading').classList.remove('hidden');
            document.getElementById('pulse-result').classList.add('hidden');

            setTimeout(() => {
                const randomHeat = Math.random();
                const resultBox = document.getElementById('pulse-result');
                let resultHTML = "";

                if (randomHeat > 0.7) {
                    resultBox.className = "p-6 rounded-xl border-l-4 text-left bg-red-500/10 border-red-500 text-red-200";
                    resultHTML = `<p class="text-xl font-bold">DANGER: Viral Spike Detected. High Polarization.</p>`;
                } else if (randomHeat > 0.4) {
                    resultBox.className = "p-6 rounded-xl border-l-4 text-left bg-yellow-500/10 border-yellow-500 text-yellow-200";
                    resultHTML = `<p class="text-xl font-bold">CAUTION: Rising Interest. Tread carefully.</p>`;
                } else {
                    resultBox.className = "p-6 rounded-xl border-l-4 text-left bg-green-500/10 border-green-500 text-green-200";
                    resultHTML = `<p class="text-xl font-bold">SAFE: Low Volume. Safe to post.</p>`;
                }

                resultBox.innerHTML = resultHTML + `<p class="text-sm opacity-70 mt-2">Data source: Google Trends API (simulated)</p>`;
                
                document.getElementById('pulse-loading').classList.add('hidden');
                resultBox.classList.remove('hidden');
            }, 1500);
        }
    </script>
</body>
</html>

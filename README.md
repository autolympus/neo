<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NEO BEAT AI — LIVE MARKET</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Kanit:wght@300;400;600;800&display=swap');
        body { font-family: 'Kanit', sans-serif; background-color: #050505; color: #f4f4f5; }
        .font-mono { font-family: 'Share Tech Mono', monospace; }
        @keyframes pulse-glow {
            0%, 100% { box-shadow: 0 0 15px rgba(34, 197, 94, 0.15); }
            50% { box-shadow: 0 0 30px rgba(34, 197, 94, 0.4); }
        }
        .ai-card-glow { animation: pulse-glow 3s infinite ease-in-out; }
        .terminal-scroll::-webkit-scrollbar { width: 4px; }
        .terminal-scroll::-webkit-scrollbar-thumb { background: #27272a; border-radius: 4px; }
        .price-up { color: #22c55e; transition: color 0.3s; }
        .price-down { color: #ef4444; transition: color 0.3s; }
        .price-loading { color: #facc15; }
    </style>
</head>
<body class="min-h-screen p-3 md:p-6 flex flex-col items-center justify-start">

    <!-- Main Container - UI เดิม 100% -->
    <div class="w-full max-w-xl bg-zinc-950 border border-zinc-800 rounded-2xl p-4 md:p-6 shadow-2xl ai-card-glow">
        
        <!-- Header -->
        <div class="flex justify-between items-center border-b border-zinc-800 pb-3 mb-4">
            <div>
                <h1 class="text-sm md:text-base font-extrabold tracking-wider text-emerald-400 flex items-center gap-2">
                    <span id="header-pulse" class="w-2.5 h-2.5 rounded-full bg-zinc-600"></span>
                    NEO BEAT AI <span class="text-xs font-normal text-amber-400">LIVE MARKET</span>
                </h1>
                <p class="text-[10px] text-zinc-500">Real Market Data Feed — Simulated Trading Only</p>
            </div>
            <span id="conn-badge" class="px-2.5 py-1 rounded text-[10px] font-mono font-bold bg-amber-950 border border-amber-700 text-amber-400">DISCONNECTED</span>
        </div>

        <!-- Info Bar -->
        <div class="bg-blue-950/40 border border-blue-900/50 rounded-lg p-2 mb-3 text-[10px] text-blue-300">
            📡 แหล่งข้อมูลราคาจริง: Frankfurter API (EUR/USD Real Feed)
        </div>

        <!-- Buttons -->
        <div class="grid grid-cols-3 gap-2 mb-4 font-mono text-xs">
            <button id="btn-toggle" onclick="toggleBot()" class="px-3 py-2.5 rounded-xl font-bold bg-emerald-600 hover:bg-emerald-500 text-black transition-all">START BOT</button>
            <button id="btn-sound" onclick="enableSound()" class="px-3 py-2.5 rounded-xl font-bold bg-zinc-900 border border-zinc-700 text-zinc-300 hover:text-white transition-all">SOUND: OFF 🔇</button>
            <button onclick="clearStats()" class="px-3 py-2.5 rounded-xl font-bold bg-zinc-900 border border-zinc-700 text-rose-400 hover:bg-rose-950 transition-all">RESET STATS</button>
        </div>

        <!-- Price & Status -->
        <div class="grid grid-cols-2 gap-2 mb-4 font-mono">
            <div class="bg-zinc-900/80 border border-zinc-800 p-3 rounded-xl text-center">
                <span class="text-[10px] text-zinc-500 block mb-1">EUR/USD LIVE</span>
                <span id="txt-live-price" class="text-lg md:text-xl font-bold text-amber-400 price-loading">LOADING...</span>
            </div>
            <div class="bg-zinc-900/80 border border-zinc-800 p-3 rounded-xl text-center">
                <span class="text-[10px] text-zinc-500 block mb-1">BOT STATUS</span>
                <span id="txt-bot-status" class="text-xs md:text-sm font-bold text-zinc-400">DISCONNECTED</span>
            </div>
        </div>

        <!-- Signal Output -->
        <div class="bg-black border border-zinc-800 rounded-xl p-4 mb-4 text-center relative overflow-hidden">
            <div id="signal-glow" class="absolute inset-0 opacity-10 pointer-events-none transition-all duration-500 bg-zinc-500"></div>
            <span class="text-[10px] font-mono uppercase tracking-widest text-zinc-500 block mb-1">AUTONOMOUS DECISION OUTPUT</span>
            <div id="txt-signal" class="text-2xl md:text-3xl font-extrabold tracking-widest font-mono text-zinc-400 my-2">WAITING FOR SETUP</div>
            <p id="txt-reason" class="text-xs text-zinc-400 font-mono">Press START BOT to begin...</p>
            
            <!-- Countdown -->
            <div id="countdown-box" class="hidden mt-3 pt-3 border-t border-zinc-800 flex items-center justify-center gap-3 font-mono">
                <span class="text-xs text-amber-400 animate-pulse">⚠️ PRE-ENTRY COUNTDOWN:</span>
                <span id="txt-countdown" class="text-lg font-bold text-amber-400 bg-amber-950/50 px-3 py-1 rounded border border-amber-800">10s</span>
            </div>
        </div>

        <!-- Stats -->
        <div class="grid grid-cols-3 gap-2 mb-4 font-mono text-center text-xs">
            <div>
                <span class="text-[9px] text-zinc-500 block">WIN / LOSS</span>
                <span id="stat-wl" class="font-bold">0 / 0</span>
            </div>
            <div>
                <span class="text-[9px] text-zinc-500 block">WIN RATE</span>
                <span id="stat-rate" class="font-bold text-cyan-400">0.0%</span>
            </div>
            <div>
                <span class="text-[9px] text-zinc-500 block">TOTAL TRADES</span>
                <span id="stat-total" class="font-bold text-amber-400">0</span>
            </div>
        </div>

        <!-- Logs -->
        <div class="bg-black border border-zinc-800 rounded-xl p-3 font-mono text-[11px]">
            <div class="flex justify-between items-center border-b border-zinc-800 pb-2 mb-2 text-zinc-400">
                <span>📋 LIVE MARKET LOGS</span>
                <span id="log-count" class="text-zinc-600 text-[10px]">Events: 0</span>
            </div>
            <div id="log-container" class="terminal-scroll h-28 overflow-y-auto text-zinc-300 space-y-1">
                <div class="text-amber-400">[System Ready] — Click START BOT to initialize.</div>
            </div>
        </div>

        <!-- Disclaimer -->
        <div class="mt-3 text-[9px] text-zinc-600 text-center border-t border-zinc-900 pt-2">
            ⚠️ SIMULATION ONLY: Real market data — No real orders are sent. Not financial advice.
        </div>
    </div>

    <!-- Application Script (Fixed & Fully Functional) -->
    <script>
        // ===== STATE =====
        const botState = {
            isRunning: false,
            soundEnabled: false,
            currentPrice: null,
            previousPrice: null,
            priceHistory: [],
            signal: 'WAITING',
            countdown: 10,
            countdownTimer: null,
            marketTimer: null,
            activeTrade: null,
            stats: { wins: 0, losses: 0, total: 0 }
        };

        let audioCtx = null;
        const API_URL = 'https://api.frankfurter.dev/v2/latest?base=USD&symbols=EUR';

        // ===== INIT LOAD SAVED STATS =====
        try {
            const saved = localStorage.getItem('neoBeatStats');
            if (saved) {
                botState.stats = JSON.parse(saved);
                updateStatsDisplay();
            }
        } catch (e) {}

        // ===== AUDIO SYSTEM =====
        function enableSound() {
            try {
                if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                if (audioCtx.state === 'suspended') audioCtx.resume();
                botState.soundEnabled = true;
                const btn = document.getElementById('btn-sound');
                btn.textContent = "SOUND: ON 🔊";
                btn.className = "px-3 py-2.5 rounded-xl font-bold bg-emerald-950 border border-emerald-700 text-emerald-400";
                playBeep(880, 0.1);
                log('AUDIO', 'Sound enabled successfully.');
            } catch (e) {}
        }

        function playBeep(freq = 600, duration = 0.1) {
            if (!botState.soundEnabled || !audioCtx) return;
            try {
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
                gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + duration);
                osc.connect(gain);
                gain.connect(audioCtx.destination);
                osc.start();
                osc.stop(audioCtx.currentTime + duration);
            } catch (e) {}
        }

        // ===== HELPER: LOGGING =====
        function log(type, message) {
            const container = document.getElementById('log-container');
            const count = document.getElementById('log-count');
            const time = new Date().toLocaleTimeString('th-TH', { hour12: false });
            const entry = document.createElement('div');
            entry.innerHTML = `<span class="text-zinc-500">[${time}]</span> <span class="text-emerald-400">[${type}]</span> ${message}`;
            container.appendChild(entry);
            container.scrollTop = container.scrollHeight;
            count.textContent = `Events: ${container.children.length}`;
            while (container.children.length > 50) container.removeChild(container.firstChild);
        }

        // ===== HELPER: UPDATE SIGNAL DISPLAY (รองรับ BUY / SELL ตาม UI เดิม) =====
        function updateSignalDisplay(signal, reasonText) {
            const signalEl = document.getElementById('txt-signal');
            const reasonEl = document.getElementById('txt-reason');
            const glow = document.getElementById('signal-glow');
            
            botState.signal = signal;
            signalEl.className = 'text-2xl md:text-3xl font-extrabold tracking-widest font-mono my-2';
            
            switch(signal) {
                case 'BUY':
                    signalEl.textContent = 'BUY';
                    signalEl.classList.add('text-emerald-400');
                    glow.style.backgroundColor = '#22c55e';
                    break;
                case 'SELL':
                    signalEl.textContent = 'SELL';
                    signalEl.classList.add('text-rose-400');
                    glow.style.backgroundColor = '#ef4444';
                    break;
                case 'HOLD':
                    signalEl.textContent = 'HOLD';
                    signalEl.classList.add('text-gray-400');
                    glow.style.backgroundColor = '#6b7280';
                    break;
                default:
                    signalEl.textContent = 'WAITING';
                    signalEl.classList.add('text-zinc-400');
                    glow.style.backgroundColor = '#71717a';
            }
            glow.style.opacity = '0.15';
            reasonEl.textContent = reasonText;
        }

        // ===== HELPER: UPDATE STATS DISPLAY =====
        function updateStatsDisplay() {
            document.getElementById('stat-wl').textContent = `${botState.stats.wins} / ${botState.stats.losses}`;
            document.getElementById('stat-total').textContent = botState.stats.total;
            const rate = botState.stats.total > 0 
                ? ((botState.stats.wins / botState.stats.total) * 100).toFixed(1) 
                : '0.0';
            document.getElementById('stat-rate').textContent = `${rate}%`;
            try {
                localStorage.setItem('neoBeatStats', JSON.stringify(botState.stats));
            } catch (e) {}
        }

        // ===== FETCH REAL MARKET DATA =====
        async function fetchMarketData() {
            if (!botState.isRunning) return;
            try {
                const response = await fetch(API_URL, { cache: 'no-cache' });
                if (!response.ok) throw new Error(`HTTP ${response.status}`);
                const data = await response.json();
                
                if (data && data.conversion_rates && data.conversion_rates.EUR) {
                    const eurUsd = parseFloat((1 / data.conversion_rates.EUR).toFixed(5));
                    
                    botState.previousPrice = botState.currentPrice;
                    botState.currentPrice = eurUsd;
                    
                    botState.priceHistory.push({ price: eurUsd, time: Date.now() });
                    if (botState.priceHistory.length > 20) botState.priceHistory.shift();

                    const priceEl = document.getElementById('txt-live-price');
                    priceEl.classList.remove('price-loading');
                    if (botState.previousPrice) {
                        priceEl.classList.remove('price-up', 'price-down');
                        priceEl.classList.add(eurUsd >= botState.previousPrice ? 'price-up' : 'price-down');
                    }
                    priceEl.textContent = eurUsd.toFixed(5);

                    log('MARKET', `EUR/USD Live Price: ${eurUsd.toFixed(5)}`);

                    // วิเคราะห์ราคาจริงเมื่อไม่อยู่ในช่วงนับถอยหลังหรือถือออเดอร์
                    if (!botState.countdownTimer && !botState.activeTrade) {
                        analyzeMarket();
                    }
                }
            } catch (err) {
                log('ERROR', `Failed to fetch: ${err.message}`);
            }
        }

        // ===== REAL MARKET ANALYSIS LOGIC =====
        function analyzeMarket() {
            if (botState.priceHistory.length < 3) {
                updateSignalDisplay('HOLD', 'Collecting price data...');
                return;
            }

            const history = botState.priceHistory;
            const current = botState.currentPrice;
            const prev = history[history.length - 2].price;
            const avg = history.reduce((sum, p) => sum + p.price, 0) / history.length;
            
            const trendUp = current > avg;
            const change = prev ? ((current - prev) / prev) * 100 : 0;

            log('AI', `Analyzing — Trend: ${trendUp ? 'UP' : 'DOWN'}, Change: ${change.toFixed(4)}%`);

            // เงื่อนไขตัดสินใจจากราคาจริง
            if (trendUp && change >= 0) {
                startCountdown('BUY', 'Upward momentum confirmed from real feed');
            } else if (!trendUp && change < 0) {
                startCountdown('SELL', 'Downward momentum confirmed from real feed');
            } else {
                updateSignalDisplay('HOLD', 'Market consolidating, waiting for setup...');
            }
        }

        // ===== COUNTDOWN SYSTEM =====
        function startCountdown(signal, reason) {
            if (botState.countdownTimer) return;

            botState.countdown = 10;
            document.getElementById('countdown-box').classList.remove('hidden');
            updateSignalDisplay('WAITING', `${reason} — Entry in 10s...`);
            log('SYSTEM', `${signal} signal setup detected — 10s countdown started`);
            playBeep(800, 0.15);

            botState.countdownTimer = setInterval(() => {
                botState.countdown--;
                const countEl = document.getElementById('txt-countdown');
                if (countEl) countEl.textContent = `${botState.countdown}s`;
                playBeep(600, 0.08);

                if (botState.countdown <= 0) {
                    clearInterval(botState.countdownTimer);
                    botState.countdownTimer = null;
                    document.getElementById('countdown-box').classList.add('hidden');
                    executeSimulatedTrade(signal);
                }
            }, 1000);
        }

        // ===== EXECUTE SIMULATED TRADE =====
        function executeSimulatedTrade(signal) {
            const entryPrice = botState.currentPrice;
            botState.activeTrade = { signal: signal, entryPrice: entryPrice };
            
            updateSignalDisplay(signal, `Active ${signal} trade locked at ${entryPrice.toFixed(5)}`);
            log('TRADE', `Executed simulated ${signal} at real price: ${entryPrice.toFixed(5)}`);
            playBeep(1200, 0.3);

            // ถือออเดอร์จำลอง 15 วินาทีเพื่อวัดผลเทียบกับราคาจริง
            setTimeout(() => {
                evaluateTradeOutcome();
            }, 15000);
        }

        // ===== EVALUATE OUTCOME =====
        function evaluateTradeOutcome() {
            if (!botState.activeTrade) return;
            const trade = botState.activeTrade;
            const exitPrice = botState.currentPrice;
            let isWin = false;

            if (trade.signal === 'BUY') {
                isWin = exitPrice >= trade.entryPrice;
            } else {
                isWin = exitPrice <= trade.entryPrice;
            }

            botState.stats.total++;
            if (isWin) {
                botState.stats.wins++;
                log('RESULT', `WIN! [${trade.signal}] Entry: ${trade.entryPrice.toFixed(5)} -> Exit: ${exitPrice.toFixed(5)}`);
                playBeep(1000, 0.2);
            } else {
                botState.stats.losses++;
                log('RESULT', `LOSS! [${trade.signal}] Entry: ${trade.entryPrice.toFixed(5)} -> Exit: ${exitPrice.toFixed(5)}`);
                playBeep(300, 0.3);
            }

            updateStatsDisplay();
            botState.activeTrade = null;
            updateSignalDisplay('WAITING', 'Background worker monitoring live market structure...');
        }

        // ===== TOGGLE BOT =====
        function toggleBot() {
            botState.isRunning = !botState.isRunning;
            const btn = document.getElementById('btn-toggle');
            const badge = document.getElementById('conn-badge');
            const pulse = document.getElementById('header-pulse');
            const statusEl = document.getElementById('txt-bot-status');

            if (botState.isRunning) {
                btn.textContent = "STOP BOT";
                btn.className = "px-3 py-2.5 rounded-xl font-bold bg-rose-600 hover:bg-rose-500 text-black transition-all";
                badge.textContent = "CONNECTED (LIVE)";
                badge.className = "px-2.5 py-1 rounded text-[10px] font-mono font-bold bg-emerald-950 border border-emerald-700 text-emerald-400";
                pulse.className = "w-2.5 h-2.5 rounded-full bg-emerald-500 animate-pulse";
                statusEl.textContent = "MONITORING REAL MARKET";
                statusEl.className = "text-xs md:text-sm font-bold text-emerald-400";
                
                log('SYSTEM', 'Autonomous live background worker initialized.');
                
                // ดึงข้อมูลทันทีเมื่อกด Start
                fetchMarketData();
                
                // ดึงราคาจริงต่อเนื่องทุกๆ 5 วินาที
                botState.marketTimer = setInterval(fetchMarketData, 5000);
            } else {
                btn.textContent = "START BOT";
                btn.className = "px-3 py-2.5 rounded-xl font-bold bg-emerald-600 hover:bg-emerald-500 text-black transition-all";
                badge.textContent = "DISCONNECTED";
                badge.className = "px-2.5 py-1 rounded text-[10px] font-mono font-bold bg-amber-950 border border-amber-700 text-amber-400";
                pulse.className = "w-2.5 h-2.5 rounded-full bg-zinc-600";
                statusEl.textContent = "DISCONNECTED";
                statusEl.className = "text-xs md:text-sm font-bold text-zinc-400";

                if (botState.marketTimer) clearInterval(botState.marketTimer);
                if (botState.countdownTimer) {
                    clearInterval(botState.countdownTimer);
                    botState.countdownTimer = null;
                }
                document.getElementById('countdown-box').classList.add('hidden');
                botState.activeTrade = null;
                
                log('SYSTEM', 'Background worker stopped.');
                updateSignalDisplay('WAITING', 'Press START BOT to begin...');
            }
        }

        // ===== RESET STATS =====
        function clearStats() {
            if (confirm("ต้องการรีเซ็ตสถิติทั้งหมดใช่หรือไม่?")) {
                botState.stats = { wins: 0, losses: 0, total: 0 };
                localStorage.removeItem('neoBeatStats');
                updateStatsDisplay();
                log('STATS', 'Statistics have been cleared.');
            }
        }
    </script>
</body>
</html>

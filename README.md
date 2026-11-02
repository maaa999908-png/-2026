<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>攻城卡 (Siege Cards) - Canvas 完整網頁版</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Inter Font -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;900&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', system-ui, -apple-system, sans-serif;
            user-select: none;
            background-color: #020617;
            color: #f8fafc;
            touch-action: manipulation;
        }
        /* Custom scrollbars */
        ::-webkit-scrollbar {
            width: 5px;
            height: 5px;
        }
        ::-webkit-scrollbar-track {
            background: #0f172a;
        }
        ::-webkit-scrollbar-thumb {
            background: #334155;
            border-radius: 3px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #475569;
        }
        .card-ratio {
            aspect-ratio: 63 / 87;
        }
    </style>
</head>
<body class="h-screen w-screen overflow-hidden flex flex-col bg-slate-950 text-slate-100">

    <!-- Top Navigation Header -->
    <header class="h-14 border-b border-slate-800 bg-slate-900/90 backdrop-blur px-3 md:px-4 flex items-center justify-between z-20 shrink-0">
        <div class="flex items-center space-x-2 md:space-x-3">
            <div class="w-8 h-8 rounded-lg bg-gradient-to-tr from-amber-600 to-amber-400 flex items-center justify-center text-slate-950 font-black shadow-md shadow-amber-500/20">
                <i class="fa-solid fa-chess-rook text-base"></i>
            </div>
            <h1 class="font-extrabold text-base md:text-lg tracking-wide bg-gradient-to-r from-amber-200 via-amber-400 to-amber-500 bg-clip-text text-transparent">
                攻城卡 <span class="text-[10px] md:text-xs font-normal text-slate-400 ml-1">v4.0 英雄降臨版</span>
            </h1>
        </div>

        <!-- Status Badges & Controls -->
        <div class="flex items-center space-x-2 md:space-x-3 text-xs">
            <!-- Rules & Spells Button -->
            <button onclick="openRulesModal()" class="px-3 py-1.5 rounded-lg bg-gradient-to-r from-indigo-600 to-purple-600 hover:from-indigo-500 hover:to-purple-500 border border-indigo-400 text-white text-xs font-bold transition flex items-center space-x-1.5 shadow-lg shadow-indigo-500/30">
                <i class="fa-solid fa-book-open"></i>
                <span class="hidden sm:inline">規則與法術&英雄圖鑑</span>
                <span class="sm:hidden">規則&圖鑑</span>
            </button>

            <!-- Mode Selector Toggle -->
            <div class="flex items-center space-x-1 bg-slate-950 p-1 rounded-xl border border-slate-800">
                <button id="mode-2p-btn" onclick="setGameMode('2P')" class="px-2 py-1 rounded-lg text-[11px] font-bold transition flex items-center gap-1 text-slate-400 hover:text-slate-200">
                    <i class="fa-solid fa-users text-[10px]"></i>
                    <span class="hidden sm:inline">雙人對戰</span>
                </button>
                <button id="mode-ai-btn" onclick="setGameMode('VS_AI')" class="px-2 py-1 rounded-lg text-[11px] font-bold transition flex items-center gap-1 bg-amber-500 text-slate-950 shadow-md shadow-amber-500/20">
                    <i class="fa-solid fa-robot text-[10px]"></i>
                    <span>人機 (AI)</span>
                </button>
            </div>

            <!-- Layout Mode Indicator Toggle -->
            <button id="device-layout-btn" onclick="toggleDeviceLayout()" class="hidden md:flex px-2.5 py-1 rounded-lg bg-slate-800 hover:bg-slate-700 border border-slate-700 text-slate-300 transition items-center gap-1 text-xs">
                <i id="device-icon" class="fa-solid fa-desktop text-amber-400"></i>
                <span id="device-text">電腦模式</span>
            </button>

            <!-- Sound Toggle -->
            <button id="sound-btn" onclick="toggleSound()" class="p-2 rounded-lg bg-slate-800 hover:bg-slate-700 border border-slate-700 text-slate-300 hover:text-white transition">
                <i id="sound-icon" class="fa-solid fa-volume-high"></i>
            </button>

            <!-- Restart Button -->
            <button onclick="confirmRestart()" class="px-2.5 py-1.5 rounded-lg bg-red-950/60 hover:bg-red-900/80 border border-red-800/60 text-red-300 text-xs font-semibold transition flex items-center space-x-1">
                <i class="fa-solid fa-rotate-left"></i>
                <span class="hidden sm:inline">重來</span>
            </button>
        </div>
    </header>

    <!-- Main Responsive Game Container -->
    <main id="main-container" class="flex-1 flex flex-col md:flex-row overflow-hidden relative">

        <!-- Left/Top: Canvas Board & Phase Banner Area -->
        <div class="flex-1 flex flex-col items-center justify-between p-2 md:p-3 relative bg-slate-950/80 h-full overflow-hidden">
            
            <!-- Phase Information Banner -->
            <div id="phase-banner" class="w-full max-w-md px-3 py-1.5 md:py-2 bg-slate-900/90 border border-slate-800 rounded-xl flex items-center justify-between shadow-lg backdrop-blur shrink-0 mb-1">
                <div class="flex items-center space-x-2">
                    <span id="turn-badge" class="px-2 py-0.5 md:px-2.5 md:py-1 rounded-md text-[11px] md:text-xs font-bold uppercase tracking-wider bg-amber-600 text-white shadow">
                        P1 佈陣階段
                    </span>
                    <span id="weather-badge" class="px-2 py-0.5 md:px-2.5 md:py-1 rounded-md text-[11px] md:text-xs font-bold text-white shadow bg-emerald-600">
                        天氣: 春天
                    </span>
                    <span id="turn-phase" class="text-[11px] md:text-xs text-slate-300 font-medium">請將卡牌佈置至己方前兩排</span>
                </div>
                <div id="setup-count-wrap" class="text-[11px] md:text-xs text-amber-400 font-semibold">
                    已佈置: <span id="setup-count" class="text-xs md:text-sm font-bold text-amber-300">0</span> / 6
                </div>
            </div>

            <!-- Canvas Adaptive Outer Wrapper -->
            <div class="relative flex-1 w-full max-w-md flex items-center justify-center min-h-0 overflow-hidden my-auto">
                <canvas id="gameCanvas" class="rounded-2xl shadow-2xl border-2 border-slate-800 bg-slate-900 cursor-pointer block"></canvas>

                <!-- Floating Toast Notification -->
                <div id="toast-message" class="absolute top-4 left-1/2 -translate-x-1/2 bg-slate-900/95 border border-amber-500/60 text-amber-200 px-4 py-2 rounded-xl text-xs font-bold shadow-2xl transition-all duration-300 opacity-0 pointer-events-none z-30 flex items-center gap-2">
                    <i class="fa-solid fa-circle-info text-amber-400"></i>
                    <span id="toast-text">提示訊息</span>
                </div>
            </div>

            <!-- Bottom Floating Action Bar for Selections -->
            <div id="action-bar" class="w-full max-w-md mt-1 px-3 py-1.5 bg-slate-900/90 border border-slate-800 rounded-xl flex items-center justify-between text-xs transition-all duration-200 opacity-0 pointer-events-none transform translate-y-2 shrink-0">
                <div class="flex items-center space-x-2 text-slate-300">
                    <i class="fa-solid fa-crosshairs text-amber-400"></i>
                    <span id="action-desc" class="text-[11px] truncate">請點擊目標位置執行行動</span>
                </div>
                <button onclick="cancelSelection()" class="px-2.5 py-1 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-300 text-[11px] font-semibold border border-slate-700 transition shrink-0">
                    取消
                </button>
            </div>
        </div>

        <!-- Right/Bottom: Deck Pool, Hands, Spells & History Panel -->
        <div id="side-panel" class="w-full md:w-80 lg:w-96 border-t md:border-t-0 md:border-l border-slate-800 bg-slate-900/95 flex flex-col h-1/2 md:h-full shrink-0 shadow-2xl z-10 overflow-hidden">
            
            <!-- Setup & Battle Hand Card Area -->
            <div id="setup-pool-panel" class="p-3 md:p-4 border-b border-slate-800 flex flex-col bg-slate-900/50 shrink-0">
                <div class="flex items-center justify-between mb-1.5">
                    <div class="flex items-center space-x-2">
                        <span id="setup-pool-title" class="text-xs md:text-sm font-bold text-slate-100 flex items-center gap-1.5">
                            <i class="fa-solid fa-shield-halved text-amber-400"></i> P1 可用卡牌
                        </span>
                    </div>

                    <!-- Card Type Switcher Tabs (Battle Phase) -->
                    <div id="battle-card-tabs" class="hidden flex items-center bg-slate-950 p-1 rounded-lg border border-slate-800 text-[11px]">
                        <button id="tab-hand-btn" onclick="switchCardTab('hand')" class="px-2.5 py-0.5 rounded font-bold transition bg-amber-500 text-slate-950 shadow">
                            手牌
                        </button>
                        <button id="tab-spell-btn" onclick="switchCardTab('spell')" class="px-2.5 py-0.5 rounded font-bold transition text-slate-400 hover:text-slate-200">
                            法術 (<span id="spell-tab-count">4</span>)
                        </button>
                    </div>

                    <span id="setup-pool-count" class="text-[10px] md:text-xs font-semibold px-2 py-0.5 rounded-full bg-slate-800 text-amber-400 border border-slate-700">
                        剩餘 15 張
                    </span>
                </div>

                <p id="panel-hint" class="text-[10px] md:text-[11px] text-slate-400 mb-2 leading-tight">💡 點擊下方放大卡牌，再點擊左側棋盤進行佈置：</p>

                <!-- Dynamic HTML Cards Grid -->
                <div id="setup-card-grid" class="grid grid-cols-5 gap-1.5 max-h-40 md:max-h-48 overflow-y-auto p-1 bg-slate-950/60 rounded-xl border border-slate-800">
                    <!-- Dynamic HTML Card Buttons -->
                </div>

                <!-- Setup Control Buttons -->
                <div id="setup-controls" class="grid grid-cols-3 gap-2 mt-2 hidden">
                    <button onclick="autoSetupCurrentPlayer()" class="py-1.5 px-2 bg-slate-800 hover:bg-slate-700 text-slate-200 rounded-lg text-xs font-bold border border-slate-700 transition flex items-center justify-center space-x-1">
                        <i class="fa-solid fa-wand-magic-sparkles text-amber-400"></i>
                        <span>隨機</span>
                    </button>
                    <button onclick="clearCurrentPlayerSetup()" class="py-1.5 px-2 bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-lg text-xs font-bold border border-slate-700 transition flex items-center justify-center space-x-1">
                        <i class="fa-solid fa-trash-can text-red-400"></i>
                        <span>重置</span>
                    </button>
                    <button onclick="confirmCurrentPlayerSetup()" class="py-1.5 px-2 bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-500 hover:to-teal-500 text-white rounded-lg text-xs font-bold shadow-lg shadow-emerald-900/30 transition flex items-center justify-center space-x-1">
                        <i class="fa-solid fa-check"></i>
                        <span>確認</span>
                    </button>
                </div>
            </div>

            <!-- Player Status Summary Cards -->
            <div class="p-2.5 border-b border-slate-800 grid grid-cols-2 gap-2 text-xs bg-slate-950/40 shrink-0">
                <!-- P1 Info -->
                <div class="p-2 rounded-xl bg-blue-950/30 border border-blue-800/40 flex flex-col space-y-0.5">
                    <div class="flex items-center justify-between text-blue-300 font-bold">
                        <span>P1 (藍方)</span>
                        <i class="fa-solid fa-chess-king"></i>
                    </div>
                    <div class="flex justify-between text-[10px] md:text-[11px] text-slate-400">
                        <span>手牌: <b id="p1-hand-count" class="text-slate-200">0</b></span>
                        <span>法術: <b id="p1-spell-count" class="text-amber-400">4</b></span>
                        <span>墓地: <b id="p1-grave-count" class="text-slate-200">0</b></span>
                    </div>
                </div>

                <!-- P2 Info -->
                <div class="p-2 rounded-xl bg-red-950/30 border border-red-800/40 flex flex-col space-y-0.5">
                    <div class="flex items-center justify-between text-rose-300 font-bold">
                        <span>P2 (紅方)</span>
                        <i class="fa-solid fa-chess-king"></i>
                    </div>
                    <div class="flex justify-between text-[10px] md:text-[11px] text-slate-400">
                        <span>手牌: <b id="p2-hand-count" class="text-slate-200">0</b></span>
                        <span>法術: <b id="p2-spell-count" class="text-amber-400">4</b></span>
                        <span>墓地: <b id="p2-grave-count" class="text-slate-200">0</b></span>
                    </div>
                </div>
            </div>

            <!-- Battle History Log Feed -->
            <div class="flex-1 flex flex-col p-2.5 overflow-hidden min-h-0">
                <div class="text-[11px] font-bold text-slate-400 mb-1 flex items-center justify-between">
                    <span class="flex items-center space-x-1">
                        <i class="fa-solid fa-scroll text-amber-400"></i>
                        <span>戰鬥歷史日誌</span>
                    </span>
                    <button onclick="clearLog()" class="text-[10px] text-slate-500 hover:text-slate-300 transition">清空</button>
                </div>
                <div id="battle-log" class="flex-1 overflow-y-auto space-y-1.5 pr-1 text-[11px] font-mono leading-snug">
                    <!-- Dynamic Log Entries -->
                </div>
            </div>
        </div>
    </main>

    <!-- Victory Modal -->
    <div id="victory-modal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md flex items-center justify-center z-50 opacity-0 pointer-events-none transition-all duration-300">
        <div class="bg-slate-900 border-2 border-amber-500/60 p-6 rounded-2xl max-w-sm w-full text-center shadow-2xl transform scale-90 transition-all duration-300" id="victory-box">
            <div class="w-16 h-16 rounded-full bg-gradient-to-tr from-amber-500 to-yellow-300 text-slate-950 flex items-center justify-center mx-auto mb-4 text-2xl font-black shadow-lg shadow-amber-500/30">
                <i class="fa-solid fa-trophy"></i>
            </div>
            <h2 id="winner-title" class="text-2xl font-black text-amber-300 mb-1">玩家 1 獲勝！</h2>
            <p id="winner-desc" class="text-xs text-slate-300 mb-6">成功擊潰對手國王，奪得最終勝利！</p>
            <button onclick="initGame()" class="w-full py-3 bg-gradient-to-r from-amber-500 to-amber-600 hover:from-amber-400 hover:to-amber-500 text-slate-950 font-bold rounded-xl shadow-lg transition">
                再玩一局
            </button>
        </div>
    </div>

    <!-- Hero Selection Modal -->
    <div id="hero-modal" class="fixed inset-0 bg-slate-950/90 backdrop-blur-md flex items-center justify-center z-[70] opacity-0 pointer-events-none transition-all duration-300 p-4">
        <div class="bg-slate-900 border-2 border-amber-500/60 p-5 md:p-6 rounded-2xl max-w-7xl w-full text-center shadow-2xl transition-all duration-300" id="hero-box">
            <h2 id="hero-modal-title" class="text-xl md:text-2xl font-black text-amber-300 mb-2">P1 選擇英雄</h2>
            <p class="text-xs text-slate-300 mb-4">選擇一名英雄與本命法術（開局會外加 4 張隨機法術）</p>
            <div class="grid grid-cols-2 lg:grid-cols-7 gap-3 md:gap-4">
                <button id="hero-btn-true_king" onclick="selectHero('true_king')" class="p-3 md:p-4 bg-slate-800 hover:bg-slate-700 border-2 border-slate-600 hover:border-amber-500 rounded-xl flex flex-col items-center transition text-left h-full">
                    <div class="text-3xl md:text-4xl mb-2">👑</div>
                    <div class="font-bold text-amber-400 mb-1 text-sm md:text-base">真王</div>
                    <div class="text-[10px] md:text-xs text-slate-300 leading-tight">
                        替換國王(需墓地>6)。<br>
                        未覺醒：僅能被 0-6 數字牌擊殺。<br>
                        覺醒(御駕親征)：自由移動、致命弱點縮小至僅能被 0-3 數字牌擊殺。
                    </div>
                </button>
                <button id="hero-btn-deity" onclick="selectHero('deity')" class="p-3 md:p-4 bg-slate-800 hover:bg-slate-700 border-2 border-slate-600 hover:border-blue-500 rounded-xl flex flex-col items-center transition text-left h-full">
                    <div class="text-3xl md:text-4xl mb-2">⚡</div>
                    <div class="font-bold text-blue-400 mb-1 text-sm md:text-base">天神</div>
                    <div class="text-[10px] md:text-xs text-slate-300 leading-tight">
                        登場(一次性)：向全場未知牌(除12)劈出雷電。<br>
                        擊殺數字牌獲得雷電。<br>
                        覺醒(雷電)：抹殺除國王外的全場卡牌(含自己)。
                    </div>
                </button>
                <button id="hero-btn-angry_one" onclick="selectHero('angry_one')" class="p-3 md:p-4 bg-slate-800 hover:bg-slate-700 border-2 border-slate-600 hover:border-red-500 rounded-xl flex flex-col items-center transition text-left h-full">
                    <div class="text-3xl md:text-4xl mb-2">😡</div>
                    <div class="font-bold text-red-500 mb-1 text-sm md:text-base">憤怒者</div>
                    <div class="text-[10px] md:text-xs text-slate-300 leading-tight">
                        登場：己方攻擊力永久 +1。<br>
                        覺醒(狂暴)：免疫 0-3 攻擊。己方攻擊力再 +2 (最高11，0不變)。
                    </div>
                </button>
                <button id="hero-btn-mage" onclick="selectHero('mage')" class="p-3 md:p-4 bg-slate-800 hover:bg-slate-700 border-2 border-slate-600 hover:border-purple-500 rounded-xl flex flex-col items-center transition text-left h-full">
                    <div class="text-3xl md:text-4xl mb-2">🧙</div>
                    <div class="font-bold text-purple-400 mb-1 text-sm md:text-base">法師</div>
                    <div class="text-[10px] md:text-xs text-slate-300 leading-tight">
                        被動：每移動/攻擊 3 步獲隨機法術。<br>
                        覺醒(抽牌)：回合初自動湧現臨時法術，須立即決定是否施放。
                    </div>
                </button>
                <button id="hero-btn-centurion" onclick="selectHero('centurion')" class="p-3 md:p-4 bg-slate-800 hover:bg-slate-700 border-2 border-slate-600 hover:border-emerald-500 rounded-xl flex flex-col items-center transition text-left h-full">
                    <div class="text-3xl md:text-4xl mb-2">🛡️</div>
                    <div class="font-bold text-emerald-400 mb-1 text-sm md:text-base">百夫長</div>
                    <div class="text-[10px] md:text-xs text-slate-300 leading-tight">
                        登場：代替國王，獲雙倍數字牌，無法術。<br>
                        覺醒(9,10,11死亡)：己方攻防視為上一張陣亡牌數值。
                    </div>
                </button>
                <button id="hero-btn-sea_king" onclick="selectHero('sea_king')" class="p-3 md:p-4 bg-slate-800 hover:bg-slate-700 border-2 border-slate-600 hover:border-cyan-500 rounded-xl flex flex-col items-center transition text-left h-full">
                    <div class="text-3xl md:text-4xl mb-2">🌊</div>
                    <div class="font-bold text-cyan-400 mb-1 text-sm md:text-base">海王</div>
                    <div class="text-[10px] md:text-xs text-slate-300 leading-tight">
                        被動：己方單位免疫控制且無視河流傷害。<br>
                        覺醒(河流)：每 4 回合自動生成河流，退潮時抹殺敵方前排所有 0-6。
                    </div>
                </button>
                <button id="hero-btn-jieju" onclick="selectHero('jieju')" class="p-3 md:p-4 bg-slate-800 hover:bg-slate-700 border-2 border-slate-600 hover:border-indigo-500 rounded-xl flex flex-col items-center transition text-left h-full">
                    <div class="text-3xl md:text-4xl mb-2">🌉</div>
                    <div class="font-bold text-indigo-400 mb-1 text-sm md:text-base">傑局</div>
                    <div class="text-[10px] md:text-xs text-slate-300 leading-tight">
                        登場：己方陣前永久生成淚橋，死後消失。<br>
                        覺醒(淚橋)：每 3 回合獲得一次開關淚橋的機會(不可儲存)。
                    </div>
                </button>
            </div>
        </div>
    </div>

    <!-- Confirm Restart Modal -->
    <div id="confirm-modal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md flex items-center justify-center z-50 opacity-0 pointer-events-none transition-all duration-300">
        <div class="bg-slate-900 border border-slate-700 p-5 rounded-2xl max-w-xs w-full text-center shadow-2xl">
            <i class="fa-solid fa-triangle-exclamation text-amber-400 text-3xl mb-2"></i>
            <h3 class="text-base font-bold text-slate-100 mb-1">確定要重新開始對局嗎？</h3>
            <p class="text-xs text-slate-400 mb-4">當前對局進度將無法恢復。</p>
            <div class="grid grid-cols-2 gap-2">
                <button onclick="closeConfirmModal()" class="py-2 bg-slate-800 hover:bg-slate-700 text-slate-300 text-xs font-bold rounded-xl border border-slate-700">取消</button>
                <button onclick="initGame(); closeConfirmModal();" class="py-2 bg-red-600 hover:bg-red-500 text-white text-xs font-bold rounded-xl shadow">確認重來</button>
            </div>
        </div>
    </div>

    <!-- First Move Choice Modal (VS_AI only) -->
    <div id="first-move-modal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md flex items-center justify-center z-[60] opacity-0 pointer-events-none transition-all duration-300">
        <div class="bg-slate-900 border border-amber-500/60 p-6 rounded-2xl max-w-sm w-full text-center shadow-2xl transform scale-90 transition-all duration-300" id="first-move-box">
            <i class="fa-solid fa-hand-point-up text-amber-400 text-4xl mb-4"></i>
            <h2 class="text-xl font-black text-slate-100 mb-2">請選擇先手玩家</h2>
            <p class="text-xs text-slate-400 mb-6">決定戰鬥階段由誰率先行動（這將影響您的佈陣策略）。</p>
            <div class="grid grid-cols-2 gap-3">
                <button onclick="setFirstMove(1)" class="py-3 bg-gradient-to-r from-blue-600 to-blue-500 hover:from-blue-500 hover:to-blue-400 text-white font-bold rounded-xl shadow-lg transition flex flex-col items-center justify-center">
                    <span class="text-sm">我先手</span>
                    <span class="text-[10px] text-blue-200">(P1 藍方)</span>
                </button>
                <button onclick="setFirstMove(2)" class="py-3 bg-gradient-to-r from-red-600 to-red-500 hover:from-red-500 hover:to-red-400 text-white font-bold rounded-xl shadow-lg transition flex flex-col items-center justify-center">
                    <span class="text-sm">AI 先手</span>
                    <span class="text-[10px] text-red-200">(P2 紅方)</span>
                </button>
            </div>
        </div>
    </div>

    <!-- Pass Device Modal (Strictly Hides Board in 2P Mode) -->
    <div id="pass-device-modal" class="fixed inset-0 bg-slate-950 flex flex-col items-center justify-center z-[100] hidden">
        <i class="fa-solid fa-people-arrows text-amber-400 text-6xl mb-6"></i>
        <h2 class="text-2xl font-black text-slate-100 mb-2">請將設備交給下一位玩家</h2>
        <p id="next-player-text" class="text-amber-400 font-bold text-lg mb-8">輪到 P2 的回合</p>
        <button onclick="confirmPassDevice()" class="px-8 py-3 bg-gradient-to-r from-amber-500 to-amber-600 hover:from-amber-400 hover:to-amber-500 text-slate-950 text-lg font-bold rounded-xl shadow-lg transition">
            我是 <span id="ready-player-label">P2</span>，準備開始
        </button>
    </div>

    <!-- Rules & Spells Dictionary Modal -->
    <div id="rules-modal" class="fixed inset-0 bg-slate-950/85 backdrop-blur-md flex items-center justify-center z-[110] opacity-0 pointer-events-none transition-all duration-300 p-3 md:p-6">
        <div class="bg-slate-900 border border-indigo-500/50 rounded-2xl w-full max-w-3xl max-h-full flex flex-col shadow-2xl transform scale-95 transition-all duration-300" id="rules-modal-box">
            <!-- Header -->
            <div class="flex items-center justify-between border-b border-slate-800 p-4 shrink-0">
                <h3 class="text-lg md:text-xl font-extrabold text-indigo-300 flex items-center gap-2">
                    <i class="fa-solid fa-book-journal-whills"></i> 遊戲規則、法術與英雄圖鑑
                </h3>
                <button onclick="closeRulesModal()" class="text-slate-400 hover:text-white text-xl p-1 transition">
                    <i class="fa-solid fa-xmark"></i>
                </button>
            </div>
            
            <!-- Modal Tabs (New) -->
            <div class="flex p-2 md:px-4 md:py-3 bg-slate-950/50 border-b border-slate-800 gap-2 shrink-0 overflow-x-auto custom-scrollbar">
                <button id="modal-tab-rules" onclick="switchRulesTab('rules')" class="flex-1 py-2 px-1 rounded-lg bg-indigo-600 text-white text-xs md:text-sm font-bold transition shadow-md shadow-indigo-900/50 whitespace-nowrap">
                    <i class="fa-solid fa-scroll"></i> 核心機制
                </button>
                <button id="modal-tab-spells" onclick="switchRulesTab('spells')" class="flex-1 py-2 px-1 rounded-lg bg-slate-800 text-slate-400 hover:bg-slate-700 hover:text-slate-200 text-xs md:text-sm font-bold transition border border-slate-700 whitespace-nowrap">
                    <i class="fa-solid fa-wand-magic-sparkles"></i> 法術圖鑑
                </button>
                <button id="modal-tab-heroes" onclick="switchRulesTab('heroes')" class="flex-1 py-2 px-1 rounded-lg bg-slate-800 text-slate-400 hover:bg-slate-700 hover:text-slate-200 text-xs md:text-sm font-bold transition border border-slate-700 whitespace-nowrap">
                    <i class="fa-solid fa-mask"></i> 英雄圖鑑
                </button>
            </div>

            <!-- Content Scroll Area -->
            <div class="overflow-y-auto p-4 md:p-6 text-sm text-slate-300 custom-scrollbar">
                
                <!-- Game Rules Section -->
                <div id="modal-content-rules" class="block space-y-6">
                    <section>
                        <h4 class="text-amber-400 font-bold mb-3 text-base flex items-center gap-2">
                            <i class="fa-solid fa-scale-balanced text-amber-500"></i> 核心規則
                        </h4>
                        <ul class="list-disc pl-5 space-y-2 leading-relaxed text-xs md:text-sm bg-slate-950/50 p-4 rounded-xl border border-slate-800">
                            <li><b class="text-slate-200">佈陣階段：</b>雙方在己方前兩排秘密佈置 6 張手牌，國王(12)固定於王座不可更換。</li>
                            <li><b class="text-slate-200">英雄機制：</b>開局選擇英雄進入手牌。戰鬥階段需耗費行動點部署。英雄免疫敵方控制/直接法術傷害。</li>
                            <li><b class="text-slate-200">大小比較：</b>戰鬥時數字大者勝，<span class="text-amber-300 font-bold">數值相同時由主動攻擊方獲得勝利</span>。<span class="text-amber-300 font-bold">特殊規則：0 與 11 交鋒、0 碰 0 雙方同歸於盡</span>。</li>
                            <li><b class="text-slate-200">暗牌機制：</b>普通移動保持暗牌。僅在「發起攻擊」或「被攻擊」時，雙方卡牌才會翻開。</li>
                            <li><b class="text-slate-200">無敵判定：</b>無敵期間無法被擊殺。若攻擊大於自身限制的暗牌，雙方均不死並保持原位。</li>
                            <li><b class="text-slate-200">天氣系統：</b>開局隨機天氣。雷雨(連環閃電/火削弱)、烈夏(火焰留火團/冰凍縮短)、冬天(隨機冰凍/火削弱/火不解冰)、春天(無影響)。</li>
                            <li><b class="text-slate-200">勝利條件：</b>成功擊殺敵方國王，或真王陣亡。殲滅敵方場上及手牌所有戰力亦可獲勝。</li>
                        </ul>
                    </section>
                </div>

                <!-- Spells Glossary Section -->
                <div id="modal-content-spells" class="hidden space-y-6">
                    <section>
                        <h4 class="text-indigo-400 font-bold mb-3 text-base flex items-center gap-2">
                            <i class="fa-solid fa-book-sparkles text-indigo-500"></i> 全法術技能圖鑑 (共 24 種)
                        </h4>
                        <p class="text-[11px] text-slate-400 mb-3">開局每位玩家會獲得自己所選英雄的「本命法術」，外加完全隨機獲得 4 張法術（可能獲得重複法術）。</p>
                        <div id="rules-spell-list" class="flex flex-col gap-3">
                            <!-- Spells dynamically injected by JS -->
                        </div>
                    </section>
                </div>

                <!-- Heroes Glossary Section -->
                <div id="modal-content-heroes" class="hidden space-y-6">
                    <section>
                        <h4 class="text-amber-400 font-bold mb-3 text-base flex items-center gap-2">
                            <i class="fa-solid fa-mask text-amber-500"></i> 英雄圖鑑 (共 6 位)
                        </h4>
                        <p class="text-[11px] text-slate-400 mb-3">英雄免疫敵方直接法術指定（開牌除外）。主動發起攻擊時攻擊力視為 12，被動防禦時防禦力視為 0。</p>
                        <div id="rules-hero-list" class="flex flex-col gap-3">
                            <!-- Heroes dynamically injected by JS -->
                        </div>
                    </section>
                </div>

            </div>
        </div>
    </div>

    <!-- Mage Spell Modal -->
    <div id="mage-spell-modal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md flex items-center justify-center z-[80] opacity-0 pointer-events-none transition-all duration-300">
        <div class="bg-slate-900 border border-purple-500/60 p-6 rounded-2xl max-w-sm w-full text-center shadow-2xl transform scale-90 transition-all duration-300" id="mage-modal-box">
            <i id="mage-spell-icon" class="fa-solid fa-wand-magic-sparkles text-4xl text-purple-400 mb-3"></i>
            <h2 class="text-xl font-black text-slate-100 mb-1">無盡法力湧現！</h2>
            <h3 id="mage-spell-name" class="text-lg font-bold text-amber-300 mb-2">法術名稱</h3>
            <p id="mage-spell-desc" class="text-xs text-slate-300 mb-6">法術描述...</p>
            <div class="grid grid-cols-2 gap-3">
                <button onclick="declineMageSpell()" class="py-2.5 bg-slate-800 hover:bg-slate-700 border border-slate-700 text-slate-300 font-bold rounded-xl transition">
                    放棄施放
                </button>
                <button onclick="acceptMageSpell()" class="py-2.5 bg-purple-600 hover:bg-purple-500 text-white font-bold rounded-xl shadow-lg transition">
                    立即施放
                </button>
            </div>
        </div>
    </div>

    <!-- Jieju Spell Modal -->
    <div id="jieju-spell-modal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md flex items-center justify-center z-[80] opacity-0 pointer-events-none transition-all duration-300">
        <div class="bg-slate-900 border border-indigo-500/60 p-6 rounded-2xl max-w-sm w-full text-center shadow-2xl transform scale-90 transition-all duration-300" id="jieju-modal-box">
            <i class="fa-solid fa-bridge text-4xl text-indigo-400 mb-3"></i>
            <h2 class="text-xl font-black text-slate-100 mb-1">淚橋控制權就緒</h2>
            <h3 class="text-lg font-bold text-amber-300 mb-2">控制淚橋</h3>
            <p class="text-xs text-slate-300 mb-6">您獲得了 1 次專屬淚橋的開關機會，是否要立即執行操作？（也可先收起，於本回合內任意時機施放）</p>
            <div class="grid grid-cols-2 gap-3">
                <button onclick="storeJiejuSpell()" class="py-2.5 bg-slate-800 hover:bg-slate-700 border border-slate-700 text-slate-300 font-bold rounded-xl transition">
                    稍後再用
                </button>
                <button onclick="acceptJiejuSpell()" class="py-2.5 bg-indigo-600 hover:bg-indigo-500 text-white font-bold rounded-xl shadow-lg transition">
                    立即執行
                </button>
            </div>
        </div>
    </div>

    <!-- JavaScript Game Logic Engine -->
    <script>
        /* --- Web Audio API Synthesizer --- */
        class SoundEngine {
            constructor() {
                this.ctx = null;
                this.muted = false;
            }

            init() {
                if (!this.ctx) {
                    const AudioCtx = window.AudioContext || window.webkitAudioContext;
                    if (AudioCtx) this.ctx = new AudioCtx();
                }
                if (this.ctx && this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }

            playBeep(freq, duration, type = 'sine', volume = 0.1) {
                if (this.muted) return;
                this.init();
                if (!this.ctx) return;

                try {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = type;
                    osc.frequency.setValueAtTime(freq, this.ctx.currentTime);
                    gain.gain.setValueAtTime(volume, this.ctx.currentTime);
                    gain.gain.exponentialRampToValueAtTime(0.0001, this.ctx.currentTime + duration);
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start();
                    osc.stop(this.ctx.currentTime + duration);
                } catch(e) {}
            }

            playSelect() { this.playBeep(520, 0.08, 'sine', 0.1); }
            playMove() { 
                this.playBeep(300, 0.06, 'triangle', 0.12);
                setTimeout(() => this.playBeep(450, 0.08, 'sine', 0.1), 50);
            }
            playAttack() {
                this.playBeep(180, 0.15, 'sawtooth', 0.2);
                setTimeout(() => this.playBeep(120, 0.2, 'square', 0.15), 60);
            }
            playSpell() {
                this.playBeep(600, 0.1, 'sine', 0.15);
                setTimeout(() => this.playBeep(850, 0.15, 'sine', 0.2), 80);
                setTimeout(() => this.playBeep(1100, 0.2, 'triangle', 0.15), 160);
            }
            playVictory() {
                const notes = [440, 554, 659, 880];
                notes.forEach((n, i) => {
                    setTimeout(() => this.playBeep(n, 0.25, 'triangle', 0.15), i * 120);
                });
            }
            playError() { this.playBeep(150, 0.2, 'sawtooth', 0.15); }
        }

        const soundEngine = new SoundEngine();

        function toggleSound() {
            soundEngine.muted = !soundEngine.muted;
            const icon = document.getElementById('sound-icon');
            if (icon) {
                icon.className = soundEngine.muted ? 'fa-solid fa-volume-xmark text-red-400' : 'fa-solid fa-volume-high';
            }
        }

        /* --- Complete 17 Spells Definition --- */
        const SPELL_DEFINITIONS = {
            thunder: { id: 'thunder', name: '雷電', icon: 'fa-bolt', desc: '指定消滅一張未知狀態的敵方牌（或用於天神覺醒）。' },
            fire: { id: 'fire', name: '火焰', icon: 'fa-fire', desc: '指定消滅一張已知狀態且數字 ≤ 9 的敵方牌（或解除冰凍）。' },
            freeze: { id: 'freeze', name: '冰凍', icon: 'fa-snowflake', desc: '指定一張敵方牌使之無法移動（可被火焰解除）。' },
            guandao: { id: 'guandao', name: '青龍偃月刀', icon: 'fa-dragon', desc: '指定一張牌，本回合擊殺單位後可再移動一步（不可攻王）。' },
            mimic: { id: 'mimic', name: '模仿', icon: 'fa-mask', desc: '複製一個本局遊戲中已使用過的法術效果。' },
            berserk: { id: 'berserk', name: '狂暴', icon: 'fa-hand-fist', desc: '使一張數字 ≤ 8 的牌數字 +3（持續 3 回合），未知牌揭露。' },
            revive: { id: 'revive', name: '復活', icon: 'fa-ankh', desc: '從墓地選擇一張 0 至 12 的牌放回戰場（包含國王），復活後已知。' },
            hat: { id: 'hat', name: '蔡只因帽', icon: 'fa-hat-wizard', desc: '使一張數字 ≤ 6 的友方牌獲得永久無敵，代價是永久不能攻擊國王。' },
            scythe: { id: 'scythe', name: '死神鐮刀', icon: 'fa-skull', desc: '將一張非國王敵牌強制拽至己方陣地，若有己方牌則觸發戰鬥。' },
            expedition: { id: 'expedition', name: '御駕親征', icon: 'fa-crown', desc: '己方國王永久解除移動限制，獲得 1 次免死護盾（或用於真王覺醒）。' },
            river: { id: 'river', name: '河流', icon: 'fa-water', desc: '在雙方前線生成持續 3 回合河流，強渡即淹死（冰凍可解）。' },
            drawSpells: { id: 'drawSpells', name: '抽牌', icon: 'fa-clone', desc: '己方累計死亡 ≥4 張牌時可用，抽 3 張新法術牌。' },
            reveal: { id: 'reveal', name: '開牌', icon: 'fa-eye', desc: '選擇一名玩家，將其場上所有未知的牌變為已知狀態。' },
            invincible3: { id: 'invincible3', name: '無敵', icon: 'fa-shield-halved', desc: '選一張牌3回合無敵（第1回合全攻，第2回合攻≤8，第3回合攻≤4，不可攻王）。' },
            stealth4: { id: 'stealth4', name: '隱謐', icon: 'fa-user-ninja', desc: '己方普通單位4回合進入未知狀態，攻擊/被攻擊不翻面。' },
            trap: { id: 'trap', name: '陷阱', icon: 'fa-radiation', desc: '在非第三排空位放置可見陷阱，踩中即死（一次性）。' },
            tunnel: { id: 'tunnel', name: '地洞', icon: 'fa-archway', desc: '在對方1、2排空位放置地洞，友軍可無視距離移動至該格。' },
            puppet: { id: 'puppet', name: '傀儡', icon: 'fa-ghost', desc: '指定敵方一張 ≤ 8 數字牌，獲得其控制權 3 回合（不可攻王）。' },
            storm: { id: 'storm', name: '風暴', icon: 'fa-cloud-bolt', desc: '指定場上 3 張敵方牌打亂位置，並使其變為已知狀態。' },
            tearBridge: { id: 'tearBridge', name: '淚橋', icon: 'fa-bridge-water', desc: '在己方陣前(第3排)生成淚橋(持續3回合)。消失時橋上單位直接死亡。' },
            petrify: { id: 'petrify', name: '石化', icon: 'fa-cubes', desc: '指定敵方數字牌，使其進入“石化”狀態(不可移動)。敵方可通過一次行動解除，解除後該牌防禦力 -3 (最低為 0)。' },
            rewind: { id: 'rewind', name: '回溯', icon: 'fa-backward-fast', desc: '將對局回到 3 回合前的狀態，但使用過的法術不會回復。' },
            obelisk: { id: 'obelisk', name: '奸碑', icon: 'fa-monument', desc: '在指定空格建立一個奸碑(不可移動，防禦力為 0)。敵方需擊殺奸碑後才可進入奸碑所在之橫排。' },
            soloDance: { id: 'soloDance', name: '獨舞', icon: 'fa-person-praying', desc: '當場上己方僅剩下一張數字牌時(國王不計入內)可用，該數字牌攻擊力和防禦力 +5。' },
            toggleBridge: { id: 'toggleBridge', name: '控制淚橋', icon: 'fa-bridge', desc: '傑局專屬：控制專屬淚橋的開啟或關閉(回合結束未用即消失)。' }
        };

        const ALL_SPELL_KEYS = Object.keys(SPELL_DEFINITIONS).filter(k => k !== 'toggleBridge'); // 隱藏特殊衍生法術

        const HEROES = {
            true_king: { id: 'true_king', name: '真王', spell: 'expedition' },
            deity: { id: 'deity', name: '天神', spell: 'thunder' },
            angry_one: { id: 'angry_one', name: '憤怒者', spell: 'berserk' },
            mage: { id: 'mage', name: '法師', spell: 'drawSpells' },
            centurion: { id: 'centurion', name: '百夫長', spell: null },
            sea_king: { id: 'sea_king', name: '海王', spell: 'river' },
            jieju: { id: 'jieju', name: '傑局', spell: 'tearBridge' }
        };

        /* --- Board Layout (3 Cols x Dynamic Rows) --- */
        const COLS = 3;
        let ROWS = 7; // 改為變數以支援動態地圖擴張

        function isValidCell(r, c) {
            if (r < 0 || r >= ROWS || c < 0 || c >= COLS) return false;
            if ((r === 0 || r === ROWS - 1) && c !== 1) return false;
            return true;
        }

        function getSetupRows(player) {
            return player === 1 ? [ROWS - 3, ROWS - 2] : [1, 2];
        }

        const GAME_PHASES = {
            HERO_SELECT_P1: 'HERO_SELECT_P1',
            HERO_SELECT_P2: 'HERO_SELECT_P2',
            SETUP_P1: 'SETUP_P1',
            SETUP_P2: 'SETUP_P2',
            BATTLE: 'BATTLE',
            END: 'END'
        };

        const DEFAULT_NUMBERS = [0, 0, 0, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11];

        // Card Object Factory
        function createCard(val, player, isRevealed = false) {
            return {
                id: Math.random().toString(36).substring(2, 9),
                val: val,
                player: player,
                isRevealed: val === 12 ? true : isRevealed,
                name: val === 12 ? '國王' : `數字 ${val}`,
                isHero: false,
                heroId: null,
                isAwakened: false,
                isFrozen: false,
                isInvincible: false,
                cantAttackKing: false,
                guandaoBuff: false,
                berserkBuff: 0,
                berserkTurns: 0,
                invincibleTurns: 0,
                invinciblePhase: 0,
                kingShield: 0,
                kingUnrestricted: false,
                freezeTimer: 0
            };
        }

        function createObeliskCard(player) {
            return {
                id: Math.random().toString(36).substring(2, 9),
                val: 0,
                player: player,
                isRevealed: true,
                name: '奸碑',
                isHero: false,
                isObelisk: true,
                cantAttackKing: true
            };
        }

        function createHeroCard(heroId, player) {
            return {
                id: Math.random().toString(36).substring(2, 9),
                val: (heroId === 'true_king' || heroId === 'centurion') ? 12 : -1, // True King & Centurion are 12, Deity loses to numbers
                player: player,
                isRevealed: true,
                name: HEROES[heroId].name,
                isHero: true,
                heroId: heroId,
                isAwakened: false,
                isFrozen: false,
                isInvincible: false,
                cantAttackKing: false,
                guandaoBuff: false,
                berserkBuff: 0,
                berserkTurns: 0,
                invincibleTurns: 0,
                invinciblePhase: 0,
                kingShield: 0,
                kingUnrestricted: false,
                freezeTimer: 0,
                steps: 0, // 法師步數追蹤
                puppetTurns: 0,
                originalPlayer: null
            };
        }

        // Global Game State Structure
        const WEATHER_TYPES = ['春天', '雷雨', '烈夏', '冬天'];

        let gameState = {
            gameMode: 'VS_AI',
            currentWeather: '春天',
            phase: GAME_PHASES.HERO_SELECT_P1,
            currentPlayer: 1,
            firstPlayer: 1,
            activeTab: 'hand',
            board: Array(ROWS).fill(null).map(() => Array(COLS).fill(null)),
            heroes: { 1: null, 2: null },
            setupPool: { 1: [], 2: [] },
            decks: { 1: [], 2: [] },
            hands: { 1: [], 2: [] },
            spells: { 1: [], 2: [] },
            usedSpells: { 1: [], 2: [] },
            graveyards: { 1: [], 2: [] },
            stealthTurns: { 1: 0, 2: 0 },
            traps: [],
            tunnels: [],
            fireballs: [],
            tearBridges: [],
            stormTargets: [],
            riverDuration: 0,
            riverFrozen: false,
            scytheStep: null,
            reviveCardChoice: null,
            mimickedSpellKey: null,
            selectedPos: null,
            selectedSpellIndex: null,
            selectedPoolIndex: null,
            validMoves: [],
            turnCount: 1,
            turnsWithoutDeath: 0,
            winner: null,
            playerAtkBuff: { 1: 0, 2: 0 },
            angryOneAwakened: { 1: false, 2: false },
            centurionAwakened: { 1: false, 2: false },
            seaKingAwakened: { 1: false, 2: false },
            seaKingTurnCounter: { 1: 0, 2: 0 },
            jiejuAwakened: { 1: false, 2: false },
            jiejuTurnCounter: { 1: 0, 2: 0 },
            jiejuBridgeActive: { 1: false, 2: false },
            jiejuDied: null,
            pendingJiejuSpellPlayer: null,
            pendingMageSpellTrigger: false,
            lastDeadCardValue: null,
            history: []
        };

        function addToGraveyard(card) {
            if (!card) return;
            gameState.graveyards[card.originalPlayer || card.player].push(card);
            gameState.lastDeadCardValue = card.val;
            
            if (card.isHero && card.heroId === 'jieju') {
                gameState.jiejuDied = card.player; // 標記傑局死亡，等待連鎖判定
            }
            
            for (let p of [1, 2]) {
                if (gameState.heroes[p] === 'centurion' && !gameState.centurionAwakened[p]) {
                    let grave = gameState.graveyards[p];
                    let has9 = grave.some(c => c.val === 9 && !c.isHero);
                    let has10 = grave.some(c => c.val === 10 && !c.isHero);
                    let has11 = grave.some(c => c.val === 11 && !c.isHero);
                    if (has9 && has10 && has11) {
                        gameState.centurionAwakened[p] = true;
                        addLog('sys', `🛡️ 【百夫長】達成條件 (9,10,11皆已陣亡)，完成覺醒！己方全場卡牌戰鬥力將強制受亡者之氣影響！`);
                        for (let r=0; r<ROWS; r++) {
                            for (let c=0; c<COLS; c++) {
                                let hc = gameState.board[r][c];
                                if (hc && hc.player === p && hc.isHero && hc.heroId === 'centurion') {
                                    hc.isAwakened = true;
                                }
                            }
                        }
                    }
                }
            }
        }

        let canvas, ctx;
        let isDeviceMobile = false;

        const tunnelImg = new Image();
        tunnelImg.src = 'image_8c5dc3.jpg';

        /* --- Device Layout Detection --- */
        function checkDeviceAndSetLayout() {
            const userAgentMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
            const isSmallScreen = window.innerWidth < 768;
            isDeviceMobile = userAgentMobile || isSmallScreen;

            const deviceBtnText = document.getElementById('device-text');
            const deviceIcon = document.getElementById('device-icon');
            if (deviceBtnText && deviceIcon) {
                if (isDeviceMobile) {
                    deviceBtnText.textContent = '行動模式';
                    deviceIcon.className = 'fa-solid fa-mobile-screen-button text-amber-400';
                } else {
                    deviceBtnText.textContent = '電腦模式';
                    deviceIcon.className = 'fa-solid fa-desktop text-amber-400';
                }
            }
        }

        function toggleDeviceLayout() {
            isDeviceMobile = !isDeviceMobile;
            checkDeviceAndSetLayout();
            resizeCanvas();
        }

        function setFirstMove(player) {
            soundEngine.playSelect();
            gameState.firstPlayer = player;
            const modal = document.getElementById('first-move-modal');
            const box = document.getElementById('first-move-box');
            if (modal && box) {
                modal.classList.add('opacity-0', 'pointer-events-none');
                box.classList.add('scale-90');
            }
            addLog('sys', `🎯 決定先後手：戰鬥階段將由 ${player === 1 ? '您 (P1)' : 'AI (P2)'} 先行回合。`);
        }

        /* --- Game Initialization & Hero Selection --- */
        function initGame() {
            gameState.currentPlayer = 1;
            gameState.firstPlayer = 1;
            gameState.activeTab = 'hand';
            gameState.winner = null;
            gameState.selectedPos = null;
            gameState.selectedSpellIndex = null;
            gameState.selectedPoolIndex = null;
            gameState.scytheStep = null;
            gameState.reviveCardChoice = null;
            gameState.mimickedSpellKey = null;
            gameState.validMoves = [];
            gameState.turnCount = 1;
            gameState.turnsWithoutDeath = 0;
            gameState.riverDuration = 0;
            gameState.riverFrozen = false;
            gameState.stealthTurns = { 1: 0, 2: 0 };
            gameState.traps = [];
            gameState.tunnels = [];
            gameState.fireballs = [];
            gameState.tearBridges = [];
            gameState.stormTargets = [];
            
            ROWS = 7; // 重置地圖大小為預設的 7 橫排
            gameState.board = Array(ROWS).fill(null).map(() => Array(COLS).fill(null));

            gameState.playerAtkBuff = { 1: 0, 2: 0 };
            gameState.angryOneAwakened = { 1: false, 2: false };
            gameState.mageAwakened = { 1: false, 2: false };
            gameState.centurionAwakened = { 1: false, 2: false };
            gameState.seaKingAwakened = { 1: false, 2: false };
            gameState.seaKingTurnCounter = { 1: 0, 2: 0 };
            gameState.jiejuAwakened = { 1: false, 2: false };
            gameState.jiejuTurnCounter = { 1: 0, 2: 0 };
            gameState.jiejuBridgeActive = { 1: false, 2: false };
            gameState.jiejuDied = null;
            gameState.pendingJiejuSpellPlayer = null;
            gameState.pendingMageSpellTrigger = false;
            gameState.lastDeadCardValue = null;
            gameState.pendingMageSpell = null;
            gameState.history = [];

            gameState.currentWeather = WEATHER_TYPES[Math.floor(Math.random() * WEATHER_TYPES.length)];
            gameState.hands = { 1: [], 2: [] };
            gameState.decks = { 1: [], 2: [] };
            gameState.graveyards = { 1: [], 2: [] };
            gameState.usedSpells = { 1: [], 2: [] };
            gameState.spells = { 1: [], 2: [] };
            gameState.heroes = { 1: null, 2: null };

            // Setup 15 default cards for each player
            gameState.setupPool[1] = [...DEFAULT_NUMBERS];
            gameState.setupPool[2] = [...DEFAULT_NUMBERS];

            // Pre-place Kings at (ROWS-1,1) and (0,1)
            gameState.board[ROWS - 1][1] = createCard(12, 1, true);
            gameState.board[0][1] = createCard(12, 2, true);

            const modal = document.getElementById('victory-modal');
            if (modal) modal.classList.add('opacity-0', 'pointer-events-none');

            clearLog();
            addLog('sys', '🎮 遊戲初始化完成！進入英雄選擇階段。');

            updateUI();
            resizeCanvas();

            // 顯示英雄選擇介面
            gameState.phase = GAME_PHASES.HERO_SELECT_P1;
            const heroModal = document.getElementById('hero-modal');
            const heroTitle = document.getElementById('hero-modal-title');
            if (heroTitle) heroTitle.textContent = 'P1 選擇英雄';

            // 重置英雄按鈕狀態 (解除禁用與反灰)
            ['true_king', 'deity', 'angry_one', 'mage', 'centurion', 'sea_king', 'jieju'].forEach(h => {
                const btn = document.getElementById(`hero-btn-${h}`);
                if (btn) {
                    btn.disabled = false;
                    btn.classList.remove('opacity-50', 'cursor-not-allowed', 'grayscale');
                }
            });

            if (heroModal) heroModal.classList.remove('opacity-0', 'pointer-events-none');
        }

        function selectHero(heroId) {
            soundEngine.playSelect();
            const p = gameState.currentPlayer;
            gameState.heroes[p] = heroId;
            
            if (heroId === 'centurion') {
                gameState.spells[p] = [];
                gameState.setupPool[p] = [...DEFAULT_NUMBERS, ...DEFAULT_NUMBERS];
                // 百夫長直接代替國王
                for (let r = 0; r < ROWS; r++) {
                    for (let c = 0; c < COLS; c++) {
                        if (gameState.board[r][c] && gameState.board[r][c].player === p && gameState.board[r][c].val === 12) {
                            gameState.board[r][c] = createHeroCard('centurion', p);
                        }
                    }
                }
            } else {
                gameState.spells[p] = [HEROES[heroId].spell];
                for (let i = 0; i < 4; i++) {
                    gameState.spells[p].push(ALL_SPELL_KEYS[Math.floor(Math.random() * ALL_SPELL_KEYS.length)]);
                }
            }

            if (p === 1) {
                if (gameState.gameMode === 'VS_AI') {
                    // AI 自動選擇與 P1 不同的剩餘英雄
                    gameState.currentPlayer = 2;
                    let availableHeroes = Object.keys(HEROES).filter(h => h !== heroId);
                    let aiHero = availableHeroes[Math.floor(Math.random() * availableHeroes.length)];
                    selectHero(aiHero);
                } else {
                    gameState.currentPlayer = 2;
                    gameState.phase = GAME_PHASES.HERO_SELECT_P2;
                    document.getElementById('hero-modal-title').textContent = 'P2 選擇英雄';

                    // 禁用 P1 已選的英雄，避免重複選擇
                    const chosenBtn = document.getElementById(`hero-btn-${heroId}`);
                    if (chosenBtn) {
                        chosenBtn.disabled = true;
                        chosenBtn.classList.add('opacity-50', 'cursor-not-allowed', 'grayscale');
                    }
                }
            } else {
                // 雙方選擇完畢，進入佈陣階段
                const heroModal = document.getElementById('hero-modal');
                if (heroModal) heroModal.classList.add('opacity-0', 'pointer-events-none');
                
                gameState.currentPlayer = 1;
                gameState.phase = GAME_PHASES.SETUP_P1;
                addLog('sys', '🎮 雙方英雄選擇完畢！請進行秘密佈陣。');
                
                if (gameState.gameMode === 'VS_AI') {
                    const fmModal = document.getElementById('first-move-modal');
                    const fmBox = document.getElementById('first-move-box');
                    if (fmModal && fmBox) {
                        fmModal.classList.remove('opacity-0', 'pointer-events-none');
                        fmBox.classList.remove('scale-90');
                    }
                }
                updateUI();
                render();
            }
        }

        function setGameMode(mode) {
            if (gameState.gameMode === mode) return;
            gameState.gameMode = mode;
            
            const p2Btn = document.getElementById('mode-2p-btn');
            const aiBtn = document.getElementById('mode-ai-btn');
            
            if (mode === '2P') {
                p2Btn.className = 'px-2 py-1 rounded-lg text-[11px] font-bold transition flex items-center gap-1 bg-amber-500 text-slate-950 shadow-md shadow-amber-500/20';
                aiBtn.className = 'px-2 py-1 rounded-lg text-[11px] font-bold transition flex items-center gap-1 text-slate-400 hover:text-slate-200';
            } else {
                aiBtn.className = 'px-2 py-1 rounded-lg text-[11px] font-bold transition flex items-center gap-1 bg-amber-500 text-slate-950 shadow-md shadow-amber-500/20';
                p2Btn.className = 'px-2 py-1 rounded-lg text-[11px] font-bold transition flex items-center gap-1 text-slate-400 hover:text-slate-200';
            }
            addLog('sys', `🎮 切換至 ${mode === '2P' ? '雙人對戰' : '人機 (AI)'} 模式。`);
            initGame();
        }

        /* --- Canvas Resize (Height Constrained 63:87) --- */
        function resizeCanvas() {
            if (!canvas || !canvas.parentElement) return;
            const container = canvas.parentElement;
            const availableW = container.clientWidth - 16;
            const availableH = container.clientHeight - 16;
            
            // 3 cols, 7 rows; each cell ratio 63:87 -> aspect ratio = (3*63) / (7*87)
            const targetRatio = (COLS * 63) / (ROWS * 87);
            
            let w = availableW;
            let h = w / targetRatio;
            
            if (h > availableH) {
                h = availableH;
                w = h * targetRatio;
            }
            
            canvas.width = Math.max(180, Math.floor(w));
            canvas.height = Math.max(260, Math.floor(h));
            render();
        }

        function showActionBar(show, text = '') {
            const bar = document.getElementById('action-bar');
            const desc = document.getElementById('action-desc');
            if (!bar) return;
            if (show) {
                if (desc) desc.textContent = text;
                bar.classList.remove('opacity-0', 'pointer-events-none', 'translate-y-2');
                const controls = document.getElementById('setup-controls');
                if (controls) controls.classList.add('hidden');
            } else {
                bar.classList.add('opacity-0', 'pointer-events-none', 'translate-y-2');
                const controls = document.getElementById('setup-controls');
                if (controls && (gameState.phase === GAME_PHASES.SETUP_P1 || gameState.phase === GAME_PHASES.SETUP_P2)) {
                    controls.classList.remove('hidden');
                }
            }
        }

        function cancelSelection() {
            gameState.selectedPos = null;
            gameState.selectedPoolIndex = null;
            gameState.selectedSpellIndex = null;
            gameState.scytheStep = null;
            gameState.reviveCardChoice = null;
            gameState.mimickedSpellKey = null;
            gameState.stormTargets = [];
            gameState.validMoves = [];
            showActionBar(false);
            renderSetupPoolHTML();
            render();
        }

        /* --- Setup Controls --- */
        function autoSetupCurrentPlayer() {
            const p = gameState.currentPlayer;
            if (gameState.phase !== GAME_PHASES.SETUP_P1 && gameState.phase !== GAME_PHASES.SETUP_P2) return;
            
            soundEngine.playSelect();
            const rows = getSetupRows(p);
            const validCells = [];
            for (let r of rows) {
                for (let c = 0; c < COLS; c++) {
                    if (isValidCell(r, c) && !gameState.board[r][c]) {
                        validCells.push({ r, c });
                    }
                }
            }
            
            shuffleArray(validCells);
            while (gameState.setupPool[p].length > 0 && validCells.length > 0) {
                const val = gameState.setupPool[p].pop();
                const cell = validCells.pop();
                gameState.board[cell.r][cell.c] = createCard(val, p, false);
            }
            
            cancelSelection();
            updateUI();
            render();
        }

        function clearCurrentPlayerSetup() {
            const p = gameState.currentPlayer;
            if (gameState.phase !== GAME_PHASES.SETUP_P1 && gameState.phase !== GAME_PHASES.SETUP_P2) return;
            
            soundEngine.playSelect();
            const rows = getSetupRows(p);
            for (let r of rows) {
                for (let c = 0; c < COLS; c++) {
                    if (isValidCell(r, c) && gameState.board[r][c] && gameState.board[r][c].val !== 12) {
                        gameState.setupPool[p].push(gameState.board[r][c].val);
                        gameState.board[r][c] = null;
                    }
                }
            }
            cancelSelection();
            updateUI();
            render();
        }

        function confirmCurrentPlayerSetup() {
            const p = gameState.currentPlayer;
            if (gameState.phase !== GAME_PHASES.SETUP_P1 && gameState.phase !== GAME_PHASES.SETUP_P2) return;
            
            const rows = getSetupRows(p);
            let placedCount = 0;
            for (let r of rows) {
                for (let c = 0; c < COLS; c++) {
                    if (isValidCell(r, c) && gameState.board[r][c] && gameState.board[r][c].val !== 12) {
                        placedCount++;
                    }
                }
            }
            
            if (placedCount < 6) {
                showToast(`請佈置滿 6 張卡牌！(目前已佈置 ${placedCount}/6)`);
                soundEngine.playError();
                return;
            }
            
            const remainingDeck = gameState.setupPool[p].map(val => createCard(val, p, false));
            shuffleArray(remainingDeck);
            gameState.decks[p] = remainingDeck;
            gameState.setupPool[p] = [];
            
            addLog(p === 1 ? 'p1' : 'p2', `✅ P${p} 完成陣地秘密佈陣！`);
            
            if (gameState.phase === GAME_PHASES.SETUP_P1) {
                if (gameState.gameMode === 'VS_AI') {
                    // Auto setup for AI (P2)
                    const aiRows = getSetupRows(2);
                    const aiCells = [];
                    for (let r of aiRows) {
                        for (let c = 0; c < COLS; c++) {
                            if (isValidCell(r, c) && !gameState.board[r][c]) aiCells.push({ r, c });
                        }
                    }
                    shuffleArray(aiCells);
                    const aiPool = [...gameState.setupPool[2]];
                    shuffleArray(aiPool);
                    
                    for (let i = 0; i < 6 && aiCells.length > 0; i++) {
                        const val = aiPool.pop();
                        const cell = aiCells.pop();
                        gameState.board[cell.r][cell.c] = createCard(val, 2, false);
                    }
                    
                    gameState.decks[2] = aiPool.map(val => createCard(val, 2, false));
                    shuffleArray(gameState.decks[2]);
                    gameState.setupPool[2] = [];
                    
                    addLog('p2', `🤖 AI 自動完成了秘密佈陣！`);
                    startBattlePhase();
                    
                    cancelSelection();
                    updateUI();
                    render();

                    if (gameState.currentPlayer === 2) {
                        triggerAiTurn();
                    }
                } else {
                    gameState.phase = GAME_PHASES.SETUP_P2;
                    gameState.currentPlayer = 2;
                    addLog('sys', `請 P2 進行秘密佈陣。`);
                    
                    cancelSelection();
                    showPassDeviceModal(2);
                }
            } else if (gameState.phase === GAME_PHASES.SETUP_P2) {
                startBattlePhase();
                cancelSelection();
                
                if (gameState.gameMode === '2P') {
                    showPassDeviceModal(gameState.currentPlayer);
                } else {
                    updateUI();
                    render();
                }
            }
        }

        function startBattlePhase() {
            gameState.phase = GAME_PHASES.BATTLE;
            gameState.currentPlayer = gameState.gameMode === 'VS_AI' ? gameState.firstPlayer : 1;
            
            // 戰鬥階段開始，將雙方英雄直接加入手牌 (百夫長除外，因其已在場上替代國王)
            if (gameState.heroes[1] !== 'centurion') gameState.hands[1].push(createHeroCard(gameState.heroes[1], 1));
            if (gameState.heroes[2] !== 'centurion') gameState.hands[2].push(createHeroCard(gameState.heroes[2], 2));

            // Initial draw: 3 cards each
            for (let i = 0; i < 3; i++) {
                drawCard(1);
                drawCard(2);
            }
            
            saveGameStateHistory();
            addLog('sys', `⚔️ 戰鬥階段正式開始！英雄已發配至手牌！輪到 P${gameState.currentPlayer} 回合。`);
        }

        function saveGameStateHistory() {
            try {
                const snapshot = structuredClone({
                    board: gameState.board,
                    hands: gameState.hands,
                    decks: gameState.decks,
                    graveyards: gameState.graveyards,
                    playerAtkBuff: gameState.playerAtkBuff,
                    angryOneAwakened: gameState.angryOneAwakened,
                    centurionAwakened: gameState.centurionAwakened,
                    seaKingAwakened: gameState.seaKingAwakened,
                    seaKingTurnCounter: gameState.seaKingTurnCounter,
                    jiejuAwakened: gameState.jiejuAwakened,
                    jiejuBridgeActive: gameState.jiejuBridgeActive,
                    stealthTurns: gameState.stealthTurns,
                    traps: gameState.traps,
                    tunnels: gameState.tunnels,
                    fireballs: gameState.fireballs,
                    tearBridges: gameState.tearBridges,
                    turnCount: gameState.turnCount,
                    riverDuration: gameState.riverDuration
                });
                gameState.history.push(snapshot);
                if (gameState.history.length > 10) gameState.history.shift();
            } catch (e) {
                console.error("History save failed", e);
            }
        }

        function updateUI() {
            const statusText = document.getElementById('game-status');
            const turnBadge = document.getElementById('turn-badge');
            const turnPhase = document.getElementById('turn-phase');
            const setupCount = document.getElementById('setup-count');
            const setupCountWrap = document.getElementById('setup-count-wrap');
            
            const p1Hand = document.getElementById('p1-hand-count');
            const p1Spell = document.getElementById('p1-spell-count');
            const p1Grave = document.getElementById('p1-grave-count');
            
            const p2Hand = document.getElementById('p2-hand-count');
            const p2Spell = document.getElementById('p2-spell-count');
            const p2Grave = document.getElementById('p2-grave-count');
            
            if (p1Hand) p1Hand.textContent = gameState.hands[1].length;
            if (p1Spell) p1Spell.textContent = gameState.spells[1].length;
            if (p1Grave) p1Grave.textContent = gameState.graveyards[1].length;
            
            if (p2Hand) p2Hand.textContent = gameState.hands[2].length;
            if (p2Spell) p2Spell.textContent = gameState.spells[2].length;
            if (p2Grave) p2Grave.textContent = gameState.graveyards[2].length;
            
            const weatherBadge = document.getElementById('weather-badge');
            if (weatherBadge) {
                let wColor = 'bg-emerald-600'; // 春天
                if (gameState.currentWeather === '雷雨') wColor = 'bg-indigo-600';
                if (gameState.currentWeather === '烈夏') wColor = 'bg-red-600';
                if (gameState.currentWeather === '冬天') wColor = 'bg-cyan-600';
                weatherBadge.className = `px-2 py-0.5 md:px-2.5 md:py-1 rounded-md text-[11px] md:text-xs font-bold text-white shadow ${wColor}`;
                weatherBadge.textContent = `天氣: ${gameState.currentWeather}`;
            }

            const p = gameState.currentPlayer;
            
            if (gameState.phase === GAME_PHASES.HERO_SELECT_P1 || gameState.phase === GAME_PHASES.HERO_SELECT_P2) {
                if (turnBadge) turnBadge.textContent = `英雄挑選階段`;
                if (turnPhase) turnPhase.textContent = `請挑選您的本命英雄`;
                if (setupCountWrap) setupCountWrap.style.display = 'none';
            } else if (gameState.phase === GAME_PHASES.SETUP_P1 || gameState.phase === GAME_PHASES.SETUP_P2) {
                if (statusText) statusText.textContent = `P${p} 佈陣中`;
                if (turnBadge) turnBadge.textContent = `P${p} 佈陣階段`;
                if (turnPhase) turnPhase.textContent = `請將卡牌佈置至己方前兩排`;
                
                const rows = getSetupRows(p);
                let placed = 0;
                for (let r of rows) {
                    for (let c = 0; c < COLS; c++) {
                        if (isValidCell(r, c) && gameState.board[r][c] && gameState.board[r][c].val !== 12) {
                            placed++;
                        }
                    }
                }
                if (setupCount) setupCount.textContent = placed;
                if (setupCountWrap) setupCountWrap.style.display = 'block';
            } else if (gameState.phase === GAME_PHASES.BATTLE) {
                if (statusText) statusText.textContent = `第 ${gameState.turnCount} 回合 - P${p} 回合`;
                if (turnBadge) turnBadge.textContent = `P${p} 對戰回合`;
                if (turnPhase) turnPhase.textContent = p === 2 && gameState.gameMode === 'VS_AI' ? `🤖 AI 思考中...` : `請選擇單位移動/攻擊，或使用手牌/法術`;
                if (setupCountWrap) setupCountWrap.style.display = 'none';
            } else if (gameState.phase === GAME_PHASES.END) {
                if (statusText) statusText.textContent = `對局結束`;
                if (turnBadge) turnBadge.textContent = `結束`;
                if (turnPhase) turnPhase.textContent = `勝負已分！`;
            }
            
            renderSetupPoolHTML();
        }

        /* --- Card Tab Switcher --- */
        function switchCardTab(tab) {
            gameState.activeTab = tab;
            gameState.selectedPos = null;
            gameState.selectedSpellIndex = null;
            gameState.scytheStep = null;
            gameState.reviveCardChoice = null;
            gameState.mimickedSpellKey = null;
            gameState.validMoves = [];
            showActionBar(false);

            const handBtn = document.getElementById('tab-hand-btn');
            const spellBtn = document.getElementById('tab-spell-btn');

            if (tab === 'hand') {
                handBtn.className = 'px-2.5 py-0.5 rounded font-bold transition bg-amber-500 text-slate-950 shadow';
                spellBtn.className = 'px-2.5 py-0.5 rounded font-bold transition text-slate-400 hover:text-slate-200';
            } else {
                spellBtn.className = 'px-2.5 py-0.5 rounded font-bold transition bg-amber-500 text-slate-950 shadow';
                handBtn.className = 'px-2.5 py-0.5 rounded font-bold transition text-slate-400 hover:text-slate-200';
            }

            renderSetupPoolHTML();
            render();
        }

        /* --- Render Setup & Battle Pool HTML --- */
        function renderSetupPoolHTML() {
            const grid = document.getElementById('setup-card-grid');
            const title = document.getElementById('setup-pool-title');
            const count = document.getElementById('setup-pool-count');
            const controls = document.getElementById('setup-controls');
            const tabs = document.getElementById('battle-card-tabs');
            const hint = document.getElementById('panel-hint');
            const spellCountText = document.getElementById('spell-tab-count');

            if (!grid) return;

            // 在 AI 模式下，永遠只顯示玩家(P1)的手牌與法術，避免洩漏 AI 底牌
            const p = gameState.gameMode === 'VS_AI' ? 1 : gameState.currentPlayer;

            if (spellCountText) spellCountText.textContent = gameState.spells[p].length;

            if (gameState.phase === GAME_PHASES.HERO_SELECT_P1 || gameState.phase === GAME_PHASES.HERO_SELECT_P2) {
                if (tabs) tabs.classList.add('hidden');
                if (controls) controls.classList.add('hidden');
                grid.innerHTML = '';
                if (hint) hint.textContent = '等待英雄選擇...';
            } else if (gameState.phase === GAME_PHASES.SETUP_P1 || gameState.phase === GAME_PHASES.SETUP_P2) {
                if (tabs) tabs.classList.add('hidden');
                if (controls && gameState.selectedPoolIndex === null) controls.classList.remove('hidden');
                if (hint) hint.textContent = '💡 點擊下方卡牌放大，再點擊左側己方前排進行佈置：';

                const pool = gameState.setupPool[p];
                if (title) title.innerHTML = `<i class="fa-solid fa-shield-halved text-amber-400"></i> P${p} 備選卡牌庫`;
                if (count) count.textContent = `剩餘 ${pool.length} 張`;

                grid.className = 'grid grid-cols-5 gap-1.5 max-h-40 md:max-h-48 overflow-y-auto p-1 bg-slate-950/60 rounded-xl border border-slate-800';
                grid.innerHTML = pool.map((val, idx) => {
                    const isSel = gameState.selectedPoolIndex === idx;
                    return `
                        <button onclick="selectPoolCard(${idx})" class="card-ratio rounded-lg border-2 flex flex-col items-center justify-center p-1 transition-all ${
                            isSel 
                            ? 'bg-amber-500/20 border-amber-400 text-amber-300 scale-105 shadow-lg shadow-amber-500/20' 
                            : 'bg-slate-800/80 hover:bg-slate-700 border-slate-700 text-slate-200'
                        }">
                            <span class="text-sm font-black">${val}</span>
                            <span class="text-[9px] text-slate-400 mt-0.5">${val === 0 ? '反殺' : '數字'}</span>
                        </button>
                    `;
                }).join('');
            } else if (gameState.phase === GAME_PHASES.BATTLE) {
                if (tabs) tabs.classList.remove('hidden');
                if (controls) controls.classList.add('hidden');

                if (gameState.activeTab === 'hand') {
                    if (hint) hint.textContent = '💡 點擊手牌進行登場，或點擊棋盤上的單位進行移動：';
                    const hand = gameState.hands[p];
                    if (title) title.innerHTML = `<i class="fa-solid fa-hand text-amber-400"></i> P${p} 手牌區`;
                    if (count) count.textContent = `${hand.length} / 7 張`;

                    grid.className = 'grid grid-cols-4 gap-1.5 max-h-40 md:max-h-48 overflow-y-auto p-1 bg-slate-950/60 rounded-xl border border-slate-800';
                    grid.innerHTML = hand.map((card, idx) => {
                        const isSel = gameState.selectedPos && gameState.selectedPos.type === 'hand' && gameState.selectedPos.index === idx;
                        
                        let displayVal = card.val;
                        let subText = '單位';
                        let bgClass = 'bg-blue-900/60 hover:bg-blue-800 border-blue-700';
                        
                        if (card.val === 12) {
                            displayVal = '👑';
                            subText = '國王';
                        }
                        if (card.isHero) {
                            if (card.heroId === 'true_king') {
                                displayVal = '👑真';
                                bgClass = 'bg-amber-900/60 hover:bg-amber-800 border-amber-700';
                            } else if (card.heroId === 'deity') {
                                displayVal = '⚡神';
                                bgClass = 'bg-sky-900/60 hover:bg-sky-800 border-sky-700';
                            } else if (card.heroId === 'angry_one') {
                                displayVal = '😡怒';
                                bgClass = 'bg-red-900/60 hover:bg-red-800 border-red-700';
                            } else if (card.heroId === 'mage') {
                                displayVal = '🧙法';
                                bgClass = 'bg-purple-900/60 hover:bg-purple-800 border-purple-700';
                            } else if (card.heroId === 'centurion') {
                                displayVal = '🛡️百';
                                bgClass = 'bg-emerald-900/60 hover:bg-emerald-800 border-emerald-700';
                            } else if (card.heroId === 'sea_king') {
                                displayVal = '🌊海';
                                bgClass = 'bg-cyan-900/60 hover:bg-cyan-800 border-cyan-700';
                            } else if (card.heroId === 'jieju') {
                                displayVal = '🌉傑';
                                bgClass = 'bg-indigo-900/60 hover:bg-indigo-800 border-indigo-700';
                            }
                            subText = '英雄';
                        }

                        return `
                            <button onclick="selectHandCard(${idx})" class="card-ratio rounded-lg border-2 flex flex-col items-center justify-center p-1 transition-all ${
                                isSel 
                                ? 'bg-amber-500/20 border-amber-400 text-amber-300 scale-105 shadow-lg shadow-amber-500/20' 
                                : `${bgClass} text-slate-100`
                            }">
                                <span class="text-base font-black">${displayVal}</span>
                                <span class="text-[9px] text-slate-300 mt-0.5">${subText}</span>
                            </button>
                        `;
                    }).join('');
                } else {
                    // Spell Tab Rendering
                    if (hint) hint.textContent = '💡 點擊法術並對棋盤目標發動（每張限用一次）：';
                    const spellKeys = gameState.spells[p];
                    if (title) title.innerHTML = `<i class="fa-solid fa-wand-magic-sparkles text-amber-400"></i> P${p} 法術書`;
                    if (count) count.textContent = `${spellKeys.length} 張可用`;

                    grid.className = 'grid grid-cols-2 gap-1.5 max-h-40 md:max-h-48 overflow-y-auto p-1 bg-slate-950/60 rounded-xl border border-slate-800';
                    grid.innerHTML = spellKeys.map((key, idx) => {
                        const spell = SPELL_DEFINITIONS[key];
                        const isSel = gameState.selectedSpellIndex === idx;
                        return `
                            <button onclick="selectSpellToCast(${idx})" class="p-2 rounded-lg border-2 flex flex-col text-left transition-all ${
                                isSel 
                                ? 'bg-amber-500/20 border-amber-400 text-amber-300 shadow-lg shadow-amber-500/20' 
                                : 'bg-slate-800/80 hover:bg-slate-700 border-slate-700 text-slate-200'
                            }">
                                <div class="flex items-center space-x-1.5 font-bold text-xs text-amber-400 mb-0.5">
                                    <i class="fa-solid ${spell.icon}"></i>
                                    <span>${spell.name}</span>
                                </div>
                                <span class="text-[10px] text-slate-300 leading-tight">${spell.desc}</span>
                            </button>
                        `;
                    }).join('');
                }
            }
        }

        /* --- Spell Action Handlers --- */
        function selectSpellToCast(index) {
            // 防止在 AI 回合時玩家誤觸自己的法術
            if (gameState.gameMode === 'VS_AI' && gameState.currentPlayer === 2) return;

            const p = gameState.currentPlayer;
            const spellKey = gameState.spells[p][index];
            if (!spellKey) return;

            soundEngine.playSelect();

            if (gameState.selectedSpellIndex === index) {
                cancelSelection();
                return;
            }

            gameState.selectedSpellIndex = index;
            gameState.selectedPos = null;
            gameState.validMoves = [];
            gameState.scytheStep = null;
            gameState.reviveCardChoice = null;
            gameState.mimickedSpellKey = null;
            gameState.stormTargets = [];

            if (spellKey === 'revive') {
                openReviveModal();
                return;
            } else if (spellKey === 'mimic') {
                openMimicModal();
                return;
            } else if (spellKey === 'river' || spellKey === 'drawSpells' || spellKey === 'stealth4' || spellKey === 'tearBridge' || spellKey === 'rewind' || spellKey === 'soloDance' || spellKey === 'toggleBridge') {
                executeSpell(spellKey, null, null);
                return;
            } else if (spellKey === 'reveal') {
                // 開牌可直接選擇施放目標為敵方
                const enemy = p === 1 ? 2 : 1;
                executeSpell('reveal', null, null, enemy);
                return;
            }

            const targets = getValidSpellTargets(spellKey, p);
            if (targets.length === 0) {
                showToast(`無法施放 ${SPELL_DEFINITIONS[spellKey].name}：場上沒有合法目標！`);
                soundEngine.playError();
                gameState.selectedSpellIndex = null;
                renderSetupPoolHTML();
                return;
            }

            gameState.validMoves = targets.map(t => ({ r: t.r, c: t.c, type: 'place' }));
            showActionBar(true, `已選擇法術【${SPELL_DEFINITIONS[spellKey].name}】，請點擊目標進行施放`);
            renderSetupPoolHTML();
            render();
        }

        function getValidSpellTargets(spellKey, player) {
            let targets = [];
            const enemy = player === 1 ? 2 : 1;
            
            // 檢查是否有覺醒的天神要施放神罰 (點選任何牌)
            let awakenedDeity = null;
            let enemyHasSeaKing = false;
            if (spellKey === 'thunder') {
                for (let r = 0; r < ROWS; r++) {
                    for (let c = 0; c < COLS; c++) {
                        let card = gameState.board[r][c];
                        if (card && card.player === player && card.isHero && card.heroId === 'deity' && card.isAwakened) {
                            awakenedDeity = card;
                        }
                    }
                }
            }

            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    let card = gameState.board[r][c];
                    if (card && card.player === enemy && card.isHero && card.heroId === 'sea_king') {
                        enemyHasSeaKing = true;
                    }
                }
            }

            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    if (!isValidCell(r, c)) continue;
                    const card = gameState.board[r][c];
                    
                    // 英雄免疫敵方法術指定（開牌除外）
                    if (card && card.isHero && card.player !== player && spellKey !== 'reveal') {
                        if (!(spellKey === 'thunder' && awakenedDeity)) {
                            continue;
                        }
                    }

                    // 海王絕對庇護：免疫石化、冰凍、傀儡
                    if (enemyHasSeaKing && card && card.player === enemy && (spellKey === 'freeze' || spellKey === 'puppet' || spellKey === 'petrify')) {
                        continue;
                    }

                    // 真王覺醒免疫控制
                    if (card && card.isHero && card.heroId === 'true_king' && card.isAwakened && spellKey === 'freeze') {
                        continue;
                    }
                    
                    if (awakenedDeity && spellKey === 'thunder') {
                        if (card && card.val !== 12) targets.push({r, c}); // 天神覺醒：可無視敵我摧毀任何非國王卡
                        continue;
                    }

                    if (spellKey === 'thunder') {
                        // 可以劈敵方暗牌，或是劈己方的天神(用來覺醒)
                        if ((card && card.player === enemy && !card.isRevealed) || 
                            (card && card.player === player && card.isHero && card.heroId === 'deity' && !card.isAwakened)) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'fire') {
                        if ((card && card.player === enemy && card.isRevealed && card.val <= 9 && card.val !== 12) || (card && card.isFrozen)) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'freeze') {
                        if (card && card.player === enemy && card.val !== 12 && !card.isFrozen) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'guandao') {
                        if (card && card.player === player) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'berserk') {
                        if (card && ((card.val <= 8 && !card.isHero) || (card.isHero && card.heroId === 'angry_one' && !card.isAwakened && card.player === player))) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'hat') {
                        if (card && card.player === player && card.val <= 6 && !card.isInvincible && !card.isHero) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'scythe') {
                        if (card && card.player === enemy && card.val !== 12) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'expedition') {
                        // 尋找國王，或是尋找己方未覺醒的真王
                        if (card && card.player === player && ((card.val === 12 && !card.isHero) || (card.isHero && card.heroId === 'true_king' && !card.isAwakened))) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'invincible3') {
                        if (card && card.player === player && card.invincibleTurns === 0 && !card.isInvincible) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'trap') {
                        if (!card && r !== 3) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'tunnel') {
                        if (!card) {
                            if (player === 1 && (r === 0 || r === 1)) targets.push({ r, c });
                            if (player === 2 && (r === ROWS - 2 || r === ROWS - 1)) targets.push({ r, c });
                        }
                    } else if (spellKey === 'puppet') {
                        if (card && card.player === enemy && card.val <= 8 && !card.isHero) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'storm') {
                        if (card && card.player === enemy) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'petrify') {
                        if (card && card.player === enemy && card.val !== 12 && !card.isHero && !card.isPetrified) {
                            targets.push({ r, c });
                        }
                    } else if (spellKey === 'obelisk') {
                        if (!card) {
                            targets.push({ r, c });
                        }
                    }
                }
            }
            return targets;
        }

        function executeSpell(spellKey, targetR, targetC, extraParam = null) {
            const p = gameState.currentPlayer;
            const spellDef = SPELL_DEFINITIONS[spellKey];

            soundEngine.playSpell();
            addLog(p === 1 ? 'p1' : 'p2', `✨ P${p} 施放了法術【${spellDef.name}】！`);

            if (spellKey === 'thunder') {
                const target = targetR !== null ? gameState.board[targetR][targetC] : null;
                let isDeityWrath = false;
                
                // 確認是否為天神的覺醒攻擊
                if (target && target.isHero && target.heroId === 'deity' && target.player === p && target.isAwakened) {
                    isDeityWrath = true;
                }
                
                if (target && target.isHero && target.heroId === 'deity' && target.player === p && !target.isAwakened) {
                    target.isAwakened = true;
                    addLog('sys', `⚡ 【天神】沐浴在雷電之中，完成終極覺醒！引發毀滅性神罰，抹殺場上除國王外的所有卡牌（包括自己）！`);
                    
                    let killedCount = 0;
                    for (let r = 0; r < ROWS; r++) {
                        for (let c = 0; c < COLS; c++) {
                            let card = gameState.board[r][c];
                            // 抹殺除了 12 (國王/真王/百夫長) 之外的所有牌
                            if (card && card.val !== 12) {
                                addToGraveyard(card);
                                gameState.board[r][c] = null;
                                killedCount++;
                            }
                        }
                    }
                    if (killedCount > 0) {
                        gameState.turnsWithoutDeath = 0;
                    }
                } else if (isDeityWrath) {
                    addLog('combat', `⚡ 天神覺醒神罰！無視一切防禦，直接消滅了 [${target.isHero ? target.name : (target.isRevealed ? target.name : '暗牌')}]！`);
                    addToGraveyard(target);
                    gameState.board[targetR][targetC] = null;
                    gameState.turnsWithoutDeath = 0;
                } else if (target) {
                    addLog('combat', `⚡ 雷電轟擊！敵方暗牌 [${target.name}] 被直接消滅！`);
                    addToGraveyard(target);
                    gameState.board[targetR][targetC] = null;
                    gameState.turnsWithoutDeath = 0;
                    
                    if (gameState.currentWeather === '雷雨') {
                        // 雷雨天氣連環閃電邏輯
                        let validTargets = [];
                        for (let i = 0; i < ROWS; i++) {
                            for (let j = 0; j < COLS; j++) {
                                if (gameState.board[i][j] && (i !== targetR || j !== targetC)) {
                                    validTargets.push({r: i, c: j});
                                }
                            }
                        }
                        if (validTargets.length > 0) {
                            let second = validTargets[Math.floor(Math.random() * validTargets.length)];
                            let secondTarget = gameState.board[second.r][second.c];
                            addToGraveyard(secondTarget);
                            gameState.board[second.r][second.c] = null;
                            addLog('combat', `⚡ 雷雨增強：連環閃電波及了另一張牌 [${secondTarget.isHero ? secondTarget.name : (secondTarget.isRevealed ? secondTarget.name : '暗牌')}]！`);
                        }
                    }
                }
            } else if (spellKey === 'fire') {
                const target = gameState.board[targetR][targetC];
                
                if ((gameState.currentWeather === '雷雨' || gameState.currentWeather === '冬天') && target.val > 6 && target.val !== 12 && !target.isFrozen) {
                    showToast("天氣削弱：火焰無法燒死數字大於 6 的牌！");
                    soundEngine.playError();
                    addLog('sys', `🌧️ 天氣削弱：火力不足，無法燒死大於 6 的 [${target.name}]！`);
                } else if (target.isFrozen) {
                    if (gameState.currentWeather === '冬天') {
                        addLog('sys', `❄️ 冬天削弱：火焰無法融化 [${target.name}] 的冰凍！`);
                    } else {
                        target.isFrozen = false;
                        addLog('combat', `🔥 火焰融化了 [${target.name}] 身上的冰霜，恢復行動能力！`);
                    }
                } else {
                    addLog('combat', `🔥 火焰吞噬！敵方單位 [${target.name}] 被燒毀！`);
                    addToGraveyard(target);
                    gameState.board[targetR][targetC] = null;
                    gameState.turnsWithoutDeath = 0;
                    
                    if (gameState.currentWeather === '烈夏') {
                        gameState.fireballs.push({r: targetR, c: targetC});
                        addLog('sys', `☀️ 烈夏增強：火焰在 (${targetR}, ${targetC}) 留下了一團烈火！`);
                    }
                }
            } else if (spellKey === 'freeze') {
                const target = gameState.board[targetR][targetC];
                target.isFrozen = true;
                target.freezeTimer = (gameState.currentWeather === '烈夏') ? 4 : 8;
                addLog('combat', `🧊 冰凍咒！敵方單位 [${target.isHero ? target.name : (target.isRevealed ? target.name : '暗牌')}] 被凍結 ${target.freezeTimer} 回合！`);
            } else if (spellKey === 'guandao') {
                const target = gameState.board[targetR][targetC];
                target.guandaoBuff = true;
                addLog('combat', `🗡️ [${target.isHero ? target.name : (target.isRevealed ? target.name : '暗牌')}] 獲得青龍偃月刀加持！擊殺單位後可再移動 1 步。`);
            } else if (spellKey === 'berserk') {
                const target = gameState.board[targetR][targetC];
                if (target.isHero && target.heroId === 'angry_one') {
                    if (!target.isAwakened) {
                        target.isAwakened = true;
                        gameState.angryOneAwakened[target.player] = true;
                        gameState.playerAtkBuff[target.player] += 2;
                        addLog('sys', `💥 【憤怒者】進入狂暴覺醒！免疫0-3攻擊，己方全體攻擊力再 +2！`);
                    }
                } else {
                    target.isRevealed = true;
                    target.berserkBuff = 3;
                    target.berserkTurns = 3;
                    addLog('combat', `💢 [${target.name}] 陷入狂暴！攻擊力 +3（持續 3 回合）。`);
                }
            } else if (spellKey === 'hat') {
                const target = gameState.board[targetR][targetC];
                target.isInvincible = true;
                target.cantAttackKing = true;
                addLog('combat', `🎩 [${target.isHero ? target.name : (target.isRevealed ? target.name : '暗牌')}] 戴上蔡只因帽！獲得永久無敵，但永久無法攻擊國王。`);
            } else if (spellKey === 'expedition') {
                const target = gameState.board[targetR][targetC];
                if (target.isHero && target.heroId === 'true_king') {
                    if (!target.isAwakened) {
                        target.isAwakened = true;
                        target.kingUnrestricted = true;
                        addLog('sys', `🌟 【真王】接受御駕親征，完成覺醒！獲得自由移動、免疫數字牌致死與雷雨天雷擊能力！`);
                    }
                } else {
                    target.kingUnrestricted = true;
                    target.kingShield = 1;
                    addLog(p === 1 ? 'p1' : 'p2', `👑 P${p} 國王御駕親征！解除移動限制並獲得 1 次免死護盾！`);
                }
            } else if (spellKey === 'river') {
                gameState.riverDuration = 3;
                gameState.riverFrozen = false;
                addLog('sys', `🌊 雙方前線升起波濤洶湧的河流（持續 3 回合），強行跨越將被淹死！`);
                
                // 海王覺醒判定
                for (let r = 0; r < ROWS; r++) {
                    for (let c = 0; c < COLS; c++) {
                        let card = gameState.board[r][c];
                        if (card && card.player === p && card.isHero && card.heroId === 'sea_king' && !card.isAwakened) {
                            card.isAwakened = true;
                            gameState.seaKingAwakened[p] = true;
                            gameState.seaKingTurnCounter[p] = 0;
                            addLog('sys', `🌊 【海王】呼應了河流的力量，完成終極覺醒！深淵暗流與週期造河能力解鎖！`);
                        }
                    }
                }
            } else if (spellKey === 'drawSpells') {
                // 法師覺醒判定
                let mageAwakenedNow = false;
                for (let r = 0; r < ROWS; r++) {
                    for (let c = 0; c < COLS; c++) {
                        let card = gameState.board[r][c];
                        if (card && card.player === p && card.isHero && card.heroId === 'mage' && !card.isAwakened) {
                            card.isAwakened = true;
                            gameState.mageAwakened[p] = true;
                            mageAwakenedNow = true;
                            addLog('sys', `🌟 【法師】汲取了強大法力，完成終極覺醒！解鎖回合初無盡法力湧現！`);
                        }
                    }
                }

                if (gameState.graveyards[p].length >= 4) {
                    let newSpells = [...ALL_SPELL_KEYS];
                    shuffleArray(newSpells);
                    gameState.spells[p].push(...newSpells.slice(0, 3));
                    addLog(p === 1 ? 'p1' : 'p2', `📜 P${p} 條件滿足，成功額外抽取了 3 張新法術！`);
                } else if (!mageAwakenedNow) {
                    showToast('累計死亡單位不足 4 張，無法發動抽牌！');
                    soundEngine.playError();
                    return;
                }
            } else if (spellKey === 'reveal') {
                const enemyTarget = extraParam || (p === 1 ? 2 : 1);
                for (let r = 0; r < ROWS; r++) {
                    for (let c = 0; c < COLS; c++) {
                        if (gameState.board[r][c] && gameState.board[r][c].player === enemyTarget) {
                            gameState.board[r][c].isRevealed = true;
                        }
                    }
                }
                addLog('sys', `👁️ 開牌發動！P${enemyTarget} 場上所有暗牌已被強制翻開！`);
            } else if (spellKey === 'invincible3') {
                const target = gameState.board[targetR][targetC];
                target.invincibleTurns = 3;
                target.invinciblePhase = 1;
                addLog('combat', `🛡️ [${target.isHero ? target.name : (target.isRevealed ? target.name : '暗牌')}] 獲得無敵狀態！（持續 3 回合：第1回合全攻，第2回合限攻≤8，第3回合限攻≤4，不可攻王）`);
            } else if (spellKey === 'stealth4') {
                gameState.stealthTurns[p] = 4;
                for (let r = 0; r < ROWS; r++) {
                    for (let c = 0; c < COLS; c++) {
                        const card = gameState.board[r][c];
                        if (card && card.player === p && card.val !== 12) {
                            card.isRevealed = false;
                        }
                    }
                }
                addLog('sys', `🥷 隱謐發動！P${p} 所有普通單位進入暗牌狀態，4回合內戰鬥不翻面！`);
            } else if (spellKey === 'trap') {
                gameState.traps.push({ r: targetR, c: targetC });
                addLog('sys', `☢️ 陷阱已佈置於 (${targetR}, ${targetC})，踩中即死！`);
            } else if (spellKey === 'tunnel') {
                gameState.tunnels.push({ r: targetR, c: targetC, player: p });
                addLog('sys', `🕳️ 地洞已挖掘於 (${targetR}, ${targetC})，友軍可無視距離傳送！`);
            } else if (spellKey === 'puppet') {
                const target = gameState.board[targetR][targetC];
                target.puppetTurns = 3;
                target.originalPlayer = target.player;
                target.player = p;
                target.cantAttackKing = true;
                addLog('combat', `🎭 傀儡術發動！[${target.isRevealed ? target.name : '暗牌'}] 被 P${p} 強制控制（持續 3 回合，期間不可攻王）！`);
            } else if (spellKey === 'storm') {
                let targets = gameState.stormTargets;
                let positions = targets.map(t => ({r: t.r, c: t.c}));
                let cards = targets.map(t => t.card);
                
                shuffleArray(cards);
                
                for(let i=0; i<positions.length; i++) {
                    cards[i].isRevealed = true;
                    gameState.board[positions[i].r][positions[i].c] = cards[i];
                }
                addLog('combat', `🌪️ 風暴席捲！${targets.length} 張敵方卡牌位置被打亂並強制翻開！`);
                gameState.stormTargets = [];
            } else if (spellKey === 'tearBridge') {
                let hasJieju = false;
                let jiejuAwakenedNow = false;
                
                // 檢查傑局是否在場與覺醒判定
                for (let r = 0; r < ROWS; r++) {
                    for (let c = 0; c < COLS; c++) {
                        let card = gameState.board[r][c];
                        if (card && card.player === p && card.isHero && card.heroId === 'jieju') {
                            hasJieju = true;
                            if (!card.isAwakened) {
                                card.isAwakened = true;
                                gameState.jiejuAwakened[p] = true;
                                gameState.jiejuTurnCounter[p] = 0;
                                jiejuAwakenedNow = true;
                            }
                        }
                    }
                }

                if (hasJieju) {
                    // 傑局在場時，淚橋只有一條且永久，不額外生成
                    if (jiejuAwakenedNow) {
                        addLog('sys', `🌉 【傑局】與淚橋法術產生共鳴，完成覺醒！傑局吸收了法術能量，專屬淚橋維持開啟，不會自動崩塌！`);
                    } else {
                        addLog('sys', `🌉 傑局將法術的能量吸收，維持了專屬淚橋的穩定，未生成額外的橋樑。`);
                    }
                } else {
                    // 一般英雄施放，生成會自動崩塌的臨時淚橋
                    let existingBridgeIdx = gameState.tearBridges.findIndex(tb => tb.player === p);
                    if (existingBridgeIdx !== -1) {
                        gameState.tearBridges[existingBridgeIdx].duration = 3;
                        addLog('sys', `🌉 P${p} 鞏固了現有的淚橋，重新計算 3 回合崩塌時間！`);
                    } else {
                        createTearBridge(p, false);
                        addLog('sys', `🌉 淚橋降臨！戰場擴張，P${p} 在陣前生成了全新的一橫排淚橋，3 回合後消失時將抹殺橋上所有單位！`);
                    }
                }
            } else if (spellKey === 'petrify') {
                const target = gameState.board[targetR][targetC];
                target.isPetrified = true;
                addLog('combat', `🗿 石化發動！[${target.isRevealed ? target.name : '暗牌'}] 被石化，失去移動能力！`);
            } else if (spellKey === 'obelisk') {
                gameState.board[targetR][targetC] = createObeliskCard(p);
                addLog('sys', `⛩️ 奸碑豎立！P${p} 在 (${targetR}, ${targetC}) 建立了一座防禦為 0 的奸碑！`);
            } else if (spellKey === 'soloDance') {
                let numberCards = [];
                for (let r = 0; r < ROWS; r++) {
                    for (let c = 0; c < COLS; c++) {
                        let card = gameState.board[r][c];
                        if (card && card.player === p && !card.isHero && card.val !== 12 && !card.isObelisk) {
                            numberCards.push(card);
                        }
                    }
                }
                if (numberCards.length === 1) {
                    let target = numberCards[0];
                    target.soloDanceBuff = (target.soloDanceBuff || 0) + 5;
                    target.isRevealed = true;
                    addLog('combat', `💃 獨舞綻放！己方僅存的 [${target.name}] 攻擊與防禦力永久 +5！`);
                } else {
                    showToast('發動失敗：己方場上數字牌數量不為 1 張！');
                    soundEngine.playError();
                    return; // 終止發動，不消耗法術
                }
            } else if (spellKey === 'rewind') {
                if (gameState.history.length === 0) {
                    showToast('沒有足夠的歷史紀錄可以回溯！');
                    soundEngine.playError();
                    return;
                }
                // 回溯 3 回合 (雙方各行動算1回合，故退 3 個歷史紀錄，不足則取最舊)
                let targetIndex = Math.max(0, gameState.history.length - 3);
                let snapshot = structuredClone(gameState.history[targetIndex]);
                
                gameState.board = snapshot.board;
                gameState.hands = snapshot.hands;
                gameState.decks = snapshot.decks;
                gameState.graveyards = snapshot.graveyards;
                gameState.playerAtkBuff = snapshot.playerAtkBuff;
                gameState.angryOneAwakened = snapshot.angryOneAwakened;
                gameState.centurionAwakened = snapshot.centurionAwakened;
                gameState.stealthTurns = snapshot.stealthTurns;
                gameState.traps = snapshot.traps;
                gameState.tunnels = snapshot.tunnels;
                gameState.fireballs = snapshot.fireballs;
                gameState.tearBridges = snapshot.tearBridges;
                gameState.turnCount = snapshot.turnCount;
                gameState.riverDuration = snapshot.riverDuration;
                
                addLog('sys', `⏳ 回溯發動！戰局狀態已倒轉至 ${gameState.turnCount} 回合！（已用過法術不恢復）`);
            } else if (spellKey === 'toggleBridge') {
                if (gameState.jiejuBridgeActive[p]) {
                    let tbIndex = gameState.tearBridges.findIndex(tb => tb.isJieju && tb.player === p);
                    if (tbIndex !== -1) {
                        addLog('sys', `🎛️ 傑局發動控制：強制關閉了專屬淚橋，橋上單位跌入深淵！`);
                        collapseTearBridge(tbIndex);
                        gameState.jiejuBridgeActive[p] = false;
                    }
                } else {
                    addLog('sys', `🎛️ 傑局發動控制：重新開啟了專屬淚橋，戰場擴充！`);
                    createTearBridge(p, true);
                    gameState.jiejuBridgeActive[p] = true;
                }
            }

            if (gameState.selectedSpellIndex !== null) {
                const used = gameState.spells[p].splice(gameState.selectedSpellIndex, 1)[0];
                gameState.usedSpells[p].push(used);
            }

            cancelSelection();
            checkWinConditions();
            if (!gameState.winner) {
                endBattleTurn();
            }
        }

        /* --- Modals for Revive & Mimic --- */
        function openReviveModal() {
            const p = gameState.currentPlayer;
            const grave = gameState.graveyards[p];
            if (!grave || grave.length === 0) {
                showToast('墓地中沒有可復活的卡牌！');
                soundEngine.playError();
                return;
            }

            const modal = document.getElementById('spell-modal');
            const title = document.getElementById('spell-modal-title');
            const content = document.getElementById('spell-modal-content');

            title.innerHTML = '<i class="fa-solid fa-ankh text-amber-400"></i> 選擇要復活的卡牌';
            content.innerHTML = grave.map((card, idx) => {
                let cardDisplay = card.val === 12 ? '👑 國王' : card.val;
                if (card.isHero) {
                    if (card.heroId === 'true_king') cardDisplay = '👑真';
                    else if (card.heroId === 'deity') cardDisplay = '⚡神';
                    else if (card.heroId === 'angry_one') cardDisplay = '😡怒';
                    else if (card.heroId === 'mage') cardDisplay = '🧙法';
                }
                return `
                <button onclick="selectReviveCard(${idx})" class="p-2.5 bg-slate-800 hover:bg-slate-700 border border-slate-700 rounded-xl flex flex-col items-center justify-center transition">
                    <span class="text-base font-black text-amber-300">${cardDisplay}</span>
                    <span class="text-[10px] text-slate-400 mt-0.5">${card.name}</span>
                </button>
                `;
            }).join('');

            modal.classList.remove('opacity-0', 'pointer-events-none');
        }

        function selectReviveCard(idx) {
            const p = gameState.currentPlayer;
            const card = gameState.graveyards[p].splice(idx, 1)[0];
            card.isRevealed = true;
            gameState.reviveCardChoice = card;

            closeSpellModal();

            const rows = getSetupRows(p);
            gameState.validMoves = [];
            for (let r of rows) {
                for (let c = 0; c < COLS; c++) {
                    if (isValidCell(r, c) && !gameState.board[r][c]) {
                        gameState.validMoves.push({ r, c, type: 'place' });
                    }
                }
            }

            showActionBar(true, `已選擇復活 [${card.name}]，請點擊己方陣地空位進行登場`);
            render();
        }

        function openMimicModal() {
            const allUsed = gameState.usedSpells[1].concat(gameState.usedSpells[2]);
            if (allUsed.length === 0) {
                showToast('本局遊戲尚未有已使用的法術！');
                soundEngine.playError();
                return;
            }

            const modal = document.getElementById('spell-modal');
            const title = document.getElementById('spell-modal-title');
            const content = document.getElementById('spell-modal-content');

            title.innerHTML = '<i class="fa-solid fa-mask text-amber-400"></i> 選擇要模仿的法術';
            content.innerHTML = allUsed.map((key) => {
                const spell = SPELL_DEFINITIONS[key];
                return `
                    <button onclick="selectMimicSpell('${key}')" class="p-2 bg-slate-800 hover:bg-slate-700 border border-slate-700 rounded-xl flex flex-col text-left transition">
                        <span class="text-xs font-bold text-amber-400 flex items-center gap-1">
                            <i class="fa-solid ${spell.icon}"></i> ${spell.name}
                        </span>
                        <span class="text-[10px] text-slate-300 mt-0.5 leading-tight">${spell.desc}</span>
                    </button>
                `;
            }).join('');

            modal.classList.remove('opacity-0', 'pointer-events-none');
        }

        function selectMimicSpell(key) {
            closeSpellModal();
            const p = gameState.currentPlayer;
            addLog(p === 1 ? 'p1' : 'p2', `🎭 模仿法術發動：複製【${SPELL_DEFINITIONS[key].name}】！`);
            
            const targets = getValidSpellTargets(key, p);

            if (key === 'expedition' || key === 'river' || key === 'drawSpells' || key === 'reveal' || key === 'stealth4' || key === 'tearBridge' || key === 'rewind' || key === 'soloDance' || key === 'toggleBridge') {
                executeSpell(key, null, null);
            } else if (key === 'revive') {
                openReviveModal();
            } else if (targets.length === 0) {
                showToast(`模仿失敗：${SPELL_DEFINITIONS[key].name} 沒有合法目標！`);
                soundEngine.playError();
            } else {
                gameState.mimickedSpellKey = key;
                gameState.validMoves = targets.map(t => ({ r: t.r, c: t.c, type: 'place' }));
                showActionBar(true, `模仿【${SPELL_DEFINITIONS[key].name}】，請選擇目標點擊施放`);
                render();
            }
        }

        function closeSpellModal() {
            const modal = document.getElementById('spell-modal');
            if (modal) modal.classList.add('opacity-0', 'pointer-events-none');
        }

        /* --- Hand Selection & Canvas Interaction --- */
        function selectPoolCard(index) {
            // 防止在 AI 回合時玩家誤觸
            if (gameState.gameMode === 'VS_AI' && gameState.currentPlayer === 2) return;

            soundEngine.playSelect();
            const p = gameState.currentPlayer;
            if (gameState.selectedPoolIndex === index) {
                gameState.selectedPoolIndex = null;
                gameState.validMoves = [];
                showActionBar(false);
            } else {
                gameState.selectedPoolIndex = index;
                const rows = getSetupRows(p);
                gameState.validMoves = [];
                for (let r of rows) {
                    for (let c = 0; c < COLS; c++) {
                        if (isValidCell(r, c) && !gameState.board[r][c]) {
                            gameState.validMoves.push({ r, c, type: 'place' });
                        }
                    }
                }
                showActionBar(true, '已選擇卡牌，請點擊左側己方前排/中排虛線框進行放置');
            }
            renderSetupPoolHTML();
            render();
        }

        function selectHandCard(index) {
            // 防止在 AI 回合時玩家誤觸自己的手牌
            if (gameState.gameMode === 'VS_AI' && gameState.currentPlayer === 2) return;

            soundEngine.playSelect();
            const p = gameState.currentPlayer;
            if (index < 0 || index >= gameState.hands[p].length) return;

            if (gameState.selectedPos && gameState.selectedPos.type === 'hand' && gameState.selectedPos.index === index) {
                cancelSelection();
                return;
            }

            gameState.selectedPos = { type: 'hand', index: index };
            gameState.selectedSpellIndex = null;
            gameState.scytheStep = null;
            gameState.reviveCardChoice = null;
            gameState.mimickedSpellKey = null;
            gameState.validMoves = getValidPlacementsForHand(p, index);
            
            if (gameState.validMoves.length === 0 && gameState.hands[p][index].isHero && gameState.hands[p][index].heroId === 'true_king') {
                showToast('真王需要墓地超過 6 張牌才能登場替換國王！');
                soundEngine.playError();
                cancelSelection();
                return;
            }

            showActionBar(true, '已選擇手牌，請點擊棋盤指示的合法空位進行部署');
            renderSetupPoolHTML();
            render();
        }

        function handleCanvasClick(e) {
            if (gameState.winner) return;
            const rect = canvas.getBoundingClientRect();
            const scaleX = canvas.width / rect.width;
            const scaleY = canvas.height / rect.height;

            const clickX = (e.clientX - rect.left) * scaleX;
            const clickY = (e.clientY - rect.top) * scaleY;

            const cellW = canvas.width / COLS;
            const cellH = canvas.height / ROWS;

            const c = Math.floor(clickX / cellW);
            const r = Math.floor(clickY / cellH);

            if (!isValidCell(r, c)) {
                cancelSelection();
                return;
            }

            const p = gameState.currentPlayer;

            if (gameState.gameMode === 'VS_AI' && p === 2 && gameState.phase === GAME_PHASES.BATTLE) {
                return;
            }

            // SETUP PHASE
            if (gameState.phase === GAME_PHASES.SETUP_P1 || gameState.phase === GAME_PHASES.SETUP_P2) {
                const setupRows = getSetupRows(p);
                const card = gameState.board[r][c];

                if (gameState.selectedPoolIndex !== null) {
                    if (setupRows.includes(r) && !card) {
                        soundEngine.playMove();
                        const val = gameState.setupPool[p].splice(gameState.selectedPoolIndex, 1)[0];
                        gameState.board[r][c] = createCard(val, p, false);
                        gameState.selectedPoolIndex = null;
                        gameState.validMoves = [];
                        showActionBar(false);
                        updateUI();
                        render();
                    } else if (card && card.player === p) {
                        soundEngine.playSelect();
                        gameState.setupPool[p].push(card.val);
                        gameState.board[r][c] = null;
                        updateUI();
                        render();
                    }
                    return;
                }

                if (card && card.player === p && setupRows.includes(r)) {
                    soundEngine.playSelect();
                    gameState.setupPool[p].push(card.val);
                    gameState.board[r][c] = null;
                    updateUI();
                    render();
                    return;
                }
            }

            // BATTLE PHASE
            if (gameState.phase === GAME_PHASES.BATTLE) {
                // Revive placement
                if (gameState.reviveCardChoice) {
                    const move = gameState.validMoves.find(m => m.r === r && m.c === c);
                    if (move) {
                        soundEngine.playMove();
                        const card = gameState.reviveCardChoice;
                        gameState.board[r][c] = card;
                        addLog(p === 1 ? 'p1' : 'p2', `📜 復活發動！[${card.name}] 放回戰場 (${r}, ${c})。`);
                        
                        if (gameState.selectedSpellIndex !== null) {
                            const used = gameState.spells[p].splice(gameState.selectedSpellIndex, 1)[0];
                            gameState.usedSpells[p].push(used);
                        }
                        gameState.reviveCardChoice = null;
                        cancelSelection();
                        checkWinConditions();
                        if (!gameState.winner) endBattleTurn();
                        return;
                    }
                }

                // Scythe 2-step mechanic
                const isScythe = (gameState.selectedSpellIndex !== null && gameState.spells[p][gameState.selectedSpellIndex] === 'scythe') || gameState.mimickedSpellKey === 'scythe';
                
                if (isScythe) {
                    if (!gameState.scytheStep) {
                        const card = gameState.board[r][c];
                        if (card && card.player !== p && card.val !== 12 && !card.isHero) {
                            soundEngine.playSelect();
                            gameState.scytheStep = { r, c, card };
                            
                            const setupRows = getSetupRows(p);
                            gameState.validMoves = [];
                            for (let sr of setupRows) {
                                for (let sc = 0; sc < COLS; sc++) {
                                    if (isValidCell(sr, sc)) gameState.validMoves.push({ r: sr, c: sc, type: 'place' });
                                }
                            }
                            showActionBar(true, `已鎖定 [${card.isRevealed ? card.name : '暗牌'}]，請點擊己方陣地目標格進行強制拽移！`);
                            render();
                            return;
                        }
                    } else {
                        const move = gameState.validMoves.find(m => m.r === r && m.c === c);
                        if (move) {
                            soundEngine.playAttack();
                            const enemyCard = gameState.board[gameState.scytheStep.r][gameState.scytheStep.c];
                            gameState.board[gameState.scytheStep.r][gameState.scytheStep.c] = null;

                            const destCard = gameState.board[r][c];
                            if (!destCard) {
                                gameState.board[r][c] = enemyCard;
                                addLog('combat', `💀 死神鐮刀發動！將 [${enemyCard.isRevealed ? enemyCard.name : '暗牌'}] 強制拽移至己方陣地 (${r}, ${c})！`);
                            } else if (destCard.player === p) {
                                addLog('combat', `💀 死神鐮刀發動！將 [${enemyCard.isRevealed ? enemyCard.name : '暗牌'}] 拽移至己方 [${destCard.isRevealed ? destCard.name : '暗牌'}] 面前觸發戰鬥！`);
                                const result = resolveCombat(destCard, enemyCard);
                                if (result.defenderDead) {
                                    addToGraveyard(enemyCard);
                                }
                                if (result.attackerDead) {
                                    addToGraveyard(destCard);
                                    if (!result.defenderDead) {
                                        gameState.board[r][c] = enemyCard;
                                    } else {
                                        gameState.board[r][c] = null;
                                    }
                                }
                                if (!result.attackerDead && !result.defenderDead) {
                                    // 拽過來的牌沒死，我方的牌也沒死，將敵人退回原位
                                    gameState.board[gameState.scytheStep.r][gameState.scytheStep.c] = enemyCard;
                                    addLog('combat', `🛡️ 雙方僵持！[${enemyCard.name}] 觸發機制未陣亡，退回原位！`);
                                }
                                if (result.attackerDead || result.defenderDead) {
                                    gameState.turnsWithoutDeath = 0;
                                }
                            }

                            const finalDestCard = gameState.board[r][c];
                            if (finalDestCard && finalDestCard.player !== p) {
                                const fbIdx = gameState.fireballs.findIndex(fb => fb.r === r && fb.c === c);
                                if (fbIdx !== -1) {
                                    gameState.fireballs.splice(fbIdx, 1);
                                    addLog('combat', `🔥 [${finalDestCard.name}] 位移後踩到了火團！`);
                                    if (finalDestCard.val <= 6) {
                                        soundEngine.playAttack();
                                        addLog('combat', `💥 烈火焚身！[${finalDestCard.name}] 被火團燒死了！`);
                                        addToGraveyard(finalDestCard);
                                        gameState.board[r][c] = null;
                                        gameState.turnsWithoutDeath = 0;
                                    }
                                }
                            }

                            if (gameState.selectedSpellIndex !== null) {
                                const used = gameState.spells[p].splice(gameState.selectedSpellIndex, 1)[0];
                                gameState.usedSpells[p].push(used);
                            }
                            gameState.scytheStep = null;
                            gameState.mimickedSpellKey = null;
                            cancelSelection();
                            checkWinConditions();
                            if (!gameState.winner) endBattleTurn();
                            return;
                        }
                    }
                }

                // Spell Target Cast
                if (gameState.selectedSpellIndex !== null || gameState.mimickedSpellKey) {
                    const spellKey = gameState.mimickedSpellKey || gameState.spells[p][gameState.selectedSpellIndex];
                    
                    if (spellKey === 'storm') {
                        const move = gameState.validMoves.find(m => m.r === r && m.c === c);
                        if (move) {
                            const targetCard = gameState.board[r][c];
                            if (targetCard && !gameState.stormTargets.some(t => t.r === r && t.c === c)) {
                                soundEngine.playSelect();
                                gameState.stormTargets.push({r, c, card: targetCard});
                                
                                if (gameState.stormTargets.length === 3 || gameState.stormTargets.length === gameState.validMoves.length) {
                                    gameState.mimickedSpellKey = null;
                                    executeSpell('storm', null, null);
                                } else {
                                    showActionBar(true, `已選擇 ${gameState.stormTargets.length}/3 張敵方牌，請繼續點擊...`);
                                }
                                render();
                            }
                            return;
                        }
                    } else {
                        const move = gameState.validMoves.find(m => m.r === r && m.c === c);
                        if (move) {
                            gameState.mimickedSpellKey = null;
                            executeSpell(spellKey, r, c);
                            return;
                        }
                    }
                }

                // Normal Board Movement / Combat
                if (gameState.selectedPos || gameState.selectedSpellIndex !== null) {
                    const move = gameState.validMoves.find(m => m.r === r && m.c === c);
                    if (move) {
                        executeMove(gameState.selectedPos, move);
                        return;
                    }
                }

                const card = gameState.board[r][c];
                if (card && card.player === p) {
                    soundEngine.playSelect();
                    gameState.selectedPos = { type: 'board', r, c };
                    gameState.selectedSpellIndex = null;
                    gameState.scytheStep = null;
                    gameState.reviveCardChoice = null;
                    gameState.mimickedSpellKey = null;
                    gameState.validMoves = getValidMovesForCard(r, c);
                    showActionBar(true, `已選擇 ${card.isHero ? card.name : (card.isRevealed ? card.name : '暗牌')}，請點擊目標進行移動或攻擊`);
                    renderSetupPoolHTML();
                    render();
                    return;
                }

                cancelSelection();
            }
        }

        function handleCanvasMouseMove(e) {
            if (!canvas) return;
            const rect = canvas.getBoundingClientRect();
            const clickX = (e.clientX - rect.left) * (canvas.width / rect.width);
            const clickY = (e.clientY - rect.top) * (canvas.height / rect.height);

            const c = Math.floor(clickX / (canvas.width / COLS));
            const r = Math.floor(clickY / (canvas.height / ROWS));

            if (isValidCell(r, c)) {
                canvas.style.cursor = 'pointer';
            } else {
                canvas.style.cursor = 'default';
            }
        }

        /* --- Move Rules & Combat Engine --- */
        function getValidMovesForCard(r, c) {
            const card = gameState.board[r][c];
            if (!card || card.player !== gameState.currentPlayer) return [];

            let moves = [];

            if (card.isPetrified) {
                moves.push({ r: r, c: c, type: 'unpetrify' });
                return moves;
            }

            if (card.isFrozen) {
                showToast('該單位已被冰凍，無法移動！');
                return [];
            }

            // 地洞傳送邏輯
            gameState.tunnels.forEach(t => {
                if (t.player === card.player && !gameState.board[t.r][t.c] && (t.r !== r || t.c !== c)) {
                    moves.push({ r: t.r, c: t.c, type: 'move' });
                }
            });

            const dirs = [
                [-1, -1], [-1, 0], [-1, 1],
                [0, -1],           [0, 1],
                [1, -1],  [1, 0],  [1, 1]
            ];

            for (let [dr, dc] of dirs) {
                let nr = r + dr;
                let nc = c + dc;

                if (isValidCell(nr, nc)) {
                    // 百夫長受限於國王相同的移動範圍
                    if ((card.val === 12 || (card.isHero && card.heroId === 'true_king' && !card.isAwakened) || (card.isHero && card.heroId === 'centurion')) && !card.kingUnrestricted) {
                        if (card.player === 1 && nr < ROWS - 2) continue;
                        if (card.player === 2 && nr > 1) continue;
                    }

                    // 奸碑阻擋邏輯 (若跨越橫排且目標橫排有敵方奸碑，則只能選擇攻擊該奸碑)
                    if (nr !== r) {
                        let hasEnemyObelisk = false;
                        for (let cc = 0; cc < COLS; cc++) {
                            let cell = gameState.board[nr][cc];
                            if (cell && cell.isObelisk && cell.player !== card.player) {
                                hasEnemyObelisk = true;
                                break;
                            }
                        }
                        if (hasEnemyObelisk) {
                            let targetCell = gameState.board[nr][nc];
                            if (!targetCell || !targetCell.isObelisk || targetCell.player === card.player) {
                                continue; 
                            }
                        }
                    }

                    const target = gameState.board[nr][nc];
                    // 不能攻擊國王或真王
                    if (target && (target.val === 12 || (target.isHero && target.heroId === 'true_king')) && card.cantAttackKing) {
                        continue;
                    }

                    if (card.invincibleTurns > 0) {
                        if (target && (target.val === 12 || (target.isHero && target.heroId === 'true_king'))) continue; // 無敵期間不可攻王
                        if (target && target.isRevealed) {
                            let targetDef = target.isHero ? 0 : target.val;
                            if (card.invinciblePhase >= 2 && targetDef > 8) continue;
                            if (card.invinciblePhase >= 3 && targetDef > 4) continue;
                        }
                    }

                    if (!target) {
                        moves.push({ r: nr, c: nc, type: 'move' });
                    } else if (target.player !== card.player) {
                        moves.push({ r: nr, c: nc, type: 'attack' });
                    }
                }
            }
            return moves;
        }

        function getValidPlacementsForHand(player, cardIndex) {
            let placements = [];
            const card = gameState.hands[player][cardIndex];
            
            if (card && card.isHero) {
                if (card.heroId === 'true_king') {
                    // 真王需墓地 > 6 才能部署替換國王
                    if (gameState.graveyards[player].length > 6) {
                        for (let r = 0; r < ROWS; r++) {
                            for (let c = 0; c < COLS; c++) {
                                const cell = gameState.board[r][c];
                                if (cell && cell.player === player && cell.val === 12) {
                                    placements.push({r, c, type: 'place'});
                                }
                            }
                        }
                    }
                    return placements;
                } else if (card.heroId === 'deity') {
                    // 天神部署在第一、二排 (P2) 或第四、五排 (P1)
                    const allowedRows = player === 1 ? [4, 5] : [1, 2];
                    for (let r of allowedRows) {
                        for (let c = 0; c < COLS; c++) {
                            if (isValidCell(r, c) && !gameState.board[r][c]) {
                                placements.push({ r, c, type: 'place' });
                            }
                        }
                    }
                    return placements;
                }
            }

            const allowedRows = player === 1 ? [4, 5] : [1, 2];
            for (let r of allowedRows) {
                for (let c = 0; c < COLS; c++) {
                    if (isValidCell(r, c) && !gameState.board[r][c]) {
                        placements.push({ r, c, type: 'place' });
                    }
                }
            }
            return placements;
        }

        function triggerTrueKingThunder(player) {
            const enemy = player === 1 ? 2 : 1;
            let targets = [];
            for(let r=0; r<ROWS; r++){
                for(let c=0; c<COLS; c++){
                    const card = gameState.board[r][c];
                    if (card && card.player === enemy && card.val !== 12) {
                        targets.push(card);
                    }
                }
            }
            if (targets.length > 0) {
                const target = targets[Math.floor(Math.random() * targets.length)];
                addLog('sys', `⚡ 真王覺醒 x 雷雨天！降下神雷劈死了 [${target.isRevealed ? target.name : '暗牌'}]！`);
                for(let r=0; r<ROWS; r++){
                    for(let c=0; c<COLS; c++){
                        if (gameState.board[r][c] === target) {
                            addToGraveyard(target);
                            gameState.board[r][c] = null;
                        }
                    }
                }
            }
        }

        function executeMove(from, move) {
            const p = gameState.currentPlayer;

            if (from.type === 'hand') {
                soundEngine.playMove();
                const card = gameState.hands[p].splice(from.index, 1)[0];
                
                if (card.isHero && card.heroId === 'true_king') {
                    // 真王替換國王
                    for (let r = 0; r < ROWS; r++) {
                        for (let c = 0; c < COLS; c++) {
                            if (gameState.board[r][c] && gameState.board[r][c].player === p && gameState.board[r][c].val === 12) {
                                gameState.board[r][c] = null; // 舊國王直接離場
                            }
                        }
                    }
                    gameState.board[move.r][move.c] = card;
                    addLog(p === 1 ? 'p1' : 'p2', `👑 英雄【真王】降臨！直接替換了原國王！`);
                } else {
                    gameState.board[move.r][move.c] = card;
                    addLog(p === 1 ? 'p1' : 'p2', `P${p} 從手牌部署了 ${card.isHero ? card.name : '1 張卡牌'} 至場上。`);
                    
                    if (card.isHero && card.heroId === 'deity') {
                        // 天神登場特效
                        addLog('sys', `⚡ 英雄【天神】降臨！釋放全場無差別雷電！`);
                        let killed = 0;
                        for(let rr=0; rr<ROWS; rr++) {
                            for(let cc=0; cc<COLS; cc++) {
                                let target = gameState.board[rr][cc];
                                if (target && target.val !== 12 && !target.isRevealed && target.id !== card.id) {
                                    gameState.graveyards[target.player].push(target);
                                    gameState.board[rr][cc] = null;
                                    killed++;
                                    addLog('combat', `⚡ 天神神罰！未知牌 [${target.name}] 被劈死！`);
                                }
                            }
                        }
                        if (killed > 0) {
                            gameState.turnsWithoutDeath = 0;
                            addLog('sys', `⚡ 天神登場消滅了 ${killed} 張數字牌！（注意：登場當回合不累計雷電法術）`);
                        }
                    } else if (card.isHero && card.heroId === 'angry_one') {
                        gameState.playerAtkBuff[p] += 1;
                        addLog('sys', `😡 英雄【憤怒者】降臨！所有己方單位攻擊力永久 +1！`);
                    } else if (card.isHero && card.heroId === 'jieju') {
                        addLog('sys', `🌉 英雄【傑局】降臨！引動深淵之力，在陣前永久生成一條專屬淚橋！`);
                        createTearBridge(p, true);
                        gameState.jiejuBridgeActive[p] = true;
                    }
                }
            } else if (from.type === 'board') {
                const attacker = gameState.board[from.r][from.c];
                let isAttackerDead = false;

                // Check River Crossing Drowning
                const isEnteringRiver = (move.r === 3);
                if (isEnteringRiver && gameState.riverDuration > 0 && !gameState.riverFrozen) {
                    let hasSeaKing = false;
                    for (let rr = 0; rr < ROWS; rr++) {
                        for (let cc = 0; cc < COLS; cc++) {
                            let cCard = gameState.board[rr][cc];
                            if (cCard && cCard.player === attacker.player && cCard.isHero && cCard.heroId === 'sea_king') hasSeaKing = true;
                        }
                    }
                    
                    if (hasSeaKing) {
                        addLog('sys', `🌊 海王庇護：[${attacker.isHero ? attacker.name : (attacker.isRevealed ? attacker.name : '暗牌')}] 無視河流急流，安全渡河！`);
                    } else {
                        soundEngine.playAttack();
                        addLog('combat', `🌊 警告！[${attacker.isHero ? attacker.name : (attacker.isRevealed ? attacker.name : '暗牌')}] 踏入湍急的河流，不幸淹死！`);
                        addToGraveyard(attacker);
                        gameState.board[from.r][from.c] = null;
                        gameState.turnsWithoutDeath = 0;
                        cancelSelection();
                        checkWinConditions();
                        if (!gameState.winner) endBattleTurn();
                        return;
                    }
                }

                if (move.type === 'move') {
                    soundEngine.playMove();
                    gameState.board[move.r][move.c] = attacker;
                    gameState.board[from.r][from.c] = null;
                    addLog(p === 1 ? 'p1' : 'p2', `P${p} 將 ${attacker.isHero ? attacker.name : (attacker.isRevealed ? attacker.name : '暗牌')} 移動至 (${move.r}, ${move.c})。`);
                } else if (move.type === 'unpetrify') {
                    soundEngine.playMove();
                    attacker.isPetrified = false;
                    attacker.defPenalty = (attacker.defPenalty || 0) + 3;
                    addLog(p === 1 ? 'p1' : 'p2', `🗿 P${p} 耗費一次行動解除了 [${attacker.name}] 的石化狀態！其防禦力永久下降 3 點。`);
                } else if (move.type === 'attack') {
                    soundEngine.playAttack();
                    const defender = gameState.board[move.r][move.c];
                    const result = resolveCombat(attacker, defender);
                    isAttackerDead = result.attackerDead;

                    if (result.defenderDead) {
                        addToGraveyard(defender);
                        gameState.board[move.r][move.c] = null;
                    }
                    if (result.attackerDead) {
                        addToGraveyard(attacker);
                        gameState.board[from.r][from.c] = null;
                    }
                    if (result.attackerDead || result.defenderDead) {
                        gameState.turnsWithoutDeath = 0;
                    }

                    if (!result.attackerDead && result.defenderDead && attacker.guandaoBuff) {
                        attacker.guandaoBuff = false;
                        attacker.cantAttackKing = true;
                        gameState.board[move.r][move.c] = attacker;
                        gameState.board[from.r][from.c] = null;

                        addLog('combat', `🗡️ 青龍偃月刀發威！[${attacker.name}] 可立即再移動一步（不可攻王）。`);
                        gameState.selectedPos = { type: 'board', r: move.r, c: move.c };
                        gameState.validMoves = getValidMovesForCard(move.r, move.c);
                        showActionBar(true, '🗡️ 青龍偃月刀再動！請點擊目標格進行第二步移動');
                        render();
                        return;
                    }

                    if (!result.attackerDead && result.defenderDead) {
                        gameState.board[move.r][move.c] = attacker;
                        gameState.board[from.r][from.c] = null;
                    }
                }

                // 法師步數追蹤與獲取法術
                if (!isAttackerDead && attacker.isHero && attacker.heroId === 'mage') {
                    attacker.steps = (attacker.steps || 0) + 1;
                    if (attacker.steps % 3 === 0) {
                        let validSpells = ALL_SPELL_KEYS.filter(s => s !== 'drawSpells');
                        let newSpell = validSpells[Math.floor(Math.random() * validSpells.length)];
                        gameState.spells[p].push(newSpell);
                        addLog('sys', `🔮 【法師】凝結魔力！累積移動/攻擊 3 步，獲得了一張額外法術【${SPELL_DEFINITIONS[newSpell].name}】！`);
                    }
                }
            }

            // 陷阱觸發判定
            const endCard = gameState.board[move.r][move.c];
            if (endCard) {
                const trapIdx = gameState.traps.findIndex(t => t.r === move.r && t.c === move.c);
                if (trapIdx !== -1) {
                    gameState.traps.splice(trapIdx, 1);
                    soundEngine.playAttack();
                    addLog('combat', `☢️ 踩中陷阱！[${endCard.name}] 被無情消滅！`);
                    addToGraveyard(endCard);
                    gameState.board[move.r][move.c] = null;
                    gameState.turnsWithoutDeath = 0;
                }
                
                const fbIdx = gameState.fireballs.findIndex(fb => fb.r === move.r && fb.c === move.c);
                if (fbIdx !== -1) {
                    gameState.fireballs.splice(fbIdx, 1);
                    addLog('combat', `🔥 [${endCard.name}] 踩到了地上的火團！`);
                    if (endCard.val <= 6) {
                        soundEngine.playAttack();
                        addLog('combat', `💥 烈火焚身！[${endCard.name}] 被火團燒死了！`);
                        addToGraveyard(endCard);
                        gameState.board[move.r][move.c] = null;
                        gameState.turnsWithoutDeath = 0;
                    }
                }
            }

            cancelSelection();
            checkWinConditions();

            if (!gameState.winner) {
                endBattleTurn();
            }
        }

        function resolveCombat(attacker, defender) {
            if (!gameState.stealthTurns[attacker.player]) attacker.isRevealed = true;
            if (!gameState.stealthTurns[defender.player]) defender.isRevealed = true;

            // 亡者之氣 (百夫長覺醒) 數值覆蓋邏輯 (僅限己方)
            let aBase = attacker.isHero ? 12 : attacker.val;
            let dBase = defender.isHero ? 0 : defender.val;
            
            let aOverride = false;
            let dOverride = false;

            if (gameState.centurionAwakened[attacker.player] && gameState.lastDeadCardValue !== null) {
                aBase = gameState.lastDeadCardValue;
                aOverride = true;
            }
            if (gameState.centurionAwakened[defender.player] && gameState.lastDeadCardValue !== null) {
                dBase = gameState.lastDeadCardValue;
                dOverride = true;
            }

            let aBuff = (attacker.val !== 0 && !attacker.isHero && attacker.val !== 12) ? gameState.playerAtkBuff[attacker.player] : 0;
            let aVal = aBase + (attacker.berserkBuff || 0) + aBuff + (attacker.soloDanceBuff || 0);
            
            // 憤怒者覺醒限制：最高為 11
            if (!attacker.isHero && attacker.val !== 12 && gameState.angryOneAwakened[attacker.player]) {
                if (aVal > 11) aVal = 11;
            }

            // 防禦力 (生命值) 不受增益光環影響
            let dVal = dBase + (defender.berserkBuff || 0) + (defender.soloDanceBuff || 0) - (defender.defPenalty || 0);
            if (dVal < 0) dVal = 0;

            let overrideMsg = '';
            if (aOverride || dOverride) {
                overrideMsg = `(受亡靈影響`;
                if (aOverride && dOverride) overrideMsg += `雙方皆視為 ${gameState.lastDeadCardValue}) `;
                else if (aOverride) overrideMsg += `P${attacker.player}視為 ${gameState.lastDeadCardValue}) `;
                else overrideMsg += `P${defender.player}視為 ${gameState.lastDeadCardValue}) `;
            }
            addLog('combat', `💥 對決爆發！${overrideMsg}P${attacker.player} [${attacker.isHero ? attacker.name : (attacker.isRevealed ? attacker.name : '暗牌')}] (攻:${aVal}) ⚔️ P${defender.player} [${defender.isHero ? defender.name : (defender.isRevealed ? defender.name : '暗牌')}] (防:${dVal})`);

            // 攻擊方無敵狀態限制判定
            if (attacker.invincibleTurns > 0) {
                if (defender.val === 12 || (defender.isHero && defender.heroId === 'true_king')) {
                    addLog('combat', `🛡️ 無敵限制：[${attacker.name}] 在無敵期間無法攻擊國王或真王，雙方僵持，保持原位！`);
                    return { attackerDead: false, defenderDead: false };
                }
                if (attacker.invinciblePhase >= 2 && dVal > 8) {
                    addLog('combat', `🛡️ 無敵限制：[${attacker.name}] 處於無敵第 ${attacker.invinciblePhase} 回合，只能攻擊防禦 8 以下單位，無法擊退 [${defender.name}]，雙方僵持！`);
                    return { attackerDead: false, defenderDead: false };
                }
                if (attacker.invinciblePhase >= 3 && dVal > 4) {
                    addLog('combat', `🛡️ 無敵限制：[${attacker.name}] 處於無敵第 ${attacker.invinciblePhase} 回合，只能攻擊防禦 4 以下單位，無法擊退 [${defender.name}]，雙方僵持！`);
                    return { attackerDead: false, defenderDead: false };
                }
            }

            let attackerDead = false;
            let defenderDead = false;

            // 先判定原始戰鬥勝負結果
            if (defender.val === 12 && !defender.isHero && defender.kingShield > 0) {
                defender.kingShield--;
                addLog('combat', `👑 P${defender.player} 國王觸發御駕親征免死護盾，免疫本次致死打擊！`);
                attackerDead = true;
            } else if ((attacker.val === 0 && !attacker.isHero && defender.val === 11 && !defender.isHero) || 
                       (attacker.val === 11 && !attacker.isHero && defender.val === 0 && !defender.isHero)) {
                addLog('combat', '💥 特殊規則：0 與 11 交鋒，雙方同歸於盡！');
                attackerDead = true;
                defenderDead = true;
            } else if (attacker.val === 0 && !attacker.isHero && defender.val === 0 && !defender.isHero) {
                addLog('combat', '💥 特殊規則：0 攻擊 0，雙方同歸於盡！');
                attackerDead = true;
                defenderDead = true;
            } else if (defender.val === 12 && !defender.isHero) {
                addLog('combat', '👑 特殊規則：國王受到攻擊！國王陣亡，攻擊者亦被反擊消滅！');
                attackerDead = true;
                defenderDead = true;
            } else if (attacker.val === 12 && !attacker.isHero) {
                addLog('combat', '👑 國王出征：直接消滅敵方單位！');
                attackerDead = false;
                defenderDead = true;
            } else if (aVal > dVal) {
                addLog('combat', `⚔️ 戰鬥結果：攻擊方 (${aVal} > ${dVal}) 獲勝，守備方陣亡。`);
                attackerDead = false;
                defenderDead = true;
            } else if (aVal < dVal) {
                addLog('combat', `⚔️ 戰鬥結果：守備方防禦 (${dVal} > ${aVal}) 數值較大，攻擊方反噬陣亡。`);
                attackerDead = true;
                defenderDead = false;
            } else {
                addLog('combat', `⚔️ 戰鬥結果：數值相同 (${aVal} vs ${dVal})，主動攻擊方獲得勝利！`);
                attackerDead = false;
                defenderDead = true;
            }

            // 無敵狀態免死救場
            if (defenderDead && (defender.isInvincible || defender.invincibleTurns > 0)) {
                addLog('combat', `🎩 [${defender.name}] 擁有無敵護體，免疫了本次致死打擊，成功存活！`);
                defenderDead = false;
            }
            if (attackerDead && (attacker.isInvincible || attacker.invincibleTurns > 0)) {
                addLog('combat', `🛡️ [${attacker.name}] 擁有無敵狀態護體，免疫了本次致死反擊！雙方留在原地。`);
                attackerDead = false;
            }
            
            // 真王絕對防禦保護邏輯：未覺醒防0-6，覺醒後防0-3
            if (defenderDead && defender.isHero && defender.heroId === 'true_king') {
                const maxKillVal = defender.isAwakened ? 3 : 6;
                const canKill = !attacker.isHero && attacker.val >= 0 && attacker.val <= maxKillVal;
                if (!canKill) {
                    addLog('combat', `👑 【真王】威壓：免疫了 [${attacker.isHero ? attacker.name : (attacker.val === 12 ? '國王' : attacker.name)}] 的打擊！真王在${defender.isAwakened ? '覺醒後僅可被 0-3' : '未覺醒時僅可被 0-6'} 數字牌殺死。`);
                    defenderDead = false;
                }
            }
            if (attackerDead && attacker.isHero && attacker.heroId === 'true_king') {
                const maxKillVal = attacker.isAwakened ? 3 : 6;
                const canKill = !defender.isHero && defender.val >= 0 && defender.val <= maxKillVal;
                if (!canKill) {
                    addLog('combat', `👑 【真王】威壓：免疫了 [${defender.isHero ? defender.name : (defender.val === 12 ? '國王' : defender.name)}] 的反噬！真王在${attacker.isAwakened ? '覺醒後僅可被 0-3' : '未覺醒時僅可被 0-6'} 數字牌殺死。`);
                    attackerDead = false;
                }
            }
            
            // 憤怒者覺醒護體 (免疫 0-3)
            if (defenderDead && defender.isHero && defender.heroId === 'angry_one' && defender.isAwakened) {
                const is0to3 = !attacker.isHero && attacker.val >= 0 && attacker.val <= 3;
                if (is0to3) {
                    addLog('combat', `😡 【憤怒者】覺醒護體：免疫了 [${attacker.isRevealed ? attacker.name : '暗牌'}] 的致死打擊！`);
                    defenderDead = false;
                }
            }
            if (attackerDead && attacker.isHero && attacker.heroId === 'angry_one' && attacker.isAwakened) {
                const is0to3 = !defender.isHero && defender.val >= 0 && defender.val <= 3;
                if (is0to3) {
                    addLog('combat', `😡 【憤怒者】覺醒護體：免疫了 [${defender.isRevealed ? defender.name : '暗牌'}] 的反噬！`);
                    attackerDead = false;
                }
            }

            // 天神被動：擊殺數字牌獲得雷電
            if (attacker.isHero && attacker.heroId === 'deity' && !attackerDead && defenderDead && defender.val !== 12 && !defender.isHero) {
                gameState.spells[attacker.player].push('thunder');
                addLog('sys', `⚡ 天神擊殺了數字牌，P${attacker.player} 獲得了 1 張【雷電】法術！`);
            }
            if (defender.isHero && defender.heroId === 'deity' && !defenderDead && attackerDead && attacker.val !== 12 && !attacker.isHero) {
                gameState.spells[defender.player].push('thunder');
                addLog('sys', `⚡ 天神反殺了數字牌，P${defender.player} 獲得了 1 張【雷電】法術！`);
            }

            // 真王雷雨天天氣特效
            if (attacker.isHero && attacker.heroId === 'true_king' && attacker.isAwakened && !attackerDead && defenderDead && gameState.currentWeather === '雷雨') {
                triggerTrueKingThunder(attacker.player);
            }
            if (defender.isHero && defender.heroId === 'true_king' && defender.isAwakened && !defenderDead && attackerDead && gameState.currentWeather === '雷雨') {
                triggerTrueKingThunder(defender.player);
            }

            return { attackerDead, defenderDead };
        }

        function drawCard(player) {
            const nonHeroCount = gameState.hands[player].filter(c => !c.isHero).length;
            if (nonHeroCount >= 7) {
                addLog(player === 1 ? 'p1' : 'p2', `P${player} 手牌已達上限 (7張)，暫停抽牌。`);
                return;
            }
            
            if (gameState.decks[player].length > 0) {
                const drawn = gameState.decks[player].pop();
                gameState.hands[player].push(drawn);
                addLog(player === 1 ? 'p1' : 'p2', `P${player} 從牌庫抽了 1 張手牌。`);
            }
        }

        function endBattleTurn() {
            if (gameState.currentPlayer === 2) {
                gameState.turnsWithoutDeath++;
                if (gameState.turnsWithoutDeath >= 8) {
                    addLog('sys', '🕊️ 連續 8 回合無單位陣亡，觸發和平協議！');
                    triggerVictory('DRAW', '連續 8 回合雙方無卡牌陣亡，遊戲平局！');
                    return;
                }
            }

            if (gameState.riverDuration > 0) {
                gameState.riverDuration--;
                if (gameState.riverDuration === 0) {
                    addLog('sys', '🌊 河流效果已結束，水流退去。');
                    
                    // 海王暗流絞殺
                    for (let p = 1; p <= 2; p++) {
                        if (gameState.seaKingAwakened && gameState.seaKingAwakened[p]) {
                            let skAlive = false;
                            for (let r = 0; r < ROWS; r++) {
                                for (let c = 0; c < COLS; c++) {
                                    let card = gameState.board[r][c];
                                    if (card && card.player === p && card.isHero && card.heroId === 'sea_king') skAlive = true;
                                }
                            }
                            if (skAlive) {
                                addLog('combat', `🌊 【海王】覺醒發威：河流退潮引發深淵暗流，準備絞殺敵方前排！`);
                                let enemy = p === 1 ? 2 : 1;
                                let targetRow = enemy === 1 ? ROWS - 3 : 2;
                                let killedCount = 0;
                                for (let c = 0; c < COLS; c++) {
                                    let target = gameState.board[targetRow][c];
                                    if (target && target.player === enemy && !target.isHero && target.val >= 0 && target.val <= 6) {
                                        addToGraveyard(target);
                                        gameState.board[targetRow][c] = null;
                                        addLog('combat', `🌊 暗流無情地絞殺了 [${target.isRevealed ? target.name : '暗牌'}]！`);
                                        killedCount++;
                                    }
                                }
                                if (killedCount > 0) gameState.turnsWithoutDeath = 0;
                            }
                        }
                    }
                }
            }

            // Process Tear Bridges
            for (let i = gameState.tearBridges.length - 1; i >= 0; i--) {
                let tb = gameState.tearBridges[i];
                if (tb.player === gameState.currentPlayer && tb.duration !== Infinity) {
                    tb.duration--;
                    if (tb.duration <= 0) {
                        collapseTearBridge(i);
                    }
                }
            }
            
            // 僅遞減當前回合結束玩家的隱謐狀態
            gameState.stealthTurns[gameState.currentPlayer] = Math.max(0, gameState.stealthTurns[gameState.currentPlayer] - 1);

            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    const card = gameState.board[r][c];
                    // 僅遞減當前回合結束玩家的卡牌狀態
                    if (card && card.player === gameState.currentPlayer) {
                        if (card.puppetTurns > 0) {
                            card.puppetTurns--;
                            if (card.puppetTurns <= 0) {
                                card.player = card.originalPlayer;
                                card.cantAttackKing = false;
                                addLog('sys', `🎭 [${card.name}] 掙脫了傀儡控制，回歸 P${card.player} 陣營！`);
                            }
                        }
                        if (card.isFrozen && card.freezeTimer !== undefined) {
                            card.freezeTimer--;
                            if (card.freezeTimer <= 0) {
                                card.isFrozen = false;
                                addLog('sys', `♨️ [${card.isHero ? card.name : card.name}] 的冰凍狀態已隨時間解除。`);
                            }
                        }
                        if (card.berserkTurns > 0) {
                            card.berserkTurns--;
                            if (card.berserkTurns === 0) {
                                card.berserkBuff = 0;
                                addLog('sys', `💢 [${card.name}] 狂暴狀態結束，攻擊力恢復正常。`);
                            }
                        }
                        if (card.invincibleTurns > 0) {
                            card.invincibleTurns--;
                            if (card.invincibleTurns > 0) {
                                card.invinciblePhase++;
                            } else {
                                card.invinciblePhase = 0;
                                addLog('sys', `🛡️ [${card.name}] 的無敵狀態已解除。`);
                            }
                        }
                    }
                }
            }

            const endingPlayer = gameState.currentPlayer;
            
            // 清除未使用的傑局淚橋控制法術
            const toggleIdx = gameState.spells[endingPlayer].indexOf('toggleBridge');
            if (toggleIdx !== -1) {
                gameState.spells[endingPlayer].splice(toggleIdx, 1);
                addLog('sys', `💨 P${endingPlayer} 的【控制淚橋】機會因回合結束而消散。`);
            }
            
            if (gameState.seaKingAwakened && gameState.seaKingAwakened[endingPlayer]) {
                let skAlive = false;
                for(let r=0;r<ROWS;r++) for(let c=0;c<COLS;c++) {
                    let card = gameState.board[r][c];
                    if(card && card.player === endingPlayer && card.isHero && card.heroId === 'sea_king') skAlive = true;
                }
                if (skAlive) {
                    gameState.seaKingTurnCounter[endingPlayer]++;
                    if (gameState.seaKingTurnCounter[endingPlayer] >= 4) {
                        gameState.seaKingTurnCounter[endingPlayer] = 0;
                        gameState.riverDuration = 3;
                        gameState.riverFrozen = false;
                        addLog('sys', `🌊 【海王】神力湧動：每 4 回合自動召喚，波濤洶湧的河流再次升起！`);
                    }
                }
            }

            gameState.currentPlayer = gameState.currentPlayer === 1 ? 2 : 1;
            const nextPlayer = gameState.currentPlayer;

            // 傑局覺醒充能邏輯
            if (gameState.jiejuAwakened[nextPlayer]) {
                let jiejuAlive = false;
                for(let r=0;r<ROWS;r++) for(let c=0;c<COLS;c++) {
                    let card = gameState.board[r][c];
                    if(card && card.player === nextPlayer && card.isHero && card.heroId === 'jieju') jiejuAlive = true;
                }
                if (jiejuAlive) {
                    gameState.jiejuTurnCounter[nextPlayer]++;
                    if (gameState.jiejuTurnCounter[nextPlayer] >= 3) {
                        gameState.jiejuTurnCounter[nextPlayer] = 0;
                        gameState.pendingJiejuSpellPlayer = nextPlayer;
                        addLog('sys', `🎛️ 【傑局】充能完畢！獲得 1 次控制淚橋的機會（請選擇是否立即發動）！`);
                    }
                }
            }

            if (gameState.currentPlayer === 1) {
                gameState.turnCount++;
                
                // 每 12 回合轉換一次天氣
                if (gameState.turnCount > 1 && (gameState.turnCount - 1) % 12 === 0) {
                    let nextWeather = gameState.currentWeather;
                    // 確保一定會換成不一樣的天氣
                    while (nextWeather === gameState.currentWeather) {
                        nextWeather = WEATHER_TYPES[Math.floor(Math.random() * WEATHER_TYPES.length)];
                    }
                    gameState.currentWeather = nextWeather;
                    addLog('sys', `🌤️ 氣象異象！每 12 回合的天氣週期已到，當前天氣轉變為：【${gameState.currentWeather}】。`);
                    showToast(`天氣變換為：${gameState.currentWeather}`);
                }

                // 冬天效果：每 4 個回合隨機冰凍
                if (gameState.currentWeather === '冬天' && gameState.turnCount % 4 === 0) {
                    let frozenCount = 0;
                    let unfrozenUnits = [];
                    for(let r=0; r<ROWS; r++){
                        for(let c=0; c<COLS; c++){
                            let unit = gameState.board[r][c];
                            if (unit) {
                                if (unit.isFrozen) {
                                    frozenCount++;
                                } else {
                                    // 檢查該部隊之陣營是否有海王庇護（免疫控制）
                                    let hasSeaKing = false;
                                    for (let sr = 0; sr < ROWS; sr++) {
                                        for (let sc = 0; sc < COLS; sc++) {
                                            let heroCheck = gameState.board[sr][sc];
                                            if (heroCheck && heroCheck.player === unit.player && heroCheck.isHero && heroCheck.heroId === 'sea_king') {
                                                hasSeaKing = true;
                                            }
                                        }
                                    }
                                    // 只有非海王庇護的單位才會加入可被冰凍的名單
                                    if (!hasSeaKing) {
                                        unfrozenUnits.push(unit);
                                    }
                                }
                            }
                        }
                    }
                    if (frozenCount < 2 && unfrozenUnits.length > 0) {
                        let target = unfrozenUnits[Math.floor(Math.random() * unfrozenUnits.length)];
                        target.isFrozen = true;
                        target.freezeTimer = 8; // 預設 8 回合
                        addLog('sys', `❄️ 冬天效果：刺骨寒風隨機將 [${target.isHero ? target.name : target.name}] 冰凍了！`);
                    }
                }
            }

            const p = gameState.currentPlayer;
            drawCard(p);
            
            saveGameStateHistory();

            // 回合初技能彈窗觸發判定
            if (gameState.mageAwakened[p] && isMageAlive(p)) {
                gameState.pendingMageSpellTrigger = true;
            }

            if (gameState.gameMode === '2P') {
                showPassDeviceModal(p);
            } else {
                checkStartOfTurnPopups(p);
            }
        }

        function isMageAlive(player) {
            for(let r=0; r<ROWS; r++){
                for(let c=0; c<COLS; c++){
                    let card = gameState.board[r][c];
                    if(card && card.player === player && card.isHero && card.heroId === 'mage') return true;
                }
            }
            return false;
        }

        function checkStartOfTurnPopups(p) {
            if (gameState.pendingMageSpellTrigger) {
                gameState.pendingMageSpellTrigger = false;
                triggerMageAwakenedSpell(p);
                return;
            }
            if (gameState.pendingJiejuSpellPlayer === p) {
                gameState.pendingJiejuSpellPlayer = null;
                triggerJiejuAwakenedSpell(p);
                return;
            }
            
            updateUI();
            render();
            if (gameState.gameMode === 'VS_AI' && p === 2 && !gameState.winner) {
                triggerAiTurn();
            }
        }

        function triggerMageAwakenedSpell(player) {
            let availableSpells = [...ALL_SPELL_KEYS];
            gameState.pendingMageSpell = availableSpells[Math.floor(Math.random() * availableSpells.length)];
            
            if (gameState.gameMode === 'VS_AI' && player === 2) {
                gameState.spells[2].push(gameState.pendingMageSpell);
                addLog('p2', `🤖 AI 的法師覺醒，自動獲得了臨時法術【${SPELL_DEFINITIONS[gameState.pendingMageSpell].name}】！`);
                gameState.pendingMageSpell = null;
                checkStartOfTurnPopups(player);
                return;
            }

            const modal = document.getElementById('mage-spell-modal');
            const spellName = document.getElementById('mage-spell-name');
            const spellDesc = document.getElementById('mage-spell-desc');
            const spellIcon = document.getElementById('mage-spell-icon');
            
            let spDef = SPELL_DEFINITIONS[gameState.pendingMageSpell];
            spellName.textContent = spDef.name;
            spellDesc.textContent = spDef.desc;
            spellIcon.className = `fa-solid ${spDef.icon} text-4xl text-purple-400 mb-3`;
            
            modal.classList.remove('opacity-0', 'pointer-events-none');
            modal.querySelector('#mage-modal-box').classList.remove('scale-90');
        }

        function acceptMageSpell() {
            closeMageModal();
            const p = gameState.currentPlayer;
            if(gameState.pendingMageSpell) {
                gameState.spells[p].push(gameState.pendingMageSpell);
                let idx = gameState.spells[p].length - 1;
                gameState.pendingMageSpell = null;
                
                gameState.activeTab = 'spell';
                gameState.mageForcedSpellIndex = idx; // 標記為強制法術
                
                checkStartOfTurnPopups(p);
                selectSpellToCast(idx);
                showToast("請立即點擊目標施放！若取消將會失去此法術。");
            }
        }

        function declineMageSpell() {
            closeMageModal();
            addLog(gameState.currentPlayer === 1 ? 'p1' : 'p2', `💨 放棄了法師湧現的臨時法術。`);
            gameState.pendingMageSpell = null;
            checkStartOfTurnPopups(gameState.currentPlayer);
        }

        function closeMageModal() {
            const modal = document.getElementById('mage-spell-modal');
            modal.classList.add('opacity-0', 'pointer-events-none');
            modal.querySelector('#mage-modal-box').classList.add('scale-90');
        }

        function triggerJiejuAwakenedSpell(player) {
            if (gameState.gameMode === 'VS_AI' && player === 2) {
                gameState.spells[2].push('toggleBridge');
                addLog('p2', `🤖 AI 的傑局充能完畢，獲得了臨時法術【控制淚橋】！`);
                checkStartOfTurnPopups(player);
                return;
            }

            const modal = document.getElementById('jieju-spell-modal');
            modal.classList.remove('opacity-0', 'pointer-events-none');
            modal.querySelector('#jieju-modal-box').classList.remove('scale-90');
        }

        function closeJiejuModal() {
            const modal = document.getElementById('jieju-spell-modal');
            modal.classList.add('opacity-0', 'pointer-events-none');
            modal.querySelector('#jieju-modal-box').classList.add('scale-90');
        }

        function storeJiejuSpell() {
            closeJiejuModal();
            const p = gameState.currentPlayer;
            gameState.spells[p].push('toggleBridge');
            addLog(p === 1 ? 'p1' : 'p2', `🌉 暫存了【控制淚橋】，可於本回合任意時機施放。`);
            checkStartOfTurnPopups(p);
        }

        function acceptJiejuSpell() {
            closeJiejuModal();
            const p = gameState.currentPlayer;
            gameState.spells[p].push('toggleBridge');
            let idx = gameState.spells[p].length - 1;
            
            gameState.activeTab = 'spell';
            addLog(p === 1 ? 'p1' : 'p2', `🌉 決定立即執行【控制淚橋】！`);
            checkStartOfTurnPopups(p);
            selectSpellToCast(idx); 
        }

        /* --- Strategic AI Engine --- */
        function triggerAiTurn() {
            const statusElem = document.getElementById('turn-phase');
            if (statusElem) statusElem.textContent = '🤖 AI 正在進行 5 回合戰術推演 (5-7秒)...';

            let count = 0;
            const thinkingInterval = setInterval(() => {
                count++;
                if (count === 1) addLog('p2', '🤖 AI [全局帳本]: 更新盤面控制權與雙方資源推算...');
                if (count === 3) addLog('p2', '🤖 AI [全知視角]: 掃描敵方所有暗牌、真實戰力與手牌資訊...');
                if (count === 5) addLog('p2', '🤖 AI [深思熟慮]: 執行上帝視角 Minimax 演繹與英雄戰術推演...');
            }, 1000);

            setTimeout(() => {
                clearInterval(thinkingInterval);
                executeAiChoice();
            }, 5500);
        }

        function executeAiChoice() {
            if (gameState.winner || gameState.phase !== GAME_PHASES.BATTLE || gameState.currentPlayer !== 2) return;

            let possibleActions = [];

            const handPlacementsForCard = (idx) => getValidPlacementsForHand(2, idx);
            for (let i = 0; i < gameState.hands[2].length; i++) {
                let placements = handPlacementsForCard(i);
                for (let p of placements) {
                    possibleActions.push({
                        type: 'hand',
                        handIndex: i,
                        r: p.r,
                        c: p.c,
                        card: gameState.hands[2][i]
                    });
                }
            }

            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    const card = gameState.board[r][c];
                    if (card && card.player === 2) {
                        const moves = getValidMovesForCard(r, c);
                        for (let m of moves) {
                            possibleActions.push({
                                type: 'board',
                                fromR: r,
                                fromC: c,
                                r: m.r,
                                c: m.c,
                                moveType: m.type,
                                card: card
                            });
                        }
                    }
                }
            }

            // AI considers Spells
            for (let i = 0; i < gameState.spells[2].length; i++) {
                const spellKey = gameState.spells[2][i];
                if (['scythe', 'mimic'].includes(spellKey)) continue; // AI skip complex multi-step spells for now

                if (['river', 'drawSpells', 'stealth4', 'toggleBridge'].includes(spellKey)) {
                    // drawSpells is checked in depth during scoring for Mage awakening
                    possibleActions.push({ type: 'spell_special', spellKey, spellIndex: i });
                } else if (spellKey === 'reveal') {
                    possibleActions.push({ type: 'spell_special', spellKey, spellIndex: i, targetPlayer: 1 });
                } else if (spellKey === 'revive') {
                    if (gameState.graveyards[2].some(c => !c.isHero)) {
                        const rows = getSetupRows(2);
                        let emptyCell = null;
                        for (let sr of rows) {
                            for (let sc = 0; sc < COLS; sc++) {
                                if (isValidCell(sr, sc) && !gameState.board[sr][sc]) emptyCell = {r: sr, c: sc};
                            }
                        }
                        if (emptyCell) {
                            possibleActions.push({ type: 'spell_special', spellKey, spellIndex: i, r: emptyCell.r, c: emptyCell.c });
                        }
                    }
                } else {
                    const targets = getValidSpellTargets(spellKey, 2);
                    for (let t of targets) {
                        possibleActions.push({ type: 'spell', spellKey, spellIndex: i, r: t.r, c: t.c });
                    }
                }
            }

            if (possibleActions.length === 0) {
                addLog('p2', '🤖 AI 無可行行動，跳過回合。');
                endBattleTurn();
                return;
            }

            let aiEnemyTerritoryCards = 0;
            for (let r of [ROWS - 3, ROWS - 2, ROWS - 1]) {
                for (let c = 0; c < COLS; c++) {
                    if (gameState.board[r][c] && gameState.board[r][c].player === 2) {
                        aiEnemyTerritoryCards++;
                    }
                }
            }

            let bestAction = null;
            let maxScore = -99999;

            for (let action of possibleActions) {
                let score = 5.0;

                // Death zone constraint check for AI
                if (action.r === ROWS - 2 && (action.type === 'board' || action.type === 'hand')) {
                    const target = gameState.board[action.r][action.c];
                    const isKingKill = target && target.player === 1 && (target.val === 12 || (target.isHero && target.heroId === 'true_king'));
                    if (aiEnemyTerritoryCards <= 2 && !isKingKill) {
                        score -= 8.0;
                    }
                }
                
                // Avoid Traps
                if (gameState.traps.some(t => t.r === action.r && t.c === action.c)) {
                    score -= 100.0;
                }
                
                // Avoid Fireballs if value <= 6
                if (gameState.fireballs.some(fb => fb.r === action.r && fb.c === action.c) && action.card && action.card.val <= 6) {
                    score -= 100.0;
                }

                // Avoid Drowning in River
                if (action.r === 3 && gameState.riverDuration > 0 && !gameState.riverFrozen && action.type === 'board') {
                    score -= 100.0;
                }

                if (action.type === 'board') {
                    const target = gameState.board[action.r][action.c];
                    
                    if (action.moveType === 'unpetrify') {
                        score += 30.0; // AI 優先解石化
                    } else if (action.moveType === 'attack' && target) {
                        
                        let aBaseAi = action.card.isHero ? 12 : action.card.val;
                        let dBaseAi = target.isHero ? 0 : target.val;

                        if (gameState.centurionAwakened[2] && gameState.lastDeadCardValue !== null) {
                            aBaseAi = gameState.lastDeadCardValue;
                        }
                        if (gameState.centurionAwakened[target.player] && gameState.lastDeadCardValue !== null) {
                            dBaseAi = gameState.lastDeadCardValue;
                        }

                        let aBuffAi = (action.card.val !== 0 && !action.card.isHero && action.card.val !== 12) ? gameState.playerAtkBuff[2] : 0;
                        let aValAi = aBaseAi + (action.card.berserkBuff || 0) + aBuffAi + (action.card.soloDanceBuff || 0);
                        if (!action.card.isHero && action.card.val !== 12 && gameState.angryOneAwakened[2] && aValAi > 11) aValAi = 11;
                        
                        let dValAi = dBaseAi + (target.berserkBuff || 0) + (target.soloDanceBuff || 0) - (target.defPenalty || 0);
                        if (dValAi < 0) dValAi = 0;

                        let aiWins = false;
                        let aiDies = false;

                        if (target.val === 12 && !target.isHero && target.kingShield > 0) { aiDies = true; }
                        else if ((action.card.val === 0 && !action.card.isHero && target.val === 11 && !target.isHero) || (action.card.val === 11 && !action.card.isHero && target.val === 0 && !target.isHero)) { aiDies = true; aiWins = true; }
                        else if (action.card.val === 0 && !action.card.isHero && target.val === 0 && !target.isHero) { aiDies = true; aiWins = true; }
                        else if (target.val === 12 && !target.isHero) { aiDies = true; aiWins = true; }
                        else if (action.card.val === 12 && !action.card.isHero) { aiWins = true; }
                        else if (aValAi > dValAi) { aiWins = true; }
                        else if (aValAi < dValAi) { aiDies = true; }
                        else { aiWins = true; aiDies = false; } // 數值相同：主動攻擊方獲勝

                        // 真王與憤怒者免疫判定
                        if (aiWins && target.isHero && target.heroId === 'true_king') {
                            const maxKillVal = target.isAwakened ? 3 : 6;
                            const canKill = !action.card.isHero && action.card.val >= 0 && action.card.val <= maxKillVal;
                            if (!canKill) aiWins = false;
                        }
                        if (aiDies && action.card.isHero && action.card.heroId === 'true_king') {
                            const maxKillVal = action.card.isAwakened ? 3 : 6;
                            const canKill = !target.isHero && target.val >= 0 && target.val <= maxKillVal;
                            if (!canKill) aiDies = false;
                        }
                        if (aiWins && target.isHero && target.heroId === 'angry_one' && target.isAwakened) {
                            const is0to3 = !action.card.isHero && action.card.val >= 0 && action.card.val <= 3;
                            if (is0to3) aiWins = false;
                        }
                        if (aiDies && action.card.isHero && action.card.heroId === 'angry_one' && action.card.isAwakened) {
                            const is0to3 = !target.isHero && target.val >= 0 && target.val <= 3;
                            if (is0to3) aiDies = false;
                        }
                        
                        if (target.invincibleTurns > 0 || target.isInvincible) aiWins = false;
                        if (action.card.invincibleTurns > 0 || action.card.isInvincible) aiDies = false;

                        if (aiWins && !aiDies) {
                            if (target.val === 12 || (target.isHero && target.heroId === 'true_king')) score += 1000.0; // 必勝
                            else score += 15.0 + (target.val * 0.8);
                        } else if (aiWins && aiDies) {
                            if (target.val === 12 || (target.isHero && target.heroId === 'true_king')) score += 1000.0;
                            if (action.card.val < target.val) score += 8.0; // 賺翻
                            else score -= 5.0; // 虧本
                        } else if (!aiWins && aiDies) {
                            score -= 50.0; // 自殺
                        } else {
                            score -= 2.0; // 撞牆
                        }
                    } else if (action.moveType === 'move') {
                        if (action.r > action.fromR) score += 1.5;
                        if (action.r === ROWS - 2) score += 3.0; // 逼近底線
                    }
                } else if (action.type === 'hand') {
                    score += 2.0;
                    if (action.card.isHero) {
                        if (action.card.heroId === 'true_king') score += 200.0; // 必下真王
                        else if (action.card.heroId === 'deity') {
                            let killCount = 0;
                            for(let r=0; r<ROWS; r++){
                                for(let c=0; c<COLS; c++){
                                    let t = gameState.board[r][c];
                                    if (t && t.player === 1 && !t.isRevealed && t.val !== 12) killCount++;
                                }
                            }
                            score += 30.0 + (killCount * 10.0);
                        }
                        else if (action.card.heroId === 'angry_one') score += 40.0;
                        else if (action.card.heroId === 'mage') score += 35.0;
                    }
                } else if (action.type.startsWith('spell')) {
                    let baseScore = 5.0;
                    const target = action.r !== undefined ? gameState.board[action.r][action.c] : null;

                    switch (action.spellKey) {
                        case 'thunder':
                            if (target && target.player === 2 && target.isHero && target.heroId === 'deity' && !target.isAwakened) {
                                baseScore = 60.0; // 優先覺醒天神
                            } else if (target && target.player === 1 && !target.isRevealed) {
                                baseScore = 15.0 + (target.val * 1.5); // 【全知視角】專劈大牌
                                if (target.isHero) baseScore += 20.0;
                            }
                            break;
                        case 'berserk':
                            if (target && target.player === 2 && target.isHero && target.heroId === 'angry_one' && !target.isAwakened) {
                                baseScore = 60.0; // 優先覺醒憤怒者
                            } else if (target && target.player === 2) {
                                baseScore = 8.0 + (target.val * 0.5);
                            }
                            break;
                        case 'expedition':
                            if (target && target.player === 2 && target.isHero && target.heroId === 'true_king' && !target.isAwakened) {
                                baseScore = 80.0; // 覺醒真王最高優先
                            } else if (target && target.player === 2) {
                                baseScore = 15.0;
                            }
                            break;
                        case 'drawSpells':
                            let mageToAwaken = false;
                            for (let r=0; r<ROWS; r++) {
                                for (let c=0; c<COLS; c++) {
                                    let t = gameState.board[r][c];
                                    if (t && t.player === 2 && t.isHero && t.heroId === 'mage' && !t.isAwakened) mageToAwaken = true;
                                }
                            }
                            if (mageToAwaken) baseScore = 60.0; // 優先覺醒法師
                            else if (gameState.graveyards[2].length >= 4) baseScore = 15.0;
                            else baseScore = -999.0;
                            break;
                        case 'fire':
                            baseScore = (target && target.val > 6) ? 12.0 : 6.0;
                            break;
                        case 'invincible3':
                        case 'hat':
                            baseScore = (target && target.val >= 8) ? 12.0 : 5.5;
                            break;
                        case 'guandao':
                            baseScore = (target && target.val >= 7) ? 10.0 : 5.0;
                            break;
                        case 'reveal':
                        case 'stealth4':
                            baseScore = 8.0;
                            break;
                        case 'revive':
                            baseScore = 15.0;
                            break;
                        case 'trap':
                        case 'tunnel':
                            baseScore = action.r >= ROWS - 3 ? 9.0 : 4.0;
                            break;
                        case 'puppet':
                            baseScore = (target && target.val >= 5) ? 12.0 : 6.0;
                            break;
                        case 'storm':
                            baseScore = 10.0;
                            break;
                        case 'tearBridge':
                            baseScore = 8.0;
                            break;
                        case 'obelisk':
                            baseScore = 9.0;
                            break;
                        case 'toggleBridge':
                            let aiTbIdx = gameState.tearBridges.findIndex(tb => tb.isJieju && tb.player === 2);
                            if (aiTbIdx !== -1) {
                                let tb = gameState.tearBridges[aiTbIdx];
                                let enemies = 0, allies = 0;
                                for (let cc = 0; cc < COLS; cc++) {
                                    if (gameState.board[tb.row][cc] && gameState.board[tb.row][cc].player === 1) enemies++;
                                    if (gameState.board[tb.row][cc] && gameState.board[tb.row][cc].player === 2) allies++;
                                }
                                baseScore = (enemies > allies) ? 80.0 : -50.0;
                            } else {
                                baseScore = 20.0; // 若關閉，AI傾向重新開啟獲取空間
                            }
                            break;
                        case 'rewind':
                            baseScore = 7.0; // AI 不擅長判斷回溯時機，給予中等權重
                            break;
                        case 'petrify':
                            baseScore = (target && target.val >= 7) ? 14.0 : 6.0;
                            break;
                        case 'soloDance':
                            baseScore = 15.0;
                            break;
                        default:
                            baseScore = 6.0;
                    }
                    score += baseScore;
                }

                if (score > maxScore) {
                    maxScore = score;
                    bestAction = action;
                }
            }

            if (!bestAction) bestAction = possibleActions[0];

            addLog('p2', `🤖 AI 最終抉擇 [優勢標尺: ${maxScore.toFixed(1)}/10]: 執行最優行動。`);

            if (bestAction.type === 'hand') {
                gameState.selectedPos = { type: 'hand', index: bestAction.handIndex };
                executeMove(gameState.selectedPos, { r: bestAction.r, c: bestAction.c, type: 'place' });
            } else if (bestAction.type === 'board') {
                gameState.selectedPos = { type: 'board', r: bestAction.fromR, c: bestAction.fromC };
                executeMove(gameState.selectedPos, { r: bestAction.r, c: bestAction.c, type: bestAction.moveType });
            } else if (bestAction.type === 'spell') {
                gameState.selectedSpellIndex = bestAction.spellIndex;
                executeSpell(bestAction.spellKey, bestAction.r, bestAction.c);
            } else if (bestAction.type === 'spell_special') {
                gameState.selectedSpellIndex = bestAction.spellIndex;
                if (bestAction.spellKey === 'revive') {
                    // AI 最優復活邏輯（復活數字最大的一張普通牌）
                    let revivable = gameState.graveyards[2].filter(c => !c.isHero);
                    revivable.sort((a, b) => a.val - b.val);
                    const targetCard = revivable.pop();
                    const actualIdx = gameState.graveyards[2].indexOf(targetCard);
                    const card = gameState.graveyards[2].splice(actualIdx, 1)[0];
                    
                    card.isRevealed = true;
                    gameState.board[bestAction.r][bestAction.c] = card;
                    addLog('p2', `📜 復活發動！[${card.name}] 放回戰場 (${bestAction.r}, ${bestAction.c})。`);
                    
                    const used = gameState.spells[2].splice(bestAction.spellIndex, 1)[0];
                    gameState.usedSpells[2].push(used);
                    cancelSelection();
                    checkWinConditions();
                    if (!gameState.winner) endBattleTurn();
                } else if (bestAction.spellKey === 'reveal') {
                    executeSpell('reveal', null, null, bestAction.targetPlayer);
                } else if (bestAction.spellKey === 'storm') {
                    let targets = getValidSpellTargets('storm', 2);
                    shuffleArray(targets);
                    gameState.stormTargets = targets.slice(0, 3).map(t => ({r: t.r, c: t.c, card: gameState.board[t.r][t.c]}));
                    executeSpell('storm', null, null);
                } else if (bestAction.spellKey === 'tearBridge' || bestAction.spellKey === 'toggleBridge') {
                    executeSpell(bestAction.spellKey, null, null);
                } else {
                    executeSpell(bestAction.spellKey, null, null);
                }
            }
        }

        /* --- Canvas Graphics Rendering Engine --- */
        function isCardVisibleToViewer(card) {
            if (!card) return false;
            if (card.isRevealed) return true;
            if (gameState.gameMode === 'VS_AI') {
                return card.player === 1;
            } else {
                return card.player === gameState.currentPlayer;
            }
        }

        function drawCardOnCanvas(card, x, y, w, h, isSelected) {
            ctx.save();

            const visible = isCardVisibleToViewer(card);

            if (isSelected) {
                ctx.shadowColor = '#f59e0b';
                ctx.shadowBlur = 12;
            }

            if (!visible) {
                ctx.fillStyle = '#334155';
                ctx.fillRect(x, y, w, h);

                ctx.strokeStyle = '#475569';
                ctx.lineWidth = 2;
                ctx.strokeRect(x, y, w, h);

                ctx.fillStyle = '#94a3b8';
                ctx.font = 'bold 20px Inter, sans-serif';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText('?', x + w / 2, y + h / 2);
            } else {
                const isP1 = card.player === 1;
                
                if (card.isHero) {
                    if (card.heroId === 'true_king') ctx.fillStyle = isP1 ? '#9a3412' : '#7f1d1d';
                    else if (card.heroId === 'deity') ctx.fillStyle = isP1 ? '#0369a1' : '#be185d';
                    else if (card.heroId === 'angry_one') ctx.fillStyle = isP1 ? '#b91c1c' : '#991b1b';
                    else if (card.heroId === 'mage') ctx.fillStyle = isP1 ? '#7e22ce' : '#9333ea';
                    else if (card.heroId === 'centurion') ctx.fillStyle = isP1 ? '#047857' : '#065f46';
                    else if (card.heroId === 'sea_king') ctx.fillStyle = isP1 ? '#0891b2' : '#0e7490';
                    else if (card.heroId === 'jieju') ctx.fillStyle = isP1 ? '#4338ca' : '#3730a3';
                } else {
                    ctx.fillStyle = isP1 ? '#1d4ed8' : '#be123c';
                }
                
                ctx.fillRect(x, y, w, h);

                ctx.strokeStyle = card.isRevealed ? '#f59e0b' : (isP1 ? '#60a5fa' : '#f87171');
                if (card.isHero && card.isAwakened) ctx.strokeStyle = '#c084fc'; // 覺醒紫框發光
                ctx.lineWidth = card.isRevealed ? 3 : 2;
                ctx.strokeRect(x, y, w, h);

                ctx.fillStyle = '#ffffff';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.font = 'bold 22px Inter, sans-serif';

                let displayVal = card.val + (card.berserkBuff || 0);
                let valText = card.val === 12 ? '👑' : displayVal.toString();
                if (card.isHero) {
                    if (card.heroId === 'true_king') valText = '👑真';
                    else if (card.heroId === 'deity') valText = '⚡神';
                    else if (card.heroId === 'angry_one') valText = '😡怒';
                    else if (card.heroId === 'mage') valText = '🧙法';
                    else if (card.heroId === 'centurion') valText = '🛡️百';
                    else if (card.heroId === 'sea_king') valText = '🌊海';
                    else if (card.heroId === 'jieju') valText = '🌉傑';
                } else if (card.isObelisk) {
                    valText = '⛩️碑';
                }
                
                ctx.fillText(valText, x + w / 2, y + h / 2 - 4);

                let badges = '';
                if (card.isFrozen) badges += '🧊';
                if (card.isInvincible) badges += '🛡️';
                if (card.invincibleTurns > 0) badges += `🛡️${card.invinciblePhase}`;
                if (card.guandaoBuff) badges += '🗡️';
                if (card.kingShield > 0) badges += '🔰';
                if (card.berserkTurns > 0) badges += '💢';
                if (card.isPetrified) badges += '🗿';
                if (gameState.stealthTurns[card.player] > 0 && card.val !== 12 && !card.isRevealed) badges += '🥷';
                if (gameState.playerAtkBuff[card.player] > 0 && !card.isHero && card.val !== 0 && card.val !== 12) badges += `⚔️+${gameState.playerAtkBuff[card.player]}`;
                if (card.soloDanceBuff > 0) badges += `💃+${card.soloDanceBuff}`;
                if (card.defPenalty > 0) badges += `📉-${card.defPenalty}`;

                ctx.font = '10px Inter, sans-serif';
                ctx.fillStyle = card.isRevealed ? '#fef08a' : '#cbd5e1';
                ctx.fillText(badges ? `${badges}` : (card.isRevealed ? '已知' : '暗牌'), x + w / 2, y + h - 10);
            }

            // Puppet Aura (Ghostly green border)
            if (card.puppetTurns > 0 && visible) {
                ctx.strokeStyle = '#22c55e';
                ctx.lineWidth = 3;
                ctx.setLineDash([5, 5]);
                ctx.strokeRect(x - 2, y - 2, w + 4, h + 4);
                ctx.setLineDash([]);
            }

            // Stealth Aura (Purple border effect)
            if (gameState.stealthTurns[card.player] > 0 && card.val !== 12 && !card.isRevealed) {
                ctx.strokeStyle = '#a855f7';
                ctx.lineWidth = 3;
                ctx.setLineDash([5, 5]);
                ctx.strokeRect(x - 2, y - 2, w + 4, h + 4);
                ctx.setLineDash([]);
            }
            
            // Awakened Hero Aura
            if (card.isHero && card.isAwakened && visible) {
                ctx.strokeStyle = '#d8b4fe';
                ctx.lineWidth = 2;
                ctx.setLineDash([2, 4]);
                ctx.strokeRect(x - 3, y - 3, w + 6, h + 6);
                ctx.setLineDash([]);
            }

            ctx.restore();
        }

        function render() {
            if (!ctx || !canvas) return;

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const cellW = canvas.width / COLS;
            const cellH = canvas.height / ROWS;

            // Draw Chess Grid Cells
            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    if (!isValidCell(r, c)) continue;

                    const x = c * cellW;
                    const y = r * cellH;

                    if (r === 0 || r === ROWS - 1) {
                        ctx.fillStyle = '#1e1b4b'; // Castle Throne Dark Purple
                    } else if (r < Math.floor(ROWS / 2)) {
                        ctx.fillStyle = (r + c) % 2 === 0 ? '#1e293b' : '#0f172a';
                    } else {
                        ctx.fillStyle = (r + c) % 2 === 0 ? '#172554' : '#0f172a';
                    }

                    ctx.fillRect(x + 2, y + 2, cellW - 4, cellH - 4);

                    ctx.strokeStyle = '#334155';
                    ctx.lineWidth = 1;
                    ctx.strokeRect(x + 2, y + 2, cellW - 4, cellH - 4);
                }
            }

            // Draw Active River Row
            if (gameState.riverDuration > 0) {
                const y = 3 * cellH;
                ctx.save();
                ctx.fillStyle = gameState.riverFrozen ? 'rgba(147, 197, 253, 0.6)' : 'rgba(14, 165, 233, 0.5)';
                ctx.fillRect(0, y + 4, canvas.width, cellH - 8);

                ctx.fillStyle = '#ffffff';
                ctx.font = 'bold 11px Inter, sans-serif';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(
                    gameState.riverFrozen ? `🧊 冰凍河流 (${gameState.riverDuration} 回合)` : `🌊 奔騰河流 (${gameState.riverDuration} 回合)`, 
                    canvas.width / 2, 
                    y + cellH / 2
                );
                ctx.restore();
            }

            // Draw Tear Bridges
            gameState.tearBridges.forEach(tb => {
                const y = tb.row * cellH;
                ctx.save();
                ctx.fillStyle = tb.isJieju ? 'rgba(79, 70, 229, 0.3)' : 'rgba(168, 162, 158, 0.4)';
                ctx.fillRect(0, y + 4, canvas.width, cellH - 8);

                ctx.fillStyle = '#ffffff';
                ctx.font = 'bold 11px Inter, sans-serif';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(tb.isJieju ? `🌉 專屬淚橋 (傑局死亡坍塌)` : `🌉 淚橋 (${tb.duration} 回合後坍塌)`, canvas.width / 2, y + cellH / 2);
                ctx.restore();
            });

            // Red Midline Boundary
            ctx.strokeStyle = '#ef4444';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(0, 3 * cellH);
            ctx.lineTo(canvas.width, 3 * cellH);
            ctx.stroke();

            // Draw Tunnels
            gameState.tunnels.forEach(t => {
                const x = t.c * cellW;
                const y = t.r * cellH;
                
                if (tunnelImg.complete && tunnelImg.naturalWidth > 0) {
                    ctx.drawImage(tunnelImg, x + 4, y + 4, cellW - 8, cellH - 8);
                    ctx.strokeStyle = '#a855f7';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(x + 4, y + 4, cellW - 8, cellH - 8);
                } else {
                    ctx.fillStyle = 'rgba(168, 85, 247, 0.2)'; // purple bg
                    ctx.fillRect(x + 4, y + 4, cellW - 8, cellH - 8);
                    ctx.fillStyle = '#d8b4fe';
                    ctx.font = '24px Inter';
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'middle';
                    ctx.fillText('🕳️', x + cellW / 2, y + cellH / 2);
                }
            });

            // Draw Traps
            gameState.traps.forEach(t => {
                const x = t.c * cellW;
                const y = t.r * cellH;
                ctx.fillStyle = 'rgba(239, 68, 68, 0.2)'; // red bg
                ctx.fillRect(x + 4, y + 4, cellW - 8, cellH - 8);
                ctx.fillStyle = '#fca5a5';
                ctx.font = '24px Inter';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText('☢️', x + cellW / 2, y + cellH / 2);
            });
            
            // Draw Fireballs
            gameState.fireballs.forEach(fb => {
                const x = fb.c * cellW;
                const y = fb.r * cellH;
                ctx.fillStyle = 'rgba(239, 68, 68, 0.3)';
                ctx.fillRect(x + 4, y + 4, cellW - 8, cellH - 8);
                ctx.fillStyle = '#f97316';
                ctx.font = '24px Inter';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText('🔥', x + cellW / 2, y + cellH / 2);
            });

            // Highlight Storm Targets
            gameState.stormTargets.forEach(t => {
                const x = t.c * cellW;
                const y = t.r * cellH;
                ctx.fillStyle = 'rgba(168, 85, 247, 0.4)'; // Purple highlight
                ctx.fillRect(x + 4, y + 4, cellW - 8, cellH - 8);
            });

            // Highlight Valid Moves
            for (let m of gameState.validMoves) {
                const x = m.c * cellW;
                const y = m.r * cellH;
                ctx.fillStyle = m.type === 'attack' ? 'rgba(239, 68, 68, 0.4)' : 'rgba(245, 158, 11, 0.3)';
                ctx.fillRect(x + 4, y + 4, cellW - 8, cellH - 8);
                ctx.strokeStyle = m.type === 'attack' ? '#f87171' : '#f59e0b';
                ctx.lineWidth = 2;
                ctx.strokeRect(x + 4, y + 4, cellW - 8, cellH - 8);
            }

            // Render Cards on Board
            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    if (!isValidCell(r, c)) continue;

                    const card = gameState.board[r][c];
                    if (card) {
                        const x = c * cellW + 6;
                        const y = r * cellH + 6;
                        const w = cellW - 12;
                        const h = cellH - 12;

                        const isSelected = gameState.selectedPos && gameState.selectedPos.type === 'board'
                            && gameState.selectedPos.r === r && gameState.selectedPos.c === c;

                        drawCardOnCanvas(card, x, y, w, h, isSelected);
                    }
                }
            }
        }

        /* --- Victory Conditions --- */
        function checkWinConditions() {
            // 處理傑局死亡引發的連鎖坍塌 (可無限嵌套處理連鎖反應)
            while (gameState.jiejuDied) {
                let p = gameState.jiejuDied;
                gameState.jiejuDied = null; 
                let tbIndex = gameState.tearBridges.findIndex(tb => tb.isJieju && tb.player === p);
                if (tbIndex !== -1) {
                    addLog('sys', `💀 【傑局】陣亡，其專屬淚橋失去維持力，隨之崩塌！`);
                    collapseTearBridge(tbIndex);
                }
            }

            let p1King = false;
            let p2King = false;

            let p1Units = gameState.hands[1].length;
            let p2Units = gameState.hands[2].length;

            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    if (!isValidCell(r, c)) continue;
                    const card = gameState.board[r][c];
                    if (card) {
                        // 若場上有國王或真王存活
                        if (card.val === 12 || (card.isHero && card.heroId === 'true_king')) {
                            if (card.player === 1) p1King = true;
                            if (card.player === 2) p2King = true;
                        } else {
                            if (card.player === 1) p1Units++;
                            if (card.player === 2) p2Units++;
                        }
                    }
                }
            }

            if (!p1King && !p2King) {
                triggerVictory('DRAW', '雙方國王/真王同時陣亡，平局！');
            } else if (!p2King) {
                triggerVictory(1, '玩家 1 擊潰敵方國王或真王，贏得勝利！');
            } else if (!p1King) {
                triggerVictory(2, '玩家 2 擊潰敵方國王或真王，贏得勝利！');
            } else if (p1Units === 0 && p2Units === 0) {
                triggerVictory('DRAW', '雙方戰力耗盡，平局！');
            } else if (p2Units === 0) {
                triggerVictory(1, '玩家 2 已無任何戰力，玩家 1 獲勝！');
            } else if (p1Units === 0) {
                triggerVictory(2, '玩家 1 已無任何戰力，玩家 2 獲勝！');
            }
        }

        function triggerVictory(winner, desc) {
            gameState.winner = winner;
            gameState.phase = GAME_PHASES.END;
            soundEngine.playVictory();

            const modal = document.getElementById('victory-modal');
            const box = document.getElementById('victory-box');
            const title = document.getElementById('winner-title');
            const descElem = document.getElementById('winner-desc');

            if (title) title.textContent = winner === 'DRAW' ? '遊戲平局' : `玩家 ${winner} 獲勝！`;
            if (descElem) descElem.textContent = desc;

            if (modal && box) {
                modal.classList.remove('opacity-0', 'pointer-events-none');
                box.classList.remove('scale-90');
            }
        }

        /* --- UI Helpers & Confirm Modals --- */
        function shuffleArray(arr) {
            for (let i = arr.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [arr[i], arr[j]] = [arr[j], arr[i]];
            }
        }

        function showToast(msg) {
            const toast = document.getElementById('toast-message');
            const text = document.getElementById('toast-text');
            if (text) text.textContent = msg;
            if (toast) {
                toast.classList.remove('opacity-0');
                setTimeout(() => {
                    toast.classList.add('opacity-0');
                }, 2200);
            }
        }

        function confirmRestart() {
            const modal = document.getElementById('confirm-modal');
            if (modal) modal.classList.remove('opacity-0', 'pointer-events-none');
        }

        function closeConfirmModal() {
            const modal = document.getElementById('confirm-modal');
            if (modal) modal.classList.add('opacity-0', 'pointer-events-none');
        }

        function showPassDeviceModal(nextPlayerId) {
            const modal = document.getElementById('pass-device-modal');
            const nextText = document.getElementById('next-player-text');
            const readyLabel = document.getElementById('ready-player-label');

            if (nextText) nextText.textContent = `輪到 P${nextPlayerId} 的回合`;
            if (readyLabel) readyLabel.textContent = `P${nextPlayerId}`;

            if (modal) {
                modal.classList.remove('hidden');
            }
        }

        function confirmPassDevice() {
            const modal = document.getElementById('pass-device-modal');
            if (modal) {
                modal.classList.add('hidden');
            }
            checkStartOfTurnPopups(gameState.currentPlayer);
        }

        function addLog(type, text) {
            const logBox = document.getElementById('battle-log');
            if (!logBox) return;

            const entry = document.createElement('div');
            entry.className = 'p-1.5 rounded-lg border text-[11px] leading-relaxed transition-all';

            if (type === 'combat') {
                entry.className += ' bg-rose-950/40 border-rose-800/60 text-rose-200';
            } else if (type === 'p1') {
                entry.className += ' bg-blue-950/40 border-blue-800/60 text-blue-200';
            } else if (type === 'p2') {
                entry.className += ' bg-red-950/40 border-red-800/60 text-red-200';
            } else {
                entry.className += ' bg-slate-800/60 border-slate-700 text-amber-300';
            }

            entry.innerHTML = text;
            logBox.appendChild(entry);
            logBox.scrollTop = logBox.scrollHeight;
        }

        function clearLog() {
            const logBox = document.getElementById('battle-log');
            if (logBox) logBox.innerHTML = '';
        }

        /* --- Tear Bridge Helper Engine --- */
        function createTearBridge(p, isPermanent) {
            const insertIdx = 3; 
            gameState.board.splice(insertIdx, 0, [null, null, null]);
            ROWS = gameState.board.length; 
            
            gameState.traps.forEach(t => { if (t.r >= insertIdx) t.r++; });
            gameState.tunnels.forEach(t => { if (t.r >= insertIdx) t.r++; });
            gameState.fireballs.forEach(t => { if (t.r >= insertIdx) t.r++; });
            gameState.tearBridges.forEach(tb => { if (tb.row >= insertIdx) tb.row++; });
            
            gameState.tearBridges.push({ row: insertIdx, duration: isPermanent ? Infinity : 3, player: p, isJieju: isPermanent });
            resizeCanvas(); 
        }

        function collapseTearBridge(i) {
            let tb = gameState.tearBridges[i];
            addLog('sys', `🌉 淚橋崩塌！第 ${tb.row} 排墜入深淵，戰場縮回！`);
            for (let c = 0; c < COLS; c++) {
                let card = gameState.board[tb.row][c];
                if (card) {
                    addToGraveyard(card);
                    gameState.board[tb.row][c] = null;
                    gameState.turnsWithoutDeath = 0;
                }
            }
            
            const removeIdx = tb.row;
            gameState.board.splice(removeIdx, 1);
            ROWS = gameState.board.length;
            
            gameState.traps = gameState.traps.filter(t => t.r !== removeIdx);
            gameState.tunnels = gameState.tunnels.filter(t => t.r !== removeIdx);
            gameState.fireballs = gameState.fireballs.filter(t => t.r !== removeIdx);
            
            gameState.traps.forEach(t => { if (t.r > removeIdx) t.r--; });
            gameState.tunnels.forEach(t => { if (t.r > removeIdx) t.r--; });
            gameState.fireballs.forEach(t => { if (t.r > removeIdx) t.r--; });
            gameState.tearBridges.forEach(otherTb => { if (otherTb.row > removeIdx) otherTb.row--; });
            
            gameState.tearBridges.splice(i, 1);
            resizeCanvas();
        }

        /* --- Rules & Spells Dictionary UI --- */
        function switchRulesTab(tab) {
            soundEngine.playSelect();
            const tabs = ['rules', 'spells', 'heroes'];
            
            tabs.forEach(t => {
                const btn = document.getElementById(`modal-tab-${t}`);
                const content = document.getElementById(`modal-content-${t}`);
                
                if (!btn || !content) return;

                if (t === tab) {
                    btn.className = 'flex-1 py-2 px-1 rounded-lg bg-indigo-600 text-white text-xs md:text-sm font-bold transition shadow-md shadow-indigo-900/50 whitespace-nowrap';
                    content.classList.remove('hidden');
                    content.classList.add('block');
                } else {
                    btn.className = 'flex-1 py-2 px-1 rounded-lg bg-slate-800 text-slate-400 hover:bg-slate-700 hover:text-slate-200 text-xs md:text-sm font-bold transition border border-slate-700 whitespace-nowrap';
                    content.classList.add('hidden');
                    content.classList.remove('block');
                }
            });
        }

        function openRulesModal() {
            soundEngine.playSelect();
            const modal = document.getElementById('rules-modal');
            const box = document.getElementById('rules-modal-box');
            const spellList = document.getElementById('rules-spell-list');
            const heroList = document.getElementById('rules-hero-list');
            
            if (spellList && spellList.children.length === 0) {
                spellList.innerHTML = '<ul class="space-y-3">' + ALL_SPELL_KEYS.map((key, index) => {
                    const sp = SPELL_DEFINITIONS[key];
                    return `
                        <li class="p-3 bg-slate-800/40 border border-slate-700/80 rounded-xl flex items-start gap-3 hover:bg-slate-800 transition shadow-sm">
                            <div class="w-8 h-8 rounded-md bg-indigo-900/40 text-indigo-400 flex items-center justify-center shrink-0 mt-0.5 border border-indigo-700/50 shadow-inner">
                                <i class="fa-solid ${sp.icon} text-sm"></i>
                            </div>
                            <div class="flex-1">
                                <div class="font-bold text-indigo-300 text-sm mb-1">${index + 1}. ${sp.name}</div>
                                <div class="text-xs md:text-sm text-slate-300 leading-relaxed">${sp.desc}</div>
                            </div>
                        </li>
                    `;
                }).join('') + '</ul>';
            }
            
            if (heroList && heroList.children.length === 0) {
                heroList.innerHTML = `
                    <ul class="space-y-3">
                        <!-- 真王 -->
                        <li class="p-3 bg-slate-800/40 border border-slate-700/80 rounded-xl flex flex-col sm:flex-row items-start gap-3 hover:bg-slate-800 transition shadow-sm">
                            <div class="w-12 h-12 rounded-md bg-amber-900/40 text-amber-400 flex items-center justify-center shrink-0 border border-amber-700/50 shadow-inner text-2xl">
                                👑
                            </div>
                            <div class="flex-1">
                                <div class="font-bold text-amber-300 text-sm mb-1">真王 (True King)</div>
                                <div class="text-xs md:text-sm text-slate-300 leading-relaxed space-y-1">
                                    <p><span class="text-amber-400 font-semibold">基礎技能：</span>可於墓地超過 6 張牌時，直接部署替換國王。</p>
                                    <p><span class="text-amber-400 font-semibold">未覺醒狀態：</span>僅能被 0-6 數字牌擊殺。</p>
                                    <p><span class="text-amber-400 font-semibold">本命法術/覺醒：</span>對真王使用本命法術「御駕親征」後完成覺醒。</p>
                                    <p><span class="text-amber-400 font-semibold">覺醒強化：</span>獲得自由移動能力。致命弱點進一步縮小，僅能被 0-3 數字牌擊殺。</p>
                                </div>
                            </div>
                        </li>
                        <!-- 天神 -->
                        <li class="p-3 bg-slate-800/40 border border-slate-700/80 rounded-xl flex flex-col sm:flex-row items-start gap-3 hover:bg-slate-800 transition shadow-sm">
                            <div class="w-12 h-12 rounded-md bg-blue-900/40 text-blue-400 flex items-center justify-center shrink-0 border border-blue-700/50 shadow-inner text-2xl">
                                ⚡
                            </div>
                            <div class="flex-1">
                                <div class="font-bold text-blue-300 text-sm mb-1">天神 (Deity)</div>
                                <div class="text-xs md:text-sm text-slate-300 leading-relaxed space-y-1">
                                    <p><span class="text-blue-400 font-semibold">登場效果 (一次性)：</span>向場上所有未知牌 (除 12 外) 自動劈出一道雷電。並且天神每擊殺一張數字牌時便可獲得一張「雷電」法術。</p>
                                    <p><span class="text-blue-400 font-semibold">本命法術/覺醒：</span>對天神使用本命法術「雷電」後完成覺醒。</p>
                                    <p><span class="text-blue-400 font-semibold">覺醒技能：</span>引發毀滅性神罰，抹除場上任何牌（包括自己、友方、受保護英雄）（僅國王/真王/百夫長除外）。</p>
                                </div>
                            </div>
                        </li>
                        <!-- 憤怒者 -->
                        <li class="p-3 bg-slate-800/40 border border-slate-700/80 rounded-xl flex flex-col sm:flex-row items-start gap-3 hover:bg-slate-800 transition shadow-sm">
                            <div class="w-12 h-12 rounded-md bg-red-900/40 text-red-500 flex items-center justify-center shrink-0 border border-red-700/50 shadow-inner text-2xl">
                                😡
                            </div>
                            <div class="flex-1">
                                <div class="font-bold text-red-400 text-sm mb-1">憤怒者 (Angry One)</div>
                                <div class="text-xs md:text-sm text-slate-300 leading-relaxed space-y-1">
                                    <p><span class="text-red-400 font-semibold">登場效果：</span>所有己方單位攻擊力永久 +1。</p>
                                    <p><span class="text-red-400 font-semibold">本命法術/覺醒：</span>對憤怒者使用本命法術「狂暴」後完成覺醒。</p>
                                    <p><span class="text-red-400 font-semibold">覺醒技能：</span>免疫所有 0-3 數字牌的攻擊。己方所有單位攻擊力再 +2 (含原加成共 +3，最高上限為 11，0 保持不變)。</p>
                                </div>
                            </div>
                        </li>
                        <!-- 法師 -->
                        <li class="p-3 bg-slate-800/40 border border-slate-700/80 rounded-xl flex flex-col sm:flex-row items-start gap-3 hover:bg-slate-800 transition shadow-sm">
                            <div class="w-12 h-12 rounded-md bg-purple-900/40 text-purple-400 flex items-center justify-center shrink-0 border border-purple-700/50 shadow-inner text-2xl">
                                🧙
                            </div>
                            <div class="flex-1">
                                <div class="font-bold text-purple-300 text-sm mb-1">法師 (Mage)</div>
                                <div class="text-xs md:text-sm text-slate-300 leading-relaxed space-y-1">
                                    <p><span class="text-purple-400 font-semibold">被動技能：</span>法師每移動或攻擊 3 步，己方隨機獲得一張法術牌。</p>
                                    <p><span class="text-purple-400 font-semibold">本命法術/覺醒：</span>使用本命法術「抽牌」後完成覺醒。</p>
                                    <p><span class="text-purple-400 font-semibold">覺醒技能 (無盡法力)：</span>每回合開始時，將會隨機湧現一張臨時法術，玩家必須立即決定是否施放，若放棄則該法術消散。</p>
                                </div>
                            </div>
                        </li>
                        <!-- 百夫長 -->
                        <li class="p-3 bg-slate-800/40 border border-slate-700/80 rounded-xl flex flex-col sm:flex-row items-start gap-3 hover:bg-slate-800 transition shadow-sm">
                            <div class="w-12 h-12 rounded-md bg-emerald-900/40 text-emerald-400 flex items-center justify-center shrink-0 border border-emerald-700/50 shadow-inner text-2xl">
                                🛡️
                            </div>
                            <div class="flex-1">
                                <div class="font-bold text-emerald-300 text-sm mb-1">百夫長 (Centurion)</div>
                                <div class="text-xs md:text-sm text-slate-300 leading-relaxed space-y-1">
                                    <p><span class="text-emerald-400 font-semibold">基礎技能：</span>選擇該英雄後，該英雄直接代替國王，代替後玩家擁有雙倍牌庫 (既所有數字牌雙倍)，但是沒有任何法術。</p>
                                    <p><span class="text-emerald-400 font-semibold">覺醒技能：</span>滿足 10；9；11 分別死亡一張後覺醒。百夫長覺醒後，倘有牌死亡後，僅己方場上所有牌的攻擊力和生命值將會視為上一張死亡的牌的本質數值 (但是牌本質的數值不變)。</p>
                                </div>
                            </div>
                        </li>
                        <!-- 海王 -->
                        <li class="p-3 bg-slate-800/40 border border-slate-700/80 rounded-xl flex flex-col sm:flex-row items-start gap-3 hover:bg-slate-800 transition shadow-sm">
                            <div class="w-12 h-12 rounded-md bg-cyan-900/40 text-cyan-400 flex items-center justify-center shrink-0 border border-cyan-700/50 shadow-inner text-2xl">
                                🌊
                            </div>
                            <div class="flex-1">
                                <div class="font-bold text-cyan-300 text-sm mb-1">海王 (Sea King)</div>
                                <div class="text-xs md:text-sm text-slate-300 leading-relaxed space-y-1">
                                    <p><span class="text-cyan-400 font-semibold">基礎技能：</span>己方單位免疫所有控制效果 (石化/冰凍/傀儡)，並且無視河流淹死傷害。</p>
                                    <p><span class="text-cyan-400 font-semibold">本命法術/覺醒：</span>對海王使用本命法術「河流」後完成覺醒。</p>
                                    <p><span class="text-cyan-400 font-semibold">波濤與絞殺：</span>覺醒後，場上每 4 個己方回合將自動生成一次河流；並且在任何河流效果結束退去時，敵方前排的所有 0-6 數字牌將會被直接抹殺。</p>
                                </div>
                            </div>
                        </li>
                        <!-- 傑局 -->
                        <li class="p-3 bg-slate-800/40 border border-slate-700/80 rounded-xl flex flex-col sm:flex-row items-start gap-3 hover:bg-slate-800 transition shadow-sm">
                            <div class="w-12 h-12 rounded-md bg-indigo-900/40 text-indigo-400 flex items-center justify-center shrink-0 border border-indigo-700/50 shadow-inner text-2xl">
                                🌉
                            </div>
                            <div class="flex-1">
                                <div class="font-bold text-indigo-300 text-sm mb-1">傑局 (Mastermind)</div>
                                <div class="text-xs md:text-sm text-slate-300 leading-relaxed space-y-1">
                                    <p><span class="text-indigo-400 font-semibold">基礎技能：</span>在己方場地前永久生成一條專屬淚橋，直到傑局死亡後，淚橋才會消失 (橋上單位墜入深淵)。</p>
                                    <p><span class="text-indigo-400 font-semibold">本命法術/覺醒：</span>對傑局使用本命法術「淚橋」後完成覺醒。</p>
                                    <p><span class="text-indigo-400 font-semibold">覺醒技能：</span>每 3 個回合傑局有一次可以控制淚橋開啟或關閉的機會 (獲得一張臨時法術，回合結束未用即消失不可儲存)。</p>
                                </div>
                            </div>
                        </li>
                    </ul>
                `;
            }
            
            switchRulesTab('rules');

            if (modal && box) {
                modal.classList.remove('opacity-0', 'pointer-events-none');
                box.classList.remove('scale-95');
            }
        }

        function closeRulesModal() {
            soundEngine.playSelect();
            const modal = document.getElementById('rules-modal');
            const box = document.getElementById('rules-modal-box');
            if (modal && box) {
                modal.classList.add('opacity-0', 'pointer-events-none');
                box.classList.add('scale-95');
            }
        }

        /* --- Window Load Initialization --- */
        window.onload = function() {
            canvas = document.getElementById('gameCanvas');
            if (canvas) ctx = canvas.getContext('2d');

            if (canvas) {
                canvas.addEventListener('click', handleCanvasClick);
                canvas.addEventListener('mousemove', handleCanvasMouseMove);
            }

            window.addEventListener('resize', () => {
                checkDeviceAndSetLayout();
                resizeCanvas();
            });

            checkDeviceAndSetLayout();
            initGame();
        }

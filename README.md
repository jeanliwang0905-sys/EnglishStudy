# English-Practice
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>小學生英文單字酷學習</title>
    <style>
        :root {
            --primary: #FFD700; /* 亮黃 */
            --secondary: #87CEEB; /* 天空藍 */
            --accent: #FF69B4; /* 粉紅 */
            --vowel: #FF4136; /* 母音紅 */
            --consonant: #0074D9; /* 子音藍 */
            --bg: #FFF9E6;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg);
            margin: 0;
            padding: 20px;
            text-align: center;
        }

        /* 控制面板 */
        .controls {
            background: white;
            padding: 15px;
            border-radius: 15px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
            margin-bottom: 20px;
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 10px;
        }

        select, button {
            padding: 10px;
            border-radius: 8px;
            border: 2px solid var(--secondary);
            font-size: 16px;
        }

        /* 標籤頁切換 */
        .tabs {
            display: flex;
            justify-content: center;
            margin-bottom: 20px;
        }

        .tab-btn {
            background: #ddd;
            border: none;
            padding: 10px 20px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.3s;
        }

        .tab-btn.active {
            background: var(--secondary);
            color: white;
            border-bottom: 4px solid var(--accent);
        }

        /* 內容區塊 */
        .content-card {
            background: white;
            max-width: 600px;
            margin: 0 auto;
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 8px 20px rgba(0,0,0,0.1);
            min-height: 300px;
        }

        /* 單字樣式 */
        .word-display {
            font-size: 48px;
            font-weight: bold;
            margin: 20px 0;
            letter-spacing: 2px;
        }

        .vowel { color: var(--vowel); }
        .consonant { color: var(--consonant); }
        .syllable-divider { color: #ccc; margin: 0 5px; }

        .translation { font-size: 24px; color: #666; margin-bottom: 10px; }
        .example { font-size: 18px; color: #888; font-style: italic; }

        /* 積木模式按鈕 */
        .block-container {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 10px;
            margin-top: 20px;
        }

        .letter-btn {
            width: 50px;
            height: 50px;
            font-size: 24px;
            background: var(--primary);
            border: none;
            border-radius: 8px;
            cursor: pointer;
            box-shadow: 0 4px 0 #b39700;
        }

        .letter-btn:active { transform: translateY(4px); box-shadow: none; }

        /* 聽寫模式輸入 */
        .dictation-input {
            font-size: 32px;
            width: 80%;
            text-align: center;
            border: 3px solid var(--secondary);
            border-radius: 10px;
            padding: 10px;
        }

        .hidden { display: none; }
    </style>
</head>
<body>

    <h1>🍎 英語小學堂</h1>

    <div class="controls">
        <label>選組：<select id="levelSelect" onchange="initGame()"></select></label>
        <label>語音：<select id="voiceSelect"></select></label>
        <button onclick="speakCurrentWord()">🔊 播放聲音</button>
    </div>

    <div class="tabs">
        <button class="tab-btn active" onclick="switchTab('A')">模式 A: 學習</button>
        <button class="tab-btn" onclick="switchTab('B')">模式 B: 積木</button>
        <button class="tab-btn" onclick="switchTab('C')">模式 C: 聽寫</button>
    </div>

    <div class="content-card">
        <div id="displayArea">
            </div>
        <div id="feedback" style="margin-top: 20px; font-weight: bold; font-size: 20px;"></div>
        <div style="margin-top: 30px;">
            <button onclick="prevWord()">⬅️ 上一個</button>
            <span id="progressIndicator">0 / 0</span>
            <button onclick="nextWord()">下一個 ➡️</button>
        </div>
    </div>

<script>
    // --- 資料區 ---
    // 您可以將 Gemini 產生的單字表貼在這裡
    const rawData = [
        { word: "apple", syllable: "ap-ple", chinese: "蘋果", example: "I eat an apple every day." },
        { word: "banana", syllable: "ba-na-na", chinese: "香蕉", example: "The monkey likes bananas." },
        { word: "cat", syllable: "cat", chinese: "貓", example: "The cat is sleeping on the mat." },
        { word: "dog", syllable: "dog", chinese: "狗", example: "My dog is very friendly." },
        { word: "elephant", syllable: "el-e-phant", chinese: "大象", example: "An elephant has a long nose." },
        { word: "flower", syllable: "flow-er", chinese: "花", example: "This flower smells good." },
        { word: "grape", syllable: "grape", chinese: "葡萄", example: "I like purple grapes." },
        { word: "house", syllable: "house", chinese: "房子", example: "My house is small but cozy." },
        { word: "ice", syllable: "ice", chinese: "冰", example: "Put some ice in my water." },
        { word: "juice", syllable: "juice", chinese: "果汁", example: "I want some orange juice." },
        { word: "kite", syllable: "kite", chinese: "風箏", example: "The kite flies high in the sky." }
    ];

    let currentLevelWords = [];
    let currentIndex = 0;
    let currentTab = 'A';
    let showSyllable = false;
    let userInput = ""; // 用於積木模式

    // --- 初始化語音系統 ---
    const synth = window.speechSynthesis;
    const voiceSelect = document.getElementById('voiceSelect');

    function loadVoices() {
        const voices = synth.getVoices();
        voiceSelect.innerHTML = '';
        voices.forEach((voice, i) => {
            const option = document.createElement('option');
            option.value = i;
            option.textContent = `${voice.name} (${voice.lang})`;
            if (voice.lang.includes('en-US') || voice.name.includes('Google US')) {
                option.selected = true;
            }
            voiceSelect.appendChild(option);
        });
    }

    if (speechSynthesis.onvoiceschanged !== undefined) {
        speechSynthesis.onvoiceschanged = loadVoices;
    }
    loadVoices();

    function speakCurrentWord() {
        if (!currentLevelWords[currentIndex]) return;
        const utterance = new SpeechSynthesisUtterance(currentLevelWords[currentIndex].word);
        const voices = synth.getVoices();
        utterance.voice = voices[voiceSelect.value];
        utterance.rate = 0.9;
        synth.speak(utterance);
    }

    // --- 功能邏輯 ---

    function initGame() {
        const level = document.getElementById('levelSelect').value;
        if (level === "all") {
            currentLevelWords = [...rawData];
        } else {
            const start = parseInt(level) * 10;
            currentLevelWords = rawData.slice(start, start + 10);
        }
        currentIndex = 0;
        renderCurrentWord();
    }

    function createLevelOptions() {
        const select = document.getElementById('levelSelect');
        const numLevels = Math.ceil(rawData.length / 10);
        for (let i = 0; i < numLevels; i++) {
            let opt = document.createElement('option');
            opt.value = i;
            opt.textContent = `Level ${i + 1}`;
            select.appendChild(opt);
        }
        let allOpt = document.createElement('option');
        allOpt.value = "all";
        allOpt.textContent = "全部單字";
        select.appendChild(allOpt);
    }

    function switchTab(tab) {
        currentTab = tab;
        document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
        event.target.classList.add('active');
        userInput = "";
        document.getElementById('feedback').textContent = "";
        renderCurrentWord();
    }

    function colorizeWord(word) {
        const vowels = "aeiouAEIOU";
        return word.split('').map(char => {
            if (vowels.includes(char)) return `<span class="vowel">${char}</span>`;
            if (/[a-zA-Z]/.test(char)) return `<span class="consonant">${char}</span>`;
            return char;
        }).join('');
    }

    function renderCurrentWord() {
        const wordData = currentLevelWords[currentIndex];
        const display = document.getElementById('displayArea');
        const progress = document.getElementById('progressIndicator');
        
        if (!wordData) {
            display.innerHTML = "<h2>請選取關卡</h2>";
            return;
        }

        progress.textContent = `${currentIndex + 1} / ${currentLevelWords.length}`;
        display.innerHTML = "";

        if (currentTab === 'A') {
            // 模式 A: 學習
            const wordHTML = showSyllable ? 
                wordData.syllable.split('-').map(s => colorizeWord(s)).join('<span class="syllable-divider">|</span>') : 
                colorizeWord(wordData.word);

            display.innerHTML = `
                <div class="translation">${wordData.chinese}</div>
                <div class="word-display">${wordHTML}</div>
                <button onclick="toggleSyllable()">${showSyllable ? "隱藏音節" : "顯示音節"}</button>
                <div class="example" style="margin-top:20px">${wordData.example}</div>
            `;
        } 
        else if (currentTab === 'B') {
            // 模式 B: 積木
            display.innerHTML = `
                <div class="translation">${wordData.chinese}</div>
                <div class="word-display" style="min-height:60px">${userInput}</div>
                <div class="block-container" id="blockArea"></div>
                <button onclick="userInput=''; renderCurrentWord();" style="margin-top:20px; background:#ffcccb">重來</button>
            `;
            
            // 產生亂序字母
            if (userInput === "") {
                const letters = wordData.word.split('').sort(() => Math.random() - 0.5);
                const blockArea = document.getElementById('blockArea');
                letters.forEach(l => {
                    const btn = document.createElement('button');
                    btn.className = 'letter-btn';
                    btn.textContent = l;
                    btn.onclick = () => {
                        userInput += l;
                        checkBlockAnswer();
                        renderCurrentWord();
                    };
                    blockArea.appendChild(btn);
                });
            }
        } 
        else if (currentTab === 'C') {
            // 模式 C: 聽寫
            display.innerHTML = `
                <div class="translation">${wordData.chinese}</div>
                <div style="margin: 20px 0;">
                    <input type="text" id="dInput" class="dictation-input" autocomplete="off" 
                           onkeyup="if(event.key==='Enter') checkDictation()">
                </div>
                <button onclick="checkDictation()">檢查答案</button>
            `;
            setTimeout(() => document.getElementById('dInput').focus(), 100);
        }
    }

    function toggleSyllable() {
        showSyllable = !showSyllable;
        renderCurrentWord();
    }

    function checkBlockAnswer() {
        const target = currentLevelWords[currentIndex].word;
        if (userInput.length === target.length) {
            if (userInput === target) {
                document.getElementById('feedback').textContent = "🎨 太棒了！答對了！";
                speakCurrentWord();
            } else {
                document.getElementById('feedback').textContent = "❌ 再試一次喔！";
            }
        }
    }

    function checkDictation() {
        const val = document.getElementById('dInput').value.toLowerCase().trim();
        const target = currentLevelWords[currentIndex].word.toLowerCase();
        if (val === target) {
            document.getElementById('feedback').textContent = "🌟 答對了！你真棒！";
            speakCurrentWord();
        } else {
            document.getElementById('feedback').textContent = "✍️ 拼錯囉，再檢查一下！";
        }
    }

    function nextWord() {
        if (currentIndex < currentLevelWords.length - 1) {
            currentIndex++;
            userInput = "";
            document.getElementById('feedback').textContent = "";
            renderCurrentWord();
        }
    }

    function prevWord() {
        if (currentIndex > 0) {
            currentIndex--;
            userInput = "";
            document.getElementById('feedback').textContent = "";
            renderCurrentWord();
        }
    }

    // 初始化
    createLevelOptions();
    initGame();
</script>
</body>
</html>

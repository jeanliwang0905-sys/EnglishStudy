# English-Practice 
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>小學生英文單字酷學習</title>
    <style>
        :root {
            --primary: #FFD700;
            --secondary: #87CEEB;
            --accent: #FF69B4;
            --vowel: #FF4136;
            --consonant: #0074D9;
            --bg: #FFF9E6;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg);
            margin: 0;
            padding: 20px;
            text-align: center;
        }

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
            cursor: pointer;
        }

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

        .content-card {
            background: white;
            max-width: 600px;
            margin: 0 auto;
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 8px 20px rgba(0,0,0,0.1);
            min-height: 350px;
        }

        .word-display {
            font-size: 48px;
            font-weight: bold;
            margin: 20px 0;
            letter-spacing: 2px;
            min-height: 60px;
        }

        .vowel { color: var(--vowel); }
        .consonant { color: var(--consonant); }
        .syllable-divider { color: #ccc; margin: 0 5px; }

        .translation { font-size: 24px; color: #666; margin-bottom: 10px; }
        .example { font-size: 18px; color: #888; font-style: italic; }

        .block-container {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 10px;
            margin-top: 20px;
            min-height: 60px;
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
            transition: 0.1s;
        }

        .letter-btn:disabled {
            background: #eee;
            box-shadow: none;
            color: #ccc;
            cursor: not-allowed;
            transform: translateY(4px);
        }

        .letter-btn:active:not(:disabled) { transform: translateY(4px); box-shadow: none; }

        .dictation-input {
            font-size: 32px;
            width: 80%;
            text-align: center;
            border: 3px solid var(--secondary);
            border-radius: 10px;
            padding: 10px;
        }

        #feedback {
            margin-top: 20px;
            font-weight: bold;
            font-size: 20px;
            height: 30px;
        }
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
        <button class="tab-btn active" onclick="switchTab('A', this)">模式 A: 學習</button>
        <button class="tab-btn" onclick="switchTab('B', this)">模式 B: 積木</button>
        <button class="tab-btn" onclick="switchTab('C', this)">模式 C: 聽寫</button>
    </div>

    <div class="content-card">
        <div id="displayArea"></div>
        <div id="feedback"></div>
        <div style="margin-top: 30px;">
            <button onclick="prevWord()">⬅️ 上一個</button>
            <span id="progressIndicator">0 / 0</span>
            <button onclick="nextWord()">下一個 ➡️</button>
        </div>
    </div>

<script>
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
    let userInput = ""; 
    let shuffledLetters = []; // 新增：儲存目前單字的亂序字母

    const synth = window.speechSynthesis;
    const voiceSelect = document.getElementById('voiceSelect');

    function loadVoices() {
        const voices = synth.getVoices();
        voiceSelect.innerHTML = '';
        voices.forEach((voice, i) => {
            if (voice.lang.includes('en')) {
                const option = document.createElement('option');
                option.value = i;
                option.textContent = `${voice.name} (${voice.lang})`;
                if (voice.lang === 'en-US') option.selected = true;
                voiceSelect.appendChild(option);
            }
        });
    }

    if (speechSynthesis.onvoiceschanged !== undefined) {
        speechSynthesis.onvoiceschanged = loadVoices;
    }
    loadVoices();

    function speakCurrentWord() {
        if (!currentLevelWords[currentIndex]) return;
        synth.cancel(); // 停止之前的聲音
        const utterance = new SpeechSynthesisUtterance(currentLevelWords[currentIndex].word);
        const voices = synth.getVoices();
        utterance.voice = voices[voiceSelect.value];
        utterance.rate = 0.8;
        synth.speak(utterance);
    }

    function initGame() {
        const level = document.getElementById('levelSelect').value;
        if (level === "all") {
            currentLevelWords = [...rawData];
        } else {
            const start = parseInt(level) * 10;
            currentLevelWords = rawData.slice(start, start + 10);
        }
        currentIndex = 0;
        resetState();
        renderCurrentWord();
    }

    function resetState() {
        userInput = "";
        document.getElementById('feedback').textContent = "";
        if (currentLevelWords[currentIndex]) {
            // 初始化積木順序
            shuffledLetters = currentLevelWords[currentIndex].word.split('')
                .map((l, index) => ({ char: l, originalIndex: index, used: false }))
                .sort(() => Math.random() - 0.5);
        }
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

    function switchTab(tab, btn) {
        currentTab = tab;
        document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        resetState();
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
            display.innerHTML = `
                <div class="translation">${wordData.chinese}</div>
                <div class="word-display">${userInput}</div>
                <div class="block-container" id="blockArea"></div>
                <div style="margin-top:20px">
                    <button onclick="resetState(); renderCurrentWord();" style="background:#ffcccb">重來</button>
                </div>
            `;
            
            const blockArea = document.getElementById('blockArea');
            shuffledLetters.forEach((item, idx) => {
                const btn = document.createElement('button');
                btn.className = 'letter-btn';
                btn.textContent = item.char;
                btn.disabled = item.used;
                btn.onclick = () => {
                    userInput += item.char;
                    item.used = true;
                    renderCurrentWord();
                    checkBlockAnswer();
                };
                blockArea.appendChild(btn);
            });
        } 
        else if (currentTab === 'C') {
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
        const feedback = document.getElementById('feedback');
        if (userInput.length === target.length) {
            if (userInput === target) {
                feedback.innerHTML = "<span style='color:green'>🎨 太棒了！答對了！</span>";
                speakCurrentWord();
            } else {
                feedback.innerHTML = "<span style='color:red'>❌ 拼錯了，再試一次！</span>";
                // 兩秒後自動重選，方便學生繼續練習
                setTimeout(() => {
                    resetState();
                    renderCurrentWord();
                }, 1500);
            }
        }
    }

    function checkDictation() {
        const val = document.getElementById('dInput').value.toLowerCase().trim();
        const target = currentLevelWords[currentIndex].word.toLowerCase();
        const feedback = document.getElementById('feedback');
        if (val === target) {
            feedback.innerHTML = "<span style='color:green'>🌟 答對了！你真棒！</span>";
            speakCurrentWord();
        } else {
            feedback.innerHTML = "<span style='color:red'>✍️ 拼錯囉，再檢查一下！</span>";
        }
    }

    function nextWord() {
        if (currentIndex < currentLevelWords.length - 1) {
            currentIndex++;
            resetState();
            renderCurrentWord();
        }
    }

    function prevWord() {
        if (currentIndex > 0) {
            currentIndex--;
            resetState();
            renderCurrentWord();
        }
    }

    createLevelOptions();
    initGame();
</script>
</body>
</html>

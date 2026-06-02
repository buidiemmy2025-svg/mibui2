<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>App Chấm Điểm Phát Âm Tiếng Trung</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f2f5;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: white;
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            text-align: center;
            max-width: 450px;
            width: 100%;
        }
        h1 {
            color: #1a73e8;
            font-size: 24px;
            margin-bottom: 20px;
        }
        .phrase-box {
            background-color: #e8f0fe;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
        }
        .chinese-text {
            font-size: 28px;
            font-weight: bold;
            color: #333;
            margin: 0;
        }
        .pinyin-text {
            font-size: 16px;
            color: #666;
            margin: 5px 0 0 0;
        }
        .btn-mic {
            background-color: #1a73e8;
            color: white;
            border: none;
            padding: 15px 30px;
            font-size: 16px;
            border-radius: 50px;
            cursor: pointer;
            transition: background 0.3s;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
        }
        .btn-mic:hover {
            background-color: #1557b0;
        }
        .btn-mic:disabled {
            background-color: #ccc;
        }
        .status {
            margin-top: 15px;
            font-style: italic;
            color: #555;
        }
        .result-box {
            margin-top: 25px;
            border-top: 2px solid #eee;
            padding-top: 20px;
        }
        .score {
            font-size: 48px;
            font-weight: bold;
            margin: 10px 0;
        }
        .good { color: #2ea44f; }
        .average { color: #f9ab00; }
        .bad { color: #d93025; }
    </style>
</head>
<body>

<div class="container">
    <h1>Phát Âm Tiếng Trung</h1>
    
    <div class="phrase-box">
        <p class="chinese-text" id="target-text">你好</p>
        <p class="pinyin-text" id="target-pinyin">nǐ hǎo (Xin chào)</p>
    </div>

    <button class="btn-mic" id="btn-start">🎙️ Bấm vào đây để Đọc</button>
    <p class="status" id="status">Sẵn sàng...</p>

    <div class="result-box">
        <p>Máy nghe được: <strong id="user-spoken" style="color: #1a73e8;">...</strong></p>
        <p>Điểm số của bạn:</p>
        <div class="score" id="score-display">--</div>
    </div>
    
    <button onclick="changePhrase()" style="margin-top: 15px; background: none; border: 1px solid #1a73e8; color: #1a73e8; padding: 5px 10px; border-radius: 4px; cursor: pointer;">Đổi câu khác</button>
</div>

<script>
    // Danh sách các câu mẫu để luyện tập
    const phrases = [
        { text: "你好", pinyin: "nǐ hǎo (Xin chào)" },
        { text: "谢谢", pinyin: "xièxie (Cảm ơn)" },
        { text: "我爱你", pinyin: "wǒ ài nǐ (Tôi yêu bạn)" },
        { text: "中国", pinyin: "Zhōngguó (Trung Quốc)" },
        { text: "我想 học 汉语", pinyin: "wǒ xiǎng xué hànyǔ (Tôi muốn học tiếng Trung)" }
    ];

    let currentIdx = 0;

    // Kiểm tra xem trình duyệt có hỗ trợ nhận diện giọng nói không
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    
    if (!SpeechRecognition) {
        alert("Trình duyệt của bạn không hỗ trợ Web Speech API. Hãy dùng Google Chrome trên máy tính nhé!");
    } else {
        const recognition = new SpeechRecognition();
        // Cấu hình nhận diện tiếng Trung (Mẫu mặc định: zh-CN là tiếng Phổ thông đại lục)
        recognition.lang = 'zh-CN'; 
        recognition.interimResults = false;
        recognition.maxAlternatives = 1;

        const btnStart = document.getElementById('btn-start');
        const statusText = document.getElementById('status');
        const userSpokenText = document.getElementById('user-spoken');
        const scoreDisplay = document.getElementById('score-display');
        const targetTextElement = document.getElementById('target-text');

        btnStart.addEventListener('click', () => {
            recognition.start();
            statusText.innerText = "🎤 Đang lắng nghe... Hãy nói đi!";
            btnStart.disabled = true;
        });

        recognition.onspeechend = () => {
            recognition.stop();
            statusText.innerText = "🔄 Đang xử lý âm thanh...";
            btnStart.disabled = false;
        };

        recognition.onerror = (event) => {
            statusText.innerText = "❌ Lỗi: " + event.error;
            btnStart.disabled = false;
        };

        recognition.onresult = (event) => {
            const resultText = event.results[0][0].transcript;
            // Xóa dấu câu nếu có để so sánh chính xác hơn
            const cleanResult = resultText.replace(/[.,\/#!$%\^&\*;:{}=\-_`~()？。，！]/g,"");
            userSpokenText.innerText = cleanResult;

            const targetText = targetTextElement.innerText;

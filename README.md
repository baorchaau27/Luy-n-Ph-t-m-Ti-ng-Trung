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
            max-width: 500px;
            width: 100%;
        }
        h1 {
            color: #2c3e50;
            margin-bottom: 20px;
        }
        .flashcard {
            background-color: #f8f9fa;
            border: 2px solid #e9ecef;
            padding: 20px;
            border-radius: 12px;
            margin-bottom: 20px;
        }
        .pinyin {
            font-size: 1.2rem;
            color: #e74c3c;
            font-weight: bold;
        }
        .chinese {
            font-size: 2.5rem;
            color: #2c3e50;
            margin: 10px 0;
            letter-spacing: 2px;
        }
        .meaning {
            font-size: 1rem;
            color: #7f8c8d;
            font-style: italic;
        }
        button {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 12px 24px;
            font-size: 1rem;
            border-radius: 30px;
            cursor: pointer;
            transition: all 0.3s;
            box-shadow: 0 4px 6px rgba(52, 152, 219, 0.2);
        }
        button:hover {
            background-color: #2980b9;
            transform: translateY(-2px);
        }
        button:disabled {
            background-color: #bdc3c7;
            cursor: not-allowed;
        }
        .status {
            margin-top: 15px;
            font-weight: bold;
            color: #7f8c8d;
        }
        .result-box {
            margin-top: 20px;
            padding: 15px;
            border-radius: 8px;
            display: none;
        }
        .success {
            background-color: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        .fail {
            background-color: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
    </style>
</head>
  <H1> Lê Đặng Bảo Châu </H1>
<body>

<div class="container">
    <h1>Phát Âm Tiếng Trung</h1>
    
    <div class="flashcard">
        <div class="pinyin" id="target-pinyin">Nǐ hǎo</div>
        <div class="chinese" id="target-chinese">你好</div>
        <div class="meaning" id="target-meaning">(Xin chào)</div>
    </div>

    <button id="btn-speak">🎤 Bật Micro và Nói</button>
    <button id="btn-next" style="background-color: #2ecc71; display:none;">Câu tiếp theo ➡️</button>
    
    <div class="status" id="status">Sẵn sàng. Hãy nhấn nút để nói.</div>
    
    <div class="result-box" id="result-box">
        <p><strong>Bạn vừa nói:</strong> <span id="user-speech" style="font-size: 1.3rem;"></span></p>
        <p id="score-text"></p>
    </div>
</div>

<script>
    // Danh sách câu hỏi mẫu
    const lessons = [
        { chinese: "你好", pinyin: "Nǐ hǎo", meaning: "Xin chào" },
        { chinese: "谢谢", pinyin: "Xièxie", meaning: "Cảm ơn" },
        { chinese: "我爱你", pinyin: "Wǒ ài nǐ", meaning: "Tôi yêu bạn" },
        { chinese: "中国", pinyin: "Zhōngguó", meaning: "Trung Quốc" },
        { chinese: "你叫什么名字", pinyin: "Nǐ jiào shénme míngzi", meaning: "Bạn tên là gì?" }
    ];

    let currentIdx = 0;

    // Cấu hình Web Speech API của Google (Miễn phí)
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    
    if (!SpeechRecognition) {
        alert("Trình duyệt của bạn không hỗ trợ Web Speech API. Vui lòng dùng Google Chrome!");
    }

    const recognition = new SpeechRecognition();
    recognition.lang = 'zh-CN'; // Đặt ngôn ngữ nhận diện là tiếng Trung Phổ Thông
    recognition.interimResults = false; // Chỉ lấy kết quả cuối cùng
    recognition.maxAlternatives = 1;

    const btnSpeak = document.getElementById('btn-speak');
    const btnNext = document.getElementById('btn-next');
    const statusText = document.getElementById('status');
    const resultBox = document.getElementById('result-box');
    const userSpeech = document.getElementById('user-speech');
    const scoreText = document.getElementById('score-text');

    // Hàm cập nhật câu hỏi lên giao diện
    function loadLesson() {
        document.getElementById('target-pinyin').innerText = lessons[currentIdx].pinyin;
        document.getElementById('target-chinese').innerText = lessons[currentIdx].chinese;
        document.getElementById('target-meaning').innerText = `(${lessons[currentIdx].meaning})`;
        resultBox.style.display = 'none';
        btnNext.style.display = 'none';
        btnSpeak.style.display = 'inline-block';
        statusText.innerText = "Sẵn sàng. Hãy nhấn nút để nói.";
    }

    // Sự kiện khi nhấn nút Nói
    btnSpeak.addEventListener('click', () => {
        recognition.start();
        statusText.innerText = "🎙️ Đang nghe... Hãy đọc câu phía trên!";
        btnSpeak.disabled = true;
    });

    // Khi Google xử lý xong và trả về kết quả dạng chữ
    recognition.onresult = (event) => {
        const resultText = event.results

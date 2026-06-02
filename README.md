<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>App Chấm Điểm Tiếng Trung</title>
    <style>
        body { background: #f0f2f5; font-family: sans-serif; text-align: center; padding: 50px; }
        .box { background: white; max-width: 400px; margin: 0 auto; padding: 30px; border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
        h1 { font-size: 36px; margin: 10px 0; color: #333; }
        p { color: #666; font-size: 18px; }
        button { width: 100%; padding: 12px; margin: 8px 0; font-size: 16px; font-weight: bold; border: none; border-radius: 8px; cursor: pointer; }
        .btn-audio { background: #e4e6eb; color: #050505; }
        .btn-speak { background: #1877f2; color: white; }
        .btn-speak.recording { background: #fa3e3e; }
        .result { margin-top: 20px; padding: 15px; background: #e7f3ff; border-radius: 8px; display: none; }
    </style>
</head>
<body>

<div class="box">
    <p>Hãy đọc câu này:</p>
    <h1 id="txtWord">你好</h1>
    <p id="txtPinyin">Nǐ hǎo</p>
    
    <button class="btn-audio" onclick="playAudio()">🔊 Nghe Đọc Mẫu</button>
    <button class="btn-speak" id="btnMic" onclick="startRecording()">🎙️ Bắt Đầu Nói</button>

    <div class="result" id="resultArea">
        <h3 style="margin:0; color:#1877f2;">Kết quả: <span id="txtScore">0%</span></h3>
        <p style="margin: 5px 0 0 0; font-size: 14px;">Bạn đã nói: "<span id="txtHeard">...</span>"</p>
    </div>
</div>

<script>
    var phrase = "你好";
    var recognition;
    
    // 1. Hàm nghe đọc mẫu (Không cần quyền Micro)
    function playAudio() {
        if ('speechSynthesis' in window) {
            window.speechSynthesis.cancel();
            var msg = new SpeechSynthesisUtterance(phrase);
            msg.lang = 'zh-CN';
            window.speechSynthesis.speak(msg);
        } else {
            alert("Trình duyệt của bạn không hỗ trợ tính năng phát âm.");
        }
    }

    // 2. Hàm nhận diện giọng nói
    function startRecording() {
        var SpeechLib = window.SpeechRecognition || window.webkitSpeechRecognition;
        if (!SpeechLib) {
            alert("Trình duyệt không hỗ trợ API này. Vui lòng dùng Google Chrome!");
            return;
        }

        if (!recognition) {
            recognition = new SpeechLib();
            recognition.lang = 'zh-CN';
            
            recognition.onstart = function() {
                document.getElementById('btnMic').textContent = "🛑 Đang lắng nghe... Hãy nói!";
                document.getElementById('btnMic').className = "btn-speak recording";
            };

            recognition.onend = function() {
                document.getElementById('btnMic').textContent = "🎙️ Bắt Đầu Nói";
                document.getElementById('btnMic').className = "btn-speak";
            };

            recognition.onerror = function(e) {
                alert("Lỗi hệ thống: " + e.error + ". Hãy chắc chắn bạn đã cho phép mở Micro!");
            };

            recognition.onresult = function(event) {
                var resultText = event.results[0][0].transcript;
                document.getElementById('txtHeard').textContent = resultText;
                
                // Thuật toán so khớp chữ đơn giản
                var cleanUser = resultText.replace(/[。，？！、]/g, "").trim();
                var score = (cleanUser === phrase) ? 100 : (phrase.includes(cleanUser) && cleanUser !== "" ? 50 : 0);
                
                document.getElementById('txtScore').textContent = score + "%";
                document.getElementById('resultArea').style.display = "block";
            };
        }

        try {
            recognition.start();
        } catch(err) {
            recognition.stop();
        }
    }
</script>

</body>
</html>

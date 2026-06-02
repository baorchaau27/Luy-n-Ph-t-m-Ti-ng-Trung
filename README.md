<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>App Chấm Điểm Tiếng Trung</title>

<style>
body{
    font-family:Arial,sans-serif;
    max-width:900px;
    margin:auto;
    padding:20px;
    background:#f5f5f5;
}

.card{
    background:white;
    padding:20px;
    border-radius:10px;
    margin-bottom:20px;
    box-shadow:0 2px 10px rgba(0,0,0,.1);
}

textarea{
    width:100%;
    min-height:100px;
    padding:10px;
}

button{
    padding:10px 20px;
    margin:5px;
    cursor:pointer;
}

.score{
    font-size:22px;
    font-weight:bold;
    color:green;
}
</style>

</head>
<body>

<h1>🇨🇳 App Chấm Điểm Tiếng Trung</h1>

<div class="card">

<h3>Câu mẫu</h3>

<textarea id="target">
你好，我叫李明。我今年二十岁。我喜欢学习汉语。
</textarea>

</div>

<div class="card">

<h3>Bài nói / bài viết của học sinh</h3>

<textarea id="student"></textarea>

<br>

<button onclick="startSpeech()">🎤 Ghi âm</button>

<button onclick="gradeBasic()">📊 Chấm nhanh</button>

<button onclick="gradeAI()">🤖 Chấm bằng Gemini AI</button>

</div>

<div class="card">

<h3>Kết quả</h3>

<div id="result"></div>

</div>

<script>

function similarity(a,b){

    a=a.trim();
    b=b.trim();

    let same=0;

    for(let i=0;i<Math.min(a.length,b.length);i++){
        if(a[i]===b[i]) same++;
    }

    return Math.round(
        same/Math.max(a.length,b.length)*100
    );
}

function gradeBasic(){

    const target=
    document.getElementById("target").value;

    const student=
    document.getElementById("student").value;

    const score=
    similarity(target,student);

    let level="";

    if(score>=90) level="Xuất sắc";
    else if(score>=75) level="Tốt";
    else if(score>=60) level="Khá";
    else level="Cần cải thiện";

    document.getElementById("result").innerHTML=
    `
    <div class="score">Điểm: ${score}/100</div>
    <p>Mức đánh giá: ${level}</p>
    `;
}

function startSpeech(){

    const SpeechRecognition=
        window.SpeechRecognition ||
        window.webkitSpeechRecognition;

    if(!SpeechRecognition){
        alert("Trình duyệt không hỗ trợ ghi âm.");
        return;
    }

    const recognition=
    new SpeechRecognition();

    recognition.lang="zh-CN";

    recognition.start();

    recognition.onresult=function(event){

        const text=
        event.results[0][0].transcript;

        document.getElementById("student").value=text;
    };
}

async function gradeAI(){

    const target=
    document.getElementById("target").value;

    const student=
    document.getElementById("student").value;

    const apiKey=
    "DAN_API_KEY_GEMINI_VAO_DAY";

    const prompt=`
Bạn là giáo viên tiếng Trung.

Câu mẫu:
${target}

Câu học sinh:
${student}

Hãy:

1. Chấm điểm 100
2. Đánh giá phát âm (nếu có thể)
3. Đánh giá ngữ pháp
4. Đánh giá từ vựng
5. Chỉ ra lỗi sai
6. Đưa phiên bản đúng hơn

Trả lời bằng tiếng Việt.
`;

    try{

        document.getElementById("result").innerHTML=
        "Đang chấm...";

        const response=
        await fetch(
        `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`,
        {
            method:"POST",
            headers:{
                "Content-Type":"application/json"
            },
            body:JSON.stringify({
                contents:[
                    {
                        parts:[
                            {
                                text:prompt
                            }
                        ]
                    }
                ]
            })
        });

        const data=
        await response.json();

        const answer=
        data.candidates[0].content.parts[0].text;

        document.getElementById("result").innerHTML=
        `<pre>${answer}</pre>`;

    }catch(error){

        document.getElementById("result").innerHTML=
        "Lỗi: "+error.message;
    }
}

</script>

</body>
</html>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>دائرة الأحكام - Iron Judge</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <style>
        :root {
            --gold: #ffd700;
            --dark: #0f0f0f;
            --red: #ff4d4d;
        }

        body {
            background-color: var(--dark);
            color: white;
            font-family: 'Cairo', sans-serif;
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }

        /* تصميم العجلة */
        .wheel-container {
            position: relative;
            width: 300px;
            height: 300px;
            border: 5px solid var(--gold);
            border-radius: 50%;
            margin-bottom: 30px;
            box-shadow: 0 0 30px rgba(255, 215, 0, 0.3);
        }

        .wheel {
            width: 100%;
            height: 100%;
            border-radius: 50%;
            transition: transform 4s cubic-bezier(0.17, 0.67, 0.12, 0.99);
            background: conic-gradient(
                #1a1a1a 0% 25%, 
                #333 25% 50%, 
                #1a1a1a 50% 75%, 
                #333 75% 100%
            );
        }

        .pointer {
            position: absolute;
            top: -20px;
            left: 50%;
            transform: translateX(-50%);
            width: 0;
            height: 0;
            border-left: 15px solid transparent;
            border-right: 15px solid transparent;
            border-top: 30px solid var(--red);
            z-index: 10;
        }

        .info-card {
            background: #1a1a1a;
            padding: 15px;
            border-radius: 15px;
            width: 85%;
            border: 1px solid #333;
            text-align: center;
            margin-bottom: 20px;
        }

        .btn-spin {
            background: var(--gold);
            color: black;
            border: none;
            padding: 15px 40px;
            font-size: 20px;
            font-weight: bold;
            border-radius: 50px;
            cursor: pointer;
            box-shadow: 0 5px 15px rgba(255, 215, 0, 0.4);
        }

        .btn-spin:disabled {
            background: #555;
            cursor: not-allowed;
        }

        #result {
            margin-top: 15px;
            font-weight: bold;
            color: var(--gold);
            min-height: 24px;
        }
    </style>
</head>
<body>

    <div class="info-card">
        <div>المقاتل: <span id="user-name" style="color:var(--gold)">...</span></div>
        <div id="balance-info" style="font-size: 0.9em; margin-top: 5px;">رصيدي: جاري التحميل...</div>
    </div>

    <div class="wheel-container">
        <div class="pointer"></div>
        <div class="wheel" id="wheel"></div>
    </div>

    <div id="result">اضغط لتدوير العجلة</div>
    <button class="btn-spin" id="spinBtn" onclick="spinWheel()">إطلاق الحكم ⚖️</button>

    <script>
        const tele = window.Telegram.WebApp;
        tele.expand();
        tele.ready();

        const user = tele.initDataUnsafe.user;
        if (user) {
            document.getElementById('user-name').innerText = user.first_name;
        }

        let isSpinning = false;
        const judgments = ["إعدام بنقاطك", "مجزرة جماعية", "عفو ملكي", "سرقة رصيد", "مضاعفة النقاط", "حكم قاسي"];

        function spinWheel() {
            if (isSpinning) return;
            
            isSpinning = true;
            const wheel = document.getElementById('wheel');
            const btn = document.getElementById('spinBtn');
            const resultText = document.getElementById('result');
            
            btn.disabled = true;
            resultText.innerText = "جاري النطق بالحكم...";

            const randomDeg = Math.floor(Math.random() * 360) + 3600; // 10 دورات كاملة على الأقل
            wheel.style.transform = `rotate(${randomDeg}deg)`;

            setTimeout(() => {
                isSpinning = false;
                const actualDeg = randomDeg % 360;
                const index = Math.floor(actualDeg / (360 / judgments.length));
                const finalJudgment = judgments[index];
                
                resultText.innerText = `الحكم: ${finalJudgment}`;
                
                // إرسال البيانات للبوت (دالة handle_app_data)
                const data = {
                    winner: user ? user.id : 0,
                    judgment: finalJudgment,
                    points: 20
                };
                
                setTimeout(() => {
                    tele.sendData(JSON.stringify(data));
                }, 1500);
                
            }, 4000);
        }

        tele.setHeaderColor('#0f0f0f');
    </script>
</body>
</html>

import telebot
from telebot import types
import json
import os
from http.server import BaseHTTPRequestHandler, HTTPServer
import threading

# --- الإعدادات الأساسية ---
TOKEN = "8640531885:AAFL4Z15bhZ3R0l9sn7rZaFkBkzODxQcxok"
bot = telebot.TeleBot(TOKEN, parse_mode="HTML")

# هذا الرابط سيتم تحديثه بعد رفعه على GitHub Pages
WEB_APP_URL = "https://your-username.github.io/roulette-app/" 

# --- واجهة الـ HTML (التصميم المرتب والألوان اللطيفة) ---
HTML_CONTENT = """
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>دائرة الأحكام - Mini App</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Cairo', sans-serif;
            background-color: #f8f9fa;
            margin: 0; padding: 15px;
            color: #2d3436; text-align: center;
        }
        .app-container {
            max-width: 500px; margin: 0 auto;
        }
        .header {
            background: linear-gradient(135deg, #a29bfe, #6c5ce7);
            color: white; padding: 25px;
            border-radius: 20px; margin-bottom: 20px;
            box-shadow: 0 10px 20px rgba(108, 92, 231, 0.2);
        }
        .balance-grid {
            display: grid; grid-template-columns: 1fr 1fr; gap: 15px;
            margin-bottom: 25px;
        }
        .card {
            background: white; padding: 15px;
            border-radius: 15px; box-shadow: 0 4px 6px rgba(0,0,0,0.05);
        }
        .card b { display: block; color: #6c5ce7; font-size: 1.2em; }
        
        .wheel-box {
            position: relative; width: 260px; height: 260px;
            margin: 30px auto; border: 10px solid white;
            border-radius: 50%; box-shadow: 0 15px 35px rgba(0,0,0,0.1);
            background: conic-gradient(#ff7675 0% 25%, #74b9ff 25% 50%, #ffeaa7 50% 75%, #55efc4 75% 100%);
            transition: transform 4s cubic-bezier(0.15, 0, 0.15, 1);
        }
        .wheel-pointer {
            position: absolute; top: -20px; left: 50%;
            transform: translateX(-50%); width: 0; height: 0;
            border-left: 15px solid transparent; border-right: 15px solid transparent;
            border-top: 30px solid #d63031; z-index: 10;
        }
        
        .btn-main {
            background: #6c5ce7; color: white; border: none;
            padding: 15px 40px; border-radius: 50px;
            font-size: 18px; font-weight: bold; cursor: pointer;
            box-shadow: 0 5px 15px rgba(108, 92, 231, 0.4);
            transition: 0.3s;
        }
        .btn-main:active { transform: scale(0.95); }

        .shop-title { margin-top: 40px; color: #636e72; font-size: 1.1em; }
        .shop-grid {
            display: grid; grid-template-columns: 1fr 1fr; gap: 10px;
            margin-top: 15px;
        }
        .item {
            background: #fff; padding: 12px; border-radius: 12px;
            border: 1px solid #dfe6e9; font-size: 0.9em;
        }
    </style>
</head>
<body>
    <div class="app-container">
        <div class="header">
            <h3>دائرة الأحكام <span id="user-icon">🎡</span></h3>
            <p>مرحباً بك يا <span id="player-name">...</span></p>
        </div>

        <div class="balance-grid">
            <div class="card">
                <small>رصيدي 💰</small>
                <b id="my-bal">500</b>
            </div>
            <div class="card">
                <small>رصيده 💎</small>
                <b id="his-bal">0</b>
            </div>
        </div>

        <div class="wheel-pointer"></div>
        <div class="wheel-box" id="wheel"></div>
        
        <button class="btn-main" onclick="startSpin()">تدوير الآن 🔥</button>

        <div class="shop-title">متجر الأدوات 🛡️</div>
        <div class="shop-grid">
            <div class="item">تذكرة حماية<br><b>100ن</b></div>
            <div class="item">تغيير ضحية<br><b>50ن</b></div>
        </div>
    </div>

    <script>
        const tg = window.Telegram.WebApp;
        tg.expand();
        tg.ready();
        
        document.getElementById('player-name').innerText = tg.initDataUnsafe.user?.first_name || "لاعب";

        function startSpin() {
            const wheel = document.getElementById('wheel');
            const deg = Math.floor(Math.random() * 5000) + 1000;
            wheel.style.transform = `rotate(${deg}deg)`;
            
            setTimeout(() => {
                tg.sendData(JSON.stringify({
                    "action": "spin_done",
                    "points": 25
                }));
                tg.close();
            }, 4500);
        }
    </script>
</body>
</html>
"""

# --- منطق البوت (Backend) ---

@bot.message_handler(commands=['start'])
def welcome(message):
    user_name = message.from_user.first_name
    
    welcome_text = (
        f"مرحبًا بك {user_name} في بوت دائرة الأحكام "
        f"<tg-emoji emoji-id='5226711870492126219'>🎡</tg-emoji>\n\n"
        f"<tg-emoji emoji-id='5371010084304342436'>⚖️</tg-emoji> **رصيدي ورصيده متصل بالـ App**\n"
        f"<tg-emoji emoji-id='5255885075072966358'>✨</tg-emoji> **ألوان واجهة عصرية**\n\n"
        "اضغط على الزر أدناه لفتح تجربة الـ Mini App الكاملة:"
    )

    markup = types.InlineKeyboardMarkup()
    # تأكد من وضع رابط GitHub Pages الخاص بك هنا بعد الرفع
    markup.add(types.InlineKeyboardButton(
        text="فتح التطبيق المصغر 🚀", 
        web_app=types.WebAppInfo(url=WEB_APP_URL)
    ))
    
    bot.reply_to(message, welcome_text, reply_markup=markup)

@bot.message_handler(content_types=['web_app_data'])
def web_app_data_handler(message):
    data = json.loads(message.web_app_data.data)
    if data['action'] == "spin_done":
        bot.send_message(message.chat.id, 
            f"<tg-emoji emoji-id='5244590801438138696'>🏆</tg-emoji> **انتهت التدويرة!**\n"
            f"تم إضافة {data['points']} نقطة إلى رصيدك بنجاح."
        )

# --- تعليمات التشغيل على GitHub ---
def print_github_instructions():
    print("-" * 30)
    print("🚀 تعليمات الاستضافة على GitHub:")
    print("1. أنشئ مستودع (Repository) جديد باسم 'roulette-app'.")
    print("2. ارفع كود الـ HTML الموجود داخل متغير HTML_CONTENT أعلاه في ملف اسمه index.html.")
    print("3. فعل GitHub Pages من الإعدادات.")
    print("4. انسخ الرابط وضعه في متغير WEB_APP_URL داخل هذا الكود.")
    print("-" * 30)

if __name__ == "__main__":
    print_github_instructions()
    print("البوت يعمل الآن... المطور @vipadrian")
    bot.infinity_polling()
    

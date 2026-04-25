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
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>دائرة الأحكام - Mini App</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #6c5ce7;
            --secondary: #a29bfe;
            --accent: #fdcb6e;
            --bg: #f8f9fa;
            --white: #ffffff;
        }
        body {
            font-family: 'Cairo', sans-serif;
            background-color: var(--bg);
            margin: 0; padding: 15px;
            color: #2d3436; text-align: center;
            overflow-x: hidden;
        }
        .header {
            background: linear-gradient(135deg, var(--secondary), var(--primary));
            color: white; padding: 25px;
            border-radius: 20px; margin-bottom: 20px;
            box-shadow: 0 10px 20px rgba(108, 92, 231, 0.2);
        }
        .balance-container {
            display: grid; grid-template-columns: 1fr 1fr; gap: 15px;
            margin-bottom: 25px;
        }
        .card {
            background: var(--white); padding: 15px;
            border-radius: 15px; box-shadow: 0 4px 6px rgba(0,0,0,0.05);
        }
        .card b { display: block; color: var(--primary); font-size: 1.3em; margin-top: 5px; }
        
        .wheel-wrapper {
            position: relative; width: 280px; height: 280px;
            margin: 30px auto;
        }
        .wheel-box {
            width: 100%; height: 100%;
            border: 8px solid var(--white);
            border-radius: 50%; box-shadow: 0 15px 35px rgba(0,0,0,0.1);
            background: conic-gradient(
                #ff7675 0% 25%, 
                #74b9ff 25% 50%, 
                #ffeaa7 50% 75%, 
                #55efc4 75% 100%
            );
            transition: transform 4s cubic-bezier(0.15, 0, 0.15, 1);
        }
        .pointer {
            position: absolute; top: -15px; left: 50%;
            transform: translateX(-50%); width: 0; height: 0;
            border-left: 15px solid transparent; border-right: 15px solid transparent;
            border-top: 25px solid #d63031; z-index: 10;
        }
        
        .btn-spin {
            background: var(--primary); color: white; border: none;
            padding: 15px 45px; border-radius: 50px;
            font-size: 18px; font-weight: bold; cursor: pointer;
            box-shadow: 0 5px 15px rgba(108, 92, 231, 0.4);
            margin-top: 20px; transition: 0.3s;
        }
        .btn-spin:active { transform: scale(0.92); }

        .section-title { 
            margin-top: 35px; color: #636e72; font-size: 1.1em;
            display: flex; align-items: center; justify-content: center; gap: 8px;
        }
        .shop-grid {
            display: grid; grid-template-columns: 1fr 1fr; gap: 12px;
            margin-top: 15px;
        }
        .shop-item {
            background: var(--white); padding: 12px; border-radius: 12px;
            border: 1px solid #dfe6e9; font-size: 0.9em;
            transition: 0.3s;
        }
        .shop-item:active { background: #f1f2f6; }
        .shop-item b { color: #e17055; }
    </style>
</head>
<body>
    <div class="app-container">
        <div class="header">
            <h3>دائرة الأحكام 🎡</h3>
            <p>مرحباً بك يا <span id="player-name">...</span></p>
        </div>

        <div class="balance-container">
            <div class="card">
                <small>رصيدي 💰</small>
                <b id="my-bal">500</b>
            </div>
            <div class="card">
                <small>رصيده 💎</small>
                <b id="his-bal">0</b>
            </div>
        </div>

        <div class="wheel-wrapper">
            <div class="pointer"></div>
            <div class="wheel-box" id="wheel"></div>
        </div>
        
        <button class="btn-spin" onclick="runRoulette()">تدوير العجلة 🔥</button>

        <div class="section-title">
            <span>متجر الأدوات</span> 🛡️
        </div>
        <div class="shop-grid">
            <div class="item shop-item">تذكرة حماية<br><b>100ن</b></div>
            <div class="item shop-item">تغيير ضحية<br><b>50ن</b></div>
        </div>
    </div>

    <script>
        const tg = window.Telegram.WebApp;
        tg.expand();
        tg.ready();
        
        // جلب اسم المستخدم من تليجرام
        document.getElementById('player-name').innerText = tg.initDataUnsafe.user?.first_name || "لاعب";

        function runRoulette() {
            const wheel = document.getElementById('wheel');
            // توليد درجة عشوائية كبيرة للدوران
            const deg = Math.floor(Math.random() * 5000) + 2000;
            wheel.style.transform = `rotate(${deg}deg)`;
            
            // إرسال البيانات للبوت بعد انتهاء الدوران
            setTimeout(() => {
                tg.sendData(JSON.stringify({
                    "action": "spin_done",
                    "points": 25,
                    "status": "success"
                }));
                // إغلاق التطبيق بعد الانتهاء (اختياري)
                // tg.close(); 
            }, 4200);
        }
    </script>
</body>
</html>

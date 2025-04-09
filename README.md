<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>پخش کننده ویدیوی امن با تاریخ انقضا</title>
    <style>
        :root {
            --primary-color: #e74c3c;
            --bg-color: #1a1a1a;
        }
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            -webkit-touch-callout: none;
        }
        body {
            font-family: -apple-system, Tahoma;
            background: var(--bg-color);
            color: white;
            height: 100vh;
            overflow: hidden;
            -webkit-tap-highlight-color: transparent;
        }
        #authOverlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.96);
            backdrop-filter: blur(10px);
            -webkit-backdrop-filter: blur(10px);
            z-index: 1000;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .auth-box {
            background: #222;
            padding: 2rem;
            border-radius: 15px;
            text-align: center;
            border: 2px solid var(--primary-color);
            width: 90%;
            max-width: 400px;
            -webkit-user-select: none;
        }
        #passwordInput {
            padding: 12px;
            margin: 1rem 0;
            width: 100%;
            background: #333;
            border: 1px solid #444;
            color: white;
            border-radius: 8px;
            font-size: 16px;
            -webkit-appearance: none;
        }
        #unlockBtn {
            padding: 12px 30px;
            background: var(--primary-color);
            border: none;
            border-radius: 8px;
            color: white;
            cursor: pointer;
            transition: 0.3s;
            width: 100%;
            font-size: 16px;
            -webkit-tap-highlight-color: transparent;
        }
        #videoContainer {
            display: none;
            width: 100%;
            height: 100vh;
            position: relative;
            -webkit-overflow-scrolling: touch;
        }
        #manualPlay {
            display: none;
            position: fixed;
            bottom: 30px;
            left: 50%;
            transform: translateX(-50%);
            padding: 15px 30px;
            background: var(--primary-color);
            color: white;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            z-index: 1001;
        }
        .expired {
            color: var(--primary-color);
            font-size: 24px;
            text-align: center;
            margin-top: 50px;
        }
        #countdown {
            color: #2ecc71;
            text-align: center;
            margin: 20px 0;
        }
        @media (max-width: 768px) {
            .auth-box { padding: 1.5rem; }
            #videoContainer { height: 60vh; }
        }
    </style>
</head>
<body>

<div id="authOverlay">
    <div class="auth-box">
        <h2 style="color: var(--primary-color); margin-bottom: 1rem;">🔐 سکانس های از فیلم من</h2>
        <input type="password" id="passwordInput" placeholder="رمزی که بهت دادمو بزن" autocomplete="off">
        <button id="unlockBtn" onclick="checkPassword()">باز کردن ویدیو</button>
        <p id="attempts" style="margin-top: 1rem; color: #888; font-size: 0.9rem;"></p>
        <div id="countdown"></div>
    </div>
</div>

<div id="expiredMessage" class="expired" style="display: none;">⏳ دیگه الان میخوای ببینی...نمیخواد ولش کن!</div>

<div id="videoContainer">
    <iframe
        id="videoPlayer"
        src="https://drive.google.com/file/d/1bxcATOCnRysjQHlBcNUDRR8Ct_RugwK0/preview?rm=minimal&embedded=true"
        allow="autoplay; fullscreen; accelerometer; gyroscope; encrypted-media; picture-in-picture"
        allowfullscreen
        webkitallowfullscreen
        mozallowfullscreen
        playsinline
        webkit-playsinline
        gesture="media"
        style="width: 100%; height: 100%; border: none; -webkit-transform: translateZ(0);"
        scrolling="no"
        sandbox="allow-scripts allow-same-origin">
    </iframe>
</div>

<button id="manualPlay" onclick="forcePlay()">▶️ پخش سکانس</button>

<script>
    // ================= تنظیمات سیستم =================
    const SECRET_PASSWORD = "1234"; // 🔑 رمز دلخواه
    const MAX_ATTEMPTS = 3; // حداکثر تلاش مجاز
    const EXPIRATION_DATE = new Date(2025, 3, 9, 03, 59).getTime(); // ⏰ تاریخ انقضا (ماه 0-11)

    let attempts = 0;
    let isValidLink = true;
    let expirationCheckInterval;

    // ================= سیستم بررسی انقضا =================
    function startExpirationCheck() {
        checkExpiration();
        expirationCheckInterval = setInterval(checkExpiration, 10000);
    }

    function checkExpiration() {
        const now = new Date().getTime();
        if(now > EXPIRATION_DATE) {
            handleExpiration();
            return false;
        }
        updateCountdown();
        return true;
    }

    function handleExpiration() {
        document.getElementById('authOverlay').style.display = 'none';
        document.getElementById('expiredMessage').style.display = 'block';
        document.getElementById('videoContainer').style.display = 'none';
        document.getElementById('manualPlay').style.display = 'none';
        clearInterval(expirationCheckInterval);
        isValidLink = false;
    }

    // ================= شمارش معکوس =================
    function updateCountdown() {
        if(!isValidLink) return;

        const now = new Date().getTime();
        const remaining = EXPIRATION_DATE - now;

        const days = Math.floor(remaining / (1000 * 60 * 60 * 24));
        const hours = Math.floor((remaining % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
        const minutes = Math.floor((remaining % (1000 * 60 * 60)) / (1000 * 60));
        const seconds = Math.floor((remaining % (1000 * 60)) / 1000);

        document.getElementById('countdown').innerHTML = 
            `⏳ زمان باقی مانده: ${days} روز ${hours} ساعت ${minutes} دقیقه ${seconds} ثانیه`;
    }

    // ================= سیستم رمز عبور =================
    function checkPassword() {
        if(!checkExpiration()) return;

        attempts++;
        const userPass = document.getElementById('passwordInput').value;
        
        if(userPass === SECRET_PASSWORD) {
            unlockVideo();
        } else {
            handleWrongPassword();
        }
    }

    function unlockVideo() {
        document.getElementById('authOverlay').style.display = 'none';
        document.getElementById('videoContainer').style.display = 'block';
        document.getElementById('manualPlay').style.display = 'block';
        startExpirationCheck();
        
        const iframe = document.getElementById('videoPlayer');
        iframe.src += "&autoplay=1&mute=0";
        
        setTimeout(() => {
            try {
                iframe.contentWindow.postMessage('{"method":"play"}', '*');
            } catch (error) {
                console.log('Auto-play blocked');
            }
        }, 1000);
        
        document.body.addEventListener('touchstart', handleFirstTouch, {once: true});
    }

    function handleFirstTouch() {
        const iframe = document.getElementById('videoPlayer');
        try {
            iframe.contentWindow.postMessage('{"method":"play"}', '*');
            document.getElementById('manualPlay').style.display = 'none';
        } catch (error) {
            console.log('Safari autoplay blocked');
        }
    }

    function handleWrongPassword() {
        const attemptsDisplay = document.getElementById('attempts');
        if(attempts >= MAX_ATTEMPTS) {
            alert('⛔ دسترسی مسدود شد!');
            window.location.href = 'about:blank';
        } else {
            attemptsDisplay.textContent = `تلاش باقی مانده: ${MAX_ATTEMPTS - attempts}`;
            document.getElementById('passwordInput').style.borderColor = 'red';
            setTimeout(() => {
                document.getElementById('passwordInput').style.borderColor = '#444';
            }, 1000);
        }
    }

    // ================= پخش دستی برای iOS =================
    function forcePlay() {
        const iframe = document.getElementById('videoPlayer');
        try {
            iframe.contentWindow.postMessage('{"method":"play"}', '*');
            document.getElementById('manualPlay').style.display = 'none';
        } catch (error) {
            alert('لطفا صفحه را رفرش کنید');
        }
    }

    // ================= امنیت پیشرفته =================
    document.addEventListener('contextmenu', e => e.preventDefault());
    document.addEventListener('keydown', e => {
        if(e.key === 'F12' || (e.ctrlKey && e.shiftKey && e.key === 'I')) e.preventDefault();
    });
    setInterval(() => {
        if(window.outerHeight - window.innerHeight > 200) window.location.reload();
    }, 1000);

    // ================= راه اندازی اولیه =================
    checkExpiration();
</script>

</body>
</html>

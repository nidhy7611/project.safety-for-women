# project.safety-for-women
Girls Safety Web App: Developed a web application with an SOS alert system that shares real-time location and emergency messages with pre-saved contacts using JavaScript and browser APIs.
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SafeHer 🚨 | Your safety, one tap away</title>
    
    <style>
        /* === CSS Variables for Theming === */
        :root {
            --bg-color: #f4f7f6;
            --text-color: #2c3e50;
            --nav-bg: #ffffff;
            --card-bg: #ffffff;
            --card-shadow: 0 8px 16px rgba(0,0,0,0.08);
            --primary-red: #e74c3c;
            --primary-red-hover: #c0392b;
            --success-color: #2ecc71;
            --accent-color: #3498db;
            --font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            --transition: all 0.3s ease;
        }

        [data-theme="dark"] {
            --bg-color: #121212;
            --text-color: #ecf0f1;
            --nav-bg: #1e1e1e;
            --card-bg: #242424;
            --card-shadow: 0 8px 16px rgba(0,0,0,0.4);
        }

        * { margin: 0; padding: 0; box-sizing: border-box; scroll-behavior: smooth; }
        body {
            font-family: var(--font-family); background-color: var(--bg-color);
            color: var(--text-color); transition: var(--transition);
            overflow: hidden; /* Prevent scrolling while locked */
        }

        /* =========================================
           📱 REALISTIC SMARTPHONE LOCK SCREEN
        ========================================= */
        #lockScreen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(135deg, #1c1c1c 0%, #000000 100%);
            z-index: 9999; display: flex; flex-direction: column; align-items: center; justify-content: space-between;
            color: white; padding: 4rem 2rem 3rem 2rem; backdrop-filter: blur(10px);
            transition: opacity 0.5s ease, visibility 0.5s ease;
        }

        .lock-header { text-align: center; margin-bottom: 2rem; }
        .lock-icon { font-size: 1.5rem; margin-bottom: 10px; display: block; }
        .lock-time { font-size: 5rem; font-weight: 200; line-height: 1; margin-bottom: 10px; letter-spacing: -2px; }
        .lock-date { font-size: 1.2rem; font-weight: 400; color: #d1d1d6; }

        .passcode-area { display: flex; flex-direction: column; align-items: center; flex-grow: 1; justify-content: center; }
        .lock-prompt { font-size: 1.1rem; margin-bottom: 25px; letter-spacing: 1px; font-weight: 500; }
        
        .dots-container { display: flex; gap: 20px; margin-bottom: 40px; height: 16px; }
        .dot { width: 14px; height: 14px; border-radius: 50%; border: 1.5px solid #ffffff; transition: background 0.1s; }
        .dot.filled { background-color: #ffffff; }
        
        .shake { animation: shake 0.4s cubic-bezier(.36,.07,.19,.97) both; }
        @keyframes shake {
            10%, 90% { transform: translate3d(-1px, 0, 0); }
            20%, 80% { transform: translate3d(2px, 0, 0); }
            30%, 50%, 70% { transform: translate3d(-4px, 0, 0); }
            40%, 60% { transform: translate3d(4px, 0, 0); }
        }

        .numpad { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px 30px; margin-bottom: 2rem; }
        .num-btn {
            width: 75px; height: 75px; border-radius: 50%; border: none;
            background: rgba(255, 255, 255, 0.15); color: white; font-size: 2rem; font-weight: 400;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            cursor: pointer; backdrop-filter: blur(5px); transition: background 0.2s, transform 0.1s;
        }
        .num-btn:active { background: rgba(255, 255, 255, 0.4); transform: scale(0.95); }
        .num-letters { font-size: 0.6rem; font-weight: 600; letter-spacing: 2px; margin-top: -5px; color: #d1d1d6; }
        .num-btn.empty { background: transparent; cursor: default; }
        .num-btn.action { font-size: 1.2rem; font-weight: 500; background: transparent; }

        .emergency-text { color: var(--primary-red); font-weight: 600; font-size: 1rem; cursor: pointer; margin-top: auto; }

        /* =========================================
           MAIN APP (Hidden by default)
        ========================================= */
        #appContent { display: none; }

        nav {
            position: sticky; top: 0; background: var(--nav-bg); padding: 1rem 5%;
            display: flex; justify-content: space-between; align-items: center;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1); z-index: 1000;
        }
        .logo { font-size: 1.5rem; font-weight: 800; color: var(--primary-red); text-decoration: none; }
        .nav-links { display: flex; gap: 1.5rem; align-items: center; }
        .nav-links a { text-decoration: none; color: var(--text-color); font-weight: 600; }
        .theme-toggle { background: none; border: none; font-size: 1.5rem; cursor: pointer; }

        section { padding: 5rem 5%; min-height: 80vh; display: flex; flex-direction: column; align-items: center; text-align: center; }
        h2.section-title { font-size: 2.5rem; margin-bottom: 1rem; }
        p.tagline { font-size: 1.2rem; color: #7f8c8d; margin-bottom: 3rem; }

        .sos-container { position: relative; margin-bottom: 2rem; }
        .sos-btn {
            width: 220px; height: 220px; border-radius: 50%; background: var(--primary-red);
            color: white; font-size: 3.5rem; font-weight: 900; border: none; cursor: pointer;
            box-shadow: 0 0 0 0 rgba(231, 76, 60, 0.7); animation: pulse 1.5s infinite;
            transition: transform 0.2s; z-index: 2; position: relative;
        }
        .sos-btn:active { transform: scale(0.95); }
        @keyframes pulse {
            0% { transform: scale(0.95); box-shadow: 0 0 0 0 rgba(231, 76, 60, 0.7); }
            70% { transform: scale(1); box-shadow: 0 0 0 40px rgba(231, 76, 60, 0); }
            100% { transform: scale(0.95); box-shadow: 0 0 0 0 rgba(231, 76, 60, 0); }
        }

        .voice-btn {
            background: var(--accent-color); color: white; border: none; padding: 12px 24px;
            border-radius: 25px; font-size: 1rem; cursor: pointer; margin-bottom: 2rem;
            display: flex; align-items: center; gap: 8px;
        }
        
        .status-card {
            display: none; background: var(--card-bg); padding: 1.5rem; border-radius: 12px;
            box-shadow: var(--card-shadow); max-width: 500px; width: 100%; border-left: 5px solid var(--success-color);
            animation: slideDown 0.5s ease;
        }
        @keyframes slideDown { from { opacity: 0; transform: translateY(-20px); } to { opacity: 1; transform: translateY(0); } }
        .status-alert { color: var(--success-color); font-weight: bold; font-size: 1.2rem; margin-bottom: 10px; }
        .status-desc { font-size: 0.9rem; color: var(--text-color); margin-bottom: 10px; }
        .coords-box { background: #eee; color: #333; padding: 10px; border-radius: 6px; font-family: monospace; }

        .action-btn { background: var(--text-color); color: var(--bg-color); padding: 15px 30px; border: none; border-radius: 30px; font-size: 1.2rem; cursor: pointer; font-weight: bold; }
        
        #fakeCallScreen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.95); color: white; z-index: 2000;
            display: none; flex-direction: column; justify-content: center; align-items: center;
        }
        .caller-img { width: 120px; height: 120px; border-radius: 50%; object-fit: cover; margin-bottom: 20px; border: 3px solid white; }
        .caller-name { font-size: 3rem; margin-bottom: 10px; }
        .call-status { font-size: 1.2rem; color: #bdc3c7; margin-bottom: 50px; }
        .call-buttons { display: flex; gap: 40px; }
        .call-btn { width: 80px; height: 80px; border-radius: 50%; border: none; font-size: 2rem; cursor: pointer; display: flex; justify-content: center; align-items: center; color: white; }
        .btn-decline { background: var(--primary-red); }
        .btn-accept { background: var(--success-color); animation: pulse 1.5s infinite; }

        .map-wrapper { width: 100%; max-width: 900px; height: 450px; border-radius: 15px; overflow: hidden; box-shadow: var(--card-shadow); }
        iframe { width: 100%; height: 100%; border: none; }

        .tips-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 2rem; width: 100%; max-width: 1100px; }
        .tip-card { background: var(--card-bg); padding: 2rem; border-radius: 15px; box-shadow: var(--card-shadow); text-align: left; transition: transform 0.3s; }
        .tip-card:hover { transform: translateY(-8px); }
        .tip-icon { font-size: 3rem; margin-bottom: 1rem; }

        @media (max-width: 768px) {
            .nav-links { display: none; }
            .sos-btn { width: 180px; height: 180px; font-size: 2.5rem; }
            section { padding: 3rem 5%; }
        }
    </style>
</head>
<body>

    <div id="lockScreen">
        <div class="lock-header">
            <span class="lock-icon">🔒</span>
            <div class="lock-time" id="clockTime">12:00</div>
            <div class="lock-date" id="clockDate">Monday, January 1</div>
        </div>

        <div class="passcode-area">
            <div class="lock-prompt" id="lockPromptText">Enter Passcode</div>
            <div class="dots-container" id="dotsContainer">
                <div class="dot"></div>
                <div class="dot"></div>
                <div class="dot"></div>
                <div class="dot"></div>
            </div>

            <div class="numpad">
                <button class="num-btn" onclick="pressKey('1')">1 <span class="num-letters"></span></button>
                <button class="num-btn" onclick="pressKey('2')">2 <span class="num-letters">ABC</span></button>
                <button class="num-btn" onclick="pressKey('3')">3 <span class="num-letters">DEF</span></button>
                <button class="num-btn" onclick="pressKey('4')">4 <span class="num-letters">GHI</span></button>
                <button class="num-btn" onclick="pressKey('5')">5 <span class="num-letters">JKL</span></button>
                <button class="num-btn" onclick="pressKey('6')">6 <span class="num-letters">MNO</span></button>
                <button class="num-btn" onclick="pressKey('7')">7 <span class="num-letters">PQRS</span></button>
                <button class="num-btn" onclick="pressKey('8')">8 <span class="num-letters">TUV</span></button>
                <button class="num-btn" onclick="pressKey('9')">9 <span class="num-letters">WXYZ</span></button>
                <button class="num-btn empty"></button>
                <button class="num-btn" onclick="pressKey('0')">0</button>
                <button class="num-btn action" onclick="deleteKey()">Cancel</button>
            </div>
        </div>

        <div class="emergency-text" onclick="triggerSOSFromLock()">Emergency</div>
    </div>

    <div id="appContent">
        <nav>
            <a href="#home" class="logo">SafeHer 🚨</a>
            <div class="nav-links">
                <a href="#home">Home</a>
                <a href="#fake-call">Fake Call</a>
                <a href="#safe-places">Safe Places</a>
                <a href="#safety-tips">Tips</a>
            </div>
            <button class="theme-toggle" id="themeToggle" onclick="toggleTheme()">🌙</button>
        </nav>

        <section id="home">
            <h2 class="section-title">SafeHer 🚨</h2>
            <p class="tagline">Your safety, one tap away.</p>

            <button class="voice-btn" onclick="startVoiceRecognition()" id="voiceBtn">
                🎤 Activate Voice Help ("Help me")
            </button>

            <div class="sos-container">
                <button class="sos-btn" onclick="triggerSOS()">SOS</button>
            </div>

            <div class="status-card" id="statusCard">
                <div class="status-alert">✅ Emergency Triggered!</div>
                <div class="status-desc">Your live location has been successfully securely fetched and shared with your trusted contacts and local authorities.</div>
                <div class="coords-box" id="coordsDisplay">Fetching coordinates...</div>
            </div>
        </section>

        <section id="fake-call" style="background: var(--nav-bg);">
            <h2 class="section-title">Discreet Escape</h2>
            <p class="tagline">Uncomfortable situation? Trigger a fake incoming call to excuse yourself.</p>
            <button class="action-btn" onclick="initiateFakeCall()">📱 Trigger Fake Call</button>
        </section>

        <div id="fakeCallScreen">
            <img src="https://images.unsplash.com/photo-1544005313-94ddf0286df2?w=400&h=400&fit=crop" alt="Caller Image" class="caller-img">
            <div class="caller-name">Mom ❤️</div>
            <div class="call-status" id="callStatusText">Incoming call...</div>
            <div class="call-buttons">
                <button class="call-btn btn-decline" onclick="endFakeCall()">📞</button>
                <button class="call-btn btn-accept" id="acceptBtn" onclick="acceptFakeCall()">📞</button>
            </div>
            <audio id="ringtone" loop>
                <source src="https://www.soundjay.com/phone/sounds/telephone-ring-04.mp3" type="audio/mpeg">
            </audio>
        </div>

        <section id="safe-places">
            <h2 class="section-title">Nearby Safe Places</h2>
            <p class="tagline">Locate the nearest police stations immediately.</p>
            <div class="map-wrapper">
                <iframe src="https://maps.google.com/maps?q=police+stations+near+me&t=&z=13&ie=UTF8&iwloc=&output=embed" loading="lazy"></iframe>
            </div>
        </section>

        <section id="safety-tips" style="background: var(--nav-bg);">
            <h2 class="section-title">Awareness & Prevention</h2>
            <div class="tips-grid">
                <div class="tip-card">
                    <div class="tip-icon">📍</div>
                    <h3>Share Your Ride</h3>
                    <p>Always share your cab details and live location with a trusted contact before starting your journey.</p>
                </div>
                <div class="tip-card">
                    <div class="tip-icon">🤫</div>
                    <h3>Discreet SOS</h3>
                    <p>Learn to use the SafeHer hidden lock screen feature to trigger an SOS without alerting an attacker.</p>
                </div>
                <div class="tip-card">
                    <div class="tip-icon">🎒</div>
                    <h3>Defense Items</h3>
                    <p>Keep a personal alarm or pepper spray accessible in your pocket.</p>
                </div>
                <div class="tip-card">
                    <div class="tip-icon">🗣️</div>
                    <h3>Voice Commands</h3>
                    <p>Use the Voice Help feature to activate emergency protocols hands-free.</p>
                </div>
            </div>
        </section>
    </div>

    <script>
        /* =========================================
           1. Lock Screen Logic (Click & Keyboard)
        ========================================= */
        let currentPin = "";
        const dots = document.querySelectorAll('.dot');
        const dotsContainer = document.getElementById('dotsContainer');
        const promptText = document.getElementById('lockPromptText');
        const appContent = document.getElementById('appContent');
        const lockScreen = document.getElementById('lockScreen');

        // Clock Update
        function updateClock() {
            const now = new Date();
            let hours = now.getHours();
            let minutes = now.getMinutes();
            hours = hours % 12;
            hours = hours ? hours : 12; 
            minutes = minutes < 10 ? '0' + minutes : minutes;
            document.getElementById('clockTime').textContent = `${hours}:${minutes}`;
            const options = { weekday: 'long', month: 'long', day: 'numeric' };
            document.getElementById('clockDate').textContent = now.toLocaleDateString('en-US', options);
        }
        setInterval(updateClock, 1000);
        updateClock();

        function updateDots() {
            dots.forEach((dot, index) => {
                if (index < currentPin.length) {
                    dot.classList.add('filled');
                } else {
                    dot.classList.remove('filled');
                }
            });
        }

        function pressKey(num) {
            if (currentPin.length < 4) {
                currentPin += num;
                updateDots();
            }
            if (currentPin.length === 4) {
                setTimeout(checkPin, 100); 
            }
        }

        function deleteKey() {
            currentPin = currentPin.slice(0, -1);
            updateDots();
            promptText.textContent = "Enter Passcode";
        }

        function checkPin() {
            if (currentPin === "1234") {
                unlockApp(false);
            } else if (currentPin === "0000") {
                unlockApp(true); // Unlock AND trigger SOS
            } else {
                dotsContainer.classList.add('shake');
                promptText.textContent = "Wrong Passcode";
                setTimeout(() => {
                    dotsContainer.classList.remove('shake');
                    currentPin = "";
                    updateDots();
                }, 400);
            }
        }

        function unlockApp(triggerEmergency) {
            lockScreen.style.opacity = '0';
            setTimeout(() => {
                lockScreen.style.display = 'none';
                appContent.style.display = 'block';
                document.body.style.overflow = 'auto'; 
                if (triggerEmergency) { triggerSOS(); }
            }, 500);
        }

        function triggerSOSFromLock() {
            unlockApp(true);
        }

        // PHYSICAL KEYBOARD SUPPORT
        document.addEventListener('keydown', function(event) {
            if (lockScreen.style.display !== 'none') {
                const key = event.key;
                if (key >= '0' && key <= '9') {
                    pressKey(key);
                } else if (key === 'Backspace' || key === 'Delete') {
                    event.preventDefault(); // Prevents browser from going back a page
                    deleteKey();
                }
            }
        });

        /* =========================================
           2. App Features (Theme, SOS, Call)
        ========================================= */
        function toggleTheme() {
            const body = document.body;
            const btn = document.getElementById('themeToggle');
            if (body.getAttribute('data-theme') === 'dark') {
                body.removeAttribute('data-theme'); btn.textContent = '🌙';
            } else {
                body.setAttribute('data-theme', 'dark'); btn.textContent = '☀️';
            }
        }

        function triggerSOS() {
            const statusCard = document.getElementById('statusCard');
            const coordsDisplay = document.getElementById('coordsDisplay');
            statusCard.style.display = 'block';
            coordsDisplay.textContent = "Acquiring GPS Signal...";

            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(
                    (position) => {
                        const lat = position.coords.latitude;
                        const lon = position.coords.longitude;
                        coordsDisplay.innerHTML = `📍 Lat: ${lat.toFixed(5)} <br> 📍 Lon: ${lon.toFixed(5)}`;
                    },
                    (error) => { coordsDisplay.textContent = "Error: Location Permissions blocked."; }
                );
            } else {
                coordsDisplay.textContent = "Geolocation not supported.";
            }
        }

        function startVoiceRecognition() {
            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            const voiceBtn = document.getElementById('voiceBtn');
            if (SpeechRecognition) {
                const recognition = new SpeechRecognition();
                recognition.onstart = () => { voiceBtn.innerHTML = "🎙️ Listening..."; voiceBtn.style.background = "#e67e22"; };
                recognition.onresult = (e) => {
                    if (e.results[0][0].transcript.toLowerCase().includes('help')) { triggerSOS(); alert("SOS Triggered!"); }
                    voiceBtn.innerHTML = "🎤 Activate Voice Help"; voiceBtn.style.background = "var(--accent-color)";
                };
                recognition.onerror = () => { voiceBtn.innerHTML = "🎤 Activate Voice Help"; voiceBtn.style.background = "var(--accent-color)"; };
                recognition.start();
            } else {
                alert("Speech Recognition API not supported in this browser.");
            }
        }

        const fakeCallScreen = document.getElementById('fakeCallScreen');
        const ringtone = document.getElementById('ringtone');
        const acceptBtn = document.getElementById('acceptBtn');
        const callStatusText = document.getElementById('callStatusText');
        let timer;

        function initiateFakeCall() {
            setTimeout(() => {
                fakeCallScreen.style.display = 'flex'; callStatusText.textContent = "Incoming call..."; acceptBtn.style.display = 'flex';
                ringtone.currentTime = 0; ringtone.play().catch(e => console.log("Audio blocked."));
            }, 2000);
        }

        function acceptFakeCall() {
            ringtone.pause(); acceptBtn.style.display = 'none'; callStatusText.textContent = "00:01";
            let sec = 1;
            timer = setInterval(() => { sec++; callStatusText.textContent = `00:${sec < 10 ? '0'+sec : sec}`; }, 1000);
        }

        function endFakeCall() {
            ringtone.pause(); clearInterval(timer); fakeCallScreen.style.display = 'none';
        }
    </script>
</body>
</html>

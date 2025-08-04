<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Ad Reward System</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #1f1c2c, #928dab);
      color: #fff;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }
    .container {
      text-align: center;
      background: linear-gradient(to right, #232526, #414345);
      padding: 25px;
      border-radius: 15px;
      box-shadow: 0 8px 20px rgba(0, 0, 0, 0.6);
      width: 320px;
    }
    .container h1 {
      font-size: 24px;
      background: linear-gradient(to right, #00c6ff, #0072ff);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      margin-bottom: 15px;
    }
    .developer {
      font-size: 13px;
      background: linear-gradient(45deg, #11998e, #38ef7d);
      padding: 6px 12px;
      border-radius: 8px;
      margin-bottom: 20px;
      display: inline-block;
      color: #fff;
      font-weight: bold;
      box-shadow: 0 3px 0 #0e6b5f;
      cursor: pointer;
    }
    .developer:hover {
      opacity: 0.9;
    }
    .stats p, .user-info p {
      font-size: 15px;
      margin: 6px 0;
    }
    .progress-circle {
      width: 90px;
      height: 90px;
      border: 6px solid #ff9800;
      border-radius: 50%;
      background: #2a2a2a;
      display: flex;
      justify-content: center;
      align-items: center;
      margin: 12px auto;
      box-shadow: inset 0 0 10px #ff9800, 0 0 10px rgba(255, 152, 0, 0.5);
    }
    .progress-circle span {
      font-size: 18px;
      font-weight: bold;
      color: #ffeb3b;
    }

    .buttons button {
      width: 90%;
      margin: 8px 0;
      padding: 12px;
      font-size: 15px;
      border: none;
      border-radius: 8px;
      font-weight: bold;
      background: linear-gradient(to bottom, #43cea2, #185a9d);
      color: white;
      box-shadow: 0 5px 0 #0e4c7d;
      transition: all 0.2s ease;
      cursor: pointer;
    }
    .buttons button:hover {
      transform: translateY(-2px);
      background: linear-gradient(to bottom, #f7971e, #ffd200);
      box-shadow: 0 7px 0 #d17d00;
    }
    .buttons button:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }

    .withdraw-section {
      margin-top: 20px;
      display: none;
      padding: 20px;
      background: linear-gradient(to right, #373b44, #4286f4);
      border-radius: 10px;
      box-shadow: 0 4px 15px rgba(0, 0, 0, 0.5);
    }
    .withdraw-section h3 {
      color: #ffcc00;
      margin-bottom: 15px;
    }
    .withdraw-section input, .withdraw-section select {
      width: 100%;
      padding: 12px;
      margin: 8px 0;
      border-radius: 6px;
      border: none;
      background-color: #f0f0f0;
      color: #333;
      font-weight: bold;
    }
    .withdraw-section input:focus, .withdraw-section select:focus {
      border: 2px solid #00e676;
      outline: none;
    }
    .withdraw-section button {
      background: linear-gradient(to bottom, #00b09b, #96c93d);
      color: white;
      width: 100%;
      padding: 12px;
      font-size: 16px;
      margin-top: 10px;
      border: none;
      border-radius: 6px;
      font-weight: bold;
      box-shadow: 0 4px 0 #4e8c21;
    }
    .withdraw-section button:hover {
      background: linear-gradient(to bottom, #f7971e, #ffd200);
    }
    #withdraw-status {
      color: #ff4d4d;
      font-size: 14px;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>MoneTag Telegram Bot</h1>
    <div class="developer" onclick="window.location.href='https://t.me/nsmtech_bd'">
      Developer by NSM TECH BD
    </div>
    <div class="user-info">
      <p>Welcome, <span id="user-name"></span></p>
    </div>

    <div class="stats">
      <p>Watched Ads: <span id="watched-ads">0</span></p>
      <p>Earned Points: <span id="earned-points">0</span></p>
    </div>

    <div class="progress-circle">
      <span id="ads-progress">0%</span>
    </div>

    <div class="buttons">
      <button onclick="watchAd()">🎬 Watch Ad</button>
      <button id="auto-ad-btn" onclick="startAutoAds()">⚙️ Auto Show Ads</button>
      <button id="stop-auto-btn" onclick="stopAutoAds()" disabled>⛔ Stop Auto Ads</button>
      <button onclick="showWithdrawForm()">💸 Withdraw Points</button>
    </div>

    <div class="withdraw-section" id="withdraw-section">
      <h3>Withdrawal Request</h3>
      <input type="number" id="withdraw-amount" placeholder="Enter Points to Withdraw" />
      <select id="payment-method">
        <option value="bkash">Bkash</option>
        <option value="nagad">Nagad</option>
        <option value="manual">Manual</option>
      </select>
      <input type="text" id="withdraw-phone" placeholder="Enter Phone Number" />
      <button onclick="withdrawPoints()">✅ Withdraw</button>
      <p id="withdraw-status"></p>
    </div>
  </div>

  <script src='//libtl.com/sdk.js' data-zone='9611027' data-sdk='show_9611027'></script>


  <script>
    const MIN_WITHDRAW_POINTS = 5;
    const ADMIN_USER_ID = 7694112804;
    const BOT_TOKEN = "YOUR_BOT_TOKEN_HERE";
    let watchedAdsCount = 0;
    let earnedPoints = 0.00;
    let autoAdInterval;

    const telegramUserName = "@exampleUser";
    document.getElementById("user-name").textContent = telegramUserName;

    if (localStorage.getItem('watchedAdsCount')) {
      watchedAdsCount = parseInt(localStorage.getItem('watchedAdsCount'));
      earnedPoints = parseFloat(localStorage.getItem('earnedPoints'));
      document.getElementById('watched-ads').textContent = watchedAdsCount;
      document.getElementById('earned-points').textContent = earnedPoints.toFixed(2);
    }

    function watchAd() {
      if (typeof show_9611027 === 'function') {
        show_9611027().then(() => {
          watchedAdsCount++;
          earnedPoints += 0.5;
          document.getElementById('watched-ads').textContent = watchedAdsCount;
          document.getElementById('earned-points').textContent = earnedPoints.toFixed(2);
          localStorage.setItem('watchedAdsCount', watchedAdsCount);
          localStorage.setItem('earnedPoints', earnedPoints.toFixed(2));
          updateProgressCircle();
        });
      }
    }

    function updateProgressCircle() {
      const percent = Math.min((watchedAdsCount / 10) * 100, 100);
      document.getElementById('ads-progress').textContent = `${Math.floor(percent)}%`;
    }

    function startAutoAds() {
      autoAdInterval = setInterval(watchAd, 5000);
      document.getElementById('auto-ad-btn').disabled = true;
      document.getElementById('stop-auto-btn').disabled = false;
    }

    function stopAutoAds() {
      clearInterval(autoAdInterval);
      document.getElementById('auto-ad-btn').disabled = false;
      document.getElementById('stop-auto-btn').disabled = true;
    }

    function showWithdrawForm() {
      document.getElementById('withdraw-section').style.display = 'block';
    }

    function withdrawPoints() {
      const amount = parseFloat(document.getElementById('withdraw-amount').value);
      const method = document.getElementById('payment-method').value;
      const phone = document.getElementById('withdraw-phone').value;

      if (amount < MIN_WITHDRAW_POINTS) {
        document.getElementById('withdraw-status').textContent = `Minimum withdrawal is ${MIN_WITHDRAW_POINTS} points.`;
        return;
      }

      if (amount > earnedPoints) {
        document.getElementById('withdraw-status').textContent = `Insufficient balance.`;
        return;
      }

      earnedPoints -= amount;
      document.getElementById('earned-points').textContent = earnedPoints.toFixed(2);
      localStorage.setItem('earnedPoints', earnedPoints.toFixed(2));

      const msg = `Withdrawal Request from ${telegramUserName}:\nAmount: ${amount} points\nPayment Method: ${method}\nPhone: ${phone}`;
      sendWithdrawRequestToAdmin(msg);

      document.getElementById('withdraw-status').textContent = 'Withdrawal request sent!';
      document.getElementById('withdraw-amount').value = '';
      document.getElementById('withdraw-phone').value = '';
    }

    function sendWithdrawRequestToAdmin(message) {
      fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendMessage?chat_id=${ADMIN_USER_ID}&text=${encodeURIComponent(message)}`)
        .then(res => res.json())
        .then(data => console.log("Sent:", data))
        .catch(err => console.error("Error:", err));
    }
  </script>
</body>
</html>

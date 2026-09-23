# PickOne_コード完成版

## プロジェクト

PickOne_2026.9.23

## 情報の種類

コード完成版

## 検索キーワード

PickOne、kira-nihongo-online、コード完成版

## 保存場所

GitHub / GAS

## 内容

<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>PickOne</title>
  <link href="https://fonts.googleapis.com/css2?family=M+PLUS+Rounded+1c&display=swap" rel="stylesheet">
  <style>
    body {
      font-family: 'M PLUS Rounded 1c', sans-serif;
      padding: 20px;
      background-color: #fff;
      color: #000;
      transition: background 0.5s ease;
    }

    textarea, input {
      width: 100%;
      margin: 10px 0;
      background-color: #f0f0f0;
      color: #000;
      border: 1px solid #ccc;
      padding: 8px;
    }

    button {
      padding: 10px 20px;
      font-size: 16px;
      background-color: #007acc;
      color: white;
      border: none;
      cursor: pointer;
      margin-right: 10px;
    }

    button:hover {
      background-color: #005fa3;
    }

    #result {
      font-size: 24px;
      margin-top: 20px;
      opacity: 0;
      transition: opacity 0.6s ease, transform 0.6s ease;
      transform: scale(0.9);
    }

    #result.show {
      opacity: 1;
      transform: scale(1);
    }

    #history {
      margin-top: 20px;
      font-size: 16px;
    }

    #history li {
      margin: 4px 0;
    }

　　.header {
　　  display: flex;
　　  align-items: center;
　　  gap: 12px;
　　  margin-bottom: 25px;
　　}

　　.header-logo {
　　  width: 48px;
　　  height: 48px;
　　  object-fit: contain;
　　}

　　.header-title {
　　  margin: 0;
　　  font-size: 32px;
　　  line-height: 1;
　　  font-weight: bold;
　　}

　　.header-subtitle {
　　  margin: 5px 0 0;
　　  font-size: 14px;
　　  color: #666;
　　}

  </style>
</head>
<body id="body">

 <div style="display:flex; align-items:center; gap:10px; margin-bottom:25px;">
   <img
     src="トータルロゴ.png"
     alt="PickOne"
     style="width:35px; height:35px; object-fit:contain;"
   >

   <div>
     <h1 id="title" style="margin:0; font-size:32px; line-height:32px;">
       PickOne
     </h1>

   </div>
 </div>

  <h2>リスト</h2>
  <textarea id="nameList" rows="5" placeholder="名前を改行で入力してください"></textarea>
  <label>表示までの秒数：<input type="number" id="delayName" value="0" min="0"></label>
  <button id="drawBtn" onclick="drawName()">抽選する</button>
  <button onclick="clearHistory()">履歴をクリア</button>

  <div id="result"></div>
  <ul id="history"></ul>

  <script>

    const DATA_URL = 'https://script.google.com/macros/s/AKfycbwye_JuuU9AppUZtOHbxAZw41ROsUfhlF1BghxxnvNUS0ObWAg8MkbI7iXN1IsayFjb/exec';

    async function loadNamesFromSheet() {
      try {
        const response = await fetch(DATA_URL);

        if (!response.ok) {
          throw new Error('HTTP ' + response.status);
        }

        const data = await response.json();

        document.getElementById("nameList").value =
          data.names.join("\n");

      } catch (error) {
        console.error('JSON取得失敗:', error);
        alert('出席者データの取得に失敗しました。');
      }
    }

    let voiceJP = null;

    function loadVoices() {
      const voices = speechSynthesis.getVoices();

      voiceJP =
        voices.find(v => v.name.includes("七海")) ||
        voices.find(v => v.name.includes("Nanami")) ||
        voices.find(v => v.lang === "ja-JP");

      console.log("選択音声:", voiceJP?.name);
    }

    speechSynthesis.onvoiceschanged = loadVoices;
    loadVoices();

    let usedNames = [];

    window.addEventListener('load', loadNamesFromSheet);

    function drawName() {
      const allNames = document.getElementById("nameList").value
        .split("\n")
        .map(n => n.trim())
        .filter(n => n !== "");
      const delay = parseInt(document.getElementById("delayName").value) * 1000;
      const resultDiv = document.getElementById("result");
      const drawBtn = document.getElementById("drawBtn");

      if (allNames.length === 0) {
        resultDiv.textContent = "⚠️ 名前を入力してください。";
        resultDiv.classList.add("show");
        return;
      }

      const remainingNames = allNames.filter(n => !usedNames.includes(n));

      if (remainingNames.length === 0) {
        resultDiv.textContent = "✅ 全員が抽選されました。履歴をクリアして再開してください。";
        resultDiv.classList.add("show");
        drawBtn.disabled = false;
        return;
      }

      resultDiv.textContent = "抽選中...";
      resultDiv.classList.remove("show");
      drawBtn.disabled = true;

      setTimeout(() => {
        const result = remainingNames[Math.floor(Math.random() * remainingNames.length)];
        usedNames.push(result);

        resultDiv.textContent = `🎉 抽選結果：${result}`;
        resultDiv.classList.add("show");
        drawBtn.disabled = false;

        const history = document.getElementById("history");
        const li = document.createElement("li");
        li.textContent = `✅ ${result}`;
        history.appendChild(li);

        const utterance = new SpeechSynthesisUtterance(`抽選結果は ${result} です`);

        utterance.lang = "ja-JP";

        if (voiceJP) {
          utterance.voice = voiceJP;
        }

        utterance.rate = 1.0;
        utterance.pitch = 1.0;

        speechSynthesis.cancel();
        speechSynthesis.speak(utterance);
      }, delay);
    }

    function clearHistory() {
      document.getElementById("history").innerHTML = "";
      document.getElementById("result").textContent = "";
      usedNames = [];
    }
  </script>
</body>
</html>

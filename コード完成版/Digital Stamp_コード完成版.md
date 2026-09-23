# Digital Stamp_コード完成版

## プロジェクト

Digital Stamp_2026.9.23

## 情報の種類

コード完成版

## 検索キーワード

Digital Stamp、digital-stamp、コード完成版

## 保存場所

GitHub

## 内容

<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Digital Stamp</title>
  <style>

    body {
      font-family: sans-serif;
      background-color: #ffffff;
      padding: 20px;
      width: 800px;
      margin: 0 auto;
      box-sizing: border-box;
    }

    .container {
      display: flex;
      gap: 30px;
      align-items: flex-start;
    }
    .canvas-area {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    canvas {
      background-color: #ffffff;
      border: 1px solid #ccc;
    }
    .controls {
      flex: 1;
      max-width: 400px;
    }
    input, select {
      margin: 5px 0;
      padding: 5px;
      font-size: 16px;
      width: 100%;
    }
    label {
      font-weight: bold;
      margin-top: 10px;
      display: block;
    }
    .date-row {
      display: flex;
      align-items: center;
      gap: 10px;
    }
    .date-row input[type="text"] {
      flex: 1;
    }

    h2 {
      display: flex;
      align-items: center;
      gap: 10px;
    }

    h2 img {
      width: 32px;
      height: 32px;
      object-fit: contain;
    }

    .save-controls {
      margin-top: 8px;
      display: flex;
      flex-direction: column;
      gap: 4px;
    }

    .save-controls button {
      width: 70px;
      margin: 0 2px;
    }

    .save-controls button.saved {
      background-color: #cc0000;
      color: #ffffff;
    }

  </style>
</head>
<body>
  <h2><img src="トータルロゴ.png" alt="Logo"> Digital Stamp</h2>

  <div class="container">
    <div class="canvas-area">
      <canvas id="stampCanvas" width="300" height="300"></canvas>
      <button id="downloadButton">Download PNG</button>

   <div class="save-controls">
     <div>
       <button id="saveButton1">Save 1</button>
       <button id="loadButton1">Load 1</button>
     </div>
     <div>
       <button id="saveButton2">Save 2</button>
       <button id="loadButton2">Load 2</button>
     </div>
     <div>
       <button id="saveButton3">Save 3</button>
       <button id="loadButton3">Load 3</button>
     </div>
   </div>

    </div>

    <div class="controls">
      <label>Top Message</label>
      <input type="text" id="message" value="Sample Message">
      <input type="range" id="messageSize" min="10" max="40" value="18">
      <input type="range" id="messageY" min="20" max="100" value="100">

      <label>Main Text</label>
      <input type="text" id="kanji" value="Sample">
      <input type="range" id="kanjiSize" min="40" max="120" value="64">
      <input type="range" id="kanjiY" min="100" max="200" value="170">

      <label>Date</label>
      <div class="date-row">
        <input type="text" id="date" value="">
        <label><input type="checkbox" id="autoDate" checked> Use today's date</label>
      </div>
      <input type="range" id="dateSize" min="10" max="40" value="20">
      <input type="range" id="dateY" min="200" max="280" value="230">

      <label>Color</label>
      <input type="color" id="color" value="#cc0000">
    </div>
  </div>

  <script>
    function getTodayFormatted() {
      const today = new Date();
      const yy = String(today.getFullYear()).slice(2);
      const mm = String(today.getMonth() + 1).padStart(2, '0');
      const dd = String(today.getDate()).padStart(2, '0');
      return `'${yy}.${mm}.${dd}`;
    }

    function drawStamp() {
      const canvas = document.getElementById("stampCanvas");
      const ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = "#ffffff";
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      const message = document.getElementById("message").value;
      const messageSize = document.getElementById("messageSize").value;
      const messageY = document.getElementById("messageY").value;

      const kanji = document.getElementById("kanji").value;
      const kanjiSize = document.getElementById("kanjiSize").value;
      const kanjiY = document.getElementById("kanjiY").value;

      const autoDate = document.getElementById("autoDate").checked;
      const dateInput = document.getElementById("date");
      const date = autoDate ? getTodayFormatted() : dateInput.value;
      if (autoDate) dateInput.value = date;

      const dateSize = document.getElementById("dateSize").value;
      const dateY = document.getElementById("dateY").value;

      const color = document.getElementById("color").value;

      ctx.strokeStyle = color;
      ctx.lineWidth = 8;
      ctx.beginPath();
      ctx.arc(150, 150, 120, 0, 2 * Math.PI);
      ctx.stroke();

      ctx.fillStyle = color;
      ctx.font = `bold ${messageSize}px sans-serif`;
      ctx.textAlign = "center";
      ctx.fillText(message, 150, parseInt(messageY));

      ctx.font = `bold ${kanjiSize}px serif`;
      ctx.fillText(kanji, 150, parseInt(kanjiY));

      ctx.font = `bold ${dateSize}px sans-serif`;
      ctx.fillText(date, 150, parseInt(dateY));

      saveSettings();
    }

    document.getElementById("downloadButton").addEventListener("click", () => {
      const canvas = document.getElementById("stampCanvas");
      const link = document.createElement("a");

      link.download = "digital-stamp.png";
      link.href = canvas.toDataURL("image/png");
      link.click();
    });

    function saveSettings() {
      const settings = {
        messageSize: document.getElementById("messageSize").value,
        messageY: document.getElementById("messageY").value,
        kanjiSize: document.getElementById("kanjiSize").value,
        kanjiY: document.getElementById("kanjiY").value,
        dateSize: document.getElementById("dateSize").value,
        dateY: document.getElementById("dateY").value
      };
      localStorage.setItem("stampSettings", JSON.stringify(settings));
    }

    function loadSavedSettings() {
      const saved = localStorage.getItem("stampSettings");
      if (saved) {
        const s = JSON.parse(saved);
        document.getElementById("messageSize").value = s.messageSize;
        document.getElementById("messageY").value = s.messageY;
        document.getElementById("kanjiSize").value = s.kanjiSize;
        document.getElementById("kanjiY").value = s.kanjiY;
        document.getElementById("dateSize").value = s.dateSize;
        document.getElementById("dateY").value = s.dateY;
      }
    }

    window.onload = () => {
      document.getElementById("date").value = getTodayFormatted();
      loadSavedSettings();
      drawStamp();
    };

    document.querySelectorAll("input").forEach(input => {
      input.addEventListener("input", drawStamp);
    });

    function getCurrentSettings() {
      return {
        message: document.getElementById("message").value,
        messageSize: document.getElementById("messageSize").value,
        messageY: document.getElementById("messageY").value,

        kanji: document.getElementById("kanji").value,
        kanjiSize: document.getElementById("kanjiSize").value,
        kanjiY: document.getElementById("kanjiY").value,

        date: document.getElementById("date").value,
        autoDate: document.getElementById("autoDate").checked,
        dateSize: document.getElementById("dateSize").value,
        dateY: document.getElementById("dateY").value,

        color: document.getElementById("color").value
      };
    }

    function saveToSlot(slotNumber) {
      const settings = getCurrentSettings();

      localStorage.setItem(
        `stampSlot${slotNumber}`,
        JSON.stringify(settings)
      );

      document
        .getElementById(`saveButton${slotNumber}`)
        .classList.add("saved");
    }

    function loadFromSlot(slotNumber) {
      const saved = localStorage.getItem(`stampSlot${slotNumber}`);

      if (!saved) {
        return;
      }

      const settings = JSON.parse(saved);

      document.getElementById("message").value = settings.message;
      document.getElementById("messageSize").value = settings.messageSize;
      document.getElementById("messageY").value = settings.messageY;

      document.getElementById("kanji").value = settings.kanji;
      document.getElementById("kanjiSize").value = settings.kanjiSize;
      document.getElementById("kanjiY").value = settings.kanjiY;

      document.getElementById("date").value = settings.date;
      document.getElementById("autoDate").checked = settings.autoDate;
      document.getElementById("dateSize").value = settings.dateSize;
      document.getElementById("dateY").value = settings.dateY;

      document.getElementById("color").value = settings.color;

      drawStamp();
    }

    function updateSaveButtons() {
      for (let i = 1; i <= 3; i++) {
        const button = document.getElementById(`saveButton${i}`);

        if (localStorage.getItem(`stampSlot${i}`)) {
          button.classList.add("saved");
        }
      }
    }

    document.getElementById("saveButton1").addEventListener("click", () => {
      saveToSlot(1);
    });

    document.getElementById("loadButton1").addEventListener("click", () => {
      loadFromSlot(1);
    });

    document.getElementById("saveButton2").addEventListener("click", () => {
      saveToSlot(2);
    });

    document.getElementById("loadButton2").addEventListener("click", () => {
      loadFromSlot(2);
    });

    document.getElementById("saveButton3").addEventListener("click", () => {
      saveToSlot(3);
    });

    document.getElementById("loadButton3").addEventListener("click", () => {
      loadFromSlot(3);
    });

    updateSaveButtons();

  </script>
</body>
</html>

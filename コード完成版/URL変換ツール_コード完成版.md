# URL変換ツール_コード完成版

## プロジェクト

URL変換ツール_2026.9.23

## 情報の種類

コード完成版

## 検索キーワード

URL変換ツール、url-converter-tool、コード 完成版

## 保存場所

GitHub

## 内容

<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>URL変換ツール</title>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      padding: 20px;
      font-family: sans-serif;
      background: #ffffff;
      color: #222222;
    }

    .container {
      width: 100%;
      max-width: 900px;
      margin: 0 auto;
    }

    .header {
      display: flex;
      align-items: center;
      gap: 12px;
      margin-bottom: 25px;
    }

    .header img {
      width: 42px;
      height: 42px;
      object-fit: contain;
    }

    .header h1 {
      margin: 0;
      font-size: 28px;
    }

    .mode-buttons {
      display: flex;
      gap: 10px;
      margin-bottom: 20px;
    }

    .mode-buttons button {
      flex: 1;
      padding: 14px 10px;
      border: 1px solid #cccccc;
      border-radius: 8px;
      background: #f5f5f5;
      font-size: 16px;
      cursor: pointer;
    }

  #downloadButton.active {
    background: #7e57c2;
    color: #ffffff;
  }

  #slideButton.active {
    background: #fbbc04;
    color: #222222;
  }

  #dropboxButton.active {
    background: #0061ff;
    color: #ffffff;
  }

  #appendButton.active {
    background: #34a853;
    color: #ffffff;
  }
    .tool-panel {
      width: 100%;
    }

    .tool-panel p {
      margin: 0 0 8px;
      font-size: 15px;
    }

    input {
      width: 100%;
      padding: 12px;
      font-size: 16px;
      border: 1px solid #cccccc;
      border-radius: 6px;
      margin-bottom: 10px;
    }

    .convert-button {
      width: 100%;
      padding: 12px;
      font-size: 16px;
      border: none;
      border-radius: 6px;
      background: #333333;
      color: #ffffff;
      cursor: pointer;
    }

    #output {
      width: 100%;
      min-height: 50px;
      margin-top: 20px;
      padding: 12px;
      background: #f0f0f0;
      border-radius: 6px;
      word-break: break-all;
    }

    .button-row {
      display: flex;
      gap: 10px;
      margin-top: 10px;
    }

    .button-row button {
      flex: 1;
      padding: 12px;
      font-size: 16px;
      border: 1px solid #cccccc;
      border-radius: 6px;
      background: #ffffff;
      cursor: pointer;
    }

    @media (max-width: 600px) {
      body {
        padding: 15px;
      }

      .header h1 {
        font-size: 24px;
      }

      .mode-buttons button {
        font-size: 14px;
      }
    }
  </style>
</head>

<body>

<div class="container">

  <div class="header">
    <img src="トータルロゴ.png" alt="Logo">
    <h1>URL変換ツール</h1>
  </div>


  <div class="mode-buttons">

    <button
      type="button"
      id="downloadButton"
      class="active"
      onclick="selectDownload()">
      ダウンロードURL変換
    </button>

    <button
      type="button"
      id="slideButton"
      onclick="selectSlide()">
      Google スライド
    </button>

    <button
      type="button"
      id="dropboxButton"
      onclick="selectDropbox()">
      Dropbox
    </button>

    <button
      type="button"
      id="appendButton"
      onclick="selectAppend()">
      URL末尾追加
    </button>

  </div>


  <div class="tool-panel">

    <p>共有URLを入力してください：</p>

    <input
      type="text"
      id="mainInput"
      placeholder="URLを入力してください">


    <p>追加する文字列：</p>

    <input
      type="text"
      id="appendText"
      placeholder="追加する文字列を入力してください">


    <button
      type="button"
      class="convert-button"
      onclick="convert()">
      変換する
    </button>

  </div>


  <div id="output"></div>


  <div class="button-row">

    <button
      type="button"
      onclick="copyUrl()">
      コピー
    </button>

    <button
      type="button"
      onclick="clearAll()">
      クリア
    </button>

  </div>

</div>


<script>

/* =========================================
   現在のモード
   ========================================= */

let currentMode = "download";


/* =========================================
   ボタン表示
   ========================================= */

function updateButtons(activeId) {

  document
    .getElementById("downloadButton")
    .classList.remove("active");

  document
    .getElementById("slideButton")
    .classList.remove("active");

  document
    .getElementById("dropboxButton")
    .classList.remove("active");

  document
    .getElementById("appendButton")
    .classList.remove("active");

  document
    .getElementById(activeId)
    .classList.add("active");
}

/* =========================================
   Google Driveを選択
   ========================================= */

function selectDownload() {

  currentMode = "download";

  updateButtons("downloadButton");

  document.getElementById("output").innerText = "";
}


/* =========================================
   Google スライドを選択
   ========================================= */

function selectSlide() {

  currentMode = "slide";

  updateButtons("slideButton");

  document.getElementById("output").innerText = "";
}


/* =========================================
   Dropboxを選択
   ========================================= */

function selectDropbox() {

  currentMode = "dropbox";

  updateButtons("dropboxButton");

  document.getElementById("output").innerText = "";
}


/* =========================================
   URL末尾追加を選択
   ========================================= */

function selectAppend() {

  currentMode = "append";

  updateButtons("appendButton");

  document.getElementById("output").innerText = "";
}


/* =========================================
   変換
   ========================================= */

function convert() {

  if (currentMode === "download") {

    convertDownload();

    return;
  }


  if (currentMode === "slide") {

    convertSlide();

    return;
  }


  if (currentMode === "dropbox") {

    convertDropbox();

    return;
  }


  if (currentMode === "append") {

    convertAppend();

    return;
  }

}


/* =========================================
   Google Drive
   ========================================= */

function convertDownload() {

  const input =
    document
      .getElementById("mainInput")
      .value
      .trim();

  const output =
    document.getElementById("output");

  if (!input) {

    output.innerText =
      "URLを入力してください。";

    return;
  }

  const match =
    input.match(
      /presentation\/d\/([a-zA-Z0-9_-]+)/
    );

  if (!match) {

    output.innerText =
      "正しいGoogleスライドURLではありません。";

    return;
  }

  const presentationId =
    match[1];

  output.innerText =
    "https://docs.google.com/presentation/d/" +
    presentationId +
    "/export/pdf";
}

/* =========================================
   Google スライド
   ========================================= */

function convertSlide() {

  const input =
    document.getElementById("mainInput").value.trim();

  const output =
    document.getElementById("output");

  if (!input) {
    output.innerText =
      "URLを入力してください。";
    return;
  }

  const match =
    input.match(
      /presentation\/d\/([a-zA-Z0-9_-]+)/
    );

  const slideMatch =
    input.match(
      /slide=([^&#]+)/
    );

  if (!match) {
    output.innerText =
      "❌ 正しいGoogleスライドURLではありません。";
    return;
  }

  const id = match[1];

  const slide =
    slideMatch ? slideMatch[1] : "id.p";

  const result =
    `https://docs.google.com/presentation/d/${id}/present?slide=${slide}`;

  output.innerText =
    result;
}

/* =========================================
   Dropbox
   ========================================= */

function convertDropbox() {

  const input =
    document.getElementById("mainInput").value.trim();

  const output =
    document.getElementById("output");

  if (!input) {
    output.innerText =
      "URLを入力してください。";
    return;
  }

  const result =
    input.replace(/[?&]dl=0(?:&|$)/, function(match) {
      return match.startsWith("?") ? "?dl=1" : "&dl=1";
    });

  output.innerText =
    result;
}


/* =========================================
   URL末尾追加
   ========================================= */

function convertAppend() {

  const input =
    document
      .getElementById("mainInput")
      .value
      .trim();


  const addition =
    document
      .getElementById("appendText")
      .value;


  if (!input) {

    document.getElementById("output").innerText =
      "URLを入力してください。";

    return;
  }


  if (!addition) {

    document.getElementById("output").innerText =
      "追加する文字列を入力してください。";

    return;
  }


  document.getElementById("output").innerText =
    input + addition;
}


/* =========================================
   コピー
   ========================================= */

function copyUrl() {

  const text =
    document
      .getElementById("output")
      .innerText
      .trim();


  if (!text) {

    alert("コピーするURLがありません。");

    return;
  }


  navigator.clipboard
    .writeText(text)
    .then(function() {

      alert("コピーしました！");

    })
    .catch(function() {

      alert("コピーできませんでした。");

    });
}


/* =========================================
   クリア
   ========================================= */

function clearAll() {

  document.getElementById("mainInput").value = "";

  document.getElementById("appendText").value = "";

  document.getElementById("output").innerText = "";
}

</script>

</body>
</html>

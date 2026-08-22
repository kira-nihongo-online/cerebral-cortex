# cerebral-cortex_コード 完成版

## プロジェクト

Cerebral Cortex

## 情報の種類

コード完成版

## 検索キーワード

Cerebral Cortex 、2026.8.22

## 保存場所

Cerebral Cortex／コード完成版

## 内容

<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Cerebral Cortex</title>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #f5f6f8;
      color: #222;
    }

    header {
      background: #202124;
      color: white;
      padding: 20px 30px;
    }

    header h1 {
      margin: 0;
      font-size: 24px;
    }

    header p {
      margin: 5px 0 0;
      color: #bbb;
      font-size: 13px;
    }

    main {
      max-width: 1100px;
      margin: 30px auto;
      padding: 0 20px;
    }

    section {
      background: white;
      border-radius: 8px;
      padding: 25px;
      margin-bottom: 25px;
      box-shadow: 0 1px 4px rgba(0,0,0,0.08);
    }

    h2 {
      margin-top: 0;
      font-size: 18px;
    }

    .form-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 15px;
    }

    .full {
      grid-column: 1 / -1;
    }

    label {
      display: block;
      font-size: 13px;
      margin-bottom: 5px;
      font-weight: bold;
    }

    input,
    select,
    textarea {
      width: 100%;
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 5px;
      font-size: 14px;
    }

    textarea {
      min-height: 120px;
      resize: vertical;
    }

    button {
      border: none;
      border-radius: 5px;
      padding: 10px 18px;
      cursor: pointer;
      font-size: 14px;
    }

    .save-button {
      background: #222;
      color: white;
      margin-top: 15px;
    }

    .search-box {
      margin-bottom: 20px;
    }

    .memory {
      border: 1px solid #ddd;
      border-radius: 6px;
      margin-bottom: 12px;
      overflow: hidden;
    }

    .memory-title {
      font-weight: bold;
      font-size: 16px;
      padding: 15px;
      cursor: pointer;
    }

    .memory-title:hover {
      background: #f5f5f5;
    }

    .memory-details {
      display: none;
      padding: 0 15px 15px;
    }

    .memory.open .memory-details {
      display: block;
    }

    .memory-meta {
      color: #666;
      font-size: 12px;
      margin-top: 5px;
    }

    .memory-content {
      margin-top: 10px;
      white-space: pre-wrap;
      font-size: 14px;
    }

    .empty {
      color: #888;
      font-size: 14px;
    }

    @media (max-width: 700px) {
      .form-grid {
        grid-template-columns: 1fr;
      }

      .full {
        grid-column: auto;
      }
    }

    .header-brand {
      display: flex;
      align-items: center;
      gap: 12px;
    }

    .header-brand img {
      width: 42px;
      height: 42px;
      object-fit: contain;
    }

    .header-brand h1 {
      margin: 0;
      font-size: 24px;
    }

    .header-brand p {
      margin: 5px 0 0;
      color: #bbb;
      font-size: 13px;
    }    

  </style>
</head>

<body>

<header>
  <div class="header-brand">
    <img src="トータルロゴ.png" alt="Cerebral Cortex">
    <div>
      <h1>Cerebral Cortex</h1>
      <p>Knowledge & Development Memory System</p>
    </div>
  </div>
</header>
<main>

  <!-- 登録 -->
  <section>
    <h2>記憶を登録</h2>

    <div class="form-grid">

      <div>
        <label>プロジェクト名</label>
        <input id="project" placeholder="例：朗読プロンプター">
      </div>

      <div>
        <label>情報の種類</label>
        <select id="type">
          <option value="共通ルール">共通ルール</option>
          <option value="コード完成版">コード完成版</option>
          <option value="設計">設計</option>
          <option value="仕様">仕様</option>
          <option value="決定事項">決定事項</option>
          <option value="その他">その他</option>
        </select>
      </div>

      <div class="full">
        <label>保存ファイル名</label>
        <input id="title" placeholder="例：TTS処理の仕様">
      </div>

      <div class="full">
        <label>検索キーワード</label>
        <input id="keywords" placeholder="例：TTS、音声、MP3、朗読">
      </div>

      <div class="full">
        <label>保存場所</label>
        <input id="location" placeholder="例：GitHub / projects / reading-prompter">
      </div>

      <div class="full">
        <label>内容</label>
        <textarea id="content" placeholder="ここに記憶する内容を入力"></textarea>
      </div>

    </div>

    <button class="save-button" onclick="saveMemory()">
      記憶を登録
    </button>

  <!-- 検索 -->
  <section>
    <h2>記憶を検索</h2>

    <!--==================================================*
     * 記憶検索
     *==================================================-->

    <div class="search-box">
      <input
        id="search"
        placeholder="プロジェクト名・種類・キーワードなどで検索"
      >

      <button type="button" id="searchButton">
        検索
      </button>
    </div>

    <div id="memoryList"></div>
  </section>

</main>


<script>
/*==================================================*
 * Cerebral Cortex - saveMemory()
 *==================================================*/
  const STORAGE_KEY = "cerebralCortexMemories";

  function getMemories() {
    return JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
  }


/*==================================================*
 * Cerebral Cortex - saveMemory()
 * 保存成功後にフォームをクリア
 *==================================================*/

async function saveMemory() {

  const memory = {
    project: document.getElementById("project").value,
    type: document.getElementById("type").value,
    title: document.getElementById("title").value,
    keywords: document.getElementById("keywords").value,
    location: document.getElementById("location").value,
    content: document.getElementById("content").value
  };

  if (!memory.project || !memory.title || !memory.content) {
    alert("プロジェクト名・タイトル・内容を入力してください。");
    return;
  }

  const url = 'https://script.google.com/macros/s/AKfycbwzS3XaYHIeLODSTjMDYfunnjUfSRWXgMKtRJPHoQUGyx8x6QMMoLgdLeV0XgduLWad/exec';

  try {

    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'text/plain;charset=utf-8'
      },
      body: JSON.stringify(memory)
    });

    const result = await response.json();

    console.log('Apps Script response:', result);

    if (!result.success) {
      alert('保存に失敗しました。');
      return;
    }

    document.getElementById("project").value = "";
    document.getElementById("title").value = "";
    document.getElementById("keywords").value = "";
    document.getElementById("location").value = "";
    document.getElementById("content").value = "";

    alert('記憶を保存しました。');

  } catch (error) {

    console.error(error);
    alert('送信エラー：' + error.message);

  }
}

/*==================================================*
 * Cerebral Cortex - displayMemories()
 * 空検索時は一覧を表示しない
 *==================================================*/

async function displayMemories() {

  const list = document.getElementById("memoryList");
  const search = document
    .getElementById("search")
    .value
    .trim()
    .toLowerCase();

  if (!search) {
    list.innerHTML = "";
    return;
  }

  const url = 'https://script.google.com/macros/s/AKfycbwzS3XaYHIeLODSTjMDYfunnjUfSRWXgMKtRJPHoQUGyx8x6QMMoLgdLeV0XgduLWad/exec';

  try {

    const response = await fetch(url);
    const result = await response.json();

    if (!result.success) {
      list.innerHTML =
        '<div class="empty">記憶の取得に失敗しました。</div>';
      return;
    }

    const memories = result.memories;

    console.log("取得した記憶:", memories);
    console.log("検索文字:", search);
    console.log("1件目のデータ:", memories[0]);

    const filtered = memories.filter(memory => {

      const text = [
        memory.type,
        memory.title,
        memory.content,
        memory.path
      ]
      .join(" ")
      .toLowerCase();

      return text.includes(search);
    });

    if (filtered.length === 0) {
      list.innerHTML =
        '<div class="empty">該当する記憶はありません。</div>';
      return;
    }

    /*==================================================*
     * Cerebral Cortex - 検索結果表示
     *==================================================*/

    list.innerHTML = filtered
      .map((memory, index) => `

        <div class="memory">

          <div
            class="memory-title"
            onclick="toggleMemory(${index})"
          >
            ${escapeHtml(memory.title)}
          </div>

          <div class="memory-details">

            <div class="memory-meta">
              ${escapeHtml(memory.type)}
              ／
              ${escapeHtml(memory.path)}
            </div>

            <div class="memory-content">
              ${escapeHtml(memory.content)}
            </div>

            <button
              type="button"
              class="gpt-send-button"
              onclick='sendToGPT(${JSON.stringify(memory).replace(/'/g, "&#39;")})'
            >
              GPTに送信
            </button>

          </div>

        </div>

      `)
      .join("");

    window.cortexSearchResults = filtered;
  } catch (error) {

    console.error(error);

    list.innerHTML =
      '<div class="empty">記憶の取得に失敗しました。</div>';
  }
}

/*==================================================*
 * Cerebral Cortex - 検索ボタン
 *==================================================*/

document.getElementById("searchButton").addEventListener("click", function() {
  displayMemories();
});

/*==================================================*
 * Cerebral Cortex - Enterキー検索
 *==================================================*/

document.getElementById("search").addEventListener("keydown", function(event) {

  if (event.key === "Enter") {
    displayMemories();
  }

});

/*==================================================*
 * Cerebral Cortex - escapeHtml() ＋ 初期表示
 *==================================================*/
  function escapeHtml(text) {

    return text
      .replace(/&/g, "&amp;")
      .replace(/</g, "&lt;")
      .replace(/>/g, "&gt;")
      .replace(/"/g, "&quot;")
      .replace(/'/g, "&#039;");
  }

  function toggleMemory(index) {

    const memories = document.querySelectorAll(".memory");
    const memory = memories[index];

    if (!memory) {
      return;
    }

    memory.classList.toggle("open");

    loadMemoryContent(index);
  }

  function loadMemoryContent(index) {

    const memory = window.cortexSearchResults[index];

    if (!memory) {
      return;
    }

    const content = memory.content;

    const getSection = (name, nextName) => {

      const pattern = nextName
        ? new RegExp(
            `## ${name}\\s*\\n([\\s\\S]*?)(?=## ${nextName}\\s*\\n|$)`
          )
        : new RegExp(
            `## ${name}\\s*\\n([\\s\\S]*)`
          );

      const match = content.match(pattern);

      return match ? match[1].trim() : "";
    };

    document.getElementById("project").value =
      getSection("プロジェクト", "情報の種類");

    document.getElementById("type").value =
      getSection("情報の種類", "検索キーワード");

    document.getElementById("title").value =
      memory.title;

    document.getElementById("keywords").value =
      getSection("検索キーワード", "保存場所");

    document.getElementById("location").value =
      getSection("保存場所", "内容");

    document.getElementById("content").value =
      getSection("内容");

  }

    // ↓↓↓ ここに追加 ↓↓↓

    function sendToGPT(memory) {

      const text =
  `【Cerebral Cortex】

  タイトル：${memory.title}
  情報の種類：${memory.type}
  保存場所：${memory.path}

  内容：
  ${memory.content}`;

      navigator.clipboard.writeText(text)
        .then(() => {
          alert("GPTに送信する情報をコピーしました。");
        })
        .catch(error => {
          console.error(error);
          alert("コピーに失敗しました。");
        });
    }

    // ↑↑↑ ここまで追加 ↑↑↑


  displayMemories();

</script>

</body>
</html>

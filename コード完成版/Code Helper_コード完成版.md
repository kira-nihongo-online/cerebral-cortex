# Code Helper_コード完成版

## プロジェクト

Code Helper_2026.8.22

## 情報の種類

コード完成版

## 検索キーワード

Code Helper、code-helper、コード完成版

## 保存場所

GitHub

## 内容

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Code Helper</title>

<style>
body {
  margin: 0;
  display: flex;
  height: 100vh;
  font-family: Arial;
}

/* =========================
   左（コード）
========================= */
#code {
  width: 50%;
  padding: 10px;
  font-family: monospace;
  background: #1e1e1e;
  color: #ffffff;
  border: none;
  outline: none;
  resize: none;
  min-width: 120px;
  caret-color: #00ffcc;
　line-height: 1.4;
}

#code::selection {
  background: yellow;
  color: black;
}

/* =========================
   リサイズ
========================= */
#resizer {
  width: 5px;
  cursor: col-resize;
  background: #ccc;
}

/* =========================
   右
========================= */
#sidebar {
  width: 50%;
  min-width: 200px;
  padding: 10px;
  border-left: 1px solid #ccc;
  overflow-y: auto;
  box-sizing: border-box;
}

/* =========================
   Functions
========================= */
.func {
  padding: 6px;
  border-bottom: 1px solid #eee;
}

.func:hover {
  background: #f0f0f0;
}
</style>
</head>

<body>

<textarea
  id="code"
  placeholder="ここにコード貼る"
  autocorrect="off"
  autocapitalize="off"
  spellcheck="false"
></textarea>

<div id="resizer"></div>

<div id="sidebar">

  <!-- 検索 -->
  <input id="search" placeholder="検索" style="width:100%; padding:5px;">
  <button onclick="clearSearch()">クリア</button>
  <button onclick="nextMatch()">次へ</button>
  <span id="searchInfo"></span>

  <br><br>

  <!-- 操作 -->
  <button onclick="copyAll()">全コピー</button>
　<button onclick="clearAll()">全クリア</button>

  <br><br>

  <button onclick="undo()">← 戻る</button>
  <button onclick="redo()">→ 進む</button>


　<br><br>

　<div style="display: flex; flex-direction: column; gap: 2px; margin-top: 5px;">
  <button onclick="saveSnapshot()">保存</button>
  <button onclick="loadSnapshot()">復元</button>
  <span id="saveStatus" style="font-weight: bold;"></span>
  </div>

  <div id="list"></div>

</div>

<script>

/* =========================
   初期
========================= */
const codeArea = document.getElementById('code');
const listDiv = document.getElementById('list');

let functions = [];
let matches = [];
let currentIndex = -1;

/* =========================
   ■ コード履歴（Undo / Redo）
========================= */
let codeHistory = [];
let historyIndex = -1;
let saveTimer = null;

/* =========================
   ■ 見出しデータ
========================= */
let sections = [];

/* =========================
   入力
========================= */
codeArea.addEventListener('input', () => {
  parseFunctions();
  parseSections(); // ← これ追加

  renderSections(); // ← ★ここに追加 

  const status = document.getElementById('saveStatus');
  if (status) {
    status.textContent = "● 未保存";
    status.style.color = "red";
  }

// ===== 履歴保存（遅延） =====
clearTimeout(saveTimer);

saveTimer = setTimeout(() => {
  const current = codeArea.value;

  // 同じ内容なら保存しない
  if (historyIndex >= 0 && codeHistory[historyIndex] === current) return;

  // 進む履歴があったら削除
  codeHistory = codeHistory.slice(0, historyIndex + 1);

  // 追加
  codeHistory.push(current);
  historyIndex++;

  // 上限15
  if (codeHistory.length > 15) {
    codeHistory.shift();
    historyIndex--;
  }

}, 500);
});

/* =========================
   ■ Functions解析
========================= */
function parseFunctions() {
  const code = codeArea.value;
  functions = [];

  const regex = /function\s+([a-zA-Z0-9_]+)/g;
  let match;

  while ((match = regex.exec(code)) !== null) {
    functions.push({
      name: match[1],
      start: match.index
    });
  }  
}

/* =========================
   ■ 見出し解析
========================= */
function parseSections() {
  const code = codeArea.value;
  sections = [];

  const regex = /\/\*\s*=+\s*\n\s*■\s*(.*?)\n[\s\S]*?=+\s*\*\//g;
  let match;

  while ((match = regex.exec(code)) !== null) {
    sections.push({
      name: match[1].trim(),
      start: match.index
    });
  }  
}

/* =========================
   表示
========================= */
function renderList() {
  listDiv.innerHTML = '';

  functions.forEach(fn => {
    const div = document.createElement('div');
    div.className = 'func';
    div.textContent = fn.name;

    div.onclick = () => {
      codeArea.focus();
      codeArea.setSelectionRange(fn.start, fn.start + 20);
    };

    listDiv.appendChild(div);
  });
}


/* =========================
   ■ 見出し表示
========================= */
function renderSections() {
  listDiv.innerHTML = '';

  // ===== Sections =====
  const header = document.createElement('h3');
  header.textContent = "Sections";
  listDiv.appendChild(header);

  sections.forEach(sec => {
    const div = document.createElement('div');
    div.className = 'func';
    div.textContent = sec.name;

    // ★ ここ追加（ジャンプ＋ハイライト）
    div.onclick = () => {
      codeArea.focus();
      codeArea.setSelectionRange(sec.start, sec.start + 50);
    };

    listDiv.appendChild(div);
  });

  // ===== Functions =====
  const funcHeader = document.createElement('h3');
  funcHeader.textContent = "Functions";
  listDiv.appendChild(funcHeader);

  functions.forEach(fn => {
    const div = document.createElement('div');
    div.className = 'func';
    div.textContent = fn.name;

    div.onclick = () => {
      codeArea.focus();
      codeArea.setSelectionRange(fn.start, fn.start + 20);
    };

    listDiv.appendChild(div);
  });
}

/* =========================
   検索
========================= */
function searchCode() {
  const key = document.getElementById('search').value.toLowerCase();
  const text = codeArea.value.toLowerCase();

  matches = [];
  currentIndex = -1;

  if (!key) return;

  let pos = text.indexOf(key);
  while (pos !== -1) {
    matches.push(pos);
    pos = text.indexOf(key, pos + 1);
  }

  if (matches.length > 0) {
    currentIndex = 0;
    jumpToMatch();
  }

  updateInfo();
}

function nextMatch() {
  if (matches.length === 0) return;
  currentIndex = (currentIndex + 1) % matches.length;
  jumpToMatch();
  updateInfo();
}

function jumpToMatch() {
  const key = document.getElementById('search').value;
  const pos = matches[currentIndex];

  codeArea.focus();
  codeArea.setSelectionRange(pos, pos + key.length);

  // ★ ダミーdivで正確な位置取得
  const div = document.createElement("div");
  const style = window.getComputedStyle(codeArea);

  div.style.position = "absolute";
  div.style.visibility = "hidden";
  div.style.whiteSpace = "pre-wrap";
  div.style.wordWrap = "break-word";
  div.style.font = style.font;
  div.style.padding = style.padding;
  div.style.width = codeArea.clientWidth + "px";
  div.style.lineHeight = style.lineHeight;

  div.textContent = codeArea.value.substring(0, pos);

  document.body.appendChild(div);

  const height = div.scrollHeight;

  document.body.removeChild(div);

  // ★ 上寄せ＋少し余白
  codeArea.scrollTop = height - 20;
}

function updateInfo() {
  document.getElementById('searchInfo').textContent =
    matches.length ? (currentIndex + 1) + "/" + matches.length : "0/0";
}

document.getElementById('search').addEventListener('input', searchCode);

/* =========================
   クリア
========================= */
function clearSearch() {
  document.getElementById('search').value = "";
  matches = [];
  currentIndex = -1;
  updateInfo();
}

/* =========================
   コピー
========================= */
function copyAll() {
  navigator.clipboard.writeText(codeArea.value);
}

function clearAll() {
  if (!confirm("全部消す？")) return;
  codeArea.value = "";
  clearSearch();
}

/* =========================
   リサイズ
========================= */
const resizer = document.getElementById('resizer');
const sidebar = document.getElementById('sidebar');

let isResizing = false;
let startX, startWidth;

resizer.addEventListener('mousedown', (e) => {
  isResizing = true;
  startX = e.clientX;
  startWidth = codeArea.offsetWidth;
});

document.addEventListener('mousemove', (e) => {
  if (!isResizing) return;

  const dx = e.clientX - startX;
  let newWidth = startWidth + dx;

  const total = document.body.clientWidth;

  if (newWidth < 120) newWidth = 120;
  if (newWidth > total - 200) newWidth = total - 200;

  codeArea.style.width = newWidth + 'px';
  sidebar.style.width = (total - newWidth - 5) + 'px';
});

document.addEventListener('mouseup', () => {
  isResizing = false;
});

/* =========================
   初期サイズ
========================= */
window.onload = () => {
  const total = window.innerWidth;
  codeArea.style.width = total * 0.9 + 'px';
  sidebar.style.width = total * 0.1 + 'px';

  // ★ これ追加
  codeHistory = [codeArea.value];
  historyIndex = 0;
};

/* =========================
   ■ スマートペースト（列ズレ防止）
========================= */
codeArea.addEventListener('paste', (e) => {

  e.preventDefault();

  const paste = (e.clipboardData || window.clipboardData).getData('text');

  const start = codeArea.selectionStart;
  const end = codeArea.selectionEnd;

  const text = codeArea.value;

  // ===== 現在行開始位置 =====
  const lineStart = text.lastIndexOf('\n', start - 1) + 1;

  // ===== 現在行テキスト =====
  const currentLine = text.substring(lineStart, start);

  // ===== 貼り付け先インデント =====
  const baseIndentMatch = currentLine.match(/^\s*/);
  const baseIndent = baseIndentMatch ? baseIndentMatch[0] : '';

  // ===== 行分割 =====
  let lines = paste.replace(/\r/g, '').split('\n');

  // ===== 空行だけなら普通貼り付け =====
  if (lines.length === 0) return;

  // ===== 最小インデント取得 =====
  let minIndent = null;

  lines.forEach(line => {

    // 空行無視
    if (!line.trim()) return;

    const match = line.match(/^(\s*)/);
    const indent = match ? match[1].length : 0;

    if (minIndent === null || indent < minIndent) {
      minIndent = indent;
    }
  });

  if (minIndent === null) minIndent = 0;

  // ===== インデント再構築 =====
  lines = lines.map((line, index) => {

    // 空行維持
    if (!line.trim()) return '';

    // 共通インデント削除
    const trimmed = line.slice(minIndent);

    // 1行目
    if (index === 0) {
      return trimmed;
    }

    // 2行目以降
    return baseIndent + trimmed;
  });

  const adjusted = lines.join('\n');

  // ===== 挿入 =====
  codeArea.value =
    text.substring(0, start) +
    adjusted +
    text.substring(end);

  // ===== カーソル位置 =====
  const pos = start + adjusted.length;

  codeArea.selectionStart = pos;
  codeArea.selectionEnd = pos;

  // ===== 更新 =====
  parseFunctions();
  parseSections();
  renderSections();
});

/* =========================
   復元
========================= */
function loadSnapshot() {
  const saved = localStorage.getItem("code_snapshot");

  if (!saved) {
    alert("保存データないよ");
    return;
  }

  codeArea.value = saved;

  // 状態更新
  parseFunctions();
  searchCode();

  alert("復元した👍");
}


/* =========================
   手動保存（安全版）
========================= */
function saveSnapshot() {
  const code = codeArea.value;
  localStorage.setItem("code_snapshot", code);

  const status = document.getElementById('saveStatus');
  if (status) {
    status.textContent = "✔ 保存済み";
    status.style.color = "green";

    setTimeout(() => {
      status.textContent = "";
    }, 2000);
  }
}

/* =========================
   ■ 自動バックアップ
========================= */
setInterval(() => {
  const code = codeArea.value;

  // 空は保存しない
  if (!code) return;

  localStorage.setItem("code_backup", code);

}, 5000);

/* =========================
   ■ 戻る（Undo）
========================= */
function undo() {
  if (historyIndex <= 0) return;

  historyIndex--;
  codeArea.value = codeHistory[historyIndex];

  parseFunctions();
  parseSections(); 
}

/* =========================
   ■ 進む（Redo）
========================= */
function redo() {
  if (historyIndex >= codeHistory.length - 1) return;

  historyIndex++;
  codeArea.value = codeHistory[historyIndex];

  parseFunctions();
  parseSections();  
}

/* =========================
   ■ 起動時 自動復元（確実版）
========================= */
const backup = localStorage.getItem("code_backup");
if (backup) {
  codeArea.value = backup;
  parseFunctions();
  parseSections();
}

</script>

</body>
</html>

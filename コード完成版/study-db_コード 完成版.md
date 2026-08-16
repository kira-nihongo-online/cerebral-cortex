# study-db_コード 完成版

## プロジェクト

Study DB

## 情報の種類

コード完成版

## 検索キーワード

Study DB コード 完成版 GitHub

## 保存場所

Study DB／コード完成版

## 内容

<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
  margin:0;
  height:100vh;
  background:#f5f5f5;
  overflow-y:auto;
}

#card{
  width:100%;
  height:calc(100vh - 60px);
  margin-top:0;

  display:flex;
  flex-direction:column;

}

#front, #back{
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width:100%;
}

.top{
  display:flex;
  justify-content:center;
  align-items:flex-end;
  height: 200px;

  margin-top:60px;  /* ←ここに移動 */

}

.bottom{
  display:flex;
  justify-content:center;
  align-items:center;
  flex-direction:column;
  text-align:center;
  height: 300px;
}

#jpImg{
  height: 180px;      /* ←好きなサイズに調整OK */
  object-fit: contain;
}

#meaningFront,
#meaningBack{
  max-width:300px;
  max-height:220px;

  width:auto;
  height:auto;

  object-fit:contain;

  margin:20px auto;
}

#thai{ font-size:60px; }
#eng{ font-size:40px; color:#555; }
#memo{ font-size:25px; color:#777; }

#topBar{
  position:absolute;
  top:10px;
  left:0;
  width:100%;
  display:flex;
  padding:0 20px;
  font-size:16px;
  color:#666;
}

#centerTitle{
  position:absolute;
  left:50%;
  transform:translateX(-50%);
}

#rightInfo{
  margin-left:auto;
}

/* ボタン */
#checkBtn{
  position:fixed;
  bottom:20px;
  right:20px;
  width:60px;
  height:60px;
  border:none;
  border-radius:12px;
  background:#222;
  color:#fff;
  font-size:24px;
}

#speakBtn{
  position:fixed;
  bottom:20px;
  left:20px;
  width:60px;
  height:60px;
  border:none;
  border-radius:12px;
  background:#222;
  color:#fff;
  font-size:24px;
}

/* 上のチェック非表示 */
#rightInfo button{
  display:none;
}

/* ===================== ヘッダー土台 ===================== */
#header {
  display: flex;
  align-items: center;
  padding: 0 20px;
  box-sizing: border-box;
}

#searchBox{
  margin-left:auto;
  margin-right:30px;

  width:280px;
  height:34px;

  padding:0 12px;

  font-size:15px;

  border:1px solid #ccc;
  border-radius:6px;

  outline:none;
}

#header-left{
  display:flex;
  align-items:center;
  width:100%;
}

#header-left img {
  height: 40px;
  object-fit: contain;
  display: block;
  margin-right: 10px;
}

#settingBtn{
  margin-left:10px;
  font-size:25px;
  border:none;
  background:none;
  cursor:pointer;
  color:#1976d2;   /* ←これが色 */
  line-height:1;
}

#settingsModal {
  display: none;
}

/* =========================
   一覧テーブル
========================= */

#listTable{
  width:100%;
  border-collapse:collapse;
  table-layout:fixed;
  background:#fff;
}

#listTable thead{
  background:#f3f3f3;
}

#listTable th,
#listTable td{
  padding:10px 12px;
  text-align:left;
  border-bottom:1px solid #ddd;
}

#listTable th{
  font-weight:600;
}

.setting-row{
  display:flex;
  align-items:center;
  gap:15px;
}

.setting-row .label{
  width:60px;
}

</style>
</head>

<body tabindex="0">

<!-- ===================== 上部ヘッダー ===================== -->
<div id="header" style="display:none;">

  <div id="header-left">
    <img id="headerLogo">

    <span id="title"></span>

    <button id="settingBtn">⚙</button>

    <input
      type="text"
      id="searchBox"
      placeholder="🔍 検索">
  </div>

</div>

<div id="card">

  <!-- ==============================
  一覧表示
  ============================== -->
  <div id="list" style="display:none;">

    <table id="listTable">

      <thead>
        <tr>
          <th>ひらがな</th>
          <th>漢字</th>
          <th>タイ語</th>
          <th>英語</th>
        </tr>
      </thead>

      <tbody id="listBody">

      </tbody>

    </table>

  </div>

<script>
let allCards = [];
let cards = [];

let index = 0;
let isFront = true;

let skipVoice = false;
let isFirstLoad = true;
let isReady = false;

const imageCache = {};


// ==============================
// URL lesson取得
// ==============================
function getLesson(){
  const params = new URLSearchParams(window.location.search);
  return params.get("lesson");
}

// ==============================
// 【画像IDチェック】空データ除外
// ==============================
function isValidId(v){
  return v && v.trim() !== "" && v !== "null";
}

// ==============================
// 【画像】キャッシュ取得
// ==============================
async function getImageCached(fileId){

  if(!fileId) return null;

  if(imageCache[fileId]){
    return imageCache[fileId];
  }

  const apiUrl =
    'https://script.google.com/macros/s/AKfycbw5uC6lwJf3LVCLPaCjfUw0DpwvsflywLcS6V2-1HaxcxUNqlf44ovlE7AHFfrEm1g/exec?image='
    + fileId;

  const response = await fetch(apiUrl);

  const imageUrl = (await response.text()).trim();

  imageCache[fileId] = imageUrl;

  return imageUrl;
}

// ==============================
// タイトル更新
// ==============================
function updateTitle() {

  const titleEl = document.getElementById("title");

  if (!cards.length) {
    titleEl.textContent = "Study DB";
    return;
  }

  const card = cards[0];

  titleEl.textContent =
    `Study DB | ${card.title} | ${card.lesson}`;

}

// ==============================
// ヘッダーロゴ
// ==============================
function setHeaderLogo() {

  document.getElementById("headerLogo").src =
    "トータルロゴ.png";

}

// ==============================
// 設定
// ==============================
function openSettings(){

  const settings = getSettings();

  document.querySelector(
    `input[name="sound"][value="${settings.sound}"]`
  ).checked = true;

  document.querySelector(
    `input[name="memo"][value="${settings.memo}"]`
  ).checked = true;

  const orderEl = document.querySelector(
    `input[name="order"][value="${settings.order}"]`
  );

  if(orderEl) orderEl.checked = true;

  document.getElementById("settingModal").style.display = "block";

}

function closeSettings(){

  document.getElementById("settingModal").style.display = "none";

}

function saveSettings(){

  const sound =
    document.querySelector('input[name="sound"]:checked').value;

  const memo =
    document.querySelector('input[name="memo"]:checked').value;

  const order =
    document.querySelector('input[name="order"]:checked').value;

  localStorage.setItem("tangoSettings", JSON.stringify({
    sound: sound,
    memo: memo,
    order: order
  }));

  closeSettings();

}

function getSettings() {

  const settings = Object.assign(
    {
      sound:"on",
      memo:"on",
      order:"normal"
    },
    JSON.parse(localStorage.getItem("tangoSettings")) || {}
  );

  document.querySelectorAll('input[name="sound"]').forEach(r => {
    r.checked = (r.value === settings.sound);
  });

  document.querySelectorAll('input[name="memo"]').forEach(r => {
    r.checked = (r.value === settings.memo);
  });

  document.querySelectorAll('input[name="order"]').forEach(r => {
    r.checked = (r.value === settings.order);
  });

  return settings;

}

document.getElementById("settingBtn").addEventListener("click", (e) => {

  e.stopPropagation();

  openSettings();

});

// =========================
// 初期起動
// =========================
async function initApp() {

  setHeaderLogo();

  // データ取得
  const response = await fetch(
    'https://script.google.com/macros/s/AKfycbw5uC6lwJf3LVCLPaCjfUw0DpwvsflywLcS6V2-1HaxcxUNqlf44ovlE7AHFfrEm1g/exec'
  );

  const data = await response.json();

  allCards = data || [];
  cards = [...allCards];

  const lesson = getLesson();

  if (lesson) {

    cards = cards.filter(
      c => String(c.lessonNo) === String(lesson)
    );

  }

  getSettings();

  const settings = getSettings();

  if(settings.order === "random"){
    cards.sort(() => Math.random() - 0.5);
  }

  // =========================
  // meaning画像だけ先読み
  // =========================
  const promises = [];

  cards.slice(0, 1).forEach(c => {

    if(isValidId(c.meaningImg)){

      promises.push(
        getImageCached(c.meaningImg)
      );

    }

    if(isValidId(c.hiraImg)){

      promises.push(
        getImageCached(c.hiraImg)
      );

    }

  });

  await Promise.all(promises);

  // =========================
  // ローディング
  // =========================
  const header  = document.getElementById("header");

  header.style.display = "flex";

  document.getElementById("list").style.display = "block";

  // 初回表示
  updateTitle();

  showList();

  const searchBox = document.getElementById("searchBox");

  searchBox.addEventListener("input", function(){

    const keyword = this.value.trim().toLowerCase();

    if(keyword === ""){
      showList(cards);
      return;
    }

    console.log(allCards[0]);

    const result = allCards
      .filter(c => {

        return (
          String(c.hira  || "").toLowerCase().includes(keyword) ||
          String(c.kanji || "").toLowerCase().includes(keyword) ||
          String(c.thai  || "").toLowerCase().includes(keyword) ||
          String(c.eng   || "").toLowerCase().includes(keyword)

        );

      })
      .map(c => ({

        ...c,

        displayHira: `【${c.lesson}】 ${c.hira}`

      }));

    showList(result);

  });

  isReady = true;

  document.body.focus();
  window.focus();

  // ==============================
  // 一覧表示
  // ==============================
  function showList(list = cards){

    const listBody = document.getElementById("listBody");

    listBody.innerHTML = "";

    list.forEach(c => {

      listBody.innerHTML += `
        <tr>
          <td>${c.displayHira || c.hira || ""}</td>
          <td>${c.kanji || ""}</td>
          <td>${c.thai || ""}</td>
          <td>${c.eng || ""}</td>
        </tr>
      `;

    });

  }

  // =========================
  // キーボード
  // =========================
  document.addEventListener("keydown", function(e){

    if(!isReady) return;

    if(e.key === "Enter"){

      if(isFront){

        isFront = false;

      }else{

        isFront = true;
        index = (index + 1) % cards.length;

      }

      showCard();

    }

    if(e.key === "Backspace"){

    if (document.activeElement === searchBox) {
      return;
    }

      e.preventDefault();

      if(isFront){

        index = (index - 1 + cards.length) % cards.length;
        isFront = false;

      }else{

        isFront = true;

      }

      showCard();

    }

  });

}

initApp();
</script>
<!-- ==============================
【設定モーダル】
============================== -->
<div id="settingModal" style="display:none; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.5); z-index:3000;">

  <div style="background:#fff; width:90%; max-width:400px; margin:80px auto; padding:20px; border-radius:10px; max-height:80vh; overflow:auto;">

    <h3>設定</h3>



    <!-- 音声 -->
    <div class="setting-row">
      <span class="label">音声：</span>
      <label><input type="radio" name="sound" value="on"> ON</label>
      <label><input type="radio" name="sound" value="off"> OFF</label>
    </div>

    <!-- メモ -->
    <div class="setting-row">
      <span class="label">メモ：</span>
      <label><input type="radio" name="memo" value="on"> ON</label>
      <label><input type="radio" name="memo" value="off"> OFF</label>
    </div>

    <!-- ★ここに追加 -->
    <div class="setting-row">
      <span class="label">順番：</span>
      <label><input type="radio" name="order" value="normal"> 通常</label>
      <label><input type="radio" name="order" value="random"> ランダム</label>
    </div>

    <br>

    <button onclick="saveSettings()">保存</button>
    <button onclick="closeSettings()">閉じる</button>

  </div>
</div>
</body>
</html>

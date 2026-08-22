# 漢字テストメーカー _GASコード

## プロジェクト

漢字テストメーカー

## 情報の種類

その他

## 検索キーワード

漢字テストメーカー、GASコード、generateTestSet、getSavedTestSet、doGet、exportLessonCsv、normalize、createImageIdList、runCreatePDF、createTestPDF、createPDF

## 保存場所

コード完成版/漢字テストメーカー_コード完成版.md

## 内容

// ================================
// 問題生成エンジン
// ================================

function generateTestSet(lesson) {

  Logger.log("lesson = " + lesson);

  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("漢字リスト");
  const data = sheet.getDataRange().getValues();

  const list = [];

  // ==============================
  // 課の漢字取得
  // ==============================
  for (let i = 1; i < data.length; i++) {

    if (Number(data[i][0]) === Number(lesson)) {

      list.push({
        kanji: data[i][2],
        reading: data[i][3],
        id: data[i][4]
      });

    }

  }

  Logger.log("list length = " + list.length);

  // ==============================
  // シャッフル関数
  // ==============================
  function shuffle(array) {

    for (let i = array.length - 1; i > 0; i--) {

      const j = Math.floor(Math.random() * (i + 1));

      const temp = array[i];
      array[i] = array[j];
      array[j] = temp;

    }

    return array;

  }

  // ==============================
  // A問題作成
  // ==============================
  const shuffledA = shuffle([...list]);
  const testA = shuffledA.slice(0, 20);

  // ==============================
  // B問題作成
  // ==============================
  const shuffledB = shuffle([...list]);
  const testB = shuffledB.slice(0, 20);

  // ==============================
  // 発行番号作成
  // 形式：課番号-月/日-001
  // ==============================

  const today = new Date();

  const month = today.getMonth() + 1;
  const day = today.getDate();

  const dateKey =
    Number(lesson) + "-" +
    month + "/" +
    day;

  const properties = PropertiesService.getScriptProperties();

  const countKey = "KANJI_ISSUE_COUNT_" + dateKey;

  let count = Number(properties.getProperty(countKey) || 0);

  count++;

  properties.setProperty(countKey, String(count));

  const issueId =
    dateKey + "-" +
    String(count).padStart(3, "0");


  // ==============================
  // 問題セット
  // ==============================

  const result = {
    issueId: issueId,
    A: testA,
    B: testB
  };

  // ==============================
  // 問題保存（スライドとPDF一致）
  // ==============================
  const cache = CacheService.getScriptCache();
  cache.put("KANJI_TEST_" + lesson, JSON.stringify(result), 3600);

  return result;

}

// ================================
// 保存された問題取得
// ================================

function getSavedTestSet(lesson){

  const cache = CacheService.getScriptCache();
  const data = cache.get("KANJI_TEST_" + lesson);

  if(data){
    return JSON.parse(data);
  }

  return null;

}

// ================================
// Webアプリ入口
// ================================

function doGet(e){

  const lesson = Number(e.parameter.lesson) || 1;

  // ------------------------------
  // GitHub用 キャッシュクリア
  // ------------------------------
  if (e.parameter.mode == "clear") {

    const cache = CacheService.getScriptCache();

    cache.remove("KANJI_TEST_" + lesson);

    return ContentService
      .createTextOutput(
        JSON.stringify({
          success: true,
          lesson: lesson
        })
      )
      .setMimeType(ContentService.MimeType.JSON);

  }

  // ------------------------------
  // GitHub用 JSON
  // ------------------------------
  if (e.parameter.mode == "json") {

    const data = getSavedTestSet(lesson);

    return ContentService
      .createTextOutput(JSON.stringify(data))
      .setMimeType(ContentService.MimeType.JSON);

  }

    // ------------------------------
  // GIT用 PNGプレビュー
  // ------------------------------
  if (e.parameter.mode == "png") {

    const data = getSavedTestSet(lesson);

    if (!data) {
      return ContentService
        .createTextOutput("問題データがありません")
        .setMimeType(ContentService.MimeType.TEXT);
    }

    const type = e.parameter.type;
    const isAnswer = e.parameter.answer === "true";

    const list =
      type === "A" ? data.A :
      type === "B" ? data.B :
      null;

    if (!list) {
      return ContentService
        .createTextOutput("画像タイプが不正です")
        .setMimeType(ContentService.MimeType.TEXT);
    }

        let html =
        "<html><head><meta charset='UTF-8'>" +
        "<style>" +
        "body{margin:20px;font-family:Arial,sans-serif;}" +
        ".title{font-size:24px;font-weight:bold;margin-bottom:20px;}" +
        ".grid{display:grid;grid-template-columns:repeat(4,1fr);gap:15px;}" +
        ".card{border:1px solid #ccc;padding:10px;text-align:center;}" +
        ".card img{width:100%;height:auto;}" +
        "</style></head><body>" +

        "<div class='title'>" +
        "画像" + type +
        (isAnswer ? "　答えプレビュー" : "　問題プレビュー") +
        "</div>" +

        "<div class='grid'>";

      list.forEach(function(item, index){

        const imageUrl =
          "https://drive.google.com/thumbnail?id=" +
          item.id +
          "&sz=w1000";

        html +=
          "<div class='card'>" +
          "<div>" + (index + 1) + "</div>" +
          "<img src='" + imageUrl + "'>" +
          "</div>";

      });

      html +=
        "</div>" +
        "</body></html>";

      return HtmlService
        .createHtmlOutput(html)
        .setTitle(
          "画像" + type +
          (isAnswer ? " 答えプレビュー" : " 問題プレビュー")
        );
  }

  // ------------------------------
  // 従来どおり画面表示
  // ------------------------------
  const template = HtmlService.createTemplateFromFile("dual");

  template.lesson = lesson;

  return template.evaluate();

}


//-------------------------------------
// 課CSVダウンロード
//-------------------------------------
function exportLessonCsv() {

  const ui = SpreadsheetApp.getUi();
  const response = ui.prompt("CSVを出力する課番号（数字）");

  if (response.getSelectedButton() != ui.Button.OK) return;

  const lesson = normalize(response.getResponseText());

  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const source = ss.getSheetByName("漢字リスト");
  const data = source.getDataRange().getValues();

  const rows = [];

  for (let i = 1; i < data.length; i++) {
    if (normalize(data[i][0]) == lesson) {

      rows.push([
        data[i][0],
        data[i][1],
        data[i][2],
        data[i][3]
      ]);
    }
  }

  if (rows.length == 0) {
    ui.alert("データがありません");
    return;
  }

  let csv = "課,No,漢字,読み\n";

  rows.forEach(r => {
    csv += r.join(",") + "\n";
  });

  const html = HtmlService.createHtmlOutput(
   '<a download="test_cards.csv" href="data:text/csv;charset=utf-8,' + encodeURIComponent(csv) + '">CSVダウンロード</a>'
  );

  SpreadsheetApp.getUi().showModalDialog(html, "CSV作成");
}


//-------------------------------------
// 空白・前後スペース除去
//-------------------------------------
function normalize(text) {
  if (!text) return "";
  return text.toString().replace(/\s+/g, "").trim();
}

// ==============================
// 指定した課のTestCards画像IDリスト作成
// ==============================

function createImageIdList() {

  // ------------------------------
  // URL発行シートから課番号取得
  // ------------------------------

  const controlSheet =
    SpreadsheetApp.getActiveSpreadsheet()
      .getSheetByName("URL発行");

  const lesson =
    Number(controlSheet.getRange("B2").getValue());

  if (!lesson) {
    SpreadsheetApp.getUi().alert("課番号を指定してください");
    return;
  }

  // ------------------------------
  // 漢字リスト取得
  // ------------------------------

  const sheet =
    SpreadsheetApp.getActiveSpreadsheet()
      .getSheetByName("漢字リスト");

  const data =
    sheet.getDataRange().getValues();

  // ------------------------------
  // TestCardsフォルダ
  // ------------------------------

  const TESTCARDS_FOLDER_ID =
    "1wKxkySp2_z5cdyL6V6zjliYkIbYOv9go";

  const root =
    DriveApp.getFolderById(TESTCARDS_FOLDER_ID);

  const lessonFolders =
    root.getFolders();

  const imageMap = {};

  // ------------------------------
  // PNG画像ID取得
  // ------------------------------

  while (lessonFolders.hasNext()) {

    const lessonFolder =
      lessonFolders.next();

    const files =
      lessonFolder.getFiles();

    while (files.hasNext()) {

      const file =
        files.next();

      if (
        file.getName()
          .toLowerCase()
          .endsWith(".png")
      ) {

        const name =
          file.getName()
            .replace(/\.png$/i, "");

        const id =
          file.getId();

        imageMap[name] = id;

      }

    }

  }

  // ------------------------------
  // 指定した課だけ画像IDを設定
  // ------------------------------

  for (let i = 1; i < data.length; i++) {

    if (Number(data[i][0]) !== lesson) {
      continue;
    }

    const kanji =
      data[i][2];

    if (imageMap[kanji]) {

      sheet
        .getRange(i + 1, 5)
        .setValue(imageMap[kanji]);

    }

  }

  SpreadsheetApp.getUi().alert(
    lesson + "課のPNG画像処理が完了しました"
  );

}

// ================================
// 指定した課のキャッシュクリア
// ================================

function clearLessonCacheFromSheet() {

  const sheet =
    SpreadsheetApp.getActiveSpreadsheet()
      .getSheetByName("URL発行");

  const lesson =
    Number(sheet.getRange("B2").getValue());

  if (!lesson) {
    SpreadsheetApp.getUi().alert("課番号を指定してください");
    return;
  }

  const cache =
    CacheService.getScriptCache();

  cache.remove("KANJI_TEST_" + lesson);

  SpreadsheetApp.getUi().alert(
    lesson + "課のキャッシュをクリアしました"
  );

}

// ================================
// 指定した課の問題を正式発行
// ================================

function issueTestSetFromSheet() {

  const sheet =
    SpreadsheetApp.getActiveSpreadsheet()
      .getSheetByName("URL発行");

  const lesson =
    Number(sheet.getRange("B2").getValue());

  if (!lesson) {
    SpreadsheetApp.getUi().alert("課番号を指定してください");
    return;
  }

  const data = generateTestSet(lesson);

  SpreadsheetApp.getUi().alert(
    lesson + "課の問題を発行しました\n\n発行番号：" + data.issueId
  );

}

// ==============================
// GITを開く
// ==============================

function openGitUrlFromSheet() {

  const sheet =
    SpreadsheetApp.getActiveSpreadsheet()
      .getSheetByName("URL発行");

  const url =
    sheet.getRange("B3").getValue();

  if (!url) {
    SpreadsheetApp.getUi().alert("GitHub URLがありません");
    return;
  }

    const html = HtmlService.createHtmlOutput(
    '<a href="' + url + '" target="_blank" ' +
    'onclick="google.script.host.close()" ' +
    'style="font-size:20px;">GITを開く</a>'
  );

  SpreadsheetApp.getUi().showModalDialog(
    html,
    "GITを開く"
  );
}

### 現在の構成

漢字テストメーカーの問題生成・保存・Webアプリ表示・画像ID管理・GIT連携を行うGASコード。

PDF生成機能は削除済み。現在はPDFを使用しない。

### generateTestSet(lesson)

「漢字リスト」シートから指定した課のデータを取得し、A問題とB問題をそれぞれシャッフルして最大20問ずつ作成する。

取得するデータはC列の漢字、D列の読み、E列の画像ID。

A・B問題と発行番号を問題セットとして作成し、Script Cacheに保存する。

キャッシュ保持時間は3600秒（1時間）。

### getSavedTestSet(lesson)

Script Cacheに保存された問題セットを取得する。

保存されていない場合はnullを返す。

### doGet(e)

lessonパラメータから課番号を取得する。

mode=clearの場合は、指定課の問題キャッシュを削除する。

mode=jsonの場合は、保存された問題セットをJSON形式で返す。

mode=pngの場合は、保存されたA/B問題セットを使用して画像プレビューを生成する。

それ以外の場合はHTMLファイル「dual」をテンプレートとして表示する。

### exportLessonCsv()

入力した課番号をもとに「漢字リスト」シートから該当データを抽出し、「課,No,漢字,読み」のCSVを作成する。

### normalize(text)

文字列から空白文字を除去し、前後の空白を取り除く。

### createImageIdList()

Google DriveのTestCardsフォルダ内にあるPNG画像を取得し、画像ファイル名と「漢字リスト」シートC列の漢字を照合する。

一致した画像のGoogle DriveファイルIDをE列へ設定する。

### clearLessonCacheFromSheet()

「URL発行」シートのB2から課番号を取得し、その課の問題キャッシュを削除する。

### issueTestSetFromSheet()

「URL発行」シートのB2から課番号を取得し、generateTestSet(lesson)を実行して問題を正式発行する。

新しいA/B問題セットと発行番号を作成して保存し、発行番号を表示する。

### openGitUrlFromSheet()

「URL発行」シートのB3に設定されたGIT URLを取得し、ポップアップからGITを開く。

「GITを開く」をクリックするとGITを新しいタブで開き、ポップアップを閉じる。

### 現在の運用

PDFを使用せず、正式発行した問題セットを基準にGIT側でテスト・問題プレビュー・答えプレビューを表示する。

正式発行したA/B問題セットと発行番号をScript Cacheに保存し、GIT側は保存された問題セットを取得して使用する。

画像問題とテスト問題は、同じA/B問題セットに含まれる画像IDを使用して表示する。

GIT側ではA/B問題・答えのプレビューをA4サイズで表示し、PNG保存できる。

### コード内で参照されているスプレッドシートのシート名

「漢字リスト」
「URL発行」

### コード内で参照されているHTMLファイル

「dual」

### コード内で使用されているGoogle DriveフォルダID

「1wKxkySp2_z5cdyL6V6zjliYkIbYOv9go」

上記以外のスプレッドシート構成、HTMLファイルの内容、Google Driveフォルダ名、GitHubリポジトリ構成については、このコードだけでは確認できない。

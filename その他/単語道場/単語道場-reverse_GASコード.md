# 単語道場/単語道場-reverse_GASコード

## プロジェクト

単語道場/単語道場-reverse_2026.8.23

## 情報の種類

その他

## 検索キーワード

単語道場、tango-dojo、単語道場-reverse、tango-dojo-reverse、GASコード

## 保存場所

GitHub／GAS

## 内容

## 対応アプリ

- 単語道場（日本語 → タイ語）
- 単語道場-Reverse（タイ語 → 日本語）

## 共通データ構成

「単語道場」と「単語道場-Reverse」は、同じGoogleスプレッドシートの「単語リスト」データを共有して使用する。

両アプリは別々の独立したアプリケーションとして動作する。

データは共通であり、通常版・Reverse版それぞれで同じ単語データを使用する。

## Google Apps Script

【Google Apps Script】
// ==============================
// 意味画像ID 自動入力（安定版）
// ==============================
function setMeaningImageIds() {

  const FOLDER_ID = "1WNbayD-_lNSVlSfUZ5fzi9VX7I8PzQok"; // ←ここだけ変える

  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("単語リスト");
  const data = sheet.getDataRange().getValues();
  const headers = data[0];

  const colNo  = headers.indexOf("No");
  const colImg = headers.indexOf("意味画像");

  const folder = DriveApp.getFolderById(FOLDER_ID);
  const files = folder.getFiles();

  let fileMap = {};

  while (files.hasNext()) {
    const file = files.next();
    const name = file.getName(); // 001.png

    const match = name.match(/^(\d+)/);
    if (match) {
      const num = parseInt(match[1], 10);
      fileMap[num] = file.getId();
    }
  }

  for (let i = 1; i < data.length; i++) {

    const rowNo = data[i][colNo];

    if (fileMap[rowNo]) {
      sheet.getRange(i + 1, colImg + 1).setValue(fileMap[rowNo]);
    }
  }

  SpreadsheetApp.getUi().alert("意味画像ID 入力完了");
}

// ==============================
// 日本語画像ID 自動入力（課対応版）
// ==============================
function setJapaneseImageIds() {

  const FOLDER_ID = "1y0PiTsUTbOfSX5lbutf0SELElIKxghrr";

  const sheet = SpreadsheetApp
    .getActiveSpreadsheet()
    .getSheetByName("単語リスト");

  const data = sheet.getDataRange().getValues();
  const headers = data[0];

  const colLesson = headers.indexOf("課No");
  const colNo     = headers.indexOf("No");
  const colImg    = headers.indexOf("日本語画像");

  const folder = DriveApp.getFolderById(FOLDER_ID);
  const files = folder.getFiles();

  let fileMap = {};

  // ==========================
  // ファイル名 → ID
  // 例: 01_001.png
  // ==========================
  while (files.hasNext()) {

    const file = files.next();
    const name = file.getName();

    const match = name.match(/^(\d+)_(\d+)\.png$/i);

    if (match) {

      const lessonNo = String(parseInt(match[1], 10)).padStart(2, "0");
      const cardNo   = String(parseInt(match[2], 10)).padStart(3, "0");

      const key = lessonNo + "_" + cardNo;

      fileMap[key] = file.getId();
    }
  }

  // ==========================
  // シートへ書き込み
  // ==========================
  for (let i = 1; i < data.length; i++) {

    const lessonNo =
      String(data[i][colLesson]).padStart(2, "0");

    const cardNo =
      String(data[i][colNo]).padStart(3, "0");

    const key = lessonNo + "_" + cardNo;

    if (fileMap[key]) {

      sheet
        .getRange(i + 1, colImg + 1)
        .setValue(fileMap[key]);
    }
  }

  SpreadsheetApp.getUi().alert("日本語画像ID 入力完了");
}
// ==============================
// 課を入力してシート作成（1コード）
// ==============================
function splitLesson() {

  const ui = SpreadsheetApp.getUi();

  // 入力ダイアログ（シート標準）
  const res = ui.prompt("課名を入力", "例：第1課", ui.ButtonSet.OK_CANCEL);

  if (res.getSelectedButton() !== ui.Button.OK) return;

  const targetLesson = res.getResponseText().trim();
  if (!targetLesson) return;

  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const src = ss.getSheetByName("単語リスト");
  const data = src.getDataRange().getValues();

  const headers = data[0];
  const colLesson = headers.indexOf("課名");

  let sheet = ss.getSheetByName(targetLesson);

  if (!sheet) {
    sheet = ss.insertSheet(targetLesson);
  } else {
    sheet.clear();
  }

  sheet.appendRow(headers);

  for (let i = 1; i < data.length; i++) {
    if (data[i][colLesson] === targetLesson) {
      sheet.appendRow(data[i]);
    }
  }

  ui.alert(targetLesson + " 作成完了");
}

// ==============================
// 入口（JSON API版）
// ==============================
function doGet(e) {

  // ==========================
  // 画像返却
  // ==========================
  if (e.parameter.image) {

    const fileId = e.parameter.image;

    const url =
      "https://lh3.googleusercontent.com/d/"
      + fileId;

    return ContentService
      .createTextOutput(url)
      .setMimeType(ContentService.MimeType.TEXT);

  }

  // ==========================
  // カードデータ返却
  // ==========================
  const sheet = SpreadsheetApp
    .getActiveSpreadsheet()
    .getSheetByName("単語リスト");

  const data = sheet.getDataRange().getValues();

  const result = [];

  for (let i = 1; i < data.length; i++) {

    const row = data[i];

    result.push({

      title: row[0],
      lessonNo: row[1],
      lesson: row[2],

      hira: row[4],
      kanji: row[5],
      hiraImg: row[8],
      meaningImg: row[9],

      thai: row[6],
      eng: row[7],
      memo: row[10]

    });

  }

  return ContentService
    .createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);

}
// ==============================
// 画像取得
// ==============================
function getImage(fileId) {
  const blob = DriveApp.getFileById(fileId).getBlob();
  return Utilities.base64Encode(blob.getBytes());
}

// ==============================
// 【GAS】カード取得（修正版 v2）
// ==============================
function getAllCards() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("単語リスト");
  const data = sheet.getDataRange().getValues();

  const result = [];

  for (let i = 1; i < data.length; i++) {
    const row = data[i];

    result.push({
      title: row[0],      // ★A列：教材
      lesson: row[2],     // ★C列：課

      hira: row[4],        // E列
      hiraImg: row[8],     // I列
      meaningImg: row[9],  // J列
      thai: row[6],
      eng: row[7],
      memo: row[10]
    });
  }

  return result;
}

## 主な処理

・意味画像ID自動入力
「単語リスト」シートのNoとGoogle Drive内の画像ファイル名を照合し、意味画像のファイルIDを自動入力する。

・日本語画像ID自動入力
「課No」と「No」を使用して、01_001.png形式のファイル名と照合し、日本語画像のファイルIDを自動入力する。

・課別シート作成
「単語リスト」から指定した課名のデータだけを抽出し、課ごとのシートを自動作成する。

・doGet
Webアプリからのリクエストを受け付ける。
imageパラメータがある場合はGoogle Drive画像の表示URLを返す。
通常のアクセスでは「単語リスト」から単語カードデータをJSON形式で返す。

・getImage
Google DriveのファイルIDから画像を取得し、Base64形式に変換する。

・getAllCards
「単語リスト」から単語カードデータを取得して配列として返す。

## スプレッドシート連携

対象シート：

「単語リスト」

「単語道場」と「単語道場-Reverse」の両方が同じスプレッドシートを使用する。

Webアプリへ返す主なデータ：

教材、課No、課、ひらがな、漢字、タイ語、英語、日本語画像ID、意味画像ID、メモ。

## Google Drive連携

意味画像と日本語画像をGoogle Driveで管理し、ファイル名から対象カードを特定してファイルIDを取得する。

意味画像は通常版・Reverse版で共有する。

日本語画像も通常版・Reverse版で共有する。

画像IDを含む単語データは共通の「単語リスト」から取得する。

## 運用方針

単語データの追加・編集は「単語道場」側の共通データを基準として行う。

「単語道場-Reverse」は同じ単語データを使用し、アプリケーション側で表示方向・音声方向を反転して使用する。

両アプリは独立しているため、一方のアプリケーションコードを変更しても、もう一方のアプリケーションには直接影響しない。

## URL連携

URL台帳では、

- B列：単語道場（日本語 → タイ語）
- C列：単語道場-Reverse（タイ語 → 日本語）
- D列：Study DB

として管理する。

単語道場-ReverseのURL形式：

https://kira-nihongo-online.github.io/tango-dojo-reverse/?mode=thjp&lesson=課No


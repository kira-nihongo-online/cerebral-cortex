# 一文字道場_GASコード

## プロジェクト

一文字道場

## 情報の種類

その他

## 検索キーワード

一文字道場 GAS Google Apps Script スプレッドシート 一文字リスト Google Drive 画像 JSON API

## 保存場所

一文字道場／その他

## 内容

Google Apps Script（GAS）を使用して「一文字リスト」のデータを管理し、Google Driveの画像と連携してWebアプリへJSON形式でデータを提供する。

// ==============================
// 意味画像ID 自動入力
// ==============================
function setMeaningImageIds() {

  const FOLDER_ID = "1aBYh6AFHsflJX8ZXm1oqPdJNjbIQG22M";

  const sheet = SpreadsheetApp
    .getActiveSpreadsheet()
    .getSheetByName("一文字リスト");

  const data = sheet.getDataRange().getValues();
  const headers = data[0];

  const colNo  = headers.indexOf("No");
  const colImg = headers.indexOf("意味画像");

  const folder = DriveApp.getFolderById(FOLDER_ID);
  const files = folder.getFiles();

  let fileMap = {};

  while (files.hasNext()) {

    const file = files.next();
    const name = file.getName();

    const match = name.match(/^(\d+)/);

    if (match) {

      const num = parseInt(match[1], 10);
      fileMap[num] = file.getId();

    }
  }

  for (let i = 1; i < data.length; i++) {

    const rowNo = data[i][colNo];

    if (fileMap[rowNo]) {

      sheet
        .getRange(i + 1, colImg + 1)
        .setValue(fileMap[rowNo]);

    }
  }

  SpreadsheetApp.getUi().alert("意味画像ID 入力完了");
}

// ==============================
// 日本語画像ID 自動入力
// ==============================
function setJapaneseImageIds() {

  const FOLDER_ID = "1Y4HyY21Vd0XjOrlraK4iW8n7SRR5U0Of";

  const sheet = SpreadsheetApp
    .getActiveSpreadsheet()
    .getSheetByName("一文字リスト");

  const data = sheet.getDataRange().getValues();
  const headers = data[0];

  const colLesson = headers.indexOf("課No");
  const colNo     = headers.indexOf("No");
  const colImg    = headers.indexOf("日本語画像");

  const folder = DriveApp.getFolderById(FOLDER_ID);
  const files = folder.getFiles();

  let fileMap = {};

  while (files.hasNext()) {

    const file = files.next();
    const name = file.getName();

    const match = name.match(/^(\d+)_(\d+)\.png$/i);

    if (match) {

      const lessonNo =
        String(parseInt(match[1], 10)).padStart(2, "0");

      const cardNo =
        String(parseInt(match[2], 10)).padStart(3, "0");

      const key = lessonNo + "_" + cardNo;

      fileMap[key] = file.getId();

    }
  }

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
// 課を入力してシート作成
// ==============================
function splitLesson() {

  const ui = SpreadsheetApp.getUi();

  const res =
    ui.prompt(
      "課名を入力",
      "例：第1課",
      ui.ButtonSet.OK_CANCEL
    );

  if (res.getSelectedButton() !== ui.Button.OK) return;

  const targetLesson =
    res.getResponseText().trim();

  if (!targetLesson) return;

  const ss = SpreadsheetApp.getActiveSpreadsheet();

  const src =
    ss.getSheetByName("一文字リスト");

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

  if (e.parameter.image) {

    const fileId = e.parameter.image;

    const url =
      "https://lh3.googleusercontent.com/d/"
      + fileId;

    return ContentService
      .createTextOutput(url)
      .setMimeType(ContentService.MimeType.TEXT);

  }

  const sheet = SpreadsheetApp
    .getActiveSpreadsheet()
    .getSheetByName("一文字リスト");

  const data = sheet.getDataRange().getValues();

  const result = [];

  for (let i = 1; i < data.length; i++) {

    const row = data[i];

    result.push({

      title: row[0],
      lessonNo: row[1],
      lesson: row[2],

      hira: row[4],
      roman: row[5],

      hiraImg: row[6],
      meaningImg: row[7]


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

  const blob =
    DriveApp.getFileById(fileId).getBlob();

  return Utilities.base64Encode(
    blob.getBytes()
  );

}

// ==============================
// カード取得
// ==============================
function getAllCards() {

  const sheet = SpreadsheetApp
    .getActiveSpreadsheet()
    .getSheetByName("一文字リスト");

  const data = sheet.getDataRange().getValues();

  const result = [];

  for (let i = 1; i < data.length; i++) {

    const row = data[i];

    result.push({

      title: row[0],
      lessonNo: row[1],
      lesson: row[2],

      hira: row[4],
      roman: row[5],

      hiraImg: row[6],
      meaningImg: row[7]

    });

  }

  return result;

}

【対象スプレッドシート】

使用するシート：

・一文字リスト

「一文字リスト」から以下のデータを取得する。

・教材
・課No
・課名
・No
・ひらがな
・ローマ字
・日本語画像ID
・意味画像ID

【意味画像ID自動入力】

Google Driveの指定フォルダにある画像ファイルを検索し、ファイル名の先頭にある番号と「一文字リスト」のNoを照合する。

一致した場合、そのファイルIDを「意味画像」列へ自動入力する。

【日本語画像ID自動入力】

Google Driveの指定フォルダにある画像ファイルを検索し、「課No」と「No」を使用して画像ファイル名と照合する。

画像ファイル名は以下の形式を想定する。

01_001.png

一致したファイルのIDを「日本語画像」列へ自動入力する。

【課別シート作成】

「一文字リスト」から指定した課名のデータだけを抽出し、課名と同じ名前のシートを作成する。

既に同名シートが存在する場合は内容をクリアしてからデータを再作成する。

【JSON API】

doGet()でWebアプリからのリクエストを受け付ける。

imageパラメータが指定された場合は、指定されたGoogle DriveファイルIDから画像URLを生成して返す。

通常のアクセスでは「一文字リスト」からカードデータを取得し、JSON形式で返す。

返却するカードデータ：

・title
・lessonNo
・lesson
・hira
・roman
・hiraImg
・meaningImg

【画像取得】

getImage(fileId)でGoogle Driveの指定ファイルを取得し、画像データをBase64形式へ変換する。

【カード取得】

getAllCards()で「一文字リスト」の全カードデータを取得し、配列として返す。

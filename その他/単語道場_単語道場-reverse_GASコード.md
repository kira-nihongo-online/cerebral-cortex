# 単語道場_単語道場-reverse_GASコード

## プロジェクト

単語道場_2026.8.24

## 情報の種類

その他

## 検索キーワード

単語道場、単語道場-reverse、tango-dojo、tango-dojo-reverse、GASコード

## 保存場所

GitHub／GAS

## 内容

## 対応アプリ

- 単語道場（日本語 → タイ語）
- 単語道場-Reverse（タイ語 → 日本語）
- Study DB

## 共通データ構成

「単語道場」と「単語道場-Reverse」は、同じGoogleスプレッドシートの「単語リスト」データを共有して使用する。

両アプリは別々の独立したアプリケーションとして動作する。

Study DBも独立したアプリケーションとして動作し、単語道場・単語道場-Reverseの機能を内部に組み込まない。

Study DBはURL台帳を利用して、学習方向に応じて単語道場または単語道場-ReverseへURLジャンプする。

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
// カードデータ＋URL台帳返却
// ==========================

const ss = SpreadsheetApp.getActiveSpreadsheet();

const sheet = ss.getSheetByName("単語リスト");
const urlSheet = ss.getSheetByName("URL台帳");

const data = sheet.getDataRange().getValues();
const urlData = urlSheet.getDataRange().getValues();

// ==========================
// URL台帳を課Noで検索できる形にする
// A列：課No
// B列：JP→TH URL
// C列：TH→JP URL
// D列：Study DB URL
// ==========================

const urlMap = {};

for (let i = 1; i < urlData.length; i++) {

  const row = urlData[i];

  const lessonNo = String(row[0]).trim();

  if (!lessonNo) continue;

  urlMap[lessonNo] = {

    jpThUrl: row[1] || "",
    thJpUrl: row[2] || "",
    studyDbUrl: row[3] || ""

  };

}

// ==========================
// カードデータ
// ==========================

const result = [];

for (let i = 1; i < data.length; i++) {

  const row = data[i];

  const lessonNo = String(row[1]).trim();

  const urls = urlMap[lessonNo] || {};

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
    memo: row[10],

    // URL台帳
    jpThUrl: urls.jpThUrl || "",
    thJpUrl: urls.thJpUrl || "",
    studyDbUrl: urls.studyDbUrl || ""

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



主な処理
・意味画像ID自動入力

「単語リスト」シートのNoとGoogle Drive内の画像ファイル名を照合し、意味画像のファイルIDを自動入力する。

・日本語画像ID自動入力

「課No」と「No」を使用して、01_001.png形式のファイル名と照合し、日本語画像のファイルIDを自動入力する。

・課別シート作成

「単語リスト」から指定した課名のデータだけを抽出し、課ごとのシートを自動作成する。

・doGet

Webアプリからのリクエストを受け付ける。

image パラメータがある場合は、Google Drive画像の表示URLを返す。

通常のアクセスでは、

「単語リスト」の単語カードデータ
「URL台帳」の学習URL

を取得し、課Noに対応するURLを単語カードデータに付加してJSON形式で返す。

・URL台帳連携

「URL台帳」のA列の課Noを基準としてURLを対応させる。

B列：日本語 → タイ語
C列：タイ語 → 日本語
D列：Study DB

単語リストの課NoとURL台帳の課Noを照合し、各カードに以下のURLを付加する。

jpThUrl
thJpUrl
studyDbUrl

これにより、Study DBは現在の課Noに対応する学習URLを取得できる。

・getImage

Google DriveのファイルIDから画像を取得し、Base64形式に変換する。

・getAllCards

「単語リスト」から単語カードデータを取得して配列として返す。

スプレッドシート連携
対象シート

「単語リスト」

「URL台帳」

単語リスト

「単語道場」と「単語道場-Reverse」の両方が同じスプレッドシートの「単語リスト」を使用する。

Webアプリへ返す主なデータ：

教材
課No
課
ひらがな
漢字
タイ語
英語
日本語画像ID
意味画像ID
メモ
URL台帳

URL台帳は以下の構成で管理する。

A列：課No
B列：単語道場（日本語 → タイ語）
C列：単語道場-Reverse（タイ語 → 日本語）
D列：Study DB
E列：教材
F列：課名

課Noをキーとして、単語データとURLを対応させる。

Google Drive連携

意味画像と日本語画像をGoogle Driveで管理し、ファイル名から対象カードを特定してファイルIDを取得する。

意味画像は通常版・Reverse版で共有する。

日本語画像も通常版・Reverse版で共有する。

画像IDを含む単語データは共通の「単語リスト」から取得する。

アプリケーション構成

以下はすべて独立したアプリケーションとして維持する。

単語道場

学習方向：

日本語 → タイ語

単語道場-Reverse

学習方向：

タイ語 → 日本語

Study DB

Study DBから各課の学習方向を選択し、URL台帳の対応URLへジャンプする。

Study DBに単語道場の学習機能を組み込まない。

Study DBはURLによって単語道場または単語道場-Reverseを呼び出す。

URL連携

URL台帳では、

B列：単語道場（日本語 → タイ語）
C列：単語道場-Reverse（タイ語 → 日本語）
D列：Study DB

として管理する。

Study DBで lesson パラメータにより課Noを判定し、URL台帳の同じ課Noの行からURLを取得する。

日本語 → タイ語

Study DBの「日本語 → タイ語」を選択すると、URL台帳B列の該当課URLへジャンプする。

タイ語 → 日本語

Study DBの「タイ語 → 日本語」を選択すると、URL台帳C列の該当課URLへジャンプする。

Reverse URL形式
https://kira-nihongo-online.github.io/tango-dojo-reverse/?mode=thjp&lesson=課No
運用方針

単語データの追加・編集は「単語道場」側の共通データを基準として行う。

「単語道場-Reverse」は同じ単語データを使用し、アプリケーション側で表示方向・音声方向を反転して使用する。

Study DBは共通の単語データとURL台帳を利用するが、単語道場・Reverseとは独立したアプリケーションとして維持する。

URL台帳を変更することで、各課からジャンプする先のURLを管理できる。

両アプリは独立しているため、一方のアプリケーションコードを変更しても、もう一方のアプリケーションには直接影響しない。

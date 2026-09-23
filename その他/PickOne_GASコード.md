# PickOne_GASコード

## プロジェクト

PickOne_2026.9.23

## 情報の種類

その他

## 検索キーワード

PickOne、GASコード

## 保存場所

GitHub／GAS

## 内容

PickOneと連携するGoogle Apps Script。

複数の出席管理スプレッドシートから「当日出欠」シートの出席者を取得し、「クラス選択」シートのA〜F列へクラス別に展開する。

Googleフォームから送信されたCustomの名前は、「クラス選択」シートのG列へ追加する。

「クラス選択」シートのA12:Gにある名前をまとめて取得し、`names` 配列のJSONとしてWebアプリへ返す。PickOneはこのJSONを取得して抽選対象の名前リストとして使用する。

また、スプレッドシートからPickOne本体とCustomフォームを開くためのメニュー用ダイアログを提供する。

現在使用しているGASコードとスプレッドシート構成を、PickOne連携の基準として扱う。

## Google Apps Script

【Google Apps Script】

function getEP42Data() {
  const SOURCE_ID = '1r5dTJ-z8kDt9VuX6kf8B3-3LuJQcgJu_UaBLasuP8zA';
  const SOURCE_SHEET = '当日出欠';

  const targetSS = SpreadsheetApp.getActiveSpreadsheet();
  const targetSheet = targetSS.getSheetByName('クラス選択');

  const sourceSS = SpreadsheetApp.openById(SOURCE_ID);
  const sourceSheet = sourceSS.getSheetByName(SOURCE_SHEET);

  const data = sourceSheet.getDataRange().getValues();

  // EP4/2の抽出欄をクリア
  targetSheet.getRange('A12:A').clearContent();

  // E列に「出席」を含む行だけ抽出し、C列の名前を取得
  const students = data.slice(1)
    .filter(row => String(row[4]).includes('出席'))
    .map(row => [row[2]]);

  // A12から名前だけ表示
  if (students.length > 0) {
    targetSheet
      .getRange(12, 1, students.length, 1)
      .setValues(students);
  }
}

function getEP52Data() {
  const SOURCE_ID = '1MJlRKgJj6Dm7RQfHKOPJz4W03GYfG3B8ztPXqNy8ggw';
  const SOURCE_SHEET = '当日出欠';

  const targetSS = SpreadsheetApp.getActiveSpreadsheet();
  const targetSheet = targetSS.getSheetByName('クラス選択');

  const sourceSS = SpreadsheetApp.openById(SOURCE_ID);
  const sourceSheet = sourceSS.getSheetByName(SOURCE_SHEET);

  const data = sourceSheet.getDataRange().getValues();

  // EP5/2の抽出欄をクリア
  targetSheet.getRange('B12:B').clearContent();

  // E列に「出席」を含む行だけ抽出し、C列の名前を取得
  const students = data.slice(1)
    .filter(row => String(row[4]).includes('出席'))
    .map(row => [row[2]]);

  // B12から名前だけ表示
  if (students.length > 0) {
    targetSheet
      .getRange(12, 2, students.length, 1)
      .setValues(students);
  }
}

function getEP62Data() {
  const SOURCE_ID = '1U0sLq_wTAu5T-11WSJEVlvvjHly8yOrMV4CaRuqhQoc';
  const SOURCE_SHEET = '当日出欠';

  const targetSS = SpreadsheetApp.getActiveSpreadsheet();
  const targetSheet = targetSS.getSheetByName('クラス選択');

  const sourceSS = SpreadsheetApp.openById(SOURCE_ID);
  const sourceSheet = sourceSS.getSheetByName(SOURCE_SHEET);

  const data = sourceSheet.getDataRange().getValues();

  // EP6/2の抽出欄をクリア
  targetSheet.getRange('C12:C').clearContent();

  // E列に「出席」を含む行だけ抽出し、C列の名前を取得
  const students = data.slice(1)
    .filter(row => String(row[4]).includes('出席'))
    .map(row => [row[2]]);

  // C12から名前だけ表示
  if (students.length > 0) {
    targetSheet
      .getRange(12, 3, students.length, 1)
      .setValues(students);
  }
}

function get413Data() {
  const SOURCE_ID = '1zlwG3G9EsLSmlj-YPXvxuLWBv_nLIbBKWoUdlwzet58';
  const SOURCE_SHEET = '当日出欠';

  const targetSS = SpreadsheetApp.getActiveSpreadsheet();
  const targetSheet = targetSS.getSheetByName('クラス選択');

  const sourceSS = SpreadsheetApp.openById(SOURCE_ID);
  const sourceSheet = sourceSS.getSheetByName(SOURCE_SHEET);

  const data = sourceSheet.getDataRange().getValues();

  // 4/13の抽出欄をクリア
  targetSheet.getRange('D12:D').clearContent();

  // E列に「出席」を含む行だけ抽出し、C列の名前を取得
  const students = data.slice(1)
    .filter(row => String(row[4]).includes('出席'))
    .map(row => [row[2]]);

  // D12から名前だけ表示
  if (students.length > 0) {
    targetSheet
      .getRange(12, 4, students.length, 1)
      .setValues(students);
  }
}

function get514Data() {
  const SOURCE_ID = '1yPmZWYNPSBQhd5V1YVEcWtRJKGErMnGRTPsHTK0NbtM';
  const SOURCE_SHEET = '当日出欠';

  const targetSS = SpreadsheetApp.getActiveSpreadsheet();
  const targetSheet = targetSS.getSheetByName('クラス選択');

  const sourceSS = SpreadsheetApp.openById(SOURCE_ID);
  const sourceSheet = sourceSS.getSheetByName(SOURCE_SHEET);

  const data = sourceSheet.getDataRange().getValues();

  // 5/14の抽出欄をクリア
  targetSheet.getRange('E12:E').clearContent();

  // E列に「出席」を含む行だけ抽出し、C列の名前を取得
  const students = data.slice(1)
    .filter(row => String(row[4]).includes('出席'))
    .map(row => [row[2]]);

  // E12から名前だけ表示
  if (students.length > 0) {
    targetSheet
      .getRange(12, 5, students.length, 1)
      .setValues(students);
  }
}

function get614Data() {
  const SOURCE_ID = '1dY08lV0Z8QlFIYDkmFRST7UC46tAxrDg9TeGgZa2DXg';
  const SOURCE_SHEET = '当日出欠';

  const targetSS = SpreadsheetApp.getActiveSpreadsheet();
  const targetSheet = targetSS.getSheetByName('クラス選択');

  const sourceSS = SpreadsheetApp.openById(SOURCE_ID);
  const sourceSheet = sourceSS.getSheetByName(SOURCE_SHEET);

  const data = sourceSheet.getDataRange().getValues();

  // 6/14の抽出欄をクリア
  targetSheet.getRange('F12:F').clearContent();

  // E列に「出席」を含む行だけ抽出し、C列の名前を取得
  const students = data.slice(1)
    .filter(row => String(row[4]).includes('出席'))
    .map(row => [row[2]]);

  // F12から名前だけ表示
  if (students.length > 0) {
    targetSheet
      .getRange(12, 6, students.length, 1)
      .setValues(students);
  }
}

function addCustomFromForm(e) {
  // フォーム回答シート以外からの実行は無視
  const responseSheet = e.range.getSheet();
  if (responseSheet.getName() !== 'フォームの回答 1') return;

  const targetSS = SpreadsheetApp.getActiveSpreadsheet();
  const targetSheet = targetSS.getSheetByName('クラス選択');

  // タイムスタンプを除いて、名前1〜名前10を取得
  const names = e.values
    .slice(1)
    .map(name => String(name).trim())
    .filter(name => name !== '');

  if (names.length === 0) return;

  // Custom（G列）の最後の名前を探す
  const lastRow = Math.max(targetSheet.getLastRow(), 12);
  const customValues = targetSheet
    .getRange(12, 7, lastRow - 11, 1)
    .getValues();

  let writeRow = 12;

  for (let i = customValues.length - 1; i >= 0; i--) {
    if (String(customValues[i][0]).trim() !== '') {
      writeRow = 12 + i + 1;
      break;
    }
  }

  // G列に追加
  targetSheet
    .getRange(writeRow, 7, names.length, 1)
    .setValues(names.map(name => [name]));
}

function clearAllData() {
  const targetSS = SpreadsheetApp.getActiveSpreadsheet();
  const targetSheet = targetSS.getSheetByName('クラス選択');

  targetSheet.getRange('A12:G').clearContent();
}

function openPickOne() {
  const url = 'https://kira-nihongo-online.github.io/PickOne/';

  const html = HtmlService.createHtmlOutput(`
    <!DOCTYPE html>
    <html>
      <head>
        <base target="_blank">
        <style>
          body {
            font-family: Arial, sans-serif;
            text-align: center;
            padding: 25px 20px;
          }

          h2 {
            margin: 0 0 20px;
          }

          a {
            display: inline-block;
            padding: 12px 24px;
            background: #007acc;
            color: white;
            text-decoration: none;
            border-radius: 8px;
            font-size: 16px;
          }

          a:hover {
            background: #005fa3;
          }
        </style>
      </head>

      <body>
        <h2>PickOne</h2>

        <a href="${url}" target="_blank">
          PickOneを開く
        </a>
      </body>
    </html>
  `)
  .setWidth(300)
  .setHeight(180);

  SpreadsheetApp.getUi().showModalDialog(html, 'PickOne');
}

function doGet() {
  const sheet = SpreadsheetApp
    .getActiveSpreadsheet()
    .getSheetByName('クラス選択');

  const lastRow = sheet.getLastRow();

  let names = [];

  if (lastRow >= 12) {
    const values = sheet
      .getRange(12, 1, lastRow - 11, 7)
      .getDisplayValues();

    names = values
      .flat()
      .map(name => String(name).trim())
      .filter(name => name !== '');
  }

  const result = {
    names: names
  };

  return ContentService
    .createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);
}

function openCustomForm() {
  const FORM_URL = 'https://docs.google.com/forms/d/e/1FAIpQLSeHXY-Oxxt7f7yIzgIUxInz6BpOoVx8d-mQjgePxKn33rQ7Bg/viewform';

  const qrUrl =
    'https://quickchart.io/qr?text=' +
    encodeURIComponent(FORM_URL) +
    '&size=180';

  const html = HtmlService.createHtmlOutput(`
    <div style="text-align:center; font-family:Arial,sans-serif;">
      <a href="${FORM_URL}" target="_blank"
         style="font-size:16px;">
        Customフォームを開く
      </a>

      <div style="margin-top:15px;">
        <img src="${qrUrl}" width="180" height="180">
      </div>

      <div style="font-size:13px; margin-top:8px;">
        กรุณาสแกนด้วยโทรศัพท์มือถือ
      </div>
    </div>
  `)
  .setWidth(300)
  .setHeight(330);

  SpreadsheetApp.getUi().showModalDialog(html, 'Custom');
}

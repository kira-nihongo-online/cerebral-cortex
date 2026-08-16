# 漢字道場_GASコード

## プロジェクト

漢字道場

## 情報の種類

その他

## 検索キーワード

漢字道場、GAS、Google Apps Script、doGet、getLessonData、generateLessonUrls、setFileIds_1_10、setFileIds_11_20、setFileIds_21_30、漢字リスト、URL発行、FileId

## 保存場所

その他/漢字道場_GASコード.md

## 内容

漢字道場_GASコード

//-------------------------------------
// 空白・前後スペース除去
//-------------------------------------
function normalize(text) {
  if (!text) return "";
  return text.toString().replace(/\s+/g, "").trim();
}

function getCardImageUrl(fileId) {
  return "https://lh3.googleusercontent.com/d/" + fileId;
}

/*************************************************
 * API
 *************************************************/
function doGet(e) {

  if (!e || !e.parameter || !e.parameter.lesson) {

    return ContentService
      .createTextOutput(JSON.stringify({
        error: "lesson が指定されていません。"
      }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  const lesson = normalize(e.parameter.lesson);
  const cards = getLessonData(lesson);

  return ContentService
    .createTextOutput(JSON.stringify(cards))
    .setMimeType(ContentService.MimeType.JSON);
}
// =============================
// 課選択ダイアログ
// =============================
function selectLesson() {
  const ui = SpreadsheetApp.getUi();
  const response = ui.prompt("何課を出力しますか？（数字だけ）");

  if (response.getSelectedButton() == ui.Button.OK) {
    const lesson = normalize(response.getResponseText());
    createLessonSheet(lesson);
  }
}

// =============================
// 課ごとにシート生成
// =============================
function createLessonSheet(lesson) {

  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const source = ss.getSheetByName("漢字リスト");
  const data = source.getDataRange().getValues();

  const outputName = "第" + lesson + "課";
  let output = ss.getSheetByName(outputName);

  if (!output) {
    output = ss.insertSheet(outputName);
  } else {
    output.clear();
  }

  const header = data[0];
  const filtered = [header];

  for (let i = 1; i < data.length; i++) {
    if (normalize(data[i][0]) == lesson) {
      filtered.push(data[i]);
    }
  }

  if (filtered.length === 1) {
    SpreadsheetApp.getUi().alert("データがありません");
    return;
  }

  output.getRange(1,1,filtered.length,filtered[0].length).setValues(filtered);
}

function getLessonData(lesson) {

  const SPREADSHEET_ID = "1VZxlEtek4WJRMZVYCkuTEAI2ZfjediJxFYkVuR8S2K4";
  const ss = SpreadsheetApp.openById(SPREADSHEET_ID);
  const sheet = ss.getSheetByName("漢字リスト");
  const data = sheet.getDataRange().getValues();

  const result = [];
  const target = normalize(lesson);

  for (let i = 1; i < data.length; i++) {

    const rowLesson = normalize(data[i][0]);

    if (rowLesson === target) {

      const frontId = data[i][5];
      const backId  = data[i][6];

      result.push({
        kanji: data[i][2],
        hiragana: data[i][3],
        thai: data[i][4],
        front: frontId ? getCardImageUrl(frontId) : "",
        back:  backId  ? getCardImageUrl(backId)  : ""
      });
    }
  }

  return result;
}

// ========================================
// URL発行シート生成（数値順・各課1行のみ）
// ========================================
function generateLessonUrls() {

  const BASE_URL = "https://kira-nihongo-online.github.io/kanji-dojo/";

  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const listSheet = ss.getSheetByName("漢字リスト");
  const data = listSheet.getRange("A2:A").getValues().flat().filter(String);

  // 数値化してユニーク＋昇順ソート
  const lessons = [...new Set(data.map(Number))]
    .filter(n => !isNaN(n))
    .sort((a, b) => a - b);

  const urlSheet = ss.getSheetByName("URL発行");
  if (!urlSheet) throw new Error("URL発行シートがありません");

  urlSheet.clear();

  // ヘッダ
  urlSheet.getRange(1,1,1,2).setValues([["課","スタートURL"]]);

  // 出力データ作成
  const output = lessons.map(l => [
    l,
    BASE_URL + "?lesson=" + l
  ]);

  if (output.length > 0) {
    urlSheet.getRange(2,1,output.length,2).setValues(output);
  }
}

// =============================
// FileId自動取得（1〜10課）
// =============================
function setFileIds_1_10() {

  const CARDS_FOLDER_ID = "1IUwO8OF_gBvlqOt49l3hTsudpAEODMJB";

  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName("漢字リスト");
  const data = sheet.getDataRange().getValues();

  const baseFolder = DriveApp.getFolderById(CARDS_FOLDER_ID);

  for (let i = 1; i < data.length; i++) {

    const lesson = Number(data[i][0]);
    const kanji  = data[i][2];

    if (lesson < 1 || lesson > 10) continue;
    if (!kanji) continue;

    const lessonFolderName = lesson + "課";
    const folders = baseFolder.getFoldersByName(lessonFolderName);
    if (!folders.hasNext()) continue;

    const lessonFolder = folders.next();

    const frontName = kanji + "_front.png";
    const backName  = kanji + "_back.png";

    let frontId = "";
    let backId  = "";

    const frontFiles = lessonFolder.getFilesByName(frontName);
    if (frontFiles.hasNext()) frontId = frontFiles.next().getId();

    const backFiles = lessonFolder.getFilesByName(backName);
    if (backFiles.hasNext()) backId = backFiles.next().getId();

    sheet.getRange(i + 1, 6).setValue(frontId);
    sheet.getRange(i + 1, 7).setValue(backId);
  }
}

// =============================
// FileId自動取得（11〜20課）
// =============================
function setFileIds_11_20() {

  const CARDS_FOLDER_ID = "1IUwO8OF_gBvlqOt49l3hTsudpAEODMJB";

  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName("漢字リスト");
  const data = sheet.getDataRange().getValues();

  const baseFolder = DriveApp.getFolderById(CARDS_FOLDER_ID);

  for (let i = 1; i < data.length; i++) {

    const lesson = Number(data[i][0]);
    const kanji  = data[i][2];

    if (lesson < 11 || lesson > 20) continue;
    if (!kanji) continue;

    const lessonFolderName = lesson + "課";
    const folders = baseFolder.getFoldersByName(lessonFolderName);
    if (!folders.hasNext()) continue;

    const lessonFolder = folders.next();

    const frontName = kanji + "_front.png";
    const backName  = kanji + "_back.png";

    let frontId = "";
    let backId  = "";

    const frontFiles = lessonFolder.getFilesByName(frontName);
    if (frontFiles.hasNext()) frontId = frontFiles.next().getId();

    const backFiles = lessonFolder.getFilesByName(backName);
    if (backFiles.hasNext()) backId = backFiles.next().getId();

    sheet.getRange(i + 1, 6).setValue(frontId);
    sheet.getRange(i + 1, 7).setValue(backId);
  }
}

// =============================
// FileId自動取得（21〜30課）
// =============================
function setFileIds_21_30() {

  const CARDS_FOLDER_ID = "1IUwO8OF_gBvlqOt49l3hTsudpAEODMJB";

  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName("漢字リスト");
  const data = sheet.getDataRange().getValues();

  const baseFolder = DriveApp.getFolderById(CARDS_FOLDER_ID);

  for (let i = 1; i < data.length; i++) {

    const lesson = Number(data[i][0]);
    const kanji  = data[i][2];

    if (lesson < 21 || lesson > 30) continue;
    if (!kanji) continue;

    const lessonFolderName = lesson + "課";
    const folders = baseFolder.getFoldersByName(lessonFolderName);
    if (!folders.hasNext()) continue;

    const lessonFolder = folders.next();

    const frontName = kanji + "_front.png";
    const backName  = kanji + "_back.png";

    let frontId = "";
    let backId  = "";

    const frontFiles = lessonFolder.getFilesByName(frontName);
    if (frontFiles.hasNext()) frontId = frontFiles.next().getId();

    const backFiles = lessonFolder.getFilesByName(backName);
    if (backFiles.hasNext()) backId = backFiles.next().getId();

    sheet.getRange(i + 1, 6).setValue(frontId);
    sheet.getRange(i + 1, 7).setValue(backId);
  }
}

プロジェクト

漢字道場

情報の種類

その他

内容

漢字道場で使用しているGoogle Apps Scriptのコード。

API

doGet(e)をWeb APIの入口として使用する。

URLにlessonが指定されていない場合は、以下のJSONを返す。

{"error":"lesson が指定されていません。"}

lessonが指定されている場合は、getLessonData(lesson)を実行し、取得したカードデータをJSON形式で返す。

データ取得

getLessonData(lesson)では、以下のスプレッドシートを使用する。

スプレッドシートID：

1VZxlEtek4WJRMZVYCkuTEAI2ZfjediJxFYkVuR8S2K4

使用シート：

漢字リスト

課番号をA列と照合し、一致する行から以下のデータを取得する。

C列：漢字
D列：ひらがな
E列：タイ語
F列：表画像File ID
G列：裏画像File ID

取得したデータは以下の形式で返す。

kanji：漢字
hiragana：ひらがな
thai：タイ語
front：表画像URL
back：裏画像URL

画像URLはFile IDから、

https://lh3.googleusercontent.com/d/ + File ID

の形式で生成する。

課選択

selectLesson()では、Googleスプレッドシートのダイアログで出力する課番号を入力する。

入力された課番号をnormalize()で整形し、createLessonSheet(lesson)を実行する。

課ごとのシート生成

createLessonSheet(lesson)では、「漢字リスト」シートから指定した課のデータを抽出する。

作成するシート名：

第 + 課番号 + 課

該当するシートが存在しない場合は新規作成する。

存在する場合は既存内容をクリアしてから再作成する。

元の「漢字リスト」シートのヘッダーを含め、指定した課のデータを新しいシートへコピーする。

該当データがない場合は「データがありません」と表示する。

URL発行

generateLessonUrls()では、「漢字リスト」シートのA列から課番号を取得する。

課番号を数値化し、重複を除去して昇順に並べる。

「URL発行」シートを使用し、既存内容をクリアする。

ヘッダー：

課
スタートURL

GitHub PagesのベースURL：

https://kira-nihongo-online.github.io/kanji-dojo/

各課のURLは、

https://kira-nihongo-online.github.io/kanji-dojo/?lesson= + 課番号

の形式で生成する。

File ID自動取得

Google Drive内のカード画像からFile IDを取得し、「漢字リスト」シートへ登録する。

使用するGoogle DriveフォルダID：

1IUwO8OF_gBvlqOt49l3hTsudpAEODMJB

課ごとのフォルダ名：

1課、2課、3課 … の形式。

画像ファイル名：

漢字_front.png
漢字_back.png

表画像のFile IDをF列へ登録する。

裏画像のFile IDをG列へ登録する。

File ID取得処理は課の範囲ごとに分かれている。

setFileIds_1_10()
1～10課

setFileIds_11_20()
11～20課

setFileIds_21_30()
21～30課

normalize

normalize(text)では、文字列内の空白を除去し、前後の空白も取り除く。

空の値の場合は空文字を返す。

確認できるデータ構成

「漢字リスト」シートでは、このGASコードから少なくとも以下の列が使用されている。

A列：課
C列：漢字
D列：ひらがな
E列：タイ語
F列：表画像File ID
G列：裏画像File ID

また、URL発行処理では「URL発行」シートを使用する。

GitHub

GitHub PagesのURLとして、以下のリポジトリの公開URLがコード内に設定されている。

kira-nihongo-online/kanji-dojo

公開URL：

https://kira-nihongo-online.github.io/kanji-dojo/

コード内で確認できる主な関数

normalize(text)

getCardImageUrl(fileId)

doGet(e)

selectLesson()

createLessonSheet(lesson)

getLessonData(lesson)

generateLessonUrls()

setFileIds_1_10()

setFileIds_11_20()

setFileIds_21_30()

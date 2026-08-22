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

  const result = {
    A: testA,
    B: testB
  };

  // ==============================
  // 問題保存（スライドとPDF一致）
  // ==============================
  const cache = CacheService.getScriptCache();
  cache.put("KANJI_TEST_" + lesson, JSON.stringify(result), 21600);

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

  return generateTestSet(lesson);

}


// ================================
// Webアプリ入口
// ================================

function doGet(e){

  const lesson = Number(e.parameter.lesson) || 1;

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
// TestCards画像IDリスト作成
// ==============================
function createImageIdList() {

  const TESTCARDS_FOLDER_ID = "1wKxkySp2_z5cdyL6V6zjliYkIbYOv9go";

  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("漢字リスト");
  const data = sheet.getDataRange().getValues();

  const root = DriveApp.getFolderById(TESTCARDS_FOLDER_ID);
  const lessonFolders = root.getFolders();

  const imageMap = {};

  while (lessonFolders.hasNext()) {

    const lessonFolder = lessonFolders.next();
    const files = lessonFolder.getFiles();

    while (files.hasNext()) {

      const file = files.next();

      if (file.getName().toLowerCase().endsWith(".png")) {

        const name = file.getName().replace(/\.png$/i, "");
        const id = file.getId();

        imageMap[name] = id;

      }

    }

  }

  for (let i = 1; i < data.length; i++) {

    const kanji = data[i][2];

    if (imageMap[kanji]) {
      sheet.getRange(i + 1, 5).setValue(imageMap[kanji]);
    }

  }

  Logger.log("画像ID作成完了");

}


// ================================
// PDF生成実行（シートボタン用）
// ================================

function runCreatePDF(){

  const ui = SpreadsheetApp.getUi();

  const res = ui.prompt(
    "PDF作成",
    "課番号を入力してください（例：27）",
    ui.ButtonSet.OK_CANCEL
  );

  if(res.getSelectedButton() != ui.Button.OK) return;

  const lesson = res.getResponseText();

  createTestPDF(lesson);

  ui.alert("PDF作成完了");

}


// ================================
// PDF作成
// ================================

function createTestPDF(lesson){

  const data = generateTestSet(lesson);

  createPDF("A", lesson, data.A, false);
  createPDF("A", lesson, data.A, true);

  createPDF("B", lesson, data.B, false);
  createPDF("B", lesson, data.B, true);

}


// ================================
// PDF生成本体
// ================================

function createPDF(type, lesson, list, isAnswer){

  const template = HtmlService.createTemplateFromFile("pdf");

  template.type = type;
  template.lesson = lesson;
  template.list = list;
  template.isAnswer = isAnswer;

  const html = template.evaluate().getContent();

  const blob = Utilities.newBlob(html,"text/html")
  .getAs("application/pdf");

  const name =
    "KanjiTest_" +
    type +
    "_Lesson" +
    lesson +
    (isAnswer ? "_Answer" : "_Question") +
    ".pdf";

  DriveApp.createFile(blob).setName(name);

}

問題生成エンジン、保存された問題取得、Webアプリ入口、課CSVダウンロード、空白・前後スペース除去、TestCards画像IDリスト作成、PDF生成実行、PDF作成、PDF生成本体のGASコード。

generateTestSet(lesson)：
「漢字リスト」シートから指定した課のデータを取得し、A問題とB問題をそれぞれシャッフルして最大20問ずつ作成する。取得するデータはC列の漢字、D列の読み、E列のID。作成したA・B問題をScript Cacheに「KANJI_TEST_」＋課番号のキーで21600秒保存する。

getSavedTestSet(lesson)：
Script Cacheに保存された問題セットを取得する。保存されていない場合はgenerateTestSet(lesson)を実行する。

doGet(e)：
lessonパラメータから課番号を取得する。mode=jsonの場合は問題セットをJSON形式で返す。それ以外の場合はHTMLファイル「dual」をテンプレートとして表示する。

exportLessonCsv()：
入力した課番号をもとに「漢字リスト」シートから該当するデータを抽出し、「課,No,漢字,読み」のCSVを作成する。ダウンロード名はtest_cards.csv。

normalize(text)：
文字列から空白文字を除去し、前後の空白を取り除く。

createImageIdList()：
Google Driveの指定フォルダ内にあるサブフォルダからPNG画像を取得し、画像ファイル名と「漢字リスト」シートC列の漢字を照合する。一致した画像のGoogle DriveファイルIDをE列へ設定する。

runCreatePDF()：
課番号を入力し、createTestPDF(lesson)を実行してPDFを作成する。

createTestPDF(lesson)：
generateTestSet(lesson)で作成したA問題・B問題について、それぞれ問題PDFと解答PDFを作成する。

createPDF(type, lesson, list, isAnswer)：
HTMLファイル「pdf」をテンプレートとして使用し、指定された問題データからPDFを生成する。生成するファイル名は「KanjiTest_」＋問題タイプ＋「_Lesson」＋課番号＋「_Question.pdf」または「_Answer.pdf」。

コード内で参照されているスプレッドシートのシート名：
「漢字リスト」

コード内で参照されているHTMLファイル：
「dual」
「pdf」

コード内で使用されているGoogle DriveフォルダID：
「1wKxkySp2_z5cdyL6V6zjliYkIbYOv9go」

上記以外のスプレッドシート構成、HTMLファイルの内容、Google Driveフォルダ名、GitHubリポジトリ構成については、このコードだけでは確認できない。

# 解答システム_GASコード

## プロジェクト

解答システム_2026.8.22

## 情報の種類

その他

## 検索キーワード

解答システム、kaito-system、GASコード

## 保存場所

GitHub／GAS

## 内容

Google Apps Script（GAS）とGoogleスプレッドシートを使用して、教材情報・解答情報・受験結果を管理する。

// =========================
// 設定
// =========================
const SHEET_MATERIALS = "教材マスター";
const SHEET_ANSWERS = "解答マスター";
const SHEET_RESULTS = "受験結果";

// =========================
// GET API
// =========================
function doGet(e) {

  const action = e.parameter.action;

  switch (action) {

    case "materials":
      return outputJson(getMaterials());

    case "answers":
      return outputJson(getAnswers(e.parameter.materialId));

    default:
      return outputJson({
        success: false,
        message: "Unknown action"
      });

  }

}

// =========================
// JSON出力
// =========================
function outputJson(data){

  return ContentService
    .createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);

}

// =========================
// 教材一覧取得
// =========================
function getMaterials(){

  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_MATERIALS);

  const values = sheet.getDataRange().getValues();

  values.shift(); // ヘッダー削除

  return values.map(r => ({
    materialId : r[0],
    name       : r[1],
    level      : r[2],
    count      : r[3],
    url        : r[4]
  }));

}

// =========================
// 解答取得
// =========================
function getAnswers(materialId){

  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_ANSWERS);

  const values = sheet.getDataRange().getValues();

  values.shift();

  const parts = materialId.split("-");
  const examId = parts[0] + "-" + parts[1];
  const section = parts[2];

  const result = [];

  values.forEach(r=>{

    if (r[0] != examId) return;

    if (section != "all") {

  if (section == "choukai") {
    if (!r[3].startsWith("choukai")) return;
  } else {
    if (r[3] != section) return;
  }

}

    result.push({

      materialId : r[0],
      level      : r[1],
      number     : r[2],
      type       : r[3],
      answer     : r[4],
      point      : r[5],
      memo       : r[6]

    });

  });

  return result;

}

// =========================
// POST API
// =========================
function doPost(e){

  const data = JSON.parse(e.postData.contents);

  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_RESULTS);

  const materialId = data.materialId;
  const userName = data.userName;
  const answers = data.answers;

  const examId = materialId.split("-").slice(0, 2).join("-");
  const answerMaster = getAnswers(examId + "-all");

  const correctAnswers = {};

  answerMaster.forEach(item => {

    const key =
      item.type == "goi"
        ? "goi-" + item.number
        : item.type == "bunpou"
        ? "bunpou-" + item.number
        : item.type == "dokkai"
        ? "dokkai-" + item.number
        : item.type.startsWith("choukai")
        ? item.type + "-" + item.number
        : String(item.number);

    correctAnswers[key] = item.answer;

  });
  
  let score = 0;
  let goiScore = 0;
  let goiTotal = 0;
  let bunpouScore = 0;
  let bunpouTotal = 0;
  let dokkaiScore = 0;
  let dokkaiTotal = 0;
  let choukaiScore = 0;
  let choukaiTotal = 0;

  const questionResult = {};

  answerMaster.forEach(item => {

    if (item.type == "goi") {
        goiTotal++;
    }

    if (item.type == "bunpou") {
        bunpouTotal++;
    }

    if (item.type == "dokkai") {
        dokkaiTotal++;
    }

    if (item.type.startsWith("choukai")) {
        choukaiTotal++;
    }

});

  for (const question in answers) {

    const isCorrect =
      String(answers[question]) == String(correctAnswers[question]);

      if (question.startsWith("goi-")) {

      if (isCorrect) {
          goiScore++;
      }

  }

      if (question.startsWith("bunpou-")) {

      if (isCorrect) {
          bunpouScore++;
      }

  }

      if (question.startsWith("dokkai-")) {

      if (isCorrect) {
          dokkaiScore++;
      }

  }

      if (question.startsWith("choukai")) {

          if (isCorrect) {
              choukaiScore++;
          }

      }

      if (isCorrect) {
       score++;
      }

      questionResult[question] = isCorrect;

    sheet.appendRow([
      new Date(),
      userName,
      materialId,
      question,
      answers[question],
      isCorrect ? "○" : "×"
    ]);

  }

  return outputJson({
      success: true,
      score,
      total: answerMaster.length,

      questionResult: questionResult,
      correctAnswers: correctAnswers,

      sectionScore: {
          goi: {
              score: goiScore,
              total: goiTotal
          },

          bunpou: {
              score: bunpouScore,
              total: bunpouTotal
          },

          dokkai: {
              score: dokkaiScore,
              total: dokkaiTotal
          },

          choukai: {
              score: choukaiScore,
              total: choukaiTotal
          },
      }
  });

  }

使用するスプレッドシートのシート：

・教材マスター
・解答マスター
・受験結果

【教材マスター】

教材一覧を管理する。

列構成：

A列：materialId
B列：name
C列：level
D列：count
E列：url

【解答マスター】

各教材の正解データを管理する。

列構成：

A列：materialId / examId
B列：level
C列：number
D列：type
E列：answer
F列：point
G列：memo

typeには以下の区分を使用する。

・goi：語彙
・bunpou：文法
・dokkai：読解
・choukaiから始まる値：聴解

【受験結果】

受験者が回答した問題ごとの結果を記録する。

記録内容：

・日時
・ユーザー名
・教材ID
・問題識別子
・ユーザーの回答
・正誤（○／×）

【GAS API】

GETで以下のAPIを提供する。

action=materials
→ 教材マスターから教材一覧を取得。

action=answers&materialId=○○
→ 指定教材の解答データを取得。

materialIdは「試験ID-セクション」の形式で処理する。

sectionがallの場合は試験IDに該当する全問題を取得する。

sectionがchoukaiの場合は、typeがchoukaiで始まる問題を取得する。

それ以外のsectionでは、typeが指定されたsectionと一致する問題を取得する。

POSTでは受験結果を受け取る。

POSTデータ：

・materialId
・userName
・answers

materialIdから試験IDを取得し、その試験全体の解答マスターを取得する。

ユーザーの回答と正解を比較して、以下を計算する。

・総合得点
・語彙得点／問題数
・文法得点／問題数
・読解得点／問題数
・聴解得点／問題数

各問題について正誤を判定し、「○」または「×」として受験結果シートへ記録する。

APIのレスポンスには以下を含める。

・success
・score
・total
・questionResult
・correctAnswers
・sectionScore

sectionScoreには、語彙・文法・読解・聴解それぞれのscoreとtotalを含める。

# 朗読プロンプター_GASコード

## プロジェクト

朗読プロンプター

## 情報の種類

その他

## 検索キーワード

朗読プロンプター、2026.8.22

## 保存場所

GitHub / スプレッド_APPS

## 内容

function testTTS() {

  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("TTS原稿");

  const data = sheet.getDataRange().getValues();

  let text = "";

  for(let i = 1; i < data.length; i++){

    if(data[i][0] === "sample"){

      text += data[i][2] + "\n";

    }

  }

  Logger.log(text);

}

function doGet(e){

  const sheet = SpreadsheetApp
    .getActiveSpreadsheet()
    .getSheetByName("TTS原稿");

  const data = sheet.getDataRange().getValues();

  const lesson = e.parameter.lesson;

  let text = "";

  for(let i = 1; i < data.length; i++){

    if(data[i][0] === lesson){

      text += data[i][2] + "\n";

    }

  }

  return ContentService
    .createTextOutput(text)
    .setMimeType(ContentService.MimeType.TEXT);

}

朗読プロンプターのTTS原稿取得用Google Apps Script。

Googleスプレッドシートの「TTS原稿」シートを参照する。

testTTS() は、A列が sample の行を対象に、C列の文章を取得して改行で連結し、Loggerに出力する。TTS原稿の取得確認用。

doGet(e) は、URLパラメータ lesson を受け取り、A列が指定されたlessonと一致する行のC列の文章を取得して改行で連結し、プレーンテキストとして返す。朗読プロンプターからレッスン別のTTS原稿を取得するために使用する。

朗読プロンプター本体では、GASのWebアプリURLに ?lesson=... を付けてTTS原稿を取得する構成になっている。

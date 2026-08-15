# G3オンライン領収書_GASコード

## プロジェクト

G3オンライン領収書

## 情報の種類

その他

## 検索キーワード

G3オンライン領収書、G3 Online Receipt、GAS、Google Apps Script、Master_Data、出席、受講回数、出席履歴、月シート、領収書、GitHub、JSON API、ID

## 保存場所



## 内容

【概要】
// =======================================
// ★ QR出欠（1日1回制限・時間更新型）
// =======================================
function doGet(e) {

  // =======================================
  // 領収書データ要求
  // =======================================
  if (e.parameter.mode === "receipt") {
    return ContentService
      .createTextOutput(
        JSON.stringify(getReceiptData(e.parameter.id))
      )
      .setMimeType(ContentService.MimeType.JSON);
  }

  const ss = SpreadsheetApp.getActive();
  const sheet = ss.getSheetByName('Master_Data');
  if (!sheet) {
    return ContentService.createTextOutput('Master_Dataがありません');
  }

  const idParam = e?.parameter?.id;
  if (!idParam) {
    return ContentService.createTextOutput('IDなし');
  }

  const now = new Date();
  const todayStr = Utilities.formatDate(now, 'Asia/Bangkok', 'yyyy/MM/dd');
  const data = sheet.getDataRange().getValues();
  const headers = data[0];

  // ===== ヘッダー位置取得 =====
  const colIndex = {};
  headers.forEach((h, i) => colIndex[String(h).trim()] = i);

  if (colIndex["ID"] === undefined ||
      colIndex["出席（มาเรียน）"] === undefined ||
      colIndex["最終更新時刻"] === undefined ||
      colIndex["受講回数"] === undefined) {
    return ContentService.createTextOutput('必要な列が不足しています');
  }

  let updatedRow = null;
  let updatedRowNumber = 0;

  // ===== Master_Data 更新 =====
  for (let i = 1; i < data.length; i++) {

    if (String(data[i][colIndex["ID"]]).trim() === String(idParam).trim()) {

      sheet.getRange(i + 1, colIndex["出席（มาเรียน）"] + 1)
        .setValue('出席（มาเรียน）');

      sheet.getRange(i + 1, colIndex["最終更新時刻"] + 1)
        .setValue(now)
        .setNumberFormat("yyyy/MM/dd HH:mm:ss");

      const lessonCount =
          Number(data[i][colIndex["受講回数"]]) || 0;

      sheet.getRange(i + 1, colIndex["受講回数"] + 1)
          .setValue(lessonCount + 1);

      SpreadsheetApp.flush();

      // ===== 今回の受講回数を取得 =====
      const currentLessonCount =
          Number(sheet.getRange(i + 1, colIndex["受講回数"] + 1).getValue()) || 0;

      Logger.log("今回の受講回数 = " + currentLessonCount);

      // ===== 履歴保存先の列を決定 =====
      const historyColumn = 11 + currentLessonCount - 1;

      Logger.log("履歴保存先の列番号 = " + historyColumn);

      // ===== 出席履歴に今日の日付を保存 =====
      sheet.getRange(i + 1, historyColumn)
          .setValue(now)
          .setNumberFormat("yyyy/MM/dd");

      Logger.log("出席履歴を保存 = " + Utilities.formatDate(now, "Asia/Bangkok", "yyyy/MM/dd"));

      updatedRowNumber = i + 1;

      updatedRow = sheet
        .getRange(i + 1, 1, 1, sheet.getLastColumn())
        .getValues()[0];

      break;
    }
  }

  if (!updatedRow) {
    return ContentService.createTextOutput('ID未登録');
  }

  // ===== 月シート =====
  const monthName = Utilities.formatDate(now, 'Asia/Bangkok', 'yyyy-MM');
  let monthSheet = ss.getSheetByName(monthName);

  if (!monthSheet) {
    monthSheet = ss.insertSheet(monthName);
    monthSheet.getRange(1, 1, 1, headers.length).setValues([headers]);
    monthSheet.protect().setWarningOnly(true);
  }

  const monthData = monthSheet.getDataRange().getValues();
  const monthHeaders = monthData[0];

  const mColIndex = {};
  monthHeaders.forEach((h, i) => mColIndex[h.trim()] = i);

  let foundRow = null;

  // ===== 同日チェック =====
  for (let i = 1; i < monthData.length; i++) {

    const rowId = String(monthData[i][mColIndex["ID"]]).trim();
    const rowDate = Utilities.formatDate(
      new Date(monthData[i][mColIndex["最終更新時刻"]]),
      'Asia/Bangkok',
      'yyyy/MM/dd'
    );

    if (rowId === String(idParam).trim() && rowDate === todayStr) {
      foundRow = i + 1;
      break;
    }
  }

  if (foundRow) {

    // ★ 時刻だけ更新
    monthSheet.getRange(foundRow, mColIndex["最終更新時刻"] + 1)
      .setValue(now)
      .setNumberFormat("yyyy/MM/dd HH:mm:ss");

    // ★ 受講回数を元に戻す
    const currentCount =
        Number(sheet.getRange(updatedRowNumber, colIndex["受講回数"] + 1).getValue()) || 0;

    if (currentCount > 0) {
      sheet.getRange(updatedRowNumber, colIndex["受講回数"] + 1)
          .setValue(currentCount - 1);
    }

    return ContentService.createTextOutput('時間更新');
  }

  // ★ 新規追加
  monthSheet.appendRow(updatedRow);

  return ContentService.createTextOutput('新規登録');

  }

// =======================================
// 月シート履歴修復（同日ID重複削除）
// =======================================
function repairMonthSheet() {

  const ss = SpreadsheetApp.getActive();
  const sheet = ss.getSheetByName("2026-03");

  const data = sheet.getDataRange().getValues();
  const headers = data[0];

  const colIndex = {};
  headers.forEach((h,i)=> colIndex[h.trim()] = i);

  const map = {};
  const rowsToDelete = [];

  for (let i = 1; i < data.length; i++) {

    const id = data[i][colIndex["ID"]];
    const date = Utilities.formatDate(
      new Date(data[i][colIndex["最終更新時刻"]]),
      "Asia/Bangkok",
      "yyyy/MM/dd"
    );

    const key = id + "_" + date;

    if (map[key]) {
      rowsToDelete.push(i+1);
    } else {
      map[key] = true;
    }
  }

  rowsToDelete.reverse().forEach(r => sheet.deleteRow(r));
}

// =======================================
// 領収書データ取得
// =======================================
function getReceiptData(id) {

  const ss = SpreadsheetApp.getActive();
  const sheet = ss.getSheetByName("Master_Data");

  const data = sheet.getDataRange().getValues();

  // ===== ID指定がない場合は2行目（今まで通り）=====
  if (!id) {

    const row = data[1];

    // ===== 1～30回の履歴取得 =====
    const history = [];

    for (let i = 0; i < 30; i++) {
      const value = row[10 + i]; // K列～AN列

      if (value) {
        history.push(
          Utilities.formatDate(
            new Date(value),
            "Asia/Bangkok",
            "yyyy/MM/dd"
          )
        );
      } else {
        history.push("");
      }
    }

    return {
      name: row[1],
      course: row[5],
      date: Utilities.formatDate(
        new Date(row[6]),
        "Asia/Bangkok",
        "yyyy/MM/dd"
      ),
      count: row[7],
      history: history
    };
  }

  // ===== ID検索 =====
  for (let i = 1; i < data.length; i++) {

    if (String(data[i][2]).trim() === String(id).trim()) {

      const row = data[i];

      // ===== 1～30回の履歴取得 =====
      const history = [];

      for (let j = 0; j < 30; j++) {
        const value = row[10 + j]; // K列～AN列

        if (value) {
          history.push(
            Utilities.formatDate(
              new Date(value),
              "Asia/Bangkok",
              "yyyy/MM/dd"
            )
          );
        } else {
          history.push("");
        }
      }

      return {
        name: row[1],
        course: row[5],
        date: Utilities.formatDate(
          new Date(row[6]),
          "Asia/Bangkok",
          "yyyy/MM/dd"
        ),
        count: row[7],
        history: history
      };
    }
  }

  return {
    error: "ID not found"
  };
}

// =======================================
// ★ 履歴取得テスト
// =======================================
function testReceiptData() {

  const id = "BETA-004";

  const result = getReceiptData(id);

  Logger.log(JSON.stringify(result, null, 2));
}

// =======================================
// 出席取消
// =======================================
function cancelAttendance() {

  const ss = SpreadsheetApp.getActive();
  const master = ss.getSheetByName("Master_Data");

  const row = master.getActiveCell().getRow();

  if (row <= 1) {
    SpreadsheetApp.getUi().alert("生徒を選択してください。");
    return;
  }

  // ===== H列（受講回数） =====
  const lessonCol = 8;

  let count = Number(master.getRange(row, lessonCol).getValue()) || 0;

  if (count <= 0) {
    SpreadsheetApp.getUi().alert("受講回数は0です。");
    return;
  }

  // ===== ID取得 =====
  const id = master.getRange(row, 3).getValue(); // C列(ID)

  // ===== 今月シート =====
  const monthName = Utilities.formatDate(new Date(), "Asia/Bangkok", "yyyy-MM");
  const monthSheet = ss.getSheetByName(monthName);

  if (!monthSheet) {
    SpreadsheetApp.getUi().alert("今月シートがありません。");
    return;
  }

  const data = monthSheet.getDataRange().getValues();

  // 下から探す（最新データ）
  for (let i = data.length - 1; i >= 1; i--) {

    if (String(data[i][2]).trim() === String(id).trim()) {

      monthSheet.deleteRow(i + 1);

      break;
    }

  }

  // ===== 出席履歴を削除 =====
  // 現在の受講回数に対応する履歴を削除
  const historyColumn = 11 + count - 1;

  master.getRange(row, historyColumn).clearContent();

  // ===== 受講回数 -1 =====
  master.getRange(row, lessonCol).setValue(count - 1);

  SpreadsheetApp.getUi().alert("出席を取り消しました。");
  }

// =======================================
// ★ セル選択（送信用ID保存）
// =======================================
function onSelectionChange(e) {

  // ===== シート取得 =====
  const sheet = e.range.getSheet();

  // Master_Data以外は終了
  if (sheet.getName() != "Master_Data") return;

  // ===== 行・列取得 =====
  const row = e.range.getRow();
  const col = e.range.getColumn();

  // I列以外は終了
  if (col != 9) return;

  // ===== 行データ取得 =====
  const data = sheet
    .getRange(row, 1, 1, sheet.getLastColumn())
    .getValues()[0];

  // ===== ID保存 =====
  PropertiesService
    .getScriptProperties()
    .setProperty("SELECTED_ID", data[2]);

  // ===== 確認ログ =====
  Logger.log("保存ID = " + data[2]);

}

// =======================================
// ★ GitHub送信
// =======================================
function sendToGitHub() {

  // ===== 保存済みID取得 =====
  const id = PropertiesService
    .getScriptProperties()
    .getProperty("SELECTED_ID");

  // ===== ID未選択 =====
  if (!id) {
    SpreadsheetApp.getUi().alert("先に生徒を選択してください。");
    return;
  }

  // ===== Master_Data取得 =====
  const sheet = SpreadsheetApp
    .getActive()
    .getSheetByName("Master_Data");

  const data = sheet.getDataRange().getValues();

  let row = null;

  // ===== ID検索 =====
  for (let i = 1; i < data.length; i++) {

    if (String(data[i][2]).trim() == String(id).trim()) {

      row = data[i];
      break;

    }

  }

  if (!row) {
    SpreadsheetApp.getUi().alert("データが見つかりません。");
    return;
  }

  // ===== GitHub URL =====
  const githubUrl =
    "https://kira-nihongo-online.github.io/g3-online-receipt/?id=" +
    encodeURIComponent(id);

  // ===== 領収日 =====
  const receiptDate = Utilities.formatDate(
    new Date(row[6]),
    "Asia/Bangkok",
    "yyyy/MM/dd"
  );

  // ===== HTML =====
  const html = HtmlService.createHtmlOutput(
    '<div style="font-family:Arial;padding:20px;">' +

    '<h2 style="margin-top:0;">GitHub送信確認</h2>' +

    '<table style="border-collapse:collapse;">' +

    '<tr><td><b>名前</b></td><td>：' + row[1] + '</td></tr>' +
    '<tr><td><b>ID</b></td><td>：' + row[2] + '</td></tr>' +
    '<tr><td><b>コース</b></td><td>：' + row[5] + '回</td></tr>' +
    '<tr><td><b>領収日</b></td><td>：' + receiptDate + '</td></tr>' +
    '<tr><td><b>受講回数</b></td><td>：' + row[7] + '回</td></tr>' +

    '</table>' +

    '<hr>' +

    '<p>' +
    '<a href="' + githubUrl + '" target="_blank" ' +
    'style="font-size:16px;font-weight:bold;">' +
    '🔗 領収書を開く' +
    '</a>' +
    '</p>' +

    '</div>'
  )
  .setWidth(520)
  .setHeight(300);

  SpreadsheetApp.getUi().showModalDialog(
    html,
    "GitHub送信"
  );

}


Googleスプレッドシートの「Master_Data」をデータの基盤として使用し、出席情報・受講回数・出席履歴を管理する。

また、Webアプリからの領収書データ取得、月別履歴の管理、出席取消、GitHub上の領収書ページへのリンク生成などを行う。

【主な処理】

・doGet
Webアプリからのリクエストを受け付ける。

mode=receiptの場合：
IDを指定して領収書データをJSON形式で返す。

通常の場合：
IDを使用して「Master_Data」を検索し、出席処理を行う。

・Master_Data更新
IDが一致する生徒について、
出席状態を「出席（มาเรียน）」に変更する。

「最終更新時刻」を現在時刻に更新する。

「受講回数」を1増加する。

受講回数に対応する出席履歴欄へ当日の日付を保存する。

・月別シート
「yyyy-MM」形式の月別シートを使用する。

存在しない場合は自動作成し、「Master_Data」と同じヘッダーを設定する。

同じIDが同日にすでに登録されている場合は、新しい行を追加せず、最終更新時刻だけを更新する。

その場合、Master_Dataの受講回数は元に戻す。

新規の場合は月別シートへ行を追加する。

・月別履歴修復
指定した月シートについて、同じID・同じ日付の重複データを確認し、重複行を削除する。

・getReceiptData
「Master_Data」から領収書表示用のデータを取得する。

ID指定時は該当する生徒を検索する。

取得する主な情報：
・名前
・コース
・領収日
・受講回数
・受講履歴

受講履歴はK列～AN列の最大30回分を取得する。

・testReceiptData
指定したIDを使用してgetReceiptDataをテストし、結果をログへ出力する。

・cancelAttendance
「Master_Data」で選択した生徒の最新の出席を取り消す。

今月の月別シートから該当IDの最新データを削除する。

Master_Dataの対応する出席履歴を削除する。

受講回数を1減らす。

・onSelectionChange
「Master_Data」のI列を選択したとき、その行のIDをScript Propertiesへ保存する。

保存キー：
SELECTED_ID

・sendToGitHub
Script Propertiesに保存されたIDを使用して対象生徒を検索する。

GitHub Pages上のG3オンライン領収書URLを生成する。

生成するURL：
https://kira-nihongo-online.github.io/g3-online-receipt/?id=対象ID

名前、ID、コース、領収日、受講回数を確認画面に表示し、「領収書を開く」リンクを表示する。

【使用するスプレッドシート】
メインシート：
Master_Data

月別シート：
yyyy-MM形式で自動作成する。

【重要な列】
GASでは以下の項目を使用する。

・ID
・出席（มาเรียน）
・最終更新時刻
・受講回数

領収書データでは名前、コース、領収日、受講回数、最大30回分の履歴を使用する。

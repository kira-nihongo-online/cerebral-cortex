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

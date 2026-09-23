# URL変換ツール_仕様

## プロジェクト

URL変換ツール_2026.9.23

## 情報の種類

仕様

## 検索キーワード

URL変換ツール、url-converter-tool、仕様

## 保存場所

GitHub

## 内容

URL変換ツールは、入力されたURLを用途に応じて変換する。

変換モード

1. ダウンロードURL変換

GoogleスライドのURLからプレゼンテーションIDを取得する。
https://docs.google.com/presentation/d/ID/export/pdf の形式に変換する。

2. Google スライド

GoogleスライドのURLからプレゼンテーションIDを取得する。
URLに slide= がある場合はその値を使用する。
slide= がない場合は id.p を使用する。
https://docs.google.com/presentation/d/ID/present?slide=SLIDE の形式に変換する。

3. Dropbox

URLの dl=0 を dl=1 に変更する。

4. URL末尾追加

入力されたURLの末尾に、指定された文字列を追加する。
共通操作
「変換する」で選択中の変換を実行する。
変換結果を表示する。
「コピー」で変換結果をコピーする。
「クリア」で入力内容と変換結果を消去する。
URLが入力されていない場合はエラーを表示する。
必要な入力が不足している場合はエラーを表示する。

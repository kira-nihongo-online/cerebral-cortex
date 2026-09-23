# Digital Stamp_設計

## プロジェクト

Digital Stamp_2026.9.23

## 情報の種類

設計

## 検索キーワード

Digital Stamp、digital-stamp、設計

## 保存場所

GitHub

## 内容

Digital Stampは、ブラウザ上で円形のデジタルスタンプを作成し、PNG画像として保存するシンプルなWebツール。

画面は、左側にスタンプのプレビュー、右側に設定項目を配置する構成。

スタンプはCanvas上に描画し、入力内容やサイズ・位置・色の変更に合わせてリアルタイムに更新する。

設定項目は以下の構成。

- Top Message
- Main Text
- Date
- 各文字のサイズ・位置
- 色
- 今日の日付を使用する設定

作成したスタンプはPNGとしてダウンロードできる。

また、設定を3つのSaveスロットに保存し、後からLoadして再利用できる。

保存にはlocalStorageを使用する。

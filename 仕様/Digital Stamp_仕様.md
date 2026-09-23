# Digital Stamp_仕様

## プロジェクト

Digital Stamp

## 情報の種類

仕様

## 検索キーワード

Digital Stamp、digital-stamp、仕様

## 保存場所

GitHub

## 内容

Digital Stampは、入力した文字と日付を円形のスタンプとしてCanvasに表示し、PNG画像としてダウンロードできる。

### スタンプ

- 円形のスタンプを生成する。
- Top Messageを表示する。
- Main Textを表示する。
- 日付を表示する。
- Top Message、Main Text、日付は、それぞれ文字サイズとY位置を変更できる。
- 色を変更できる。
- 日付は今日の日付を自動表示できる。
- 日付形式は `'YY.MM.DD`。

### 保存・読み込み

- 設定をSave 1、Save 2、Save 3の3つのスロットに保存できる。
- 各スロットから設定をLoadできる。
- 保存データはlocalStorageを使用する。
- 保存済みのSaveボタンは赤色で表示する。

### ダウンロード

- 作成したスタンプをPNG画像としてダウンロードできる。
- ファイル名は `digital-stamp.png`。

### 基本動作

- 入力や設定を変更すると、Canvas上のスタンプ表示を更新する。
- 「今日の日付を使用」を有効にすると、現在の日付をスタンプに使用する。

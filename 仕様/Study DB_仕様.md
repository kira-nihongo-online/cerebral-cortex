# Study DB_仕様

## プロジェクト

Study DB_2026.8.22

## 情報の種類

仕様

## 検索キーワード

Study DB、study-db、仕様

## 保存場所

GitHub／GAS

## 内容

Study DBは、日本語教材の単語を検索・表示するWebアプリ。

単語データは、単語道場側のGoogleスプレッドシート「単語リスト」を共通データとして使用する。

Study DBは単語道場および単語道場-Reverseとは独立したWebアプリケーションとして動作する。

---

## 課指定

URLのlessonパラメータによって、表示する課を指定する。

### lesson=1、2、3……

`lesson=1`、`lesson=2`、`lesson=3`……の場合は、指定された課の単語一覧を表示する。

例：

```text
https://kira-nihongo-online.github.io/study-db/?lesson=1

この場合、第1課の単語データを表示する。

課が変われば、lessonパラメータに応じて対応する課のデータを表示する。

lesson=0

lesson=0は検索専用として使用する。

lesson=0の場合は、初期状態では単語一覧を表示しない。

検索ボックスから検索した場合のみ検索結果を表示する。

検索結果の表示には、既存のStudy DBの検索結果表示を使用する。

授業教材や授業スライド等から、lesson=0のURLを別タブで開いて検索用として利用できる。

課別表示

既存の課別表示、

lesson=1

lesson=2

lesson=3

……

の動作は維持する。

指定されたlessonに対応する単語データを表示する。

学習方向モーダル

課別表示時には、現在の課に対応した学習方向モーダルを使用する。

例：

第1課を学習

日本語 → タイ語

タイ語 → 日本語

キャンセル

課が変わった場合は、表示されている課に対応してモーダルの課名も変わる。

例：

第2課を学習
第3課を学習

のように対応する。

設定アイコン

Study DBのヘッダーに設定アイコンを表示する。

設定アイコンを押すと、現在の課に対応した学習方向モーダルを表示する。

設定アイコン専用の別の設定項目は使用しない。

設定アイコンから、現在の課の学習方向を選択できる。

学習方向

学習方向は2種類とする。

日本語 → タイ語

「日本語 → タイ語」を選択した場合、URL台帳B列の該当課URLへジャンプする。

ジャンプ先：

単語道場

学習方向：

日本語 → タイ語

URL形式：

https://kira-nihongo-online.github.io/tango-dojo/?mode=jp&lesson=課No
タイ語 → 日本語

「タイ語 → 日本語」を選択した場合、URL台帳C列の該当課URLへジャンプする。

ジャンプ先：

単語道場-Reverse

学習方向：

タイ語 → 日本語

URL形式：

https://kira-nihongo-online.github.io/tango-dojo-reverse/?mode=thjp&lesson=課No
URL台帳連携

Study DBは単語道場側の「URL台帳」を使用する。

URL台帳の構成：

A列：課No
B列：JP→TH URL
C列：TH→JP URL
D列：Study DB URL
E列：教材
F列：課名

B列

単語道場のURL。

日本語 → タイ語で使用する。

C列

単語道場-ReverseのURL。

タイ語 → 日本語で使用する。

D列

Study DBの課別URL。

lessonパラメータによって対象課を指定する。

URLの取得

URLはGoogle Apps Scriptを経由して取得する。

GASはURL台帳の課Noと、単語リストの課Noを対応させる。

Study DBは現在の課Noに対応するURLを取得する。

取得するURL：

・jpThUrl
・thJpUrl
・studyDbUrl

単語データ

Study DBは、単語道場側の「単語リスト」を共通データとして使用する。

主なデータ：

・教材
・課No
・課名
・No
・ひらがな
・漢字
・タイ語
・英語
・日本語画像ID
・意味画像ID
・メモ

Study DB専用の単語リストは使用しない。

Google Apps Script

Study DBはGoogle Apps ScriptのWeb APIを利用して単語データを取得する。

GASから、

・単語リストの単語データ
・URL台帳のURLデータ

を取得する。

課Noを基準として単語データとURLを対応させる。

Google Drive画像

日本語画像および意味画像はGoogle Driveで管理する。

画像IDは共通の「単語リスト」に保存する。

Study DBはGASから取得した画像IDを使用する。

単語道場・単語道場-Reverse・Study DBで画像データを個別に管理しない。

アプリケーションの独立性

Study DBは独立したWebアプリケーションとして維持する。

単語道場も独立したWebアプリケーションとして維持する。

単語道場-Reverseも独立したWebアプリケーションとして維持する。

3つのアプリケーションを一本化しない。

Study DBに単語道場や単語道場-Reverseの学習機能を組み込まない。

Study DBからURLによって各アプリケーションへ移動する。

データ連携の考え方

アプリケーションを共有するのではなく、データを共有する。

共通して使用するもの：

・Googleスプレッドシート
・単語リスト
・URL台帳
・Google Apps Script
・Google Drive画像
・画像ID

Study DBは共通データを利用し、学習方向の選択後はURL台帳に登録されたURLへジャンプする。

全体動作
Study DB
   ↓
lessonパラメータで課を指定
   ↓
指定された課の単語一覧を表示
   ↓
学習方向モーダル
   ↓
日本語 → タイ語
   │
   └→ URL台帳B列
          ↓
       単語道場

または

   ↓
タイ語 → 日本語
   │
   └→ URL台帳C列
          ↓
       単語道場-Reverse
検索専用モード
lesson=0
   ↓
単語一覧は初期表示しない
   ↓
検索ボックス
   ↓
検索
   ↓
検索結果を表示

lesson=0の検索専用動作は、課別表示とは別の用途として維持する。

独立スプレッドシート

Study DB専用の独立スプレッドシートは使用しない。

単語道場側の既存スプレッドシート、

「単語道場（Tango Dojo）日本語/タイ語」

を共通データ基盤として使用する。

現在の仕様

Study DBは、

・lessonパラメータによる課指定
・lesson=0による検索専用モード
・課別単語一覧表示
・学習方向モーダル
・設定アイコンからの学習方向モーダル表示
・日本語 → タイ語の単語道場へのジャンプ
・タイ語 → 日本語の単語道場-Reverseへのジャンプ
・GASによる共通単語データ取得
・URL台帳による課別URL管理

を行う。

Study DB・単語道場・単語道場-Reverseは独立したアプリケーションとして維持する。

# Study DB_その他

## プロジェクト

Study DB_2026.8.24

## 情報の種類

その他

## 検索キーワード

Study DB、study-db、その他

## 保存場所

GitHub／GAS

## 内容

Study DBは、単語道場・単語道場-Reverseとは独立したWebアプリケーションとして動作する。

Study DB専用の単語データベースを別に持つのではなく、単語道場側のGoogleスプレッドシートにある共通データをGoogle Apps Script経由で取得して使用する。

---

## 使用するGoogleスプレッドシート

スプレッドシート：

「単語道場（Tango Dojo）日本語/タイ語」

Study DBが使用する主なシート：

・単語リスト
・URL台帳

---

## 単語リストとの関係

「単語リスト」は、単語道場・単語道場-Reverse・Study DBで共通して使用する単語データ。

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

Study DBはGoogle Apps Scriptからこのデータを取得する。

Study DB専用の単語リストは作成しない。

---

## URL台帳との関係

「URL台帳」は、教材・課ごとの各アプリケーションのURLを管理する。

### URL台帳の列構成

A列：課No  
B列：JP→TH URL  
C列：TH→JP URL  
D列：Study DB URL  
E列：教材  
F列：課名  

---

## Study DB URL

URL台帳D列でStudy DBの課別URLを管理する。

例：

https://kira-nihongo-online.github.io/study-db/?lesson=3

lessonパラメータによってStudy DBで表示する課を指定する。

---

## Study DBの課判定

Study DBはURLのlessonパラメータから課Noを取得する。

例：

```text
https://kira-nihongo-online.github.io/study-db/?lesson=1
↓

lesson = 1

↓

第1課の単語データを表示する。

課が変われば、同じ仕組みで対応する課のデータを表示する。

Google Apps Scriptとの関係

Study DBはGoogle Apps ScriptのWeb APIからデータを取得する。

GASは、

・単語リスト
・URL台帳

を読み込む。

単語リストの課NoとURL台帳の課Noを対応させ、単語データと課ごとのURLをStudy DBへ返す。

URLデータとして、

・jpThUrl
・thJpUrl
・studyDbUrl

を使用する。

学習方向モーダル

Study DBでは、現在開いている課に対応した学習方向モーダルを使用する。

例：

第1課を学習

日本語 → タイ語

タイ語 → 日本語

キャンセル

課が変われば、表示する課名も自動的に変わる。

例：

第1課を学習
第2課を学習
第3課を学習

のように対応する。

設定アイコン

Study DBのヘッダーには設定アイコンを表示する。

設定アイコンを押すと、現在の課の学習方向モーダルを再表示する。

Study DBに単語道場の学習機能を組み込むものではない。

学習方向によるURL連携
日本語 → タイ語

Study DBで、

「日本語 → タイ語」

を選択する。

↓

現在の課Noを確認。

↓

GASから取得したURL台帳データから、同じ課Noの jpThUrl を取得。

↓

URL台帳B列のURLへジャンプ。

↓

単語道場を開く。

タイ語 → 日本語

Study DBで、

「タイ語 → 日本語」

を選択する。

↓

現在の課Noを確認。

↓

GASから取得したURL台帳データから、同じ課Noの thJpUrl を取得。

↓

URL台帳C列のURLへジャンプ。

↓

単語道場-Reverseを開く。

URLの役割
B列

単語道場

日本語 → タイ語

https://kira-nihongo-online.github.io/tango-dojo/?mode=jp&lesson=課No
C列

単語道場-Reverse

タイ語 → 日本語

https://kira-nihongo-online.github.io/tango-dojo-reverse/?mode=thjp&lesson=課No
D列

Study DB

https://kira-nihongo-online.github.io/study-db/?lesson=課No
アプリケーション構成
Study DB

独立したWebアプリケーション。

役割：

・課ごとの単語データを表示する
・学習方向モーダルを表示する
・選択された学習方向に応じてURLへジャンプする

単語道場

独立したWebアプリケーション。

学習方向：

日本語 → タイ語

単語道場-Reverse

独立したWebアプリケーション。

学習方向：

タイ語 → 日本語

アプリケーション間の関係

3つのアプリケーションは一本化しない。

Study DB
   ↓
学習方向モーダル
   ↓
   ├── 日本語 → タイ語
   │       ↓
   │   URL台帳B列
   │       ↓
   │   単語道場
   │
   └── タイ語 → 日本語
           ↓
       URL台帳C列
           ↓
       単語道場-Reverse

Study DBに単語道場または単語道場-Reverseの学習機能を組み込まない。

Study DBはURLによって独立したアプリケーションを呼び出す。

データ共有の考え方

アプリケーションを共有するのではなく、データを共有する。

共通データ：

・単語リスト
・画像ID
・Google Drive上の画像
・URL台帳

単語道場と単語道場-Reverseは同じ単語データを使用する。

Study DBも同じ単語データをGAS経由で取得する。

完成した連携構成

現在の構成は、

Googleスプレッドシート
├─ 単語リスト
└─ URL台帳
       ↓
Google Apps Script
       ↓
Study DB
       ↓
学習方向モーダル
       ↓
   ┌──────────────┐
   │              │
   ↓              ↓
B列 JP→TH       C列 TH→JP
   ↓              ↓
単語道場       単語道場-Reverse

となっている。

これにより、これまで個別に存在していたStudy DB・単語道場・単語道場-Reverseを、アプリケーション自体は独立したまま、課と学習方向に応じて相互に移動できる構成になっている。

運用方針

単語データの追加・編集は共通の「単語リスト」を基準として行う。

単語道場と単語道場-Reverseで別々の単語データを作成しない。

Study DBでも別の単語データを作成しない。

課ごとのURLは「URL台帳」で管理する。

Study DBから単語道場・単語道場-Reverseへ移動する場合は、URL台帳のB列またはC列を使用する。

URLの変更が必要になった場合は、URL台帳を更新する。

現在の完成状態

Study DBの課別表示、学習方向モーダル、設定アイコンからのモーダル再表示、URL台帳との連携、単語道場・単語道場-Reverseへのジャンプまで完成している。

Study DB・単語道場・単語道場-Reverseは、それぞれ独立したアプリケーションとして維持されている。


# Study DB_設計

## プロジェクト

Study DB_2026.8.24

## 情報の種類

設計

## 検索キーワード

Study DB、study-db、設計

## 保存場所

GitHub／GAS

## 内容

Study DBは、単語道場および単語道場-Reverseと連携して教材・課単位で学習データを利用する独立したWebアプリとして構成する。

Study DB・単語道場・単語道場-Reverseは、それぞれ独立したアプリケーションとして維持する。

アプリケーションを一本化するのではなく、Googleスプレッドシート・Google Apps Script・URL台帳を共通のデータ基盤として連携する。

---

## 基本構成

単語道場側で管理されている共通データを基盤とし、教材・課ごとにStudy DBへアクセスできる構成とする。

単語道場のGoogleスプレッドシートには、

・単語リスト
・URL台帳

を配置する。

「単語リスト」は単語データの共通管理元として使用する。

「URL台帳」は各アプリケーションの課別URLを管理する。

---

## 単語データの管理

単語データは、単語道場側の「単語リスト」を共通データとして利用する。

Study DB専用の単語データベースは作成しない。

単語リストには、

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

を保持する。

Study DBはGoogle Apps Scriptを経由してこのデータを取得する。

---

## URL管理

単語道場側の「URL台帳」に、教材・課ごとのURLを登録する。

URL台帳の構成：

・A列：課No
・B列：JP→TH URL
・C列：TH→JP URL
・D列：Study DB URL
・E列：教材
・F列：課名

Study DB URLはD列で管理する。

Study DBへアクセスするときは、lessonパラメータによって対象となる課を指定する。

例：

https://kira-nihongo-online.github.io/study-db/?lesson=1

---

## Google Apps Script

Google Apps Scriptを、単語データとURL台帳をStudy DBへ渡す連携基盤として使用する。

GASは、

・単語リスト
・URL台帳

を読み込む。

単語リストの課NoとURL台帳の課Noを照合し、課Noに対応するURLを単語データとともに返す。

URLデータとして、

・jpThUrl
・thJpUrl
・studyDbUrl

を返す。

これにより、Study DB側で現在の課Noに対応する学習URLを利用できる。

---

## 課の判定

Study DBはURLのlessonパラメータから課Noを取得する。

例：

https://kira-nihongo-online.github.io/study-db/?lesson=1

↓

lesson = 1

↓

単語リストから課Noが1のデータを表示する。

課Noが変われば、同じ仕組みで別の課に対応する。

課ごとに別のStudy DBアプリを作成する必要はない。

---

## 学習方向モーダル

Study DBでは、現在開いている課に対応する学習方向モーダルを表示する。

例：

第1課を開いた場合：

第1課を学習

・日本語 → タイ語
・タイ語 → 日本語
・キャンセル

課が変われば、モーダルの課名も自動的に対応する。

例：

第2課の場合：

第2課を学習

第3課の場合：

第3課を学習

同じモーダルを全課で使用する。

---

## 設定アイコン

Study DBのヘッダーに設定アイコンを配置する。

設定アイコンを押すと、現在の課の学習方向モーダルを再表示する。

設定専用の別モーダルは使用しない。

設定アイコンと学習方向モーダルを接続し、同じ学習方向選択UIを使用する。

---

## 学習方向によるアプリケーション連携

### 日本語 → タイ語

Study DBで「日本語 → タイ語」を選択する。

現在の課Noを基準にGASから取得したURLを参照する。

URL台帳B列のJP→TH URLへジャンプする。

ジャンプ先：

単語道場

学習方向：

日本語 → タイ語

URL形式：

https://kira-nihongo-online.github.io/tango-dojo/?mode=jp&lesson=課No

---

### タイ語 → 日本語

Study DBで「タイ語 → 日本語」を選択する。

現在の課Noを基準にGASから取得したURLを参照する。

URL台帳C列のTH→JP URLへジャンプする。

ジャンプ先：

単語道場-Reverse

学習方向：

タイ語 → 日本語

URL形式：

https://kira-nihongo-online.github.io/tango-dojo-reverse/?mode=thjp&lesson=課No

---

## アプリケーションの独立性

以下の3つは独立したアプリケーションとして維持する。

・Study DB
・単語道場
・単語道場-Reverse

Study DBに単語道場の学習機能を組み込まない。

単語道場と単語道場-Reverseを一つのアプリケーションに統合しない。

各アプリケーションは独立したGitHubリポジトリ・Webアプリとして動作する。

---

## データ共有の考え方

アプリケーションを共有するのではなく、データ基盤を共有する。

共有するもの：

・単語リスト
・画像ID
・Google Drive上の画像
・URL台帳
・Google Apps Scriptによるデータ取得

単語道場と単語道場-Reverseは同じ単語データを使用する。

Study DBも同じ単語データをGAS経由で使用する。

---

## 全体設計

```text
Googleスプレッドシート
│
├─ 単語リスト
│      │
│      └─ 単語データ
│
└─ URL台帳
       │
       ├─ B列：JP→TH
       ├─ C列：TH→JP
       └─ D列：Study DB
              │
              ↓
      Google Apps Script
              │
              ├──────────────┐
              ↓              ↓
          Study DB        単語データ
              │
              ↓
        学習方向モーダル
          │          │
          ↓          ↓
       B列URL      C列URL
          ↓          ↓
      単語道場    Reverse

設計上の重要事項

Study DBを入口として各課の学習方向を選択できる構成とする。

Study DBから直接単語道場の機能を実行するのではなく、URL台帳に登録されたURLへジャンプする。

通常版とReverse版のURLは分離して管理する。

単語データは共通の「単語リスト」を使用する。

課Noを共通キーとして、単語データとURLを対応させる。

Study DB URLはURL台帳D列で管理する。

単語道場URLはURL台帳B列で管理する。

単語道場-Reverse URLはURL台帳C列で管理する。

運用方針

単語データの追加・編集は共通の「単語リスト」を基準として行う。

URLの追加・変更は「URL台帳」を基準として行う。

Study DB側に単語データや学習URLを個別に重複して保持しない。

URL台帳のB列・C列を変更することで、Study DBからジャンプする先を管理できる。

アプリケーション側の機能を共有するのではなく、共通データとURLを介して連携する。

現在の完成状態

Study DBは、

・課別の単語データ表示
・課に対応した学習方向モーダル
・設定アイコンからの学習方向モーダル再表示
・GASによる単語データ取得
・GASによるURL台帳データ取得
・課NoによるURL対応
・日本語 → タイ語の単語道場へのジャンプ
・タイ語 → 日本語の単語道場-Reverseへのジャンプ

まで完成している。

Study DB・単語道場・単語道場-Reverseは独立したアプリケーションのまま、共通データとURLを介して一つの学習システムとして連携する構成になっている。

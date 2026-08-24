# 単語道場_単語道場-reverse_スプレッドシート構成

## プロジェクト

単語道場_単語道場-reverse_2026.8.24

## 情報の種類

その他

## 検索キーワード

単語道場、単語道場-reverse、tango-dojo、tango-dojo-reverseスプレッドシート構成

## 保存場所

GitHub／GAS

## 内容

# 単語道場_単語道場-reverse_スプレッドシート構成

## 基本方針

このスプレッドシートは、

・単語道場（日本語 → タイ語）
・単語道場-Reverse（タイ語 → 日本語）
・Study DB

のデータ連携に使用する。

単語道場と単語道場-Reverseは、同じGoogleスプレッドシートの「単語リスト」データを共有して使用する。

Study DBも同じ「単語リスト」データをGAS経由で取得して使用する。

また、3つのアプリケーションは同じ「URL台帳」を利用して、各課のアクセス先を共有する。

ただし、3つのアプリケーションは一本化しない。

それぞれ独立したWebアプリケーションとして維持する。

共通するのは、Googleスプレッドシート、Google Apps Script、Google Drive、URL台帳などのデータ基盤である。

---

## スプレッドシート

単語道場（Tango Dojo）日本語/タイ語

---

## シート構成

確認できているシート：

・単語リスト
・修正-
・URL台帳

※「修正-」シートの内容は未確認。

---

# 【単語リスト】

単語データを管理するメインシート。

## 列構成

A列：教材  
B列：課No  
C列：課名  
D列：No  
E列：ひらがな  
F列：漢字  
G列：タイ語  
H列：英語  
I列：日本語画像  
J列：意味画像  
K列：メモ  

## データ例

教材：

みんなの日本語①

課No：

1

課名：

第1課

No：

1、2、3……

ひらがな：

わたし、あなた、あのひと……

漢字：

必要に応じて入力

タイ語：

各単語のタイ語訳

英語：

各単語の英語訳

日本語画像：

Google Driveの画像ファイルID

意味画像：

Google Driveの意味画像ファイルID

メモ：

補足情報

---

## シート上の操作

右側に以下の操作ボタンがある。

・CSV作成
・日本語画像
・意味画像

「日本語画像」および「意味画像」は、GASによってGoogle Drive上の画像ファイルとデータを照合し、画像IDを入力する処理に関連する。

---

## データ共有

「単語リスト」のデータは、

・単語道場
・単語道場-Reverse
・Study DB

で共有して使用する。

日本語画像ID・意味画像IDも共通で使用する。

Reverse版のために別の単語リストを作成する必要はない。

Study DBのために別の単語データを作成する必要もない。

---

# 【URL台帳】

教材・課ごとのURLを管理するシート。

## 列構成

A列：課No  
B列：JP→TH URL  
C列：TH→JP URL  
D列：Study DB URL  
E列：教材  
F列：課名  

---

## A列：課No

各URLを対応させるための課番号。

例：

1  
2  
3  
4  
……

単語リストのB列「課No」と対応する。

---

## B列：JP→TH URL

単語道場の教材ページURLを課Noごとに管理する。

学習方向：

日本語 → タイ語

URL形式：

https://kira-nihongo-online.github.io/tango-dojo/?mode=jp&lesson=課No

例：

https://kira-nihongo-online.github.io/tango-dojo/?mode=jp&lesson=1

---

## C列：TH→JP URL

単語道場-Reverseの教材ページURLを課Noごとに管理する。

学習方向：

タイ語 → 日本語

URL形式：

https://kira-nihongo-online.github.io/tango-dojo-reverse/?mode=thjp&lesson=課No

例：

https://kira-nihongo-online.github.io/tango-dojo-reverse/?mode=thjp&lesson=1

単語道場-Reverseは単語道場とは別の独立したWebアプリケーションとして動作する。

---

## D列：Study DB URL

Study DB側の教材ページURLを課Noごとに管理する。

URLにはlessonパラメータを使用する。

例：

https://kira-nihongo-online.github.io/study-db/?lesson=3

Study DBから単語道場または単語道場-Reverseへ移動した後、「戻る」を選択した場合も、このD列のURLを使用する。

現在の課Noに対応するD列のStudy DB URLへ戻る。

そのため、

・単語道場 第1課 → Study DB 第1課
・単語道場 第2課 → Study DB 第2課
・単語道場-Reverse 第1課 → Study DB 第1課
・単語道場-Reverse 第2課 → Study DB 第2課

のように、通常版・Reverse版のどちらからでも同じ課のStudy DBへ戻ることができる。

---

## E列：教材

課Noに対応する教材名を管理する。

例：

みんなの日本語①

---

## F列：課名

課Noに対応する課名を管理する。

例：

第1課  
第2課  
第3課  

---

## URL台帳の役割

URL台帳は、各課のアクセス先を一元管理する。

Study DBから学習方向を選択した場合、

・日本語 → タイ語
  → B列のURL

・タイ語 → 日本語
  → C列のURL

を使用する。

D列はStudy DB自体の課別URLを管理する。

また、単語道場・単語道場-ReverseからStudy DBへ戻る場合にも、D列のURLを使用する。

つまり、

B列：Study DB → 単語道場

C列：Study DB → 単語道場-Reverse

D列：単語道場／Reverse → Study DB

という双方向のURL連携を管理する。

---

# 【Google Apps Scriptとの関係】

「単語リスト」は単語データの共通メインデータとして使用する。

「URL台帳」は課Noと各アプリケーションのURLを対応させる連携データとして使用する。

GASは、

・単語リスト
・URL台帳

の両方を読み取る。

---

## GASが単語リストから取得するデータ

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

---

## GASがURL台帳から取得するデータ

・課No
・JP→TH URL
・TH→JP URL
・Study DB URL

---

## URL対応

GASはURL台帳のA列「課No」を基準としてURLを管理する。

単語リストの課NoとURL台帳の課Noを照合する。

例えば、

単語リスト：

課No = 1

↓

URL台帳：

A列 = 1

↓

B列：

日本語 → タイ語

↓

C列：

タイ語 → 日本語

↓

D列：

Study DB

という対応になる。

---

## GASから返すURLデータ

単語データと一緒に、課Noに対応するURLを返す。

返却するURLデータ：

・jpThUrl
・thJpUrl
・studyDbUrl

単語道場・単語道場-Reverseは、現在の課に対応する `studyDbUrl` を使用してStudy DBへ戻ることができる。

Study DBは、`jpThUrl` と `thJpUrl` を使用して学習方向に応じたURLへジャンプする。

---

# 【Google Driveとの関係】

日本語画像および意味画像はGoogle Driveで管理する。

Google Drive上の画像ファイルIDを「単語リスト」の、

・日本語画像
・意味画像

に保存する。

GASがファイル名と「課No」「No」等を照合して画像IDを自動入力する。

---

## 画像データ共有

日本語画像・意味画像は、単語道場と単語道場-Reverseの両方で共有する。

同じ単語に対して別々の画像を用意する必要はない。

Study DBも共通の画像IDを含む単語データをGAS経由で取得する。

---

# 【データの全体関係】

## 単語データ

単語データ

↓

Googleスプレッドシート「単語リスト」

↓

Google Apps Script

↓

・単語道場
・単語道場-Reverse
・Study DB

---

## 画像データ

画像データ

↓

Google Drive

↓

Google Apps Script

↓

画像ID・画像URL

↓

・単語道場
・単語道場-Reverse
・Study DB

---

## URLデータ

URL管理

↓

Googleスプレッドシート「URL台帳」

↓

Google Apps Script

↓

課Noに対応するURL

↓

Study DB

↓

学習方向を選択

↓

URL台帳B列

または

URL台帳C列

↓

・単語道場
・単語道場-Reverse

---

## Study DBからの戻り

単語道場

または

単語道場-Reverse

↓

設定モーダル

↓

「戻る」

↓

現在の課No

↓

URL台帳D列

↓

同じ課のStudy DB

---

# 【アプリケーション構成】

## 単語道場

GitHubリポジトリ：

tango-dojo

学習方向：

日本語 → タイ語

---

## 単語道場-Reverse

GitHubリポジトリ：

tango-dojo-reverse

学習方向：

タイ語 → 日本語

---

## Study DB

GitHubリポジトリ：

study-db

Study DBは、課ごとの単語データを表示する。

課を開くと学習方向モーダルを表示する。

学習方向：

・日本語 → タイ語
・タイ語 → 日本語

を選択できる。

選択した方向に応じてURL台帳の対応URLへジャンプする。

---

# 【Study DBとの連携】

Study DBはURL台帳を直接管理するのではなく、GASから取得したURLデータを使用する。

## 課の判定

Study DBはURLのlessonパラメータによって課を判定する。

例：

https://kira-nihongo-online.github.io/study-db/?lesson=1

↓

lesson = 1

↓

第1課の単語データを表示する。

---

## 学習方向モーダル

Study DBでは、開いている課に対応して、

「第1課を学習」

「第2課を学習」

「第3課を学習」

のように課名を表示する。

課が変わっても同じモーダルを使用する。

---

## 学習方向選択

### 日本語 → タイ語

Study DBの「日本語 → タイ語」を選択する。

↓

現在の課Noを取得。

↓

URL台帳の同じ課Noを探す。

↓

B列のJP→TH URLを取得。

↓

単語道場へジャンプする。

---

### タイ語 → 日本語

Study DBの「タイ語 → 日本語」を選択する。

↓

現在の課Noを取得。

↓

URL台帳の同じ課Noを探す。

↓

C列のTH→JP URLを取得。

↓

単語道場-Reverseへジャンプする。

---

## 設定アイコン

Study DBの右上に設定アイコンを表示する。

設定アイコンを押すと、現在の課の学習方向モーダルを再表示する。

Study DBに別の学習機能を組み込むものではない。

---

# 【単語道場・単語道場-ReverseからStudy DBへ戻る】

単語道場と単語道場-Reverseの設定モーダルには「戻る」ボタンを配置する。

「戻る」を選択すると、現在の課Noに対応するURL台帳D列のStudy DB URLへ移動する。

現在の課Noを基準にするため、戻る先は必ず同じ課になる。

例：

単語道場

第1課

↓

戻る

↓

URL台帳D列・課No=1

↓

Study DB 第1課

---

単語道場-Reverse

第1課

↓

戻る

↓

URL台帳D列・課No=1

↓

Study DB 第1課

---

# 【アプリケーション間の関係】

単語道場と単語道場-Reverseは、アプリケーションとして独立している。

Study DBも独立したアプリケーションとして維持する。

3つを一つのアプリケーションに統合しない。

共通するのは、

・Googleスプレッドシート
・単語リスト
・URL台帳
・Google Apps Script
・Google Drive上の画像
・画像ID

などのデータ基盤である。

---

# 【重要な設計】

「アプリケーションを共有する」のではなく、

「データとURLを共有する」。

単語道場と単語道場-Reverseは、同じ単語リストを使用する。

Study DBも同じ単語リストをGAS経由で使用する。

3つのアプリケーションは同じURL台帳を利用する。

Study DBはURL台帳の情報を利用して、独立した単語道場または単語道場-Reverseへ移動する。

単語道場・単語道場-Reverseは、URL台帳D列を利用して同じ課のStudy DBへ戻る。

Study DBに単語道場の学習機能そのものを組み込まない。

---

# 【運用方針】

単語データの追加・編集は共通の「単語リスト」を基準として行う。

通常版とReverse版それぞれに同じデータを入力する必要はない。

単語道場-Reverseは、共通データを使用しながら、アプリケーション側で表示方向・音声方向を反転して使用する。

Study DBも共通の単語データを使用する。

URL台帳では、通常版とReverse版の各課URLを別々に管理する。

Study DBから各課の学習方向を選択する場合は、URL台帳のB列またはC列に登録されたURLを使用する。

単語道場または単語道場-ReverseからStudy DBへ戻る場合は、URL台帳D列の現在の課に対応するURLを使用する。

URL台帳を変更することで、各課からジャンプする先のURLを管理できる。

---

# 【現在の完成状態】

単語道場、単語道場-Reverse、Study DBの3アプリケーションは、それぞれ独立した状態を維持している。

共通のGoogleスプレッドシートをデータ基盤として使用し、

・単語リスト
・URL台帳

をGASから取得できる構成になっている。

Study DBでは、課ごとの学習方向モーダルから、

・日本語 → タイ語
・タイ語 → 日本語

を選択できる。

選択した学習方向に応じて、URL台帳のB列またはC列の該当課URLへジャンプする。

単語道場・単語道場-Reverseの設定モーダルには「戻る」を配置し、URL台帳D列の該当課URLを使用して、同じ課のStudy DBへ戻ることができる。

これにより、

Study DB
↓
単語道場／単語道場-Reverse
↓
同じ課のStudy DB

という往復のURL連携が成立している。

3つのアプリケーションを統合するのではなく、共通のスプレッドシート・GAS・URL台帳を介して一つの学習システムとして連携する構成になっている。

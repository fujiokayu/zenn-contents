---
title: "Recent proposals for Big MASVS Refactoring"
emoji: "☎️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Security, セキュリティ]
published: true
---

## はじめに

本稿では OWASP MASVS(Mobile Application Security Verification Standard) で今後予定している大規模なリファクタリングに関するプロポーザルについて紹介していきます。
MASVS や MSTG の概要については触れないので、ご興味のある方は各プロジェクトの公式リポジトリをご参照ください。

- [OWASP Mobile Application Security Verification Standard](https://github.com/OWASP/owasp-masvs)
- [OWASP Mobile Security Testing Guide](https://github.com/OWASP/owasp-mstg)

---

それでは、MASVS の GitHub Discussions で [big-masvs-refactoring カテゴリ](https://github.com/OWASP/owasp-masvs/discussions/categories/big-masvs-refactoring)に含まれる議題を古い順に紹介していきます。
※ 2022年現在、MASVS/MSTG に関する提案は OWASP Slack ではなく GitHub Discussions の利用を推奨されています。

## [Refactor of MASVS / MASVS v2 #553](https://github.com/OWASP/owasp-masvs/discussions/553)

- on 15 Jul 2021
- comment: closed

この Discussion は今後のリファクタリングについて大まかな方向性を示すものです。

### Current Problems

- 第 1 章（アーキテクチャ）には、外部からの視点で検証できない要件が複数含まれている。
  - その結果、外部評価で「MASVS をフルにカバーできない」理由を説明することが難しい。
- どのコントロールが自動化できるのか／ブラックボックスで検証できるのか／完全なソースコードレビューが必要なのかが明確でない
- バックエンドの様々な脆弱性に関して、より徹底している ASVS と重複している部分がある
- 一部のコントロールは非常に広範であり、複数の部分を含む非常に広範なテストケースを必要とする
- いくつかのコントロールは重複している

### Proposal

- ASVS との重複を排除し、モバイルに特化したコントロールにフォーカスする
- 具体的で実行可能な MSTG テストケースにリンクできるように、コントロールをより具体的にする
  - MSTG のリファクタリングと同時に行われる予定。特定の項目を 2 つの非常に異なる方法でテストする必要がある場合、それらを分割する必要があるという考えている
- 冗長性/重複を取り除く
- コントロールに明確な区分があることを確認する
  - 例えば、どのコントロールが自動で評価できるのか、どのコントロールが完全なソースコードレビューを必要とするのかが明確であるべき

### Consequences

- MASVS v2（現在 v1.2）に移行し、識別子を（MSTG-XX ではなく）MASVS-XX に更新することが望ましい
- この MASVS のリファクタリングと同時に、MASVS の各コントロールが明確に定義されたテストケースを持つように、MSTG のリファクタリングも並行して行う予定

### Next steps

- このリファクタリングはかなり大掛かりなもので、多くの人と一緒にやるのはおそらくカオスになる
- MASVS v2 の最初のドラフトを作成し、それを発表してコミュニティのフィードバックを伺いながら進めていく予定

### comments

コメントは割愛。
一点、

- [julepka on 20 Jul 2021](https://github.com/OWASP/owasp-masvs/discussions/553#discussioncomment-1023599)
  - 唯一の懸念は、ASVS（バックエンド）なしで MASVS（モバイルのみ）に基づく審査・監査を要求することが可能になることです。ASVS を無視したモバイルアプリは、セキュリティレベルが高くならないことを強調しておく必要があると思います。

というコメントは留意しておくべきと思いました。
それと、コメントの最後では MASVS & MSTG Refactoring の資料が共有されています。
<https://drive.google.com/file/d/18-SMIDaNFPKHK8V6wWQqvj0UTu4toteQ/view>

この資料では今後の大まかな方向性を提示しているので押さえておくと良いでしょう。Discussion では触れられていない、メンテナンスの自動化などについても言及されています。

![](https://storage.googleapis.com/zenn-user-upload/3aa36b4f9fac-20220326.png)

## [MASVS-NETWORK Refactoring (till 31.12.21) #559](https://github.com/OWASP/owasp-masvs/discussions/559)

- on 3 Sep 2021
- comment: closed

こちらも既存項目の説明や変更点の詳細は省きます。

- Note:
  - 要件は MSTG ではなく MASVS に起因しているため、ID の命名法は "MSTG--ID "から "MASVS--ID "に変更される予定
  - 既存の要件の MASVS-ID は、リファクタリングプロセスの一環として変更される可能性がある

# | MASVS-ID | Description | L1 | L2
-- | -- | -- | -- | --
5.1 | MASVS-NETWORK-1 | All HTTP requests in transit between the app and endpoints are encrypted. | x | x
5.2 | MASVS-NETWORK-2 | All Non-HTTP requests in transit between the app and endpoints are encrypted. | x | x
5.3 | MASVS-NETWORK-3 | The network security settings in the app are in line with current best practices. | x | x
5.4 | MASVS-NETWORK-4 | The app verifies the X.509 certificate of the remote endpoint(s) when the secure channel is established. Only certificates signed by a trusted Certificate Authority (CA) that are part of the system storage (or "host platform"?) are accepted. | x | x
5.5 | MASVS-NETWORK-5 | The app either uses its own certificate store, or pins the endpoint certificate or public key, and subsequently does not establish connections with endpoints that offer a different certificate or key, even if signed by a trusted CA. |   | x

- Mapping between MASVS v1.3 to MASVS v2.0:

MSTG-ID (MASVS v1.3) | MASVS-ID (MASVS v2.0) | Status | Change of ID Number
-- | -- | -- | --
MSTG-NETWORK-1 | MASVS-NETWORK-1 | Reworded | No
  | MASVS-NETWORK-2 | New |  
MSTG-NETWORK-2 | MASVS-NETWORK-3 | Reworded | Yes
MSTG-NETWORK-3 | MASVS-NETWORK-4 | Reworded | Yes
MSTG-NETWORK-4 | MASVS-NETWORK-5 | No change | Yes
MSTG-NETWORK-5 |   | Removed |  
MSTG-NETWORK-6 |   | Removed |  

以下は変更点の要約です。

- MSTG-NETWORK-1 データはネットワーク上で TLS を使用して暗号化されている。セキュアチャネルがアプリ全体を通して一貫して使用されている。
  - HTTP{S} と それ以外の通信についてチェック項目を分けた
- MSTG-NETWORK-2 TLS 設定は現在のベストプラクティスと一致している。モバイルオペレーティングシステムが推奨される標準規格をサポートしていない場合には可能な限り近い状態である。
  - TLS Settings という文言を network security settings に置き換え。
- MSTG-NETWORK-3 セキュアチャネルが確立されたときに、アプリはリモートエンドポイントの X.509 証明書を検証している。信頼された CA により署名された証明書のみが受け入れられている。
  - システムストレージで利用可能な CA に要件を限定する
- MSTG-NETWORK-4 アプリは自身の証明書ストアを使用するか、エンドポイント証明書もしくは公開鍵をピンニングしている。信頼された CA により署名された場合でも、別の証明書や鍵を提供するエンドポイントとの接続を確立していない。
  - ピン留めについては変更なし
- MSTG-NETWORK-5 アプリは登録やアカウントリカバリーなどの重要な操作において（電子メールや SMS などの）単方向のセキュアでない通信チャネルに依存していない。
  - ASVS 2.5.6 でカバーされるので Remove
- MSTG-NETWORK-6 アプリは最新の接続ライブラリとセキュリティライブラリにのみ依存している。
  - MSTG-CODE-5 でカバーされるため Remove

## [MASVS-CRYPTO Refactoring (till 31.01.22) #612](https://github.com/OWASP/owasp-masvs/discussions/612)

- on 29 Dec 2021
- comment: closed

こちらも既存項目の説明や変更点の詳細は省きます。
提案内容はスプレッドシートで共有されています。
<https://docs.google.com/spreadsheets/d/1oyVMntRLaLHMWNfQqrWk3e0Q_HUI0Yo1opRB_hyYrpY/edit#gid=0>

以下は従来の要件の変更について

- MSTG-CRYPTO-1 アプリは暗号化の唯一の方法としてハードコードされた鍵による対称暗号化に依存していない。
  - ハードコードされた鍵の問題は、それ自体が安全に保管されず（MASVS-STORAGE-2）、暗号化するものを保護できていない(MASVS-STORAGE-1) という 2 つの失敗であると言える。
  - ハードコードされた鍵の使用は「暗号のバッドプラクティス」とみなされるため、新設される MASVS-CRYPTO-2 の対象となる。
  - よってこの項目は Remove
- MSTG-CRYPTO-2 アプリは実績のある暗号化プリミティブの実装を使用している。
  - "非推奨"であることが証明されても、それを使ってはいけないという要件を分離させることに価値はない。
    - 例：あるアプリが SHA-1 を使っていて、3.2 は通るが 3.4 は通らないというようなケース
    - 執筆者補足：3.4 は「アプリはセキュリティ上の目的で広く非推奨と考えられる暗号プロトコルやアルゴリズムを使用していない。」。SHA1 は実績があるので 3.2 を Pass で通過する
- MSTG-CRYPTO-3 アプリは特定のユースケースに適した暗号化プリミティブを使用している。業界のベストプラクティスに基づくパラメータで構成されている。
  - 新設される MASVS-CRYPTO-2 でカバーすることを検討
- MSTG-CRYPTO-4 アプリはセキュリティ上の目的で広く非推奨と考えられる暗号プロトコルやアルゴリズムを使用していない。
  - MSTG-CRYPTO-2 でカバーされるので Remove
- MSTG-CRYPTO-5 アプリは複数の目的のために同じ暗号化鍵を再利用していない。
  - 問題は鍵の再利用だけではないので変更する
- MSTG-CRYPTO-6 
  - 新設される MASVS-CRYPTO-2 でカバーするため Remove

以下は新しい要件

- MASVS-CRYPTO-1 The app uses platform provided cryptographic implementations, unless independently security vetted.
  - アプリは独立したセキュリティ検証を受けない限り、プラットフォームが提供する暗号化機能を使用している
    - カスタムメイドの暗号を使用しない
    - プラットフォームが提供する暗号 API を優先する（e.g. Apple CryptoKit, Android recommendations）。
    - 可能であれば、暗号化操作は安全なハードウェアの内部で実行する(e.g. iOS SE)。
    - それ以外の場合は、十分にテストされ、吟味されたライブラリの使用を検討すること。
      - 例：wolfSSL, boringSSL, openSSL, NaCl(Threema)
    - アプリが非標準、証明されていない、または許可されていない暗号化実装を使用して暗号化アルゴリズムを実装している場合は Fail。(CWE-1240)
- MASVS-CRYPTO-2 The app employs current strong cryptography and uses it according to industry best practices.
  - アプリは昨今の強力な暗号を採用し、業界のベストプラクティスに従ってそれを使用している
    - 1) 以下のような現行の強力な暗号（NIST/FIPS-140 認証など）のみを採用する
      - primitives (digital signatures, one-way hash functions, ciphers, public key cryptography, etc.)
      - protocols (TLS, SSH, IPsec, Diffie-Hellman, etc.)
      - algorithms (AES, RSA, SHA, etc.)
    - 2) 常に業界やプラットフォームのベストプラクティス（NIST、BSI に準拠した鍵長など）に従って使用している
    - 1)と2)が成立している場合のみ Pass とする
    - もちろん、セキュリティに関連しないコード（UUID/GUID など）が非推奨/弱い暗号を使用しても構わない
- MASVS-CRYPTO-3 The app performs key management according to industry best practices.
  - アプリは業界のベストプラクティスに従ってキーマネージメントを行なっている
    - 暗号鍵を扱うアプリは、以下に準拠する必要がある
      - 鍵の再利用禁止
      - 鍵のローテーション
      - 鍵の認証
      - etc.(詳しくはテストケースを参照)
- MASVS-CRYPTO-4 All cryptographic material must be stored in a tamper-resistant environment.
  - 暗号に関連するすべての要素は、耐タンパー性のある環境で保管されなければならない
    - 暗号鍵を扱う高リスクのアプリのみ準拠が必要
    - 適用レベルは R

## [MASVS-PLATFORM Refactoring (till 09.04.22) #635](https://github.com/OWASP/owasp-masvs/discussions/635)

- on 10 Mar 2022
- comment: Open

スプレッドシートは以下。
<https://docs.google.com/spreadsheets/d/1gRQAp2s6KYQA8KUDZpcEammMMCjxu3wVIQkAGO8ca2Q/edit#gid=0>

以下は従来の要件の変更について

- MSTG-PLATFORM-1 アプリは必要となる最低限のパーミッションのみを要求している。
  - Keep
- MSTG-PLATFORM-2 外部ソースおよびユーザーからの入力はすべて検証されており、必要に応じてサニタイズされている。これには UI、インテントやカスタム URL などの IPC メカニズム、ネットワークソースを介して受信したデータを含んでいる。
  - MASVS-PLATFORM-2 (IPC, URL schemes)、MASVS-PLATFORM-5(UI)でカバーすることになる
- MSTG-PLATFORM-3 アプリはメカニズムが適切に保護されていない限り、カスタムURLスキームを介して機密な機能をエクスポートしていない。
  - MASVS-PLATFORM-2 でカバーすることになる
- MSTG-PLATFORM-4 アプリはメカニズムが適切に保護されていない限り、IPC機構を通じて機密な機能をエクスポートしていない。
  - MASVS-PLATFORM-5（MSTG-PLATFORM-12）でのテストになる
- MSTG-PLATFORM-5 明示的に必要でない限りWebViewでJavaScriptが無効化されている。
- MSTG-PLATFORM-6 WebViewは最低限必要のプロトコルハンドラのセットのみを許可するよう構成されている（理想的には、httpsのみがサポートされている）。file, tel, app-id などの潜在的に危険なハンドラは無効化されている。
- MSTG-PLATFORM-7 アプリのネイティブメソッドがWebViewに公開されている場合、WebViewはアプリパッケージ内に含まれるJavaScriptのみをレンダリングしている。
- MSTG-PLATFORM-10 WebViewを破棄する前にWebViewのキャッシュ、ストレージ、ロードされたリソース (JavaScript など) をクリアしている。
  - MASVS-PLATFORM-3 でカバーすることになる
- MSTG-PLATFORM-8 オブジェクトのデシリアライゼーションは、もしあれば、安全なシリアライゼーション API を使用して実装されている。
  - MASVS-CODE に移動する
- MSTG-PLATFORM-9 アプリはスクリーンオーバーレイ攻撃から自らを保護している。 (Android のみ)
- MSTG-PLATFORM-11 機密データを入力する場合は常に、アプリはカスタムサードパーティキーボードを使用できないようにしている。(iOS のみ)
  - MASVS-PLATFORM-5 でカバーすることになる

以下は新しい要件

- MASVS-PLATFORM-1 アプリは必要となる最低限のパーミッションのみを要求している。
  - MSTG-PLATFORM-1 から変わらず
- MASVS-PLATFORM-2 No sensitive data is exposed via unprotected IPC mechanisms.
  - 機密データは保護されていない IPC メカニズムを介して公開されていない。
    - すべての可能な IPC 方式（URL からアプリ、アプリからアプリ、アプリ内グループ、OS からアプリ）を対象とする
- MASVS-PLATFORM-3 WebViews are configured securely and prevent sensitive functionality exposure.
  - WebView はセキュアに設定され、機密性が高い機能の露出を防いでいる
- MASVS-PLATFORM-4 No sensitive data is included in platform backups.
  - 機密データはプラットフォームが生成するバックアップに含まれていない
    - アプリは独自のバックアップを作成することもできるが、それは MASVS-STORAGE-1 でカバーされており、そのための特定のテストがある
- MASVS-PLATFORM-5 No sensitive data is unnecessarily exposed nor can be intercepted via the user interface.
  - 機密情報は不必要に露出しておらず、ユーザーインターフェースを通じて傍受されることもない
    - サードパーティのキーボード、スクリーンショット、クリップボード、スクリーンオーバーレイ攻撃などによる機密データの傍受/漏洩に対処する

## [MASVS-STORAGE Refactoring (till 15.03.22) #627](https://github.com/OWASP/owasp-masvs/discussions/627)

- on 15 Mar 2022
- comment: Closed

スプレッドシートは以下。
<https://docs.google.com/spreadsheets/d/1-mddga8DtFu5Ef9vcyPNrgaFMx44Bv5AcPv4ZqFtTHM/edit#gid=0>

以下は従来の要件の変更について

- MSTG-STORAGE-1 個人識別情報、ユーザー資格情報、暗号化鍵などの機密データを格納するために、システムの資格情報保存機能を使用している。
  - Android KeyStore と iOS KeyChain を指す"platform keystore""という用語を使っていく
- MSTG-STORAGE-2 機密データはアプリコンテナまたはシステムの資格情報保存機能の外部に保存されていない。
  - MSTG-STORAGE-1 でカバーし、暗号鍵に焦点を当ててコントロールする
- MSTG-STORAGE-3 機密データはアプリケーションログに書き込まれていない。
  - application という言葉を削除し、他のログ（例えばシステムログ）を含める
- MSTG-STORAGE-4 機密データはアーキテクチャに必要な部分でない限りサードパーティと共有されていない。
  - データプライバシーに関する MASVS-STORAGE-5（MSTG-STORAGE-12）でのテストに含む
- MSTG-STORAGE-5 機密データを処理するテキスト入力では、キーボードキャッシュが無効にされている。
- MSTG-STORAGE-6 機密データは IPC メカニズムを介して公開されていない。
- MSTG-STORAGE-7 パスワードやピンなどの機密データは、ユーザーインタフェースを介して公開されていない。
- MSTG-STORAGE-8 機密データはモバイルオペレーティングシステムにより生成されるバックアップに含まれていない。
- MSTG-STORAGE-9 バックグラウンドへ移動した際にアプリはビューから機密データを削除している。
  - MASVS-PLATFORM の範囲に含む
- MSTG-STORAGE-10 アプリは必要以上に長くメモリ内に機密データを保持せず、使用後は明示的にメモリがクリアされている。
  - 付加価値のない「明示的に（explicitly）」という言葉を削除した。MSTG ではメモリがクリアされることの意味を説明し、その実装時やテスト時の課題について詳しく解説している
- MSTG-STORAGE-11 アプリは最低限のデバイスアクセスセキュリティポリシーを適用しており、ユーザーにデバイスパスコードを設定することなどを必要としている。
  - MASVS-STORAGE-1 でのテストに含める
  - パスコードが設定されていない場合、MASVS-STORAGE-1 はデータを十分に保護しながら暗号化することができないため、Fail となるはず
- MSTG-STORAGE-12 アプリは処理される個人識別情報の種類をユーザーに通知しており、同様にユーザーがアプリを使用する際に従うべきセキュリティのベストプラクティスについて通知している。
  - よりシンプルに、より抽象度を上げるために文言を変える
- MSTG-STORAGE-13 機密データをモバイルデバイス上のローカルに保存していない。代わりに、必要な時にリモートエンドポイントからデータを取得し、メモリ内にのみ保持している。
  - アーキテクチャの問題なので、MASVS-ARCH に移動する
- MSTG-STORAGE-14 機密データをローカルに保存する必要がある場合には、認証が必要なハードウェア支援ストレージから取得した鍵を使用して暗号化している。
  - 他のコントロール（またはそれに対応するテスト）ですでにカバーされている 3 つの異なる側面がある
    - 1) ローカルに保存されたデータを（安全に保存された）鍵で暗号化する → MASVS-STORAGE-1 でカバーされる
    - 2) 鍵がハードウェアバックアップストレージから生成された場合→ MASVS-STORAGE-2 でカバーされる
    - 3) 鍵が認証を必要とすること：NIST.SP.800-175B に準拠した鍵管理に関する MASVS-CRYPTO-3 のテストとなる
- MSTG-STORAGE-15 認証の試行が過度の回数にわたり失敗した後には、アプリのローカルストレージを消去している。
  - MASVS-STORAGE-1 でカバー。データを適切に暗号化することで、アンインストール時や過度の認証失敗時に削除されなくても安全であるはず

以下は新しい要件

- MASVS-STORAGE-1 Sensitive data is stored in encrypted form.
  - 機密データは暗号化されて保存されている
    - この要件は Sensitive Data 全般に関するもので、保存場所に関係なく保護（暗号化）されなければならないということ
    - Sensitive Data とは以下を指す
      - ユーザーデータ、PII、位置情報、財務、健康、メディア、アプリ活動、ブラウジング/検索、デバイス ID、有料リソースの取引履歴、など
      - 認証データ：認証情報、トークン、パスフレーズ、PIN、セッション識別子など
      - その他のデータ：OTA アップデート
      - 暗号鍵（特殊なケース、MASVS-STORAGE-2でカバー）
- MASVS-STORAGE-2 Cryptographic keys are stored inside the platform keystore or using equivalent protection.
  - 暗号鍵はプラットフォームキーストア内に保存されるか、同等の保護を使用して保存されている
    - platform keystore: Android KeyStore と iOS KeyChain を指す
    - "同等の保護": これは例えば、エンベロープ暗号化やキーラッピング、 プラットフォームキーストアにある他の鍵を使って鍵を暗号化(KEK (Key Encryption Keys)しているケース
- MASVS-STORAGE-3 Sensitive data is never written to logs.
  - 機密データは決してログに書き込まれていない
    - ログの対象をアプリケーションログに限定しない
- MASVS-STORAGE-4 No sensitive data lives in memory longer than necessary, and is cleared after use.
  - アプリは必要以上に長くメモリ内に機密データを保持せず、使用後はクリアされている
    - フォーカスは変わらず、メモリ内の機密データ露出を防ぐこと
- MASVS-STORAGE-5 The app follows privacy best practices when processing sensitive user data.
  - アプリは機密性の高いユーザーデータを処理する際、プライバシーのベストプラクティスに従っている
    - プライバシーのテストは複雑なタスクであり、プラットフォーム（Android/iOS）は急速に改善されている
    - このコントロールはその迅速な進化をカバーするために十分な抽象化を提供し、詳細はより機敏な方法で成長する MSTG に委ねるべき

---

以上です。Big MASVS Refactoring の名の通り、非常に大規模なリファクタリングであることが伝わったでしょうか。
全体的に要件が明確になり簡素化もされているので、今後のアセスメントのコストが効率化され、また顧客にとっても理解されやすいものになったと思います。

なお、MASVS-PLATFORM はまだコメントを受け付けているので、ご意見のある方はぜひ Discussion に参加してください。

個人的には、V4 を丸々っと ASVS に委ねてしまうと Local Authentication に関する MSTG-AUTH-8 や MSTG-AUTH-12 を検証する機会がなくなってしまうのでは、という点が気になっています。
また、ASVS では PKCE([RFC7636](https://datatracker.ietf.org/doc/html/rfc7636)) に関する要件がないので、これも MASVS で取り扱う箇所があると良いはず。従来の MSTG だと Testing OAuth 2.0 Flows (MSTG-AUTH-1 and MSTG-AUTH-3)が該当。
この辺りは別途、メンテナ達と相談していきたいと思います。

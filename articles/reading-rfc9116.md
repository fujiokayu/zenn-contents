---
title: "RFC 9116 から読み解く正しい security.txt の書き方"
emoji: "🔏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [security, セキュリティ, web, rfc]
published: false
---

security.txt は非常にシンプルな text/plain ファイルであり、既成のものをコピペして `.well-known/security.txt` として配置するだけであれば、ものの数分で対応できることでしょう。
また、[自動で作成してくれる Web アプリ](https://securitytxt.org/)も公式で用意されています。

ですが、例えば「Preferred-Languages に優先順位はあるの？」、「攻撃者によって security.txt が改ざんされたらどうなるの？」といったことは気にならないでしょうか。
私はとても気になります。

というわけで、早速 RFC9116 を読んでいきましょう。

:::message
この文章は RFC 9116 の完全な翻訳を試みるものではなく、正しく security.txt を書くために必要な部分を抽出したものであり、用語定義などを記した Section 1 や IANA における位置付けを示す Section 6 などは割愛しています。
また、何度も出てくる以下のような記載は省略しています。
>値が Web URI を示す場合、それは必ず `https://` で始まらなければなりません([RFC7230 のセクション 2.7.2](https://www.rfc-editor.org/rfc/rfc7230#section-2.7.2) に従ってください)。

そのため、記法の意図や詳細な背景、最新の情報を知りたい場合は原文を参照してください。

- [RFC 9116 A File Format to Aid in Security Vulnerability Disclosure](https://www.rfc-editor.org/rfc/rfc9116#name-redirects)
:::

tl;dr は[こちら](#まとめ)

## Abstract

Section 1 は割愛すると書きましたが、概要にだけは最初に触れておきます。security.txt は第三者であるセキュリティリサーチャーのために書かれたものであり、この RFC は組織における security.txt の実装者とそれを参照する第三者であるセキュリティリサーチャーに向けて書かれています。（と、解釈しています）

> セキュリティリサーチャーによってセキュリティ脆弱性が発見された場合に、適切な報告ルートが欠如していることが多くあり、その結果、脆弱性が報告されずに放置されることがあります。
> この文書では、リサーチャーが脆弱性を報告しやすくするために、組織が脆弱性開示の慣行を記述するのに役立つ機械解析可能なフォーマット（`security.txt`）を定義します。

RFC 9116 の現在のステータス、正誤表、フィードバックを提供する方法に関する情報は[こちら](https://www.rfc-editor.org/info/rfc9116)から入手できます。

## 2 The Specification

security.txt の書式は機械で解析可能であり、必ず ABNF(Augmented Backus–Naur form) 記法に従う必要があります。

フィールドには必ず `name` を含み、`name` はフィールドの最初の部分からコロンまで(例: "Contact:")で、`value` はフィールド名の後に記載します。

- Example:
  - `mailto:security@example.com"`

[RFC5322 のセクション 3.2.5 ](https://www.rfc-editor.org/rfc/rfc5322#section-3.2.5)で "unstructured" について定義されている構文に従ってください。なお、ファイルには空行を含んでも構いません。

### 2.1 Comments

記号 "#" (`%x23`) で始まる行はすべてコメントとして解釈される必要があります。コメントの内容には、`%x21-7E` と `%x80-FFFFF` の範囲にある ASCII または Unicode 文字と、タブ文字 (`%x09`) とスペース文字 (`%x20`) を含めることができます。

- Example:
  - `# This is a comment.`

### 2.2 Line Separator

すべての行は carriage return と改行文字(CRLF / `%x0D%x0A`)、または単なる改行文字(LF / `%x0A`)のいずれかで終わる必要があります。

### 2.3 デジタル署名

security.txt ファイルには、[RFC4880のセクション 7 ](https://www.rfc-editor.org/rfc/rfc4880#section-7)で説明されているように、 OpenPGP の平文署名を使用してデジタル署名することが推奨されます。電子署名を使用する場合、組織は `Canonical` フィールドを使用することも推奨されます。

署名を生成するために使用される鍵の検証に関して、使用される鍵が本当に信頼できるものであることを確認するのは常にセキュリティリサーチャーの責任となります。

### 2.4 拡張性

他の多くのフォーマットやプロトコルと同様、このフォーマットも変化し続けるインターネットの状況に合わせて時間の経過とともに変更する必要があるかもしれません。そのため IANA レジストリによって拡張性が提供されており、このプロセスで登録されるフィールドはすべてオプションとみなさなければなりません。拡張性と相互運用性を促進するために、リサーチャーは明示的にサポートしていないフィールドは無視しなければいけません。

一般的に、実装者は「自分が行うことに関しては保守的に、他者から受け入れることに関してはリベラルに」なるべきです ([RFC0793](https://www.rfc-editor.org/rfc/rfc9116#RFC0793))。

### 2.5. Field Definitions

特に断りのない限り、すべてのフィールドはオプションとみなされなければなりません。

#### 2.5.1. Acknowledgments

`Acknowledgments` フィールドはセキュリティリサーチャーの報告が評価されるページへのリンクを示します。
参照されるページにはセキュリティ脆弱性を報告し、その脆弱性を改善するために協力したセキュリティリサーチャーを列挙してください。
組織は、将来の攻撃を防ぐために公表される脆弱性情報を制限するように注意すべきです。

- Example:
  - `Acknowledgments: https://example.com/hall-of-fame.html`

- Example security acknowledgments page:

```html
We would like to thank the following researchers:

(2017-04-15) Frank Denis - Reflected cross-site scripting
(2017-01-02) Alice Quinn  - SQL injection
(2016-12-24) John Buchner - Stored cross-site scripting
(2016-06-10) Anna Richmond - A server configuration issue
```

#### 2.5.2. Canonical

`Canonical` フィールドは security.txt ファイルがある正規の URI を示し、通常は `https://example.com/.well-known/security.txt` のようになります。

このフィールドは与えられた URI から取得された security.txt がその URI に適用されることを意図していることを示します。しかし、ファイル内にリストされているすべての正規 URI に適用されると解釈していけません。リサーチャーはデジタル署名のような付加的な信頼の仕組みを使用して、特定の正規 URI が適用可能であるという判断を下すべきです。

このフィールドが security.txt ファイル内に現れ、そのファイルを取得するために使用された URI がどの `canonical` フィールド内にもリストされていない場合、そのファイルのコンテンツは信用されるべきではありません。

```
Canonical: https://www.example.com/.well-known/security.txt
Canonical: https://someserver.example.com/.well-known/security.txt
```

#### 2.5.3. Contact

`Contact` フィールドは電子メールアドレス、電話番号、および／または連絡先情報を持つ Web Page のような、リサーチャーがセキュリティ脆弱性を報告するために使用すべき方法を示します。このフィールドは security.txt ファイルに常に存在しなければなりません。
この項目に記されるメールアドレスは、[RFC2142 のセクション 4](https://www.rfc-editor.org/rfc/rfc2142#section-4) で定義されている規範を使用するべきです。

値は [RFC3986 のセクション 3](https://www.rfc-editor.org/rfc/rfc3986#section-3) で述べられている URI 構文に従わなければなりません。
これは [RFC6068](https://www.rfc-editor.org/rfc/rfc9116#RFC6068) と [RFC3966](https://www.rfc-editor.org/rfc/rfc9116#RFC3966) で定義されているように、電子メールアドレスと電話番号を指定するときには `mailto` と `tel` URI スキーマを使用しなければならないことを意味します。このフィールドの値が電子メールアドレスの場合、暗号化を使用することが推奨されます。

これらは優先順位の高い順にリストされるべきであり、以下の例では、最初のメールアドレス `security@example.com` が最も優先される連絡方法です。

```
Contact: mailto:security@example.com
Contact: mailto:security%2Buri%2Bencoded@example.com
Contact: tel:+1-201-555-0123
Contact: https://example.com/security-contact.html
```

#### 2.5.4. Encryption

`Encryption` フィールドはセキュリティリサーチャーが暗号化通信に使用すべき暗号鍵を示します。鍵はこのフィールドに記述してはいけません。
代わりに、このフィールドの値は鍵を取得できる場所を指す URI である必要があります。

鍵の真正性を検証する場合、指定された鍵が本当に信頼できるものであることを確認するのは常にセキュリティリサーチャーの責任です。
リサーチャーは、この鍵がセクション 2.3 で言及されているデジタル署名を生成するために使用されていると仮定してはいけません。

- Example of an OpenPGP key available from a web server:
  - `Encryption: https://example.com/pgp-key.txt`
- Example of an OpenPGP key available from an OPENPGPKEY DNS record:
  - `Encryption: dns:5d2d37ab76d47d36._openpgpkey.example.com?type=OPENPGPKEY`
- Example of an OpenPGP key being referenced by its fingerprint:
  - `Encryption: openpgp4fpr:5f2de5521c63a801ab59ccb603d49de44b29100f`

#### 2.5.5. Expires

`Expires` フィールドは、security.txt ファイルに含まれるデータが古いとみなされ、使用されるべきではないとされる日時を示します。
このフィールドの値は [RFC3339](https://www.rfc-editor.org/rfc/rfc9116#RFC3339) で定義されている [ISO.8601-1](https://www.rfc-editor.org/rfc/rfc9116#ISO.8601-1) と [ISO.8601-2](https://www.rfc-editor.org/rfc/rfc9116#ISO.8601-2) のインターネットプロファイルに従ってフォーマットされます。

このフィールドの値は、陳腐化を避けるために、1 年以内の値であることが推奨されます。

このフィールドは常に存在しなければならず、複数回出現してはなりません。

`Expires: 2021-12-31T18:37:07z`

#### 2.5.6. Hiring

`Hiring` フィールドはベンダーのセキュリティ関連の求人情報へのリンクに使用されます。

`Hiring: https://example.com/jobs.html`

#### 2.5.7. Policy

"Policy" フィールドは脆弱性公開のポリシーがある場所へのリンクを示す。これはセキュリティリサーチャーが組織の脆弱性報告に対するポリシーを理解するのに役立ちます。

`Policy: https://example.com/disclosure-policy.html`

#### 2.5.8. Preferred-Languages

`Preferred-Languages` フィールドは、セキュリティレポート提出時に優先される自然言語のセットを示すために使用できます。
このセットはカンマで区切って複数の値を列挙してもよく、このフィールドが含まれる場合、少なくとも 1 つの値を列挙しなければなりません。
このセット内の値は言語タグである [RFC5646](https://www.rfc-editor.org/info/rfc5646) に準拠し、このフィールドがない場合、セキュリティリサーチャーは([RFC2277 のセクション 4.5](https://www.rfc-editor.org/rfc/rfc2277#section-4.5) に従って)英語が使用される言語であると仮定できます。

このフィールドが現れる順番は優先順位を示すものではなく、また、このフィールドは 2 回以上現れてはいけません。

- Example (English, Spanish and French):
  - `Preferred-Languages: en, es, fr`

### 2.6. Example of an Unsigned security.txt File

```
# Our security address
Contact: mailto:security@example.com

# Our OpenPGP key
Encryption: https://example.com/pgp-key.txt

# Our security policy
Policy: https://example.com/security-policy.html

# Our security acknowledgments page
Acknowledgments: https://example.com/hall-of-fame.html

Expires: 2021-12-31T18:37:07z
```

### 2.7. Example of a Signed security.txt File

```
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

# Canonical URI
Canonical: https://example.com/.well-known/security.txt

# Our security address
Contact: mailto:security@example.com

# Our OpenPGP key
Encryption: https://example.com/pgp-key.txt

# Our security policy
Policy: https://example.com/security-policy.html

# Our security acknowledgments page
Acknowledgments: https://example.com/hall-of-fame.html

Expires: 2021-12-31T18:37:07z
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v2.2

[signature]
-----END PGP SIGNATURE-----
```

## 3. Location of the security.txt File

Web ベースのサービスの場合、組織は security.txt ファイルを `/.well-known/` パスの下に置かなければいけません。
レガシーとの互換性のために、security.txt ファイルはトップレベルのパスに置かれるか、[RFC7231 のセクション 6.4](https://www.rfc-editor.org/rfc/rfc7231#section-6.4) に従って `/.well-known/` パスの security.txt ファイルにリダイレクトされる可能性があります。
security.txt ファイルが両方の場所に存在する場合、`/.well-known/` パスのものが使用される必要があります。

このファイルは HTTP 1.0 またはそれ以上のバージョンでアクセスされなければならず、ファイルアクセスには `https` スキームを使用しなければいけません。ファイルアクセスの Content-Type は `text/plain` で、デフォルトの charset パラメータは `utf-8` です（[RFC2046 のセクション 4.1.3](https://www.rfc-editor.org/rfc/rfc2046#section-4.1.3) に準拠）。

security.txt ファイルと、そのようなファイル内に示されたリソースを検索するとリダイレクトされる可能性があります。リサーチャーはこれらのリダイレクトが悪意のあるものでないこと、または攻撃者が管理するリソースを指していないことを確認するために追加の分析する必要があります。

### 3.1. Scope of the File

security.txt ファイルは、それを取得するために使用された URI のドメインまたは IP アドレスにのみ適用されなければならず、そのサブドメインや親ドメインには適用されません。
また、security.txt ファイルはそのファイルを公開する組織が提供する製品やサービスにも適用できます。

この仕様は脆弱性対応のためのものであり、実装者がこれをインシデント対応に使用したい場合、セクション 5.1 で議論されている追加のセキュリティ上の考慮事項に注意すべきです。

組織は、（セクション 2.5.7 に従って）ポリシーを使用し、脆弱性開示プロセスの範囲と詳細に関する追加の詳細を提供すべきです。

以下にいくつかの例を示す：

```
# example.com のみに適用
https://example.com/.well-known/security.txt

# subdomain.example.com のみに適用
https://subdomain.example.com/.well-known/security.txt

# IPv4 address 192.0.2.0 のみに適用
https://192.0.2.0/.well-known/security.txt

# IPv6 address 2001:db8:8:4::2 のみに適用
https://[2001:db8:8:4::2]/.well-known/security.txt
```

## 4. File Format Description and ABNF Grammar

security.txt ファイルのフォーマットは、[RFC2046 のセクション4.1.3](https://www.rfc-editor.org/rfc/rfc2046#section-4.1.3) で定義されているようにプレーンテキストでなければならず、Net-Unicode 形式[RFC5198](https://www.rfc-editor.org/rfc/rfc9116#RFC5198) の UTF-8([RFC3629](https://www.rfc-editor.org/rfc/rfc9116#RFC3629)) を使用してエンコードする必要があります。

## 5. Security Considerations

URI と `well-known` リソースを使用するため、以下で説明する考慮事項に加え、 [RFC3986](https://www.rfc-editor.org/rfc/rfc9116#RFC3986) と [RFC8615](https://www.rfc-editor.org/rfc/rfc9116#RFC8615) のセキュリティに関する考慮事項が適用されます。

### 5.1. Compromised Files and Incident Response

ウェブサイトを侵害した攻撃者は、security.txt ファイルも侵害したり、自分のサイトへのリダイレクトを設定したりもできます。その結果、セキュリティ・レポートが組織に届かなくなったり、レポートが攻撃者に送信されたりする可能性があります。

これを防ぐために、組織は "Canonical" フィールドを使用してファイルの場所を示し、security.txt ファイルにデジタル署名を行い、ファイルと参照されるリソースを定期的に監視して改ざんを検出するようにします。

セキュリティリサーチャーは、security.txt ファイルに含まれる情報を使用する前に電子署名を検証し、利用可能な履歴記録をチェックすることを含めて security.txt ファイルを検証すべきです。もし security.txt ファイルが疑わしいか、危ういようであれば使用するべきではありません。

推奨されませんが、実装者は、インシデント対応のために security.txt ファイルで公開された情報を使用することもできます。そのような場合、攻撃者によって侵害された可能性があるため、そのような情報を信用する前に細心の注意を払うべきです。リサーチャーは PGP 署名の帯域外検証、DNSSEC ベースのアプローチなど、このようなデータを検証するために追加の方法を使用する必要があります。

### 5.2. Redirects

ファイルやファイル内で参照されているリソースを取得する際、攻撃者がコントロールする別のドメインや IP アドレスにつながる可能性があるため、リサーチャーはリダイレクトを記録しておく必要があります。
ファイルに含まれる情報を使用する前に、そのようなリダイレクトをさらに検査することが推奨されます。

### 5.3. Incorrect or Stale Information

security.txt ファイルで参照される情報やリソースが不正確であったり、最新の状態に保たれていなかったりすると、セキュリティレポートが組織で受信されなかったり、誤った連絡先に送信されたりして第三者に対してセキュリティ上の問題の可能性を示すことになってしまいます。古い情報があるよりも、security.txt ファイルがない方が望ましいかもしれません。組織は `Expires` フィールドを使用して、ファイルのデータがいつ有効でなくなったかをリサーチャーに示す必要があります。

組織は、このファイル内の情報、およびウェブページ、電子メールアドレス、電話番号などの参照されるリソースが最新に保たれ、アクセス可能であり、組織によって管理され安全に保たれていることを保証する必要があります。

### 5.4. Intentionally Malformed Files, Resources, and Reports

危険なサイトや悪意のあるサイトが、解析コードの弱点を発見または悪用しようとして、非常に大きなファイルや不正な形式のファイルを作成する可能性があります。リサーチャーはそのようなコードが大きなファイルや不正なフィールドに対してロバストであることを確認する必要があり、32KB を超えるファイル、2,048 文字を超えるフィールドを持つファイル、1,000 行を超える行を含むファイルを解析しないようにコードを選択できます。ABNF 文法は、これらのファイルを検証する方法としても使用できます。

同じ懸念が security.txt ファイル内で参照されるその他のリソースや、このファイルを公開した結果として受け取ったセキュリティ報告書にも適用されます。そのようなリソースやレポートは、敵対的であったり、不正であったり、悪意があったりする可能性があります。

### 5.5. No Implied Permission for Testing

security.txt ファイルが存在すると、リサーチャーはそのファイルが公開されているドメインや IP アドレスに対して、あるいはそのファイルを公開している組織が提供する製品やサービスに対して、セキュリティテストを行う許可を与えていると解釈するかもしれません。その結果、リサーチャーによるその組織に対するテストが増加するかもしれません。

一方、security.txt ファイルを公開しないという決定は、そのウェブサイトを運営している組織にとっては、リサーチャーに対してその特定のサイトやプロジェクトをテストする許可を拒否するというシグナルを送る方法であると解釈されるかもしれません。その結果、リサーチャーがその組織にセキュリティ上の問題を報告することに対して、反発を招くかもしれません。

リサーチャーは security.txt ファイルの有無によってセキュリティ・テストの許可を与えるか否かを決めつけるべきではありません。そのような許可は、会社の脆弱性開示ポリシー、または新しいフィールドに示されているかもしれません。

### 5.6. Multi-User Environments

マルチユーザー/マルチテナント環境では、security.txt ファイルの場所がユーザーに乗っ取られる可能性があります。第三者が `/security.txt` と `/.well-known/security.txt` の両方の名前を持つページを作成できないように、組織は security.txt 名前空間をルートに確保する必要があります。

### 5.7. Protecting Data in Transit

security.txt ファイルが転送(transport)中に改ざんされないように保護するために、実装者は、ファイルそれ自体を提供するときと、その中で参照されるすべての Web URI を取得するときに HTTPS を¥使用しなければなりません。TLS Handshake の一部として、リサーチャーは提供された X.509 証明書を [RFC6125](https://www.rfc-editor.org/rfc/rfc9116#RFC6125) と以下の考慮事項に従って検証するべきです：

- 照合は DNS-ID 識別子に対してのみ行われる。
- サーバー証明書に含まれる DNS ドメイン名は、識別子内の左端のラベルとしてワイルドカード文字 `*` を含んでもよい。

証明書は、オンライン証明書ステータスプロトコル(OCSP: [RFC6960](https://www.rfc-editor.org/rfc/rfc9116#RFC6960))、証明書失効リスト (CRL)、または同様の仕組みによって失効が確認されることもあります。

security.txt ファイルが HTTPS 経由で提供できない場合、または無効な証明書で提供されている場合、転送中に内容が変更されている可能性があるため、人力での追加的な検証を推奨します。

追加の保護レイヤーとして、組織は security.txt ファイルに OpenPGP でデジタル署名することも推奨されます。また、転送中にセキュリティ・レポートが改ざんされたり観察されたりすることを防ぐため、レ ポートの提出に HTTPS が使用されていない限り、組織は暗号化キーを指定すべきです。

しかし、そのような鍵の有効性の判断は、本仕様の範囲外です。セキュリティリサーチャーは、鍵を検証するための他の安全な手段を確立する必要があります。

### 5.8. Spam and Spurious Reports

[RFC2142](https://www.rfc-editor.org/rfc/rfc9116#RFC2142) の懸念と同様に、security.txt ファイルが組織によって公開されると、スパム報告によるサービス拒否攻撃が容易になります。
加えて、レポートが自動化された方法で、および、または人による分析なしに自動化されたスキャンの結果として送信される可能性が高くなります。
攻撃者はまた、このファイルを無関係な第三者のリソースや連絡先情報を列挙してスパムを送る方法として使用することもできます。

組織は、このファイルを公開するメリットと、セキュリティ・レポートの分析に必要となる可能性のあるデメリットやリソースの増加とを比較検討する必要があります。

セキュリティリサーチャーは、自動化された方法でレポートを提出する前、あるいは自動化されたスキャンの結果としてレポートを提出する前に、security.txt ファイル内のすべての情報を確認してください。

---

## まとめ

まとめましょう。私個人の見解も含めます。

- 必須なのは `Contact` フィールドと `Expires` フィールド
  - 連絡先が email や電話番号である場合は mailto と tel schema を
  - Expires に設定する値は 1 年以内であることが推奨
- 改竄に備えるために GNU PGP 署名が推奨される
  - 個人的には、リサーチャーのうちどれくらいがちゃんと署名検証してくれるのか疑問
    - 完全性を保証するには有用だけど、当然ながら侵害を検知する仕組みではない
    - 改竄を検知する仕組みは別で検討しておきたい
- スコープとして、security.txt が置かれた URI のドメインまたは IP アドレスにのみ適用されなければならず(MUST)、そのサブドメインや親ドメインには適用されない
  - 一方、security.txt ファイルはそのファイルを公開する組織が提供する製品やサービスにも適用できる(MAY)
- 日本のリサーチャーに明示的に伝える意味でも、Preferred-Languages で日本語がサポートされていることは伝えた方が良さそう
- security.txt を置くことでそこに DoS や大量のレポートが届いて業務が阻害される可能性もあるから、メリットデメリットは見極めて
- `Hiring` フィールドはユーモアがあって良いですね。「security.txt を見て来ました」って security engineer が面接に来たら採用したくなりそうです。

といったところでしょうか。

## おまけ

security.txt の PGP 署名についてはこちらで手順がまとまっていました。

- [Implementing security.txt.](https://pieterbakker.com/implementing-security-txt/)

また、この RFC では security.txt にメリットもデメリットもあると記載されていますが、JPCERT CC からのメッセージにも目を通した上で自組織での実装を検討するのが良いと考えています。

- [A File Format to Aid in Security Vulnerability Disclosure - 正しくつながる第一歩](https://blogs.jpcert.or.jp/ja/2022/08/rfc9116-introduction.html)

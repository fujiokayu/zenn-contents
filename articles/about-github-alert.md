---
title: "GitHub の Dependabot Alerts とは何か"
emoji: "🤔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Security, GitHub]
published: true
---

この記事は[セキュリティ Advent Calendar 2020](https://qiita.com/advent-calendar/2020/security) の19日目です。

<!-- TOC -->

- [はじめに](#はじめに)
- [Dependabot とは何か](#dependabot-とは何か)
- [Dependency Alerts が通知する脆弱性とは何か](#dependency-alerts-が通知する脆弱性とは何か)
  - [リポジトリの Dependency graph とは何か](#リポジトリの-dependency-graph-とは何か)
    - [Dependency graph の制限事項](#dependency-graph-の制限事項)
  - [GitHub Advisory Database とは何か](#github-advisory-database-とは何か)
  - [WhiteSource から通知されるデータとは何か](#whitesource-から通知されるデータとは何か)
- [サポートしているエコシステム](#サポートしているエコシステム)
- [サポートしているリポジトリ](#サポートしているリポジトリ)
- [注意事項](#注意事項)
- [どのように Dependabot Alerts と向き合っていくべきか](#どのように-dependabot-alerts-と向き合っていくべきか)
- [References](#references)

<!-- /TOC -->

## はじめに

![](https://storage.googleapis.com/zenn-user-upload/brcdms04eyeqop5lm2ja3uzdwu4p)

GitHub を利用している人で JavaScript などの言語を使っている場合、Dependabot Alerts は日常的に見かけるものであり、日頃からなんとなくその恩恵に預かっているのではないでしょうか。

`npm audit` でも同様にライブラリの脆弱性を検知する機会はありますが、この機能は明示的に `npm audit` コマンドを叩いた時や `npm install` を実行した時などしか利用できないため、使用しているライブラリに新たな脆弱性が検知された時に自動的に通知してくれる Dependabot Alerts は非常に便利な機能です。

しかし、この通知がどういったリポジトリを対象にしていて、どのようなデータに基づいて通知しているのか、説明できるほどの知識がなかったので改めて調べてみました。

## Dependabot とは何か

[Dependabot](https://dependabot.com/) は依存パッケージの脆弱性を検知し、それを解決するためのプルリクを自動で生成してくれるサービスです。
元々独立したサービスでしたが、[2019年に GitHub が買収](https://dependabot.com/blog/hello-github/)したことにより、現在は GitHub 公式の機能として利用されています。
この記事では GitHub が格リポジトリに対して自動的に通知してくれる Dependabot Alerts を対象に話を進めますが、現在も GitHub の Marketplace から個別に利用できます。

- [Dependabot Preview](https://github.com/marketplace/dependabot-preview)

[後述](#サポートしているエコシステム)する通り自動的に通知される Alerts がサポートしているエコシステムには限りがありますが、個別にインストールした場合は alpha 版の機能である [Rust](https://dependabot.com/rust/) や [Go](https://dependabot.com/go/) にも対応しているようです。


## Dependency Alerts が通知する脆弱性とは何か

GitHub はリポジトリ内の脆弱な依存関係を検出したときに Dependabot のアラートを作成し、その内容はリポジトリのセキュリティタブに表示されます。

![](https://storage.googleapis.com/zenn-user-upload/dy54fbpz3lx5zxkk1ur7bdyp294i)

アラートが作成されるタイミングは以下です。

- 新しい脆弱性が [GitHub Advisory Database](https://github.com/advisories) に追加された時
- [WhiteSource](https://www.whitesourcesoftware.com/vulnerability-database/) から新しい脆弱性のデータが処理された時
- コントリビューターがリポジトリの Dependency graph を更新した時

もう少し詳しく見ていきましょう。

### リポジトリの Dependency graph とは何か

まず、基本的な概念である Dependency graph(依存関係グラフ)について説明します。

GitHub の Dependency graph はリポジトリに格納されているマニフェストとロックファイルに基づいて作成されます。
これはリポジトリページの [insights] > [Dependency graph] から確認できます。

![](https://storage.googleapis.com/zenn-user-upload/7fxp4xye4h99mb40pkpj29xxoik7)

もしプライベートリポジトリでこの機能が有効化されていない場合は、リポジトリページの [Settings] > [Security & analysis] から有効化できます。

サポートされているマニフェストやロックファイルを変更したり、デフォルトブランチに追加したりするコミットを GitHub にプッシュすると、依存関係のグラフが自動的に更新されます。
サポートされているエコシステムとマニフェストファイルについては以下の「[サポートしているエコシステム](#サポートしているエコシステム)」を参照してください。

#### Dependency graph の制限事項

以下の2つのケースでは Dependency graph に制限を与えることに注意が必要です。

1. Processing limits
    - 0.5 MB を超えるサイズのマニフェストは、エンタープライズアカウントでのみ処理されます。それ以外のアカウントでは、0.5MB を超えるマニフェストは無視され、Dependabot アラートは作成されません。
    - デフォルトでは、GitHub はリポジトリごとに 20 個以上のマニフェストを処理しません。制限を増やす必要がある場合は、GitHub サポートか GitHub プレミアムサポートに連絡が必要
2. Visualization limits
    - リポジトリ Dependency graph ビューには100個のマニフェストしか表示されません。GitHub 内に表示されていないマニフェストに対しても Dependabot Alerts は作成されます。


### GitHub Advisory Database とは何か

[GitHub Advisory Database](https://github.com/advisories) には、GitHub の Dependency graph によって追跡されるパッケージにマップされた脆弱性の**キュレーションされた**リストが含まれています。
登録される脆弱性は以下のソースに基づいています。

- [National Vulnerability Database](https://nvd.nist.gov/)
- 機械学習と人間によるレビューの組み合わせによって検知された、GitHub 上のパブリックなコミットにおける脆弱性
- GitHub にレポートされたセキュリティ・アドバイザリー
- [npm Security advisories database](https://www.npmjs.com/advisories)

重要度のレベルは CVSS バージョン 3.0 標準に従って決定されますが、GitHub は Low 〜 Critical のレベルのみを通知し、CVSS のスコアは公開していません。

実際に使っていると `npm audit` は時々大量の Low レベルの脆弱性を大量に検知することがありますが、私は Dependabot Alerts が大量（数百件とか）のアラートをあげるようなケースに遭遇したことはありません。
これは GitHub が独自にキュレーションをすることによって、実際に刺さらないような脆弱性を事前に省いているためかと考えています。（希望的観測）

なお、GitHub Advisory Database は2019年11月に開始され、当初は2017年からサポートされているエコシステムの脆弱性情報を含むように構成されています。データベースに CVE を追加する際には、より新しい CVE、およびより新しいバージョンのソフトウェアに影響を与える CVE を優先的にキュレーションするとのことです。


### WhiteSource から通知されるデータとは何か

以前 Software Design で連載されていた DevSecOps の記事でも紹介されていたので知っている人も多いかもしれませんが、[WhiteSource](https://www.whitesourcesoftware.com/) は OSS のセキュリティやライセンス管理を行うサービスであり、そのサービスを提供する会社です。

[GitHub の公式ドキュメント](https://docs.github.com/ja/free-pro-team@latest/github/managing-security-vulnerabilities/about-alerts-for-vulnerable-dependencies#detection-of-vulnerable-dependencies)が指しているリンクは [WhiteSource Vulnerability Database](https://www.whitesourcesoftware.com/vulnerability-database/#) なので、おそらくここに追加された脆弱性が対象となっているのだと考えられます。

WhiteSource のオープンソース脆弱性データベースは、200 以上のプログラミング言語と 300 万以上のオープンソース コンポーネントをカバーしていて、NVD、security advisories、OSS の Issue Tracker など、さまざまなソースからの情報を 1 日に何度も集約しているとのこと。
[昔書いた記事](https://qiita.com/yuuhu04/items/b2851dedfaa59d634b65)でも一度お値段を調べたのですが、会社として個別に導入しようとすると中々お高い製品です。
- [Pricing](https://www.whitesourcesoftware.com/whitesource-pricing/)

## サポートしているエコシステム

Dependabot Alerts は上述の Dependency graph に基づいて解析を行うため、サポートされるのは言語ではなくパッケージマネージャー単位になります。

:::message alert
そのため、Python で setup.py 内に依存関係が記載されている場合や、Java や Scala でも Maven を使っていない場合はアラートが作成されないことに注意してください。
また、対象のエコシステムを利用している場合でも package-lock.json をリポジトリにコミットしていないケースでもその恩恵は受けられません。
:::

対象のパッケージマネージャーは[この公式ドキュメント](https://docs.github.com/ja/free-pro-team@latest/github/visualizing-repository-data-with-graphs/about-the-dependency-graph)の下の方に記載されています。

![](https://storage.googleapis.com/zenn-user-upload/z95q1tpixbh7iw15q5i5j0eoby2p)

## サポートしているリポジトリ

パブリックリポジトリでは自動的に適用されます。
プライベートリポジトリでも、上述の通り、[Settings] > [Security & analysis] で有効化することで使えるようになります。
公式の [Configuring Dependabot security updates](https://docs.github.com/ja/free-pro-team@latest/github/managing-security-vulnerabilities/configuring-dependabot-security-updates) では[サポートされているリポジトリ](https://docs.github.com/ja/free-pro-team@latest/github/managing-security-vulnerabilities/configuring-dependabot-security-updates#%E3%82%B5%E3%83%9D%E3%83%BC%E3%83%88%E3%81%95%E3%82%8C%E3%81%A6%E3%81%84%E3%82%8B%E3%83%AA%E3%83%9D%E3%82%B8%E3%83%88%E3%83%AA)の中でフォークやアーカイブリポジトリを対象外と書いていますが、恐らく Dependabot Alerts も同じ条件が当てはまるんじゃないかと思います。（憶測）

## 注意事項

[About alerts for vulnerable dependencies](https://docs.github.com/ja/free-pro-team@latest/github/managing-security-vulnerabilities/about-alerts-for-vulnerable-dependencies) には以下の注意が記載されています。

:::message
GitHub のセキュリティ機能はすべての脆弱性を網羅しているわけではありません。私たちは常に脆弱性データベースを更新し、最新の情報でアラートを生成しようとしていますが、すべての脆弱性を把握したり、既知の脆弱性について保証された時間内にお伝えすることはできません。これらの機能は、潜在的な脆弱性やその他の問題について、各依存関係を人力でレビューすることに代わるものではなく、必要に応じてセキュリティサービスに相談するか、徹底的な脆弱性レビューを行うことをお勧めします。
:::

## どのように Dependabot Alerts と向き合っていくべきか

稼働中のアプリにおいて脆弱性に関するパッチマネジメントは非常に重要ですが、Dependabot Alerts だけに対応していれば良い、というわけではなさそうです。

GitHub はグローバルなサービスなので JVN でのみ登録されている脆弱性まで検知することはありませんし、前述の通りサポートしているエコシステムには限りがありますし、2017年より前に報告された脆弱性は検知の対象とならない可能性があります(そんなに長いことメンテされていないライブラリはそもそも使うべきではありませんが)。
また、Windows や Android などのプラットフォームの脆弱性というものもあって、これらはプラットフォームのメンテナが対応するべき問題ではありますが、アプリケーションの開発者も状況に応じて適切なサポート OS のバージョンを更新していく必要があります。

突き詰めるとセキュリティは終わりのない戦いになってしまうので、アプリケーションの要件に応じて対応するレベルを適切に設定することが重要です。
とはいえ使用している OSS の脆弱性の検知と対処はその中でも特に重要な要素であり、保守すべき対象のアプリケーションが上述のエコシステムを適切に使用している場合、Dependabot Alerts は非常に優秀な機能だと考えています。

以上です。もし間違えている箇所などありましたら、コメントや[プルリク](https://github.com/fujiokayu/zenn-contents)をいただけましたら幸いです。

## References

- [セキュリティ Advent Calendar 2020](https://qiita.com/advent-calendar/2020/security)
- [Dependabot](https://dependabot.com/)
- [Dependabot Preview](https://github.com/marketplace/dependabot-preview)
- [GitHub Docs](https://docs.github.com/ja)
  - [About managing vulnerable dependencies](https://docs.github.com/ja/free-pro-team@latest/github/managing-security-vulnerabilities/about-managing-vulnerable-dependencies)
  - [About the dependency graph](https://docs.github.com/ja/free-pro-team@latest/github/visualizing-repository-data-with-graphs/about-the-dependency-graph)
  - [Browsing the Advisory Database](https://docs.github.com/ja/free-pro-team@latest/github/managing-security-vulnerabilities/browsing-security-vulnerabilities-in-the-github-advisory-database)
  - [About alerts for vulnerable dependencies](https://docs.github.com/ja/free-pro-team@latest/github/managing-security-vulnerabilities/about-alerts-for-vulnerable-dependencies)
  - [Troubleshooting the detection of vulnerable dependencies](https://docs.github.com/ja/free-pro-team@latest/github/managing-security-vulnerabilities/troubleshooting-the-detection-of-vulnerable-dependencies)
- [National Vulnerability Database](https://nvd.nist.gov/)
- [npm Security advisories](https://www.npmjs.com/advisories)
- [WhiteSource Vulnerability Database](https://www.whitesourcesoftware.com/vulnerability-database/)
---
title: "GitHub Dependabot Alerts における EPSS Score とは何か"
emoji: "⚠️"
type: "tech"
topics: ["security", "dependabot", "EPSS", "SSVC", "github"]
published: true
---

こんにちは。皆さん Dependabot Alerts は活用していますか？
本記事では、[GitHub Dependabot Alerts](https://docs.github.com/ja/code-security/dependabot/dependabot-alerts) にいつの間にか統合されていた EPSS Score について紹介します。

![Dependabot に表示される EPSS Score](https://storage.googleapis.com/zenn-user-upload/ea24712a7b9f-20250712.png)

## EPSS とは何か

[EPSS (Exploit Prediction Scoring System)](https://www.first.org/epss/) は Black Hat USA 2019 の発表で広く知られるようになった比較的新しい脆弱性の評価指標です ([Exploit Prediction Scoring System (EPSS) - Black Hat USA 2019](https://i.blackhat.com/USA-19/Thursday/us-19-Roytman-Predictive-Vulnerability-Scoring-System-wp.pdf))。

2020年には [FIRST (Forum of Incident Response and Security Teams)](https://www.first.org/) で [EPSS SIG](https://portal.first.org/g/epss-sig) が発足し、翌年には API の公開、その後もバージョンアップが進んで2025年3月には version 4 がリリースされました。

EPSS の現在の定義としては、特定の脆弱性に対して「今後30日間に悪用が観測される確率を毎日推定」した値を 0.00000 〜 1.00000 の範囲で予測するスコアとなっています。

Dependabot Alerts には2025年2月に統合されています。
- [Dependabot helps users focus on the most important alerts by including EPSS scores that indicate likelihood of exploitation, now generally available](https://github.blog/changelog/2025-02-19-dependabot-helps-users-focus-on-the-most-important-alerts-by-including-epss-scores-that-indicate-likelihood-of-exploitation-now-generally-available/)

### EPSS Score の根拠となるデータ

EPSS の予測モデルは以下の特徴量を基に算出されています。

- [Common Platform Enumerations](https://nvd.nist.gov/products/cpe) (CPE)
- 脆弱性の経過年数（CVE が [MITRE CVE list](https://cve.mitre.org/cve/search_cve_list.html) に公開されてからの日数）
- 内容を定義するカテゴリラベル付きのリファレンス、および正規化して抽出された複数語表現: [MITRE CVE list](https://cve.mitre.org/cve/search_cve_list.html)
- [Common Weakness Enumeration](https://nvd.nist.gov/vuln/categories) (CWE)
- [CVSS metrics](https://nvd.nist.gov/vuln-metrics/cvss)
- Web サイトにおける CVE に関する議論やリスト: [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)、[Google Project Zero](https://github.com/googleprojectzero)、[Zero Day Initiative](https://www.zerodayinitiative.com/)、その他追加予定
- 公開されているエクスプロイトコード: [Exploit-DB](https://www.exploit-db.com/)、[GitHub](https://github.com/)、[Metasploit](https://www.metasploit.com/)
- 攻撃系ツール / スキャナーへの適用: [Intrigue](https://github.com/intrigueio/)、[sn1per](https://github.com/1N3/Sn1per)、[jaeles](https://github.com/jaeles-project/jaeles)、[nuclei](https://docs.projectdiscovery.io/tools/nuclei/overview)

モデルの構造には統計的なロジスティック回帰が使われており、データは毎日更新されています。

詳細は [The EPSS Model](https://www.first.org/epss/model) を参照してください。
また、データモデルは [FIRST のリポジトリ](https://github.com/FIRSTdotorg/epss/tree/main/notebooks/jupyter)でも公開されています。

### EPSS に対する評価

2024年に報告された "[A VISUAL EXPLORATION OF EXPLOITATION IN THE WILD](https://www.cyentia.com/epss-study/)" というレポートでは、EPSS Score の性能を検証、評価した結果がまとめられています。

ここでは [KEV (Known Exploited Vulnerabilities Catalog)](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) と比較してのカバレッジ優位性、従来の CVSS ベースの戦略と比較してのコスト優位性について評価しており、CVSS ベースの脆弱性対応と比較すると6倍の効率性を達成できることが強調されています。

また、[GreyNoise](https://www.greynoise.io/)、[Shadow Server Foundation](https://www.shadowserver.org/)、[Fortinet](https://www.fortinet.com/jp)、[AlienVault](https://otx.alienvault.com/)、[Cisco](https://www.cisco.com/)、[F5](https://www.f5.com/)、[Efflux](https://efflux.io/)、[Cyentia](https://www.cyentia.com/) など多数の組織からの継続的なデータ提供についても言及されています。

## Dependabot Alerts における EPSS Score とその活用方法

Dependabot では、FIRST から毎日の同期アクションを通じて EPSS Score を取得しています。

- [GitHub Docs - EPSS スコアの情報](https://docs.github.com/ja/enterprise-cloud@latest/code-security/security-advisories/working-with-global-security-advisories-from-the-github-advisory-database/about-the-github-advisory-database?learn=security_advisories&learnProduct=code-security#about-epss-scores)

上記のドキュメントにも記載されている通り、この活用方法は [FIRST の User Guide](https://www.first.org/epss/user-guide) に詳しく、EPSS と CVSS を併用した優先順位付けが推奨されています。

下の図は、2021年5月16日のデータに基づくEPSSスコアとCVSSスコアの相関関係を示しています。

![](https://storage.googleapis.com/zenn-user-upload/b185c5fbf807-20250712.png)

このグラフは、攻撃者が最も大きな影響を与える脆弱性や、必ずしも容易に悪用できる脆弱性だけを狙っているわけではないことを示しています。

左下に集中した点は悪用可能性が低く（EPSS）、悪用された時の影響（CVSS）も低いと言えます。
一方で左上の点集合は悪用可能性が高いもののその影響は少なく、右下の点集合はその逆です。

この場合、注力して直ちに対応すべきは右上の点集合であることが分かります。
そして、2021年と少し古いデータではありますが、EPSS Score が 50% を超える脆弱性は全体の 5% に過ぎません。

## トリアージにおける留意事項

ここまで EPSS Score について調べた中で、注意すべきことが2点あると感じました。以下は個人の感想です。

### EPSS Score は流動的である

このスコアの性質上、ある日 PoC が公開されたり、悪用された事例が公表されたりするとそのスコアは大きく変わります。
つまり、日常的に脆弱性管理をする中でこのスコアは日次で見直しする必要があり、自動化して確認する手法が必要になります。

幸い、GitHub では Alerts の一覧を EPSS Score で sort することで最新の EPSS Score のランキングを表示できます。

![](https://storage.googleapis.com/zenn-user-upload/67f110f97406-20250712.png)

独自の Alerting system を作るのも効果的です。定期的に API を叩いて一定の基準を超えた脆弱性が検出されたら Slack に通知するのとかが良さそう。

### 悪用の難易度と影響度はシステムのコンテキストに依存する

脆弱性は多くの複雑なコンテキストに影響を受けるため、CVSS と EPSS のみを基準に対応優先度を考えるのでは不十分です。

例えば、API server であっても同一 VPC 内でしか呼び出せない Internal なものや、一部のユーザーのみが利用できるように制御された管理画面、開発者が手元でのみ利用する CLI tool では悪用の難易度は大きく異なります。当然ながら EPSS ではこのコンテキストを評価していません。

[SSVC (Stakeholder-Specific Vulnerability Categorization)](https://www.cisa.gov/stakeholder-specific-vulnerability-categorization-ssvc) についての説明はここでは省略しますが、こうしたモデルによって攻撃を受けるリソースが持つコンテキスト（対象へのアクセス容易性、保持するデータの重要性等）に留意した上で、CVSS や EPSS はその中の1つの要素として機能するべきです。

そして、これらを適切に管理するためには ASM（Attack Surface Management）や CSPM（Cloud Security Posture Management）が運用できていることが望ましいです。

それが難しければ（大多数の人にとって難しいと思う）、本当に大事な資産から少しずつ GitHub Repository の Custom Property などで「すごく大事」「それなりに大事」などの色をつけていくことが最初のステップになると思います。

また、EPSS Score についても組織ごとに対応するべき閾値を決めることは悩ましいと思います。
個人的な意見としては、95% 以上、90、80 と徐々にカバレッジを広げながら、組織として許容できるリスクレベルと現実的に対応可能なコストをすり合わせていくのが良いと考えています。

## 昨今の脆弱性管理における toil

年々発行される CVE は増え続けており、2023 年には 28,818 件だった登録数が 2024 年には 40,009 件へと約 38 ％増加しています。こうした脆弱性管理の作業は、終わりがなく、キリの無い toil (徒労) であることは間違いありません。

![負の傾斜](https://storage.googleapis.com/zenn-user-upload/bfc2f4a52b03-20250713.png)
_出典: [JerryGamblin.com - 2024 CVE Data Review](https://jerrygamblin.com/2025/01/05/2024-cve-data-review/)_

リスクベースという言葉が一般的になって久しく、これ自体は望ましい考え方だと思いますが、それぞれのリスクに対する解像度についてはまだまだ課題が多いと思います。
EPSS や SSVC などのモデルを通じてリスクに対する解像度を上げながら、より効果的なセキュリティ対策を考えていきましょう。

:::message alert
深刻な脆弱性がないからといって Major update を何度も見送っているとその更新作業はより困難になります。
いざという時にすぐに Update できるよう、依存ライブラリは可能な限り最新に保ちましょう。
:::

## References

- [GitHub Dependabot Alerts](https://docs.github.com/ja/code-security/dependabot/dependabot-alerts)
- [EPSS (Exploit Prediction Scoring System)](https://www.first.org/epss/)
- [Black Hat USA 2019 — Predictive Vulnerability Scoring System (EPSS) Whitepaper](https://i.blackhat.com/USA-19/Thursday/us-19-Roytman-Predictive-Vulnerability-Scoring-System-wp.pdf)
- [FIRST (Forum of Incident Response and Security Teams)](https://www.first.org/)
- [FIRST — EPSS SIG](https://portal.first.org/g/epss-sig)
- [The EPSS Model](https://www.first.org/epss/model)
- [EPSS User Guide](https://www.first.org/epss/user-guide)
- [FIRSTdotorg / epss GitHub Repository](https://github.com/FIRSTdotorg/epss)
- [GitHub Changelog — Dependabot integrates EPSS scores](https://github.blog/changelog/2025-02-19-dependabot-helps-users-focus-on-the-most-important-alerts-by-including-epss-scores-that-indicate-likelihood-of-exploitation-now-generally-available/)
- [GitHub Docs — About EPSS scores](https://docs.github.com/ja/enterprise-cloud@latest/code-security/security-advisories/working-with-global-security-advisories-from-the-github-advisory-database/about-the-github-advisory-database#about-epss-scores)
- [NVD — Common Platform Enumerations (CPE)](https://nvd.nist.gov/products/cpe)
- [MITRE — CVE List](https://cve.mitre.org/cve/search_cve_list.html)
- [NVD — Common Weakness Enumeration (CWE) Categories](https://nvd.nist.gov/vuln/categories)
- [NVD — CVSS Metrics](https://nvd.nist.gov/vuln-metrics/cvss)
- [CISA — Known Exploited Vulnerabilities (KEV) Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [Google Project Zero (GitHub)](https://github.com/googleprojectzero)
- [Zero Day Initiative (ZDI)](https://www.zerodayinitiative.com/)
- [Exploit Database (Exploit-DB)](https://www.exploit-db.com/)
- [Metasploit Framework](https://www.metasploit.com/)
- [ProjectDiscovery — Nuclei](https://docs.projectdiscovery.io/tools/nuclei/overview)
- [Cyentia Institute — *A Visual Exploration of Exploitation in the Wild* (EPSS Study 2024)](https://www.cyentia.com/epss-study/)
- [Stakeholder-Specific Vulnerability Categorization (SSVC)](https://cisa.gov/stakeholder-specific-vulnerability-categorization-ssvc)
- [JerryGamblin.com - 2024 CVE Data Review](https://jerrygamblin.com/2025/01/05/2024-cve-data-review/)

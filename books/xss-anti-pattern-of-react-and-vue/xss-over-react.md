---
title: "React における XSS"
free: true
---

React で開発者が実装するのは仮想 DOM であり、実際にブラウザに読み込ませる DOM をレンダリングする際に View 層の不適切な文字列は適宜エスケープされます。
そのため `<script>` のような危険な文字列はブラウザに表示される時には無害化されており、仮にユーザーが入力した `<script>alert("Injection!")</script>` といった文字列をそのまま表示するアプリケーションであっても、表示されるのはエスケープされた文字列リテラルであり、ブラウザに解釈され得る JavaScript コードにはなりません。

これらのエスケープ機構が機能する範囲では XSS が刺さるケースはないと思いますが、残念ながらこのエスケープ処理が行われないパターンが React には存在します。

# innerHTML を使用するケース(dangerouslySetInnerHTML)

これだけ禍々しい名前が付いていればおいそれと使うケースはほとんどないと思いますが、ユーザーが直接テキストのスタイルを設定するようなケースでは手っ取り早く実装できる手段ではあります。

以下では XSS をハードコードしていますが、これはエンドユーザーの入力を動的に処理する場合にも再現します。

```js
const userInput = <b>"Hi, <img src='' onerror='alert(0)' />"</b>;
return <div dangerouslySetInnerHTML={{ __html: userInput }} />;
```

これによって懸念されるリスクは [element.innerHTML](https://developer.mozilla.org/ja/docs/Web/API/Element/innerHTML) をそのまま使用することに相当します。
その場合にどのような XSS が成立するのかについての詳細は、次章の [HTML の挿入](xss-over-vue#html-の挿入)に記載します。

`dangerouslySetInnerHTML` は利用しないに越したことはありませんが、どうしても利用したい場合は、セットでエスケープ処理を実装することが必要になります。
車輪を再発明する必要はないので、[DOMPurify](https://github.com/cure53/DOMPurify) などのサードパーティー製ライブラリを利用するのが良いでしょう。
独自実装で `<script>` タグのエスケープなどを実装すると、`<scr<script>ipt>` などの入力によってエスケープがバイパスされるなどの危険性があるため、余程腕に自身がある開発者以外は避けた方が賢明です。

また、サードパーティー製のライブラリであっても iframe タグ内のスクリプトは適切にエスケープされない例もあるので、ライブラリの採用は慎重に検討した上で行ってください。
[CodeSandbox](https://codesandbox.io/index2) で見つけた [React-intl XSS](https://codesandbox.io/s/x864p) では、[React Intl](https://formatjs.io/) の `FormattedHTMLMessage` を使用した XSS の実装が紹介されています。

# 受け取ったユーザー入力をそのまま href 属性に渡すケース

現実的に起こり得るパターンだと思うのがこれです。
props や state 経由で href タグにデータを渡すと、DOM にレンダリングされる前にデータがエスケープされません。

:::message
2024/12/10 追記

React v19 で対策が merge されました。

> - Javascript URLs are replaced with functions that throw errors ([#26507](https://github.com/facebook/react/pull/26507), [#29808](https://github.com/facebook/react/pull/29808) by @sebmarkbage and @kassens)
>
> https://github.com/facebook/react/releases/tag/v19.0.0

javascript: が設定されたリンクは `javascript:throw new Error('React has blocked a javascript: URL as a security precaution.')`に置き換えられるようです。

:::

私が以前実装(の途中で放置)したやられアプリにこの脆弱性を実装しています。
[amplify-goat](https://github.com/fujiokayu/amplify-goat) は React で実装されたシンプルな Todo アプリですが、`https` で始まる Todo はリンク URL として解釈するというお節介な機能が実装されており、その結果以下の脆弱なコードが実装されました。

```js
          links.map((url) => (
            <p key={url.id}>
              {url.name} : <a href={url.description}>{url.description}</a>
            </p>
          ))
```

href に直接渡された `url.description` はエスケープされずにレンダリングされます。
ここに以下の入力を行います。

```js
javascript: alert("https://evil-faas-functions")
```

このペイロードはエスケープされずに href タグに直接渡されるため、XSS が成立します。
このアプリはサインインしたユーザーの Todo しか表示しないため一見反射型 XSS に見えますが、なにも考えずにフレームワークに頼り切って実装した結果アクセスコントロールにも脆弱性が混入してしまったため、他人の Todo としてデータベースに登録することが可能であり、蓄積型 XSS として深刻な脆弱性になります。

![](https://storage.googleapis.com/zenn-user-upload/mvk88miactn0dp3sczuojx0ingdq =400x)

原理的には `https://` に続く URL として悪意のある Lambda 関数や Cloud Functions を指定し、Local Storage に格納されている JWT に含まれる Access Token を盗み出してクラウド上にログ出力するようにすれば、ATO(Account Take Over) も可能になります。

amplify-goat は未完ながら PlayGround も用意しているので、良かったら遊んでいってください。
アプリの要件上会員登録が必須でその情報は Amazon Cognito に登録されますが、本アプリの利用以外の理由では個人情報は使用しません。
ただし Cognito に登録されたアカウント情報や Dynamo DB 上に登録された Todo データは私が気まぐれに Clean Up することについてはご了承いただけたらと存じます。

- [amplify-goat - PlayGround](https://amplify-goat-20200224131521-hostingbucket-ampgoat.s3-ap-northeast-1.amazonaws.com/index.html)

アクセスコントロールの脆弱性を突けば admin ユーザーが登録した Todo を参照することもできますし、admin ユーザーとして新しい Todo を追加することも可能です。

# 危険な JavaScript の挿入

セキュリティ診断などでソースコードを解析する場合、私はよく [eval()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/eval) がソースコード内に含まれていないか確認しています。
昨今でこのようなコードが実装されるケースは少ないと思いますが、eval などの危険な関数にユーザー入力を渡すような実装はフレームワークを問わずアンチパターンです。

[Exploiting Script Injection Flaws in ReactJS Apps](https://medium.com/dailyjs/昨今でこのようなコードが実装されるケースは少ないと思いますが、exploiting-script-injection-flaws-in-reactjs-883fb1fe36c1) からコードを抜粋します。

```js
function antiPattern() {
  eval(this.state.attacker_supplied);
}
// Or even crazier
fn = new Function("..." + attacker_supplied + "...");
fn();
```

eval 以外にも危険な関数はありますが、ESLint などで [Disallow eval()](https://eslint.org/docs/rules/no-eval) などの Rule を適宜設定し、機械的に防ぐようにしておくことが望ましいと言えます。

# SSR を使用しているケース

[Exploiting Script Injection Flaws in ReactJS Apps](https://medium.com/dailyjs/exploiting-script-injection-flaws-in-reactjs-883fb1fe36c1) には、SSR 用の公式 Redux コードサンプルに XSS の脆弱性が発生していたことが挙げられています。
現在、Redux の Preparing the Initial State における [Security Considerations](https://redux.js.org/recipes/server-rendering#security-considerations) のドキュメントでは以下のように記載されています。

>この単純な例では、入力を数値に強制することで十分に安全です。フリーフォームテキストのようなより複雑な入力を扱う場合は、その入力を [xss-filters](https://github.com/yahoo/xss-filters) のような適切なサニタイズ関数を通して実行する必要があります。
>
>さらに、状態出力をサニタイズすることでセキュリティの追加レイヤーを追加することができます。`JSON.stringify` はスクリプトの注入を受ける可能性があります。これに対抗するには、JSON 文字列に HTML タグやその他の危険な文字をスクラブすることができます。これは、例えば `JSON.stringify(state).replace(/</g, '\u003c')` のような文字列の単純なテキスト置換か、[serialize-javascript](https://github.com/yahoo/serialize-javascript) のようなより洗練されたライブラリを介して行うことができます。
>
>ユーザー生成コンテンツ(UGC)や入力に依存するコードをより多く導入したため、アプリケーションの攻撃範囲が広がりました。クロスサイトスクリプティング(XSS)攻撃やコードインジェクションのようなものを防ぐために、入力が適切にサニタイズされていることを確認することは、どのようなアプリケーションにとっても重要です。

SSR に限ったことではありませんが、ユーザーが入力した値を検証せずにデシリアライズすることは望ましくありません。

# React.createElement に任意の属性を指定できるケース

通常 React.createElement() を開発者が呼び出すのはレアなケースだと思いますが、諸事情によって [JSX なしで React を使う](https://ja.reactjs.org/docs/react-without-jsx.html)ようなケースでは、Type と Props はエスケープされずに処理されるため留意が必要です。
ただし、学習目的以外で React の Top-Level API を利用するケースはほとんどないものと思います。

# 未知、または既知の脆弱性を利用した XSS

未知の脆弱性を未然に防ぐことは本質的に不可能です。
このリスクを緩和するためには、攻撃されたことを検知するためのバックエンドのロギングや監視機能、WAF などによる対処が必要になります。

一方で、既知の脆弱性は開発や運用の中で未然に防ぐことが可能です。
1例として、2018年と少し昔の話ですが、Next.js で XSS が成立するという脆弱性が見つかりました。

- [JVNDB-2018-010921](https://jvndb.jvn.jp/ja/contents/2018/JVNDB-2018-010921.html)

この時に Vercel が発表した Release Note は以下です。

- [7.0.2 Release Notes](https://github.com/vercel/next.js/releases/tag/7.0.2)

最新のバージョンのフレームワークを利用する限り、(メンテされていないフレームワークを除き)基本的に既知の脆弱性は対応されているはずです。
ただし、過去に実装したアプリのパッチマネジメントを適切に行わない場合、既知の脆弱性が悪用される恐れがあります。
GitHub や `npm audit` の警告を軽視せず、また、可能であれば日々公開される CVE などの脆弱性情報の定期的なチェックを行うことが望ましいと言えます。

# References

- [MDN - element.innerHTML](https://developer.mozilla.org/ja/docs/Web/API/Element/innerHTML)
- [DOMPurify](https://github.com/cure53/DOMPurify)
- [React-intl XSS](https://codesandbox.io/s/x864p)
- [Preventing XSS in React Applications](https://dev.to/spukas/preventing-xss-in-react-applications-5f5j)
- [Avoiding XSS in React is Still Hard](https://medium.com/javascript-security/avoiding-xss-in-react-is-still-hard-d2b5c7ad9412)
- [amplify-goat](https://github.com/fujiokayu/amplify-goat)
- [MDN web docs - eval()](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/eval)
- [Exploiting Script Injection Flaws in ReactJS Apps](https://medium.com/dailyjs/exploiting-script-injection-flaws-in-reactjs-883fb1fe36c1)
- [Redux - Security Considerations](https://redux.js.org/recipes/server-rendering#security-considerations)
- [JVNDB-2018-010921](https://jvndb.jvn.jp/ja/contents/2018/JVNDB-2018-010921.html)
- [Next.js - 7.0.2 Release Notes](https://github.com/vercel/next.js/releases/tag/7.0.2)
- [3 Security Pitfalls Every React Developer Should Know](https://medium.com/swlh/3-security-pitfalls-every-react-developer-should-know-c04715b876b5)
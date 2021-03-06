---
title: "Vue における XSS"
free: true
---

# Vue の公式ドキュメントに記載されているケース

Vue はセキュリティに関して素晴らしい公式ドキュメントがあります。
Vue を使用して実装している人は是非一度読んでみてください。

- [Vue.js - セキュリティ](https://jp.vuejs.org/v2/guide/security.html)

このドキュメントの[ルール No.1: 信頼できないテンプレートを絶対に使わない](https://jp.vuejs.org/v2/guide/security.html#%E3%83%AB%E3%83%BC%E3%83%AB-No-1-%E4%BF%A1%E9%A0%BC%E3%81%A7%E3%81%8D%E3%81%AA%E3%81%84%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%E3%82%92%E7%B5%B6%E5%AF%BE%E3%81%AB%E4%BD%BF%E3%82%8F%E3%81%AA%E3%81%84)では「信頼できないコンテンツをコンポーネントのテンプレートとして絶対に使わない」こと、と書いてあり、例として以下のコードが示されています。

```js
new Vue({
  el: '#app',
  template: `<div>` + userProvidedString + `</div>` // 絶対にしてはいけない
})
```

続く [Vue が行っているセキュリティ対策](https://jp.vuejs.org/v2/guide/security.html#Vue-%E3%81%8C%E8%A1%8C%E3%81%A3%E3%81%A6%E3%81%84%E3%82%8B%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E5%AF%BE%E7%AD%96)では [React と Vue のエスケープ機構について](3.md)で触れたエスケープ機構について説明されます。
一見するとこの対策によって Injection Safe なように見受けられますが、その次の章には[潜在的な危険](https://jp.vuejs.org/v2/guide/security.html#%E6%BD%9C%E5%9C%A8%E7%9A%84%E3%81%AA%E5%8D%B1%E9%99%BA)の説明があります。

## HTML の挿入

Vue では HTML コンテンツを自動でエスケープ処理し、誤って実行可能な HTML をアプリケーション内に挿入することを防いでいますが、HTML が安全なことが事前にわかっている場合は明示的にそれをレンダリングすることが可能であり、v-html を使う場合は XSS が刺さりやすいということは広く知られています。

- テンプレートを利用する場合

```js
<div v-html="userProvidedHtml"></div>
```

- 描画関数を利用する場合:

```js
h('div', {
  domProps: {
    innerHTML: this.userProvidedHtml
  }
})
```

- JSX による描画関数を利用する場合:

```js
<div domPropsInnerHTML={this.userProvidedHtml}></div>
```

[React の章]((#jsx-を使わないケースdangerouslysetinnerhtml))でも少し触れましたが、これらのケースは [element.innerHTML](https://developer.mozilla.org/ja/docs/Web/API/Element/innerHTML) をそのまま使用することに相当します。
この詳細について、リンクの MDN の「セキュリティの考慮事項」から転載します。
>ウェブページにテキストを挿入するために innerHTML を使用している例は珍しくありませんありません。これがサイト上の攻撃ベクトルになる可能性があり、潜在的なセキュリティリスクが生じます。

```js
const name = "John";
// 'el' を HTML の DOM 要素と想定します
el.innerHTML = name; // この場合は無害

// ...

name = "<script>alert('I am John in an annoying alert!')</script>";
el.innerHTML = name; // この場合は無害
```

>これはクロスサイトスクリプティング攻撃のように見えますが、結果的には無害です。 HTML5 では innerHTML で挿入された `<script>` タグは実行するべきではないと定義しているからです。
>
>しかし、次のように `<script>` を使わずに JavaScript を実行する方法もあるので、制御することができない文字列を設定するために innerHTML を使用するたびに、セキュリティリスクは残ります。

```js
const name = "<img src='x' onerror='alert(1)'>";
el.innerHTML = name; // アラートが表示される
```

>このため、プレーンテキストを挿入するときには innerHTML を使用せず、代わりに Node.textContent を使用することをお勧めします。これは渡されたコンテンツを HTML として解釈するのではなく、生テキストとして挿入します。

v-html を使った簡易的な [Playground](https://vue-xss-poc.vercel.app/) を用意しました。肩慣らしにここで遊んでみましょう。
※ 下段の v-bind は次の節で使用します。

:::message
なお、Playground の Vue のバージョンは3系です。

```shell
~/r/vue-xss-poc > npm list vue
vue-xss-poc@0.1.0 /Users/fujiokayu/repo/vue-xss-poc
└── vue@3.0.0
```

:::

上述の MDN に記載されている通り、

```js
<script>alert('I am John in an annoying alert!')</script>
```

のような単純な XSS は刺さりません。
しかし、以下のようなケースは刺さります。

```js
<img src='x' onerror='alert(1)'>
```

他の攻撃方法が知りたければ、[OWASP XSS Filter Evasion Cheat Sheet](https://owasp.org/www-community/xss-filter-evasion-cheatsheet) を参照してみてください。

```js
<IFRAME SRC=# onmouseover="alert(document.cookie)"></IFRAME>
// 展開された IFRAME に mouseover すると発火します。
```

や

```js
<EMBED SRC="data:image/svg+xml;base64,PHN2ZyB4bWxuczpzdmc9Imh0dH A6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcv MjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hs aW5rIiB2ZXJzaW9uPSIxLjAiIHg9IjAiIHk9IjAiIHdpZHRoPSIxOTQiIGhlaWdodD0iMjAw IiBpZD0ieHNzIj48c2NyaXB0IHR5cGU9InRleHQvZWNtYXNjcmlwdCI+YWxlcnQoIlh TUyIpOzwvc2NyaXB0Pjwvc3ZnPg==" type="image/svg+xml" AllowScriptAccess="always"></EMBED>
```

など、既知の XSS の多くが刺さることを確認できます。

## v-bind:href にユーザーインプットを渡すケース

公式の記載を転載します。
>このような URL の場合:
>
>```js
><a v-bind:href="userProvidedUrl">
>  click me
></a>
>```
>
>URL を “無害化 (sanitize)” して javascript: の利用による JavaScript 実行を防いでいない場合、潜在的なセキュリティの問題があります。これの対策に、[sanitize-url](https://www.npmjs.com/package/@braintree/sanitize-url) 等のライブラリがあります。しかし注意してください:
>:::message alert
>フロントエンドで URL の無害化 (sanitize)処理を行ったことがある場合、それはすでにセキュリティ上の問題をはらんでいます。ユーザの入力による URL は、常にバックエンドでデータベースに保存する前の処理が必要です。そうすることでモバイルのネイティブアプリを含め、API に接続するすべてのクライアントで問題を回避することができます。また、無害化 (sanitize)処理がされた URL だとしても、Vue はリンク先の安全性を保証することはできません。
>:::

ここで起こっていることは[前章](4)で触れた[受け取ったユーザー入力をそのまま href 属性に渡すケース](4#受け取ったユーザー入力をそのまま-href-属性に渡すケース)と全く同じです。

先述の [Playground](https://vue-xss-poc.vercel.app/) にある [input your url] というプレースホルダーのテキストボックスに、以下のシンプルなペイロードを入力して [Click Me] ボタンをクリックしてみてください。

```js
javascript: alert("XSS")
```

なお、[公式ドキュメント](https://jp.vuejs.org/v2/guide/security.html#%E3%82%B9%E3%82%BF%E3%82%A4%E3%83%AB%E3%81%AE%E6%8C%BF%E5%85%A5)では続けて `v-bind` におけるクリックジャッキング脆弱性を紹介しているので、興味のある方は是非読んでみてください。

## JavaScript の挿入

[Vue が行っているセキュリティ対策](https://jp.vuejs.org/v2/guide/security.html#Vue-%E3%81%8C%E8%A1%8C%E3%81%A3%E3%81%A6%E3%81%84%E3%82%8B%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E5%AF%BE%E7%AD%96)からの転載です。

>テンプレートと描画関数は副作用をもつべきではないため、Vue で `<script>` 要素をレンダリングすることは強くお勧めしません。ただしこれは、実行時に JavaScript として評価される文字列を含めるための唯一の方法ではありません。
>
>すべての HTML 要素には、onclick や onfocus、onmouseenter などの JavaScript 文字列を値として受け入れる属性があります。ユーザによって入力された JavaScript をこれらのイベント属性にバインドすることは、潜在的なセキュリティリスクとなるので避ける必要があります。

前章で触れた [eval](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/eval) などもそうですが、フレームワークが予期していない JavaScript が挿入されるような実装をする場合、そのフレームワークの持つ恩恵を受けることはできません。

# SSR されたデータに任意のユーザー入力が含まれるケース

SSR によって XSS が成立するケースについては、[dotboris/vuejs-serverside-template-xss](https://github.com/dotboris/vuejs-serverside-template-xss) で詳しく説明されています。
このリポジトリでは、サーバーサイドレンダリングと Vue.js の両方を使用している Web アプリが XSS の脆弱性にさらされる可能性があることを示しています。
[Running the demo](https://github.com/dotboris/vuejs-serverside-template-xss) を参考にセットアップし、是非、実際に試してみてください。
`docker-compose` コマンドで直ぐにローカル環境で確認できるようになります。

![](https://storage.googleapis.com/zenn-user-upload/93jrvqlpazlb8w6an7arui5k78rr =300x)

画面に表示されているテキストボックスに値を入れて [Go!] ボタンをクリックすると、入力した値がクエリパラメーターとして使用されていることがわかります。
`http://localhost:8080/?injectme=test`

また、画面上にも入力したテキストが表示されます。
ここで単純なスクリプト、`<script>alert('xss')</script>` を挿入しても問題なくエスケープされて表示されていることがわかります。

:::message alert
**Spoiler Alert**
やられアプリとして [vuejs-serverside-template-xss](https://github.com/dotboris/vuejs-serverside-template-xss) で遊んでみたい方は、以下はネタバレ注意です。
:::

ソースコードを見ると、[htmlspecialchars](https://www.php.net/manual/ja/function.htmlspecialchars.php) でエスケープした「安全なデータ」をサーバーサイドから受け取り、クライアントサイドの Vue でそのまま表示していることがわかります。

```html
  <div id="injectable-app">
    <div>
      You have injected:
      <?= htmlspecialchars((string) $_GET['injectme']) ?>
    </div>

    <button type="button" @click="dec">-</button>
    {{counter}}
    <button type="button" @click="inc">+</button>
  </div>
```

一見安全な実装に見えますが、これは典型的な信頼境界線の問題であり、SSR を使用する場合はサーバーからの入力も疑う必要があります。

Vue ではムスターシュで式が実行されるため、`{{ 2 + 2 }}` というペイロードをテキストボックスに入力すると、式が実行されることが分かります。
次に `{{ alert('xss') }}` を入力したところ、カウンターが消えるなどのバグらしい挙動が見られます。
コンソールを見ると以下のエラーが確認できます。

```bash
TypeError: alert is not a function
    at Proxy.eval (eval at createFunction (vue.js:10518), <anonymous>:2:114)
    at Vue$3.Vue._render (vue.js:4465)
    at Vue$3.updateComponent (vue.js:2765)
    at Watcher.get (vue.js:3113)
    at new Watcher (vue.js:3102)
    at mountComponent (vue.js:2772)
    at Vue$3.$mount (vue.js:8416)
    at Vue$3.$mount (vue.js:10777)
    at Vue$3.Vue._init (vue.js:4557)
    at new Vue$3 (vue.js:4646)
```

Vueの 式は、レンダリングする Vue インスタンスのコンテキストで評価されます。
つまり、{{foobar }}をレンダリングしようとすると template 内の foobar プロパティを探します。
このため、template のプロパティに alert('xss') が存在しなかったために予期せぬエラーが発生してしまいました。

このリポジトリの作者がこのサンドボックスを回避するためのペイロードを紹介しています。
`{{ constructor.constructor("alert('xss')")() }}`

これは AngularJS に以前実装されていたサンドボックス回避として広く知られているテクニックで、PortSwigger 社のブログでも紹介されています。

- [DOM based AngularJS sandbox escapes](https://portswigger.net/research/dom-based-angularjs-sandbox-escapes)

constructor は Object（template）のコンストラクタを指し、constructor.constructor は関数コンストラクタになり、文字列から関数を生成して任意のコードを実行することができます。
上記のペイロードを送った場合、以下のように解釈されます。
`function("alert('xss')")()`

このケースでは、ユーザーの入力（クエリパラメータ）を受け取り、HTML ページをレンダリングするためにそれを使用する PHP バックエンドがあり、それは HTML 要素の入力をエスケープし、単純な XSS が不可能であることを確実にします。
しかし、ページがブラウザに表示されると、Vue.js はこの HTML の一部を受け取り、テンプレートの一部としてレンダリングします。
結果としては、この HTML に対して複雑な eval を行っていることと同等です。

これを防ぐ簡単な方法は、サーバ側の値をクライアント側のテンプレートに注入するときには常に v-pre ディレクティブを使用することですが

```js
<div id="injectable-app">
  <div v-pre>
    You have injected:
    <?= htmlspecialchars($_GET['injectme']) ?>
  </div>

  <button type="button" @click="dec">-</button>
  {{counter}}
  <button type="button" @click="inc">+</button>
</div>
```

この解決策について、dotboris はより体系的な解決策としてページの中でグローバル変数を定義して、 すべてのサーバ側の変数を一度変数に保管する方法を紹介しています。

```js
<div id="injectable-app">
  <div>
    You have injected: {{ SERVER_VARS.injectMe }}
  </div>

  <button type="button" @click="dec">-</button>
  {{counter}}
  <button type="button" @click="inc">+</button>
</div>
<script src="https://cdn.jsdelivr.net/npm/vue@2.5.13/dist/vue.js"></script>
<?php
$serverVars = [
  'injectMe' => (string) $_GET['injectme']
];
?>
<script>
  window.SERVER_VARS = JSON.parse(atob('<?= base64_encode(json_encode($serverVars)) ?>'));
  Vue.prototype.SERVER_VARS = window.SERVER_VARS;
</script>
```

React や Angular 2+ のようなフレームワークはテンプレートを HTML で書かせないようにしていますし、この手の攻撃は刺さりにくいです。
これについて dotboris は素晴らしい格言でこの README を締めています。

>React や Angualar 2+ が安全だと言っているわけではありません。脆弱性があるからこそ努力しなければならないだけです。

# 未知、または既知の脆弱性を利用した XSS

これについては前章で述べたことと同様です。
Vue や Vue を使用したフレームワーク ([Nuxt.js](https://ja.nuxtjs.org/) など)でも脆弱性は報告されています。

過去に実装したアプリのパッチマネジメントを適切に行わない場合、既知の脆弱性が悪用される恐れがあるので、どのようなフレームワークを利用していたとしても、日々のパッチマネジメントや情報収集を欠かさないことが望ましいと言えます。

# References

- [Vue.js - セキュリティ](https://jp.vuejs.org/v2/guide/security.html)
- [MDN - element.innerHTML](https://developer.mozilla.org/ja/docs/Web/API/Element/innerHTML)
- [vue-xss-poc(Playground)](https://github.com/fujiokayu/vue-xss-poc)
- [OWASP XSS Filter Evasion Cheat Sheet](https://owasp.org/www-community/xss-filter-evasion-cheatsheet) 
- [sanitize-url](https://www.npmjs.com/package/@braintree/sanitize-url)
- [dotboris/vuejs-serverside-template-xss](https://github.com/dotboris/vuejs-serverside-template-xss)
- [htmlspecialchars](https://www.php.net/manual/ja/function.htmlspecialchars.php)
- [DOM based AngularJS sandbox escapes](https://portswigger.net/research/dom-based-angularjs-sandbox-escapes)

---
title: "React と Vue のエスケープ機構について"
free: true
---

React にも Vue にも、インプットされる文字列にはエスケープ処理が施されます。

- [js.reactjs.org - 文字列リテラル](https://ja.reactjs.org/docs/jsx-in-depth.html#string-literals)
- [Vue が行っているセキュリティ対策](https://jp.vuejs.org/v2/guide/security.html#Vue-%E3%81%8C%E8%A1%8C%E3%81%A3%E3%81%A6%E3%81%84%E3%82%8B%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E5%AF%BE%E7%AD%96)

そのため、React では以下の式が等価であり

```js
<MyComponent message="&lt;3" />

<MyComponent message={'<3'} />
```

Vue では以下の式は

```js
'<script>alert("hi")</script>'
```

以下の式に変換されます。

```js
&lt;script&gt;alert(&quot;hi&quot;)&lt;/script&gt;
```

一般的に XSS 脆弱性を防ぐためにエスケープ処理を施すことはグッドパターンであり、ユーザーが直接入力した文字列リテラルをそのまま処理することは高いリスクを伴います。
※一方で、攻撃を無害化してアラートを挙げずに処理することは [OWASP TOP 10 - A10:2017-Insufficient Logging & Monitoring](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A10-Insufficient_Logging%252526Monitoring) が示す「不十分なロギングと監視」に該当することに留意が必要です。

これらのエスケープ機構によって React や Vue は XSS によるリスクを低減していますが、いくつかのパターンではこれらのエスケープ機構が機能しないケースが存在するため、「React だから大丈夫」「Vue を使っていれば XSS は起こらない」と断定することはできません。

代表的な例では `dangerouslySetInnerHTML` や `v-html` の使用が挙げられますが、残念ながらそれだけではありません。
次章からは、それぞれのフレームワークで混入し得る XSS 脆弱性について説明します。

# References

- [js.reactjs.org - 文字列リテラル](https://ja.reactjs.org/docs/jsx-in-depth.html#string-literals)
- [Vue が行っているセキュリティ対策](https://jp.vuejs.org/v2/guide/security.html#Vue-%E3%81%8C%E8%A1%8C%E3%81%A3%E3%81%A6%E3%81%84%E3%82%8B%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E5%AF%BE%E7%AD%96)
- [OWASP TOP 10 - A10:2017-Insufficient Logging & Monitoring](https://owasp.org/www-project-top-ten/OWASP_Top_Ten_2017/Top_10-2017_A10-Insufficient_Logging%252526Monitoring)
---
title: "昨今のフロントエンドで使われているフレームワークについて"
free: true
---

# 昨今のフロントエンドで使われているフレームワークについて

[The Software House](https://tsh.io/) が2020年に4500人のフロントエンドエンジニアを対象に調査した [State of Frontend 2020](https://tsh.io/state-of-frontend/) では、以下の調査結果が報告されていました。

## この1年間で利用したフレームワークは？

![](https://storage.googleapis.com/zenn-user-upload/365x41g9hbvxucmkm3x456on38ss)

## 今後も使っていきたいフレームワーク、学んでいきたいフレームワークは？

![](https://storage.googleapis.com/zenn-user-upload/6pyz8x9esmq372yzlnuncizvfmjw)

こういった調査は対象となるサンプルによって結果が異なることが多々ありますが、

- React が最も使われている
- Vue のシェアが増えてきている

という点は昨今の潮流として定着してきているのではないでしょうか。  
※特に日本では海外よりも Vue の人気が高い印象があります。

React と Vue では、仮想 DOM を活用しているという点やコンテンツを自動的にエスケープするという点などで共通点が見られます。

本書では、この2つのフレームワークに実装されたエスケープ機構、およびこれらのフレームワークで実装される可能性がある XSS 脆弱性について説明します。

# References

- [State of Frontend 2020](https://tsh.io/state-of-frontend/)

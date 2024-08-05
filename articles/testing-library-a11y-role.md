---
title: "React Testing LibraryでのWAI-ARIAロールの活用事例" # 記事のタイトル
emoji: "♿️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ['React', 'TypeScript', "TestingLibrary", "accessibility"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

こんにちは！サイボウズ様で業務委託でフロントエンドエンジニアをしている [Nokogiri](https://twitter.com/nkgrnkgr) です。
このたびはCybozu Summer Blog Fes'24にて執筆の機会をいただきましたので、僭越ながら寄稿させていただきます。

今回は**React Testing LibraryでのWAI-ARIAロールの活用事例**について紹介させていただきます。

## 前提

Reactで書かれたUIをテストする方法の一つとして[Testing Library](https://testing-library.com/) があります。

Testing Libraryでは要素取得に[WAI-ARIA ロールを利用することを推奨](https://testing-library.com/docs/queries/about/#priority)しています。（ロールは視覚/マウス ユーザーだけでなく、支援技術を使用するユーザーのエクスペリエンスを反映するため）


実は[WAI-ARIA ロール](https://developer.mozilla.org/ja/docs/Web/Accessibility/ARIA/Roles)には66個のロールがありますが、実際によくWeb開発で使われるUIにどのようなロールが付与されているのかを知りたくなりました。


## 記事執筆に当たり調査したこと

一般によく使われており、かつa11y対応がされているUIライブラリである[chakra](https://v2.chakra-ui.com/) の コンポーネントがどのようなロールが付与さているかを調査しました。

[chakraのコンポーネント集](https://v2.chakra-ui.com/docs/components)を見てロールをまとめるだけでもいいのですが、せっかくなので自分でchakraのコンポーネントを使ってWebアプリを作り、Testing Libraryを使って自動テストを書いてみました。

[chakraのコンポーネント集](https://v2.chakra-ui.com/docs/components)からよく使われると思われるものやロールに特徴があるもの**26個**を抜粋して作成しています。



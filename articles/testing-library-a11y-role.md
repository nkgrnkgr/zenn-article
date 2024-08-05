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

## 事例紹介

以下26個のコンポーネントに関してそれぞれ「UI」「ロールを使ったテスト」を記述します。

### Modal

## 成果物

今回の調査に当たって作成したものは以下のレポジトリに公開しています。
自動テストを書くときの参考にしていただけると幸いです。

https://github.com/nkgrnkgr/testing-library-and-a11y/

## 付録A: Chromeで[WAI-ARIA ロール](https://developer.mozilla.org/ja/docs/Web/Accessibility/ARIA/Roles)を確認する方法

ChromeのdevtoolのsettingsからExperimentsを選択し **Enable full accessibility tree view in the Styles tab** にチェックを入れます。

Elementsタブにアクセシビリティマークが表示されるようになります。

## 付録B: useIdを使って表示とlabelを紐付ける

入力要素ではない表示要素に対して何の要素なのか説明を紐付ける際に以下のように実装することで読み上げができます。

Reactでは [**useId**](https://ja.react.dev/reference/react/useId) というランダムな id を生成するHooksを提供しています。

これを使うことでa11yの紐付けが容易になります。

またテストでは **getByLabelText** を使うことで表示要素が取得できます。


コンポーネント
```tsx
const id = useId();
return (
  <div>
    <p id={id}>手数料</p>
    <p aria-labelledby={id}>250円</p>
  </div>
)

```

テスト
```tsx
const fee = screen.getByLabelText('手数料');
expect(fee.textContent).toBe('250円');

```





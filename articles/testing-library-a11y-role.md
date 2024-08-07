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

## 成果物

今回の調査に当たって作成したものは以下のレポジトリに公開しています。
自動テストを書くときの参考にしていただけると幸いです。

https://github.com/nkgrnkgr/testing-library-and-a11y/

## UI毎のロール早見表

| UI | Role |
| ---- | ---- |
| TD | TD |
| TD | TD |

## 事例紹介

以下26個のコンポーネントに関してそれぞれ「UI」「ロールを使ったテスト」を記述します。


### 警告ダイアログ

![警告ダイアログ](https://storage.googleapis.com/zenn-user-upload/d2260e98980f-20240807.gif)

- ロール：**alertdialog**

テストコード

```tsx
const user = userEvent.setup();
const button = screen.getByRole("button", {
  name: "警告ダイアログを開く",
});
await user.click(button);

const dialog = await screen.findByRole("alertdialog", {
  name: "削除の確認",
});
expect(dialog).toBeInTheDocument();
```

https://github.com/nkgrnkgr/testing-library-and-a11y/tree/main/src/components/AlertDialogDisplay

### 警告表示

![警告表示](https://storage.googleapis.com/zenn-user-upload/d137df93dd36-20240807.png)

- ロール：**alert**

テストコード
```tsx
const alert = screen.getByRole("alert");
expect(alert).toHaveTextContent("アラート");
```

https://github.com/nkgrnkgr/testing-library-and-a11y/tree/main/src/components/AlertDisplay


### パンくず

![パンくず](https://storage.googleapis.com/zenn-user-upload/6a7ab180da0f-20240807.png)

- ロール：**navigation**
- 内部のリンクのロールは**link**

テストコード

```tsx
const breadcrumb = screen.getByRole("navigation", {
  name: "breadcrumb",
});
expect(breadcrumb).toBeInTheDocument();

const items = within(breadcrumb)
  .getAllByRole("listitem")
  .map((i) => {
    const link = within(i).queryByRole("link");
    return link === null
      ? {
          text: i.textContent,
          href: "",
        }
      : {
          text: link.textContent,
          href: link.getAttribute("href"),
        };
  });
expect(items).toEqual([
  {
    text: "Home",
    href: "#",
  },
  {
    text: "一覧",
    href: "#",
  },
  {
    text: "アイテム1",
    href: "",
  },
]);
```

### チェックボックス

![チェックボックス](https://storage.googleapis.com/zenn-user-upload/e8e174337dc3-20240807.png)

- ロール：**checkbox**

テストコード
```tsx
const checkbox = screen.getByRole("checkbox", {
  name: "Checkbox",
}) as HTMLInputElement;
expect(checkbox.checked).toBe(true);
```

### コード表示


![コード表示](https://storage.googleapis.com/zenn-user-upload/8f37435ba852-20240807.png)

- ロール：**code**

テストコード
```tsx
const code = screen.getByRole("code", {
  name: "JavaScript Code!",
});
expect(code).toBeInTheDocument();
```

### 区切り線


![区切り線](https://storage.googleapis.com/zenn-user-upload/302ba55cf1bd-20240807.png)

- ロール：**separator**

テストコード
```tsx
const divider = screen.getByRole("separator", {
  name: "区切り",
});
expect(divider).toBeInTheDocument();
```

### ドロアー

TODO: ここに画像

- ロール：**dialog**

テストコード
```tsx
const user = userEvent.setup();
const button = screen.getByRole("button", {
  name: "ドロアーを開く",
});
await user.click(button);
const drawer = await screen.findByRole("dialog", {
  name: "アカウントの作成",
});
expect(drawer).toBeInTheDocument();
```

### テキスト入力


TODO: ここに画像

- ロール：**textbox**

テストコード
```tsx
const user = userEvent.setup();
const emailInput = screen.getByRole("textbox", {
  name: "Email",
}) as HTMLInputElement;
expect(emailInput.value).toBe("xxx@gmail.com");
await user.clear(emailInput);
expect(
  screen.getByText("Emailのフォーマットが正しくありません"),
).toBeInTheDocument();
```

### 見出し

TODO: ここに画像

- ロール：**heading**

テストコード
```tsx
const heading = screen.getByRole("heading", {
  name: "見出し",
});
expect(heading).toBeInTheDocument();
```

### 画像

TODO: ここに画像

- ロール：**img**

```tsx
const image = screen.getByRole("img", {
  name: "150 x 150 placeholder",
});
expect(image).toBeInTheDocument();
```

### リンク

TODO: ここに画像

- ロール：**link**

```tsx
const link = screen.getByRole("link", {
  name: "Github.com",
});
expect(link).toBeInTheDocument();
expect(link.getAttribute("href")).toBe("https://github.com");
```

### リスト

TODO: ここに画像

- ロール：**list**
- リスト内の要素は **listitem**

```tsx
const link = screen.getByRole("link", {
  name: "Github.com",
});
const list = screen.getByRole("list", {
  name: "リスト",
});
expect(list).toBeInTheDocument();
const listItems = within(list).getAllByRole("listitem");
expect(listItems).toHaveLength(3);
const textContents = listItems.map((i) => i.textContent);
expect(textContents.sort()).toEqual(
  ["アイテム 1", "アイテム 2", "アイテム 3"].sort(),
);
```

### ローディング

TODO: ここに画像

- ロール：**alert**

```tsx
const loading = screen.getByRole("alert");
expect(loading).toBeInTheDocument();
```

### メニュー

TODO: ここに画像

- ロール：**menu**
- メニュー内の要素は **menuitem**

```tsx
const user = userEvent.setup();
const button = screen.getByRole("button", { name: "メニューを開く" });
await user.click(button);

const menu = await screen.findByRole("menu");
const menuItems = within(menu).getAllByRole("menuitem");
expect(menuItems).toHaveLength(4);
const textContents = menuItems.map((i) => i.textContent);
expect(textContents.sort()).toEqual(
  ["個人設定", "購入履歴", "アカウントの切り替え", "ログアウト"].sort(),
);
```

### モーダル

![modalの画像](https://storage.googleapis.com/zenn-user-upload/9b9b3c237cf2-20240807.gif)

- ロール：**dialog**

テストコード

```tsx
const user = userEvent.setup();
const button = screen.getByRole("button", { name: "モーダルを開く" });
await user.click(button);

const dialog = await screen.findByRole("dialog");
expect(dialog).toBeInTheDocument();

const closeButton = within(dialog).getByRole("button", {
  name: "閉じる",
});
await user.click(closeButton);
await waitFor(() => {
  expect(screen.queryByRole("dialog")).toBeNull();
});
```

コード：
https://github.com/nkgrnkgr/testing-library-and-a11y/tree/main/src/components/ModalDisplay

### 数値入力

TODO: ここに画像

- ロール：**spinbutton**

```tsx
const user = userEvent.setup();
const numberInput = screen.getByRole("spinbutton", {
  name: "数値入力",
}) as HTMLInputElement;
expect(numberInput.value).toBe("10");
await user.clear(numberInput);
await user.type(numberInput, "999");
expect(numberInput.value).toBe("999");
await user.type(numberInput, "{arrowup}");
expect(numberInput.value).toBe("1000");
await user.type(numberInput, "{arrowdown}");
expect(numberInput.value).toBe("999");
```

### ポップオーバー

TODO: ここに画像

- ロール：**dialog**

```tsx
const user = userEvent.setup();
const button = screen.getByRole("button", { name: "ポップオーバーを開く" });
await user.click(button);

const dialog = await screen.findByRole("dialog", {
  name: "ポップオーバー",
});
expect(dialog).toBeInTheDocument();

const closeButton = within(dialog).getByRole("button", {
  name: "Close",
});
await user.click(closeButton);
await waitFor(() => {
  expect(screen.queryByRole("dialog")).toBeNull();
});
```


## 付録A: Chromeで[WAI-ARIA ロール](https://developer.mozilla.org/ja/docs/Web/Accessibility/ARIA/Roles)を確認する方法

ChromeのdevtoolのsettingsからExperimentsを選択し **Enable full accessibility tree view in the Styles tab** にチェックを入れます。

Elementsタブにアクセシビリティマークが表示されるようになります。

TODO: ここに画像を挿入

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

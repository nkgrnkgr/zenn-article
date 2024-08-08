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

[chakraのコンポーネント集](https://v2.chakra-ui.com/docs/components)からよく使われると思われるものやロールに特徴があるもの**25個**を抜粋して作成しています。

## 成果物

今回の調査に当たって作成したものは以下のレポジトリに公開しています。
自動テストを書くときの参考にしていただけると幸いです。

https://github.com/nkgrnkgr/testing-library-and-a11y/

## UI毎のロール早見表

| UI | Role |
| ---- | ---- |
| 警告ダイアログ | alertdialog |
| 警告表示 | alert |
| パンくず | navigation |
| チェックボックス | checkbox |
| コード表示 | code |
| 区切り線 | separator |
| ドロアー | dialog |
| テキスト入力 | textbox |
| 見出し | heading |
| リンク | link |
| リスト | list, listitem |
| ローディング | alert |
| メニュー | menu, menuitem |
| モーダル | dialog |
| 数値入力 | spinbutton |
| ポップオーバー | dialog |
| プログレスバー | progressbar |
| ラジオ | radiogroup, radio |
| セレクト | combobox |
| スタッツ | term |
| タブとコンテンツ | tab, tabpanel |
| テーブル | table, rowgroup, columnheader, row, cell |
| テキス入力(複数行) | textbox |
| トースト | - |
| ツールチップ | tooltip |

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

![ドロアー](https://storage.googleapis.com/zenn-user-upload/c11c6dd61cfd-20240808.gif)

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


![テキスト入力](https://storage.googleapis.com/zenn-user-upload/153f6ba50b9c-20240808.png)

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

![見出し](https://storage.googleapis.com/zenn-user-upload/bdb3cf964328-20240808.png)

- ロール：**heading**

テストコード
```tsx
const heading = screen.getByRole("heading", {
  name: "見出し",
});
expect(heading).toBeInTheDocument();
```

### 画像

![画像](https://storage.googleapis.com/zenn-user-upload/bcc7a8ea899d-20240808.png)

- ロール：**img**

```tsx
const image = screen.getByRole("img", {
  name: "150 x 150 placeholder",
});
expect(image).toBeInTheDocument();
```

### リンク

![リンク](https://storage.googleapis.com/zenn-user-upload/9b5ccc1ed23c-20240808.png)

- ロール：**link**

```tsx
const link = screen.getByRole("link", {
  name: "Github.com",
});
expect(link).toBeInTheDocument();
expect(link.getAttribute("href")).toBe("https://github.com");
```

### リスト

![リスト](https://storage.googleapis.com/zenn-user-upload/b3417d649041-20240808.png)

- ロール：**list**
- リスト内の要素は **listitem**

```tsx
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

![ローディング](https://storage.googleapis.com/zenn-user-upload/78f7d6ea3526-20240808.gif)

- ロール：**alert**

```tsx
const loading = screen.getByRole("alert");
expect(loading).toBeInTheDocument();
```

### メニュー

![メニュー](https://storage.googleapis.com/zenn-user-upload/cc5611999223-20240808.gif)

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

![モーダル](https://storage.googleapis.com/zenn-user-upload/2dd57388ee6d-20240808.gif)

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

![数値入力](https://storage.googleapis.com/zenn-user-upload/a4b93565b2c6-20240808.png)

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

![ポップオーバー](https://storage.googleapis.com/zenn-user-upload/a89eb52d358d-20240808.gif)

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

### プログレスバー

![プログレスバー](https://storage.googleapis.com/zenn-user-upload/b35ec9c7f44c-20240808.png)

- ロール：**progressbar**

```tsx
const progressbar = screen.getByRole("progressbar");
expect(progressbar).toBeInTheDocument();
expect(progressbar).toHaveTextContent("40%");
expect(progressbar).toHaveAttribute("aria-valuemin", "0");
expect(progressbar).toHaveAttribute("aria-valuemax", "100");
expect(progressbar).toHaveAttribute("aria-valuenow", "40");
```

### ラジオ

![ラジオ](https://storage.googleapis.com/zenn-user-upload/f4848a4aa270-20240808.png)

- ロール：**radiogroup**
- 個別の選択肢は：**radio**

```tsx
const user = userEvent.setup();
const radioGroup = screen.getByRole("radiogroup", {
  name: "ユーザー区分",
});
const radioIndividual = within(radioGroup).getByRole("radio", {
  name: "個人",
}) as HTMLInputElement;
const radioCorporation = within(radioGroup).getByRole("radio", {
  name: "法人",
}) as HTMLInputElement;
const radioEtc = within(radioGroup).getByRole("radio", {
  name: "その他",
}) as HTMLInputElement;
expect(radioIndividual.checked).toBe(true);
expect(radioCorporation.checked).toBe(false);
expect(radioEtc.checked).toBe(false);

await user.click(radioCorporation);
expect(radioIndividual.checked).toBe(false);
expect(radioCorporation.checked).toBe(true);
expect(radioEtc.checked).toBe(false);
```

### セレクト

![セレクト](https://storage.googleapis.com/zenn-user-upload/3bdd482ff7a5-20240808.gif)

- ロール：**combobox**
- 本来 combobox は入力可能なtextboxに選択肢も含まれるものを指すと思うので適切ではないのかも？
- [React Aria](https://react-spectrum.adobe.com/react-aria/index.html) だとクリックされるまでは **button** で選択肢がでてからは**listbox**になっている

```tsx
const user = userEvent.setup();
const combobox = screen.getByRole("combobox", {
  name: "ユーザー区分",
});
await user.click(combobox);
await user.selectOptions(combobox, "corporation");
expect(combobox).toHaveValue("corporation");
```

### スタッツ

![スタッツ](https://storage.googleapis.com/zenn-user-upload/6487901c196c-20240808.png)

- ロール：**term**

```tsx
const statTitle = screen.getByRole("term");
expect(statTitle).toHaveTextContent("日経平均株価");
const definitions = screen.getAllByRole("definition");
const textContents = definitions.map((d) => d.textContent);
expect(textContents).toEqual([
  "35,909.70",
  "decreased by−2,216.63 (5.81%)今日",
]);
```


### タブとコンテンツ

![タブとコンテンツ](https://storage.googleapis.com/zenn-user-upload/50f7a87f541b-20240808.gif)

- ロール：**tab**
- タブと紐付くコンテンツについては：**tabpanel**

```tsx
const user = userEvent.setup();
const tabPanel1 = screen.getByRole("tabpanel", { name: "One" });
expect(tabPanel1).toBeInTheDocument();
const tab2 = screen.getByRole("tab", { name: "Two" });
await user.click(tab2);
expect(screen.getByRole("tabpanel", { name: "Two" })).toBeInTheDocument();
```

### テーブル

![テーブル](https://storage.googleapis.com/zenn-user-upload/32ee834defc8-20240808.png)

- ロール：**table**, **rowgroup**, **columnheader**, **row**, **cell**
- **rowgroup** は ヘッダー要素とボディー要素に分けられる
- **columnheader** はヘッダー要素の中のヘッダーセル

```tsx
const table = screen.getByRole("table", { name: "日本の人口推移" });
expect(table).toBeInTheDocument();
const rowgroup = within(table).getAllByRole("rowgroup");
const headers = within(rowgroup[0])
  .getAllByRole("columnheader")
  .map((header) => header.textContent);
const yearIndex = headers.indexOf("年");
const populationIndex = headers.indexOf("人口");
const increaseRateIndex = headers.indexOf("増加率(%)");
const tableBodyRows = rowgroup[1];
const rows = within(tableBodyRows).getAllByRole("row");
const year1975Row = rows.find((row) => {
  const cells = within(row).getAllByRole("cell");
  return cells[yearIndex].textContent === "1975";
});
assertExists(year1975Row);
expect(
  within(year1975Row).getAllByRole("cell")[populationIndex],
).toHaveTextContent("1億1190万人");
expect(
  within(year1975Row).getAllByRole("cell")[increaseRateIndex],
).toHaveTextContent("34.5%");
```

### テキスト入力（複数行）

![テキスト入力（複数行）](https://storage.googleapis.com/zenn-user-upload/58397e36af92-20240808.png)

- ロール：**textbox***

```tsx
const TEXT = `
Hello
world
`;
const user = userEvent.setup();
const textarea = screen.getByRole("textbox", {
  name: "コメント",
}) as HTMLInputElement;
expect(textarea.textContent).toBe("");

await user.type(textarea, TEXT);
expect(textarea.textContent).toBe(TEXT);
```

### トースト

![トースト](https://storage.googleapis.com/zenn-user-upload/f0fb04555fb5-20240808.gif)

- chakraだと**region**は最初から表示されているので内部にテキストが出現したことでトーストが表示されたと判断している

```tsx
const user = userEvent.setup();
const button = screen.getByRole("button", { name: "トーストを表示" });
await user.click(button);

const notificationRegion = screen.getByRole("region", {
  name: "Notifications-bottom",
});
const toast = await within(notificationRegion).findByText(
  "入力に誤りがあります。",
);
expect(toast).toBeInTheDocument();
```

### ツールチップ

![ツールチップ](https://storage.googleapis.com/zenn-user-upload/1a4fc832005b-20240808.gif)

- ロール：**tooltip**

```tsx
const user = userEvent.setup();
const text = screen.getByText("ツールチップを表示");
await user.hover(text);

const tooltip = await screen.findByRole("tooltip");
expect(tooltip).toBeInTheDocument();
```

## 最後に

今回紹介したUIとロールの組み合わせはあくまでchakraの実装なので、別のライブラリであれば異なるロールが付与されていることもあります。自分でUIを実装する際はいくつかのライブラリを比較して適切なロールを選択できるとよさそうです。
UIに対して適切にロールが付与されることでし視覚／マウスユーザーだけでなく支援技術を利用するユーザーにとって使いやすいUIになります。
またテストの書きやすさという観点でも積極的にロールを利用したほうがよいです。

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

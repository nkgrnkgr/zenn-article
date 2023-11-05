---
title: "Reduxでパフォーマンスをよくするためにやったこと(createEntityAdapter編)" # 記事のタイトル
emoji: "🚀" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ['React', 'TypeScript', "Redux", "performance"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

こんにちは！サイボウズ株式会社で [kintone](https://kintone.cybozu.co.jp/) というプロダクトのフロントエンドエンジニアをやっている [Nokogiri](https://twitter.com/nkgrnkgr) です。

kintoneの画面の一部ではフロントエンドの状態管理に [ReduxToolkit](https://redux-toolkit.js.org/) を採用しています。

Reduxの採用理由は以下の記事をご参照ください

https://blog.cybozu.io/entry/2023/04/20/190000

今回は kintone のフロントエンド開発でReduxアプリのパフォーマンスをよくするためにやったことについて紹介します。

## 前提

Reduxアプリのベストプラクティスについてはこのページに記述されてあります。今回パフォーマンスを改善するための取り組みも以下のプラクティスをもとに行いました。より詳細を確認したい方は公式サイトをご覧ください。

https://redux.js.org/style-guide/

## 施策と今回紹介すること

大きくわけて２つの施策を取り入れています。

- 配列の代わりにcreateEntityAdapterを使う
- selectorのメモ化をつかって最適化する

今回の記事では「配列の代わりにcreateEntityAdapterを使う」をご紹介したいと思います。
selectorのメモ化については別記事に分けて紹介しようと思います。

## 配列の代わりにcreateEntityAdapterを使う

[createEntityAdapter](https://redux-toolkit.js.org/api/createEntityAdapter) とは　正規化された状態の構造に対してCRUD操作を行うためのreducerやselectorを生成する機能です。

どのような課題があって、それをどう改善したか見ていきます。

### 配列の一部を更新すると全体が再レンダリングされる

n 件あるデータをループを回して表示する場合配列を利用することが多いと思います。

```tsx
type Item = {
  itemId: number;
  name: string;
  published: string;
  author: string;
};

const ListItem = ({ item }: { item: Item }) => {
  return (
    <li>
      <div> Name : {item.name} </div>
      <div> published: {item.name} </div>
      <div> author : {item.name} </div>
    </li>
  );
};

const List = () => {
  const items = useSelector(state => state.items)
  return (
    <ul>
      {items.map((item) => (
        <ListItem item={item} key={item.itemId} />
      ))}
    </ul>
  );
};
```

`items` が不変であれば問題ないのですが `items` の中身が可変でかつ `ListItem` から更新される場合にパフォーマンス上の問題があります。

例えば ListItem の中で以下のように `name` を変更する仕様があった場合、`name` が変更されるたびに `List` コンポーネント全体が *再レンダリング* されます。自ずと子供の component もすべて再レンダリングされます。

```tsx
const ListItem = ({ item }: { item: Item }) => {
  const dispatch = useDispatch();
  const changeValue = (value: string) => {
    dispatch(actions.changeNameValue({itemId: item.id, name: value }));
  };

  return (
    <li>
      <div>
        Name :
        <input
          type="text"
          onChange={(e) => changeValue(e.target.value)}
          value={item.name}
        />
      </div>
      <div> published: {item.name} </div>
      <div> author : {item.name} </div>
    </li>
  );
};
```

React Developer Tools の　「Highlight updates when components render.」 で確認可能です

![gif](https://storage.googleapis.com/zenn-user-upload/cf43562ed83a-20231105.gif)

これは `useSelector` で参照している `items` に変更が入るため、`items`を参照している `List` 全体が再レンダリングされるのが原因です。

このような事例に対処するには 一つ一つの要素で `memo` などを使ってキャッシュすることが一般的ですが、今回のように親の `List` が `items` 全体を参照していることで発生する問題のためこの方法は利用できません。

### 再レンダリングを防ぐためにやること

以下の変更を行うことで再レンダリングを防ぐことにします。

- List は `items` 全体を参照するのをやめ `item` を一意に識別できるキー(itemId)だけを参照する
- Item は List から `item` ではなく `itemId` だけをもらい `selector` を利用して `item` を参照するようにする
- `item` を一意に識別できるキーと `item` を別のデータ構造として個別に管理する

上記の修正を List のまま行うこともできるのですが、キーと実態を別のデータ構造として管理すると要素の追加更新時に二重メンテになるなどデメリットも多いです。パフォーマンスを良くするためとはいえ複雑さをどこまで許容するかはトレードオフになります。

### `createEntityAdapter` を使って対処する

`createEntityAdapter`はこのような問題を解決するためのインターフェースを備えています。

`createEntityAdapter`を使うことでもともと配列だったデータ構造は以下のようなデータ構造になります。
`ids` は その要素を一意に識別できるキー
`entities` は `id` をキーにしており `item` 自体を持つ

```tsx
{
  ids: [1, 2, 3],
  entities: {
    "1": {
      itemId: 1,
      name: "DragonBallabcde",
      published: "1984",
      author: "Akira Toriyama",
    },
    "2": {
      itemId: 2,
      name: "Yuyuhakusho",
      published: "1990",
      author: "Yoshihiro Togashi",
    },
    "3": {
      itemId: 3,
      name: "SLAM DUNK",
      published: "1990",
      author: "Takehiko Inoue",
    },
  },
};
```

`List` コンポーネントで `ids` を参照し、`ListItem` コンポーネントで `id` を使って `entities` から取得することで親と子が同じデータ構造を参照することを防ぎます。

もちろん要素自体が追加削除された場合は `ids` に変更があるため再レンダリングは発生します。実際にリストの要素が増えたり減ったりするなら妥当な再レンダリングだと判断できます。

`createEntityAdapter` を使った実装の場合は以下のように再レンダリングを防いでいます。

![entity](https://storage.googleapis.com/zenn-user-upload/5a5f4b1c0d88-20231105.gif)

### `createEntityAdapter` を使った具体的な実装

`state` のデータ構造は以下のように変わります。

```ts
// old
type State = {
  items: Item[]
}

// new
type State = {
  items: EntityState<Item>
}
```

`adapter` と `selector` を生成します。

```tsx
export const itemAdapter = createEntityAdapter<Item>({
  selectId: (model) => model.itemId,
});

export const itemSelector = itemAdapter.getSelectors<RootState>(
  (state) => state.example.items
);
```

`List` コンポーネントは `itemSelector#selectIds` を使って `state` から `ids` だけを参照します。 だけを参照します。

```tsx
const List = () => {
  const itemIds = useSelector(itemSelector.selectIds);
  return (
    <ul>
      {itemIds.map((id) => (
        <ListItem itemId={Number(id)} key={id} />
      ))}
    </ul>
  );
};

```

`ListItem` 側では `itemSelector` を使って `itemId` から `item` を参照します

```tsx
  // ListItem
  const item = useSelector((state: RootState) => itemSelector.selectById(state, itemId)) // item | undefined
```

`item` は理論上存在するはずなので `assert` してあげてもよいです。


## あとがき

今回は、配列の代わりに`createEntityAdapter`を使ってReactアプリで再レンダリングを防ぐ仕組みを紹介しました。
途中でも触れましたが、この実装を取り入れることでコード量は増え、複雑性は上がっておりパフォーマンスとトレードオフになっています。
`ListItem` の中身が可変でかつ再レンダリングを抑えたい場合にこの実装は有用だと考えています。

次回は「selectorのメモ化をつかって最適化する」を紹介したいと思います。


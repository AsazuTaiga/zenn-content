---
title: "注意して。<a>で<button>をラップしてはいけない"
emoji: "⛔️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [react,nextjs,html5]
published: true
---


## 2021.08追記

もともとこの記事のタイトルは「注意して。`<Link>`コンポーネントで`<button>`をラップしてはいけない」でした。
しかし、厳密には各ライブラリによって`<Link>`コンポーネントのAPIが異なる点に対して、把握しきれていない部分もあるなと思い直しましたので（ライブラリ独自の対処法とかいろいろありそう＆今後出てきてもおかしくない）、タイトルを変更しました。

他の記事や公式ドキュメントもご参照いただけますと、バランスの取れた現実的な対処が可能になる気がします。
（関連記事や修正の私的があればコメント・PRください！）

- 関連

https://nextjs.org/docs/api-reference/next/link#if-the-child-is-a-custom-component-that-wraps-an-a-tag

https://ja.nuxtjs.org/docs/2.x/features/nuxt-components#the-nuxtlink-component

https://reactrouter.com/web/api/Link

https://router.vuejs.org/ja/api/#router-link

また、こちらの記事で、Material UIのButtonとNext.jsのLinkを組み合わた際の問題と対処法が記載されていたので、シェアします。

https://qiita.com/ainehanta/items/44fe664b4b2b0adf213b


--- 

## motivation

`<Link>`コンポーネントの誤った使い方をおすすめするQ&Aが意外と多く、ハマりかけたためです。

## TL;DR

- a要素はbutton要素などのインタラクティブコンテンツを子要素にとることができない
- 今のところ多くのルーティングライブラリで`<Link>`は`<a>`をレンダリングする
 - レンダリングするコンポーネントをちゃんと指定する場合は、その限りではないです
- 画面遷移で`<Link><button></button></Link>`を使うのはやめましょう

## 確かに画面遷移はできるけど

- 多くのルーティングライブラリ(React Router、Vue Router、あるいはNext.jsなど)に用意されている`<Link>`コンポーネントは、ブラウザのHistory API[^1]をラップすることで、アプリ内での画面遷移をサポートしています
- 大抵は名前通りのリンクっぽい見た目なのですが、ケースによっては「ボタンを押して画面遷移したい！」という欲求が生まれます。
- また、このボタンはその他のボタン（クリックイベントで何らかの処理をするボタン）とスタイリングを共通化したいパターンがよくあります。（リンクボタンとその他のボタンのスタイルを似せてしまうことがUI表現として適切なのか、という議論はさておき）
- `<Link>`コンポーネント自体は、子要素をとってレンダリングできるような作りになっているので、「スタイリングしてある`<button>`を`<Link>`の中にぶちこめば終わりじゃない？」という邪な欲求が発生します。
- しかし、これには思わぬ落とし穴があります。それは、「`<Link>`コンポーネントは大抵の場合、最終的に`<a>`をレンダリングする、ということです。

## なぜ`<a>`の子要素に`<button>`を入れてはいけないか

### HTMLの仕様上の問題

- a要素のコンテンツモデル[^2]は以下のように定義されています。

> 透過的であるが、インタラクティブコンテンツの子孫、a要素の子孫、またはtabindex属性が指定された子孫が存在してはならない。
> （HTML Standard 日本語訳 4.5.1 a要素<https://momdo.github.io/html/text-level-semantics.html#the-a-element>より）

- ここで重要なのが、「インタラクティブコンテンツの子孫」を取れないという記述です。
- `<button>`はインタラクティブコンテンツ[^3]なので、`<a>`の子要素として含めてはならないよ、ということです。

### 実際に利用者に対して発生する不都合

- Tabキーでフォーカスを移動する際に、`<a>`と`<button>`のそれぞれに対してフォーカスが当たってしまう。（`<button>`に`tabindex="-1"`を指定すれば回避はできますが、悪趣味です）
- 上記で`<button>`側にフォーカスが当たっている時にEnterキーを押下しても、当然画面遷移は起こりません

## じゃあ、リンク的な動きをするボタンをどうやって実現するの？

- 普通に`useHistory`や`useRouter`を使いましょう。
- もしくは、`<Link>`に対してボタンっぽいスタイルを当てましょう。

[^1]: https://developer.mozilla.org/ja/docs/Web/API/History_API

[^2]: コンテンツがその要素の子や子孫として含めなければならないものの、規範的な記述。https://momdo.github.io/html/dom.html#concept-element-content-model

[^3]: https://momdo.github.io/html/form-elements.html#the-button-element

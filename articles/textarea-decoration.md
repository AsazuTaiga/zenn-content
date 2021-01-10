---
title: "YouTubeのコメント欄っぽい装飾をtextarea要素に付ける方法"
emoji: "🗳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["html", "css"]
published: false
---

# TL;DR

- `<textarea>`や`<input>`は仕様的に擬似要素` ::after`、`::before `を付与されてもスタイリングして表示することができません。
- 方法 1: 擬似要素を諦めて、container で囲って`span`で装飾しましょう。
- 方法 2: `<textarea>`を諦めて、`<div contenteditable="true">`を使いましょう。
- 方法 3: Custom Elements の利用を検討しましょう。

# やりたいこと

![YouTubeのコメント欄のフォーカス挙動](https://storage.googleapis.com/zenn-user-upload/g1w2w6j3gnk94y4ej061bhkxrxps)

gif のフレーム数が少なくて伝わりづらいですが、フォーカスと同時に、下の線が中央から左右へと向けて黒く変化するアニメーションが付与されています。

# 検討 1: `border-bottom`プロパティによるアニメーション

「下線が表示される」というアニメーションを実装するにあたって、まず考えることは`border-bottom`プロパティの利用です。
しかし、実際に試せばすぐにわかることですが、この道は呆気なく断たれます。

@[jsfiddle](https://jsfiddle.net/asaszutaiga/gun0zt4x/28/)

`border-bottom`は、

- `border-bottom-color`
- `border-bottom-width`
- `border-bottom-style`
  の 3 つのプロパティをまとめて指定できるプロパティです。
  …つまり、`transition`プロパティで`border-bottom`をアニメーションさせると、色と太さのみですから、長さを動的に変えることはできません。

:::message

ちなみに、`border-bottom-style`はそもそも`transitionable`（トランジション可能）な値ではありません。`transitionable`の定義は以下の通りです。

> ...the property values are transitionable if they have an animation type that is neither not animatable nor discrete. https://www.w3.org/TR/2018/WD-css-transitions-1-20181011/#transitionable
>
> 筆者訳：所与のプロパティの値が`transitionable`であると言われるのは、そのプロパティの`animation type`が`not animatable`でも`discrete`でもない場合です。

[`border-style`の仕様](https://www.w3.org/TR/2020/CR-css-backgrounds-3-20201222/#border-style)を参照すると、`animation type`は`discrete`となっています。そのため、`transition`プロパティを適用できないわけです。

:::

# 検討 2: ` ::before``::after `擬似要素によるアニメーション

直接スタイルをアニメーションさせることができないとなると、第二候補は`::before`や`::after`といった擬似要素の利用になります。これも実際に試してみましょう。まずは`::before`で擬似要素として下線を足してみます。

@[jsfiddle](https://jsfiddle.net/asaszutaiga/19e5byo3/6/)

……ご覧の通りですが、`textarea`要素に対して擬似要素を表示させようとしても、うまくいきません。これはなぜでしょうか？
これを知るためには、`textarea`および擬似要素の仕様を知る必要がありそうです。（といっても、普段から HTML・CSS を触っている人には至極当たり前の内容を再確認するだけなのですが……）

まず、`::before`擬似要素から。

> ::before
> Represents a styleable child pseudo-element immediately before the originating element’s actual content.
>
> 筆者訳：スタイリング可能な子擬似要素を元の要素の実際のコンテンツの直前に表示します。
> (https://www.w3.org/TR/2020/WD-css-pseudo-4-20201231/#selectordef-before)

ここで重要なのは、"actual content"です。要は、擬似要素が表示されるのは、指定された要素の内容の中でいちばん最初の部分になる……ということです。（`::before`を使っている人には当たり前の常識でしょうが、実際には「コンテンツの直前」に入るのだという定義がされていることを確認しておきたかったまで）

次に、`textarea`の仕様を確認してみましょう。

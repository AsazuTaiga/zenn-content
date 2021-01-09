---
title: "YouTubeのコメント欄っぽい装飾をtextarea要素に付ける方法"
emoji: "🗳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["html", "css"]
published: false
---

# TL;DR

- `<textarea>`や`<input>`は仕様的に擬似要素` ::after`、`::before `を許容しません。
- 方法 1: 擬似要素を諦めて、container で囲って`span`で装飾しましょう。
- 方法 2: `<textarea>`を諦めて、`<div contenteditable="true">`を使いましょう。
- 方法 3: Custom Elements の利用を検討しましょう。

# やりたいこと

![YouTubeのコメント欄のフォーカス挙動](https://storage.googleapis.com/zenn-user-upload/g1w2w6j3gnk94y4ej061bhkxrxps)

gif のフレーム数が少なくて伝わりづらいですが、フォーカスと同時に、下の線が中央から左右へと向けて黒く変化するアニメーションが付与されています。

# 検討１：`border-bottom`プロパティによるアニメーション

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

> 筆者訳：所与のプロパティの値が`transitionable`であると言われるのは、そのプロパティの`animation type`が`not animatable`でも`discrete`でもない場合です。

[`border-style`の仕様](https://www.w3.org/TR/2020/CR-css-backgrounds-3-20201222/#border-style)を確認するとわかるように、`border-style`の`animation type`は`discrete`となっているので、`transition`プロパティを適用できないわけです。

:::

---
title: "AbortController"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [javascript]
published: false
---

# TL;DR

https://developer.mozilla.org/en-US/docs/Web/API/AbortController

# AbortControllerとは？

- HTTPリクエストを中止することができる標準API
- IE以外はだいたい対応

# どうやって使う？

```js
const ac = new AbortController()
const signal = ac.signal

const fetchApi = async () => {
  try {
    const res = fetch('https://hogehoge.com', { signal })
    console.log(res)
  } catch (e) {
    console.error(e.message)
  }
}

ac.abort()
fetchApi()
// result: error
```


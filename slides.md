---
theme: default
# https://unsplash.com/ja/%E5%86%99%E7%9C%9F/%E7%B4%AB%E9%BB%84%E8%89%B2%E3%82%AA%E3%83%AC%E3%83%B3%E3%82%B8%E8%89%B2-DKDFBtmZSz8
background: /assets/gradation-background.jpg
highlighter: shiki
lineNumbers: false
title: location-stateのすゝめ
info: |
  ## location-stateのすゝめ
  
  モダンフレームワークで損なわれたブラウザバック体験とその改善
class: text-center
drawings:
  persist: false
transition: slide-left
mdc: true
---

# location-stateのすゝめ

モダンフレームワークで損なわれたブラウザバック体験とその改善

---

# Profile

- Name: 佐藤 昭文(X: [akfm_sato](https://x.com/akfm_sato))
  - Next.js
  - Rust
  - テスト設計
  - チーム開発
- Activity
  - https://zenn.dev/akfm
  - [JS Conf 2023](https://main--remarkable-figolla-a694f0.netlify.app/1)
  - [Vercel meetup](https://zesty-basbousa-04576f.netlify.app/1)
  - [Node学園42](https://youtu.be/ONMIjHfitHM?t=9139)
  - [Rust入門本の執筆](https://www.shuwasystem.co.jp/book/9784798067315.html)

---

# Agenda

本日の主題

- ブラウザバック体験は非常に重要なUX
- Reactの`useState`の状態は復元されない
- `location-state`というライブラリを使えば簡単に復元を実装できる

---
layout: section
---

# ブラウザバックを考える

---

# ブラウザバックの重要性

ブラウザが提供する重要機能とは何だろう

- 開発者目線
  - html/CSS/JavaScriptのサポート
  - ブラウザのバグ、固有の挙動
  - 開発者ツール
  - etc...
- ユーザー目線
  - URL入力
  - ブラウザバック・フォワード
  - ブックマーク
  - プライバシー保護

---
layout: fact
---

## ユーザー目線で意外と必須と感じるものは少ない

---
layout: fact
---

## ブラウザバックはユーザーにとって当たり前(=重要)な機能

---

# ブラウザバックの仕様

[ブラウザバックの仕様](https://html.spec.whatwg.org/multipage/browsing-the-web.html#traverse-the-history-by-a-delta)はブラウザによって異なる点が多い

- bfcache(`"blazingly fast" cache`とも言うらしい)
- DOMの復元
- formの復元
- キャプチャらしきものの表示
- リロード条件

---
layout: fact
---

## 一貫して言えるのは<br>ユーザーは「戻れる」ことを期待してるということ

---
layout: fact
---

## 「戻れる」とは何か？

---
transition: fade
---

# 「戻れる」とは

ページが完全に復元されることである

- ページの内容
- スクロール位置
- 操作した内容
  - formの入力内容
  - モーダルの表示状態
  - etc...

---

# _リッチなWebアプリケーションのための7つの原則_

https://yosuke-furukawa.hatenablog.com/entry/2014/11/14/141415#5

> "戻る"のは素早く行われるべきだ。
> 
> ユーザーは"戻る"事によってデータが変更されるような事は期待していない。

---
layout: fact
---

## 現在最も忠実に復元するのは<br>bfcacheが適用された場合である

---
layout: fact
---

## しかしモダンなフレームワークでは<br>遷移をJavaScript制御で行っている

---
layout: section
---

# モダンフレームワークと<br>ブラウザバック

---

# Soft Navigation

様々な呼称があるが、ここではNext.jsに習ってSoft Navigationと呼称する

- ページ遷移がJavaScriptで制御される
- bfcacheは当然ながら適用できない
- ブラウザバック時の動作はフレームワークの仕様に強く依存する

---

# React系フレームワークのブラウザバック

ReactやViewなどの宣言的UIライブラリをベースとしたフレームワークについて考える

- `useState`で定義した状態は、ページ遷移時に破棄されるため復元できない
  - `useState`はコンポーネントのアンマウント時に状態が破棄される
  - React系フレームワークでは、ページ遷移時にページコンポーネントをマウントする
  - 前のページコンポーネントはアンマウントされる

---
layout: fact
---

## じゃあグローバルな状態にすれば破棄されないのでは？

---
layout: section
---

# 状態のスコープと<br>ライフタイム

---



TBW

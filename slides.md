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

[ブラウザバックの仕様](https://html.spec.whatwg.org/multipage/browsing-the-web.html#traverse-the-history-by-a-delta)
はブラウザによって異なる点が多い

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
- Global Stateはページ間で共通のため、ページ遷移ごとにリセットしたい時に向かない
    - e.g. モーダルの開閉状態を共有してると、ページ遷移時にリセットすることになりブラウザバック時に復元されない
    - e.g. `/hoge`にあるアコーディオンを開いて`/fuga`へ遷移、`/fuga`からリンク押下で再度`/hoge`へ遷移するとアコーディオンが開いた状態で表示される

---

# Soft Navigation時の理想

ここまでの情報も踏まえ、Soft Navigation時の理想の体験を考える

- 履歴単位で状態が保存され、Soft Navigation時にはリセットされる
- ブラウザバック・フォワード時には復元される
- リロード時には復元される

---
layout: section
---

# 状態のスコープと復元

---

# Global State vs Local State

Global StateとLocal Stateを比較する

|              | 画面遷移時にリセット | ブラウザバック時の復元 | リロード時の復元 |
|--------------|------------|-------------|----------|
| Local State  | ✅          | ❌           | ❌        |
| Global State | ❌          | 🤔          | ❌        |

<br>

<v-click>-> スコープの話と復元の話は別物</v-click>

---

# 状態のスコープと保存・復元

状態のスコープ観点に加え、保存・復元を観点に追加する

|                                                                                        | 画面遷移時にリセット | ブラウザバック時の復元 | リロード時の復元 |
|----------------------------------------------------------------------------------------|------------|-------------|----------|
| Local State                                                                            | ✅          | ❌           | ❌        |
| <span v-mark="{ at: 1, color: 'orange', type: 'underline'}">Local State+Storage</span> | ✅          | ✅           | ✅        |
| <span v-mark="{ at: 1, color: 'orange', type: 'underline'}">Local State+URL</span>     | ✅          | ✅           | ✅        |
| Global State                                                                           | ❌          | 🤔          | ❌        |
| Global State+Storage                                                                   | ❌          | 🤔          | ✅        |
| Global State+URL                                                                       | ❌          | 🤔          | ✅        |

---
layout: fact
---

## 理想のSoft Navigationに必要なのは<br>`LocalStorage\+(URL|SessionStorage)`

---
layout: fact
---

## しかしこれを実装するのは実は大変

---
layout: section
---

# 自前実装の課題

---

# 要件を整理する

理想のSoft Navigation体験を実現するための実装要件を整理する

- 機能要件
  - Navigation前に状態を保存
  - Navigation後に状態を復元、なければ初期値
  - 状態の保存はURLかSessionStorageに保存したい
- パフォーマンス要件
  - 再レンダリングは最小限
  - Layout Shiftは起こしたくない
  - バンドルサイズは極力増やしたくない
- 開発者要件
  - 冗長な実装は避ける
  - できるだけ`useState`などと類似したAPI
  - Next.jsと簡単に連携したい
- ...

---

# 実装上の課題

前述の要件を満たす実装を自分で行おうとすると、以下のような課題がある

- フレームワークに依存せず、どうやってNavigation検知するか
- いつどうやって値を保存するか
- いつどうやって値を参照するか
- 保存した値のマイグレーションをどうやってやるか
- レンダリング回数をどうやって最適化すべきか

---
layout: fact
---

## 自前で実装する大変さが伝わっただろうか

---
layout: fact
---

## 実装負担少なく、理想のSoft Navigationを実現する方法はないか

---
layout: section
---

# location-stateのすゝめ

---

TBW

---
layout: section
---

# location-stateのその他機能

---

TBW

---
layout: section
---

# まとめ

---

TBW

---

# Memo

- 状態にはスコープとライフタイムがある



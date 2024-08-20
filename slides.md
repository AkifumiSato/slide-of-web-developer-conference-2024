---
theme: default
# https://unsplash.com/ja/%E5%86%99%E7%9C%9F/%E7%B4%AB%E9%BB%84%E8%89%B2%E3%82%AA%E3%83%AC%E3%83%B3%E3%82%B8%E8%89%B2-DKDFBtmZSz8
background: /gradation-background.jpg
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

- bfcache(back forward cache)
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

# 「戻れる」とは

ページが完全に復元されることである

- ページの内容
- スクロール位置
- 操作した内容
  - formの入力内容
  - モーダルの表示状態
  - etc...

---

# リッチなWebアプリケーションのための7つの原則

https://yosuke-furukawa.hatenablog.com/entry/2014/11/14/141415#5

> _"戻る"のは素早く行われるべきだ。_
>
> _ユーザーは"戻る"事によってデータが変更されるような事は期待していない。_

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

様々な呼称があるが、ここではNext.jsに習って[Soft Navigation](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#5-soft-navigation)と呼称する

- ページ遷移がJavaScriptで制御される
- bfcacheは当然ながら適用できない
- ブラウザバック時の動作はフレームワークの仕様に強く依存する

---

# React系フレームワークのブラウザバック

ReactやVueなどの宣言的UIライブラリをベースとしたフレームワークについて考える

- `useState`で定義した状態は、ページ遷移時に破棄されるため復元できない
  - `useState`はコンポーネントのアンマウント時に状態が破棄される
  - React系フレームワークでは、ページ遷移時にページコンポーネントをマウントする
- Global Stateはページ間で共通のため、ページ遷移ごとにリセットしたい時に向かない

---

# Soft Navigation時の理想

ここまでの情報も踏まえ、Soft Navigation時の理想の体験を考える

- 履歴単位で状態が保存される
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
| ------------ | -------------------- | ---------------------- | ---------------- |
| Local State  | ✅                   | ❌                     | ❌               |
| Global State | ❌                   | ❌(条件次第で✅)       | ❌               |

<br>

<span v-mark="{ at: 1, color: 'red', type: 'underline'}" class="bold">スコープの話と復元の話は別物</span>

---
layout: fact
---

## 欲しいのは履歴位置ごとに復元するState

---

# 状態のスコープと保存・復元

状態のスコープ観点に加え、保存・復元を観点に追加する

|                                                                                         | 画面遷移時にリセット | ブラウザバック時の復元 | リロード時の復元 |
| --------------------------------------------------------------------------------------- | -------------------- | ---------------------- | ---------------- |
| Local State                                                                             | ✅                   | ❌                     | ❌               |
| <span v-mark="{ at: 1, color: 'orange', type: 'circle'}">Local State+Restoration</span> | ✅                   | ✅                     | ✅               |
| Global State                                                                            | ❌                   | ❌(条件次第で✅)       | ❌               |
| Global State+Restoration                                                                | ❌                   | ❌(条件次第で✅)       | ✅               |

---
layout: fact
---

## 理想のSoft Navigationに必要なのは<br>Local State+Restoration

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
  - Next.jsと簡単に連携したい

...

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

## 実装負担少なく、<br>理想のSoft Navigationを実現する方法はないのだろうか

---
layout: section
---

# location-stateのすゝめ

---

# location-stateとは

理想のSoft Navigationを実現するためのライブラリ

- https://www.npmjs.com/package/@location-state/core
- <Link to="25">前述の要件</Link>に対応
- Next.jsを中心に対応中
- (ユニークな`name`を必要とするので、厳密にはLocal Stateではない)

---
transition: fade
---

# location-stateのセットアップ

https://www.npmjs.com/package/@location-state/core

```shell
$ npm i @location-state/core
$ pnpm add @location-state/core
```

---
transition: fade
---

# location-stateのセットアップ

```tsx
// src/app/providers.tsx
"use client";

import { LocationStateProvider } from "@location-state/core";

export function Providers({ children }: { children: React.ReactNode }) {
  return <LocationStateProvider>{children}</LocationStateProvider>;
}
```

---

# location-stateのセットアップ

```tsx
// src/app/layout.tsx
import { Providers } from "./Providers";

// ...snip...

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

---

# location-stateの基本的な使い方

````md magic-move
```tsx
"use client";

import { useState } from "react";

export function Counter() {
  const [counter, setCounter] = useState(0);

  return (
    <div>
      <p>
        storeName: <b>{storeName}</b>, counter: <b>{counter}</b>
      </p>
      <button onClick={() => setCounter(counter + 1)}>increment</button>
    </div>
  );
}
```

```tsx {all|6-10}
"use client";

import { useLocationState } from "@location-state/core";

export function Counter() {
  const [counter, setCounter] = useLocationState({
    name: "counter", // ユニークな状態名
    defaultValue: 0, // 初期値
    storeName: "session", // 保存先(デフォルトで選べるのは`"session"|"url"`)
  });

  return (
    <div>
      <p>
        storeName: <b>{storeName}</b>, counter: <b>{counter}</b>
      </p>
      <button onClick={() => setCounter(counter + 1)}>increment</button>
    </div>
  );
}
```
````

---

# Demo

実際にlocation-stateのリポジトリ内にある`examples`で動作を確誽

- [basic](https://github.com/recruit-tech/location-state/tree/main/apps/example-next-basic)
- [unsafe-navigation](https://github.com/recruit-tech/location-state/tree/main/apps/example-next-unsafe-navigation)

---
layout: section
---

# 付録:<br>location-stateのその他機能

---

# Next.jsサポート

Next.jsを積極的にサポート

- [Pages Routerサポート用のパッケージ](https://www.npmjs.com/package/@location-state/next)
- `examples`でNext.jsのサンプルを提供・テストを実施

---

# 多彩なカスタマイズ性

Navigation連携、保存・復元処理を抽象化しているため、多彩なカスタマイズが可能

- [Syncer](https://github.com/recruit-tech/location-state/blob/main/packages/location-state-core/docs/API.md#Syncer): Navigation連携のInterface
- [Store](https://github.com/recruit-tech/location-state/blob/main/packages/location-state-core/docs/API.md#store): 保存・復元処理のInterface
- [Refine](https://github.com/recruit-tech/location-state/blob/main/packages/location-state-core/docs/API.md#type-Refine): 復元時のValidation Interface

---

# Conform連携

App Routerと相性のいいFormライブラリである[conform](https://conform.guide/)との連携もサポート

- https://www.npmjs.com/package/@location-state/conform
- [example](https://github.com/recruit-tech/location-state/tree/main/apps/example-next-conform)

---
layout: section
---

# 付録:<br>location-stateのリポジトリ構成

---

# モダンなツールや構成

location-stateのツールチェーンは積極的にモダンな構成に維持してる

- pnpm monorepo
- [biome](https://biomejs.dev/)
- turborepoによるCI高速化
- renovateによる自動バージョンアップ
- [tsup](https://tsup.egoist.dev/)

---

# RenovateとCIによる依存関係の自動アップデート戦略

単体テストはもちろん、examplesに対しPlaywrightテストも実装、CIでチェック

1. Renovateが依存ライブラリのバージョンアップのプルリクを作成
2. CIでlint、type check、単体テスト、Playwrightテストを実行
3. オールグリーンでrenovate-approveがプルリクを承認、マージ

https://github.com/recruit-tech/location-state/pull/445

---

# changesetsによるリリース自動化

packageリリース管理ツールのchangesetsを導入

- [release PR](https://github.com/recruit-tech/location-state/pull/406)
- [release note](https://github.com/recruit-tech/location-state/releases/tag/%40location-state%2Fnext%401.2.1)
- [publish](https://github.com/recruit-tech/location-state/actions/runs/9639536391/job/26581864673)

---
layout: section
---

# まとめ

---

# location-stateのすゝめ まとめ

冒頭述べたAgendaの再掲

- ブラウザバック体験は非常に重要なUX
- Reactの`useState`の状態は復元されない
- `location-state`というライブラリを使えば簡単に復元を実装できる

---
layout: fact
---

## 今日話を聞いて、ブラウザバックを気にする人が<br>少しでも増えてくれたら嬉しいです

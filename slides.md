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

- Name: 佐藤 昭文（Akifumi Sato）
  - Web Engineer(Frontend中心)
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
- React(他も同様)の`useState`の状態は復元されない
- `location.state`というライブラリを使えば簡単に復元を実装できる

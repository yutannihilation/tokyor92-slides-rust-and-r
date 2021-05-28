---
title: "RパッケージでRustを使うには: extendr入門"
output:
  md_document:
    variant: "markdown_github"
    preserve_yaml: true

defaults:
  class: 'font-medium'

theme: default
class: 'text-center'
background: ./images/top.gif
highlighter: shiki
info: |
  ## Slides for Tokyo.R#92
  
  [第92回R勉強会@東京](https://tokyor.connpass.com/event/213524/)のスライドです 
---

# Rパッケージで<br/>Rustを使うには:<br/> extendr入門

Tokyo.R\#92

Hiroaki Yutani (@yutannihilation)

---
layout: image-right
image: './images/icon.png'
---

# ドーモ！

## <mdi-twitter style="display: inline"/>:

@yutannihilation

## 好きな言語:

R、Rust、忍殺語

## 最近の趣味:

ガスコンロの電子楽器をつくってます

---
layout: image-right
image: './images/book.jpg'
---

# RユーザーのためのRStudio\[実践\]入門 第2版！

------------------------------------------------------------------------

# extendrとは？

-   RustとRを連携させるためのフレームワーク
-   RからRustを使うだけではなく、RustからRを使うこともできる（つまり、Rustの中でggplot2を呼び出してプロットしたり、とかできるらしい）

<img src="images/extendr-logo.png" width="200" />

---
layout: statement
---

# なぜRust？

<v-clicks>

→ そこにRustがあるから<br>（誰か教えてください…）

</v-clicks>

------------------------------------------------------------------------

# extendrの愉快な仲間たち

## libR-sys（Rust）:

RのC
APIに[bindgen](https://github.com/rust-lang/rust-bindgen)で生成したバインディング

## extendr（Rust）:

libR-sysを使いやすくラップしたフレームワーク

## rextendr（Rパッケージ）:

Rからextendrを使う際のヘルパーパッケージ（usethisパッケージのような立ち位置）

---
layout: section
---

# 準備

------------------------------------------------------------------------

# Rustのインストール

## macOS / Linux:

ふつうにRustをインストール（ググってください）

## Windows

MSVCのtoolchainに加えて、64bit/32bit GNU用のtargetを追加する必要がある

``` bash
rustup default stable-msvc
rustup target add x86_64-pc-windows-gnu
rustup target add i686-pc-windows-gnu
```

------------------------------------------------------------------------

# rextendrパッケージのインストール

GitHubからインストール

``` r
devtools::install_github("extendr/rextendr")
```

---
layout: section
---

# パッケージ作成

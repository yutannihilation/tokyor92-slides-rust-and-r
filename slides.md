---
title: "RパッケージでRustを使うには: extendr入門"
output:
  md_document:
    variant: "markdown_github"
    preserve_yaml: true

defaults:
  class: 'font-medium'

theme: seriph
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

<img src="/public/images/extendr-logo.png" width="200" />

---
layout: statement
---

# なぜRust？

<v-clicks>

→ そこにRustがあるから！！<br>（誰か教えてください…）

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

-   ふつうにRustをインストール（ググる）

## Windows

-   MSVCのtoolchainに加えて、64bit/32bit
    GNU用のtargetを追加する必要がある

``` bash
rustup default stable-msvc
rustup target add x86_64-pc-windows-gnu
rustup target add i686-pc-windows-gnu
```

------------------------------------------------------------------------

# rextendrパッケージのインストール

-   GitHubからインストール

``` r
devtools::install_github("extendr/rextendr")
```

---
layout: section
---

# パッケージのセットアップ

------------------------------------------------------------------------

# RStudioからパッケージ作成

<div class="container content-center justify-center">

<img src="/public/images/screenshot_1_create_package.png" style="width:70.0%;height:70.0%" />

</div>

------------------------------------------------------------------------

# Roxygenを使うように設定変更

## NAMESPACEを上書き

``` r
usethis::use_namespace()
```

## Build optionsを設定

*Build \> Configure Build Tools… \> Generate documentation with Roxygen*
に<raphael-checked style="display: inline"/>を入れる

## 不要なファイルを削除

-   `R/hello.R`
-   `man/hello.Rd`

------------------------------------------------------------------------

# extendrのデフォルト設定を生成

``` r
rextendr::use_extendr()
```

<v-clicks>

``` r
✓ Creating src/rust/src.
✓ Writing 'src/entrypoint.c'
✓ Writing 'src/Makevars'
✓ Writing 'src/Makevars.win'
✓ Writing 'src/.gitignore'
✓ Writing src/rust/Cargo.toml.
✓ Writing 'src/rust/src/lib.rs'
✓ Writing 'R/extendr-wrappers.R'
✓ Finished configuring extendr for package myextendr.
• Please update the system requirement in DESCRIPTION file.
• Please run `rextendr::document()` for changes to take effect.
```

</v-clicks>

------------------------------------------------------------------------

# 生成されたファイル

    .
    ├── R
    │   └── extendr-wrappers.R
    ...
    └── src
        ├── Makevars
        ├── Makevars.win
        ├── entrypoint.c
        └── rust
            ├── Cargo.toml
            └── src
                └── lib.rs

------------------------------------------------------------------------

# 編集する必要があるファイル

## src/rust:

extendrを使ったRustのcrate。基本的に開発のメインはここになる。

------------------------------------------------------------------------

# 基本いじらないファイル

## Makevars, Makevars.win:

パッケージインストール時に`cargo build`が走るようにする設定。

## entrypoint.c:

コンパイラにシンボルを勝手に消されないためのおまじない。

## R/extendr-wrappers.R:

Rustの関数から自動生成されたRの関数。

------------------------------------------------------------------------

# src/rust/Cargo.toml

``` toml
[package]
name = 'myextendr'
version = '0.1.0'
edition = '2018'

[lib]
crate-type = [ 'staticlib' ]

[dependencies]
extendr-api = '*'
```

------------------------------------------------------------------------

# src/rust/src/lib.rs（一部省略）

``` rust
use extendr_api::prelude::*;

/// Return string `"Hello world!"` to R.
/// @export
#[extendr]
fn hello_world() -> &'static str {
    "Hello world!"
}

extendr_module! {
    mod myextendr;
    fn hello_world;
}
```

------------------------------------------------------------------------

# src/rust/src/lib.rs

-   よく使う関数をまとめて読み込み

``` rust
use extendr_api::prelude::*;
```

-   `///`（3つ）のコメントはそのままRoxygenのコメントになる

``` rust
/// Return string `"Hello world!"` to R.
/// @export
```

-   これをつけるとRの関数が自動生成！

``` rust
#[extendr]
```

------------------------------------------------------------------------

# src/rust/src/lib.rs

-   関数をエクスポートしてRが認識できるように登録（routine
    registration）してくれるマクロ。
    新しく関数を追加したらここに入れる必要がある。

``` rust
extendr_module! {
    mod myextendr;
    fn hello_world;
}
```

---
layout: section
---

# 開発の流れ

------------------------------------------------------------------------

# 開発の流れ

1.  Rustのコードを編集
2.  `rextendr::document()`でRのコードを自動生成（Rustのコードのコンパイルもこれがやってくれる）
3.  （必要あれば）生成されたコードをRの側でいい感じにラップする（引数のチェックとか）
4.  `devtools::load_all()`（やテスト）で動作確認

------------------------------------------------------------------------

# `rextendr::document()`

``` sh
> rextendr::document()
✓ Saving changes in the open files.
ℹ Generating extendr wrapper functions for package: myextendr.
! No library found at src/myextendr.so, recompilation is required.
Re-compiling myextendr
─  installing *source* package ‘myextendr’ ... (382ms)
   ** using staged installation
   ** libs
   rm -Rf myextendr.so ./rust/target/release/libmyextendr.a entrypoint.o
   gcc -std=gnu99 -I"/usr/share/R/include" -DNDEBUG      -fpic  -g -O2 ...
   cargo build --lib --release --manifest-path=./rust/Cargo.toml
       Updating crates.io index
      ...
      Compiling extendr-api v0.2.0
```

------------------------------------------------------------------------

# 生成されるファイル

    .
    ...
    ├── NAMESPACE                <- @exportが反映される
    ├── R
    │   └── extendr-wrappers.R   <- Rustの関数から自動生成
    └── src
        ├── myextendr.so         <- libmyextendr.aを使ってビルドされた
        └── rust                    共有オブジェクト
            └── target           <- Rustの生成物が入るディレクトリ
                └── release
                    ├── libmyextendr.a  <- Rustのコードの静的ライブラリ
                    ...                   （OSによって拡張子は違う）

------------------------------------------------------------------------

# 自動生成されたRの関数

## Rust

``` rust
/// Return string `"Hello world!"` to R.
/// @export
#[extendr]
fn hello_world() -> &'static str {
    "Hello world!"
}
```

## R

``` r
#' Return string `"Hello world!"` to R.
#' @export
hello_world <- function() .Call(wrap__hello_world)
```

------------------------------------------------------------------------

# 実行結果

``` r
devtools::load_all(".")

hello_world()
```

    #> [1] "Hello world!"

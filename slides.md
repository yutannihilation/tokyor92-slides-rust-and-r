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
download: true
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

# RユーザのためのRStudio\[実践\]入門 第2版！

紙は6月3日、電子は5月31日発売です。

<https://gihyo.jp/book/2021/978-4-297-12170-9>

---
layout: section
---

# extendr

<img src="/images/extendr-logo.png" width="600" />

------------------------------------------------------------------------

# extendrとは？

-   RustとRを連携させるためのフレームワーク
-   RからRustを使うだけではなく、RustからRを使うこともできる（つまり、Rustの中でggplot2を呼び出してプロットしたり、とかできるらしい）
-   なぜか私も中の人です…

[<img src="/images/extendr-logo.png" width="200" />](https://github.com/extendr/extendr)

---
layout: statement
---

# なぜRust？

<v-click>

→ そこにRustがあるから！！<br/>（誰か教えてください…）

</v-click>

------------------------------------------------------------------------

# ※今日話さないこと

-   Rustの何が素晴らしいのか
-   Rust入門
-   Rust側からRを操作する方法
-   R MarkdownのRust engineとか、パッケージ外でのextendrの使いみち

------------------------------------------------------------------------

# extendrの愉快な仲間たち

<v-click>

## libR-sys（Rust）:

RのC
APIに[bindgen](https://github.com/rust-lang/rust-bindgen)で生成したバインディング

</v-click>

<v-click>

## extendr（Rust）:

libR-sysを使いやすくラップしたフレームワーク

</v-click>

<v-click>

## <span class="text-red-600">r</span>extendr（Rパッケージ）:

Rからextendrを使うためのユーティリティ（usethisパッケージのような立ち位置）

</v-click>

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

<img src="/images/screenshot_1_create_package.png" style="width:70.0%;height:70.0%" />

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

<v-click>

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

</v-click>

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

# いじるファイル

## src/rust:

extendrを使ったRustのcrate。開発のメインはここ。

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

<v-click>

-   よく使う関数をまとめて読み込み

``` rust
use extendr_api::prelude::*;
```

</v-click>

<v-click>

-   `///`（3つ）のコメントはそのままRoxygenのコメントになる

``` rust
/// Return string `"Hello world!"` to R.
/// @export
```

</v-click>

<v-click>

-   これをつけるとRの関数が自動生成！

``` rust
#[extendr]
```

</v-click>

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

<v-clicks>

1.  Rustのコードを編集
2.  `rextendr::document()`でRのコードを自動生成（Rustのコードのコンパイルもこれがやってくれる）
3.  （必要あれば）生成されたコードをRの側でいい感じにラップする
4.  `devtools::load_all()`（やテスト）で動作確認

</v-clicks>

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

---
layout: section
---

# 例1) i32 (integer)を引数に取る関数

------------------------------------------------------------------------

# 自動生成されたRの関数

## Rust

``` rust
/// @export
#[extendr]
fn add(x: i32, y: i32) -> i32 {
    x + y
}
```

## R

``` r
#' @export
add <- function(x, y) .Call(wrap__add, x, y)
```

------------------------------------------------------------------------

# 実行結果

``` r
devtools::load_all(".")

# 引数の型は i32 だけど実数も渡せる
add(1, 2)
```

    #> [1] 3

``` r
# 長さ1以上だとエラーになる
add(1:2, 2:3)
```

    #> Error in add(1:2, 2:3) : 
    #>   Input must be of length 1. Vector of length >1 given.

---
layout: section
---

# 例2) Vec\<i32\>を引数に取る関数

------------------------------------------------------------------------

# 自動生成されたRの関数

## Rust

``` rust
#[extendr]
fn mult(x: Vec<i32>, y: i32) -> Vec<i32> {
    x
     .iter()
     .map(|n| n * y)
     .collect::<Vec<_>>()
}
```

## R

``` r
#' @export
mult <- function(x, y) .Call(wrap__mult, x, y)
```

------------------------------------------------------------------------

# 実行結果

``` r
devtools::load_all(".")

mult(1:5, 10)
```

    #> [1] 10 20 30 40 50

---
layout: section
---

# 例3) struct

------------------------------------------------------------------------

# struct…?

-   環境としてエクスポートされるので状態を持つことができる
-   たまに便利（正規表現のキャッシュを持たせている例:
    [rr4r](https://github.com/yutannihilation/rr4r)）

------------------------------------------------------------------------

# 自動生成されたRの関数

## Rust

<div class="grid grid-cols-2 gap-4">

<div>

``` rust
struct Counter {
  i: i32,
}
```

</div>

<div>

``` rust
/// @export
#[extendr]
impl Counter {
  fn new() -> Self {
    Self { i: 0 }
  }

  fn count(&mut self) -> i32 {
    self.i = self.i + 1;
    self.i
  }
}
```

</div>

</div>

------------------------------------------------------------------------

# 自動生成されたRの関数

## R

``` r
#' @export
Counter <- new.env(parent = emptyenv())

Counter$new <- function() .Call(wrap__Counter__new)

Counter$count <- function() .Call(wrap__Counter__count, self)

#' @rdname Counter
#' @usage NULL
#' @export
`$.Counter` <- function (self, name) { func <- Counter[[name]]; environment(func) <- environment(); func }
```

------------------------------------------------------------------------

# 実行結果

``` r
devtools::load_all(".")

cnt <- Counter$new()
cnt$count()
```

    #> [1] 1

``` r
cnt$count()
```

    #> [1] 2

---
layout: section
---

# その他こまごました話<br/>（時間があれば）

------------------------------------------------------------------------

# その他

<v-clicks>

-   文字列関連はlifetimeを意識しないと使えないので初心者にはハードモード。数値計算系からはじめるのがおすすめ。
-   Windowsのセットアップはやや面倒だけど、GitHub
    Actionsでビルド済みのバイナリを配布したりできるはず（調査中）
-   CRANにはすでにextendrを使っているパッケージも存在する

</v-clicks>

---
layout: section
---

# まとめ

------------------------------------------------------------------------

# まとめ

-   extendrを使うとマクロの魔術でRustの関数からRの関数を生成してくれる。
-   関数の引数の対応づけはRcppやcpp11と同じノリ。慣れている人はわりとすぐに使えるはず。
    -   ちなみに、今回はすべてRustのデータ型に変換するタイプだったが、SEXPのまま扱うこともできる（ここはよく理解できていない）
-   フィードバックお待ちしています！

------------------------------------------------------------------------

# r-wakalangにrustチャンネルをつくりました

[<img src="/images/screenshot_r-wakalang.png" width="450" />](https://r-wakalang.slack.com/archives/C023FCNPV8B)

------------------------------------------------------------------------

# References

-   extendr: <https://github.com/extendr/extendr>
-   extendrのロゴはCC-BY-SA 4.0ライセンスで配布されています:
    <https://github.com/extendr/artwork>.

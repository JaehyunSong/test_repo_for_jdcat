# Test Repository for Binder with R 4.3.3

『[私たちのR: ベストプラクティスの探求](https://www.jaysong.net/RBook/)』の学習に必要なパッケージのみ事前インストールしたまっさらなR環境


## Rの使い方

* 宋財泫・矢内勇生『私たちのR: ベストプラクティスの探求』(Web-book): <https://www.jaysong.net/RBook/>
* 関西大学総合情報学部「ミクロ/マクロ政治データ分析実習」のサポートページ: <https://www.jaysong.net/r4ps/>

## 動作環境

```bash
$ echo $BASH_VERSION
5.1.16(1)-release

$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=22.04
DISTRIB_CODENAME=jammy
DISTRIB_DESCRIPTION="Ubuntu 22.04.4 LTS"

$ gcc --version
gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0

$ R --version
R version 4.3.3 (2024-02-29) -- "Angel Food Cake"

$ rstudio-server version
2023.12.1+402 (Ocean Storm) for Ubuntu Jammy
```


## 備忘録

### 初期設定時

* 最上位branch名に注意。Binderのデフォルトはmasterだが、mainならbranch名にmainと指定
* 分析環境から初期ファイル（`apt.txt`や`environment.yml`）などを削除したい場合は`postBuild`ファイル内で指定

### 日本語作図

RStudio上で表示は問題ない。ただし、PDFで書き出す際、フォントの埋め込みが必要なのでdeviceをCairoに指定。

* Plot Paneからの操作だとCairo利用にチェック
* `ggsave()`を利用するなら`device = cairo_pdf`を指定

```r
pacman::p_load(tidyverse)

tibble(x = rnorm(10), y = rnorm(10)) %>%
  ggplot(aes(x = x, y = y)) +
  geom_point() +
  labs(x = "横軸", y = "縦軸")

ggsave(filename = "Test.pdf", plot = last_plot(), device = cairo_pdf, height = 5, width = 5)
```

PNG出力なら{ragg}で一発解決

* デバイス（`dev`）は`ragg::agg_png`、解像度（`dpi`）は300以上にしておけば、印刷しても綺麗な図ができる。むろん、文字化けの心配もない。

```{r}
ggsave(..., dev = ragg::agg_png, dpi = 300, ...)
```

### プロジェクトを開く

* Jupyter hubから`.Rproj`を選択しても開かれないため、RStudioを起動し、File > Open Project...で開く必要がある。

### 日本語が含まれたPDFの作成（Quarto）

* このレポから生成した環境なら以下のYAMLヘッダーでOK
  * ハングルもOK
* フォントは適宜`mainfont`、`sansfont`、`monofont`を修正する。GoogleとかでダウンロードするNotoなら`"Noto Sans CJK JP"`でなく、`"Noto Snas JP"`だったりする。
  * ターミナルで`fc-list :lang=ja`と入力し、日本語フォントリストから正確な名前を見ておこう。

````r
---
title: "日本語PDF作成のテスト"
author: "宋財泫"
date: today
lang: ja
format: 
  pdf:
    pdf-engine: xelatex
    documentclass: bxjsarticle
    classoption:
      - a4paper
      - 10pt
    mainfont: "Noto Serif CJK JP"
    sansfont: "Noto Sans CJK JP"
    monofont: "Noto Sans Mono CJK JP"
knitr:
  opts_chunk: 
    dpi: 300
    dev: "ragg_png"
    fig.align: "center"
    warning: false
    message: false
    error: true
---

## 第1章：Quartoとは何か

なんかすごいものらしいです。詳しくは<https://quarto.org>から。

簡単な計算もできるし、

```{r}
1 + 1
```

作図もできる。今なら\texttt{\{ragg\}}があるから日本語が含まれた図も安心ね！

```{r}
#| message: false
#| fig-width: 5
#| fig-height: 2.5
library(tidyverse)
iris |> 
  ggplot() +
  geom_point(aes(x = Sepal.Length, y = Petal.Length, color = Species)) +
  labs(x = "萼片の長さ", y = "花弁の長さ", color = "種")
```

한글도 출력 잘 됩니다!
````

## 参考

* [Specifying an R environment with a runtime.txt file](https://github.com/binder-examples/r)
* [From Zero to Binder in R!](https://github.com/alan-turing-institute/the-turing-way/blob/master/workshops/boost-research-reproducibility-binder/workshop-presentations/zero-to-binder-r.md)
* [r-conda](https://github.com/binder-examples/r-conda)
  * こっちの方がDockerイメージ生成が速いらしい
* [オンライン分析システム（実証実験）](https://meatwiki.nii.ac.jp/confluence/display/niircosap)
  * [初期設定](https://binder.cs.rcos.nii.ac.jp/) (学認)
  * [Jupyter hub](https://jupyter.cs.rcos.nii.ac.jp/) (学認)

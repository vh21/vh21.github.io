---
layout: post
title: 'Setup github page'
date: 2015-04-09 21:00
comments: true
tags: [gitpage, jekyll]
categories: jekyll
---

有朋友在問怎麼轉移到 github page ，本來 logdwon 用得好好的，只是測一下 github page ，
沒想到要打包 logdown 時發現功能不能用，乾脆怒轉到github來。

可參考 https://pages.github.com/ 。基本上github page背後的blog engine是jekyll，
一種靜態網頁的引擎。整個blog的寫作就跟summit code沒兩樣。流程如下，

## 開blog repository

首先要在 github 開一個 repository ，命名規則是`yourname.github.io`，舉例來說我的就是
`vh21.github.io`。把你的文章 push 到 github後，就可以透過 http://yourname.github.io 來瀏覽

## 安裝jekyll (optional)

裝了的好處是你可以在 local 先review，沒裝應該也可以。安裝步驟請參考
http://jekyllrb.com/docs/installation/

看起來是ruby寫的，所以需要先安裝ruby跟rubygem。

## 產生blog instance

雖然上一步的網頁教你 clone 他的 repository ，但是他的版面太陽春了。現在有很多人基於 jekyll
開發了很多 theme 。版權許可的話直接 clone 別人整好 theme 的repository吧！畢竟不是每個人都
有美術天分。

可到google上找`jekyll theme`應該就可以找到一大票。如 http://jekyllthemes.org/ 。找到喜歡
的佈景主題後，通常會提供 source ，直接 clone 下來。改一些設定後，加上你自己的文章，就可以
push 到 github了！

## 修改設定值

jekyll 的設定值在根目錄的 `_config.yml` 。看起來算好改，基本上就是一些個人資料更新。有一個
值得注意的是你的 markdown 語法，看起來似乎支援兩種（以上？）—  redcarpet、kramdown。轉換結果
 logdown 好像跟 redcarpet 比較接近（？）。所以我預設選用 redcarpet。另外他有一些語法 extension，
開就對了。`tables` 決定你的table能不能正常顯示，`autolink` 如果你有寫 `http://` 開頭的東西會
自動幫你改成連結，預設不會。所以我這幾個設定如下，

~~~
markdown: redcarpet
redcarpet:
    extensions: [tables, autolink]
~~~

其他的設定值可參考 http://jekyllrb.com/docs/configuration/ 修改。

## 發表文章

好了，基本上可以開始寫文章了，可以接受 markdown 檔案 （`file.md` or `file.markdown`），
好像也接受html。放在 `_post` 下。
如果你有 logdown 下載下來的舊文可以直接用。除了程式區塊開始前要多加
空白行。有點麻煩，用 sed 一次修改看看。

如

~~~
  old logdown article
  ` ` `
  code block
  ` ` `
~~~

要改成

~~~
  old logdown article

  ` ` `
  code block
  ` ` `
~~~

如果沒有舊文章，就看看別人的 github page 的文章來當template吧。需要有一個 header

~~~
---
layout: post
title: 'Setup github page'
date: 2015-04-09 21:00
tags: [github blog jekyll]
categories: jekyll
---

my markdown article .....

~~~

語法可參考
https://help.github.com/articles/markdown-basics/ 與
https://help.github.com/articles/github-flavored-markdown/。

## 上傳

上傳前，如果有裝 jekyll 可以先在repository的根目錄下

~~~
%  jekyll serve

~~~

此時會幫你產生最後的靜態網頁，並啟動一個 http server，所以可以從 http://localhost:4000 看到
demo。可以 review 一下文章是否有出現如你想要的結果。

最後，當然就是 `git add`、`git commit`、與`git push` 三重奏了。 push 上 github 後，很快
server 就幫你產生新的 page 了。

## 其他事項

補上一個quick start 網頁http://jekyllbootstrap.com/usage/jekyll-quick-start.html

據說 logdown 可以幫程式區塊加上標頭，但 jekyll 好像不支援（？），有人改用 Octopress，
好像有這功能，可參考wen00072的文章
[安裝octopress並佈署到github Pages上面](http://wen00072.github.io/blog/2015/03/25/octopress-installed-and-deployed-on-the-github-pages/)

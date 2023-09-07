---
title: 新手搭建Blog part2：分類、標籤
date: 2020-10-09 20:23:22
categories: Blog
tags: Blog
---
# 新手搭建Blog part2：分類、標籤

## # 發佈文章

```
$ hexo new [layout] <title>
```

如果沒有設定 `layout` 的話，則會使用 `_config.yml`中的 `default_layout` 設定代替。如果標題包含空格的話，請使用引號括起來。

```
$ hexo new “postName” # 新建文章
$ hexo new page “pageName” # 新建頁面
```

所有的文章都會在`source/_posts`裡面。一開始裡面就已經有一篇範例文章`hello-world.md`了。

常用组合

```
$ hexo d -g # 產生靜態檔後部署
$ hexo s -g # 產生靜態檔後預覽
```

如果想在 `local` 端先確認內容，可以用

```
$ hexo server #執行
```

- 如果想用線上編輯器，[HackMD](https://hackmd.io/sHXhBDrtQh-nzU5oeVotGw)、[markdown-editor](https://markdown-editor.github.io/)

- 如果用的編輯器是`vscode`，可以看看[Markdown Shortcuts](https://marketplace.visualstudio.com/items?itemName=mdickin.markdown-shortcuts)跟[Markdown Preview Github Styling](https://marketplace.visualstudio.com/items?itemName=bierner.markdown-preview-github-styles)

## # 刪除文章

首先進入到 source /_post 資料夾中，找到 helloworld.md 文件，在本地直接執行刪除。

刪除後執行以下指令

```
$ hexo clean    # 清除快取檔案和已產生的靜態檔案。
$ hexo d -g     # d = deploy, g = generate
```

等一段時間，就會發現那篇文章已經消失

## # 創建“分類”選項(Categories)

### # 新增文章分類頁面，添加type屬性

新建一个頁面，命名为 categories。命令如下：

```
$ hexo new page categories
```

接著修改剛剛產生的 source/categories/inex.md ，添加`type: "categories"`到内容中。

```
---
title: categories
date: 2020-10-09 16:22:14
type: "categories"
---
```

### # 給文章添加categories属性

讓文章增加分類，只要在 categories 後面加入類別就可以了，記得要空格！

```
---
layout: building
title: Blog
date: 2020-10-09 16:17:51
tags:
categories: 分類名稱
---
```

至此，成功給文章添加分類，點擊首頁的“分類”可以看到該分類下的所有文章。當然，只有添加了categories: xxx的文章才會被收錄到首頁的“分類”中。

注意：hexo一篇文章只能屬於一個分類，也就是說如果在“web”下方添加“java”，hexo不會產生兩個分類，而是把分類嵌套（即該文章屬於“ web”下的"java ”分類）。

## # 創建“標籤”選項 (tags)

### # 生成“標籤”頁並添加tpye屬性

```
$ hexo new page tags
```

接著修改剛剛產生的 source/tags/inex.md ，添加`type: "tags"`到内容中

```
---
title: categories
date: 2019-04-01 22:55:41
type: "tags"
---
```

### # 給文章添加“tags”屬性

讓文章增加分類，只要在 tags 後面加入標籤就可以了，一樣記得要空格！

```
---
title: 新的文章
date: 2017-05-26 12:12:57
categories: 文章類別
tags:
- 標籤1
- 標籤2
---
```

一篇文章只可以有一個類別，但是可以同時有很多個標籤，不同的標籤要用空格來分開。

## # 新建頁面的模板

另外如果不想要每次新增文章都要輸入一次 “categories:” 與 “tags”，可以修改 scaffolds/post.md

```
title: {{ title }}
date: {{ date }}
categories: {{ categories }}
tags: {{ tags }}
```

scaffolds目錄下，是新建頁面的模板，執行新建命令時，是根據這裡的模板頁來完成的，所以可以在這裡根據你自己的需求添加一些默認值。

這樣每次產生新的文章就會自動帶入 categories 跟 tags 了。

## # 側欄配置

以上修改完後，記得還要去主題配置的 menu 中把 categories 跟 tags 打開，這樣才能在側欄中看到！

打開 主題配置文件 找到Menu Settings ，把 categories 和 tags 取消註釋。

<img src="./1.png" alt="1">

另外是如果原本就有寫好的文章的話，直接修改 .md 檔加入 categories 跟 tags，即使重新 clean, generate, deploy，Hexo 是不會理你的，要重新發一篇文章才會有文章分類或標籤。

## # Reference

[Hexo 官方網站](https://hexo.io/zh-tw/index.html)

[NexT 官方網站](http://theme-next.iissnan.com/)

[[筆記] 打造自己的 blog，Hexo + Github 之二](https://kemushi54.github.io/2019/04/01/%E7%AD%86%E8%A8%98-%E6%89%93%E9%80%A0%E8%87%AA%E5%B7%B1%E7%9A%84-blog%EF%BC%8CHexo-Github-%E4%B9%8B%E4%BA%8C/)

[Hexo+GitHub，新手也可以快速建立部落格](https://blackmaple.me/hexo-tutorial/)
---
title: 從零搭建Blog part3： 圖片和其他資源
date: 2020-10-09 23:51:33
categories: Blog
tags: Blog
---
# 從零搭建Blog part3： 圖片和其他資源

## # 絕對路徑本地引用

當Hexo項目中只用到少量圖片時，可以將圖片統一放在source/images文件夾中，通過markdown語法訪問它們。

```
![](/images/image.jpg)
```

圖片既可以在首頁內容中訪問到，也可以在文章正文中訪問到。

- `![example](assetsTest/example.png)`

  幾乎所有的 markdown 編輯器都支持的相對引用的寫法，在markdown 編輯器的預覽視圖中都能正常顯示圖片，但在主頁和文章中無法正常顯示。

為了解決無法在主頁和文章中正常顯示的問題，需要安裝[hexo-asset-image](https://github.com/xcodebuild/hexo-asset-image)插件：

```shell
npm install hexo-asset-image@0.0.1 --save
```
<!--more-->



## # 相對路徑本地引用

圖片除了可以放在統一的images文件夾中，還可以放在文章自己的目錄中。文章的目錄可以通過站點配置文件_config.yml來生成。

`````_
#_config.yml
post_asset_folder: true
`````

config.yml文件中的配置項`post_asset_folder`設為`true`後，執行命令`$ hexo new post_nam`e，在source/_posts中會生成文章`post_name.md`和`同名文件夾post_name`。將圖片資源放在`post_name文件夾`中，文章就可以使用相對路徑引用圖片資源了。

```
![](image.jpg)
```

- `![example](example.png)`

  Hexo 官方文檔關於 markdown 的示例寫法，圖片不能在主頁正常顯示，但在文章中能正常顯示。此外，這種相對引用寫法在絕大數markdown 編輯器的預覽視圖中都無法正常顯示圖片（Typora 通過設置`Use Image Root Path` 可以解決，它會在文件頭添加`typora-root-url: xxxx `）。




## # 標籤插件語法引用

通過常規的MarkDown語法和相對路徑引用圖片或其它資源相對方便，但是這些包含相對位置引用資源的文章在首頁，或者存檔頁面不能正常顯示。 HEXO 3開始核心代碼中加入了相對路徑引用的標籤插件，用以解決這個問題。 （在HEXO 3之前許多開發者開發類似插件，現在被官方更新後核心代碼採用了）。

並且需要安裝插件，安裝方法為:

```
npm install https://github.com/7ym0n/hexo-asset-image --save
```

```
# 本地圖片資源，不限製圖片尺寸
{% asset_img image.jpg This is an image %}
# 網絡圖片資源，限製圖片顯示尺寸
{% img http://www.viemu.com/vi-vim-cheat-sheet.gif 200 400 vi-vim-cheat-sheet %}
```

- `{% asset_img example.png example %}`

  Hexo 官方推薦使用標籤的寫法，圖片能在主頁和文章中正常顯示，但是不能在 markdown 編輯器的預覽視圖中預覽圖片。
  

### # 啟用fancybox：點擊查看圖片大圖

我這裡使用的是Hexo的NexT主題，NexT主題中提供了fancybox的方便接口。

Usage：https://github.com/theme-next/theme-next-fancybox3
markdown用法：

```
{% img http://www.viemu.com/vi-vim-cheat-sheet.gif 600 600 "點擊查看圖片大圖:vi/vim-cheat-sheet" %}
```



## # HTML語法引用（最終手段，一定能成，推薦）

```
<img src="image.png" width="50%" height="50%" title="image" alt="image"/>
```

<mark>使用此方法要注意路徑</mark>

config.yml文件中的配置項`post_asset_folder`設為`true`

```
$ tree source/_posts 
source/_posts
├── Blog1
│   └── image1.png
└── Blog1.md

# Blog1.md 中的 HTML語法引用
<img src="./image1.png" width="50%" height="50%" title="image" alt="image"/>
```

[tree command on osx bash](https://stackoverflow.com/questions/8304172/tree-command-on-osx-bash)



## # ---Problem---

問題推測

（一）本地圖片沒有有效上傳至github倉庫中，導致引用無效

解決方案：安裝外掛（回看前文）

（二）本地圖片沒有存放在同名資料夾中

解決方案：將需要引用的本地圖片存放在與文章名相同的資料夾中

（三）圖片路徑出錯



## # reference

[資源文件夾](https://hexo.io/zh-cn/docs/asset-folders.html)

[在Hexo博客中插入图片的各种方式](https://fuhailin.github.io/Hexo-images/)

[hexo引用本地圖片無法顯示](https://www.itread01.com/content/1542264790.html)

[標籤外掛（Tag Plugins）](https://hexo.io/zh-tw/docs/tag-plugins.html)

[Hexo 搭建个人博客（五）Hexo 资源文件夹](https://blog.csdn.net/qq_32767041/article/details/103283522)
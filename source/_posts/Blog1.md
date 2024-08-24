---
title: 從零搭建Blog part1： Hexo+GitHub
date: 2020-10-09 20:26:57
categories: Blog
tags:
- Blog
- git
- ssh
---
# 從零搭建Blog part1： Hexo+GitHub

> Hexo 是一個快速、簡單且強大的網誌框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在幾秒鐘內透過漂亮的主題產生靜態檔案。
<!--more-->

## # For Mac --- strongly recommand

[Mac終端機 (Terminal)設定： iTerm 2](https://medium.com/nitas-learning-journey/mac%E7%B5%82%E7%AB%AF%E6%A9%9F-terminal-%E8%A8%AD%E5%AE%9A-iterm2-ba63efd0df6a) ---> change your terminal 

## # 安裝 Node.js + git

Mac --> homebrew

去 [Node.js](https://nodejs.org/en/) 官網下載安裝

- [Node.js](https://nodejs.org/en/) (Node.js 版本需不低於 8.6，建議使用 Node.js 10.0 及以上版本)
- [Git](https://git-scm.com/)

## # github新建一個項目

新建一個項目，如下所示：
<img src="./1.png" width="100%" height="100%" title="1" alt="1"/>


然後如下圖所示，輸入自己的項目名字，後面一定要加.github.io後綴

===作為Github Page的倉庫，一定要為`你Github的用戶名.github.io` ===

<img src="./2.png"  title="2" alt="2"/>

然後項目就建成了，點擊Settings，向下拉到最後有個GitHub Pages，點擊-------------Choose a theme選擇一個主題。然後等一會兒，再回到GitHub Pages，會變成下面這樣：

<img src="./3.png"  title="3" alt="3"/>

> 選擇主題，此時不能訪問，需要選擇一個主題，才可以訪問

點擊那個鏈接，就會出現自己的網頁

## # Hexo --> Blog

### # 創造一個資料夾專門放 Blog 檔案

創造一個資料夾專門放 Blog 檔案，以下操作都在這資料夾中。

安裝 Hexo Git，Hexo 提供了快速方便的一鍵佈署功能，只需一個指令就能將網站佈署到伺服器上 [佈署](https://hexo.io/zh-tw/docs/one-command-deployment.html)

```
$ npm install hexo-deployer-git --save
```

接下來透過`npm`安裝`Hexo`

```
$ npm install -g hexo-cli
#or
$ npm i hexo-cli -g
```

安裝完後輸入`hexo -v`驗證是否安裝成功。

```
$ hexo version
#or
$ hexo -v
```

接下來就可以開始初始化部落格

```
$ hexo init  # 初始化專門放 Blog 檔案資料夾
$ npm install # 安裝相關套件
```

<img src="./4.png"  title="4" alt="4"/>

`_config.yml`：網站 [配置](https://hexo.io/zh-tw/docs/configuration) 檔案，可以在此配置大部分的設定。
`package.json`：套件版本控制
`scaffolds`：[鷹架](https://hexo.io/zh-tw/docs/writing.html#鷹架（Scaffold）) 資料夾。建立新文章時，會根據 scaffold 來建立檔案。
`source`：原始檔案資料夾是放置內容的地方。
`themes`：[主題](https://hexo.io/zh-tw/docs/themes) 資料夾。Hexo 會根據主題來產生靜態檔案。

再來到 blog 資料夾下找一個 _config.yml 的檔案，這是 Hexo 的全域配置文件

開啟之後，拉到檔案最底部，可以看到

```
deploy:
  type:
```

將以下資訊對應輸入

```
deploy:
  type: git
  repository: http://github.com/yourname/yourname.github.io.git
  branch: master
```

> 設定中的 “:” 後一定要有一個空格
>
> 將上面的 yourname 改成 GitHub 帳號名稱

### # 1. localhost

接著執行下方兩行指令，就可以在 http://localhost:4000/ 查看Blog 。

```
$ hexo g
$ hexo s
#or
$ hexo server
```

Ctrl + c --> stop

### # 2. 部署上 GitHub

產生靜態文件後，部署上 GitHub

```
$ hexo d -g
```

等一段時間後就能到`https://yourname.github.io/`查看有沒有成功部署，接下來就可以開始寫文章哩。

#### # ---Problem+Solution---

執行`hexo d`報錯：

```
$ERROR Deployer not found: git
```

解決方案
這是因為沒安裝 hexo-deployer-git 插件，在`站點目錄`下輸入下面的插件安裝就好了：

```
$ npm install hexo-deployer-git --save
```

(如果不成功--->連接Github與本地)

## # 常用指令
[指令](https://hexo.io/zh-tw/docs/commands.html)

1. `hexo generate (hexo g)` 產生靜態檔案，會在目錄下產生public 資料夾。
2. `hexo server (hexo s)` 啟動伺服器，預設是 `http://localhost:4000/`。
3. `hexo deploy (hexo d)` 部署網站。（ 比如 github, heroku 等平台 ）
4. `hexo clean (hexo cl)` 清除快取檔案 (db.json) 和已產生的靜態檔案 (public)，如過修改檔案沒有響應到頁面，使用這個指令再啟動伺服器

## # 更換hexo主題

### # NexT

Hexo 預設的主題是[landscape](https://github.com/hexojs/hexo-theme-landscape)，而我們可以到 [這裡](https://hexo.io/themes/) 來挑選我們想要的主題，而其中最多人用的應該就屬 [NexT](https://theme-next.org/) 了。

安裝NexT

```
$ git clone https://github.com/theme-next/hexo-theme-next.git themes/next
```

接著修改網站設定`_config.yml`

```
theme: next
```

重新啟動server

```
$ hexo server
```

#### # ---Problem+Solution---

如果不成功，試著清除先前已生成的檔案

```
$ hexo clean
$ hexo d -g
```

到`https://yourname.github.io/`查看有沒有成功部署

### # 語言設定

在`_config.yml`中找到language，設定為zh-tw

```
language: zh-tw
```

這樣就可已支援繁體中文。

### # Scheme 設定

在NexT 中有多種Scheme 可選擇，其中的預設主題為Muse。大部份看到蠻多人選擇使用Pisces當主題。

在主題設定`theme/next/_config.yml`裡找到schema，把註解去掉即可開啟。

```
#scheme: Muse
#scheme: Mist
scheme: Pisces
#scheme: Gemini
```

### # 代碼高亮

NexT 使用 Tomorrow Theme 當作代碼高亮，共有5款主題可以選擇。 NexT 預設使用的是 normal，可選的值有 normal、night、 night blue、 night bright、 night eighties

```
highlight_theme: normal
```

### # 背景特效

在主題設定`themes/next/_config.yml`裡還有另外的動畫效果，要打開就把值設定為true。

## # 連接Github與本地倉庫

### # Git setting

在每一次的 Git commit (提交，我們稍後會提到) 都會記錄作者的訊息像是 name 及 email ，因此我們使用下面的指令來設定：

加上 `--global` 表示是全域的設定。你可以使用 `git config --list` 這個指令來看你的 Git 設定內容：

```git
$ git config --global user.name "username"
$ git config --global user.email "GitHub 邮箱"
```

用戶名和郵箱根據你註冊github的信息自行修改。

> 使用`git config --list`可查看配置的相關信息 --->  進入vim
>
> `:q` ---> exit
>
> `:w` ---> save
>
> `:wq` --->save and exit
>
> 或是其實 Git 的設定檔是儲存在你的家目錄的.gitconfig 隱藏檔中，你可以使用編輯器將他打開
>
> ```
> $ cat ~/.gitconfig
>     [user]
>         name = Jimmy Kuo
>         email = jimmy@gogojimmy.net
> ```

### # 生成密鑰SSH key

為了讓GitHub可以識別文件是由GitHub用戶上傳的，我們需要創建ssh密鑰

```
#生成密鑰SSH key：
$ ssh-keygen -t rsa -C "GitHub 邮箱"
```

然後一路回車鍵

<img src="./5.png"  title="5" alt="5"/>

> Enter file in which to save the key: ---> Enter == save the private ssh key
>
> Enter passphrase (empty for no passphrase): ---> Enter == without setting password for private ssh key
> Enter same passphrase again: --->  Enter == without setting password for private ssh key

接下來就會產生key到指定位置了

會有兩個key 一個有副檔名 .pub 為公鑰，而沒有副檔名的為私鑰

<mark>注意私鑰要保存好 絕對不能被竊取</mark>

### # Add public ssh key on github

打開github，在頭像下麵點擊settings，再點擊SSH and GPG keys，新建一個SSH，名字隨便。

<img src="./6.png" alt="6" width="40%" height="40%" />

<img src="./7.png" alt="7"  />

```
#公鑰
$ cat ~/.ssh/id_rsa.pub
```

將輸出的內容複製到 key 框中，點擊確定保存。

```
$ ssh -T git@github.com
```

輸入`ssh -T git@github.com`，如果如下圖所示，出現你的用戶名，那就成功了。

```
Hi github用戶名! You've successfully authenticated, but GitHub does not provide shell access.
```

### # 把檔案交給 Git 控管

[把檔案交給 Git 控管](https://gitbook.tw/chapters/using-git/add-to-git.html)

```
$ git init
$ git add .
$ git commit -m "change Theme to nexT"
```

#### # ---Problem+Solution---

[Hexo + Github page博客 themes/next 文件夹因存在.git而无法提交到git的解决办法](https://www.cnblogs.com/reboot777/p/11164193.html)

<img src="./8.png"  title="8" alt="8"/>

解決方法：

1.剪切（刪除） Theme/next/.git文件夾到其他處

打開一個Terminal終端窗口，輸入以下命令：

```
#顯示文件夾
$ defaults write com.apple.finder AppleShowAllFiles TRUE

#最後重啟finder
$ killall Finder
```

```
#隱藏文件夾
$ defaults write com.apple.finder AppleShowAllFiles FALSE

#最後重啟finder
$ killall Finder
```

<img src="./9.png"  title="9" alt="9"/>

2.從暫存區刪除該文件夾

```
$ git rm --cached themes/next
```



### # 推上遠端的 Git 伺服器

在這之前，我們所有的操作都是在自己電腦上，接下來就是要準備把東西推上遠端的 Git 伺服器了。首先，需要設定一個端節的節點

<img src="./10.png"  title="10" alt="10"/>

```
#加入遠端節點：
$ git remote add origin repositpry-link
```

origin 只是一個遠端的代名詞，會取這個名字是因為如果從 Server Clone 下來會是這個代稱。所以如果把上面那句指令翻成中文的話：「為目前所在本地端檔案庫增加一個叫做 origin 的遠端檔案庫」

```
#將本地端檔案庫（local repo）推上一個叫做 origin 的遠端檔案庫（remote repo）master 分支：
$ git push -u origin master
＃可能需要輸入
Username for 'https://github.com': 輸入 Github 上的 帳號
Password for 'https://miahsu0330@github.com':輸入 Github 上的 密碼
```

定好每次要 Push 時就不用再次輸入「上游」的名稱，Git 就會知道要推往哪個遠端節點，反之如果沒設定，就要每次都輸入遠端節點的名稱。

![11](11.png)

#### # ---Problem+Solution---

[【狀況題】怎麼有時候推不上去…](https://gitbook.tw/chapters/github/fail-to-push.html)

[Git 指令 – 常用指令](https://exfast.me/2016/05/git-instructions-instructions/)

Push

```
$ git remote add origin https:*//[github.com/XXX(username)/YYYY(projectname).git]
```

加上一個remote的位址，名叫origin，位址是github上的地址（Create a new repo就會有）
因為Git是分散式的，所以可以有多個remote.

```
$ git push -u origin master
```

將本地內容push到github上的那個位址上去。

`參數-u`
用了參數-u之後，以後就可以直接用不帶參數的git pull從之前push到的分支來pull。

此時如果origin的master分支上有一些本地沒有的提交,push會失敗.

這是因為遠端的版本比你電腦裡的這份更新，因此 Git 不讓你推上去。

所以解決的辦法是, 首先設定本地master的上游分支:

```
$ git branch --set-upstream-**to**=origin/master
```

然後pull:

```
$ git pull --rebase
```

最後再push:

```
$ git push
```

如果還是不行push，砍掉重新來過

## # Reference

[如何搭建個人 Blog 使用 Hexo + Gitpage](https://medium.com/@bebebobohaha/%E4%BD%BF%E7%94%A8-hexo-gitpage-%E6%90%AD%E5%BB%BA%E5%80%8B%E4%BA%BA-blog-5c6ed52f23db)

[Hexo+GitHub，新手也可以快速建立部落格](https://blackmaple.me/hexo-tutorial/)

[在Mac平台使用GitHub和Hexo搭建博客](https://musoucrow.github.io/2017/02/26/build_bolg/)

[[教學] 產生SSH Key並且透過KEY進行免密碼登入](https://xenby.com/b/220-%E6%95%99%E5%AD%B8-%E7%94%A2%E7%94%9Fssh-key%E4%B8%A6%E4%B8%94%E9%80%8F%E9%81%8Ekey%E9%80%B2%E8%A1%8C%E5%85%8D%E5%AF%86%E7%A2%BC%E7%99%BB%E5%85%A5)

[如何解決github+Hexo的部落格多終端同步問題](https://www.itread01.com/content/1546966625.html)

[git连接github总结](https://segmentfault.com/a/1190000007466317)

[[第二週] Git 本地端連結遠端操作（GitHub）](https://medium.com/@miahsuwork/%E7%AC%AC%E4%BA%8C%E9%80%B1-git-%E6%9C%AC%E5%9C%B0%E7%AB%AF%E8%88%87%E9%81%A0%E7%AB%AF%E6%93%8D%E4%BD%9C-github-78eec4537179)

[2.5 Git 基礎 - 與遠端協同工作](https://git-scm.com/book/zh-tw/v2/Git-%E5%9F%BA%E7%A4%8E-%E8%88%87%E9%81%A0%E7%AB%AF%E5%8D%94%E5%90%8C%E5%B7%A5%E4%BD%9C)


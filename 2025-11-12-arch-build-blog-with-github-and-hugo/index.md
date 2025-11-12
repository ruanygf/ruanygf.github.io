# arch 用 github 与 hugo 搭建博客


利用 hugo 生成 html 界面，分别建立源码仓库与网页仓库，通过 github 的 action 功能实现一次推送两个仓库更新

<!--more-->

{{< admonition tip "参考链接" true>}}

- [如何用 GitHub Pages + Hugo 搭建个人博客](https://krislinzhao.github.io/docs/create-a-wesite-using-github-pages-and-hugo/)
- [themes.gohugo/](https://themes.gohugo.io/)

{{< /admonition >}}

## 1 准备环境

- arch
- hugo
- 本地 Git + 远程 Github 账号

{{< admonition note "关于安装的 hugo 版本" true>}}

建议 hugo 安装 0.145.0 版本，当前时间（2025.11.11）使用最新版 0.152.2 版本在使用 LoveIt 主题时会出现下面的报错

```bash
YamiNook git:(main) ✗ hugo server 
WARN deprecated: the ":filename" permalink token was deprecated in Hugo 0.144.0 and will be removed in a future release. Use ":contentbasename" instead. 
Error: html/template:_markup/render-codeblock-mermaid.html:6:17: no such template "_default/_markup/render-codeblock.html"
```

如果已经安装 hugo, 执行下列操作：

```bash
paru -Rsn hugo
wget https://archive.archlinux.org/packages/h/hugo/hugo-0.145.0-3-x86_64.pkg.tar.zst
sudo pacman -U hugo-0.145.0-3-x86_64.pkg.tar.zst
hugo version # 显示 v0.145.0 即可
```

{{< /admonition >}}

## 2 构建仓库

github 上建立 2 个仓库，一个存放源码，一个存放渲染的网页，通过 github 自带的 action 功能，将代码提交到源码仓库能自动渲染同步到渲染仓库

### 2.1 远程源码仓库

名字自取，推荐和之后建立的项目名一致，**设置私有**，如我的博客名称是“yaminook”，因此取这个名字，后续本地建立的项目名称也是这个名字

### 2.2 远程渲染仓库

名称必须为“用户名.github.io”，**设置公有**

### 2.3 部署密钥（实现仓库间免密推送）

创建 SSH 密钥，让源码仓库的 Actions 有权限推送文件到 `ruanygf.github.io` 网页仓库。

在本地终端生成 SSH 密钥对（无需设置密码，一路回车）：

```bash
ssh-keygen -t ed25519 -C "hugo-deploy" -f ~/.ssh/hugo_deploy_key
```

执行后会生成两个文件：

- 私钥： `~/.ssh/hugo_deploy_key` （保密，仅配置在源文件仓库）
- 公钥： `~/.ssh/hugo_deploy_key.pub` （配置在 GitHub Pages 仓库）

配置公钥到 `ruanygf.github.io` 仓库（目标仓库）：

- 打开 GitHub 上的 `ruanygf/ruanygf.github.io` 仓库 → 进入「Settings」→ 「Deploy keys」→ 「Add deploy key」。
- Title 填「hugo-deploy-key」，Key 粘贴 `hugo_deploy_key.pub` 文件的全部内容，勾选「Allow write access」（允许写入），点击「Add key」。
配置私钥到源文件仓库：
- 打开 GitHub 上的 `ruanygf/yaminook` 仓库 → 进入「Settings」→ 「Secrets and variables」→ 「Actions」→ 「New repository secret」。
- Name 填「DEPLOY_KEY」，比如我写的是“ACTION_OF_BLOG_YAMINOOK”（必须与下文中 deploy.yml 中的 ${{ secrets.DEPLOY_KEY}} 一致），Secret 粘贴 `hugo_deploy_key` 文件的全部内容，点击「Add secret」。

## 3 构建项目

### 3.1 创建空项目

本地创建项目（包含博客源代码与渲染文件）：

```bash
# 创建项目
hugo new site YamiNook # YamiNook 为博客名
```

执行完毕项目目录如下：

```bash
➜  YamiNook tree -L 2
.
├── archetypes
│   └── default.md
├── assets
├── content
├── data
├── hugo.toml
├── i18n
├── layouts
├── static
└── themes
```

### 3.2 安装主题

主题可以在 [themes.gohugo/](https://themes.gohugo.io/) 寻找，这里以 LoveIt 为例：

```bash
git init
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
```

### 3.3 依据模板修改配置

将 `themes/LoveIt/exampleSite/` 下的所有内容复制到项目目录下进行替换

```bash
cp -r themes/LoveIt/exampleSite/* ./
```

复制完毕目录如下：

```bash
➜  YamiNook git:(master) ✗ tree -L 2
.
├── archetypes
│   └── default.md
├── assets
│   ├── css
│   ├── images
│   └── music
├── content
│   ├── about
│   ├── categories
│   ├── posts
│   └── tags
├── data
├── hugo.toml
├── i18n
├── layouts
├── static
│   ├── android-chrome-192x192.png
│   ├── android-chrome-512x512.png
│   ├── Apple-Devices-Preview.png
│   ├── apple-touch-icon.png
│   ├── browserconfig.xml
│   ├── Dillon.png
│   ├── favicon-16x16.png
│   ├── favicon-32x32.png
│   ├── favicon.ico
│   ├── mstile-150x150.png
│   ├── _redirects
│   ├── safari-pinned-tab.svg
│   └── site.webmanifest
└── themes
    └── LoveIt
```

修改 `hugo.toml` 中的内容，必要有 3 点：

```bash
baseURL = "https://ruanygf.github.io/ # 网页链接
theme = "LoveIt" #主题名称
themesDir = "./themes" # 主题目录

```

修改以上三点就可以跑了，要再进行一些别的自定义设置可以有：

```bash
title = "YamiNook"
defaultContentLanguage = "zh-cn"
languageCode = "zh-CN"
languageName = "简体中文"
hasCJKLanguage = true
gitRepo = "https://github.com/ruanygf/ruanygf.github.io"
svgFavicon = "safari-pinned-tab-half.svg" # 网站图标
```

{{< admonition note "我的 hugo.toml 自定义设置，可以与官方的进行比较查看改动" false>}}

```toml
baseURL = "https://example.com"

# theme
# 主题
theme = "LoveIt"
# themes directory
# 主题目录
themesDir = "../.."

# website title
# 网站标题
title = "LoveIt"

# determines default content language ["en", "zh-cn", "fr", "pl", ...]
# 设置默认的语言 ["en", "zh-cn", "fr", "pl", ...]
defaultContentLanguage = "en"
# language code ["en", "zh-CN", "fr", "pl", ...]
# 网站语言, 仅在这里 CN 大写 ["en", "zh-CN", "fr", "pl", ...]
languageCode = "en"
# language name ["English", "简体中文", "Français", "Polski", ...]
# 语言名称 ["English", "简体中文", "Français", "Polski", ...]
languageName = "English"
# whether to include Chinese/Japanese/Korean
# 是否包括中日韩文字
hasCJKLanguage = false

# copyright description used only for seo schema
# 版权描述，仅仅用于 SEO
copyright = ""

# whether to use robots.txt
# 是否使用 robots.txt
enableRobotsTXT = true
# whether to use git commit log
# 是否使用 git 信息
enableGitInfo = true
# whether to use emoji code
# 是否使用 emoji 代码
enableEmoji = true

# ignore some build errors
# 忽略一些构建错误
ignoreErrors = ["error-remote-getjson", "error-missing-instagram-accesstoken"]

# Pagination config
# 分页配置
[pagination]
  disableAliases = false
  pagerSize = 10
  path = "page"

# Menu config
# 菜单配置
[menu]
  [[menu.main]]
    weight = 1
    identifier = "posts"
    # you can add extra information before the name (HTML format is supported), such as icons
    # 你可以在名称 (允许 HTML 格式) 之前添加其他信息, 例如图标
    pre = ""
    # you can add extra information after the name (HTML format is supported), such as icons
    # 你可以在名称 (允许 HTML 格式) 之后添加其他信息, 例如图标
    post = ""
    name = "Posts"
    url = "/posts/"
    # title will be shown when you hover on this menu link
    # 当您将鼠标悬停在此菜单链接上时, 将显示标题
    title = ""
  [[menu.main]]
    weight = 2
    identifier = "tags"
    pre = ""
    post = ""
    name = "Tags"
    url = "/tags/"
    title = ""
  [[menu.main]]
    weight = 3
    identifier = "categories"
    pre = ""
    post = ""
    name = "Categories"
    url = "/categories/"
    title = ""

[params]
  # site default theme ["auto", "light", "dark"]
  # 网站默认主题 ["auto", "light", "dark"]
  defaultTheme = "auto"
  # public git repo url only then enableGitInfo is true
  # 公共 git 仓库路径，仅在 enableGitInfo 设为 true 时有效
  gitRepo = "https://github.com/dillonzq/LoveIt"
  # which hash function used for SRI, when empty, no SRI is used
  # ["sha256", "sha384", "sha512", "md5"]
  # 哪种哈希函数用来 SRI, 为空时表示不使用 SRI
  # ["sha256", "sha384", "sha512", "md5"]
  fingerprint = ""
  # date format
  # 日期格式
  dateFormat = "2006-01-02"
  # website title for Open Graph and Twitter Cards
  # 网站标题, 用于 Open Graph 和 Twitter Cards
  title = "LoveIt"
  # website description for RSS, SEO, Open Graph and Twitter Cards
  # 网站描述, 用于 RSS, SEO, Open Graph 和 Twitter Cards
  description = "Hugo theme - LoveIt"
  # website images for Open Graph and Twitter Cards
  # 网站图片, 用于 Open Graph 和 Twitter Cards
  images = ["/logo.png"]

  # Author config
  # 作者配置
  [params.author]
    name = "xxxx"
    email = ""
    link = ""

  # Header config
  # 页面头部导航栏配置
  [params.header]
    # desktop header mode ["fixed", "normal", "auto"]
    # 桌面端导航栏模式 ["fixed", "normal", "auto"]
    desktopMode = "fixed"
    # mobile header mode ["fixed", "normal", "auto"]
    # 移动端导航栏模式 ["fixed", "normal", "auto"]
    mobileMode = "auto"
    # Header title config
    # 页面头部导航栏标题配置
    [params.header.title]
      # URL of the LOGO
      # LOGO 的 URL
      logo = ""
      # title name
      # 标题名称
      name = "LoveIt"
      # you can add extra information before the name (HTML format is supported), such as icons
      # 你可以在名称 (允许 HTML 格式) 之前添加其他信息, 例如图标
      pre = "<i class='far fa-kiss-wink-heart fa-fw' aria-hidden='true'></i>"
      # you can add extra information after the name (HTML format is supported), such as icons
      # 你可以在名称 (允许 HTML 格式) 之后添加其他信息, 例如图标
      post = ""
      # whether to use typeit animation for title name
      # 是否为标题显示打字机动画
      typeit = false

  # Footer config
  # 页面底部信息配置
  [params.footer]
    enable = true
    # Custom content (HTML format is supported)
    # 自定义内容 (支持 HTML 格式)
    custom = ""
    # whether to show Hugo and theme info
    # 是否显示 Hugo 和主题信息
    hugo = true
    # whether to show copyright info
    # 是否显示版权信息
    copyright = true
    # whether to show the author
    # 是否显示作者
    author = true
    # site creation time
    # 网站创立年份
    since = 2019
    # ICP info only in China (HTML format is supported)
    # ICP 备案信息，仅在中国使用 (支持 HTML 格式)
    icp = ""
    # license info (HTML format is supported)
    # 许可协议信息 (支持 HTML 格式)
    license= '<a rel="license external nofollow noopener noreffer" href="https://creativecommons.org/licenses/by-nc/4.0/" target="_blank">CC BY-NC 4.0</a>'

  # Section (all posts) page config
  # Section (所有文章) 页面配置
  [params.section]
    # special amount of posts in each section page
    # section 页面每页显示文章数量
    paginate = 20
    # date format (month and day)
    # 日期格式 (月和日)
    dateFormat = "01-02"
    # amount of RSS pages
    # RSS 文章数目
    rss = 10

  # List (category or tag) page config
  # List (目录或标签) 页面配置
  [params.list]
    # special amount of posts in each list page
    # list 页面每页显示文章数量
    paginate = 20
    # date format (month and day)
    # 日期格式 (月和日)
    dateFormat = "01-02"
    # amount of RSS pages
    # RSS 文章数目
    rss = 10

  # App icon config
  # 应用图标配置
  [params.app]
    # optional site title override for the app when added to an iOS home screen or Android launcher
    # 当添加到 iOS 主屏幕或者 Android 启动器时的标题, 覆盖默认标题
    title = "LoveIt"
    # whether to omit favicon resource links
    # 是否隐藏网站图标资源链接
    noFavicon = false
    # modern SVG favicon to use in place of older style .png and .ico files
    # 更现代的 SVG 网站图标, 可替代旧的 .png 和 .ico 文件
    svgFavicon = ""
    # Android browser theme color
    # Android 浏览器主题色
    themeColor = "#ffffff"
    # Safari mask icon color
    # Safari 图标颜色
    iconColor = "#5bbad5"
    # Windows v8-11 tile color
    # Windows v8-11 磁贴颜色
    tileColor = "#da532c"

  # Search config
  # 搜索配置
  [params.search]
    enable = true
    # type of search engine ["lunr", "algolia"]
    # 搜索引擎的类型 ["lunr", "algolia"]
    type = "algolia"
    # max index length of the chunked content
    # 文章内容最长索引长度
    contentLength = 4000
    # placeholder of the search bar
    # 搜索框的占位提示语
    placeholder = ""
    # max number of results length
    # 最大结果数目
    maxResultLength = 10
    # snippet length of the result
    # 结果内容片段长度
    snippetLength = 30
    # HTML tag name of the highlight part in results
    # 搜索结果中高亮部分的 HTML 标签
    highlightTag = "em"
    # whether to use the absolute URL based on the baseURL in search index
    # 是否在搜索索引中使用基于 baseURL 的绝对路径
    absoluteURL = false
    [params.search.algolia]
      index = ""
      appID = ""
      searchKey = ""

  # Home page config
  # 主页信息设置
  [params.home]
    # amount of RSS pages
    # RSS 文章数目
    rss = 10
    # Home page profile
    # 主页个人信息
    [params.home.profile]
      enable = true
      # Gravatar Email for preferred avatar in home page
      # Gravatar 邮箱，用于优先在主页显示的头像
      gravatarEmail = ""
      # URL of avatar shown in home page
      # 主页显示头像的 URL
      avatarURL = "/images/avatar.png"
      # title shown in home page (HTML format is supported)
      # 主页显示的网站标题 (支持 HTML 格式)
      title = ""
      # subtitle shown in home page (HTML format is supported)
      # 主页显示的网站副标题 (允许 HTML 格式)
      subtitle = "A Clean, Elegant but Advanced Hugo Theme"
      # whether to use typeit animation for subtitle
      # 是否为副标题显示打字机动画
      typeit = true
      # whether to show social links
      # 是否显示社交账号
      social = true
      # disclaimer (HTML format is supported)
      # 免责声明 (支持 HTML 格式)
      disclaimer = ""
    # Home page posts
    # 主页文章列表
    [params.home.posts]
      enable = true
      # special amount of posts in each home posts page
      # 主页每页显示文章数量
      paginate = 6
  # Social config in home page
  # 主页的社交信息设置
  [params.social]
    GitHub = ""
    Linkedin = ""
    X = ""
    Instagram = ""
    Facebook = ""
    Telegram = ""
    Medium = ""
    Gitlab = ""
    Youtubelegacy = ""
    Youtubecustom = ""
    Youtubechannel = ""
    Tumblr = ""
    Quora = ""
    Keybase = ""
    Pinterest = ""
    Reddit = ""
    Codepen = ""
    FreeCodeCamp = ""
    Bitbucket = ""
    Stackoverflow = ""
    Weibo = ""
    Odnoklassniki = ""
    VK = ""
    Flickr = ""
    Xing = ""
    Snapchat = ""
    Soundcloud = ""
    Spotify = ""
    Bandcamp = ""
    Paypal = ""
    Fivehundredpx = ""
    Mix = ""
    Goodreads = ""
    Lastfm = ""
    Foursquare = ""
    Hackernews = ""
    Kickstarter = ""
    Patreon = ""
    Steam = ""
    Twitch = ""
    Strava = ""
    Skype = ""
    Whatsapp = ""
    Zhihu = ""
    Douban = ""
    Angellist = ""
    Slidershare = ""
    Jsfiddle = ""
    Deviantart = ""
    Behance = ""
    Dribbble = ""
    Wordpress = ""
    Vine = ""
    Googlescholar = ""
    Researchgate = ""
    Mastodon = ""
    Thingiverse = ""
    Devto = ""
    Gitea = ""
    XMPP = ""
    Matrix = ""
    Bilibili = ""
    Discord = ""
    DiscordInvite = ""
    Lichess = ""
    ORCID = ""
    Pleroma = ""
    Kaggle = ""
    MediaWiki = ""
    Plume = ""
    HackTheBox = ""
    RootMe= ""
    Malt = ""
    TikTok = ""
    TryHackMe = ""
    Codeberg = ""
    HuggingFace = ""
    Threads = ""
    Bluesky = ""
    Email = ""
    RSS = ""

  # Page global config
  # 文章页面全局配置
  [params.page]
    # whether to hide a page from home page
    # 是否在主页隐藏一篇文章
    hiddenFromHomePage = false
    # whether to hide a page from search results
    # 是否在搜索结果中隐藏一篇文章
    hiddenFromSearch = false
    # whether to enable twemoji
    # 是否使用 twemoji
    twemoji = false
    # whether to enable lightgallery
    # 是否使用 lightgallery
    lightgallery = false
    # whether to enable the ruby extended syntax
    # 是否使用 ruby 扩展语法
    ruby = true
    # whether to enable the fraction extended syntax
    # 是否使用 fraction 扩展语法
    fraction = true
    # whether to enable the fontawesome extended syntax
    # 是否使用 fontawesome 扩展语法
    fontawesome = true
    # whether to show link to Raw Markdown content of the content
    # 是否显示原始 Markdown 文档内容的链接
    linkToMarkdown = true
    # whether to show the full text content in RSS
    # 是否在 RSS 中显示全文内容
    rssFullText = false
    # Table of the contents config
    # 目录配置
    [params.page.toc]
      # whether to enable the table of the contents
      # 是否使用目录
      enable = true
      # whether to keep the static table of the contents in front of the post
      # 是否保持使用文章前面的静态目录
      keepStatic = false
      # whether to make the table of the contents in the sidebar automatically collapsed
      # 是否使侧边目录自动折叠展开
      auto = true
    # Code config
    # 代码配置
    [params.page.code]
      # whether to show the copy button of the code block
      # 是否显示代码块的复制按钮
      copy = true
      # the maximum number of lines of displayed code by default
      # 默认展开显示的代码行数
      maxShownLines = 50
      [params.page.code.render]
        goat = true
        mermaid = true
    # KaTeX mathematical formulas config (KaTeX https://katex.org/)
    # KaTeX 数学公式配置 (KaTeX https://katex.org/)
    [params.page.math]
      enable = false
      # default inline delimiter is $ ... $ and \( ... \)
      # 默认行内定界符是 $ ... $ 和 \( ... \)
      inlineLeftDelimiter = ""
      inlineRightDelimiter = ""
      # default block delimiter is $$ ... $$, \[ ... \], \begin{ equation} ... \end{ equation} and some other functions
      # 默认块定界符是 $$ ... $$, \[ ... \],  \begin{ equation} ... \end{ equation} 和一些其它的函数
      blockLeftDelimiter = ""
      blockRightDelimiter = ""
      # KaTeX extension copy_tex
      # KaTeX 插件 copy_tex
      copyTex = true
      # KaTeX extension mhchem
      # KaTeX 插件 mhchem
      mhchem = true
    # Mapbox GL JS config (Mapbox GL JS https://docs.mapbox.com/mapbox-gl-js)
    # Mapbox GL JS 配置 (Mapbox GL JS https://docs.mapbox.com/mapbox-gl-js)
    [params.page.mapbox]
      # access token of Mapbox GL JS
      # Mapbox GL JS 的 access token
      accessToken = "pk.eyJ1IjoiZGlsbG9uenEiLCJhIjoiY2s2czd2M2x3MDA0NjNmcGxmcjVrZmc2cyJ9.aSjv2BNuZUfARvxRYjSVZQ"
      # style for the light theme
      # 浅色主题的地图样式
      lightStyle = "mapbox://styles/mapbox/light-v10?optimize=true"
      # style for the dark theme
      # 深色主题的地图样式
      darkStyle = "mapbox://styles/mapbox/dark-v10?optimize=true"
      # whether to add NavigationControl (https://docs.mapbox.com/mapbox-gl-js/api/#navigationcontrol)
      # 是否添加 NavigationControl (https://docs.mapbox.com/mapbox-gl-js/api/#navigationcontrol)
      navigation = true
      # whether to add GeolocateControl (https://docs.mapbox.com/mapbox-gl-js/api/#geolocatecontrol)
      # 是否添加 GeolocateControl (https://docs.mapbox.com/mapbox-gl-js/api/#geolocatecontrol)
      geolocate = true
      # whether to add ScaleControl (https://docs.mapbox.com/mapbox-gl-js/api/#scalecontrol)
      # 是否添加 ScaleControl (https://docs.mapbox.com/mapbox-gl-js/api/#scalecontrol)
      scale = true
      # whether to add FullscreenControl (https://docs.mapbox.com/mapbox-gl-js/api/#fullscreencontrol)
      # 是否添加 FullscreenControl (https://docs.mapbox.com/mapbox-gl-js/api/#fullscreencontrol)
      fullscreen = true
    # Social share links in post page
    # 文章页面的分享信息设置
    [params.page.share]
      enable = true
      X = true
      Threads = true
      Facebook = true
      Linkedin = false
      Whatsapp = false
      Pinterest = false
      Tumblr = false
      HackerNews = true
      Reddit = false
      VK = false
      Buffer = false
      Xing = false
      Line = true
      Instapaper = false
      Pocket = false
      Flipboard = false
      Weibo = true
      Blogger = false
      Baidu = false
      Odnoklassniki = false
      Evernote = false
      Skype = false
      Trello = false
      Diaspora = true
      Mix = false
      Telegram = true
    # Comment config
    # 评论系统设置
    [params.page.comment]
      enable = true
      # Disqus comment config (https://disqus.com/)
      # Disqus 评论系统设置 (https://disqus.com/)
      [params.page.comment.disqus]
        enable = false
        # Disqus shortname to use Disqus in posts
        # Disqus 的 shortname，用来在文章中启用 Disqus 评论系统
        shortname = ""
      # Gitalk comment config (https://github.com/gitalk/gitalk)
      # Gitalk 评论系统设置 (https://github.com/gitalk/gitalk)
      [params.page.comment.gitalk]
        enable = false
        owner = ""
        repo = ""
        clientId = ""
        clientSecret = ""
      # Valine comment config (https://github.com/xCss/Valine)
      # Valine 评论系统设置 (https://github.com/xCss/Valine)
      [params.page.comment.valine]
        enable = true
        appId = "QGzwQXOqs5JOhN4RGPOkR2mR-MdYXbMMI"
        appKey = "WBmoGyJtbqUswvfLh6L8iEBr"
        placeholder = ""
        avatar = "mp"
        meta= ""
        pageSize = 10
        # automatically adapt the current theme i18n configuration when empty
        # 为空时自动适配当前主题 i18n 配置
        lang = ""
        visitor = true
        recordIP = true
        highlight = true
        enableQQ = false
        serverURLs = "https://valine.hugoloveit.com"
        # emoji data file name, default is "google.yml"
        # ["apple.yml", "google.yml", "facebook.yml", "twitter.yml"]
        # located in "themes/LoveIt/assets/lib/valine/emoji/" directory
        # you can store your own data files in the same path under your project:
        # "assets/lib/valine/emoji/"
        # emoji 数据文件名称, 默认是 "google.yml"
        # ["apple.yml", "google.yml", "facebook.yml", "twitter.yml"]
        # 位于 "themes/LoveIt/assets/lib/valine/emoji/" 目录
        # 可以在你的项目下相同路径存放你自己的数据文件:
        # "assets/lib/valine/emoji/"
        emoji = ""
      # Facebook comment config (https://developers.facebook.com/docs/plugins/comments)
      # Facebook 评论系统设置 (https://developers.facebook.com/docs/plugins/comments)
      [params.page.comment.facebook]
        enable = false
        width = "100%"
        numPosts = 10
        appId = ""
        # automatically adapt the current theme i18n configuration when empty
        # 为空时自动适配当前主题 i18n 配置
        languageCode = ""
      # Telegram comments config (https://comments.app/)
      # Telegram comments 评论系统设置 (https://comments.app/)
      [params.page.comment.telegram]
        enable = false
        siteID = ""
        limit = 5
        height = ""
        color = ""
        colorful = true
        dislikes = false
        outlined = false
      # Commento comment config (https://commento.io/)
      # Commento comment 评论系统设置 (https://commento.io/)
      [params.page.comment.commento]
        enable = false
      # utterances comment config (https://utteranc.es/)
      # utterances comment 评论系统设置 (https://utteranc.es/)
      [params.page.comment.utterances]
        enable = false
        # owner/repo
        repo = ""
        issueTerm = "pathname"
        label = ""
        lightTheme = "github-light"
        darkTheme = "github-dark"
      # giscus comment config (https://giscus.app/)
      # giscus comment 评论系统设置 (https://giscus.app/zh-CN)
      [params.page.comment.giscus]
        # You can refer to the official documentation of giscus to use the following configuration.
        # 你可以参考官方文档来使用下列配置
        enable = false
        repo = ""
        repoId = ""
        category = "Announcements"
        categoryId = ""
        # automatically adapt the current theme i18n configuration when empty
        # 为空时自动适配当前主题 i18n 配置
        lang = ""
        mapping = "pathname"
        reactionsEnabled = "1"
        emitMetadata = "0"
        inputPosition = "bottom"
        lazyLoading = false
        lightTheme = "light"
        darkTheme = "dark"
      # Waline comment config (https://waline.js.org/)
      # Waline 评论系统设置 (https://waline.js.org/)
      [params.page.comment.waline]
        enable = false
        serverURL = "https://love-it-waline.vercel.app"
        # automatically adapt the current theme i18n configuration when empty
        # 为空时自动适配当前主题 i18n 配置
        lang = ""
        emoji = ["https://unpkg.com/@waline/emojis@1.0.1/tw-emoji"]
    # Third-party library config
    # 第三方库配置
    [params.page.library]
      [params.page.library.css]
        # someCSS = "some.css"
        # located in "assets/" 位于 "assets/"
        # Or 或者
        # someCSS = "https://cdn.example.com/some.css"
      [params.page.library.js]
        # someJavascript = "some.js"
        # located in "assets/" 位于 "assets/"
        # Or 或者
        # someJavascript = "https://cdn.example.com/some.js"
    # Page SEO config
    # 页面 SEO 配置
    [params.page.seo]
      # image URL
      # 图片 URL
      images = []
      # Publisher info
      # 出版者信息
      [params.page.seo.publisher]
        name = "xxxx"
        logoUrl = "/images/avatar.png"

  # TypeIt config
  # TypeIt 配置
  [params.typeit]
    # typing speed between each step (measured in milliseconds)
    # 每一步的打字速度 (单位是毫秒)
    speed = 100
    # blinking speed of the cursor (measured in milliseconds)
    # 光标的闪烁速度 (单位是毫秒)
    cursorSpeed = 1000
    # character used for the cursor (HTML format is supported)
    # 光标的字符 (支持 HTML 格式)
    cursorChar = "|"
    # cursor duration after typing finishing (measured in milliseconds, "-1" means unlimited)
    # 打字结束之后光标的持续时间 (单位是毫秒, "-1" 代表无限大)
    duration = -1

  # Site verification code for Google/Bing/Yandex/Pinterest/Baidu
  # 网站验证代码，用于 Google/Bing/Yandex/Pinterest/Baidu
  [params.verification]
    google = ""
    bing = ""
    yandex = ""
    pinterest = ""
    baidu = ""

  # Site SEO config
  # 网站 SEO 配置
  [params.seo]
    # image URL
    # 图片 URL
    image = "/images/Apple-Devices-Preview.png"
    # thumbnail URL
    # 缩略图 URL
    thumbnailUrl = "/images/screenshot.png"

  # Analytics config
  # 网站分析配置
  [params.analytics]
    # Google Analytics
    [params.analytics.google]
      id = ""
      # whether to respect the browser’s “do not track” setting
      # 是否遵循浏览器的 “Do Not Track” 设置
      respectDoNotTrack = false
    # Fathom Analytics
    [params.analytics.fathom]
      id = ""
      # server url for your tracker if you're self hosting
      # 自行托管追踪器时的主机路径
      server = ""
    # Plausible Analytics
    [params.analytics.plausible]
      dataDomain = ""
    # Yandex Metrica
    [params.analytics.yandexMetrica]
      id = ""
    [params.analytics.goatCounter]
      code = ""


  # Cookie consent config
  # Cookie 许可配置
  [params.cookieconsent]
    enable = false
    # text strings used for Cookie consent banner
    # 用于 Cookie 许可横幅的文本字符串
    [params.cookieconsent.content]
      message = ""
      dismiss = ""
      link = ""

  # CDN config for third-party library files
  # 第三方库文件的 CDN 设置
  [params.cdn]
    # CDN data file name, disabled by default
    # ["jsdelivr.yml"]
    # located in "themes/LoveIt/assets/data/cdn/" directory
    # you can store your own data files in the same path under your project:
    # "assets/data/cdn/"
    # CDN 数据文件名称, 默认不启用
    # ["jsdelivr.yml"]
    # 位于 "themes/LoveIt/assets/data/cdn/" 目录
    # 可以在你的项目下相同路径存放你自己的数据文件:
    # "assets/data/cdn/"
    data = "jsdelivr.yml"

  # Compatibility config
  # 兼容性设置
  [params.compatibility]
    # whether to use Polyfill.io to be compatible with older browsers
    # 是否使用 Polyfill.io 来兼容旧式浏览器
    polyfill = false
    # whether to use object-fit-images to be compatible with older browsers
    # 是否使用 object-fit-images 来兼容旧式浏览器
    objectFit = false

# Markup related configuration in Hugo
# Hugo 解析文档的配置
[markup]
  # Syntax Highlighting (https://gohugo.io/content-management/syntax-highlighting)
  # 语法高亮设置 (https://gohugo.io/content-management/syntax-highlighting)
  [markup.highlight]
    anchorLineNos = false
    codeFences = true
    guessSyntax = false
    lineNos = false
    lineNumbersInTable = true
    noClasses = true
  # Goldmark is from Hugo 0.60 the default library used for Markdown
  # Goldmark 是 Hugo 0.60 以来的默认 Markdown 解析库
  [markup.goldmark]
    [markup.goldmark.extensions]
      definitionList = true
      footnote = true
      linkify = true
      strikethrough = true
      table = true
      taskList = true
      typographer = true
    [markup.goldmark.renderer]
      # whether to use HTML tags directly in the document
      # 是否在文档中直接使用 HTML 标签
      unsafe = true
  # Table Of Contents settings
  # 目录设置
  [markup.tableOfContents]
    startLevel = 2
    endLevel = 6

# Sitemap config
# 网站地图配置
[sitemap]
  changefreq = "weekly"
  filename = "sitemap.xml"
  priority = 0.5

# Permalinks config (https://gohugo.io/content-management/urls/#permalinks)
# Permalinks 配置 (https://gohugo.io/content-management/urls/#permalinks)
[permalinks]
  # posts = ":year/:month/:filename"
  posts = ":filename"

# Privacy config (https://gohugo.io/configuration/privacy/)
# 隐私信息配置 (https://gohugo.io/configuration/privacy/)
[privacy]
  # privacy of the Google Analytics (can also be configured in params.analytics.google)
  # Google Analytics 相关隐私设置 (也能在 params.analytics.google 配置)
  [privacy.googleAnalytics]
    # ...
  [privacy.twitter]
    # ...
  [privacy.youtube]
    # ...

# Options to make output .md files
# 用于输出 Markdown 格式文档的设置
[mediaTypes]
  [mediaTypes."text/plain"]
    suffixes = ["md"]

# Options to make output .md files
# 用于输出 Markdown 格式文档的设置
[outputFormats.MarkDown]
  mediaType = "text/plain"
  isPlainText = true
  isHTML = false

# Options to make hugo output files
# 用于 Hugo 输出文档的设置
[outputs]
  home = ["HTML", "RSS", "JSON"]
  page = ["HTML", "MarkDown"]
  section = ["HTML", "RSS"]
  taxonomy = ["HTML", "RSS"]

# Multilingual
# 多语言
[languages]
  [languages.en]
    weight = 1
    languageCode = "en"
    languageName = "English"
    hasCJKLanguage = false
    copyright = "This work is licensed under a Creative Commons Attribution-NonCommercial 4.0 International License."
    [languages.en.menu]
      [[languages.en.menu.main]]
        weight = 1
        identifier = "posts"
        pre = ""
        post = ""
        name = "Posts"
        url = "/posts/"
        title = ""
      [[languages.en.menu.main]]
        weight = 2
        identifier = "tags"
        pre = ""
        post = ""
        name = "Tags"
        url = "/tags/"
        title = ""
      [[languages.en.menu.main]]
        weight = 3
        identifier = "categories"
        pre = ""
        post = ""
        name = "Categories"
        url = "/categories/"
        title = ""
      [[languages.en.menu.main]]
        weight = 4
        identifier = "documentation"
        pre = ""
        post = ""
        name = "Docs"
        url = "/categories/documentation/"
        title = ""
      [[languages.en.menu.main]]
        weight = 5
        identifier = "about"
        pre = ""
        post = ""
        name = "About"
        url = "/about/"
        title = ""
      [[languages.en.menu.main]]
        weight = 6
        identifier = "github"
        pre = "<i class='fab fa-github fa-fw' aria-hidden='true'></i>"
        post = ""
        name = ""
        url = "https://github.com/dillonzq/LoveIt"
        title = "GitHub"
    [languages.en.params]
      [languages.en.params.search]
        enable = true
        type = "algolia"
        contentLength = 4000
        placeholder = ""
        maxResultLength = 10
        snippetLength = 30
        highlightTag = "em"
        absoluteURL = false
        [languages.en.params.search.algolia]
          index = "index"
          appID = "4D1IDY8JU6"
          searchKey = "05332ac5ed76655a511f0da583a9afac"
      [languages.en.params.home]
        rss = 10
        [languages.en.params.home.profile]
          enable = true
          gravatarEmail = ""
          avatarURL = "/images/avatar.png"
          title = ""
          subtitle = "A Clean, Elegant but Advanced Hugo Theme"
          typeit = true
          social = true
          disclaimer = ""
      [languages.en.params.social]
        GitHub = "xxxx"
        X = "xxxx"
        Instagram = "xxxx"
        Facebook = "xxxx"
        Telegram = "xxxx"
        Youtubelegacy = "xxxx"
        HuggingFace = "xxxx"
        Threads = "xxxx"
        Phone = "555-555-555"
        Email = "xxxx@xxxx.com"
        RSS = true
        [languages.en.params.social.Mastodon]
          id = "@xxxx"
          prefix = "https://mastodon.technology/"

  [languages.zh-cn]
    weight = 2
    languageCode = "zh-CN"
    languageName = "简体中文"
    hasCJKLanguage = true
    copyright = "This work is licensed under a Creative Commons Attribution-NonCommercial 4.0 International License."
    [languages.zh-cn.menu]
      [[languages.zh-cn.menu.main]]
        weight = 1
        identifier = "posts"
        pre = ""
        post = ""
        name = "所有文章"
        url = "/posts/"
        title = ""
      [[languages.zh-cn.menu.main]]
        weight = 2
        identifier = "tags"
        pre = ""
        post = ""
        name = "标签"
        url = "/tags/"
        title = ""
      [[languages.zh-cn.menu.main]]
        weight = 3
        identifier = "categories"
        pre = ""
        post = ""
        name = "分类"
        url = "/categories/"
        title = ""
      [[languages.zh-cn.menu.main]]
        weight = 4
        identifier = "documentation"
        pre = ""
        name = "文档"
        url = "/categories/documentation/"
        title = ""
      [[languages.zh-cn.menu.main]]
        weight = 5
        identifier = "about"
        pre = ""
        post = ""
        name = "关于"
        url = "/about/"
        title = ""
      [[languages.zh-cn.menu.main]]
        weight = 6
        identifier = "github"
        pre = "<i class='fab fa-github fa-fw' aria-hidden='true'></i>"
        post = ""
        name = ""
        url = "https://github.com/dillonzq/LoveIt"
        title = "GitHub"
    [languages.zh-cn.params]
      [languages.zh-cn.params.search]
        enable = true
        type = "algolia"
        contentLength = 4000
        placeholder = ""
        maxResultLength = 10
        snippetLength = 50
        highlightTag = "em"
        absoluteURL = false
        [languages.zh-cn.params.search.algolia]
          index = "index"
          appID = "GZT24PWKNT"
          searchKey = "634ab679e825b815de2663de38908f9d"
      [languages.zh-cn.params.home]
        rss = 10
        [languages.zh-cn.params.home.profile]
          enable = true
          gravatarEmail = ""
          avatarURL = "/images/avatar.png"
          title = ""
          subtitle = "一个简洁、优雅且高效的 Hugo 主题"
          typeit = true
          social = true
          disclaimer = ""
      [languages.zh-cn.params.social]
        GitHub = "xxxx"
        Weibo = "xxxx"
        Steam = "xxxx"
        Zhihu = "xxxx"
        Douban = "xxxx"
        Devto = "xxxx"
        Bilibili = "xxxx"
        Email = "xxxx@xxxx.com"
        Phone = "555-555-555"
        RSS = true
```

{{< /admonition >}}

### 3.4 本地预览

终端在项目目录下执行下列命令，如果没有报错终端会出现一个本地链接，在浏览器打开链接即可预览

```bash
hugo server # 如果想要显示草稿，使用 hugo server -D
# 会出现 Web Server is available at http://localhost:41001/ (bind address 127.0.0.1)
```

## 4 本地和远程建立连接

### 4.1 本地源码仓库与远程源码仓库链接

本地在 yaminook 文件夹下执行

```bash
git remote add origin git@github.com:ruanygf/yaminook.git
git branch -m master main # 将默认分支修改为 main 与远程保持一致
```

### 4.2 本地源码仓库与远程网页仓库链接

本地在 yaminook 文件夹下执行

```bash
# 添加配置文件
mkdir -p .github/workflows
touch .github/workflows/deploy.yml  # 新建部署脚本文件
```

将以下内容粘贴到 `deploy.yml` 中（注释部分可能需要替换关键信息），目的是当推送到 main 分支时触发执行

```bash
name: 自动部署 Hugo 网站到 GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest  
    steps:
      # 1. 拉取源文件仓库代码
      - name: 检出源文件
        uses: actions/checkout@v4
        with:
          submodules: true  
          fetch-depth: 0

      # 2. 安装 Hugo（版本与本地一致，可通过 hugo version 查看）
      - name: 安装 Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.145.0'  # 替换为本地 Hugo 版本
          extended: true  

      # 3. 构建 Hugo 静态网站（生成 public 目录）
      - name: 构建网站
        run: hugo --minify 

      # 4. 推送构建结果到 GitHub Pages 仓库
      - name: 部署到 ruanygf.github.io
        uses: peaceiris/actions-gh-pages@v4
        with:
          external_repository: ruanygf/ruanygf.github.io # # 替换为推送目标仓库（GitHub Pages 仓库）
          deploy_key: ${{ secrets.ACTION_OF_BLOG_YAMINOOK }} # 替换为身份验证 Token（需要提前创建，步骤见上文）
          publish_dir: ./public # 推送的目录（Hugo 构建后的静态文件在 public 目录，通常不需要修改）
          publish_branch: main
```

### 4.3 仓库推送

推送代码到源码仓库

```bash
git add .
git commit -m "xxxx"
git push origin main
```

查看部署状态：

- 进入源文件仓库 → 「Actions」→ 查看最新的 workflow 运行记录，若显示绿色对勾则部署成功。
- 等待一小会会发现 ruanygf.github.io 中同步了 public 目录下的内容，访问自己的博客网站（如我的是 `https://ruanygf.github.io` ），即可看到自动构建的网站（包含测试文章）。

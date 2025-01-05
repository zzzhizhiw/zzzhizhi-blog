---
title: "Hugo 博客搭建经验总结"
date: "2025-01-05"
draft: false
description: "总结了使用 PaperMod 主题搭建 Hugo 博客的过程，包括一些关键功能的配置和调试经验。"
summary: "总结了使用 PaperMod 主题搭建 Hugo 博客的过程，包括一些关键功能的配置和调试经验。"
categories: ["Web"]
tags: ["博客", "经验"]
series: []
aliases: []
---

我用的主题是 PaperMod，本文用它来作示例。

### 目录

如果要在文章页面中显示目录，请关注以下配置：

```yaml
params:
  showToc: true # 全局显示目录
  tocopen: true # 全局目录展开（目录默认是折叠列表）
```

### 多语言

这里配置很容易出问题。如果你像我一样喜欢搞多语言博客，请关注以下配置：

```yaml
defaultContentLanguage: zh-cn
defaultContentLanguageInSubdir: true
```

`defaultContentLanguage: zh-cn`

- 设置网站默认语言为简体中文
- 影响 Hugo 生成的 URL 结构
- 决定默认内容的语言标识

以上配置建议不要修改。

`defaultContentLanguageInSubdir: true`

- 控制默认语言内容是否放在子目录中
- true: 默认生成路径为 /zh-cn/posts/
- false: 默认生成路径为 /posts/

**举例：**

```yaml
# 当 defaultContentLanguageInSubdir: true
public/
├── zh-cn/          # 默认语言(中文)内容
│   └── posts/
└── en/             # 其他语言内容
    └── posts/

# 当 defaultContentLanguageInSubdir: false
public/
├── posts/          # 默认语言(中文)内容
└── en/             # 其他语言内容
    └── posts/
```

需要配置多语言 content 目录：

```yaml
languages:
  zh-cn:
    languageName: "简体中文"
    contentDir: "/content/zh-cn" # /content/<lang>
    # ... 需要做多语言配置的配置都可以写进这里
  en:
    languageName: "English"
    contentDir: "/content/en"
    # ...
```

在多语言配置下，`search.md` 和 `archive.md` 有效路径为：

```yaml
content/
├── zh-cn/
│   ├── posts/
│   ├── search.zh-cn.md # search.<lang>.md
│   └── archive.md
└── en/
    ├── posts/
    ├── search.en.md
    └── archive.md
```

以上两者的 URL 在 `hugo.yaml` 中保持一般设置即可：

```yaml
languages:
  zh-cn:
    menu:
      main:
        - identifier: "search"
          name: "搜索"
          url: "/search"
          weight: 1
        - identifier: "archives"
          name: "归档"
          url: "/archives"
          weight: 2
```

### 搜索

要支持搜索功能，必须在 `hugo.yaml` 中做如下配置：

```yaml
outputs:
  home:
    - HTML
    - RSS
    - JSON # JSON 是支持搜索的关键
```

### 字体

我想要英文使用 Ubuntu Mono，中文使用 HarmonyOS Sans SC，代码使用 JetBrains Mono，于是做了如下配置。

新建 `/themes/PaperMod/assets/css/extended/fonts.css`：

```css
@import url('https://cdn.jsdelivr.net/npm/ubuntu-mono/css/ubuntu-mono.min.css?display=swap');
@import url('https://cdn.jsdelivr.net/npm/harmonyos-sans-font/css/harmonyos-sans.min.css?display=swap');
@import url('https://cdn.jsdelivr.net/npm/jetbrains-mono/css/jetbrains-mono.min.css?display=swap');

body {
  font-family: "Ubuntu Mono", "HarmonyOS Sans SC", sans-serif;
}

code, pre {
  font-family: "JetBrains Mono", monospace;
}
```

### 代码块背景色

默认的代码块背景色亮度很低（非常黑），我修改了一下。

在 `/themes/PaperMod/assets/css/common/post-single.css` 第 206 行处，可以看到 `--code-block-bg` 变量决定了代码块背景色：

```css
.post-content pre code {
    display: grid;
    margin: auto 0;
    padding: 10px;
    color: rgb(213, 213, 214);
    background: var(--code-block-bg) !important;
    border-radius: var(--radius);
    overflow-x: auto;
    word-break: break-all;
}
```

全局搜索后发现 `/themes/PaperMod/assets/css/core/theme-vars.css` 中有两处声明了这个变量：

```css
:root {
    ...
    --code-block-bg: rgb(43, 43, 43);
    ...
}

.dark {
    ...
    --code-block-bg: rgb(43, 43, 43);
    ...
}
```

都改成 `rgb(43, 43, 43)` 就可以了。

### 评论

我使用的是 Giscus，它是利用 [GitHub Discussions](https://docs.github.com/en/discussions) 实现的评论系统。

请确保：

- 博客仓库是公开的，否则访客将无法查看 Discussions。
- [giscus](https://github.com/apps/giscus) app 已安装，否则访客将无法评论和回应。
- Discussions 功能已[在仓库中启用](https://docs.github.com/github/administering-a-repository/managing-repository-settings/enabling-or-disabling-github-discussions-for-a-repository)。

然后访问 [Giscus 网站](https://giscus.app/zh-CN)，按要求进行配置。

配置完成后，在“启用 giscus”标题下会出现形如以下内容：

```html
<script src="https://giscus.app/client.js"
        data-repo="[在此输入仓库]"
        data-repo-id="[在此输入仓库 ID]"
        data-category="[在此输入分类名]"
        data-category-id="[在此输入分类 ID]"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
</script>
```

新建 `/layouts/partials/comments.html`，将得到的 \<script> 标签复制粘贴进去。

确保 `hugo.yaml` 中启用了评论：

```yaml
params:
  comments: true
```

### 推送

敲打 `hugo -D` 命令后，Hugo 会在 `/public` 目录下构建静态博客。每次构建前一般会将 `/public` 删掉来确保新构建的内容不出问题。

在这种情况下，我希望只部署静态博客而不是整个 Hugo 框架。于是写了一个 Shell 脚本：

```shell
@echo off
hugo -D
cd public
git init
git remote add origin git@github.com:zzzhizhiw/zzzhizhi-blog.git
git checkout -b main
git fetch
git add -A
git commit -m "temp"
git reset --soft origin/main
git commit -m "update %date%"
git push
cd ..
```

结合 Cloudflare Pages 自带 CI/CD，只需一个脚本即可实现一键推送更新。

不要在 `hugo.yaml` 中配置 `baseURL`，这会导致所有部署唯一预览 URL 被重定向到 `baseURL`。

### `hugo.yaml` 示例

这是我使用的 [`hugo.yaml`](https://github.com/zzzhizhiw/zzzhizhi-blog/tree/main/hugo.yaml)。

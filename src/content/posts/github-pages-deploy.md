---
title: GitHub Pages 自动化部署完全指南
published: 2026-05-22
description: 从零开始，把 Astro 博客通过 GitHub Actions 自动部署到 GitHub Pages——面向小白的每一步
tags: [博客, GitHub Pages, GitHub Actions, CI/CD]
category: 技术
draft: false
lang: zh_CN
prevTitle: 为什么选择 Astro + Fuwari
prevSlug: why-astro
---

前两篇讲了重启博客的动机和技术选型，这篇聚焦在"怎么让它在网上被别人访问到"——也就是部署。

如果你只想快速上手，核心流程只有三步：

1. 把代码推到 GitHub
2. 在仓库设置里打开 GitHub Pages，部署源选 GitHub Actions
3. 把下面的 `.github/workflows/deploy.yml` 放到项目里，推送

剩下的全是自动的。

但如果你跟我一样，第一次做这件事时对着 YAML 文件一头雾水，不知道每个字段在干什么，遇到报错也不知道从哪查起——那么下面这份详细的说明应该能帮到你。

## GitHub Pages 是什么

GitHub Pages 是 GitHub 提供的免费静态页面托管服务。你把 HTML/CSS/JS 文件放上去，它帮你托管并提供 URL 访问。

它有几个特点：
- **免费**：只要仓库是公开的，Pages 服务不收钱
- **CDN 加速**：全球部署，国内部分地区访问速度也还可以
- **自动 HTTPS**：证书由 Let's Encrypt 自动签发和管理
- **自定义域名**：支持绑定自己的域名
- **构建上限**：每小时 10 次构建（对于个人博客完全够用）

适合托管：博客、项目文档、个人主页、作品集等纯前端站点。

GitHub Pages 分为两种：

| 类型 | 仓库名要求 | 默认域名 |
|------|-----------|---------|
| 用户/组织站点 | `用户名.github.io` | `https://用户名.github.io/` |
| 项目站点 | 任意 | `https://用户名.github.io/仓库名/` |

如果仓库名是 `用户名.github.io`，GitHub Pages 会把它当作**用户站点**，域名直接映射到根域名。其他名称则是**项目站点**，域名会带上仓库名作为路径前缀。

我的仓库是 `grwyyj/grwyyj.github.io`，属于用户站点，最终部署在 `https://grwyyj.github.io/`。

## 三种部署方式

GitHub Pages 支持三种部署源，在仓库 **Settings → Pages → Source** 中选择：

1. **Deploy from a branch**：选择一个分支（通常是 `gh-pages`），直接把这个分支的根目录内容作为站点展示。这是最传统的方式，需要本地构建后把产物提交到该分支
2. **GitHub Actions**：通过 workflow 构建并上传 artifact，GitHub 自动处理后续部署。这是目前推荐的方式
3. **Classic Pages（旧版）**：从 `main` 分支的根目录或 `/docs` 文件夹直接部署。不灵活，已逐渐被淘汰

本指南使用 **GitHub Actions** 方式，因为构建和部署完全自动化，本地只需要 `git push`。

## GitHub Actions 是什么

GitHub Actions 是 GitHub 自带的 CI/CD（持续集成/持续部署）服务。简单说就是：你在仓库里放一个配置文件，当特定事件发生时（比如有代码推送），GitHub 会在它的服务器上执行你定义的一系列操作。

对于博客部署来说，Actions 做的事情是：
1. 启动一台 Linux 虚拟机
2. 拉取代码
3. 安装 Node.js 和 pnpm
4. 安装项目依赖
5. 运行构建命令
6. 把构建产物上传
7. 把产物发布到 Pages 服务

整个过程自动完成，不需要手动操作。GitHub 给免费账户提供了每个月 2000 分钟的 Actions 运行时长——博客每次构建大约 1-2 分钟，随便用都用不完。

## 工作流文件详解

工作流文件位于 `.github/workflows/deploy.yml`。目录结构需要手动创建或由 IDE 自动补全。

完整内容如下，我们逐段拆解：

### 工作流名称和触发器

```yaml
name: Deploy to GitHub Pages
```

`name` 是工作流的显示名称，会出现在仓库的 Actions 标签页里。可以任意命名，建议跟做的事情相关。

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:
```

`on` 定义触发条件。这里有两个：
- `push` 到 `main` 分支时自动触发——也就是说你写完文章 `git push` 之后，部署就自动开始了
- `workflow_dispatch` 允许你从 GitHub 网页上手动点击"Run workflow"按钮来触发——方便测试工作流本身

如果启用了 Pull Request 流程，也可以加上 `pull_request` 触发事件，在 PR 阶段就运行构建检查，确保代码合入 main 后不会构建失败。

### 权限声明

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

GitHub Actions 运行时有权限控制。默认情况下工作流对仓库只有读取权限，但 Pages 部署需要写入权限。这三行是必须的：

- `contents: read`：读取仓库代码
- `pages: write`：向 Pages 服务推送构建产物
- `id-token: write`：生成 OIDC 身份令牌，用于验证工作流身份

缺少 `pages: write` 会导致部署时返回 403 Forbidden。如果你用的是旧版教程或模板，可能没有 `id-token: write`——新版 `actions/deploy-pages` 要求这个权限，不加会报错。

### 并发控制

```yaml
concurrency:
  group: "pages"
  cancel-in-progress: false
```

`concurrency` 确保同一时间只有一个部署在运行。如果你连续两次 `git push`，第一次部署还在跑的时候第二次触发了——`cancel-in-progress: false` 表示**不取消正在运行的部署**，让当前部署完整执行完，后续的排队等它完成再跑。

反过来，如果你只关心最新一次提交，可以把 `cancel-in-progress` 设为 `true`，这样新推送时会自动取消正在运行的旧部署，省时间也省 Actions 分钟数。

### 构建任务

工作流由多个 **job**（任务）组成，这里拆成了两个：`build` 和 `deploy`。

#### Checkout

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
```

`runs-on: ubuntu-latest` 指定运行环境。GitHub 提供 Ubuntu、Windows、macOS 三种虚拟机，Ubuntu 免费额度最高，一般项目够用。

`actions/checkout@v4` 是一个官方 action，作用是把仓库代码拉取到虚拟机里。`@v4` 是版本号，建议用主版本号（`v4`）而不是具体的 commit hash，GitHub 会确保 `v4` 系列向后兼容。

#### 安装 Node.js 和 pnpm

```yaml
      - name: Setup pnpm
        uses: pnpm/action-setup@v4
        with:
          run_install: false
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: pnpm
```

这里做了两件事：
1. 安装 pnpm（项目要求的包管理器）
2. 安装 Node.js 22 并启用 pnpm 缓存

`cache: pnpm` 是关键优化。GitHub Actions 每次运行都是全新的虚拟机，没有 `node_modules`。如果每次构建都重新下载所有依赖，100+ MB 的包下载就要花一两分钟。`cache` 会把 `node_modules` 缓存起来，后续构建直接从缓存恢复——依赖没变的话，这个步骤只需要几秒钟。

`run_install: false` 表示由 pnpm action 只负责安装 pnpm 本身，不自动运行 `pnpm install`。安装依赖交给后面的显式步骤更方便控制。

关于 **Node.js 版本**：Fuwari 要求 Node.js 18+，我用了 22（最新的 LTS 版本）。版本号要与本地开发环境一致，否则可能出现本地构建成功、CI 构建失败的情况——通常是因为某个依赖在新旧 Node 版本之间有行为差异。

#### 安装依赖

```yaml
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
```

`--frozen-lockfile`（对应 npm 的 `--frozen-lockfile`，yarn 的 `--frozen-lockfile`）要求 `pnpm-lock.yaml` 必须和 `package.json` 中声明的版本范围一致。如果一致就正常安装，如果不一致就直接报错退出。

为什么需要这个参数？因为 `pnpm-lock.yaml` 锁定了所有依赖的具体版本。如果 `package.json` 中某个依赖声明为 `^1.2.0`（允许安装 1.2.0 到 2.0.0 之间的版本），而 `pnpm-lock.yaml` 记录的是 1.3.5，那么不加 `--frozen-lockfile` 的时候，即使 lockfile 存在，pnpm 也会根据 `^1.2.0` 计算并更新。这可能安装到一个不同的次版本，而你没有在本地测试过这个版本。

用 `--frozen-lockfile` 确保 CI 环境安装的依赖**完全等于本地开发环境**。

#### 构建

```yaml
      - name: Build
        run: pnpm build
```

这里的 `pnpm build` 对应 `package.json` 中的 `scripts.build`：

```json
"build": "astro build && pagefind --site dist"
```

分两步：
1. `astro build`：Astro 读取所有 Markdown 文章、组件、样式，生成静态 HTML/CSS/JS 到 `dist/` 目录
2. `pagefind --site dist`：Pagefind 扫描 `dist/` 中的所有 HTML，解析文本内容，建立模糊搜索的索引文件

Pagefind 的索引文件也输出到 `dist/` 下（`dist/pagefind/` 目录），所以用户可以像请求普通静态资源一样获取搜索索引，不需要服务端支持。

构建过程中 Astro 会做这些事：
- 读取所有 Markdown 文件，渲染为 HTML
- 处理图片优化（如果开启了 `@astrojs/image`）
- 生成 RSS 和 Sitemap（如果配置了）
- 打包 CSS 和 JS
- 生成每个页面的预加载信息

如果文章里有语法错误或 frontmatter 字段不合法，构建会在这一步失败，错误信息会输出到日志中。

#### 上传产物

```yaml
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist
```

构建完成后，`dist/` 目录包含了站点需要的所有文件。这一步把它打包上传为一个 **artifact**（构建产物），供后续的 deploy 任务使用。

`actions/upload-pages-artifact@v3` 是 GitHub 专门为 Pages 部署设计的 action，它会做额外处理（比如压缩、添加元数据），比通用的 `upload-artifact` 更适合 Pages 场景。

`path: dist` 指定要上传的目录。如果你的 Astro 配置了不同的输出目录，改成对应的路径即可。

### 部署任务

```yaml
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**`needs: build`**：声明 `deploy` 依赖 `build`。不写的话两个 job 会并行运行，但 deploy 需要 build 产出的 artifact，所以必须等 build 完成。

**`environment`**：GitHub 的环境管理功能。`name: github-pages` 创建一个名为 "github-pages" 的部署环境，在仓库页面可以看到部署历史和状态。`url` 会显示在环境页面上，方便直接点击访问。

**`actions/deploy-pages@v4`**：官方部署 action。它会从之前上传的 artifact 中提取文件，推送到 GitHub Pages 的后端存储，然后更新 DNS 记录让域名指向最新的内容。这个过程不需要你配置任何 Pages 后端，GitHub 都处理好了。

### 完整的工作流

把上面所有段落拼起来，就是最终的工作流文件：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup pnpm
        uses: pnpm/action-setup@v4
        with:
          run_install: false
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: pnpm
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      - name: Build
        run: pnpm build
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

这个文件放到 `.github/workflows/deploy.yml` 后，推送即可触发。

## .nojekyll 的作用

GitHub Pages 背后使用 Jekyll 作为默认构建引擎。Jekyll 有一个规则：**所有以 `_` 下划线开头的文件和文件夹都会被忽略，不发布到站点中**。

而 Astro 的构建产物中，资源文件会放在类似 `dist/_astro/` 的路径下。如果 Jekyll 运行时，这些文件会被忽略，CSS/JS 加载失败，页面一片空白。

解决方案是在站点根目录放一个名为 `.nojekyll` 的空文件，告诉 GitHub Pages "不要运行 Jekyll，直接发布原始文件"。

关于 `.nojekyll` 有几个要点：
- **不需要任何内容**：空文件即可，它的存在本身就是信号
- **位置在构建产物的根目录**：对于 Astro 项目就是 `dist/` 目录下
- **自动生成**：Fuwari 的构建流程已经处理了这一点，不用手动创建
- **也可以交给 Actions**：如果模板没处理，可以在 workflow 里加一步 `run: touch dist/.nojekyll` 来手动创建

如果部署后发现页面空白，打开开发者工具看到 CSS/JS 返回 404，且 URL 中含有 `_astro` 之类的路径——90% 是 `.nojekyll` 的问题。

## 第一次部署可能遇到的问题

### Git 推送冲突

Fuwari 的模板仓库有自己的 git 提交历史。你改了配置、commit 之后，本地和远程就有了不相关的历史。推送时 Git 会拒绝：

```
! [rejected] main -> main (non-fast-forward)
error: failed to push some refs
```

如果这是新博客，不需要保留模板的提交历史，可以直接清掉重建：

```bash
rm -rf .git                          # 删除旧的 git 历史
git init                             # 重新初始化仓库
git add .
git commit -m "feat: init blog"      # 创建全新的第一个 commit
git remote add origin https://github.com/用户名/仓库名.git
git push --force origin main         # 强制推送覆盖远程
```

`--force`（或 `-f`）会强制覆盖远程仓库的内容。如果远程仓库已经有一些提交了，这些提交会被丢弃。对于新初始化的博客来说这没问题，但如果仓库里有别人的贡献或者有其他分支，慎用 force push。

> 如果你希望在 GitHub 页面而非本地初始化仓库，也可以在 GitHub 上先创建仓库，然后 `git clone` 到本地，再手动把 Fuwari 的代码复制进去。这样可以避免历史冲突，但步骤反而更多了。

### 构建失败

构建失败时，Actions 日志中会显示具体错误。常见的有：

- **TypeScript 类型错误**：`astro.config.mjs` 或 `src/config.ts` 中类型不匹配。检查配置项的字段名和类型
- **缺少依赖**：`pnpm install --frozen-lockfile` 失败，说明 `pnpm-lock.yaml` 和 `package.json` 不一致。删除 `pnpm-lock.yaml`，重新 `pnpm install` 生成新的 lockfile，提交后再推送
- **内存不足**：GitHub 免费虚拟机有 7 GB 内存，对于 Astro 博客来说完全够用。如果遇到 OOM，检查是否引入了大型二进制文件或过多的图片处理

### 部署成功但页面空白

这是最常见的问题。Actions 显示全部绿色通过，但打开域名是空白页或 404。

排查步骤：

1. **F12 打开开发者工具**，看 Console 有没有报错，Network 标签有没有 404 请求
2. **检查 `astro.config.mjs` 中的 `site` 配置**：如果是项目站点（仓库名不是 `用户名.github.io`），`site` 需要加上仓库名作为路径。例如仓库名为 `blog`，则 `site` 应为 `https://用户名.github.io/blog`
3. **确认 `.nojekyll` 存在**：访问 `https://你的域名/.nojekyll`，如果返回 404 说明没有这个文件
4. **检查资源路径**：如果 CSS/JS 文件请求的是绝对路径（`/assets/style.css`）而不是相对路径（`./assets/style.css`），可能需要设置 `base` 配置

对于 Astro 项目，`base` 配置项决定资源路径前缀。如果 `base: "/"`，资源路径是 `/assets/xxx.js`。如果 `base: "/blog"`，资源路径变成 `/blog/assets/xxx.js`。对于用户站点（`用户名.github.io`），通常 `base: "/"` 即可；对于项目站点需要额外配置。

### 缓存问题

部署成功后打开页面还是旧的——浏览器缓存了之前的页面。解决方案：

- **无痕窗口**：Ctrl+Shift+N（Chrome）或 Ctrl+Shift+P（Firefox）打开无痕窗口访问，不会使用缓存
- **硬刷新**：Ctrl+Shift+R 强制刷新页面
- **清除缓存**：DevTools → Application → Clear storage → Clear site data

Astro 构建时生成的 HTML 文件名中带有内容哈希（如 `index-abc123.css`），内容变了哈希也会变，所以理论上不会缓存错误。但浏览器缓存的是 HTML 本身，如果 HTML 被缓存了，里面的资源链接也还是旧的。上面的方法清掉 HTML 缓存即可。

## 部署策略的更多选择

上面介绍的是最主流的方案。但根据项目复杂度，还有几种变体：

### 同一工作流整合检查和部署

如果你同时想跑代码检查（lint/type-check）和构建，可以加一个 `check` job：

```yaml
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm install --frozen-lockfile
      - run: pnpm astro check    # TypeScript 类型检查
      - run: pnpm lint           # 代码风格检查

  build:
    needs: check
    # ... 同上

  deploy:
    needs: build
    # ... 同上
```

这样类型检查 → 构建 → 部署三步串行，任何一步失败都不会继续。对于多人协作的项目很有用，单人博客的话有点过于隆重了。

### 多环境部署

如果你想在正式上线之前有一个预览环境，可以配置一个 `staging` 分支：

```yaml
on:
  push:
    branches:
      - main      # 部署到生产
      - staging   # 部署到预览环境
```

配合不同的环境变量和域名。不过这需要两个 GitHub Pages 站点或者用其他托管服务，GitHub Pages 本身只支持每个仓库一个站点。

### 手动触发部署回滚

如果新部署的站点有问题，可以在 Actions 页面找到上一次成功运行的 workflow，点击 "Re-run job" 来重新部署旧版本。GitHub 会重新执行那个 workflow 并生成相同的构建产物。

## 关于 pnpm 的一些细节

项目使用 pnpm 而不是 npm 或 yarn，主要原因是 Fuwari 官方推荐，并且项目配置了 `preinstall` 钩子来强制使用 pnpm：

```json
"preinstall": "npx only-allow pnpm"
```

`only-allow` 是一个小工具，运行 `npm install` 或 `yarn` 时会直接报错退出。这是为了避免团队成员或 CI 环境意外使用不同的包管理器导致 lockfile 混乱。

pnpm 相比 npm 有以下区别：

- **速度**：pnpm 使用全局缓存和硬链接，安装速度比 npm 快得多，尤其在 CI 环境下差异明显
- **磁盘**：多个项目共享同一个缓存，省空间
- **lockfile 格式**：`pnpm-lock.yaml` 比 `package-lock.json` 更简洁，冲突更少
- **strict mode**：默认更严格，不能引用未在 `package.json` 中声明的依赖

`pnpm/action-setup` action 会自动匹配项目中 `packageManager` 字段指定的 pnpm 版本：

```json
"packageManager": "pnpm@9.14.4"
```

如果本地和 CI 的 pnpm 版本一致，可以避免很多"本地能构建、CI 构建失败"的问题。

## 关于仓库安全

公开仓库的 GitHub Actions 日志对所有人可见。注意：

- **不要**在 workflow 中硬编码密钥或 Token（通过 GitHub Secrets 传递）
- **不用**把 `.env` 文件提交到仓库（加到 `.gitignore` 里）
- Astro 博客通常不需要环境变量，但如果集成了评论区、分析工具等第三方服务，API 密钥通过 GitHub 的 Secrets → Environment 配置

如果误提交了敏感信息，立刻在 GitHub 仓库的 Settings → Secrets 中删除对应 key，并轮换相关服务的凭据。

## 部署后的验证清单

部署完成后，建议逐项检查：

- [ ] 首页正常加载，没有白屏
- [ ] 文章页面可以正常打开
- [ ] 暗色模式切换正常
- [ ] 搜索功能可以搜到文章
- [ ] 标签和分类页面显示正确
- [ ] 归档页面按时间排序正常
- [ ] RSS 订阅地址可访问
- [ ] Sitemap 可正常爬取
- [ ] 移动端显示正常（Chrome DevTools 切换设备模拟）
- [ ] 代码块高亮和复制按钮正常
- [ ] 社交链接跳转正确

这些检查做完一遍后，以后每次部署只需要关注新文章的内容是否正确展示即可。

---

至此，一个小而完整的博客就搭建完成了。后续更新只需要：

```
写文章 → git push → 自动部署 → 上线
```

三个环节，其他全是自动的。

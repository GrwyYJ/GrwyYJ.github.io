---
title: 我的 VS Code 插件清单
published: 2026-05-24
description: 整理一下目前安装的所有 VS Code 扩展，区分启用和禁用，按分类逐一说明用途
tags: [VS Code, 工具, 开发环境]
category: 工具
draft: false
lang: zh_CN
---
研究生期间断断续续攒了 120+ 个 VS Code 插件，趁毕业整理博客的机会系统地清点一遍。既是记录，也方便以后换环境时参考。

文中用 ✅ 表示当前启用的插件，⛔ 表示已禁用但保留着没卸载的。

## AI & 编程助手

| 状态 | 插件                    | 说明                                           |
| ---- | ----------------------- | ---------------------------------------------- |
| ✅    | **Claude Code**         | Anthropic 官方扩展，本博客的大部分代码由它完成 |
| ⛔    | **GitHub Copilot**      | 代码自动补全，当前禁用                         |
| ⛔    | **GitHub Copilot Chat** | Copilot 对话界面                               |
| ⛔    | **ChatGPT**             | OpenAI 官方扩展                                |
| ⛔    | **通义灵码**            | 阿里巴巴的 AI 编程助手                         |

## 主题 & 图标

| 状态 | 插件                               | 说明                           |
| ---- | ---------------------------------- | ------------------------------ |
| ✅    | **One Dark**                       | Atom 经典的暗色主题            |
| ✅    | **Dracula Official**               | Dracula 主题                   |
| ✅    | **Tokyo Night**                    | Tokyo Night 主题，目前主力在用 |
| ✅    | **Synthwave '84**                  | Retro 风格主题，霓虹效果       |
| ✅    | **Cobalt2**                        | 老牌暗色主题                   |
| ✅    | **GitHub Theme**                   | GitHub 官方主题                |
| ✅    | **Material Kit**                   | Material Design 主题           |
| ✅    | **Material Icon Theme**            | 文件图标，主力在用             |
| ✅    | **Material Product Icons**         | Material 风格产品图标          |
| ✅    | **vsc-essentials-material-themes** | Material 主题合集              |
| ✅    | **vsc-essentials-themes-core**     | 合集核心依赖                   |
| ✅    | **Peacock**                        | 不同工作区设置不同标题栏颜色   |

## Git & 版本控制

| 状态 | 插件            | 说明                                   |
| ---- | --------------- | -------------------------------------- |
| ✅    | **GitLens**     | Git 增强，行内显示提交信息、Blame 标注 |
| ✅    | **Git Graph**   | 交互式 Git 分支图谱                    |
| ✅    | **Git History** | 查看文件或分支提交历史                 |
| ✅    | **gitignore**   | 右键生成 .gitignore 模板               |
| ⛔    | **SVN**         | SVN 集成，偶尔需要用                   |

## Markdown & 写作

| 状态 | 插件                    | 说明                                       |
| ---- | ----------------------- | ------------------------------------------ |
| ✅    | **Markdown All in One** | 快捷键、目录生成、自动格式化，写博客最常用 |
| ⛔    | **markdownlint**        | Markdown 语法检查                          |
| ⛔    | **Markdown Checkbox**   | 任务列表复选框                             |
| ⛔    | **Markdown Emoji**      | Emoji 自动补全                             |
| ✅    | **Emojisense**          | 编辑器内 Emoji 提示                        |
| ✅    | **PDF**                 | 在 VS Code 里预览 PDF                      |
| ⛔    | **Preview TIFF**        | 预览 TIFF 图片                             |
| ✅    | **Code Spell Checker**  | 代码拼写检查                               |
| ✅    | **英文-汉语词典**       | 划词翻译，读英文文档用                     |
| ✅    | **Any Rule**            | 正则表达式测试和参考                       |

## Vue & 前端

| 状态 | 插件                    | 说明                                           |
| ---- | ----------------------- | ---------------------------------------------- |
| ✅    | **Volar**               | Vue 3 语言支持                                 |
| ⛔    | **Vetur**               | Vue 2 语言支持（Volar 之前的旧选择，放着备用） |
| ⛔    | **Vue Snippets**        | Vue 3 代码片段                                 |
| ⛔    | **Vue 2 Snippets**      | Vue 2 代码片段                                 |
| ⛔    | **Iconify**             | 在代码中搜索和插入图标                         |
| ⛔    | **UnoCSS**              | UnoCSS 语法提示                                |
| ✅    | **Prettier**            | 代码格式化                                     |
| ✅    | **ESLint**              | JS/TS 语法检查                                 |
| ✅    | **Stylelint**           | CSS 语法检查                                   |
| ⛔    | **StandardJS**          | Standard 风格检查                              |
| ✅    | **JavaScript Snippets** | JavaScript 代码片段                            |
| ✅    | **Auto Close Tag**      | 自动闭合 HTML/JSX 标签                         |
| ✅    | **Auto Rename Tag**     | 同步修改配对的标签名                           |
| ✅    | **HTML CSS Support**    | HTML 中 CSS class 自动补全                     |
| ✅    | **Live Server**         | 本地开发服务器，支持热更新                     |
| ✅    | **Color Highlight**     | 在代码中显示颜色预览小方块                     |
| ✅    | **SVG Previewer**       | 侧边栏预览 SVG                                 |
| ⛔    | **Path Intellisense**   | 文件路径自动补全                               |
| ⛔    | **Mithril Emmet**       | Emmet 缩写展开增强                             |
| ✅    | **JSON Editor**         | 可视化 JSON 编辑器                             |
| ✅    | **SCSS Formatter**      | scss 文件格式化                                |
| ✅    | **Stylus Assist**       | Stylus 预处理器语法支持                        |

## Python

| 状态 | 插件                           | 说明                       |
| ---- | ------------------------------ | -------------------------- |
| ✅    | **Python**                     | 微软官方的 Python 语言支持 |
| ⛔    | **Pylance**                    | Python 语言服务器，禁用中  |
| ✅    | **Python Environment Manager** | 管理 Python 虚拟环境       |
| ✅    | **Debugpy**                    | Python 调试器              |
| ⛔    | **Pylint**                     | Python 代码风格检查        |
| ✅    | **autopep8**                   | Python 代码格式化          |
| ✅    | **isort**                      | Python import 排序         |

## Java & Spring

| 状态 | 插件                               | 说明                              |
| ---- | ---------------------------------- | --------------------------------- |
| ⛔    | **Extension Pack for Java**        | Java 开发全家桶（依赖已部分禁用） |
| ⛔    | **Language Support for Java**      | Red Hat Java 语言服务器           |
| ⛔    | **Java Debug**                     | Java 调试器                       |
| ⛔    | **Java Test**                      | 运行和调试 JUnit 测试             |
| ⛔    | **Java Dependency Viewer**         | 查看依赖结构                      |
| ⛔    | **Maven for Java**                 | Maven 项目管理                    |
| ⛔    | **Gradle for Java**                | Gradle 项目管理                   |
| ⛔    | **Spring Initializr**              | 快速生成 Spring Boot 项目         |
| ✅    | **Migrate Java to Azure**          | Java 项目迁移 Azure               |
| ⛔    | **Oracle Java**                    | Oracle 官方 Java 扩展             |
| ⛔    | **IntelliCode**                    | AI 辅助代码补全和 API 示例        |
| ⛔    | **IntelliCode API Usage Examples** | 在编辑器中展示 API 使用例子       |

## C/C++ & 嵌入式

| 状态 | 插件                     | 说明                    |
| ---- | ------------------------ | ----------------------- |
| ⛔    | **C/C++**                | 微软官方 C/C++ 语言支持 |
| ⛔    | **C/C++ Extension Pack** | C/C++ 附加工具合集      |
| ⛔    | **C++ DevTools**         | C++ 开发额外工具        |
| ⛔    | **C/C++ Themes**         | C/C++ 语法高亮主题      |
| ⛔    | **CMake**                | CMake 语法高亮          |
| ⛔    | **CMake Tools**          | CMake 构建集成          |
| ⛔    | **Keil Assistant**       | Keil MDK 项目支持       |
| ✅    | **vsc-essentials**       | 嵌入式开发工具集        |

## 数据库 & 数据

| 状态 | 插件                | 说明                                         |
| ---- | ------------------- | -------------------------------------------- |
| ⛔    | **Database Client** | 数据库客户端（MySQL、PostgreSQL、SQLite 等） |
| ⛔    | **Redis**           | Redis 可视化客户端                           |
| ✅    | **Office Viewer**   | 在 VS Code 中预览 Excel、Word                |
| ⛔    | **XMind Viewer**    | 查看 XMind 思维导图                          |
| ✅    | **Rainbow CSV**     | CSV 文件列着色                               |

## Jupyter & 数据科学

| 状态 | 插件                   | 说明                    |
| ---- | ---------------------- | ----------------------- |
| ⛔    | **Jupyter**            | VS Code 中运行 Notebook |
| ⛔    | **Jupyter Keymap**     | Jupyter 快捷键映射      |
| ⛔    | **Jupyter Renderers**  | Notebook 中渲染图表     |
| ⛔    | **Jupyter Cell Tags**  | Cell 标签管理           |
| ⛔    | **Jupyter Slide Show** | Notebook 幻灯片模式     |

## 远程开发

| 状态 | 插件                      | 说明                       |
| ---- | ------------------------- | -------------------------- |
| ✅    | **Remote - SSH**          | 通过 SSH 连接远程服务器    |
| ✅    | **Remote - Containers**   | 在 Docker 容器中开发       |
| ✅    | **Remote - WSL**          | 在 WSL 中开发              |
| ✅    | **Remote Explorer**       | 远程资源管理器             |
| ✅    | **Remote Server**         | 远程服务器连接             |
| ✅    | **Remote Extension Pack** | Remote 插件合集            |
| ⛔    | **GitHub Codespaces**     | 连接 Codespaces 云开发环境 |
| ✅    | **Edge DevTools**         | 在 VS Code 中调试前端      |

## 开发效率工具

| 状态 | 插件                       | 说明                                              |
| ---- | -------------------------- | ------------------------------------------------- |
| ✅    | **Error Lens**             | 行内显示错误和警告，是提升 debug 体验最明显的插件 |
| ✅    | **Todo Tree**              | 收集代码中的 TODO/FIXME，侧边栏展示               |
| ✅    | **Bookmarks**              | 代码书签，快速跳转                                |
| ✅    | **Better Comments**        | 注释着色分类                                      |
| ✅    | **Code Runner**            | 在编辑器里直接运行代码片段                        |
| ✅    | **Indent Rainbow**         | 缩进层次着色                                      |
| ✅    | **CodeSnap**               | 格式化代码截图                                    |
| ✅    | **LiveCode**               | 实时运行代码显示结果                              |
| ✅    | **File Utils**             | 文件操作增强                                      |
| ✅    | **Terminal in Status Bar** | 状态栏显示终端按钮                                |
| ✅    | **Format Toggle**          | 一键开关自动格式化                                |
| ✅    | **Line Counter**           | 项目代码行数统计                                  |
| ✅    | **Output Colorizer**       | 终端输出着色                                      |
| ✅    | **Image Preview**          | 鼠标悬停显示图片预览                              |
| ✅    | **Open in Browser**        | 在浏览器中打开 HTML                               |
| ✅    | **quicktype**              | 从 JSON 快速生成类型定义                          |

## 其他实用工具

| 状态 | 插件             | 说明                     |
| ---- | ---------------- | ------------------------ |
| ✅    | **中文语言包**   | VS Code 简体中文界面     |
| ✅    | **i18n Ally**    | 国际化翻译管理           |
| ✅    | **DotENV**       | .env 文件语法高亮        |
| ✅    | **XML**          | XML 格式化与验证         |
| ✅    | **LeetCode**     | 在 VS Code 中刷 LeetCode |
| ✅    | **XMind Viewer** | 查看 XMind 思维导图文件  |

## 做个减法

120+ 个插件，禁用了 46 个。启用率大概 60% 出头，不算健康。

禁用的一般是这几类：

- **特定课程或项目装的**：如 Keil 助手、Jupyter 全家桶、Java 调试工具。项目做完就再也没打开过
- **模块功能被内置替代的**：如 markdownlint（Markdown All in One 够用了）、Path Intellisense（VS Code 自带路径提示）
- **留作备用的**：如 Vetur（Vue 2 旧项目）、SVN（偶尔签出）、GitHub Copilot（跟 Claude Code 交替用）

有空应该来一轮大扫除，把确定不会再用的直接卸载。

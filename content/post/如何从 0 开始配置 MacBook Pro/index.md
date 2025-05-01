---
title: 如何从 0 开始配置 MacBook Pro
description: "本文将引导你从 0 开始配置 MacBook Pro，从必备软件安装到深度系统优化，将这台新设备锻造成专属于你的、无懈可击的开发利器与生产力猛兽。"
date: 2025-01-07
image: banner.png
categories: [ "个人经验" ]
---

# 如何从 0 开始配置 MacBook Pro

最近我入手了新款 MacBook Pro 16，性能确实让人印象深刻。但拿到新机的第一件事，往往不是跑分或体验新功能，而是把它从“出厂设置”调教成自己用着顺手、能高效工作的状态。这个过程并不算短，我大概花了几个小时才基本配置完成。

为了让后来者（或许就是正在看这篇文章的你）能少走些弯路，我把整个配置过程详细记录了下来，并结合了之前的一些笔记和最新的实践。这份指南涵盖了从网络、工具安装到系统细节优化的方方面面，希望能帮你更快地让你的新 Mac 变成得心应手的生产力伙伴。

## 目录

1.  [初始设置：网络代理](#1-初始设置网络代理)
2.  [核心工具：Homebrew](#2-核心工具homebrew)
3.  [安装应用程序和命令行工具](#3-安装应用程序和命令行工具)
4.  [准备本地目录结构](#4-准备本地目录结构)
5.  [配置核心 App](#5-配置核心-app)
    - [Raycast (启动器与效率工具)](#raycast-启动器与效率工具)
    - [Warp (终端) 与 Zsh (Shell环境)](#warp-终端-与-zsh-shell环境)
    - [SSH 密钥配置](#ssh-密钥配置)
    - [安装字体](#安装字体)
    - [WebStorm (IDE)](#webstorm-ide)
    - [Visual Studio Code (编辑器)](#visual-studio-code-编辑器)
    - [Git (版本控制)](#git-版本控制)
    - [NVM 与 Node.js](#nvm-与-nodejs)
    - [Bob (翻译与 OCR)](#bob-翻译与-ocr)
    - [Obsidian (笔记软件)](#obsidian-笔记软件)
    - [SourceTree (Git GUI)](#sourcetree-git-gui)
6.  [系统设置优化](#6-系统设置优化)

---

## 1. 初始设置：网络代理

在中国大陆或其他网络受限的环境下，配置代理是顺利进行后续步骤（如下载 Homebrew、更新软件源等）的关键。

- **安装代理工具**:

  - 你需要从你的代理服务提供商获取配置文件才能使用。
  - **重要**: 在进行 Homebrew 安装和后续的网络相关操作前，请确保你的代理客户端已启动并设置为系统代理，或在终端中配置了代理环境变量。

- **终端临时代理设置** (根据你的代理客户端设置的端口调整，常用的代理工具默认为 7890):
  ```bash
  export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
  ```
  _注意：这只对当前终端会话有效。推荐在 `.zshrc` 中设置别名或函数来快速开关代理（详见 Zsh 配置部分）。_

## 2. 核心工具：Homebrew

Homebrew 是 macOS 的包管理器，用于安装和管理各种软件。

- **安装 Homebrew**:

  - 首先，确保安装了 Xcode Command Line Tools (如果未安装，Homebrew 安装脚本会自动提示安装)：
    ```bash
    xcode-select --install
    ```
  - 然后运行官方安装脚本 (请确保代理已开启):
    ```bash
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```
  - 安装完成后，根据提示将 Homebrew 添加到 PATH (通常会自动在 `.zprofile` 或 `.zshrc` 中添加)。

- **配置 Homebrew 国内镜像源 (可选，提高速度)**:

  - 如果官方源下载缓慢，可以配置国内镜像。以下提供 USTC 和 Aliyun 的配置方法，请选择其一添加到你的 `~/.zshrc` 或 `~/.zshenv` 文件中。
  - **USTC 镜像**:

    ```bash
    export HOMEBREW_INSTALL_FROM_API=1
    export HOMEBREW_API_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles/api"
    export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles"
    export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
    export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"

    ```

  - **Aliyun 镜像**:

    ```bash
    export HOMEBREW_INSTALL_FROM_API=1
    export HOMEBREW_API_DOMAIN="https://mirrors.aliyun.com/homebrew-bottles/api"
    export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.aliyun.com/homebrew-bottles"
    export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.aliyun.com/homebrew/brew.git"
    export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.aliyun.com/homebrew/homebrew-core.git"
    ```

  - 配置后，运行 `source ~/.zshrc` (或相应文件) 和 `brew update` 使配置生效。

## 3. 安装应用程序和命令行工具

使用 Homebrew 可以方便地安装 GUI 应用 (Casks) 和命令行工具 (Formulae)。

- **查找可用包**:

  - GUI 应用: [Homebrew Cask Search](https://formulae.brew.sh/cask/)
  - 命令行工具: [Homebrew Core Search](https://formulae.brew.sh/formula/)

- **安装 Casks (GUI 应用)**:

  - 下面是你提供的 Cask 列表，包含了常用的开发、效率和日常应用。
    - `adobe-creative-cloud`: Adobe 创意云套件
    - `adobe-creative-cloud-cleaner-tool`: Adobe 清理工具
    - `adrive`: 阿里云盘
    - `android-platform-tools`: Android SDK Platform-Tools
    - `android-studio`: Android 开发 IDE
    - `baidunetdisk`: 百度网盘
    - `bob`: 翻译和 OCR 工具
    - `claude`: Claude 客户端
    - `clion`: C/C++ IDE
    - `cursor`: AI 代码编辑器
    - `datagrip`: 数据库 IDE
    - `feishu`: 飞书
    - `figma`: UI 设计工具
    - `flutter`: Flutter SDK
    - `git-credential-manager`: Git 凭证管理器
    - `github`: GitHub Desktop 客户端
    - `google-chrome`: 谷歌浏览器
    - `iina`: 现代视频播放器
    - `intellij-idea`: Java IDE
    - `motrix`: 下载工具
    - `obs`: 直播和录屏软件
    - `obsidian`: 笔记软件
    - `ollama`: 本地运行大型语言模型
    - `orbstack`: Docker Desktop 替代品
    - `postman`: API 测试工具
    - `pycharm`: Python IDE
    - `raycast`: 启动器与效率工具
    - `rustrover`: Rust IDE
    - `sourcetree`: Git GUI 客户端
    - `steam`: 游戏平台
    - `tencent-lemon`: 腾讯柠檬清理
    - `visual-studio-code`: 代码编辑器
    - `vmware-fusion`: 虚拟机软件
    - `warp`: 现代终端
    - `webstorm`: 前端 IDE
    - `wechatwebdevtools`: 微信开发者工具
    - `wireshark`: 网络协议分析器
  - **安装命令 (确保代理已开启)**:

  ```bash
  brew install --cask \
    adobe-creative-cloud \
    adobe-creative-cloud-cleaner-tool \
    adrive \
    android-platform-tools \
    android-studio \
    baidunetdisk \
    battle-net \
    bob \
    claude \
    clion \
    cursor \
    datagrip \
    feishu \
    figma \
    flutter \
    git-credential-manager \
    github \
    google-chrome \
    gstreamer-runtime \
    hbuilderx \
    iina \
    intellij-idea \
    motrix \
    obs \
    obsidian \
    ollama \
    orbstack \
    postman \
    pycharm \
    raspberry-pi-imager \
    raycast \
    rustrover \
    sourcetree \
    steam \
    tencent-lemon \
    visual-studio-code \
    vmware-fusion \
    warp \
    webstorm \
    wechatwebdevtools \
    wireshark
  ```

- **安装 Formulae (命令行工具)**:
  - 以下是你原始列表中的 CLI 工具，也移除了内部注释。
    - `autojump`: 快速目录跳转工具
    - `bat`: `cat` 的替代品，带语法高亮和 Git 集成
    - `diff-so-fancy`: 增强 `git diff` 输出的可读性
    - `fd`: `find` 的替代品，简单快速
    - `ffmpeg`: 音视频处理工具
    - `fzf`: 通用的命令行模糊查找器
    - `go`: Go 语言开发环境
    - `gh`: GitHub 官方命令行工具
    - `git`: 版本控制系统 (Homebrew 会安装更新版本)
    - `mkcert`: 创建本地信任的开发证书
    - `nvm`: Node 版本管理器
    - `openjdk`: Java 开发环境
    - `pnpm`: 高效的 Node 包管理器
    - `tldr`: 社区驱动的简化 man pages
    - `tree`: 显示目录树结构
    - `wget`: 网络下载工具
    - `yarn`: Node 包管理器
  - **安装命令 (确保代理已开启)**:
  ```bash
  brew install \
    autojump \
    bat \
    diff-so-fancy \
    fd \
    ffmpeg \
    fzf \
    go \
    gh \
    git \
    mkcert \
    nvm \
    openjdk \
    pnpm \
    tldr \
    tree \
    wget
  ```

* **(可选) 安装 `ni` 和 `live-server`**:

  - 这两个工具在 `.zshrc` 的别名配置中被引用。

  ```bash
  npm install -g @antfu/ni # 统一的包管理器命令
  npm install -g live-server # 简单的本地开发服务器
  ```

- **通过 Mac App Store 安装**:

  - VidHub
  - Bob
  - _根据需要添加其他应用_

- **手动安装或通过其他渠道**:

  - Bartender (菜单栏管理)

  * Downie (视频下载)
  * Typora (Markdown 编辑器)
  * _根据需要添加其他应用_

## 4. 准备本地目录结构

良好的目录结构有助于管理代码和配置文件。

- **建议结构**:
  - `~/Code`: 用于存放所有代码仓库。
  - `~/Documents/SoftwareConfiguration`: 用于存放软件的配置文件 (如 Raycast, VS Code 设置等)。
  - _你可以根据自己的习惯调整路径。_

## 5. 配置核心 App

按照推荐顺序配置，可以减少重复设置。

### Raycast (启动器与效率工具)

- **配置**:
  - 登录账号同步配置 (如果之前使用过)。
  - 设置 **Snippets** (文本片段)。
  - **优化快捷键**: 分配易于记忆且不冲突的快捷键给常用应用或 Raycast 功能。
    - 示例：
      - `⌥ + D`：启动开发工具，如 **DataGrip**。
      - `⌥ + G`：启动 **GoLand**。
      - `⌥ + W`：打开 **WebStorm** 或其他开发工具。
      - `⌥ + V`：快速打开 **微信**，方便日常沟通。
  - 探索 Raycast Store 安装所需扩展。
- **官网**: [Raycast](https://www.raycast.com/)

### Warp (终端) 与 Zsh (Shell环境)

提供现代化、高效的命令行体验。

- **安装 Oh My Zsh** (流行的 Zsh 配置框架):

  ```bash
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
  omz update
  source ~/.zshrc
  ```

* **安装 Starship** (跨 Shell 的极速、可定制 Prompt):

  ```bash
  brew install starship
  # 将以下行添加到 ~/.zshrc 文件末尾
  echo 'eval "$(starship init zsh)"' >> ~/.zshrc
  ```

  - 官网: [Starship](https://starship.rs/)

- **安装 Zsh 插件** (增强 Shell 功能):

  ```bash
  # Clone recommended Zsh plugins into Oh My Zsh custom plugins directory
  # 将推荐的 Zsh 插件克隆到 Oh My Zsh 自定义插件目录

  # Auto suggestions based on history / 基于历史记录的自动建议
  git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

  # Syntax highlighting for commands / 命令语法高亮
  git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

  # Additional completions for Zsh / Zsh 补全增强
  git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions

  # Fast directory jumping based on frequency / 基于使用频率的快速目录跳转 (frecent)
  git clone https://github.com/agkozak/zsh-z $ZSH_CUSTOM/plugins/zsh-z
  ```

* **配置 `~/.zshrc`** :

  ```bash
  # -------------------------------- #
  # Homebrew Configuration
  # -------------------------------- #
  export HOMEBREW_INSTALL_FROM_API=1
  export HOMEBREW_API_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles/api"
  export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles"
  export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
  export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"

  # -------------------------------- #
  # Oh-My-Zsh Configuration
  # -------------------------------- #

  # Set the path to the Oh My Zsh installation directory.
  # 设置 Oh My Zsh 的安装目录路径。
  export ZSH="$HOME/.oh-my-zsh"

  # -------------------------------- #
  # Oh-My-Zsh Theme and Plugins
  # -------------------------------- #

  # git clone https://github.com/denysdovhan/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt" --depth=1
  # ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"
  ZSH_THEME="spaceship"

  # -------------------------------- #
  # Plugins
  # Define which plugins to load for Oh My Zsh.
  # -------------------------------- #

  # git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
  # git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
  # git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions
  # git clone https://github.com/agkozak/zsh-z $ZSH_CUSTOM/plugins/zsh-z
  plugins=(
    git
    zsh-autosuggestions
    zsh-completions
    zsh-syntax-highlighting
    zsh-z
  )

  # Load Oh My Zsh. This should be done after setting ZSH_THEME and plugins.
  # 加载 Oh My Zsh。应在设置 ZSH_THEME 和 plugins 后执行。
  # Documentation: [https://ohmyz.sh/](https://ohmyz.sh/)
  source $ZSH/oh-my-zsh.sh

  # -------------------------------- #
  # Environment Variables / 环境变量配置
  # -------------------------------- #

  # -- Java Environment --
  # Add Homebrew installed OpenJDK to the PATH.
  # 将 Homebrew 安装的 OpenJDK 添加到 PATH 环境变量。
  # Assumes OpenJDK was installed via `brew install openjdk`.
  # 假设已通过 `brew install openjdk` 安装。
  export PATH="/opt/homebrew/opt/openjdk/bin:$PATH"

  # -- Go (Golang) Development Environment --
  # Set GOPATH: The root of your Go workspace. Recommended: $HOME/go.
  # 设置 GOPATH: Go 工作区的根目录。推荐设置为 $HOME/go。
  export GOPATH="$HOME/go"
  # Add the Go workspace's bin directory to PATH for running installed binaries.
  # 将 Go 工作区的 bin 目录添加到 PATH，以便运行 `go install` 安装的程序。
  export PATH="$PATH:$GOPATH/bin"
  # GOROOT is usually managed automatically by Go, no need to set it manually.
  # GOROOT 通常由 Go 自动管理，无需手动设置。

  # -- Node Version Manager (NVM) --
  # Set the directory where NVM stores Node versions.
  # 设置 NVM 存储 Node 版本的目录。
  export NVM_DIR="$HOME/.nvm"
  # Load NVM script if it exists.
  # 如果 NVM 脚本存在则加载。
  [ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"
  # Load NVM bash_completion script if it exists.
  # 如果 NVM bash_completion 脚本存在则加载。
  [ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"

  # -------------------------------- #
  # Starship Prompt Initialization / Starship 提示符初始化
  # -------------------------------- #
  # Initialize Starship prompt. This overrides the ZSH_THEME prompt.
  # 初始化 Starship 提示符。这将覆盖 ZSH_THEME 设置的提示符。
  eval "$(starship init zsh)"

  # -------------------------------- #
  # Aliases and Functions / 别名与函数
  # -------------------------------- #

  # -- Node Package Manager Aliases (using antfu/ni) --
  # Requires `npm install -g @antfu/ni`. Provides a unified interface (ni, nu, nr, etc.).
  # 需要先执行 `npm install -g @antfu/ni`。提供统一的包管理命令 (ni, nu, nr 等)。
  # Source: [https://github.com/antfu/ni](https://github.com/antfu/ni)
  alias ni='ni'                # Install dependencies / 安装依赖
  alias nu='ni -u'             # Update dependencies / 更新依赖
  alias nci='ni --frozen'      # Clean install (using lockfile) / 清洁安装 (使用锁文件)
  alias na='ni add'            # Add dependency / 添加依赖
  alias nr='nr'                # Run script / 运行脚本
  alias nio="ni --prefer-offline" # Install preferring offline cache / 优先使用离线缓存安装
  alias s="nr start"           # Alias for `nr start` / `nr start` 的别名
  alias d="nr dev"             # Alias for `nr dev`
  alias b="nr build"           # Alias for `nr build`
  alias bw="nr build --watch"  # Alias for `nr build --watch`
  alias t="nr test"            # Alias for `nr test`
  alias tu="nr test -u"        # Alias for `nr test -u` (update snapshots)
  alias tw="nr test --watch"   # Alias for `nr test --watch`
  alias w="nr watch"           # Alias for `nr watch`
  alias c="nr typecheck"       # Alias for `nr typecheck`
  alias lint="nr lint"         # Alias for `nr lint`
  alias lintf="nr lint --fix"  # Alias for `nr lint --fix`
  alias release="nr release"   # Alias for `nr release`
  alias re="nr release"        # Shorter alias for `nr release`
  alias up="taze latest -r -w"

  # -- Git Aliases --

  # Go to the top level of the git repository / 跳转到 Git 仓库的根目录
  alias grt='cd "$(git rev-parse --show-toplevel)"'

  # Common Git commands / 常用 Git 命令别名
  alias gs='git status'
  alias gp='git push'
  alias gpf='git push --force' # Force push (use with caution!) / 强制推送 (谨慎使用!)
  alias gpft='git push --follow-tags' # Push commits and tags / 推送提交和标签
  alias gpl='git pull --rebase' # Pull with rebase / 使用 rebase 方式拉取更新
  alias gcl='git clone'
  alias gst='git stash' # Stash changes / 暂存更改
  alias grm='git rm'    # Remove file from git / 从 Git 中移除文件
  alias gmv='git mv'    # Move/rename file in git / 在 Git 中移动/重命名文件

  # Branch switching / 分支切换
  alias main='git checkout main' # Switch to main branch (or master) / 切换到 main 分支 (或 master)
  alias gco='git checkout'      # Alias for checkout / checkout 的别名
  alias gcob='git checkout -b' # Create and switch to a new branch / 创建并切换到新分支

  # Branch management / 分支管理
  alias gb='git branch'         # List branches / 列出分支
  alias gbd='git branch -d'     # Delete branch / 删除分支

  # Rebasing / Rebase 操作
  alias grb='git rebase'
  alias grbom='git rebase origin/main' # Rebase onto remote main/master / Rebase 到远程 main/master 分支
  alias grbc='git rebase --continue'   # Continue rebase after resolving conflicts / 解决冲突后继续 rebase

  # Logging / 查看日志
  alias gl='git log'
  alias glo='git log --oneline --graph' # Compact graph log / 紧凑的图形化日志

  # Resetting / 重置操作
  alias grh='git reset HEAD'   # Unstage changes / 取消暂存更改
  alias grh1='git reset HEAD~1' # Unstage last commit / 取消最后一次提交 (保留更改)

  # Adding changes / 添加更改
  alias ga='git add'
  alias gA='git add -A' # Add all changes (new, modified, deleted) / 添加所有更改 (新增、修改、删除)

  # Committing / 提交操作
  alias gc='git commit'
  alias gcm='git commit -m' # Commit with message / 带信息提交
  alias gca='git commit -a' # Add and commit modified/deleted files (not new) / 添加并提交修改/删除的文件 (不包括新文件)
  alias gcam='git add -A && git commit -m' # Add all and commit with message / 添加所有文件并带信息提交

  # Fetching and Rebasing / 获取并 Rebase
  alias gfrb='git fetch origin && git rebase origin/main' # Fetch from origin and rebase onto main/master / 从 origin 获取并 rebase 到 main/master

  # Cleaning working directory / 清理工作目录
  alias gxn='git clean -dn' # Dry run clean (show files to be removed) / 模拟清理 (显示将移除的文件)
  alias gx='git clean -df'  # Force clean (remove untracked files/dirs) / 强制清理 (移除未跟踪的文件/目录)

  # Utility aliases / 实用别名
  alias gsha='git rev-parse HEAD | pbcopy' # Copy current commit SHA to clipboard / 复制当前提交 SHA 到剪贴板
  alias ghci='gh run list -L 1' # List latest GitHub Actions run / 列出最新的 GitHub Actions 运行记录

  # Git log pretty function / 美化 Git 日志函数
  # Usage: glp 5 (shows last 5 commits) / 用法: glp 5 (显示最近 5 次提交)
  function glp() {
    git --no-pager log -"$1"
  }

  # Git diff function with diff-so-fancy / 使用 diff-so-fancy 的 Git diff 函数
  # Requires `brew install diff-so-fancy` / 需要 `brew install diff-so-fancy`
  # Usage: gd (diff unstaged) or gd HEAD~1 (diff against commit) / 用法: gd (对比未暂存) 或 gd HEAD~1 (对比特定提交)
  function gd() {
    if [[ -z "$1" ]]; then
      git diff --color | diff-so-fancy
    else
      git diff --color "$1" | diff-so-fancy
    fi
  }

  # Git diff cached function with diff-so-fancy / 使用 diff-so-fancy 的 Git diff --cached 函数
  # Usage: gdc (diff staged changes) / 用法: gdc (对比已暂存的更改)
  function gdc() {
    if [[ -z "$1" ]]; then
      git diff --color --cached | diff-so-fancy
    else
      git diff --color --cached "$1" | diff-so-fancy
    fi
  }

  # GitHub CLI Pull Request function / GitHub CLI Pull Request 函数
  # Usage: pr ls (list PRs) or pr 123 (checkout PR #123) / 用法: pr ls (列出PR) 或 pr 123 (切换到 PR #123)
  function pr() {
    if [ "$1" = "ls" ]; then
      gh pr list
    else
      gh pr checkout "$1"
    fi
  }

  # -- Directory Navigation Functions --
  # Custom functions based on your preferred ~/Code structure.
  # 基于你偏好的 ~/Code 目录结构的自定义函数。
  # Adjust paths if your structure is different. / 如果你的结构不同，请调整路径。

  # Navigate to project directory under ~/Code/projects
  # 跳转到 ~/Code/projects 下的项目目录
  function pj() {
    cd ~/Code/projects/"$1"
  }
  # Navigate to fork directory under ~/Code/forks
  # 跳转到 ~/Code/forks 下的 fork 目录
  function fk() {
    cd ~/Code/forks/"$1"
  }
  # Navigate to reproduction directory under ~/Code/repros
  # 跳转到 ~/Code/repros 下的复现目录
  function rp() {
    cd ~/Code/repros/"$1"
  }

  # -- Clone & Open Functions --
  # Requires GitHub CLI (`gh`) and your IDE's command-line launcher (e.g., `webstorm`, `code`).
  # 需要 GitHub CLI (`gh`) 和你的 IDE 命令行启动器 (例如 `webstorm`, `code`)。
  # Ensure the IDE launcher is in your PATH. / 确保 IDE 启动器在你的 PATH 中。

  # Improved clone function using gh cli / 使用 gh cli 改进的 clone 函数
  function clone() {
    local repo_url=$1
    # Extract repo name as default target directory / 提取仓库名作为默认目标目录
    local target_dir=${2:-$(basename "$repo_url" .git)}
    echo "Cloning $repo_url into $target_dir..."
    # Clone using gh and cd into it, return 1 on failure / 使用 gh 克隆并进入目录，失败时返回 1
    gh repo clone "$repo_url" "$target_dir" && cd "$target_dir" || return 1
  }

  # Clone to 'projects', open in WebStorm, return to previous dir / 克隆到 'projects' 目录, 用 WebStorm 打开, 返回之前目录
  function cpj() {
    local target_path=~/Code/projects # Adjust path if needed / 如有需要调整路径
    # Use subshell (...) to isolate cd and clone; && ensures webstorm runs only on success
    # 使用子 shell (...) 隔离 cd 和 clone；&& 确保只有 clone 成功才执行 webstorm
    (cd "$target_path" && clone "$@" && webstorm .)
  }
  # Clone to 'forks', open in WebStorm, return to previous dir / 克隆到 'forks' 目录...
  function cfk() {
    local target_path=~/Code/forks # Adjust path if needed / 如有需要调整路径
    (cd "$target_path" && clone "$@" && webstorm .)
  }
  # Clone to 'repros', open in WebStorm, return to previous dir / 克隆到 'repros' 目录...
  function crp() {
    local target_path=~/Code/repros # Adjust path if needed / 如有需要调整路径
    (cd "$target_path" && clone "$@" && webstorm .)
  }

  # -- Utility Functions --

  # Create a directory and change into it / 创建目录并进入
  # Usage: mkcd new_directory / 用法: mkcd new_directory
  function mkcd() {
    mkdir -p "$1" && cd "$1"
  }

  # Use live-server to serve a directory / 使用 live-server 提供目录服务
  # Requires `npm install -g live-server` / 需要 `npm install -g live-server`
  # Usage: serve (serves ./dist) or serve public (serves ./public) / 用法: serve (服务 ./dist) 或 serve public (服务 ./public)
  function serve() {
    if [[ -z "$1" ]]; then
      live-server dist # Default to serving 'dist' directory / 默认服务 'dist' 目录
    else
      live-server "$1"
    fi
  }

  # -------------------------------- #
  # Proxy Management / 代理管理
  # -------------------------------- #
  # Functions to quickly enable/disable proxy environment variables.
  # 快速启用/禁用代理环境变量的函数。
  # Adjust port numbers if your proxy uses different ones. / 如果你的代理使用不同端口，请调整端口号。

  # Function to enable proxy / 启用代理函数
  function p() {
    export https_proxy=http://127.0.0.1:7890
    export http_proxy=http://127.0.0.1:7890
    export all_proxy=socks5://127.0.0.1:7890
    echo "Proxy enabled: http://127.0.0.1:7890"
  }

  # Function to disable the proxy manually / 手动禁用代理函数
  function dp() {
    unset https_proxy
    unset http_proxy
    unset all_proxy
    echo "Proxy disabled"
  }

  # Enable Proxy at terminal start (optional, uncomment to enable)
  # 在终端启动时启用代理 (可选, 取消注释以启用)
  # p

  # --- Ensure NVM is loaded late (important for some setups) ---
  # --- 确保 NVM 在较后加载 (对某些设置很重要) ---
  # Source NVM scripts again to make sure they are available after all other initializations.
  # 再次 source NVM 脚本以确保它们在所有其他初始化之后可用。
  [ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"
  [ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"

  # End of ~/.zshrc

  # -------------------------------- #
  # Ollama Configuration
  # -------------------------------- #
  launchctl setenv OLLAMA_ORIGINS "*"
  ___MY_VMOPTIONS_SHELL_FILE="${HOME}/.jetbrains.vmoptions.sh"; if [ -f "${___MY_VMOPTIONS_SHELL_FILE}" ]; then . "${___MY_VMOPTIONS_SHELL_FILE}"; fi

  setopt no_nomatch
  ```

  - **重要**: 配置完成后，运行 `source ~/.zshrc` 使更改生效。

### SSH 密钥配置

用于安全连接 GitHub 等服务。

- **生成 SSH 密钥** (推荐使用 ed25519):

  ```bash
  mkdir -p ~/.ssh && chmod 700 ~/.ssh
  # 使用你的 GitHub 邮箱替换 "your_email@example.com"
  # 可以为 key 文件指定一个名称，如 github_id_ed25519
  ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/github_id_ed25519
  # 按提示设置密码 (强烈推荐)
  ```

- **配置 SSH Agent** (让系统记住密码):

  - 编辑或创建 `~/.ssh/config` 文件，添加以下内容:
    ```
    Host github.com
      AddKeysToAgent yes
      UseKeychain yes
      IdentityFile ~/.ssh/github_id_ed25519
    ```
  - 将密钥添加到 macOS Keychain:
    ```bash
    ssh-add --apple-use-keychain ~/.ssh/github_id_ed25519
    ```

- **添加到 GitHub**:
  ```bash
  # 确保已安装 gh (brew install gh) 并登录
  gh auth login
  # 将公钥添加到 GitHub，设置一个标题
  gh ssh-key add ~/.ssh/github_id_ed25519.pub -t "My MacBook Pro Key"
  ```

### 安装字体

选择合适的编程字体和中文字体可以提升阅读体验。

- **推荐字体**:
  - 编程字体: [JetBrains Mono](https://www.jetbrains.com/lp/mono/) (免费)
- **安装方法**: 下载字体文件 (通常是 `.ttf` 或 `.otf`)，双击打开，在弹出的 **字体册 (Font Book)** 应用中点击“安装字体”。

### WebStorm (IDE)

JetBrains 的前端开发 IDE。

- **推荐配置**:
  - **插件**: 安装常用插件，如:
    - [GitHub Copilot](https://plugins.jetbrains.com/plugin/17718-github-copilot)
    - [Atom Material Icons](https://plugins.jetbrains.com/plugin/10044-atom-material-icons) (或类似的图标主题)
    - [Rainbow Brackets](https://plugins.jetbrains.com/plugin/10080-rainbow-brackets) (彩色括号)
    - [Translation](https://plugins.jetbrains.com/plugin/8579-translation) (翻译插件)
    - [UnoCSS](https://plugins.jetbrains.com/plugin/18421-unocss) (如果你使用 UnoCSS)
    - _根据需要安装其他插件 (如 Prettier, ESLint 等)_
  - **编辑器设置**:
    - `Settings/Preferences` > `Editor` > `General` > `Code Completion`: 取消勾选 `Match case` (代码补全不区分大小写)。
    - `Settings/Preferences` > `Appearance & Behavior` > `Appearance`: 选择喜欢的主题 (如 Atom One Dark)。
    - `Settings/Preferences` > `Editor` > `Font`: 设置合适的字体 (如 JetBrains Mono) 和大小 (如 16-18)。
  - **版本控制**:
    - `Settings/Preferences` > `Version Control` > `Commit`: 勾选 `Use non-modal commit interface` (使用非模态提交界面)。

### Visual Studio Code (编辑器)

轻量且强大的代码编辑器。

- **配置同步**: 你可以通过 GitHub Gist 或 VS Code 内置的 `Settings Sync` 功能来同步你的配置。我的配置分享在这里： `kirklin/vscode-settings`。
  - 地址: [https://github.com/kirklin/vscode-settings](https://github.com/kirklin/vscode-settings)
- **推荐核心扩展**:
  - ESLint
  - Prettier - Code formatter
  - GitLens — Git supercharged
  - Docker
  - GitHub Copilot
  - Material Icon Theme (或其他图标主题)
  - _根据你的开发语言和框架安装相应扩展_

### Git (版本控制)

配置全局用户信息和一些有用的默认行为。

- **配置用户信息**:
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "your_email@example.com"
  ```
- **配置默认推送行为**:

  ```bash
  # 推送当前分支到同名远程分支
  git config --global push.default current
  # 如果远程分支不存在，自动创建
  git config --global push.autoSetupRemote true
  ```

  - 这样配置后，大多数情况下可以直接使用 `git push` 代替 `git push origin <branch-name>`。

- **配置全局忽略文件 (`~/.gitignore_global`)**:

  - 创建或编辑 `~/.gitignore_global` 文件，添加不需要纳入版本控制的全局文件模式，例如：

  ```gitignore
  # macOS specific
  .DS_Store
  *~
  ._*
  .Spotlight-V100
  .Trashes

  # IDE / Editor specific (根据你的常用工具添加)
  .idea/ # JetBrains IDEs
  .vscode/ # VS Code (如果不在项目 .gitignore 中)
  *.swp # Vim swap files

  # Build artifacts / Dependencies
  node_modules/
  npm-debug.log*
  yarn-debug.log*
  yarn-error.log*
  ```

  - 告诉 Git 使用这个全局忽略文件:
    ```bash
    git config --global core.excludesfile ~/.gitignore_global
    ```

### NVM 与 Node.js

Node Version Manager (NVM) 用于管理多个 Node.js 版本。

- **安装 NVM**: 已通过 `brew install nvm` 完成。确保 `.zshrc` 中已正确加载 NVM。
- **安装 Node.js (推荐 LTS 版本)**:
  - 查看当前 LTS 版本代号：`nvm ls-remote --lts`
  - 安装最新的 LTS 版本 (截至 2025 年初，LTS 是 v22 "Jod")：
    ```bash
    nvm install --lts
    # 或者指定版本号，例如安装 v22
    # nvm install 22
    ```
  - 设置默认 Node 版本:
    ```bash
    nvm alias default lts/* # 使用最新的 LTS 作为默认版本
    # 或者 nvm alias default 22
    ```
  - 验证安装:
    ```bash
    node -v
    npm -v
    ```

### Bob (翻译与 OCR)

macOS 上的翻译和 OCR 工具。

- **安装**: 已通过 `brew install --cask bob` 完成。
- **配置**:
  - 设置快捷键 (你的设置为 `F19+A` 划词翻译, `F19+S` 截图翻译)。
  - **安装插件**:
    - [bob-plugin-deeplx](https://github.com/clubxdev/bob-plugin-deeplx) (需要自行配置 DeepLX API)
    - [bobplugin-google-translate](https://github.com/roojay520/bobplugin-google-translate)
    - _根据需要查找并安装其他 Bob 插件。_
  - **配置服务**: 在 Bob 设置中添加和配置翻译服务 (如 DeepLX, Google Translate, 有道, 阿里等) 和 OCR 服务 (如腾讯 OCR)。

### Obsidian (笔记软件)

强大的知识管理和笔记软件。

- **配置同步**:
  - **Obsidian Sync**: 官方同步服务，可以同步笔记、设置和主题，但插件需要手动安装或通过其他方式同步。
  - **Git**: 可以将整个 Obsidian Vault (仓库) 作为 Git 仓库进行版本控制和同步。
    - **重要**: 在使用 Git 同步时，务必将 `.obsidian/workspace.json` 和 `.obsidian/workspaces/` 目录下的文件添加到 `.gitignore` 中，以避免因窗口布局等本地状态导致的冲突。
    ```gitignore
    # .gitignore for Obsidian Vault
    .obsidian/workspace.json
    .obsidian/workspaces/
    ```
  - **结合使用**: 可以使用 Git 管理整个 Vault 的版本和插件，同时使用 Obsidian Sync 同步笔记内容，实现两全其美。

### SourceTree (Git GUI)

免费的 Git 图形化客户端。

- **配置**:
  - 打开 SourceTree > Preferences (设置) > Git Tab。
  - **重要**: 选择 `Use System Git`，而不是 Embedded Git，以确保 SourceTree 使用你通过 Homebrew 安装和配置的 Git 版本，避免潜在的路径或版本冲突问题。

## 6. 系统设置优化

调整 macOS 系统设置以符合个人习惯和效率需求。

- **系统设置 (System Settings)**:
  - **通用 (General)**:
    - `默认网页浏览器 (Default Web Browser)`: 设置为 Chrome 或 Edge 等你偏好的浏览器。
    - `语言与地区 (Language & Region)`: 将 `每周第一天 (First day of week)` 改为 `星期一 (Monday)`。
  - **桌面与程序坞 (Desktop & Dock)**:
    - `程序坞 (Dock)`: 根据喜好调整大小、位置等。
    - `退出应用程序时关闭窗口 (Close windows when quitting an application)`: 默认是勾选的，取消勾选后，下次打开应用会恢复上次关闭时的窗口状态，对于某些应用（如终端、IDE）可能更方便。按需设置。
    - `调度中心 (Mission Control)`:
      - 取消勾选 `根据最近使用情况自动重新排列空间 (Automatically rearrange Spaces based on most recent use)`，保持多桌面顺序固定。
      - `触发角 (Hot Corners)`: 如果你使用 Raycast 或其他窗口管理工具，可以考虑禁用所有触发角，避免误触。
  - **显示器 (Displays)**:
    - `夜览 (Night Shift)`: 设置时间表或手动开启，减少蓝光。
    - 调整分辨率和亮度。
  - **通知 (Notifications)**:
    - 关闭不必要应用的通知权限，减少干扰。
  - **聚焦 (Spotlight)**:
    - 在 `Siri & Spotlight` > `Spotlight Privacy` 中可以添加不想被索引的文件夹。
    - 在搜索结果类别中，可以取消勾选不常用的类别，提高搜索精度和速度。
  - **隐私与安全性 (Security & Privacy)**:
    - `防火墙 (Firewall)`: 建议开启。
    - 检查各应用的权限授予情况 (位置、相机、麦克风等)。
  - **触控板 (Trackpad)**:
    - 自定义其他手势。
  - **键盘 (Keyboard)**:
    - `按键重复速度 (Key Repeat)`: 调到最快 (`Fast`)。
    - `重复前延迟 (Delay Until Repeat)`: 调到最短 (`Short`)。 _需要适应，但适应后能极大提高光标移动效率。_
    - `文本输入 (Text Input)` > `编辑 (Edit...)`:
      - 关闭 `自动纠正拼写 (Correct spelling automatically)`。
      - 关闭 `自动大写单词开头字母 (Capitalize words automatically)`。
      - 确保 `使用“ ”进行双引号输入 (Use smart quotes and dashes)` 中的引号设置符合你的编程习惯 (通常建议使用直引号，即取消勾选或设置为直引号 `"`)。
    - `输入法 (Input Sources)`: 添加并配置你喜欢的输入法 (如已安装的搜狗输入法)，移除不需要的默认输入法。
  - **触控 ID 与密码 (Touch ID & Password)**:
    - 设置登录密码。
    - 添加指纹。
    - `将 Apple Watch 用于解锁 App 和 Mac (Use your Apple Watch to unlock your apps and Mac)`: 如果你有 Apple Watch，强烈建议开启。
  - **共享 (Sharing)**:
    - `电脑名称 (Computer Name)`: 设置一个简洁易记的名称。
    - 按需开启/关闭 `屏幕共享 (Screen Sharing)`, `文件共享 (File Sharing)` 等，默认建议关闭不必要的共享服务。 `隔空播放接收器 (AirPlay Receiver)` 可以保持开启。
  - **Siri 与聚焦 (Siri & Spotlight)**:
    - 如果你不使用 Siri，可以禁用 `用“嘿 Siri”唤醒 (Listen for "Hey Siri")` 和 `按下侧边按钮使用 Siri (Press Side button for Siri)`。

---

配置完成！现在你的 MacBook Pro 应该已经具备了强大的生产力环境。记得定期使用 `brew update && brew upgrade` 更新你的软件。享受你的新 Mac 吧！

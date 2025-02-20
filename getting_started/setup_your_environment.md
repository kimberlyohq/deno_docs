## 配置你的环境 {#set-up-your-environment}

你需要正确地配置环境以高效地使用 Deno 进行开发。配置内容将会涵盖 shell 自动补全，环境变量以及你的 IDE 配置。

### 环境变量 {#environmental-variables}

以下这些环境变量会影响 Deno 的各种行为。

`DENO_DIR` 的默认值为 `$HOME/.cache/deno`，但是你可以选择其他任何路径来读写生成的源码。

如果设置了 `NO_COLOR` 变量，Deno 的输出将不会带有颜色。详情请参照 https://no-color.org/。你不需要使用 `--allow-env` 来验证是否设置了 `NO_COLOR`，只需要检查常量 `Deno.noColor` 的值，其类型为 Boolean。

### Shell 自动补全 {#shell-autocomplete}

你可以通过 `deno completions <shell>` 命令来为你的 Shell 生成自动补全脚本并输出到控制台。你仍需要将控制台的输出保存在相关的文件中以使其生效。

该功能支持以下 shell 环境：

- zsh
- bash
- fish
- powershell
- elvish

示例（bash）：

```shell
deno completions bash > /usr/local/etc/bash_completion.d/deno.bash
source /usr/local/etc/bash_completion.d/deno.bash
```

示例（不带任何框架的 zsh）：

```shell
mkdir ~/.zsh # 为你的补全创建一个文件夹。该文件夹可以位于任何位置。
deno completions zsh > ~/.zsh/_deno
```

再将此路径添加到 `.zshrc`

```shell
fpath=(~/.zsh $fpath)
autoload -Uz compinit
compinit -u
```

最后重启终端。请注意，如果自动补全未能成功载入，你可能需要先运行 `rm ~/.zcompdump/` 来删除之前生成的 compinit 文件，再运行 `compinit` 来重新生成。

示例（zsh + oh-my-zsh) [推荐 zsh 用户采用这个方案] :

```shell
mkdir ~/.oh-my-zsh/custom/plugins/deno
deno completions zsh > ~/.oh-my-zsh/custom/plugins/deno/_deno
```

随后，将 deno 插件添加到 `~/.zshrc` 文件的 "plugins" 标签下。对于 `antigen` 之类的工具来说，你的 path 将会是 `~/.antigen/bundles/robbyrussell/oh-my-zsh/plugins`，而你的指令则是 `antigen bundle deno`，以此类推。

示例（Powershell）：

```shell
deno completions powershell >> $profile
.$profile
```

以上命令默认会在 `$HOME\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1` 创建一个 Powershell 的配置文件，并且会在你每次运行 Powershell 时执行。

### 编辑器及 IDE {#editors-and-ides}

由于 Deno 的模块引入机制依赖文件拓展名，且允许通过 http 引入。而目前绝大多数编辑器和语言服务器并未对此提供原生支持，因此很多编辑器在现阶段会因为无法找到文件而报错，或者会在引入时加入不必要的文件拓展名。

为了应对这种情况，社区提供了以下编辑器插件：

#### VS Code {#vs-code}

[vscode_deno](https://github.com/denoland/vscode_deno) 的测试版已在 [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=denoland.vscode-deno)发布，欢迎反馈任何使用中遇到的问题。

#### JetBrains IDEs {#jetbrains-ides}

JetBrains 的 IDEs 的支持由
[the Deno plugin](https://plugins.jetbrains.com/plugin/14382-deno)提供。

安装之后，将 
`External Libraries > Deno Library > lib > lib.deno.d.ts` 替换为 
`deno types` 命令的输出。这会为当前版本提供自动补全和代码提示。当你每次升级 Deno 版本后，需要再次执行以上操作。此插件的详细使用说明已发布在 YouTrack 的 [评论](https://youtrack.jetbrains.com/issue/WEB-41607#focus=streamItem-27-4160152.0-0) 中。

#### Vim and NeoVim {#vim-and-neovim}

通过安装 [CoC](https://github.com/neoclide/coc.nvim)（intellisense 智能化引擎和语言服务器协议）或者 [ALE](https://github.com/dense-analysis/ale)（语法检查器和
语言服务器协议客户端）便可对 Deno/TypeScript 提供很好的支持。

##### CoC {#coc}

在安装 CoC 以后，请在 Vim 内运行 `:CocInstall coc-tsserver` 和 `:CocInstall coc-deno`。在你的项目中运行 `:CocCommand deno.initializeWorkspace` 以初始化工作空间配置。到此为止，`gd` （跳转到定义） 和 `gr` （跳转到引用）等功能应该已生效。

##### ALE {#ale}

ALE 原生支持 Deno 的 LSP（语言服务器协议），不许要额外配置。然而，如果你的 Deno 可执行文件不存在于 `$PATH`，或者拥有不同于 `deno` 的名字，亦或是你想尝试不稳定的特性与接口，你需要覆盖 ALE 的默认值。详情请见 [`:help ale-typescript`](https://github.com/dense-analysis/ale/blob/master/doc/ale-typescript.txt)。

ALE 提供了自动补全、重构、跳转定义、跳转引用等功能。不过，你需要手动配置按键绑定。你可以选择拷贝以下代码片段到你的 `vimrc`/`init.vim` 以获得最基本的支持，或者参考 [官方文档](https://github.com/dense-analysis/ale#table-of-contents)来对如何配置 ALE 获得更深的了解。

ALE 可以通过运行 `deno fmt` 来修复 linter 提出的问题。如果想要让 ALE 使用 Deno 的格式化工具，你需要配置 `ale_linter`。如果你想调整当前 buffer 配置，则使用（`let b:ale_linter = ['deno']`）。如果想为所有 TypeScript 文件提供全局配置，则需要设置成（`let g:ale_fixers={'typescript': ['deno']}`）。

```vim
" Use ALE autocompletion with Vim's 'omnifunc' setting (press <C-x><C-o> in insert mode)
autocmd FileType typescript set omnifunc=ale#completion#OmniFunc

" Make sure to use map instead of noremap when using a <Plug>(...) expression as the {rhs}
nmap gr <Plug>(ale_rename)
nmap gR <Plug>(ale_find_reference)
nmap gd <Plug>(ale_go_to_definition)
nmap gD <Plug>(ale_go_to_type_definition)

let g:ale_fixers = {'typescript': ['deno']}
let g:ale_fix_on_save = 1 " run deno fmt when saving a buffer
```

#### Emacs {#emacs}

通过组合使用 [tide](https://github.com/ananthakumaran/tide) 和 [typescript-deno-plugin](https://github.com/justjavac/typescript-deno-plugin)，Emacs 便可对使用 TypeScript 的 Deno 项目提供很好的支持。[tide](https://github.com/ananthakumaran/tide) 是在 Emacs 中使用 TypeScript 所需要的正规方式，而 [typescript-deno-plugin](https://github.com/justjavac/typescript-deno-plugin) 则是 [VSCode 官方 Deno 插件](https://github.com/denoland/vscode_deno)。

首先，确保 `tide` 已经在你的 Emacs 实例中正确配置。其次，根据 [typescript-deno-plugin](https://github.com/justjavac/typescript-deno-plugin) 的引导，在你的项目文件夹内运行 `npm install --save-dev typescript-deno-plugin typescript`（如果你仍未初始化你的项目文件夹，按需运行 `npm init -y`）。最后，在你的 `tsconfig.json` 文件内添加以下内容即可。

```jsonc
{
  "compilerOptions": {
    "plugins": [
      {
        "name": "typescript-deno-plugin",
        "enable": true, // 默认值为 `true`
        "importmap": "import_map.json"
      }
    ]
  }
}
```

#### LSP 客户端 {#lsp-clients}

Deno 在 1.6.0 以后的版本对[语言服务器协议](https://langserver.org)提供了内建支持。

如果你的编辑器支持 LSP，你可以将 Deno 用作你的 JavaScript 和 TypeScript 语言服务器。

编辑器可以通过 `deno lsp`命令来开启语言服务器。

##### 示例：Kakoune {#example-for-kakoune}

在安装 [`kak-lsp`](https://github.com/kak-lsp/kak-lsp) LSP 客户端以后，你可以你可以通过添加以下内容至 `kak-lsp.toml` 以添加 Deno 的语言服务器。

```toml
[language.deno]
filetypes = ["typescript", "javascript"]
roots = [".git"]
command = "deno"
args = ["lsp"]

[language.deno.initialization_options]
enable = true
lint = true
```

##### 示例：Vim/Neovim {#example-for-vimneovim}

在安装 [`vim-lsp`](https://github.com/prabirshrestha/vim-lsp) LSP 客户端以后，你可以你可以通过添加以下内容至 `vimrc`/`init.vim` 以添加 Deno 的语言服务器。

```vim
if executable("deno")
  augroup LspTypeScript
    autocmd!
    autocmd User lsp_setup call lsp#register_server({
    \ "name": "deno lsp",
    \ "cmd": {server_info -> ["deno", "lsp"]},
    \ "root_uri": {server_info->lsp#utils#path_to_uri(lsp#utils#find_nearest_parent_file_directory(lsp#utils#get_buffer_path(), "tsconfig.json"))},
    \ "allowlist": ["typescript", "typescript.tsx"],
    \ "initialization_options": {
    \     "enable": v:true,
    \     "lint": v:true,
    \     "unstable": v:true,
    \   },
    \ })
  augroup END
endif
```

##### 示例：Sublime Text {#example-for-sublime-text}

- 安装 [Sublime LSP 包](https://packagecontrol.io/packages/LSP)
- 安装
  [TypeScript 包](https://packagecontrol.io/packages/TypeScript) 以获得语法高亮。
- 在你的项目文件下添加 `.sublime-project` 文件，内容如下：

```jsonc
{
  "settings": {
    "LSP": {
      "deno": {
        "command": [
          "deno",
          "lsp"
        ],
        "initializationOptions": {
          // "config": "", // 添加你项目配置文件的路径
          "enable": true,
          // "importMap": "", // 添加你项目导入图的路径
          "lint": true,
          "unstable": false
        },
        "enabled": true,
        "languages": [
          {
            "languageId": "javascript",
            "scopes": ["source.js"],
            "syntaxes": [
              "Packages/Babel/JavaScript (Babel).sublime-syntax",
              "Packages/JavaScript/JavaScript.sublime-syntax"
            ]
          },
          {
            "languageId": "javascriptreact",
            "scopes": ["source.jsx"],
            "syntaxes": [
              "Packages/Babel/JavaScript (Babel).sublime-syntax",
              "Packages/JavaScript/JavaScript.sublime-syntax"
            ]
          },
          {
            "languageId": "typescript",
            "scopes": ["source.ts"],
            "syntaxes": [
              "Packages/TypeScript-TmLanguage/TypeScript.tmLanguage",
              "Packages/TypeScript Syntax/TypeScript.tmLanguage"
            ]
          },
          {
            "languageId": "typescriptreact",
            "scopes": ["source.tsx"],
            "syntaxes": [
              "Packages/TypeScript-TmLanguage/TypeScriptReact.tmLanguage",
              "Packages/TypeScript Syntax/TypeScriptReact.tmLanguage"
            ]
          }
        ]
      }
    }
  }
}
```

如果你未能在以上列表内找到你最爱的 IDE，或许你可以亲自开发扩展程序。我们的[社区 Discord 小组](https://discord.gg/deno)可以给你提供上手指导。

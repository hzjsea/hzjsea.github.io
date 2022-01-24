---
title: neovim插件
categories:
  - linux
tags:
  - linux
abbrlink: 729fdf51
date: 2020-03-26 18:03:16
---

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [neovim插件](#neovim插件)
  - [TabNine](#tabnine)
  - [rust插件](#rust插件)

<!-- /code_chunk_output -->
<!-- more -->

# neovim插件

## TabNine
这个太好用了 就是吃内存，奈何自己mac内存少，不能用
```bash
# 安装tabnine
:CocInstall coc-tabnine
# 配置文件
vi TabNine/tabnine.openConfig 
"ignore_all_lsp": true
```
## rust插件
https://github.com/rust-lang/rust.vim
```bash
Plug 'rust-lang/rust.vim'
:PlugInstall
```
---
title: vim安装
date: 2021-03-22 11:28:28
categories:  [vim]
tags: [vim]
top: 2
---


<!--more-->


# vim安装
neovim初始化记录，方便创建开发环境，兼容rust

> 如果有错误，看后面的修复



## neovim(vim)学习

github-vim doc learn : https://github.com/wsdjeg/vim-galore-zh_cn
慕课网 vim learn : https://www.imooc.com/learn/1129

## neovim安装

> 基础环境安装

- ubuntu 18.04 环境安装
```bash
sudo apt install -y npm libncurses5-dev tree libgnome2-dev libgnomeui-dev libgtk2.0-dev libatk1.0-dev libbonoboui2-dev libcairo2-dev libx11-dev libxpm-dev libxt-dev python-dev python3-dev ruby-dev lua5.1 liblua5.1-dev libperl-dev git build-essential cmake clang python3-pip ycmd python3-hamcrest gperf luajit luarocks libuv1-dev libluajit-5.1-dev libunibilium-dev libmsgpack-dev libtermkey-dev libvterm-dev libutf8proc-dev
```

- centos 7 环境安装
```bash
yum -y install npm git ncurses-devel ruby ruby-devel lua  perl perl-devel python3 python3-devel python2-devel perl-ExtUtils-Embed lrzsz cmake wget gcc gcc-c++ unzip
```

> nodejs 环境安装

```bash
bash "#!/usr/bin/bash

curl -sL install-node.now.sh/lts | bash

set -o nounset    # error when referencing undefined variable
set -o errexit    # exit when command fails

# Install latest nodejs
if [ ! -x "$(command -v node)" ]; then
    curl --fail -LSs https://install-node.now.sh/latest | sh
    export PATH="/usr/local/bin/:$PATH"
    # Or use apt-get
    # sudo apt-get install nodejs
fi

# Use package feature to install coc.nvim

# for vim8
mkdir -p ~/.vim/pack/coc/start
cd ~/.vim/pack/coc/start
curl --fail -L https://github.com/neoclide/coc.nvim/archive/release.tar.gz | tar xzfv -
# for neovim
# mkdir -p ~/.local/share/nvim/site/pack/coc/start
# cd ~/.local/share/nvim/site/pack/coc/start
# curl --fail -L https://github.com/neoclide/coc.nvim/archive/release.tar.gz | tar xzfv -

# Install extensions
mkdir -p ~/.config/coc/extensions
cd ~/.config/coc/extensions
if [ ! -f package.json ]
then
  echo '{"dependencies":{}}'> package.json
fi
# Change extension names to the extensions you need

npm install coc-snippets --global-style --ignore-scripts --no-bin-links --no-package-lock --only=prod"
```

> python相关包下载

```bash
pip3 install pylint pynvim jedi
```

> 编译安装neovim

```bash
git clone https://github.com/neovim/neovim/
cd neovim/
make CMAKE_INSTALL_PREFIX=/full/path/
make install
cd /usr/bin/
ln -s /full/path/bin/nvim nvim
```

> clone 配置

```bash
cd ~/.config/
git clone https://gitee.com/muaimingjun/nvim
```

> import ！！ 

```bash
使用
直接打开nvim会自动安装插件 注意:
如果打开(python)报错误 Error detected while processing function 95_filetype_changed: line 4: E492: Not an editor command: Semshi enable
在下面输入:UpdateRemotePlugins
```


> 插件列表


```bash
lug 'RRethy/vim-illuminate'  " 显示光标下当前单词的其他用法
Plug 'AndrewRadev/splitjoin.vim' " 在单行和多行代码形式之间切换
Plug 'vim-airline/vim-airline'   "状态栏插件
Plug 'vim-airline/vim-airline-themes' "状态栏插件
Plug 'Yggdroot/indentLine' "缩进线
Plug 'preservim/nerdtree'     "文件树
Plug 'connorholyday/vim-snazzy' " 配色方案
Plug 'mhinz/vim-startify'  " 可以显示历史文件
Plug 'kien/ctrlp.vim'  " 文件模糊搜索器
Plug 'easymotion/vim-easymotion' " 使用ss 查找两个字母并跳转   查找单词并跳转 快速定位，文件位置任我行 

" Git
Plug 'junegunn/gv.vim'  " 查看提交记录
Plug 'tpope/vim-fugitive'  " git插件
Plug 'theniceboy/vim-gitignore', { 'for': ['gitignore', 'vim-plug'] }
Plug 'fszymanski/fzf-gitignore', { 'do': ':UpdateRemotePlugins' }
"Plug 'mhinz/vim-signify'
Plug 'airblade/vim-gitgutter'  " vim显示文件变动

" Tex(一个现代的vim插件，用于编辑LaTeX文件。)
Plug 'lervag/vimtex'


" Auto Complete(自动补全)
Plug 'neoclide/coc.nvim', {'branch': 'release'}
Plug 'wellle/tmux-complete.vim'

" Undo Tree (文件树)
Plug 'mbbill/undotree'

" Markdown
Plug 'iamcco/markdown-preview.nvim', { 'do': { -> mkdp#util#install_sync() }, 'for' :['markdown', 'vim-plug'] }
Plug 'dhruvasagar/vim-table-mode', { 'on': 'TableModeToggle' }
Plug 'mzlogin/vim-markdown-toc', { 'for': ['gitignore', 'markdown'] }
Plug 'theniceboy/bullets.vim'

" HTML, CSS, JavaScript, PHP, JSON, etc.
Plug 'elzr/vim-json'
Plug 'hail2u/vim-css3-syntax', { 'for': ['vim-plug', 'php', 'html', 'javascript', 'css', 'less'] }
Plug 'spf13/PIV', { 'for' :['php', 'vim-plug'] }
Plug 'pangloss/vim-javascript', { 'for': ['vim-plug', 'php', 'html', 'javascript', 'css', 'less'] }
Plug 'yuezk/vim-js', { 'for': ['vim-plug', 'php', 'html', 'javascript', 'css', 'less'] }
Plug 'MaxMEllon/vim-jsx-pretty', { 'for': ['vim-plug', 'php', 'html', 'javascript', 'css', 'less'] }
Plug 'jelera/vim-javascript-syntax', { 'for': ['vim-plug', 'php', 'html', 'javascript', 'css', 'less'] }
"Plug 'jaxbot/browserlink.vim'

" Python
Plug 'tmhedberg/SimpylFold', { 'for' :['python', 'vim-plug'] }
Plug 'Vimjas/vim-python-pep8-indent', { 'for' :['python', 'vim-plug'] }
Plug 'numirias/semshi', { 'do': ':UpdateRemotePlugins', 'for' :['python', 'vim-plug'] }
"Plug 'vim-scripts/indentpython.vim', { 'for' :['python', 'vim-plug'] }
"Plug 'plytophogy/vim-virtualenv', { 'for' :['python', 'vim-plug'] }
Plug 'tweekmonster/braceless.vim'
Plug 'vim-scripts/indentpython.vim' " python缩进脚本

" Debugger
Plug 'puremourning/vimspector', {'do': './install_gadget.py --enable-c --enable-python '}
```


> 插件配置

```bash
" ===
" === vim-snazzy
" ===
let g:SnazzyTransparent = 1 " 背景不透明设置
colorscheme snazzy

" ===
" === nerdtree
" ===
autocmd vimenter * NERDTree
nnoremap <leader>v :NERDTreeFind<cr>
nnoremap <leader>g :NERDTreeToggle<cr>
let NERDTreeShowHidden=1
let NERDTreeIgonre=[
            \'\.git$', '\.hg$', '\.svn$', '\.stversions$', '\.pyc$', '\.pyo$', '\.swp$',
            \'.DS_Store$', '\.sass-cache$', '__pycache__$', '\.egg-info$','\.ropeproject$',
          \]

" ===
" === ctrlp
" ===
let g:ctrlp_map = '<c-p>'


" ===
" === vim-easymotion
" ===
nmap ss <Plug>(easymotion-s2)


" ==
" == GitGutter  (显示文件变动)
" ==
let g:gitgutter_signs = 0
let g:gitgutter_map_keys = 0
let g:gitgutter_override_sign_column_highlight = 0
let g:gitgutter_preview_win_floating = 1
autocmd BufWritePost * GitGutter
nnoremap <LEADER>gf :GitGutterFold<CR>
nnoremap H :GitGutterPreviewHunk<CR>
nnoremap <LEADER>g- :GitGutterPrevHunk<CR>
nnoremap <LEADER>g= :GitGutterNextHunk<CR>

" ===
" === vim-illuminate
" ===
let g:Illuminate_delay = 250
hi illuminatedWord cterm=undercurl gui=undercurl


" ===
" === coc
" ===
" fix the most annoying bug that coc has
"silent! au BufEnter,BufRead,BufNewFile * silent! unmap if
let g:coc_global_extensions = ['coc-python', 'coc-vimlsp', 'coc-html', 'coc-json', 'coc-css', 'coc-tsserver', 'coc-yank', 'coc-gitignore', 'coc-vimlsp', 'coc-tailwindcss', 'coc-stylelint', 'coc-tslint', 'coc-lists', 'coc-git', 'coc-explorer', 'coc-pyright', 'coc-sourcekit', 'coc-translator', 'coc-flutter', 'coc-todolist', 'coc-yaml', 'coc-tasks', 'coc-actions', 'coc-diagnostic']
"set statusline^=%{coc#status()}%{get(b:,'coc_current_function','')}
"nmap <silent> <TAB> <Plug>(coc-range-select)
"xmap <silent> <TAB> <Plug>(coc-range-select)
" use <tab> for trigger completion and navigate to the next complete item
function! s:check_back_space() abort
    let col = col('.') - 1
    return !col || getline('.')[col - 1]    =~ '\s'
endfunction
inoremap <silent><expr> <TAB>
    \ pumvisible() ? "\<C-n>" :
    \ <SID>check_back_space() ? "\<TAB>" :
    \ coc#refresh()
inoremap <expr><S-TAB> pumvisible() ? "\<C-p>" : "\<C-h>"
inoremap <expr> <cr> complete_info()["selected"] != "-1" ? "\<C-y>" : "\<C-g>u\<CR>"
function! s:check_back_space() abort
    let col = col('.') - 1
    return !col || getline('.')[col - 1]  =~# '\s'
endfunction
inoremap <silent><expr> <c-space> coc#refresh()
inoremap <silent><expr> <c-o> coc#refresh()

" Open up coc-commands
nnoremap <c-c> :CocCommand<CR>
" Text Objects
xmap kf <Plug>(coc-funcobj-i)
xmap af <Plug>(coc-funcobj-a)
omap kf <Plug>(coc-funcobj-i)
omap af <Plug>(coc-funcobj-a)
" Useful commands
nnoremap <silent> <space>y :<C-u>CocList -A --normal yank<cr>
nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gi <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)
nmap <leader>rn <Plug>(coc-rename)
nmap tt :CocCommand explorer<CR>
" coc-translator
nmap ts <Plug>(coc-translator-p)
" Remap for do codeAction of selected region
function! s:cocActionsOpenFromSelected(type) abort
  execute 'CocCommand actions.open ' . a:type
endfunction
xmap <silent> <leader>a :<C-u>execute 'CocCommand actions.open ' . visualmode()<CR>
nmap <silent> <leader>a :<C-u>set operatorfunc=<SID>cocActionsOpenFromSelected<CR>g@
" coctodolist
nnoremap <leader>tn :CocCommand todolist.create<CR>
nnoremap <leader>tl :CocList todolist<CR>
nnoremap <leader>tu :CocCommand todolist.download<CR>:CocCommand todolist.upload<CR>
" coc-tasks
noremap <silent> T :CocList tasks<CR>

" ===
" === vim-markdown-toc
" ===
"let g:vmt_auto_update_on_save = 0
"let g:vmt_dont_insert_fence = 1
let g:vmt_cycle_list_item_markers = 1
let g:vmt_fence_text = 'TOC'
let g:vmt_fence_closing_text = '/TOC'
" ===
" === MarkdownPreview
" ===
let g:mkdp_auto_start = 0
let g:mkdp_auto_close = 1
let g:mkdp_refresh_slow = 0
let g:mkdp_command_for_global = 0
let g:mkdp_open_to_the_world = 0
let g:mkdp_open_ip = ''
let g:mkdp_echo_preview_url = 0
let g:mkdp_browserfunc = ''
let g:mkdp_preview_options = {
            \ 'mkit': {},
            \ 'katex': {},
            \ 'uml': {},
            \ 'maid': {},
            \ 'disable_sync_scroll': 0,
            \ 'sync_scroll_type': 'middle',
            \ 'hide_yaml_meta': 1
            \ }
let g:mkdp_markdown_css = ''
let g:mkdp_highlight_css = ''
let g:mkdp_port = ''
let g:mkdp_page_title = '「${name}」'


" ===
" === Markdown Settings
" ===
" Snippets
""source $XDG_CONFIG_HOME/nvim/md-snippets.vim
source ./md-snippets.vim
" auto spell
autocmd BufRead,BufNewFile *.md setlocal spell


" ===
" === vimspector
" ===
let g:vimspector_enable_mappings = 'HUMAN'
function! s:read_template_into_buffer(template)
    " has to be a function to avoid the extra space fzf#run insers otherwise
    execute '0r ~/.config/nvim/sample_vimspector_json/'.a:template
endfunction
command! -bang -nargs=* LoadVimSpectorJsonTemplate call fzf#run({
            \   'source': 'ls -1 ~/.config/nvim/sample_vimspector_json',
            \   'down': 20,
            \   'sink': function('<sid>read_template_into_buffer')
            \ })
noremap <leader>vs :tabe .vimspector.json<CR>:LoadVimSpectorJsonTemplate<CR>
sign define vimspectorBP text=☛ texthl=Normal
sign define vimspectorBPDisabled text=☞ texthl=Normal
""sign define vimspectorPC text=ð¶ texthl=SpellBad


" ===
" === vimtex
" ===
"let g:vimtex_view_method = ''
let g:vimtex_view_general_viewer = 'llpp'
let g:vimtex_mappings_enabled = 0
let g:vimtex_text_obj_enabled = 0
let g:vimtex_motion_enabled = 0
let maplocalleader=' '

" ===
" === Undotree
" ===
noremap L :UndotreeToggle<CR>
let g:undotree_DiffAutoOpen = 1
let g:undotree_SetFocusWhenToggle = 1
let g:undotree_ShortIndicators = 1
let g:undotree_WindowLayout = 2
let g:undotree_DiffpanelHeight = 8
let g:undotree_SplitWidth = 24
function g:Undotree_CustomMap()
	nmap <buffer> u <plug>UndotreeNextState
	nmap <buffer> e <plug>UndotreePreviousState
	nmap <buffer> U 5<plug>UndotreeNextState
	nmap <buffer> E 5<plug>UndotreePreviousState
endfunc

" ===
" === vim-fugitive
" ===
nnoremap gb :Gblame<CR>
```

> 安装rust环境

```bash
cd ~
curl https://sh.rustup.rs -sSf | sh
echo export PATH="$HOME/.cargo/bin:$PATH" > .profile
source .profile
```

> 安装coc-rust-analyzer

安装完成rust之后，创建一个项目，进入之后会自动配置rust-analyzer
```bash
cargo new helloworld
cd helloworld && vim src/main.rs 
```


> vim安装8.0+ 如果没有报错 不需要使用

```bash
git clone https://github.com/vim/vim.git
cd vim
./configure  --enable-pythoninterp=yes --with-python-config-dir=/usr/lib/python2.7/config
make && make install
" 安装libncurses5-dev "
sudo apt-get install libncurses5-dev
alias vim='/usr/local/bin/vim' # 注意如果切换到zsh 需要重新alias一下

```

> 安装fzf  注意，不需要可以不安装

```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
cd ~/.fzf/
./install
```

> 安装zsh 不需要可以不安装

```bash
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
# 以上命令可能不好使，可使用如下两条命令
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh
bash ./install.sh
```

> git 每次pull push 需要密码

解决办法：

git bash进入你的项目目录，输入：

git config --global credential.helper store

再操作一次git pull or git push ，然后它会提示你输入账号密码，这一次之后就不需要再次输入密码了

# not found syntax.vim file

重新安装vim
```bash
yum remove vim*

yum install vim

yum install vim*
```
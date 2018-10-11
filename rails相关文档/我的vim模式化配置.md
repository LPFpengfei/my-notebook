### 1. 下载插件（大神配置）

```shell
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

下载vim大神的模式化插件，你可以备份到自己的gitlab上，但是没法获得作者的更新。

### 2. 设置vim字体颜色

执行下面的代码

```shell
cp -rf ~/.vim/bundle/molokai/colors ~/.vim/
```

配置vim相关字体颜色

### 3.  vim配置文件

将下面的内容覆盖粘贴到自己的 ~/.vimrc文件中

```shell
vim ~/.vimrc
```

```ruby
set nocompatible " be iMproved

set guifont=Ubuntu\ Mono\ Nerd\ Font\ Complete\ 24

" 搜索逐字符高亮
set hlsearch
set path+=**
set wildmenu
" leader 键
let mapleader = ','
"配置backspace键工作方式
set backspace=indent,eol,start
"搜索时，关键字高亮
set hls
"设置在Vim中可以使用鼠标 防止在Linux终端下无法拷贝
set mouse=a
" 复制粘贴到系统剪贴板
map <C-C> "+y<CR>
map <C-V> "+p<CR>
" 窗口缩放
"set modifiable
"设置ctags的快捷键为 Ctrl-F12  
map <F10> :!ctags -R --c++-kinds=+p --fields=+iaS --extra=+q .<CR>    
"显示行号
set number
"设置在编辑过程中右下角显示光标的行列信息
set ruler
"当一行文字很长时取消换行
"set nowrap
"在状态栏显示正在输入的命令
set showcmd
"设置历史记录条数
set history=1000
"设置取消备份 禁止临时文件生成
set nobackup
set noswapfile
"突出现实当前行列
set cursorline
"set cursorcolumn
"设置匹配模式 类似当输入一个左括号时会匹配相应的那个右括号
set showmatch
"设置C/C++方式自动对齐
set autoindent
set cindent
"开启语法高亮功能
syntax enable
"设置搜索时忽略大小写
set ignorecase
"设置Tab宽度
set tabstop=2
"设置自动对齐空格数
set shiftwidth=2
"设置按退格键时可以一次删除4个空格
set softtabstop=4
set smarttab
"将Tab键自动转换成空格 真正需要Tab键时使用[Ctrl + V + Tab]
set expandtab
"设置编码方式
set encoding=utf-8
""自动判断编码时 依次尝试一下编码
set fileencodings=ucs-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1
"检测文件类型
filetype on
"针对不同的文件采用不同的缩进方式
filetype indent on
"允许插件
filetype plugin on
"启动智能补全
filetype plugin indent on

filetype off " required! /** 从这行开始，vimrc配置 **/

set rtp+=~/.vim/bundle/Vundle.vim

call vundle#begin()
Plugin 'VundleVim/Vundle.vim'

Bundle 'tpope/vim-fugitive'
Bundle 'Lokaltog/vim-easymotion'
Bundle 'tpope/vim-rails'
Bundle 'L9'

"NerdTree 
Bundle 'scrooloose/nerdtree'
let NERDTreeWinPos='left'
let NERDTreeShowBookmarks=1
let NERDTreeWinSize=31
let NERDTreeChDirMode=1
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTreeType") &&  b:NERDTreeType == "primary") | q | endif
nmap <Leader>n :NERDTreeFind<CR>
nmap <C-Y> :NERDTreeToggle<CR>


" nerdtree-git-plugin
Bundle 'Xuyuanp/nerdtree-git-plugin'
let g:NERDTreeIndicatorMapCustom = {
      \ "Modified"  : "✹",
      \ "Staged"    : "✚",
      \ "Untracked" : "✭",
      \ "Renamed"   : "➜",
      \ "Unmerged"  : "═",
      \ "Deleted"   : "✖",
      \ "Dirty"     : "✗",
      \ "Clean"     : "✔︎",
      \ "Unknown"   : "?"
      \ }

" TagBar
Bundle 'majutsushi/tagbar'
let g:tagbar_ctags_bin='ctags'          "ctags程序的路径
let g:tagbar_width=35                   "窗口宽度的设置
"let g:tagbar_left=1                     "居左
let g:tagbar_right=1                     "居左
let g:tagbar_type_ruby = {
      \ 'kinds' : [
      \ 'm:modules',
      \ 'c:classes',
      \ 'd:describes',
      \ 'C:contexts',
      \ 'f:methods',
      \ 'F:singleton methods'
      \ ]
      \ }
map <C-I> :Tagbar<CR>

" 增强状态栏
Plugin 'vim-airline/vim-airline'
Plugin 'vim-airline/vim-airline-themes'
set laststatus=2
let g:airline#extensions#tabline#enabled=1
let g:airline#extensions#whitespace#enabled=0
let g:airline_powerline_fonts = 1
let g:airline_theme='dark'

Plugin 'edkolev/tmuxline.vim'

let g:airline#extensions#tabline#left_sep = "\uE0C4"
"let g:airline#extensions#tabline#left_alt_sep = "\uE0BD"
let g:airline_left_sep = "\uE0C4"
"let g:airline_right_sep = "\uE0C5"

map <C-H> :bp<CR>
map <C-L> :bn<CR>
map <C-B> :ls<CR>

Plugin 'scrooloose/nerdcommenter'
" Add spaces after comment delimiters by default
let g:NERDSpaceDelims = 1
" Use compact syntax for prettified multi-line comments
let g:NERDCompactSexyComs = 1
" Align line-wise comment delimiters flush left instead of following code indentation
let g:NERDDefaultAlign = 'left'
" Allow commenting and inverting empty lines (useful when commenting a region)
let g:NERDCommentEmptyLines = 1
" Enable trimming of trailing whitespace when uncommenting
let g:NERDTrimTrailingWhitespace = 1

Plugin 'tpope/vim-endwise'

Plugin 'ctrlpvim/ctrlp.vim'
"<Leader>p搜索当前目录下文件
let g:ctrlp_map = '<Leader>p'
let g:ctrlp_cmd = 'CtrlP'
"<Leader>f搜索MRU文件
nmap <Leader>f :CtrlPMRUFiles<CR>
"<Leader>b显示缓冲区文件，并可通过序号进行跳转
nmap <Leader>b :CtrlPBuffer<CR>
"设置搜索时忽略的文件
let g:ctrlp_custom_ignore = {
      \ 'dir':  '\v[\/]\.(git|hg|svn|rvm|log|tmp)$',
      \ 'file': '\v\.(exe|so|dll|zip|tar|tar.gz|pyc|tags)$',
      \ }
let g:ctrlp_working_path_mode = 0
let g:ctrlp_match_window_bottom = 1
"修改QuickFix窗口显示的最大条目数
let g:ctrlp_max_height = 15
let g:ctrlp_match_window_reversed = 1
"设置MRU最大条目数为20
let g:ctrlp_mruf_max = 20
let g:ctrlp_follow_symlinks = 1
"默认使用全路径搜索，置1后按文件名搜索，准确率会有所提高，可以用<C-d>进行切换
let g:ctrlp_by_filename = 0
"默认不使用正则表达式，置1改为默认使用正则表达式，可以用<C-r>进行切换
let g:ctrlp_regexp = 0
"自定义搜索列表的提示符
let g:ctrlp_line_prefix = '♪ '

" 缩进线条
Plugin 'Yggdroot/indentLine'
"let g:indentLine_showFirstIndentLevel = 1
"let g:indentLine_setColors = 0

Plugin 'SuperTab'
let g:SuperTabRetainCompletionType=2

" Themes

Plugin 'tomasr/molokai'
colorscheme molokai
set t_Co=256
let g:molokai_original = 1
let g:rehash256 = 1

Plugin 'morhetz/gruvbox'
" colorscheme gruvbox
" set background=dark

Plugin 'Chiel92/vim-autoformat'
Plugin 'bbatsov/rubocop' " 用于格式化ruby
noremap <C-F> :Autoformat<CR>

" NerdFonts字体
Plugin 'ryanoasis/vim-devicons'
" NerdTree高亮
Plugin 'tiagofumo/vim-nerdtree-syntax-highlight'
" GitGutter
Plugin 'airblade/vim-gitgutter'
" 多行光标
Plugin 'terryma/vim-multiple-cursors'
" ack搜索
Plugin 'mileszs/ack.vim'
cnoreabbrev Ack Ack!
nnoremap <Leader>a :Ack!<Space>
set shellpipe=>
" Dash
Plugin 'rizzatti/dash.vim'
nnoremap <Leader>d :Dash<Space>

call vundle#end()
filetype plugin indent on " required! /** vimrc文件配置结束 **/
```
### 4. vim的搜索插件

```shell
brew list | grep tag 
brew install ctags ack -y
```
vim的搜索插件，执行上面的命令，下载安装搜索插件

### 5. 下载字体

到[https://nerdfonts.com/](https://nerdfonts.com/)下载自己喜欢的字体，我选择的是 [UbuntuMono](https://github.com/ryanoasis/nerd-fonts/releases/download/v2.0.0/UbuntuMono.zip)，你也可以选择自己喜欢的字体。
字体配置如下：

![image.png](https://upload-images.jianshu.io/upload_images/5901041-8de978cacf3f6f31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 好了，尽情的开始你的vim之旅吧！

配置好的vim界面如下，喜欢的可以试一试！

![image.png](https://upload-images.jianshu.io/upload_images/5901041-bc27baa51195138e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





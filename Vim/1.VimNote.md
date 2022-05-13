# Vim Note


## vimplus 下载安装

安装 vimplus：

```bash
$ git clone https://github.com/chxuan/vimplus.git ~/.vimplus
$ cd ~/.vimplus
$ ./install.sh
```


更新 vimplus：

```bash
$ ./update.sh
```

可通过 vimplus 的 `,h` 命令查看 vimplus [帮助文档](https://github.com/chxuan/vimplus/blob/master/help.md)

Ubuntu vimplus `.vimrc` 文件中有一个插件有问题，需要注释掉，插件名字如下：

```bash
Plug 'Shougo/echodoc.vim'
```

## 安装 ctags

重新安装 ctags，使用 [Universal CTags](https://github.com/universal-ctags/ctags) (默认的软件源都是Exuberant Ctags，版本太旧了)

```bash
$ sudo apt install autoconf
$ cd /tmp
$ git clone https://github.com/universal-ctags/ctags
$ cd ctags
$ ./autogen.sh
$ ./configure --prefix=PATH  # 安装路径,自己的情况调整。
$ make
$ sudo make install
```

### 生成tags文件：

```bash
# 递归的为当前目录及子目录下的所有代码文件生成tags文件 (推荐使用此命令)
$ ctags -R

#为当前目录某些源码生成tags文件
$ ctags filename.c filename1.c file.h

#为当前目录所有.c, .h源码生成tags文件
$ ctags *.c *.h
```

为了使得字段补全有效，在生成tags时需要一些额外的参数，推荐的c++参数主要是：`ctags -R --c++-kinds=+px --fields=+iaS --extra=+q`

其中：

- 选项c++-kinds 用于指定C++语言的 tags记录类型,  --c-kinds用于指定c语言的，  通用格式是  `--{language}-kinds`
- 选项 fileds 用于指定每条标记的扩展字段域
- extra 选项用于增加额外的条目:   `f`表示为每个文件增加一个条目，  `q`为每个类增加一个条目

注意：如果新定义或修改了函数、宏、变量，需要使用以上命令更新ctag以后才能查找到

### 配置

**第一种方法**，直接指定tags路径：

- 用vim打开某个工程文件，在命令行模式设置tags源，即 `: set tags=/home/d/code/tags`
- 或者
- 在 `~/.vimrc`配置文件中添加一行 `：set tags=/home/d/code/tags`
- 直接指定了tags文件的路径，vim直接从这个路径的tags文件查找函数或变量。不建议使用。

**第二种方法**，通用配置：

在 `~/.vimrc`文件中添加下面两行：

```bash
set tags=tags;
set autochdir
```

这两个命令的作用：vim首先在当前目录里寻找tags文件，如果没有找到tags文件，或者没有找到对应的目标，就到父目录中查找，一直向上递归。因为tags文件中记录的路径总是相对于tags文件所在的路径，所以要使用第二个设置项来改变vim的当前目录。需要注意的是，第一个命令里的分号是必不可少的。

建议使用第二种方法，可以避免一些路径冲突或不存在导致的问题。使用第二种方法，只在代码工作目录生成一个tags文件就可以了。如果逐层多个目录生成了tags文件，使用 `Ctrl + ]`  查找时会显示多个相同项目。

### 常用命令

| 操作                                | 描述                                       |
| ----------------------------------- | ------------------------------------------ |
| {% em %}**Ctrl + ]**{% endem %}     | 跳转到光标所在变量、宏、函数的定义处       |
| {% em %}**Ctrl + T**{% endem %}     | 返回到跳转前的位置                         |
| {% em %}**Ctrl + W + ]**{% endem %} | 分割当前窗口，并在新窗口中显示跳转到的定义 |
| {% em %}**Ctrl + O**{% endem %}     | 返回之前的位置                             |
| {% em %}**:ts**{% endem %}          | 列出所有匹配的标签                         |

### 使用中可能遇到的问题

1、ctags已经安装并且tags文件也已经生成，通过 `ctrl + ]` 查找某个函数、宏、结构体的定义的时候，报错：“E257: cstag: tag not found”，为什么？

可能原因有三个：

（1）查找的函数、宏、结构体可能是新定义的或者有修改，tags文件还没有相应的的数据，如果确定是这种情况，需要重新执行ctags -R命令，更新tags数据，就可以找到了。

（2）如果没有在 `~/.vimrc` 配置文件（或其他类似的配置文件）中添加 `set tags=tags;set autochdir` 这两个配置，只能从tags所在目录指定路径直接打开代码文件才管用。举个栗子：代码目录为： ~/code，某个代码文件的路径是：a/b/c.c，并且在目录~/code下执行ctags -R命令生成tags文件。这种情况下，在~/code目录使用命令：vim a/b/c.c打开文件，是可以使用Ctrl + ]找到函数定义的，但是，如果先进入目录~/code/a/b，再打开文件vim c.c，这样就会找不到tag。使用上文第二种配置方法，当前目录找不到tags时候会递归向上一级目录查找，可以避免这个问题。

（3）还有一种可能是tags配置路径不对应导致，如果指定了固定的tags路径，需要确保路径存在并且此路径下存在tags文件
 检查每个vim相关的配置文件（比如/etc/vim/vimrc和~/.vimrc等）中的set tags配置项，确保路径不冲突
 比如：~/.vimrc下有这样的配置：set tags=/data/nginx/tags，而自己的tags文件在 ~/code目录下，路径不一致，找不到就会报错：“E257: cstag: tag not found”，
 还有比如，存在两个配置：set tags=/home/test1/tags set tags=/home/test2/tags，而/home/test2没有tags，也会报错。
 总之，如果set tags配置了固定的路径，一定要与tags文件路径保持一致。如果每个人都是在不同主机开发或者是同一主机但不同用户下开发，不会有这种情况出现，但如果是多人共用一个开发环境使用同一用户，只是代码所在路径不同就可能发生，亲身经历过的。

2、使用 `Ctrl + ]` 查找某个标签时，第一次查找时列出了所有匹配项选择一个后，但在第二次查找时却直接跳转到之前选择过的项而不是列出所有的匹配项，或者使用 `Ctrl + ]` 查找标签时总是直接跳转到第一个匹配的标签，但这可能并不是你想要的，这时候使用 **`:ts`**命令就可以列出所有匹配项供自己选择，或者在配置文件中添加如下配置：`map <c-]> g<c-]>`。

##  安装 gtags

请首先安装最新版本 gtags，目前版本是 6.6.2，Linux 下请自行编译最新版（Debian / Ubuntu 自带的都太老了），Mac 下检查下 brew 安装的版本至少不要低于 6.6.0 ，否则请自己编译。

安装 [gtags](https://www.gnu.org/software/global/download.html) (系统软件源一般版本比较低，建议自己编译安装)

gtags 原生支持 6 种语言（C，C++，Java，PHP4，Yacc，汇编）， 通过安装 `pygments` 扩展支持 50+ 种语言（包括 go/rust/scala 等，基本覆盖所有主流语言）。

```bash
$ pip install pygments
# $ sudo apt install global	# 安装gtags

```

保证 `.vimrc` 里要设置过两个环境变量才能正常工作：

```bash
" vimrc 中设置环境变量启用 pygments
let $GTAGSLABEL = 'native-pygments'
let $GTAGSCONF = '/path/to/share/gtags/gtags.conf'
```

第一个 GTAGSLABEL 告诉 gtags 默认 C/C++/Java 等六种原生支持的代码直接使用 gtags 本地分析器，而其他语言使用 pygments 模块。

第二个环境变量必须设置，否则会找不到 native-pygments 和 language map 的定义，Linux 下要到 /usr/local/share/gtags 里找，也可以把它拷贝成 ~/.globalrc ，Vim 配置的时候方便点。

实际使用 pygments 时，gtags 会启动 python 运行名为 pygments_parser.py 的脚本，通过管道和它通信，完成源代码分析，故需保证 gtags 能在 $PATH 里调用 python，且这个 python 安装了 pygments 模块。

正确安装后，可以通过命令行 gtags 命令和 global 进行测试，注意shell 下设置环境变量。

## 安装三个插件

安装三个插件 : [vim-gutentags 索引自动管理](https://github.com/ludovicchabant/vim-gutentags) + [索引数据库切换](https://github.com/skywind3000/gutentags_plus) + [索引预览](https://github.com/skywind3000/vim-preview)

```bash
" 静态语法检查插件
Plug 'w0rp/ale'

" Vim自动生成 tags 插件 vim-gutentag
Plug 'ludovicchabant/vim-gutentags'
Plug 'skywind3000/gutentags_plus'
Plug 'skywind3000/vim-preview'
```

## 自动生成 Gtags

使用 vim-gutentags 插件。

```bash
Plug 'ludovicchabant/vim-gutentags'
```

`.vimrc` 里加入：

```bash
" gutentags 搜索工程目录的标志，当前文件路径向上递归直到碰到这些文件/目录名
let g:gutentags_project_root = ['.root', '.svn', '.git', '.hg', '.project']

" 所生成的数据文件的名称
let g:gutentags_ctags_tagfile = '.tags'

" 同时开启 ctags 和 gtags 支持：
let g:gutentags_modules = []
if executable('ctags')
	let g:gutentags_modules += ['ctags']
endif
if executable('gtags-cscope') && executable('gtags')
	let g:gutentags_modules += ['gtags_cscope']
endif

" 将自动生成的 tags 文件全部放入 ~/.cache/tags 目录中，避免污染工程目录
let s:vim_tags = expand('~/.cache/tags')
let g:gutentags_cache_dir = s:vim_tags

" 配置 ctags 的参数，老的 Exuberant-ctags 不能有 --extra=+q，注意
let g:gutentags_ctags_extra_args = ['--fields=+niazS', '--extra=+q']
let g:gutentags_ctags_extra_args += ['--c++-kinds=+px']
let g:gutentags_ctags_extra_args += ['--c-kinds=+px']

" 如果使用 universal ctags 需要增加下面一行，老的 Exuberant-ctags 不能加下一行
let g:gutentags_ctags_extra_args += ['--output-format=e-ctags']

" 禁用 gutentags 自动加载 gtags 数据库的行为
let g:gutentags_auto_add_gtags_cscope = 0

" 检测 ~/.cache/tags 不存在就新建
if !isdirectory(s:vim_tags)
   silent! call mkdir(s:vim_tags, 'p')
endif

" 预览 quickfix 窗口 ctrl-w z 关闭
" p 预览 大p关闭
autocmd FileType qf nnoremap <silent><buffer> p :PreviewQuickfix<cr>
autocmd FileType qf nnoremap <silent><buffer> P :PreviewClose<cr>
" 往上滚动预览窗口
noremap <Leader>u :PreviewScroll -1<cr>
" 往下滚动预览窗口
noremap <leader>d :PreviewScroll +1<cr>
```

## 基于 gutentags 实现跳转

在为当前目录生成tags文件后，可以通过按键 `Ctrl + ]` 跳转到对应的定义位置，再使用命令 `Ctrl + o` 回退到原来的位置。关于跳转的具体应用，可以参考 [Vim使用ctags实现函数跳转](https://vimjc.com/vim-ctag.html)

![](/images/vim-0.gif)

另外，建议多使用 `Ctrl + W + ]` 用新窗口打开并查看光标下符号的定义，或者 `Ctrl -W }` 使用 preview 窗口预览光标下符号的定义。

预设快捷键如下

| 快捷键      | 说明                    |
| ----------- | ----------------------- |
| `<leader>cg` | 查看光标下符号的定义 |
| `<leader>cs` | 查看光标下符号的引用       |
| `<leader>cc` | 查看有哪些函数调用了该函数        |
| `<leader>cf` | 查找光标下的文件            |
| `<leader>ci` | 查找哪些文件 include 了本文件            |

查找到索引后跳到弹出的 quikfix 窗口，停留在想查看索引行上，按 `小P` 直接打开预览窗口，`大P` 关闭预览。

## 快速预览

我们从新项目仓库里查询了一个符号的引用，gtags 噼里啪啦的给了你二十多个结果，那么多结果顺着一个个打开，查看，关闭，再打开很蛋疼，可使用 [vim-preview](https://link.zhihu.com/?target=https%3A//github.com/skywind3000/vim-preview) 插件高效的在 quickfix 中先快速预览所有结果，再有针对性的打开必要文件：

```bash
Plug 'skywind3000/vim-preview'
```

![](/images/vim-1.png)

## 快捷键

以下是部分常用快捷键，可通过 vimplus 的 `,h` 命令查看 [vimplus帮助文档](https://github.com/chxuan/vimplus/blob/master/help.md)。

| 快捷键              | 说明                                      |
| ------------------- | ----------------------------------------- |
| `<leader>n`         | 打开/关闭代码资源管理器                   |
| `<leader>t`         | 打开/关闭函数列表                         |
| `<leader>a`         | .h .cpp 文件切换                          |
| `<leader>u`         | 转到函数声明                              |
| `<leader>U`         | 转到函数实现                              |
| `<leader>u`         | 转到变量声明                              |
| `<leader>o`         | 打开include文件                           |
| `<leader>y`         | 拷贝函数声明                              |
| `<leader>p`         | 生成函数实现                              |
| `<leader>w`         | 单词跳转                                  |
| `<leader>f`         | 搜索~目录下的文件                         |
| `<leader>F`         | 搜索当前目录下的文本                      |
| `<leader>ff`        | 语法错误自动修复(FixIt)                   |
| `<c-p>`             | 切换到上一个buffer                        |
| `<c-n>`             | 切换到下一个buffer                        |
| `<leader>d`         | 删除当前buffer                            |
| `<leader>D`         | 删除当前buffer外的所有buffer              |
| `<F5>`              | 显示语法错误提示窗口                      |
| `<leader>l`         | 按竖线对齐                                |
| `<leader>=`         | 按等号对齐                                |
| `Ya`                | 复制行文本到字母a                         |
| `Da`                | 剪切行文本到字母a                         |
| `Ca`                | 改写行文本到字母a                         |
| `rr`                | 替换文本                                  |
| `<leader>r`         | 全局替换，目前只支持单个文件              |
| `rev`               | 翻转当前光标下的单词或使用V模式选择的文本 |
| `gcc`               | 注释代码                                  |
| `gcap`              | 注释段落                                  |
| `vif`               | 选中函数内容                              |
| `dif`               | 删除函数内容                              |
| `cif`               | 改写函数内容                              |
| `vaf`               | 选中函数内容（包括函数名 花括号）         |
| `daf`               | 删除函数内容（包括函数名 花括号）         |
| `caf`               | 改写函数内容（包括函数名 花括号）         |
| `fa`                | 查找字母a，然后再按f键查找下一个          |
| `<leader>h`         | 打开vimplus帮助文档                       |
| `<leader>H`         | 打开当前光标所在单词的vim帮助文档         |
| `<leader><leader>t` | 生成try-catch代码块                       |
| `<leader><leader>y` | 复制当前选中到系统剪切板                  |
| `<leader><leader>i` | 安装插件                                  |
| `<leader><leader>u` | 更新插件                                  |
| `<leader><leader>c` | 删除插件                                  |

### 缓存操作

| 快捷键          | 说明               |
| --------------- | ------------------ |
| `:e <filename>` | 新建buffer打开文件 |
| `:bp`           | 切换到上一个buffer |
| `:bn`           | 切换到下一个buffer |
| `:bd`           | 删除当前buffer     |

### 窗口操作

| 快捷键            | 说明                   |
| ----------------- | ---------------------- |
| `:sp <filename>`  | 横向切分窗口并打开文件 |
| `:vsp <filename>` | 竖向切分窗口并打开文件 |
| `<c-w>h`          | 跳到左边的窗口         |
| `<c-w>j`          | 跳到下边的窗口         |
| `<c-w>k`          | 跳到上边的窗口         |
| `<c-w>l`          | 跳到右边的窗口         |
| `<c-w>c`          | 关闭当前窗口           |
| `<c-w>o`          | 关闭其他窗口           |
| `:only`           | 关闭其他窗口           |

### 光标移动

| 快捷键  | 说明                                     |
| ------- | ---------------------------------------- |
| `0`     | 光标移动到行首                           |
| `^`     | 跳到从行首开始第一个非空白字符           |
| `$`     | 光标移动到行尾                           |
| `<c-o>` | 跳到上一个位置                           |
| `<c-i>` | 跳到下一个位置                           |
| `<c-b>` | 上一页                                   |
| `<c-f>` | 下一页                                   |
| `<c-u>` | 上移半屏                                 |
| `<c-d>` | 下移半屏                                 |
| `H`     | 调到屏幕顶上                             |
| `M`     | 调到屏幕中间                             |
| `L`     | 调到屏幕下方                             |
| `:n`    | 跳到第n行                                |
| `w`     | 跳到下一个单词开头(标点或空格分隔的单词) |
| `W`     | 跳到下一个单词开头(空格分隔的单词)       |
| `e`     | 跳到下一个单词尾部(标点或空格分隔的单词) |
| `E`     | 跳到下一个单词尾部(空格分隔的单词)       |
| `b`     | 上一个单词头(标点或空格分隔的单词)       |
| `B`     | 上一个单词头(空格分隔的单词)             |
| `ge`    | 上一个单词尾                             |
| `%`     | 在配对符间移动, 可用于()、{}、[]         |
| `gg`    | 到文件首                                 |
| `G`     | 到文件尾                                 |
| `fx`    | 跳转到下一个为x的字符                    |
| `Fx`    | 跳转到上一个为x的字符                    |
| `tx`    | 跳转到下一个为x的字符前                  |
| `Tx`    | 跳转到上一个为x的字符前                  |
| `;`     | 跳到下一个搜索的结果                     |
| `[[`    | 跳转到函数开头                           |
| `]]`    | 跳转到函数结尾                           |

### 文本编辑

| 快捷键         | 说明                                                     |
| -------------- | -------------------------------------------------------- |
| `r`            | 替换当前字符                                             |
| `R`            | 进入替换模式，直至 ESC 离开                              |
| `s`            | 替换字符（删除光标处字符，并进入插入模式，前可接数量）   |
| `S`            | 替换行（删除当前行，并进入插入模式，前可接数量）         |
| `cc`           | 改写当前行（删除当前行并进入插入模式），同 S             |
| `cw`           | 改写光标开始处的当前单词                                 |
| `ciw`          | 改写光标所处的单词                                       |
| `caw`          | 改写光标所处的单词，并且包括前后空格（如果有的话）       |
| `ct,`          | 改写到逗号                                               |
| `c0`           | 改写到行首                                               |
| `c^`           | 改写到行首（第一个非零字符）                             |
| `c$`           | 改写到行末                                               |
| `C`            | 改写到行末（同 c$）                                      |
| `ci"`          | 改写双引号中的内容                                       |
| `ci'`          | 改写单引号中的内容                                       |
| `ci)`          | 改写小括号中的内容                                       |
| `ci]`          | 改写中括号中内容                                         |
| `ci}`          | 改写大括号中内容                                         |
| `cit`          | 改写 xml tag 中的内容                                    |
| `cis`          | 改写当前句子                                             |
| `ciB`          | 改写'{}'中的内容                                         |
| `c2w`          | 改写下两个单词                                           |
| `ct(`          | 改写到小括号前                                           |
| `x`            | 删除当前字符，前面可以接数字，3x代表删除三个字符         |
| `X`            | 向前删除字符                                             |
| `dd`           | 删除当前行                                               |
| `d0`           | 删除到行首                                               |
| `d^`           | 删除到行首（第一个非零字符）                             |
| `d$`           | 删除到行末                                               |
| `D`            | 删除到行末（同 d$）                                      |
| `dw`           | 删除当前单词                                             |
| `dt,`          | 删除到逗号                                               |
| `diw`          | 删除光标所处的单词                                       |
| `daw`          | 删除光标所处的单词，并包含前后空格（如果有的话）         |
| `di"`          | 删除双引号中的内容                                       |
| `di'`          | 删除单引号中的内容                                       |
| `di)`          | 删除小括号中的内容                                       |
| `di]`          | 删除中括号中内容                                         |
| `di}`          | 删除大括号中内容                                         |
| `diB`          | 删除'{}'中的内容                                         |
| `dit`          | 删除 xml tag 中的内容                                    |
| `dis`          | 删除当前句子                                             |
| `d2w`          | 删除下两个单词                                           |
| `dt(`          | 删除到小括号前                                           |
| `dgg`          | 删除到文件头部                                           |
| `dG`           | 删除到文件尾部                                           |
| `d}`           | 删除下一段                                               |
| `d{`           | 删除上一段                                               |
| `u`            | 撤销                                                     |
| `U`            | 撤销整行操作                                             |
| `CTRL-R`       | 撤销上一次 u 命令                                        |
| `J`            | 连接若干行                                               |
| `gJ`           | 连接若干行，删除空白字符                                 |
| `.`            | 重复上一次操作                                           |
| `~`            | 交换大小写                                               |
| `g~iw`         | 替换当前单词的大小写                                     |
| `gUiw`         | 将单词转成大写                                           |
| `guiw`         | 将当前单词转成小写                                       |
| `guu`          | 全行转为小写                                             |
| `gUU`          | 全行转为大写                                             |
| `gg=G`         | 缩进整个文件                                             |
| `=a{`          | 缩进光标所在代码块                                       |
| `=i{`          | 缩进光标所在代码块，不缩进"{"                            |
| `<<`           | 减少缩进                                                 |
| `>>`           | 增加缩进                                                 |
| `==`           | 自动缩进                                                 |
| `CTRL-A`       | 增加数字                                                 |
| `CTRL-X`       | 减少数字                                                 |
| `p`            | 粘贴到光标后                                             |
| `P`            | 粘贴到光标前                                             |
| `v`            | 开始标记                                                 |
| `y`            | 复制标记内容                                             |
| `V`            | 开始按行标记                                             |
| `CTRL-V`       | 开始列标记                                               |
| `y$`           | 复制当前位置到本行结束的内容                             |
| `yy`           | 复制当前行                                               |
| `Y`            | 复制当前行，同 yy                                        |
| `yt,`          | 复制到逗号                                               |
| `yiw`          | 复制当前单词                                             |
| `"+y`          | 复制当前选中到系统剪切板                                 |
| `3yy`          | 复制光标下三行内容                                       |
| `v0`           | 选中当前位置到行首                                       |
| `v$`           | 选中当前位置到行末                                       |
| `vt,`          | 选中到逗号                                               |
| `viw`          | 选中当前单词                                             |
| `vi)`          | 选中小括号内的东西                                       |
| `vi]`          | 选中中括号内的东西                                       |
| `viB`          | 选中'{}'中的内容                                         |
| `vis`          | 选中句子中的东西                                         |
| `gv`           | 重新选择上一次选中的文字                                 |
| `:set paste`   | 允许粘贴模式（避免粘贴时自动缩进影响格式）               |
| `:set nopaste` | 禁止粘贴模式                                             |
| `"?yy`         | 复制当前行到寄存器 ? ，问号代表 0-9 的寄存器名称         |
| `"?p`          | 将寄存器 ? 的内容粘贴到光标后                            |
| `"?P`          | 将寄存器 ? 的内容粘贴到光标前                            |
| `:registers`   | 显示所有寄存器内容                                       |
| `:[range]y`    | 复制范围，比如 :20,30y 是复制20到30行，:10y 是复制第十行 |
| `:[range]d`    | 删除范围，比如 :20,30d 是删除20到30行，:10d 是删除第十行 |
| `ddp`          | 交换两行内容：先删除当前行复制到寄存器，并粘贴           |

### 文件操作

| 快捷键               | 说明                                   |
| -------------------- | -------------------------------------- |
| `:w <filename>`      | 按名称保存文件                         |
| `ZZ`                 | 保存文件（如果有改动的话），并关闭窗口 |
| `:e <filename>`      | 打开文件并编辑                         |
| `:saveas <filename>` | 另存为文件                             |
| `:r <filename>`      | 读取文件并将内容插入到光标后           |
| `:r !dir`            | 将dir命令的输出捕获并插入到光标后      |
| `:wa`                | 保存所有文件                           |
| `:cd <path>`         | 切换Vim当前路径                        |
| `:new`               | 打开一个新的窗口编辑新文件             |
| `:enew`              | 在当前窗口创建新文件                   |
| `:vnew`              | 在左右切分的新窗口中编辑新文件         |
| `:tabnew`            | 在新的标签页中编辑新文件               |


### 实用命令

| 快捷键               | 说明                                             |
| -------------------- | ------------------------------------------------ |
| `/pattern`           | 从光标处向文件尾搜索 pattern                     |
| `?pattern`           | 从光标处向文件头搜索 pattern                     |
| `n`                  | 向同一方向执行上一次搜索                         |
| `N`                  | 向相反方向执行上一次搜索                         |
| `*`                  | 向前搜索光标下的单词                             |
| `#`                  | 向后搜索光标下的单词                             |
| `:s/p1/p2/g`         | 替换当前行的p1为p2                               |
| `:%s/p1/p2/g`        | 替换当前文件中的p1为p2                           |
| `:%s/<p1>/p2/g`      | 替换当前文件中的p1单词为p2                       |
| `:%s/p1/p2/gc`       | 替换当前文件中的p1为p2，并且每处询问你是否替换   |
| `:10,20s/p1/p2/g`    | 将第10到20行中所有p1替换为p2                     |
| `:%s/1\\2\/3/123/g`  | 将“1\2/3” 替换为 “123”（特殊字符使用反斜杠标注） |
| `:%s/\r//g`          | 删除 DOS 换行符 ^M                               |
| `:g/^\s*$/d`         | 删除空行                                         |
| `:g/test/d`          | 删除所有包含 test 的行                           |
| `:v/test/d`          | 删除所有不包含 test 的行                         |
| `:%s/^/test/`        | 在行首加入特定字符(也可以用宏录制来添加)         |
| `:%s/$/test/`        | 在行尾加入特定字符(也可以用宏录制来添加)         |
| `:sort`              | 排序                                             |
| `:g/^\(.\+\)$\n\1/d` | 去除重复行(先排序)                               |
| `:%s/^.\{10\}//`     | 删除每行前10个字符                               |
| `:%s/.\{10\}$//`     | 删除每行尾10个字符                               |

### 其他

| 快捷键                | 说明                       |
| --------------------- | -------------------------- |
| `vim -u NONE -N`      | 开启vim时不加载vimrc文件   |
| `vimdiff file1 file2` | 显示文件差异               |
| `vim -R filename`     | 以只读方式打开（阅读模式） |



## Reference

> [Vim配置]([http://zrainy.top/2019/08/04/Vim%E9%85%8D%E7%BD%AE/](http://zrainy.top/2019/08/04/Vim配置/))
>
> [2018 更新下vim 插件](https://cloud.tencent.com/developer/article/1336735)
>
> [Vim 8 中 C/C++ 符号索引：GTags 篇](https://zhuanlan.zhihu.com/p/36279445)
>
> [Vim自动生成tags插件vim-gutentags安装和自动跳转方法-Vim插件(10)](https://vimjc.com/vim-gutentags.html)
>
> [在Vim中使用gtags](https://www.cnblogs.com/kuang17/p/9449258.html)
>
> [源码阅读工具之Global]([http://laoma.tech/2018/05/22/%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E5%B7%A5%E5%85%B7%E4%B9%8BGlobal/](http://laoma.tech/2018/05/22/源码阅读工具之Global/))
>
> [ubuntu14.04编译gnu global 6.6.3](https://cloud.tencent.com/developer/article/1560566)

> [vimplus -- Github](https://github.com/chxuan/vimplus)
>
> [Vim使用笔记](https://www.cnblogs.com/jiqingwu/archive/2012/06/14/vim_notes.html)


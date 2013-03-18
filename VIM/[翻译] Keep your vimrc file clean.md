[翻译]Keep your vimrc file clean
================================
英文原文: [Keep your vimrc file clean][1].  
作者: Datagrok  
翻译: tisyang  
- - -

本站和其他一些网站所写的关于 *Vim* 的技巧都会让你在 `.vimrc` 文件中添加某些代码。（在 Windows 平台，对应的是 `_vimrc` 文件。）参阅 [:help vimrc-intro][2]

当你添加多次之后，`.vimrc` 文件会变得相当大并且复杂，尤其是你添加的那些分别针对特定语言的配置。更糟糕的是，一些设置有可能互不兼容。

幸运的是，*Vim* 有一个良好的内建机制去组织管理针对特定语言的配置，将它们分置在不同的文件和目录。完整资料请参考 [help vimfiles][3], [:help ftplugin-overrule][4], [:help after-directory][5]

快捷方法就是将全部针对特定语言的配置从你的 `.vimrc` 文件中移到一个名为 `.vim/ftplugin/language.vim` 的文件中。（在 Windows 上对应的是 `$HOME/vimfiles/ftplugin/language.vim` ）

比如本来的 `.vimrc` 文件是这样：

    autocmd FileType * set tabstop=2|set shiftwidth=2|set noexpandtab
    autocmd FileType python set tabstop=4|set shiftwidth=4|set expandtab
    au BufEnter *.py set ai sw=4 ts=4 sta et fo=croql

移动之后变成这样，针对 `python` 的配置移动到了一个单独的文件中：

    " File ~/.vimrc
    " ($HOME/_vimrc on Windows)
    " Global settings for all files (but may be overridden in ftplugin).
    set tabstop=2
    set shiftwidth=2
    set noexpandtab

    " File ~/.vim/ftplugin/python.vim
    " ($HOME/vimfiles/ftplugin/python.vim on Windows)
    " Python specific settings.
    setlocal tabstop=4
    setlocal shiftwidth=4
    setlocal expandtab
    setlocal autoindent
    setlocal smarttab
    setlocal formatoptions=croql

如果你想完全禁用某个随 *Vim* 发行的文件类型的插件时，首先建立你自己的文件类型配置文件（也许是空的），然后添加一行代码：

    let b:did_ftplugin = 1

如果你觉得 *Vim* 自带的某个文件类型的插件大多数功能都不错，只是想改写某些特定的配置项时，你可以将你的配置放到 ` .vim/after/ftplugin/language.vim` （在 Windows 上对应的是 `$HOME/vimfiles/after/ftplugin/language.vim` ）详见 [:help after-directory][5]

如果你想让 *Vim* 识别一个新的扩展名文件，不要在你的 `.vimrc` 文件中使用 `augroup` 。将你的配置放到正确的位置。详见 [:help ftdetect][6]

关于 `~/.vim` 目录（Windows 上的 `$HOME/vimfiles` ）你可以做的更多。目录 `~/.vim/compiler` 是一个存放应用到每个编译器基础之上的配置的好地方（比如说，我也许需要使用 *javac*, *jikes*, *ant* 或者 *make* 去编译然后解析编译器针对对一个 java 源文件的输出。） 【原文： *~/.vim/compiler is a good place to keep configuration that gets applied on a per-compiler basis(for example, I might need to use any of javac, jikes, ant, or make to compile and parse the compiler output for a java source file.)* 】 我也喜欢将一些 color schemes 存放在 `~/.vim/colors`， 用 Vim 帮助文件的格式记笔记存放在 `~/.vim/doc` 。定期运行 `:helptags ~/.vim/doc` 命令可以让我在这些笔记里使用 `:h` 跳转到指定标签。 详见 [:help helptags][7] [:help vimfiles][8]

这个技巧建议将针对特定语言的配置移到一个适当的 `ftplugin file` 中。如果想让它生效，你需要启用文件类型检测功能。输入命令 `:filetype` 来检测文件类型检测是否已经启用。在某些 Linux 发行版中，文件类型检测被禁用，这时你需要添加如下的命令到你的 vimrc 文件中：

    filetype plugin on
    " Alternative: use the following to also enable language-dependent indenting.
    filetype plugin indent on
- - - 

[1]: http://vim.wikia.com/wiki/Keep_your_vimrc_file_clean "Keep your vimrc file clean -- vim.wikia"
[2]: http://vimdoc.sourceforge.net/cgi-bin/help?tag=vimrc-intro
[3]: http://vimdoc.sourceforge.net/cgi-bin/help?tag=vimfiles
[4]: http://vimdoc.sourceforge.net/cgi-bin/help?tag=ftplugin-overrule
[5]: http://vimdoc.sourceforge.net/cgi-bin/help?tag=after-directory
[6]: http://vimdoc.sourceforge.net/cgi-bin/help?tag=ftdetect
[7]: http://vimdoc.sourceforge.net/cgi-bin/help?tag=helptags
[8]: http://vimdoc.sourceforge.net/cgi-bin/help?tag=vimfiles

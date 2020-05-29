# 安装TinyTex和使用过程中所踩的坑
> 适用环境：Ubuntu16.04

##### 安装TinyTex
- 首先介绍TinyTex的主页：https://yihui.org/tinytex/
- 安装只需要在terminal输入一条脚本即可：`wget -qO- "https://yihui.name/gh/tinytex/tools/install-unx.sh" | sh`
- 但需注意，此时所有环境变量都没有配置过，后面我们需要手动配置
- 上述脚本完成后，会出现两个文件(因此删除或转移tinytex都很方便)
    - **$HOME/.TinyTex**：里面放着tinytex主要内容，
        - bin/x86_64-linux里放着很多可执行文件，最重要的是xelatex、pdftex、luatex等编译器(驱动引擎)，它们是用来编译你所编写的`.tex`文件的，因此该文件夹要放进环境变量里：`export PATH=$PATH:/home/gehao/.TinyTex/bin/x86_64-linux`
        - texmf-dist里放着很多安装的文件
    - **$HOME/bin**：里面放着一些软连接和可执行文件，很多文件都是连接到上面提到的`/home/gehao/.TinyTex/bin/x86_64-linux`文件夹下，这个文件夹也要放进环境变量里:`export PATH=$PATH:/home/gehao/bin`
- 这个时候初步完成了配置，接下来做进一步有关vscode的配置

##### 关于latexindent在vscode上无法使用的问题
- vscode使用latex，先安装插件latex workshop，然后在`settings.json`里加入如下配置语句：
    ~~~bash
    # recipes是调用指令，会出现在vscode左侧栏TEX中的编译选项里，相当于点击一个指令就调用该指令规定的多个动作
    # 这里我把"xelatex -> bibtex -> xelatex*2"放在第一位，此时latex默认编译就是执行1次xelatex+1次bibtex+2次xelatex(有参考文献时的编译方式)
    "latex-workshop.latex.recipes": [
        {
            "name": "xelatex -> bibtex -> xelatex*2",
            "tools": [
                "xelatex",
                "bibtex",
                "xelatex",
                "xelatex"
            ]
        },
        {
            "name": "xelatex",
            "tools": [
                "xelatex"
            ]
        },
        {
            "name": "latexmk",
            "tools": [
                "latexmk"
            ]
        },
        {
            "name": "bibtex",
            "tools": [
                "bibtex"
            ]
        },
        {
            "name": "pdflatex -> bibtex -> pdflatex*2",
            "tools": [
                "pdflatex",
                "bibtex",
                "pdflatex",
                "pdflatex"
            ]
        }
    ],
    # tools里存放的是具体的命令(给recipe里调用)，这里的command必须是已经配置在环境变量里的命令，否则不会生效
    # %DOCFILE%便于中文名路径
    "latex-workshop.latex.tools": [
        {
            "name": "latexmk",
            "command": "latexmk",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "-pdf",
                "%DOCFILE%"
            ]
        },
        {
            "name": "xelatex",
            "command": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOCFILE%"
            ]
        },
        {
            "name": "pdflatex",
            "command": "pdflatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOCFILE%"
            ]
        },
        {
            "name": "bibtex",
            "command": "bibtex",
            "args": [
                "%DOCFILE%"
            ]
        }
    ],
    # tab指默认以vscode侧栏的pdf做显示预览
    "latex-workshop.view.pdf.viewer": "tab",
    # autoClean=onBuilt指的是built完后，自动删除中间产生的文件，这些文件类型自定义如下：
    "latex-workshop.latex.clean.fileTypes": [
        "*.aux",
        "*.bbl",
        "*.blg",
        "*.idx",
        "*.ind",
        "*.lof",
        "*.lot",
        "*.out",
        "*.toc",
        "*.acn",
        "*.acr",
        "*.alg",
        "*.glg",
        "*.glo",
        "*.gls",
        "*.ist",
        "*.fls",
        "*.log",
        "*.fdb_latexmk"
    ],
    "latex-workshop.latex.autoClean.run": "onBuilt",
    # 淡入latexindent路径，实际上它必须要已经配置在环境变量里才行
    "latex-workshop.latexindent.path": "latexindent",
    ~~~
- 注意：**latexindent**是用于给latex自动格式化的，它成功运行前提是有Perl环境，因此要先去`/home/gehao/.TinyTeX/texmf-dist/scripts/latexindent`目录下找到`latexindent.pl`，使用`sudo perl latexindent.pl`命令先运行该perl脚本，命令行会提示你缺少什么安装环境，按照提示安装即可，对我来说，缺少的环境：
    ~~~bash
    sudo cpan Log::Log4perl
    sudo cpan Log::Dispatch
    sudo cpan File::HomeDir
    sudo cpan YAML::Tiny
    sudo cpan Unicode::GCString
    # 以及别人没有遇到的，但坑了我很久的
    sudo cpan Log::Dispatch::file
    ~~~
- 装完该环境后，导入环境变量(这也是tinytex要导入的三个环境变量里的最后一个)：
    ~~~bash
    # 进入环境变量
    vim ~/.bashrc
    # 添加
    export PATH=$PATH:/home/gehao/.TinyTeX/texmf-dist/scripts/latexindent
    # 退出刷新
    source ~/.bashrc
    # 注意刷新不一定立即生效(不知道什么原因)，因此要耐心地多尝试几次，甚至重启后尝试
    ~~~
- 在vscode里，调用latexindent格式化的快捷键是"ctrl+shift+I"

##### tinytex很多包都没装的情况
- 建议先拿一个模板跑一下，然后根据提示安装大多数缺少的包，主要用到以下两条指令：
    ~~~bash
    tlmgr search --global --file '.sty'(包名) 或 '.tfm'(字体名) # 搜索缺少的包或字体
    tlmgr install ... # 安装对应的包或字体
    ~~~
- 举例：出现字体未安装的情况：
    ~~~bash
    tlmgr search --global --file pcrr7t.tfm
    # show:
    courier:  
    texmf-dist/fonts/tfm/adobe/courier/pcrr7t.tfm
    # then
    tlmgr install courier
    ~~~
- 一个必要安装的中文包：ctex包

##### 关于bibtex和参看文献：
- 注意到，bibtex(用于引用参考文献)是latex论文中必不可少的编译器工具，但tinytex并没有内置这个编译器的可执行文件
- 必须要先用xelatex编译一次，再用bibtex编译一次，再用xelatex编译两次之后，论文的参考文献才会成功从问号(?)变成正常的引用
- 关于LaTeX的驱动引擎(编译器)，tinytex内置的有xelatex、luatex和pdftex；因此bibtex是需要额外下载的：
    ~~~bash
    tlmgr install bibtex
    ~~~
    此时bibtex可执行文件已经在`/home/gehao/.TinyTex/bin/x86_64-linux`文件夹下了，虽然这个文件夹在环境变量里，但不知为什么总是无法在命令行里调用bibtex指令，因此我把该文件夹下的bibtex可执行文件右键点击创建链接，然后把链接放到了`/home/gehap/bin`文件夹下，这个时候刷新环境变量，在命令行里输入bibtex指令就有反应了(注：测试某可执行文件是否成功被加入环境变量，只需在terminal里测试是否能够直接调用该指令即可)
- 做到这一步，已经能够成功编译.tex且显示出正确的pdf文件了

##### 常见问题
- 使用中文字体会出现以下warning：
    ~~~
    Fandol is being set as the default font for CJK text.
    (xeCJK)	Please make sure it has been properly installed.
    Font "FandolSong-Regular" does not contain requested
    (fontspec)	Script "CJK".
    ~~~
- 虽然有warnings, 但中文还是可以正常现实的，如果要解决它们,可以换字体, 只需加入，比如： \setCJKmainfont{Microsoft YaHei}, \setCJKmainfont{SimSun}
- 事实上如果用v3.2.10 or later version的xeCJK, 使用中文字体Fandol得到时候总是会出现烦人的warning”，所以还是换字体吧…
- 如果还是遇到`Font "font-name" does not contain requested (fontspec) Script "CJK".` 的waring, 但中文又是可以正常现实的，那么只要加入`\PassOptionsToPackage{quiet}{fontspec}`即可(就是简单地disable警告)；或者直接更换字体：
    ~~~
    \documentclass{article}
    % \PassOptionsToPackage{quiet}{fontspec}
    \usepackage{xeCJK}  
    \setCJKmainfont{Microsoft YaHei}
    \begin{document}
    中文测试
    \end{document}
    ~~~

##### 正向反向跳转
- 在 LaTeX 文件中，按 Ctrl + Alt + J 跳转到对应的 PDF 文件位置
- 在 PDF 文件中，按下 Ctrl + ← 同时鼠标单机，跳转到对应的 LaTeX 文件位置
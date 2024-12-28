这个教程的GitHub页面可访问。
点击 [这里](https://toshiocp.github.io/Gobject-tutorial/).

#### 关于此教程

GObject是给GTK库的基础系统，GTK库现在的版本为4。
GTK提供Linux上的图形界面并且为GNOME桌面环境和许多程序所用。
参见[GTK 4 教程](https://github.com/ToshioCP/Gtk4-tutorial)。
理解GTK 4的一大难题是GObject的复杂程度。
这个教程对想要学习GTK 4的人有用。
本教程的读者也应读GTK 4教程因为GTK是迄今为止唯一的GObject应用（*有的库例如json-glib和语言vala就用到了GObject的核心内容，但是理解GTK确实是理解GObject的一个很好的手段*）。

[GObject API参考](https://docs.gtk.org/gobject)讲解了所有GObject的必要知识。
本教程的内容没有超过文档的范围。
它只展示了例子，如何编写GObject程序。
但是我相信本教程对学习GObject系统感到困惑的初学者很有用。
读者学习本教程时仍需参考GObject文档。

#### Generating GFM, HTML and PDF (*译者不想翻译下面部分*)

The table of contents are at the end of this file and you can see all the tutorials through the link.
However, you can make GFM, HTML or PDF by the following steps.
GFM is 'GitHub Flavored Markdown', which is used in the document files in the GitHub repository.

1. You need Linux operating system, ruby, rake, pandoc and LaTeX system.
2. download the [GObject-tutorial repository](https://github.com/ToshioCP/Gobject-tutorial) and uncompress the files.
3. change your current directory to the top directory of the files.
4. type `rake` to produce GFM files. The files are generated under `gfm` directory.
5. type `rake html` to produce HTML files. The files are generated under `docs` directory.
6. type `rake pdf` to produce a PDF file. The file is generated under `latex` directory.

This system is the same as the one in the `GTK 4 tutorial` repository.
There's a document `Readme_for_developers.md` in `gfm` directory in it.
It describes the details.

#### Contribution

If you have any questions, feel free to post an issue.
If you find any mistakes in the tutorial, post an issue or pull-request.
When you give a pull-request, correct the source files, which are under the 'src' directory, and run `rake` and `rake html`.
Then GFM and HTML files are automatically updated.

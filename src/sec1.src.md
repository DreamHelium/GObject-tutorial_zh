# 前言和许可

## 前言

### 教程文档

本教程是关于GObject库的。
它原本是用于Linux上的C编译器的，但是它现在用途更广泛，还在windows和macOS，以及Vala, python等语言上所有应用。
但是，本教程只描述_Linux上的C程序_。

虽然没有充分测试，但是以下环境满足的情况下应该能成功编译。（**原文只针对Linux平台，但是译者的其他程序在Windows上功能仍然正常，所以以下条件会比原文宽泛**）

- 通用计算机平台（~借助Termux，手机其实也可以~）
- C编译器
- GLib，尽量使用2.74或更高版本
- pkg-config
- meson和ninja

### Tools to make GFM, HTML and PDF files

This repository includes ruby programs.
They are used to create Markdown(GFM) files, HTML files, LaTeX files and a PDF file.

You need:

- Linux distribution like Ubuntu.
- Ruby programming language.
There are two ways to install it.
One is installing the distribution's package.
The other is using rbenv and ruby-build.
If you want to use the latest version of Ruby, you will need rbenv and ruby-build.
- Rake.
It is a gem, which is a library written in ruby.
Ruby package includes Rake gem as a standard library so you don't need to install it separately.

## License

Copyright (C) 2021-2022  ToshioCP (Toshio Sekiya)
Copyright (C) 2024 Dream_Helium (Chinese Translator)

GObject tutorial repository contains the tutorial document and software such as converters, generators and controllers.
All of them make up the 'GObject tutorial' package.
This package is simply called 'GObject tutorial' in the following description.

'GObject tutorial' is free; you can redistribute it and/or modify it under the terms of the following licenses.

- The license of documents in GObject tutorial is the GNU Free Documentation License as published by the Free Software Foundation;
either version 1.3 of the License or, at your opinion, any later version.
The documents are Markdown, HTML and image files.
If you generate a PDF file by running `rake pdf`, it is also included by the documents.
- The license of programs in GObject tutorial is the GNU General Public License as published by the Free Software Foundation;
either version 3 of the License or, at your option, any later version.
The programs are written in C, Ruby and other languages.

GObject tutorial is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
See the GNU License web pages for more details.

- [GNU Free Documentation License](https://www.gnu.org/licenses/fdl-1.3.html)
- [GNU General Public License](https://www.gnu.org/licenses/gpl-3.0.html)

The licenses above is effective since 15/August/2023.
Before that, GPL covered all the contents of the GObject tutorial.
But GFDL1.3 is more appropriate for documents so the license was changed.
The license above is the only effective license since 15/August/2023.

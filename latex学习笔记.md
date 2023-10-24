# latex学习笔记

## 示例

```latex
% 定义文档类型
% 英文有article, book, beamer
% 中文有ctexbook, ctexart, ctexbeamer
\documentclass{ctexart}

\documentclass[12pt, a4paper, oneside]{ctexart}  % 默认字体为12pt，A4纸，单面打印

% 加载宏包
\usepackage{amsmath, asthm, amssymb, graphicx}

\title{文档标题}
\author{jab}
\date{\today}

\begin{document}
\maketitle  % 显示标题、作者、日期等
Hello World!  % 正文放在document环境中
\end{document}
```

## 插入

### 位置关系

- htbp
  - h：当前位置；t：页面最上方；b：页面最下方；p：放在专门为浮动对象设置的页面上
  - ！：忽略其它内部要求；H：准确地放在文本中的位置
    - ht：优先放在当前页面的上部，若不满足则顺延到下页上部
- 基础技巧
  - 靠左$\to$靠右：`\usepackage[export]{adjustbox}`，然后在引用图像时加入`[width=0.5\textwidth, right]`

### 图片

```latex
\begin{figure}[htbp]
	\centering  % 图片居中
	\includegraphics[width=0.95\linewidth]{image.jpg}
	\caption{标题}
	\label{标签，可用于引用图片}
\end{figure}
```

- \linewidth: 栏宽；\textwidth: 整行宽
- 建议将图片先转化为eps格式，再插入，jpg格式放大后不清晰
  - windows下可使用latex自带的`bmeps -c [原图像名] [新图像名]`
- `\graphicspath{{imgs/}}`：指定从哪个文件夹中找图片

```latex
\usepackage{graphicx, subfig}

% 多幅图片，仅限并排
\begin{figure}[h]
	\subfloat[子图名]{
		\includegraphics[width=0.45\linewidth]{a.eps}
		\label{fig:sub_a}
	}
	\hfill
	\subfloat[子图名]{
		\includegraphics[width=0.45\linewidth]{b.eps}
		\label{fig:sub_b}
	}
	\caption{Caption for this figure with two images}
	\label{fig:image2}
\end{figure}
```

```latex
% 多幅图片，更多排版设置
\begin{figure}[h]
	\begin{minipage}[b]{0.45\linewidth}  % 创建左侧小区域，可按一般方式进行排版
		\centering
		\includegraphics[width=0.9\linewidth]{demo.jpg}  % 此处\linewidth为minipage的栏宽
		\caption{}
		\label{}
	\end{minipage}
\end{figure}
```

### 表格

- 一般性

  - ```latex
    \usepackage{subcaption}
    
    \begin{table}[htbp]
        \centering
        \caption{表格标题}
        \begin{tabular}{|l|c|r|}  % l为靠左，c为居中，r为靠右，|为竖直边框
        % % \begin{tabular*}{\linewidth}{@{}llll@{}}：占满两栏
        	\hline  % 水平边框
            1 & 2 & 3 \\  % &代表对齐，\\代表换行
            \hline
            4 & 5 & 6 \\
            \hline
            7 & 8 & 9 \\
            \hline
        \end{tabular}
    \end{table}
    ```

  - `\cline{2-3}`：该行仅第2格到第3格有框线

  - `\begin{table}`仅占用一栏，`\begin{table*}`占用整个页面宽

- 调整宽度

  - ```latex
    % 调整列间距
    \makebox[0.1\textwidth][c]{第1行列名}  % 写在第1行
    
    % 调整行间距
    \renewcommand{\arraystretch}{倍数}  % 写在\begin{table}和\begin{tabular}中间
    
    % 压缩整个表格宽度
    \resizebox{\textwidth}!{
    \begin{tabular}
    ...
    \end{tabular}}
    ```

- 三线表

  - ```latex
    % 三线表
    \usepackage{booktabs}
    % \begin{tabular}之后
    \toprule  % 最上面的粗线
    % 第1行
    \midrule % 中间的细线
    % 下面各行
    \bottomrule  % 最下面的粗线
    
    % \midrule可重复使用多次
    % \cmidrule(l){左列号-右列号}，在两列之间画上连线，列号从1开始计数
    ```

  - ```latex
    % 直接在最后加注释
    % 在数据最后一行后定义multicolumn，打通所有单元格，写注释，不画表格线
    \multicolumn{列数}{p{\textwidth}}{注释}  % 也可改成\linewidth
    
    % 对表格中需要加注释的地方标注
    \usepackage{threepartable}
    
    % 三线表
    \begin{threeparttable}  % 在\begin{table}和\begin{tabular}之间
    % \caption放在\begin{threeparttable}和\begin{tabular}之间
    % 需要注释的地方加上\tnote{1}
    % 在\end{tabular}后加上
    \begin{tablenotes}
    \footnotesize
    \item[1] explanation.
    \end{tablenotes}
    % 也可以不加\tnote{}，然后在最后直接使用\item
    ```

- 多行多列

  - ```tex
    % 多行：multirow{合并的行数}{所占大小，默认为*（不加{}）}{文本}
    % 多列：multicolumn{合并的列数}{对齐格式}{文本}
    % 被合并的位置需要用&空出，但不填东西
    ```

- 表格并排

  - ```tex
    \begin{table}
        \centering
        \begin{subtable}[h]{0.45\textwidth}
            \begin{tabular}{cc}
                train & result \\
                0 & 0 \\
            \end{tabular}
        \end{subtable}
        \begin{subtable}[h]{0.45\textwidth}
            \begin{tabular}{cc}
                train & result \\
                0 & 0 \\
            \end{tabular}
        \end{subtable}
    \end{table}
    ```


### 列表

```latex
% itemize为无序列表，enumerate为有序列表，description为描述
\begin{enumerate}
    \item 这是第一点; 
    \item 这是第二点;
    \item 这是第三点. 
\end{enumerate}

\begin{enumerate}
    \item[(1)] 这是第一点; 
    \item[(2)] 这是第二点;
    \item[(3)] 这是第三点. 
\end{enumerate}
```

### 定理

```latex
% {theorem}是环境明后才能
% {定理}指的是该环境显示的名称是“定理”
% [section]让theorem环境在每个section中单独编号
\newtheorem{theorem}{定理}[section]

% 在正文中
\begin{theorem}[定理名称]
	这里是定理的内容
\end{theorem}
```

### 页面

- 使用geometry宏包

```latex
\usepackage{geometry}
\geometry{left=2.54cm, right=2.54cm, top=3.18cm, bottom=3.18cm}
\linespread{1.5}  % 行间距
```

### 页码

```latex
% aiph为小写字母，Aiph为大写字母，Roman为大写罗马数字，arabic为阿拉伯数字
\pagenumbering{roman}
\setcounter{page}{0}  % 页码从0开始
```

### 公式

```latex
% 行内公式
$E=mc^2$
% 行间，或者用$$...$$
% $$无公式号，\begin{equation}有
\begin{equation}\label{eq4}  % 与正文间不空行，为了美观可在空行加%隔开
	E=mc^2
\end{equation}
```

- 大分式：`\dfrac{}{}`；小分式：`\frac{}{}`

- 可变大小括号：`\left(`, `\right)`

  - 中间需要隔开：`\left(..\middle|..\right)`

- 字体

  - 加粗：`\bm{}`或`\mathbf{}`
  - 取消斜体：`\mathrm{}`

- 分段函数

  - ```latex
    $$
    f(x)=\begin{cases}
        x, & x>0, \\
        -x, & x\leq 0.
    \end{cases}
    $$
    ```

- 多行公式

  - ```latex
    $$
    \begin{aligned}
    a & =b+c \\
    & =d+e
    \end{aligned}
    $$
    ```

- 矩阵

  - ```latex
    $$
    \begin{bmatrix}  % bmatrix为方括号，pmatrix为圆括号
        a & b \\
        c & d
    \end{bmatrix}
    $$
    ```

### 引用

- 引用图、表：`\ref{图、表的label}`
- 引用公式：`\eqref`

- 引用参考文献

  - 将bibtex放在refer.bib中

  - ```latex
    % 在文章最后加上，说明从refer.bib中寻找参考文献
    \bibliographystyle{IEEEbib}
    \bibliography{refer.bib}
    ```

  - 需要引用的地方，使用`\cite{参考文献的关键词}`

### 算法表

- `\usepackage[linesnumbered, ruled, vlined]{algorithm2e}`
  - `linesnumbered`：显示行号
  - `ruled`：标题显示在上方，默认在下方
  - `vlined`：同缩进前有联系那
  - `boxed`：将算法插入在一个盒子里

- ```latex
  \begin{algorithm}
  	\SetAlgoLined  % 显示end
  	\caption{算法名字}
  	\KwIn{input parameters A, B, C}
  	\KwOut{output result}
  	some description\;  % \;用于换行
  	\For{condition}{  % for循环
  		only if\;
  		\If{Condition}{
  		1\;
  		}
  	}
  	\While{not at end of this document}{  % while循环
  		if and else\;
  		\eIf{condition}{  % if-else
  			1\;
  		}{
  			2\;
  		}
  	}
  \end{algorithm}
  ```

### 脚注

- `\footnote{text for footnote}`：标在要解释的正文后面，自动创建脚注
- `\footnote[number]{text for footnote}`：指定脚注数字

## 符号

- `%`：注释
- 省略号：`\cdots`：横向居中省略号；`\vdots`：竖向省略号；`\ddots`：对角线向省略号

### 排版

- `\newpage`：换新页
- `\section{}`：小标题
- `\subsection{}`，`\subsubsection{}`

### 字体

- `\underline{}`：下划线
- `\textbf{}`：加粗

### 空格

- 小：`\ `；中：`\;`；大：`\quad`

### 其它

- 文本中有下划线，需要转义`\_`

## 操作

### vscode

- `alt+b`: build
- `ctrl+alt+v`: 查看pdf
- 在setting.json中修改latex配置
- `alt+z`：打开或关闭自动换行

### arxiv提交

- overleaf提交：overleaf能自动打包需要的文件，建议用texstudio重新编译检查问题

- 本地提交：参考文献提供在.bbl文件中，需要与.tex文件同名，不需要改.tex文件最后的\bibliography

- tips

  - ```latex
    \name{xxx$^1$, xxx$^2$, 
    \thanks{* Corresponding author\newline \indent \indent
    This work was supported by ...}}
    \address{}
    ```

  - 让arxiv使用pdflatex编译

    - 在前五行空出一行，加入`\pdfoutput=1`
    - 以`.jpg`，`.png`等格式提供图片，不用`.eps`格式

  - 避免图片文件名中有下划线_
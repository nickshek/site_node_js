---
title: 學習Latex
date: 2017-02-06
categories:
    - Latex
---
近期對Latex相當有興趣， 之前曾經稍為在Wiki了解過Latex，覺得Symbol好像看起來好像好複雜，所以便提不起興趣學了。
直至近期發現好多整齊漂亮的pdf文件原來都是由Latex的編譯器建立，便開始想了解一下，當學習一個全新的markup language。
以下的範例會建立一個pdf文件，簡單地記綠「Today I Learned」，好像[https://github.com/thoughtbot/til](https://github.com/thoughtbot/til) 或 [https://github.com/jbranchaud/til](https://github.com/jbranchaud/til)，只是用Latex編寫而已，雖然這個情況或者用markdown會比較合適，但是用來練習Latex也不錯:

首先安裝 texlive:

```
sudo apt-get install texlive-full
```

Text Editor 我覺得用atom便好了無需再額多安裝 texmaker。反正本身電腦只使用了128G SSD， 沒有太多空間...

之後把以下原始碼儲存為main.tex，不妨花點時間了解一下latex語法，推薦不妨抽少少時間看看[Latex in 157 minutes](https://tobi.oetiker.ch/lshort/lshort.pdf)

```

\documentclass{article}

\usepackage{hyperref}

\RequirePackage{fontspec}
\RequirePackage{xeCJK}

\setCJKmainfont{Droid Sans Fallback}
\XeTeXlinebreaklocale "zh"
\XeTeXlinebreakskip = 0pt plus 1pt

\title{Today I learned.今日我所學}
\author{Nick Shek}
\date{2017-02-07}

\begin{document}

\maketitle

\clearpage

\tableofcontents

\clearpage

\section{MySQL}

\begin{itemize}
\item Setting up MySQL replication without the downtime: \href{https://plusbryan.com/mysql-replication-without-downtime}{https://plusbryan.com/mysql-replication-without-downtime}
\end{itemize}

\section{Latex}

\begin{itemize}
\item Latex 中文化解決方案:
  \begin{itemize}
    \item \href{https://goo.gl/6ajvrM}{https://goo.gl/6ajvrM}
    \item \href{http://pangomi.blogspot.hk/2012/11/latex-in-ubuntu.html}{http://pangomi.blogspot.hk/2012/11/latex-in-ubuntu.html}
  \end{itemize}
\item Create Table of contents in Latex \href{https://www.sharelatex.com/learn/Table\_of\_contents}{https://www.sharelatex.com/learn/Table\_of\_contents}
\end{itemize}





\end{document}

```

確保Droid Sans Fallback字型已經安裝後，執行


```
xelatex main.tex
```

如無意外，便能成功建立一個pdf，及中文字型能正確顯示。

參考了不同網站後，現在總算能輕鬆建立pdf，而pdf本身能顯示中文。
了解Latex基本Symbol後，其實建立一份Latex 沒有想像中那麼複雜。

如有興趣，也不妨看看: [https://github.com/nickshek/latex-til](https://github.com/nickshek/latex-til)

有時間的話會不定期更新

Source: [http://milq.github.io/install-latex-ubuntu-debian/](http://milq.github.io/install-latex-ubuntu-debian/)

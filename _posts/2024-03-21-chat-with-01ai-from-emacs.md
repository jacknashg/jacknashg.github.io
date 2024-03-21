---
layout: post
category: 技术博客
title: 在 Emacs 里愉快地和“零一万物” AI 聊天
---

# 目录

1.  [程序运行逻辑图](#orgdc278ac)
    1.  [操作入口](#org53a9c22)
    2.  [url retrieve 异步逻辑](#org626f4f3)

简短的有趣尝试，通过 emacs-lisp 编程实现在 emacs 中访问 01.ai 的 API，跟大模型对话。heavily inspired by org-ai（<https://github.com/rksm/org-ai>）。


使用方式：  

1.  在 org-mode 中新建一个 #+begin\_mai &#x2026; #+end\_mai 的 special block，通过 :model 参数设定要使用的模型
2.  在 special block 中输入问题
3.  C-c C-c, 后台异步流式（stream）调用 01.ai 的 API（ <https://api.lingyiwanwu.com/v1/chat/completions> ）
4.  API 流式响应的内容被插入到 special block 中

使用效果：  
![demo](/assets/imgs/240321syxg.gif)

<a id="orgdc278ac"></a>

# 程序运行逻辑图


<a id="org53a9c22"></a>

## 操作入口

![操作入库.png](/assets/imgs/240321czrk.png)


<a id="org626f4f3"></a>

## url retrieve 异步逻辑

![异步逻辑.png](/assets/imgs/240321yblj.png)


---
layout: post
category: 技术博客
title: Emacs org-mode 的 inline latex 公式中显示中文
---

默认情况下，inline latex 公式不支持中文，使用不便。跟踪实现 org.el 中实现 inline latex 公式 preview 功能的源码，进行适当配置修改，inline 公式就能显示中文了。


inline formula preview 的调用 stack：  

1.  org&#x2013;latex-preview-region
2.  org-format-latex  
    1.  org&#x2013;make-preview-overlay
3.  org-create-formula-image  
    实现转 latex 为图片的核心功能。原理是把 latex region 的内容放在中间，同时穿靴戴帽组装成完整的 latex 文件，再通过（根据 org-preview-latex-default-process, org-preview-latex-process-alist 两个变量的定义来决定具体逻辑，默认是：） latex 转成 div，再通过 dvipng 转成 png，把 png 插入当前位置预览。  
    穿靴戴帽，靴子很简单，帽子的来源：  
	{% highlight elisp %}
	'''elisp
        (latex-header
         (or (plist-get processing-info :latex-header)
             (org-latex-make-preamble
              (org-export-get-environment (org-export-get-backend 'latex))
              org-format-latex-header
              'snippet)))
	'''
    

**所以，要让 org-mode 中 inline latex 公式 preveiw 能显示中文，步骤：**  

1.  操作变量 org-preview-latex-process-alist 值，增加自定义的内容 xelatex-chinese：  
	'''elisp
        (add-to-list 'org-preview-latex-process-alist
        	     '(xelatex-chinese
        	       :programs ("xelatex" "convert")
        	       :description "XeLaTeX with Chinese support dvi > png"
        	       :message "you need to install the programs: xelatex and divpng."
        	       :image-input-type "pdf"
        	       :image-output-type "png"
        	       :image-size-adjust (1.7 . 1.6)
        	       :latex-header "\\documentclass[preview]{standalone}\n\\usepackage{xeCJK}\n\\setCJKmainfont{华文楷体}\n\\usepackage[usenames]{color}\n\\usepackage{amsmath}\n\\pagestyle{empty}" ;; pagestyle{empty} 是必须的
        	       :latex-compiler ("xelatex -interaction nonstopmode -output-directory %o %f")
        	       :image-converter ("convert -density 150 %f %O")))
	{% endhighlight %}

2.  操作变量 org-preview-latex-default-process，配置使用 xelatex-chinese 设定  
	{% highlight elisp %}    
        (setq org-preview-latex-default-process 'xelatex-chinese)
	{% endhighlight %}

**解决方案的扩展说明**  

1.  latex 不支持中文，如果要在公式中显示中文，必须使用 xelatex 引擎
2.  xelatex 引擎只能生成 pdf，不能（像 latex 一样）生产 dvi 文件
3.  要选择合适的 documentclass，让 xelatex 生成的 pdf 只包含公式，而不是默认的（包含页眉、页脚、页边距等大量空白的完整）文档
4.  要在公式中显示中文，必须 \usepackage{xeCJK} （用于中文支持）、 \usepackage{amsmath} 并且在公式中用 \text{} 来包裹汉字
5.  \documentclass{ctexart} 在这里不适用，会生成整页文档，而不是只包含公式部分的内容
6.  在高分辨率设备上使用 convert，务必设置 density 参数为稍大数，否则图片很小，看不清楚
7.  通过 convert 的 resize （而不是 density）来放大图片，没有效果
8.  插入的 inline formula 图片可能过长，原因是 org-create-formula-image 函数中穿靴戴帽时进行了颜色设定，不明原因会导致图片过长，修改成：  
	{% highlight elisp %}
        (with-temp-file texfile
              (insert latex-header)
              (insert "\n\\begin{document}\n"
        	      string
        	      "\n\\end{document}\n"))
	{% endhighlight %}


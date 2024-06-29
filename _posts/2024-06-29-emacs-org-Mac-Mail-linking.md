---
layout: post
category: 技术博客
title: Mac 上在 Emacs org-mode 中插入邮件链接，快速打开指定邮件
---

# Table of Contents

1.  [新增 org-mode 的 link type](#orgec1ebd5)
2.  [Mac 通过 applescript + automator 实现：通过快捷键获取 Mail 中选定的 mail 的 Message-ID](#org579a3d0)
3.  [使用效果](#org882e767)


<a id="orgec1ebd5"></a>

# 新增 org-mode 的 link type

在 Mac 上使用 Emacs 的 org-mode，增加 "MacMail" link type，点击后调用 Mac 的 Mail 应用，打开指定邮件  

**以下配置，通过 M-x customize-variable org-link-parameters 配置永久生效**  

	{% highlight elisp %}
    ;; 申明新的 link type，并绑定 link 处理逻辑
    (org-link-set-parameters "MacMail"
    			 :follow (lambda (path) 
    				   (shell-command (concat "open message://%3c" path "%3e"))))
    
    ;; org 中新使用新的 link 的示例. 注意，%3c 表示 < 并  %3e 表示 >
    ;; [[MacMail:20240628103122.1fb26f5c5767c@xxyyzz][xxxyyyy]]
	{% endhighlight %}


<a id="org579a3d0"></a>

# Mac 通过 applescript + automator 实现：通过快捷键获取 Mail 中选定的 mail 的 Message-ID

**automator 中创建 quick action**  
步骤：  

1.  Mac 打开 Automator 新增一个 quick action  
    -   “工作流程收到” 设置为 “没有输入”
    -   “位于” 设置为 “邮件”
    -   从左边拖拽 “运行 AppleScript” 组件到右边，并编辑 applescript 为：  
        
   	{% highlight applescript %}
         on run {input, parameters}
            
            	tell application "Mail"
            		set selectedMessages to selection
            		if (count of selectedMessages) is equal to 0 then
            			display dialog "No message selected!" buttons {"OK"}
            			return
            		end if
            		set theMessage to item 1 of selectedMessages
            		set messageID to message id of theMessage
            		set the clipboard to messageID
            	end tell
            
            	display notification ""Message-ID 已复制！ with title "通知" sound name "Submarine"
            	delay 3
            
            
            	return input
            end run
	{% endhighlight %}			
2.  保存为名为 getMailMessageID 的 xxx
3.  操作界面效果：  
	![截屏1](/assets/imgs/20240629_01.png)

**配置 shortcut，（只）在 Mail 中 Control+Option+M 就能得到 selected mail 的 Message-ID**  

1.  Mac -> Preferences -> Keyboard -> Shortcut
2.  服务 -> 通用 -> getMailMessageID -> 点击最右边键入快捷键组合
3.  操作界面效果：  
	![截屏2](/assets/imgs/20240629_02.png)


<a id="org882e767"></a>

# 使用效果
	![截屏3](/assets/imgs/20240629_03.gif)


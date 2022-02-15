## 拓展：AHK 键盘映射

Original 米博宇 [CodeWater](javascript:void(0);)

**CodeWater**

Weixin ID CodeMiMi

About Feature 分享科研工作中的代码应用，内容涉及水利专业知识、编程基础 (Fortran, Matlab, C/C++)、数据分析、数学建模与实现、杂谈等。

*2019-11-12*

收录于话题

AutoHotkey 是 Windows 下一款修改键盘映射的工具，其强大之处在网上搜一搜就能体会到。相比通过改注册表来改键盘映射的方式，AHK 的修改方法更为**简单方便**，也更**强大**，只需几行配置代码，随时生效或停用，丰富的组合键设置更是让人有种为所欲为的感觉。到官网（https://www.autohotkey.com/）下载安装 AHK 软件，新建一个文本文件写上自己的配置代码，将文件后缀名改成. ahk，运行该文件即可。如果想开机生效，只需将该配置文件添加到开机启动项。至于内存开销，基本可以忽略。就是这么小巧，就是这么强大，就是这么任性。

独乐乐不如众乐乐，这里分享一下我个人的简单配置。如果你是 Vim 用户，这些配置将使你在 Vim 之外的地方如鱼得水，就算不是，熟练使用这些快捷操作也能让你的键盘工作事半功倍。当然你也可以根据自己的使用习惯加以修改，或者创造出自己的键盘绝技。

最基本的 AHK 语法我这里就不重复讲了，网上有很多。这里只贴代码，并说说我这么改的理由。

先看第一部分。分号后面为注释内容。CapsLock 这个最方便按到却基本没用的键被我映射成了 Esc，对 Vim 来说 Esc 是最常用的无副作用键，没事按两下不会有坏处，平时打中文敲了一长串发现前面写错了，往回删太费事，Esc 一下就清净了，CapsLock 则被我永久关闭了，它的突然出现对 Vimer 来说绝对是场噩梦。空格右边的 Alt 被我映射成了 Ctrl，因为 Alt 的使用频率远不如 Ctrl，不如让右手大拇指在敲空格之余肩负起 Ctrl 的使命。

```
;================================================================
SetCapsLockState, AlwaysOff  ; 关闭CapsLock
CapsLock::Esc                ; CapsLock 替换成 Esc
RAlt::RCtrl                  ; 右Alt 替换成 右Ctrl
```

第二部分是以 CapsLock 为基础的组合键，由于 CapsLock 很容易按到，系统及各类软件中不会使用它作组合键，这让它成了最最常用自定义组合键的不二之选。上下左右不好按？改到 KJHL 上来吧（Vimer 会心一笑）。水平移动太慢？试试按单词移动吧。Home 和 End 太远？改到 I 和 A 上来吧（Vim 中 I 表示行首插入，A 表示行尾插入）。翻页键也太远？就近找两个符号吧（[]）。鼠标滚轮也能定义？是的，CapsLock+; 表示往上滚一次，CapsLock+’表示往下滚一次，打字的时候是不是觉得顺畅了很多。至于整行复制剪切粘贴，看自己平时需求吧，Vimer 肯定是必不可少的。着重说一下向前删和向后删，一个映射到逗号，一个映射到句号，Excuse me？好吧，其实想的是小于号 < 和大于号>，这样就很形象了。**无论你是在 Word 中打字还是在 Excel 中敲数据，上下左右移动配上前删后删绝对能让你那疲于在键盘和鼠标之间奔命的右手（或左手）得到解放。**

```
; 类Vim移动操作
CapsLock & h::Send, {Left}      ; 左移
CapsLock & j::Send, {Down}      ; 下移
CapsLock & k::Send, {Up}        ; 上移
CapsLock & l::Send, {Right}     ; 右移
CapsLock & b::Send, ^{Left}     ; 左移一个单词
CapsLock & w::Send, ^{Right}    ; 右移一个单词
CapsLock & e::Send, ^{Right}    ; 右移一个单词
CapsLock & i::Send, {Home}      ; 行首
CapsLock & a::Send, {End}       ; 行尾

; 其它移动操作
CapsLock & [::Send, {PgUp}      ; 上翻页
CapsLock & ]::Send, {PgDn}      ; 下翻页
CapsLock & `;::Send, {WheelUp}  ; 鼠标滚轮向上
CapsLock & '::Send, {WheelDown} ; 鼠标滚轮向下

; 类Vim编辑操作
CapsLock & u::Send, ^z                  ; 撤消
CapsLock & r::Send, ^y                  ; 重做
CapsLock & o::Send, {End}{Enter}        ; 下方插入一行
CapsLock & d::Send, {Home}+{End}^x{Del} ; 剪切整行
CapsLock & y::Send, {Home}+{End}^c{End} ; 复制整行
CapsLock & p::Send, {End}{Enter}^v      ; 粘贴到下一行

; 其它编辑操作
CapsLock & ,::Send, {Backspace}         ; 删除前面的字符
CapsLock & .::Send, {Delete}            ; 删除后面的字符
CapsLock & Backspace::Send, +{Home}^x   ; 剪切至行首
CapsLock & Delete::Send, +{End}^x       ; 剪切至行尾
```

第三部分为鼠标功能代替，以 Ctrl+Win 为基础的组合键。为啥要代替鼠标？其实就是想在懒得动的时候有个可实现的操作吧，该用鼠标还是得用鼠标。Ctrl+Win+H/J/K/L 可以让鼠标动起来，是不是有点熟悉？又出现了 HJKL。步子太小等不及？Ctrl+Win+Alt+H/J/K/L，让移动更快一点，而且这三个组合键按起来也挺顺手。鼠标左击放在空格上，右击放在 /? 键上，顺便也把滚轮的上下左右也定义了，跟前面重复了一下，重复就重复，不想记太多，怎么按都能使。

```
;鼠标点击
^#Space::Send, {LButton}                ; 鼠标左键
^#!Space::Send, {LButton}               ; 鼠标左键
^#/::Send, {RButton}                    ; 鼠标右键
^#!/::Send, {RButton}                   ; 鼠标右键

^#`;::Send, {WheelUp}                   ; 滚轮向上
^#!`;::Send, {WheelUp}                  ; 滚轮向上
^#'::Send, {WheelDown}                  ; 滚轮向下
^#!'::Send, {WheelDown}                 ; 滚轮向下
^#,::Send, {WheelLeft}                  ; 滚轮向左
^#!,::Send, {WheelLeft}                 ; 滚轮向左
^#.::Send, {WheelRight}                 ; 滚轮向右
^#!.::Send, {WheelRight}                ; 滚轮向右

;光标移动
^#h::MouseMove, -10, 0, 0,R             ; 光标向左
^#j::MouseMove, 0, 10, 0,R              ; 光标向下
^#k::MouseMove, 0, -10, 0,R             ; 光标向上
^#l::MouseMove, 10, 0, 0,R              ; 光标向右

^#!h::MouseMove, -100, 0, 0,R           ; 光标向左，快速
^#!j::MouseMove, 0, 100, 0,R            ; 光标向下，快速
^#!k::MouseMove, 0, -100, 0,R           ; 光标向上，快速
^#!l::MouseMove, 100, 0, 0,R            ; 光标向右，快速
```

第四部分，吃喝玩乐？咳咳，听歌的时候不想动鼠标切歌，Shift+Alt+N/P 吧，不想听了，Shift+Alt + 空格暂停吧。声音太小？Shift+Alt+U（you can you up）。太大？Shift+Alt+D（no can no didi）。重要人物出现？Shift+Alt+M 一键静音。浏览器操作自己看着玩吧，一键打开网页肯定是很实用的，比如我最喜欢的 C++ Reference（邪魅一笑）。

```
;多媒体
!+n::Send, {Media_Next}
!+p::Send, {Media_Prev}
!+Space::Send, {Media_Play_Pause}

;音量键
!+u::Send, {Volume_Up}
!+d::Send, {Volume_Down}
!+m::Send, {Volume_Mute}

;浏览器
!+h::Send, {Browser_Home}
!+,::Send, {Browser_Back}
!+.::Send, {Browser_Forward}
!+r::Send, {Browser_Refresh}

;打开网页
!+=::Run, <https://en.cppreference.com/w/>
```

好了，先分享到这儿吧，有空再聊

如果觉得对你有用，请戳下面的 “在看”；如果觉得对别人有用，欢迎分享；如果喜欢本公众号，请加关注哦！

![](https://mmbiz.qpic.cn/mmbiz_jpg/2PvdnjYMmbx1FMiaicBuJ9XH4eDCIoXLJzZLMtVRicyhXfaGiaCibHjmDHIviajZAACZY5iaO6ECe5yxeVo8GP0YoytwA/640?wx_fmt=jpeg)

米博宇

您的赞赏是对作者的肯定哦😊

**长按识别前往小程序** https://mp.weixin.qq.com/s?__biz=Mzg5NTE3NDk2Ng==&mid=2247483777&idx=1&sn=1ea04534ba10b8f2c8d1e40ffc16c1d6&scene=19#wechat_redirect
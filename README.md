﻿# 简体中文论文查重系统

#### 写在前面
2017年开始，学校严抓研究生论文抄袭，遂希望开发一套校内论文查重系统，将新的论文和校内历年的研究生毕业论文以及同级的学生论文进行查重作为万方和CNKI查重的补充。本项目于在2017年完成了第一版并投入使用，后经一年多的稳定运行和维护升级，于2018年开源。项目自上线稳定运行至今已产出5000+篇查重报告。目前本项目以维护为主进行常规更新和BUG修复。  

#### 相关SDK
目前团队开发了一套新的查重SDK（JAVA版）  
项目链接：https://dreamspark.com.cn/blog/?id=1  
使用示例：https://github.com/tianlian0/duplicate-check-sample  
SDK相较于本项目更新频率较高也拥有一些更新更高级的特性。目前已有多个商用查重系统和查重网站基于此SDK开发上线。欢迎大家试用、反馈，希望它能帮助大家开发出自己的查重系统。另，C++、C#版在计划开发中。  

#### 安装使用教程
1、clone源代码  
2、使用vs打开、编译（vs需安装.NET开发组件）  
3、运行paper_checking.exe文件即可  
兼容性要求：  
windows 7及以上，vs2017及以上，需安装vc2015运行库和.NET Framework4.6。其它版本需自行测试。  
报错排除：  
1、如果运行时报pdfbox相关错误可将引用中的pdfbox-app.dll删掉重新添加，然后尝试重新编译。  
2、如果IKVM或Spire报错，可能是因为项目刚拉下来相关第三方包还未下载完毕，可等待vs自动下载或打开NuGet手动下载缺少的包。  
3、32位版本操作系统请将paper_check32.dll文件重命名为paper_check.dll来替换原文件，否则会报paper_check.dll相关错误。  
4、如果程序执行到一半闪退，一般是由于比对库中的文件，文件名中包含特殊字符（不可见字符、特殊的空格、奇怪符号等），请检查文件名。  
5、为什么查重的结果永远是0？因为这个系统是使用本地比对库进行查重的，查重之前需要先添加本地比对库，不添加比对库就查重结果肯定是0。各位正在写论文或者作业的同学们，想给自己的论文查重请自行去各大查重平台购买，这个系统不适用没有本地比对库的用户。  

#### 项目介绍与功能说明
该系统目前支持对简体中文的文件进行横向查重和纵向查重。  
两个核心功能点说明如下:  
1、纵向查重  
选择一批待查文件后，将该批文件和比对库中的文件比对。通常用以检查该批文件有没有复制比对库中的文本。  
2、横向查重  
选择一批待查文件后，在该批次文件之间进行比对，用以检查该批次文件有没有互相复制的情况。这个功能是目前主流查重平台（万方、CNKI等）支持度较差的。  

软件中涉及的其它功能点及名词说明如下：  
1、添加到比对库  
将文件添加到比对库中。这个功能是服务于纵向查重的，将文件通过这个功能添加到比对库中，在使用纵向查重功能的就可与比对库中的文件进行比对。该功能使用时只需要选中存放文件的文件夹，然后点击“添加到比对库”按钮，按钮会变灰，等待按钮恢复正常即添加完毕。该功能只会扫描文件夹中的pdf、doc、docx、txt文件，其它类型的文件将被忽略。该过程需要耗费一定时间。  
2、查重阈值的设置  
该值的设置决定了待查文件连续多少个字与其它文件相同即判定为抄袭。推荐值为10至16，用户亦可根据实际情况自行设定为1至99之间的任意整数值。  
3、保存查重报告的文件夹  
选择一个文件夹，待查重完毕后查重报告将会输出到这个文件夹中。  
4、生成统计表  
勾选此复选框，查重结束后将会生成csv格式的统计表。复选框默认勾选。  
5、中断恢复  
选中复选框，软件将不会清除上一次查重的结果继续查重。该复选框适用于应用程序中途意外退出，想从退出时的进度继续查重。  
6、关键词过滤  
将一些可能影响重复率的关键词添加到关键词过滤功能中，在查重时会将待查文本中出现的关键词全部删除，以避免它们对重复率的影响。这类关键词通常可能为学校名、机构名等。  
7、查重进程数  
该值的设置影响查重速度，默认为当前机器CPU逻辑核心数减2。  
8、格式转换线程数  
文件在查重前会进行格式转换，该值的设置影响文件格式转换的速度，默认为当前机器CPU逻辑核心数减2。  

几个注意事项：  
①非标准格式的文件查重时可能会由于转换失败导致查重失败。查重失败的文件，会在查重结束后弹窗提示。②如果待查文件在比对库中存在，且文件名不相同，则可能会导致查重重复率高于90%的情况（相当于是两篇相同的文件进行比对，至于为什么不是100%后续有说明）。  

#### 查重整体流程说明
查重的原理：  
每两篇文件之间比对连续相同的字符串，超过查重阈值即认为这些字是重复的。但是，如果单篇文本重复率在0.25%以下或重复字数在30字以下，将不判定为重复。同时，如果一篇文本从其它文本复制了某句话很多次，只认为其中的一次为重复。  

整体查重的流程如下：  
将文件转换为txt->将txt内容格式化->调用paper_check.dll进行重复比对->生成查重报告和统计表  
分步说明：  
文件格式转换：使用pdfbox和spire word free。  
文本格式化：去掉文本中的摘要、目录、参考文献及非中文字符。  
调用paper_check.dll进行重复比对：paper_check.dll是一个独立模块，目前没有开源，但可从外部调用使用。  
生成查重报告：将重复的部分标红导出rtf。  

#### 引用的库
1、IKVM  
2、Spire.Doc Free  
3、pdfbox  

#### 已知bug
极少量正常的pdf、word文件转换失败，如您发现此类文件，希望您通过issuse进行反馈并提供下载链接，十分感谢。  
比对库文件和待查文件的文件名中如包含一些奇怪的字符（如一些控制字符、不可见字符、奇怪的空格等）可能导致查重异常退出，目前建议规范文件名后再使用本系统。  

#### 其它说明
1、2018年本项目的web版开发完成（地址：https://gitee.com/mark_liu_admin/PaperCheckSys ），后该版本放弃维护，已不提供任何技术支持，感兴趣的小伙伴可以自行研究我们之前的代码。  
2、现在我们已经开发了一套全新的支持多语言的web版查重系统，不过暂未开源，如有相关需求可以通过[商业合作]联系进行试用/定制开发。该系统中使用的核心查重模块已以SDK的形式开放使用（https://dreamspark.com.cn/blog/?id=1 ）。  
3、该开源版本为一个简化版本，拥有查重所需的所有核心功能并输出一个较为简易的查重报告，有其它定制需求见[商业合作]。  

#### 项目截图
![image](https://github.com/tianlian0/paper_checking_system/blob/master/images/pic1.png)  
![image](https://github.com/tianlian0/paper_checking_system/blob/master/images/pic2.png)  
项目运行演示视频：链接：https://pan.baidu.com/s/1VM9g4CT4nAwlZXOHePkoXQ 提取码：t059  
PS：①截图部分内容涉及隐私，已马赛克处理。②截图、视频和本项目略有不同，一是因为视频和截图使用的不是最新版，二是因为[其它说明]第3条。  

#### 欢迎打赏
![image](https://github.com/tianlian0/paper_checking_system/blob/master/images/shang.png)  

#### 商业合作
本项目遵守AGPL3.0协议，同时本项目已申请三项软件著作权（证书号：3846389，4486326等）  
本项目可以提供c#/java/c++版本的定制开发或技术支持，也可定制web版或客户端版系统。欢迎各企业、高校、机构合作。商业合作专用微信/QQ：654062779  
应用场景：高校论文查重、企业合同查重、企业内部文档查重、学生作业查重等  

## Rabin
- AV：病毒专杀；
- IPS：入侵防护；（木马之类的）
- FW：企业网的安全大门；（只让符合规则的流量进入企业网）
- UTM：集成了fw, ips, av功能；
- 高级什么什么，是针对普通来说的，如AET（相对普通的逃逸技术）, APT（针对普通威胁）
	- 那么普通的威胁有哪些呢？

bring your own device: BYOD移动办公；



## programming vs scripting languages

> the perfect balance is using scripts and programs together. 
>> when you need to automate tasks, use a scripting language.   
>> when you need to make something fast, use programing language.

### programming language
- programs are a lot faster than scripts because they get compiled. 
- lack the flexibility offered by scripting languages.

### scripting language
- scripting languages (like Bash, Python, Ruby...) are mainly designed to automate certain tasks (especially Bash). 
- they are slow. scripts are interpreted, meaning that the program executes command per command, rather than throwing all commands together and translating it to machine code.



-----------i dont know
- 非法外联
- 网络钓鱼攻击的形式多种多样。
	- 如冒充银行，消费后发票下载，社交媒体中的链接之类的让你点击某链接、下载某文档等；

- 鱼叉式的网络钓鱼会更具体地针对个人。
	- 如攻击者会发送假装来自合作伙伴、上司或您的 IT 部门的电子邮件，要求你下载某个文件或者点击链接之类的。

C&C服务器的全称是Command and Control Server，即“命令及控制服务器”。随着恶意木马产业的发展，很多木马早已摆脱了过去“单打独斗”的作战方式，而是通过网络相互关联起来，通过指挥大量受到感染的计算机共同行动，进而发挥出协同效果。这样既可以集中起来同时对某个目标进行打击，也可以互相分散各自所承受的风险。这其中，进行指挥的关键节点便是C&C服务器。腾讯反病毒实验室旗下的哈勃分析系统，发布了《威胁情报态势报告——C&C 服务器篇》(以下简称《报告》)。





## DDOS

[tech-ddos refer](techDDos.md)


## OWASP
[tech-owasp refer](techOWASP.md)

针对这些技术有个专门产品  -- WAF 


## APT
[tech-apt refer](techAPT.md)




## sandbox 沙箱、沙盒
- “Sandbox”技術與主動防禦技 術原理截然不同。主動防禦是發現程序有可疑行為時立即攔截，終止運行；“Sandbox”技術是發現可疑行為讓程序繼續運行，當發現的確是病毒時才終止。
- 原理：一种类似于影子系统的，比带有宿主的虚拟机更深层的系统内核级技术。它可以接管病毒调用接口或函数的行为。并会在确认病毒行为后实行回滚机制，让系统复原。


- 功能
	- 提供‘虚拟执行环境’，让可疑文件在这个环境里使劲的折腾；
	


## honeypot
- 定义：蜜罐是一类安全资源，其价值就在于被探测、被攻击及被攻陷。

- 部署
	- 蜜网：由多台蜜罐主机组成；
	- 部署在DMZ区（公共服务区）；




## china hacker history

人物
肖力（安全攻防领域资深专家）、吴瀚清（《白帽子讲安全》作者，江湖上声名显赫的道哥）、魏兴国（网络安全领域知名专家，人称“云舒”）。以及知名的技术专家潘爱民（互联网底层技术专家）、刘嘉伟（知名架构师）


### 道哥 - 吴翰清

> 《中国黑客传说》  
> 幻影论坛  
> 阿里云云盾  

### other

http://www.cnblogs.com/milantgh/category/593187.html

#### https://www.zhihu.com/question/28704321

习科论坛的大牛：Yoco Smart（禽兽磊），newbie(牛逼哥，Linux牛),等等，牛人太多不一一列举；腾讯：tombkeeper,coolq,riusksk,blackwhite(还有没有人记得他的马甲是wordexp......)等。Keen Team:全是大牛。华为：我只记得一个keji.PanguTeam:全是大牛。启明星辰：村长，老王，村雨等。知道创宇：icbm，watercloud,余弦等。阿里巴巴：道哥，云舒，Vessial,Neeao等。新浪：陈洋，鬼仔。瀚海源：instruder,flashsky,alert7等。绿盟科技：牛人太多。百度：xi4oyu最突出吧。数字娱乐有限公司：赵武，古河，redrain等。gov&mil:你懂的，不解释～



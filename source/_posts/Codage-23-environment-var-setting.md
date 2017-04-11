Windows下的环境变量设置，不同path之间不需要空格！
不需要空格！
不需要空格！

<!-- more -->

添加空格以后，cmd无法识别，只有PowerShell可以。
这就是为什么设定环境变量后，要在程序安装路径以外的路径下调用该程序，cmd无法识别的原因。

错误写法: `c:\path1; c:\Maven\bin\; c:\path2\`
正确写法: `c:\path1;c:\Maven\bin\;c:\path2\`
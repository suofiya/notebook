# 如何用命令查看外网IP地址


1. 输入“telnet smtp.gmail.com 587”，回车；

   注意，若不能运行Telnet客户端，参阅百度经验：telnet不是内部或外部命令怎么办(Merlin67)
![image](http://f.hiphotos.baidu.com/exp/w=500/sign=9e4cf968f536afc30e0c3f658318eb85/a8014c086e061d959778839079f40ad162d9ca58.jpg)   
   
2. 输入“STARTTLS”，回车；如图：
![image](http://d.hiphotos.baidu.com/exp/w=500/sign=101a6a5cb54543a9f51bfacc2e168a7b/7af40ad162d9f2d3f42f9047abec8a136327cc58.jpg)

3. 输入“EHLO”，回车；如图：
![image](http://f.hiphotos.baidu.com/exp/w=500/sign=1ce41792324e251fe2f7e4f89787c9c2/5366d0160924ab18cafb549f37fae6cd7a890bac.jpg)

4. 即可查看到自己的外网IP地址；如图：
![image](http://e.hiphotos.baidu.com/exp/w=500/sign=adc7ffa66509c93d07f20ef7af3cf8bb/e7cd7b899e510fb36bf4c23fdb33c895d0430cac.jpg)

5. 注意：若连接失败，或不能成功查看到自己的公网IP地址，尝试重新执行命令即可。
	 退出telnet： QUIT  
![image](http://g.hiphotos.baidu.com/exp/w=500/sign=856077bc69600c33f079dec82a4d5134/5882b2b7d0a20cf4dfae4a7774094b36acaf9958.jpg)

 
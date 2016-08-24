## linux下configure，make，make install的意义

tar.gz、tar.bz2的是源代码包，需要编译之后才能安装，在编译过程中你可以指定各种参数以适应你的系统需求，比如安装位置，优化参数，要哪些功能不要哪些功能等等。
这类源代码包需要解压后（tar.gz的用 tar zxvf 解压，tar.bz2的用 tar jxvf 解压），进入解压目录，一般都有一个 INSTALL 的文本文件，里面一般都是安装的详细说明，可以用vi、nano、pico或X下面的文本编辑器（如gedit,gvim,kedit等）打开查看，安装一般就是三个步骤：

1. **configure**，这一步一般用来生成 Makefile，为下一步的编译做准备，你可以通过在 configure 后加上参数来对安装进行控制，比如
代码:
 
		./configure --prefix=/usr

   上面的意思是将该 __软件安装在 /usr 下面，执行文件就会安装在 /usr/bin （而不是默认的 /usr/local/bin),资源文件就会安装在 /usr/share（而不是默认的/usr/local/share) __ .同时一些软件的配置文件你可以通过指定 --sys-config= 参数进行设定。有一些软件还可以加上 --with、--enable、--without、--disable 等等参数对编译加以控制，你可以通过允许 ./configure --help 察看详细的说明帮助。


2. **make** ，这一步就是编译，大多数的源代码包都经过这一步进行编译（当然有些perl或python编写的软件需要调用perl或python来进行编译）。如果在 make 过程中出现 error ，你就要记下错误代码（注意不仅仅是最后一行），然后你可以向开发者提交 bugreport（一般在 INSTALL 里有提交地址），或者你的系统少了一些依赖库等，这些需要自己仔细研究错误代码。


3. **make install**，这条命令来进行安装（当然有些软件需要先运行 make check 或 make test 来进行一些测试），这一步一般需要你有 root 权限（因为要向系统写入文件）。


安装完毕后你就可以删除解压目录了。采用源代码编译方式来安装软件是 Linux 系统下最常见的安装软件方法，而且这种方法使你可以更加自由地控制安装细节，所以提倡大家多使用该方法安装软件。

PS:对于 bin 类型的安装文件，一般给该文件加上可执行权限，再运行之即可,如：
代码:
 
	chmod u+x example.bin	
	./example.bin
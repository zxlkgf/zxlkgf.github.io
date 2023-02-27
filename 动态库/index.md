# 动态库

# 01动态库的制作
命名规则  
Linux:libxxx.so  
lib:前缀（固定）  
xxx:库的名字 自己起  
.so:后缀（固定）  
在Linux下是一个可执行文件  
Windows:libxxx.dll  

# 02动态库的制作:  
gcc得到.o文件，得到和位置无关的代码  
gcc -c -fpic/-fPIC a.c b.c  
gcc得到动态库  
gcc -shared a.o b.o -o libcalc.so 

# 03工作原理
## 静态库:
gcc进行连接时，会把静态库中的代码打包到可执行程序中  
## 动态库:
gcc进行连接时，动态库的代码不会被打包到可执行程序中  
程序启动之后，动态库会被动态加载到内存中，通过ldd(list dynamic dependencies)命令检查动态库依赖关系  

## 04如何定位共享库文件呢？  
当系统加载可执行代码时，能够知道其所依赖的库的名字，但是还需要知道绝对路径，此时就需要
系统的动态载入器获取该绝对路径。对于elf格式的可执行程序，是由ld-linux.so来完成的，
它先后搜索elf文件的 DT_RPATH段-->环境变量LD_LIBRARY_PATH-->/etc/ld.so.cache文件列表
-->/lib/./usr/lib目录找到库文件后将其载入到内存

## 05解决动态库加载失败的问题
只需要在上述的环境变量LD_LIBRARY_PATH或者/etc/ld.so.cache文件列表
或者/lib/./usr/lib目录内，添加需要使用的动态库绝对路径即可

### 1.在PATH内添加LD_LIBRARY_PATH即可
```shell
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:[动态库的绝对路径]
```
但是由于配置的环境变量是临时的，终端关掉之后再打开就会失效，需要按照用户级别或者root级别配置


### 2.用户级别配置  
在Home下使用vim .bashrc 在最底下添加 export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:[动态库的绝对路径]  
然后输入 source .bashrc或者 . .bashrc

### 3.系统级别的配置
sudo vim /etc/profile  
在最后一行插入export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:[动态库的绝对路径]  
记得刷新profile 


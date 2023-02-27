# GDB多线程调试

# GDB多进程调试
使用GDB调试的时候，GDB默认只能跟踪一个进程，可以再fork函数调用之前，通过指令设置GDB调试工具跟踪父进程或者跟踪子进程，默认跟踪父进程  

设置调试父进程或者子进程:  
set follow-fork--mode [parent (默认)|child]

设置调试模式:set detach-on-fork [on|off]
默认为on,表示调试当前进程的时候，其他的进程可以运行，如果为off，调试当前进程的时候，其他进程会被GDB挂起  

查看调试进程:info inferiors  
切换当前调试进程: inferior id  
使进程脱离GDB调试:detach inferiors id  


# 闪退bug修复

- <http://www.educity.cn/wenda/418906.html>

- SIGABRT 一般是过度release 或者 发送 unrecogized selector导致。
- EXC_BAD_ACCESS 是访问已被释放的内存导致。
　Application received signal SIGSEGV
　Application received signal SIGBUS
　Application received signal SIGABRT
- IGSEGV和SIGBUS一般是因为访问已被释放的内存或者调用不存在的方法导致的
- <http://blog.csdn.net/xyxjn/article/details/43310061>



- 1. SIGABRT是处于程序控制状态下的crash，SIGABRT引起的crash是因为系统发现了应用程序正在做一些系统不希望它去做的事情(Exception)。
　

- 2. EXC_BAD_ACCESS意味着你的程序在内存管理方面有bug。与SIGABRT不同，发生EXC_BAD_ACCESS错误时，在控制台里你不会得到一个错误的信息，但是你可以通过一些设置得到这些错误信息并进一步定位内存错误发生的位置。
- 3. EXC_BAD_ACCESS意味着你的程序在内存管理方面有bug。与SIGABRT不同，发生EXC_BAD_ACCESS错误时，在控制台里你不会得到一个错误的信息，但是你可以通过一些设置得到这些错误信息并进一步定位内存错误发生的位置。
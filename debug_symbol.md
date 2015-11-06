# Debug Symbol(调试符号)
## 问题
- 最近用XCode做了一个静态库，在自己电脑上别的App project中编译使用没有任何问题，但是传给别的同事使用在编译的时候就会出现类似于下面警告。

```
	warning: (i386) /UsersLibrary/Developer/Xcode/DerivedData/ProjectName-ebyadedaazwurqcvfzmyzzacvlbg/Build/Intermediates/ ProjectName.build/Debug-iphonesimulator/ProjectName.build/Objects-normal/i386/ClassName.o unable to open object file
```

	- 通过在Google里面搜索，终于弄明白了，通过在XCode里面将Generate Debug Symbol的值设为NO，重新编译一下生成静态库，这次编译出来的静态库再也不会产生已经警告了。这是为什么呢？

## 一、 Debug Symbol(调试符号)

- 因为借助符号调试程序可以将类似

    ```
	Thread 0 Crashed:
	0 libobjc.A.dylib 0×300c87ec 0×300bb000 + 55276
	1 MobileLines 0×00006434 0×1000 + 21556
	2 MobileLines 0×000064c2 0×1000 + 21698
	3 UIKit 0×30a740ac 0×30a54000 + 131244
	```
	
*的log信息转换成*
    
    ```
	Thread 0 Crashed:
	0 libobjc.A.dylib 0×300c87ec objc_msgSend + 20
	1 MobileLines 0×00006434 -[BoardView setSelectedPiece:] (BoardView.m:321)
	2 MobileLines 0×000064c2 -[BoardView touchesBegan:withEvent:] (BoardView.m:349)
	3 UIKit 0×30a740ac -[UIWindow sendEvent:] + 264
	```
	
- 主要是方便开发人员获取调试信息

## 二、DWARF
- DWARF是一种被众多编译器和调试器使用的用于支持源代码级别调试的调试文件格式。它满足了许多程序语言的需求，比如C,C++和Fortran，而且被设计成可拓展到其它语言。DWARF是平台独立的且适用于任何处理器任何操作系统。 DWARF广泛应用于Unix,Linux和其它操作系统,以及独立的环境中。
## 三、dSYM
- 为了避免进行stripping操作后调试符号的丢失，你可以使用dwarf-with-dsym选项. DWARF with dSYM 选项在标准的DWARF之外执行一个额外的步骤：创建一个单独的MyApp.app.dSYM文件，这个文件包含你的程序的所有调试符号(这个文件其实是一个包，可以通过右键->显示包内容进行查看)。事实上，DWARF with dSYM选项允许你对你进行单步调试而不管可执行程序是否被剥离了调试信息(stripped)。这是可能的，这是因为gdb将会在你的程序的目录下查找.dSYM文件。它不需要知道对象文件(object files)的名字或者路径。如果你不除去调试符号 (strip debugging symbols), 你可以使用.o或者.dSYM文件来调试。



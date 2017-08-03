# 为什么要使用Python的os模块方法而不是执行执行shell命令

------

**问题**

使用Python的库函数执行OS-specific的动机是什么，为什么不通过os.system()或subprocess.call()执行这些命令？
如：为什么使用os.chmod()而不是os.system("chmod ...")?


**方案**
1.它更快，os.system()和subprocess.call创建新的进程，对于简单的程序是不必要的。事实上，具有shell参数的os.system()和subprocess.call通常至少要创建两个新进程：第一个是shell，第二个是正在运行的命令(如果不是shell内置命令)

2.一些命令在单独的进程中是无用的，例如，运行os.spawn("cd dir/")，它将更改子进程的当前工作目录，而不是Python进程。需要使用os.chdir()

3.不必担心shell解释的特殊字符。os.chmod(path,mode)无论文件名是什么都可以工作，而os.spawn("chmod777 " + path)如果文件名类似于'; rm -rf ~',则会失败。如果使用没有shell参数的subprocess.call，则可以解决此问题。

4.不必担心以破折号开头的文件名。os.chmod("--quiet",mode)将更改名为 --quite的文件的权限，但os.spawn("chmod 777 --quiet")将会失败。因为--quiet被解释为参数，即使对于subprocess.call(["chmod","777","--quiet"])也是如此。

5.更少的cross-platform和cross-shell的关注，因为Python的 标准库应该为你处理。系统是否具有chmod命令，是否安装，是否支持所期望它支持的参数？os模块将尽可能地作为cross-platform，并且在不可能的情况下进行文档化。

6.如果正在运行的命令已输出所要的内容，则需要解析该命令，因为可能会忘记corner-cases(其中包含空格，制表符和换行符的文件名)不在乎便携性。

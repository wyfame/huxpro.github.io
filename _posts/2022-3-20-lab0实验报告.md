# lab0实验报告

## 实验思考题

### Thinking 0.1

- 第一次新建了README.txt文件，还未执行`git add`命令，文件暂存在工作区，还未提交到版本库，是untracked状态
- 第二次是提交后再做修改，需要再做提交，处于modified状态

------

### Thinking 0.2

- add the file : git add
- stage the file : git add
- commmit : git commit

------

### Thinking 0.3

> **深夜，小明在做操作系统实验。困意一阵阵袭来，小明睡倒在了键盘上。等到小明早上醒来的时候，他惊恐地发现，他把一个重要的代码文件printf.c删除掉了。苦恼的小明向你求助，你该怎样帮他把代码文件恢复呢？**

- 使用`git checkout -- printf.c` 恢复

此时只是删除了文件，还没有git add



> **正在小明苦恼的时候，小红主动请缨帮小明解决问题。小红很爽快地在键盘上敲下了git rm printf.c，这下事情更复杂了，现在你又该如何处理才能弥补小红的过错呢**

- 同上使用`git checkout -- printf.c` 恢复



> **处理完代码文件，你正打算去找小明说他的文件已经恢复了，但突然发现小明的仓库里有一个叫Tucao.txt，你好奇地打开一看，发现是吐槽操作系统实验的，且该文件已经被添加到暂存区了，面对这样的情况，你该如何设置才能使Tucao.txt在不从工作区删除的情况下不会被git commit指令提交到版本库？**

- 使用`git rm --cached Tucao.txt`从暂存区清除

`git rm --cached<file>` 能从暂存区中删去一些不想跟踪的文件

------

### Thinking 0.4

时光机指令`git reset --hard`用于版本回退

- 此指令会彻底回退到某个版本，本地的源码也会变为回退版本的内容。
- `git reset --hard HEAD^`，HEAD后^的个数表示回退几个版本的内容，也可以用`git reset --hard <hash_code>`进行版本回退，<hash_code>可以通过`git log` 查看。
- 若只回退commit信息，可以使用`git reset --soft`

慎用撤销指令！

------

### Thinking 0.5

> **1.克隆时所有分支均被克隆，但只有HEAD指向的分支被检出。**

- 正确。

举例说明：远程仓库目前有main和dev两个分支，克隆到本地如下：

![image-20220320143001578](C:\Users\WYF\AppData\Roaming\Typora\typora-user-images\image-20220320143001578.png)

可见刚克隆后所有分支均被克隆，默认分支为main,而dev分支被隐藏。

当`git checkout dev`后，即将本地dev分支与远程分支同步，`git branch`即可显示所有分支



> 2.**克隆出的工作区中执行 git log、git status、git checkout、git commit等操作不会去访问远程版本库。**

- 正确。

正如做实验所见，在工作区执行的命令不会影响远程仓库，只有`git push`之后远程仓库才会作出响应（如返回实验result）



> **3.克隆时只有远程版本库HEAD指向的分支被克隆**

- 错误。详情见1




> 4.**克隆后工作区的默认分支处于master分支**

- 正确。

如1中图所见，克隆结束后cd进入，会默认进入到main/master分支，再执行`git branch -a`指令，除隐藏分支外，只有一个main/master分支。

------

### Thinking 0.6

> ```shell
> echo first
> echo second > output.txt
> echo third > output.txt
> echo forth >> output.txt
> ```

执行结果如下：
![image-20220320113442800](C:\Users\WYF\AppData\Roaming\Typora\typora-user-images\image-20220320113442800.png)

echo命令即在标准输出或文件中显示一行文本或字符串

">"可以改变命令的数据信道，使得">"前命令输出 的数据输出到">"后指定的文件中去。若">"后的文件不存在，则在当前目录下新建一个文件；若此文件存在，则会覆盖文件原有的内容。

">>"为重定向追加输出，将">>"前命令的输出追加到">>"后面的文件去，即在文件原有内容的下一行添加新的内容。

------

### Thinking 0.7

创建test文件：`vim test`编辑即可

运行test文件：

```shell
echo vim test > command
chmod +x test
./test > result
```

- command内容：

```
vim test
```

- result内容

```shell
Shell Start...
set a = 1
set b = 2
set c = a+b
c = 3
save c to ./file1
save b to ./file2
save a to ./file3
save file1 file2 file3 to file4
save file4 to ./result
3
2
1
```

test文件先给a,b,c分别复制为1，2，3。然后将值分别存入file3,file2,file1。最后将file1,file2,file3内容依次追加到file4中，输出便为3，2，1。

- echo指令大致有两个功能，一是在标准输出上显示文本；二是向一个文件中写入内容。

- test中`echo Shell Start...`等语句即在标准输出上直接输出文本
- test中`echo $c>file1`等语句则是第二个功能，向file1内写入c的值，因此他直接将内容写入文件，并没有在屏幕上看到“$c>file1”字样。
- `echo echo Shell Start` 和`echo 'echo Shell Start'无`区别，均为显示文本。
- `echo echo $c>file1` 是将echo $c（在本题中为echo 3)存入file1中 。（应该是echo命令的嵌套）
- `echo 'echo \$c>file1'`因为' '的存在，是将引号里的内容显示出来。

------

## 实验难点

1、上机实验中需要两次复制同一文件到另一目录中重命名，然后删除源文件。但是两次相同的cp操作只会执行一次。对此大致有两种解决办法：

一是把复制+删除操作合并为一次mv操作，在mv命令中同时进行重命名。

二是cp的时候直接进行重命名。（当时不知道有这种操作，也没有尝试）

2、对于文件编译操作，以及多次报错的*"<fibo.h> No such file* *or dirctory*"，解决如下：

- 理清文件从编译到执行的过程

  ![未命名文件(8)](C:\Users\WYF\Desktop\未命名文件(8).png)

- 显然第一步要求的编译需要处理include头文件，但对于一般的include目录，是不需要我们指令的，gcc回去系统里找；而此题中的*fibo.h*存在于另一个目录中，这时就需要用-I参数指定路径，才能完成编译。

------



## 体会与感想

本次实验难度较为友好，大约花费7小时左右。较多时间用在阅读指导书和查阅相关指令（如gcc, awk, sed等）的用法上。

- 初步掌握了命令行的基本操作，能够编写简单的脚本
- 在指令学习过程中，对于不同的参数或是选项，应手动执行输出一遍，这样能够更好的理解指令各参数的作用。
- 了解了git的基本操作，以及版本控制的必要性。

------

## 指导书反馈

- 对于git冲突解决部分的内容，一眼看上去比较难以理解（至少对我是这样），希望在指导书中可以给出一个测试方法去模拟git的冲突。例如在远程和本地同时修改一个文件，然后push时看到了冲突（此时看到具体现象才明白指导书图里的<<<<,=======,>>>>>>>>>这些东西是什么，不然我还以为真是乱码）。感觉这样亲自动手看到冲突更便于理解吧。


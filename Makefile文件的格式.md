### 1 概述
Makefile文件由一系列规则(rules)构成。每条规则的形式如下。    
\<target> : \<prerequisites>   
\[tab\]  \<commands>

上面第一行冒号前面的部分,叫做"目标"(target), 冒号后面的部分叫做"前置条件"(prerequisites);  
第二行必须由一个tab键起首,后面跟着"命令"(commands)."目标"是必需的,不可省略;  
"前置条件"和"命令"都是可选的,但是两者之中必须至少存在一个;  

每条规则就明确两件事: 构建目标的前置条件是什么,以及如何构建.  
下面就详细讲解,每条规则的这三个组成部分。

### 2 目标(target)
一个目标(target)就构成一条规则。目标通常是文件名,指明Make命令所要构建的对象,比如上文的 a.txt. 目标可以是一个文件名,也可以是多个文件名,之间用空格分隔.  
除了文件名,目标还可以是某个操作的名字,这称为"伪目标"(phony target).  
######
    clean:
      rm *.o
上面代码的目标是clean,它不是文件名,而是一个操作的名字,属于"伪目标",作用是删除对象文件。

######
    $make clean
但是,如果当前目录中,正好有一个文件叫做clean,那么这个命令不会执行。因为Make发现clean文件已经存在,就认为没有必要重新构建了,就不会执行指定的rm命令。

为了避免这种情况,可以明确声明clean是"伪目标",写法如下。
######
    .PHONY: clean
    clean:
        rm *.o temp
声明clean是"伪目标"之后,make就不会去检查是否存在一个叫做clean的文件,而是每次运行都执行对应的命令。像.PHONY这样的内置目标名还有不少,可以查看手册。

如果Make命令运行时没有指定目标,默认会执行Makefile文件的第一个目标。
##### 
    $ make
上面代码执行Makefile文件的第一个目标。

### 3 前置条件(prerequisites)
前置条件通常是一组文件名,之间用空格分隔。它指定了"目标"是否重新构建的判断标准：只要有一个前置文件不存在,或者有过更新(前置文件的last-modification时间戳比目标的时间戳新),"目标"就需要重新构建。
#####
    result.txt: source.txt
        cp source.txt result.txt
上面代码中,构建result.txt的前置条件是source.txt。如果当前目录中,source.txt已经存在,那么make result.txt可以正常运行,否则必须再写一条规则,来生成source.txt。
#####
    source.txt:
        echo "this is the source" > source.txt
上面代码中,source.txt后面没有前置条件,就意味着它跟其他文件都无关,只要这个文件还不存在,每次调用make source.txt,它都会生成。
##### 
    $ make result.txt
    $ make result.txt
上面命令连续执行两次make result.txt。第一次执行会先新建source.txt,然后再新建result.txt。第二次执行,Make发现source.txt没有变动(时间戳晚于result.txt),就不会执行任何操作,result.txt 也不会重新生成。

如果需要生成多个文件,往往采用下面的写法。
#####
    source: file1 file2 file3
上面代码中,source是一个伪目标,只有三个前置文件,没有任何对应的命令。
#####
    $ make source
执行make source命令后,就会一次性生成 file1,file2,file3 三个文件。这比下面的写法要方便很多。
#####
    $ make file1
    $ make file2
    $ make file3

### 4 命令(commands)
命令(commands)表示如何更新目标文件,由一行或多行的Shell命令组成。它是构建"目标"的具体指令,它的运行结果通常就是生成目标文件。

***每行命令之前必须有一个tab键。如果想用其他键,可以用内置变量.RECIPEPREFIX声明.***
#####
    .RECIPEPREFIX = >
    all:
    > echo Hello, world
上面代码用.RECIPEPREFIX指定,大于号(>)替代tab键。所以,每一行命令的起首变成了大于号,而不是tab键。

***需要注意的是,每行命令在一个单独的shell中执行。这些Shell之间没有继承关系.***
#####
    var-lost:
        export foo=bar
        echo "foo=[$$foo]"
上面代码执行后(make var-lost),取不到foo的值。因为两行命令在两个不同的进程执行。一个解决办法是将两行命令写在一行,中间用分号分隔。
#####
    var-kept:
        export foo=bar; echo "foo=[$$foo]"
另一个解决办法是在换行符前加反斜杠转义。
#####
    var-kept:
        export foo=bar; \
        echo "foo=[$$foo]"
最后一个方法是加上.ONESHELL:命令。
#####
    .ONESHELL:
    var-kept:
        export foo=bar; 
        echo "foo=[$$foo]"
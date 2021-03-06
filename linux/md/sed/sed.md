## 1.sed 概述

主要功能：实现文件内容的新增、修改、删除、替换、截取（或者说查找）

## 2.功能介绍

### 1.范围查找

#### 1.文件示例 

1. a.txt

```bash
111
222
333
333
444
555
```

2. b.txt

```bash
111
222
333
333
111
bbb
333
444
555
```

3. c.txt

```bash
111
222
333
aaa
333
111
bbb
333
444
555
```

4. business.log

git地址 : https://github.com/yimiqidage/markdown-note/tree/master/linux/data/sed

5. d.txt

```bash
111
222
333
/home/ttshixi/
```



#### 2.基本语法

```bash
cat -n 文件名 | sed -n -e '/查找条件-开始行/,/查找条件-结束行/p' 
## 或者简洁写法：
sed -n '/查找条件-开始行/,/查找条件-结束行/p' 文件名
```

> -n 代表安静模式，也就是只显示处理后的行；（可以试一下去掉 -n 的效果，会打印文件的所有行，额外再加上符合文本查找条件的行。
>
> -e 指定sed编辑命令。当只进行一次处理时，-e 可以省略
>
> -p 代表打印，一般是与 -n 一起使用；（可以试一下，去掉-p后，文件内容不输出了===>此处会提示语法错误）
>
> '' 单引号中间的内容，就是需要执行的脚本
>
> /**查找条件-开始行**/,/**查找条件-结束行**/  第1个双斜杠中间的内容，代表开始行的查找条件；第2个双斜杠中间的内容，代表查找结束行的查找条件；需要注意的是，查找条件，支持**正则表达式**。

####  3.示例

1.查找a.txt文件内，含有1和含有3之间的行，并打印

```bash
cat -n a.txt | sed -n '/1/,/3/p'   ## 执行命令，添加cat查找，是为了方便显示文件所在的行，便于解释。
=======> 输出结果
     1	111
     2	222
     3	333
```

> 总结：
>
> 查找开始行：第1行中含有字符串 1 ，即符合条件；从下一行开始查找符合结束行的条件；
>
> 查找结束行：第2行不含字符串 3 ，不符合条件，继续处理下一行；
>
> ​					   第3行中含有字符串 3 ，符合结束条件。第一轮查找结束。

> **注意点**：
>
> 无论是查找开始行和结束行，只要找到第一个匹配的，就结束。
>
> 如上面的示例中，如果不结束的话，应该额外增加1行显示：
>
>      4  333 ## 第4行也符合结束条件



2. 查找文件b.txt中，含有1和含有3之间的行，并打印

```bash
cat -n b.txt | sed -n '/1/,/3/p' 
=======> 输出结果
     1	111  ## 第1次循环开始
     2	222
     3	333  ## 第1次循环结束
     5	111  ## 第2次循环开始
     6	bbb
     7	333  ## 第2次循环结束
```

> 总结：
>
> 1. 查找数据时，满足开始行数据后，从下一行开始确认，是否满足结束行；（示例中第1行）
> 2. 如果下面的某一行满足结束行条件，则此次循环结束；（示例中第3行）
> 3. 从下一行继续开始下一次循环。同样是判断是否满足开始行，找到后，继续向下寻找，看是否满足结束行。（示例中第5行-第7行）
> 4. 现象就是：通过范围查找，找到的数据，是行数据不连续的数据。

3. 继续上一个需求：第一次查找结束后，不在继续向下查找

```bash
cat -n b.txt | sed -n -e '/1/,/3/p'  -e '/3/q'
======> 输出结果
     1	111
     2	222
     3	333   ## 输出结果中，没有了后面的数据。
```

> 注意点：
>
> 1. 每次对数据处理时，都需要添加 -e，但是如果只处理一次，可以省略。
> 2. q 代表退出。
> 3. /3/代表行数据中，含有字符串3，则退出。
> 4. 使用场景：大文件查找时非常必要（比如几十G的日志文件）

4. 查找文件c.txt中，含有1和含有3之间的行，并打印

```bash
cat -n c.txt | sed -n '/1/,/3/p' 
=======> 输出结果
     1	111  ## 第1次循环开始
     2	222
     3	333  ## 第1次循环结束
     6	111  ## 第2次循环开始
     7	bbb  
     8	333  ## 第2次循环结束
    10	555  ## ???? 这一行数据不符合开始条件行，为什么展示出来了？
             ## 原因：cat -n 的结果中，此条记录的行数为10，而sed命令是对前面cat命令的输出结果进行过滤的。符合开始条件的 1 的匹配条件。
```

5. 查找日志文件business.log中，异常堆栈信息

```bash
cat business.log | grep "exception" | more   ## 1、查找日志，找到异常信息，以及开始的时间点
cat business.log | sed -n '/2020-07-11 09:55:25,571/,/2020-07-11 09:55:26/p'  ## 2、执行查找命令
```

返回结果示例：

```bash
2020-07-11 09:55:25,571 ERROR [http-nio-8080-exec-1] com.ttshixi.git.logs.LogCreate : bussiness exception
java.lang.RuntimeException: bussiness exception
	at com.ttshixi.git.logs.LogCreate.testLog(LogCreate.java:28)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:190)
	....  ## 部分代码，忽略了无用代码
	2020-07-11 09:55:26,006 ERROR [http-nio-8080-exec-1] com.ttshixi.git.logs.LogCreate : bussiness exception
```



> 主要实现步骤：
>
> 1. 根据异常信息，找到日志对应的时间点，如文件示例为 **2020-07-11 09:55:25,571** ；
> 2. 将上面的时间点，做为范围查找的开始时间点；
> 3. 设置结束条件：一般找下一秒的日志，做为结束时间点就足够了。示例中设置的是：**2020-07-11 09:55:26** 并没有到毫秒。
> 4. 结束条件中，如果精确到毫秒，一定需要在毫秒中，有对应记录。否则会一直查找，直到文件结束。



优化后的代码：

```bash
cat business.log | sed -n -e  '/2020-07-11 09:55:25,571/,/2020-07-11 09:55:26/p'  -e '/2020-07-11 09:55:26/q'
```

> 优化点：
>
> 查询到需要的日志后，就不在继续查找。。。

### 2.内容替换

#### 1.基本语法

```bash
sed -i 's/需要被替换的字符串/替换后的字符串/g'
```

- s 代表要进行文件替换：sed -i '**s**/1/A/g'
- g 代表全局替换：sed -i 's/1/A/**g** '，如果只是想对某一次进行替换，改成g换成数字即可；
- 注意格式：/需要被替换的字符串/替换后字符串/ ，中间没有逗号分隔，跟范围查找有区别。
- -i 是直接进行文件替换；如果不想直接替换，只是想先预览效果，可以去掉 -i；
- / 是用来分隔s命令和参数的，可以改成其他字符。

#### 2.示例

d.txt内容示例：

```bash
111
222
333
/home/ttshixi/
```

1. 将文件b.txt中的1替换为A (练习基本的使用语法)

```bash
sed -i 's/1/A/g'  d.txt  ## linux 写法
sed -i -e 's/1/A/g' d.txt ## mac只能这样写（添加 -e )，linux两种均可。
sed -i '.bak' '1d' b.txt ## mac用法：'.bak' 指的是备份文件，追加的文件名称。mac上独有的用法。可以设置空字符串。
sed -i -e 's/1/A/2' d.txt ## 只对第2次出现的字符串1，进行替换；

=======> 执行后，查看文件内容：
AAA
222
333
```


2. 将文件d.txt中的（练习高级语法，使用自定义分隔符）

```bash
sed -i -e 's/\/ttshixi\//study\//g' d.txt  ## 常规写法：看着不够清晰
sed -i -e 's%/ttshixi/%/study/%g' d.txt  ## 分隔符修改为%，这样反斜杠就无需特殊处理了。
```

> 说明：
>
> s后面的字符串，就是分隔符，可以自定义






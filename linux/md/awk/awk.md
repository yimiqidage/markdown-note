### 1.awk常用示例

1.文件内容示例 score.txt

```bash
#姓名  年龄 分数（语文） 分数（数学）
zhangsan 14       70    90
lisi 15       75    50
wangwu 16       50     85
zhaoliu 17       90    83
```

#### 1.显示指定列

要求：显示每个人的语文分数

```bash
 awk '{print $3}' score.txt # $3代表分割后的第3列，默认分隔符为空格或者tab;  $0代表所有列
```

输出结果：

```bash
分数（语文）
70
75
50
90
```

扩充：

```b
awk '{print $3","$4}' score.txt # 显示语文、数学分数; 逗号（,）为分隔符
```



#### 2.根据条件显示

要求：显示大于60分的分数

```bash
awk ' $3>60 {print $3}' score.txt # 写法1，条件语句在{}外
awk ' {if($3>60) print $3}' score.txt # 写法2，条件语句在{}内
```

输出结果：

```b
分数（语文）
70
75
90
```

#### 3.做求和运算

要求：计算所有人的语文总分

```b
awk '{sum+=$3} END {print sum}' score.txt  # 简单写法
awk 'BEGIN {sum=0}  {sum+=$3} END {print sum}' score.txt #完整写法

====>输出结果：285
```

> 思考下，为什么第一列标题没有影响结果？
>
> ---- 标题列如果需要转为数字，需要使用 +0 运算

#### 4.做平均值计算

要求：求全班语文平均成绩

```
awk 'BEGIN {sum=0;count=0} {if(NR>1) {sum+=$3; count++}} END {print sum/count}' score.txt 
# 1.NR>1 是需要过滤掉第一行数据。NR指的是行号
# 2.如果不天剑NR>1的条件，会导致计算平均分数时，分母变大，导致结果不准确；
# 3. BEGIN {语句一} {语句二} END {语句三}

======> 输出结果：71.25
```

#### 5.计算大于60分的人的平均分数

```
awk 'BEGIN {sum=0;count=0} {if(NR>1 && $3>60) {sum+=$3; count++}} END {print sum/count}' score.txt
# 1.多个判断条件，可以使用连接符 && 进行操作；

======> 输出结果：78.3333
```

#### 6.求语文最高分和最低分

```
awk 'BEGIN {max=0;min=100} {if(NR>1){if($3>max) max=$3 ; if($4<min) min=$3}} END {print "max="max",min="min}'  score.txt 

# 1.{max=0;min=100} 是BEGIN语句块，只会执行一次；
# 2.if后面如果有多个执行语句，需要额外添加{} ；
	# if(NR>1) 是条件判断语句
	# {if($3>max) max=$3 ; if($4<min) min=$3} 是判断后的执行语句，如果不加{}，则if为真时，只会执行第一条语句，添加{}以后，就相当于java中的如下代码：
      if(NR>1){
        if($3>max) max=$3;
        if($4<min) min=$3;
      }
# 3.print "max="max",min="min 后面可以直接使用双引号拼接打印后的字符串。

======> 输出结果：max=90,min=75
```





 
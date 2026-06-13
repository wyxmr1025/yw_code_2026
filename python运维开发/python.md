# python

## python基础知识一：

##### 1、**python应用场景**：

python抓取系统信息（如cpu、磁盘、内存运行情况）、调用外部接口（企业微信告警、手机接口、短信接口、邮箱接口）、文件处理（增删改查等）、日志分析与展示（不仅可以对二维结构进行分析展示、对非二维结构数据进行分析展示）、拥有强大的数据可视化功能（如监控大屏）。

##### 2、**python解析器安装：**

>安装版本：python3.8，不能出现中文、空格以及特殊字符！！！！可以放置在D://Python/Python38目录下。

##### **3、代码格式化处理：**

使用Ctrl + Alt + L，实现快速格式化

##### **4 、python中的7中数据类型：**

数值类型、布尔类型、字符串类型、列表类型、元组类型、字典类型、集合类型

容器类型：字符串、列表、元组、集合、字典 => 1个变量同时保存多分数据

问题：如何判断一个变量到底是什么类型？

答：①使用type（变量名称），返回变量的数据类型 ②isinstance（变量名称,数据类型），只能返回True或者False（真的或者假的）

##### 5、**类型转换操作：**

int():将其他类型转换为整数

float():把其他类型转换为浮点型（小数）

str():把其他类型转换为字符串

int => float => str

##### 6、**f-string简写形式格式化输出（推荐）**

```
name = "孙悟空"
mobile = "18878569090"
print(f'姓名：{name},联系方式：{moblile}')


name = "大白菜"
price = 5.5198
print(f"今天{name}特价了，只要{price:.2f}/斤")
```

##### 7、**格式化输出中的转义字符（了解）**

在字符串中，如果出现了\t和\n,其代表的含义就是两个转义字符

```
\t:制表符，一个tab键（4个空格）的距离
\n:换行符
```

##### 8、**随机数**

```
# 导入模块
import random
# 生成随机数
random.randint(起始值，结束值)
注意：生成起始值 到结束值这个范围区间的一个随机整数
```

##### 案例：

```
import random

# 定义玩家和电脑
player = int(input("请输入您的出拳信息：（石头-1、剪刀-2、石头-3）："))
computer = random.randint(1, 3)
print(f'随机数{computer}')
# 玩家获胜
if player == 1 and computer == 2 or player == 2 and computer == 3 or player == 3 and computer == 1:
    print("恭喜你，你赢了")
# 电脑平局
elif computer == player:
    print("平局")
else:
    print("电脑获胜")
```

##### 9、**for循环基本语法**

for循环结构主要用于（==容器、序列类型=> 字符串、列表、元组、字典、集合，因为1个变量中通常保存了多个元素值==）类型数据的遍历（循环）操作

```
str1 = 'itheima'
for i in str1:
    print(i, end=' ')
```

pycharm工具继承了Debug工具：

* 下断点，从哪里开始调试  => Debug一般搭载if/while/for开头
* 启动Debug调试
* 使用step over 一步一步进行调试

##### 10、**range方法（函数）**

for循环主要场景都是==实现容器类型遍历==，for循环其实也可以结合range()函数，实现循环指定次数（循环5次、循环10次）。

python2 range()函数返回的是==列表==，而在python3中 range() 函数返回的是一个可迭代对象（类型是==对象，只能通过for循环遍历输出==），而不是列表类型，所以打印的时候不会打印列表

**主要作用**：用于生成一段连续的内容，从0-9

```
range(stop)
range(start,stop)
range(start,stop[, step])
range(5)  ==>0-4(到不了5)
range(1,5) ==> [0,1,2,3,4] 没有5
range(1,11,2) ==> 1、3、5、7、9
```

>range有个口诀：只顾头来尾不顾

##### 11、**for循环嵌套**

for循环嵌套：外层循环1次，内层循环N次

外层控制行数，内层控制列数

```
for i in range(5):
	for j in range(5):
		print('*', end='  ')
	# 外层控制行数，每循环一侧，应该输出一个换行符
	print()
```

##### 12、九九乘法表：

```
for i in range(1,10):
    for j in range(1,i+1):
        print(f'{j}x{i}={i*j}',end="\t")
    print()

1x1=1	
1(j)x2(i)=2	2x2=4	
1x3=3	2x3=6	3x3=9	
1x4=4	2x4=8	3x4=12	4x4=16	
1x5=5	2x5=10	3x5=15	4x5=20	5x5=25	
```

##### 13、**获取点号的索引下标，获取文件名称与文件后缀**:

```
 获取点好号下标，获取文件名称、后缀名
file = input("请输入文件名称：")
# 获取点号下标
index = file.rfind(".")
# 获取文件名
name = file[:index]  [:6]
# 获取后缀名
postfix = file[index:] [6:]
print(f'文件名称：{name}')
print(f'文件后缀名：{postfix}')

请输入文件名称：access.log
文件名称：access
文件后缀名：.log
```

##### 14、strip()方法

**作用**：==用于移除字符串开头和结尾的指定字符（默认为空格），他不会影响字符串中间你的字符串==

**基本语法**

```
字符串.strip()
或者
字符串.strip(指定字符)
```

> 注：---空格回车（横线）

## python基础知识二

### 一、列表类型（==重点==）

列表在运维开发中，主要负责==大批量数据==的存储！！！！

比如存储所有学生信息、存储所有商品信息、存储所有的日志文件信息等

#### 1、列表的相关操作

##### ※查操作

**查操作的相关方法：**

| 编号 | 函数  | 作用                                                      |
| ---- | ----- | --------------------------------------------------------- |
| 1    | len() | 返回列表中元素的个数                                      |
| 2    | in    | 判断指定数据在某个列表序列，如果在返回True，否则返回False |

**案例**

```
names = ["刘备","关羽","张飞"]
print(len(names))

black_ip= ["192.168.66.37","172.35.46.128"]
if "192.168.66.37" in black_ip:
    print("该IP已被加入黑名单")
else:
    print("该IP未被加入黑名单")
```



##### ※增操作

| 编号 | 函数      | 作用                 |
| ---- | --------- | -------------------- |
| 1    | apppend() | 增加指定数据到列表中 |

append(): 在列表的尾部追加元素

```
names = ["孙悟空","唐僧","猪八戒"]
#在列表的尾部追加一个元素“沙僧”
names.append("沙僧")
print(names)
```

> 注意：列表追加数据的时候，直接在原列表里面追加了指定数据，即修改了源列表，故列表为可变类型数据。



##### ※ 删操作

| 编号 | 函数     | 作用                             |
| ---- | -------- | -------------------------------- |
| 1    | del      | 根据索引移除指定列表元素         |
| 2    | remove() | 移除列表中某个数据的第一个匹配项 |

remove()方法

作用：删除匹配的元素

```
fruit = ["apple","banana","pineapple"]
fruit.remove("banana")
print(fruit)
```



##### ※改操作：

| 编号 | 函数                    | 作用                 |
| ---- | ----------------------- | -------------------- |
| 1    | 列表[索引] = 修改后的值 | 修改列表中的某个元素 |



##### ※翻转与排序操作

| 编号 | 函数      | 作用                                                       |
| ---- | --------- | ---------------------------------------------------------- |
| 1    | reverse() | 将数据序列进行倒叙排列（翻转效果）                         |
| 2    | sort()    | 对列表序列进行排序，也可以添加参数reverse=True实现从大到小 |

```
list1 = ['apple','banana','cherry']
# 实现翻转
# list1.reverse()
list1 = list1[::-1]
print(list1)

list1 = [35, 12, 18, 9, 23, 10]
# 实现从小到大
list1.sort()
print(list1)
# 实现从大到小
list1.sort(reverse=True)
print(list1)
```

#### 2、列表的循环遍历

> 什么是循环遍历：答循环遍历就是使用while或者for循环对列表中的每个数据进行打印输出

for循环（推荐）

```
list1 = ['貂蝉','大桥','小乔']
for i in list1:
	print(i)
	
# 点名系统
import random
names = ["张三","李四","王五","赵六","孙七","周八","吴九","郑十"]
index = random.randint(0,len(names)-1)
print(names[index])
```

### 二、元组类型

作用：和列表类似，都可以实现==大批量数据存储==，唯一区分，==元组一旦定义完成后，就不能修改与删除了，可以起到保护数据的目的==。

#### 1、为什么需要元组

思考：如果想要存储多个数据，但是这些数据是不能修改的数据，怎么做？

答：使用元组，元组可以存储多个数据且元组内的数据是不能修改的。

#### 2、元组定义

元组特点：定义元组使用小括号，且使用逗号隔开各个数据，数据可以是不同的数据类型。

```
# 多个数据元组
tuple1 = (10,20,30)

# 单个数据元组
tuple2 = (10,)
```

> 注意：如果定义的元组只有一个数据，那么这个数据后面也要添加逗号，否则数据类型为唯一的这个数据的数据类型。

### 三、字典类型（重点）

作用：字典类型比较适合某一事物的存储，比如一个人、一本书、一个产品、一个主机信息等等

> 我们说的这个事物往往是由多个属性组成，比如一个人（姓名、年龄、家庭住址），一个主机信息（IP、端口、账号、密码）

#### 1、为什么需要字典（dict）

思考一：比如我们要存储一个人的信息，姓名：Tom，年龄：20，性别：男，家庭住址：北京市昌平区，如何快速存储。

```
person = ['Tom',20,'男','北京市昌平区']
```

#### 2、python中字典（dict）的概念

特点：

* 符号为==大括号==（花括号） => {}
* 数据为==键值对==形式出现， => {key:value},key：键名，value：值，在同一分字典中，key必须是唯一（类似于索引下标）
* 各个键值对之间用逗号隔开。

![Snipaste_2026-04-29_10-18-58](pict\Snipaste_2026-04-29_10-18-58.png)

#### 3、字典的增操作（重点）

基本语法

```
字典名称[key] = value
注：如果key存在则修改这个key对应的值；如果不存在则新增此键值对
```

案例：定义一个空字典，然后添加name、age、以及address这样的3个key

```
person = {}
# 添加数据
person['name'] = "刘备"
person['age'] = 40
person['address'] = ''蜀中
print(person)
```

> 注意：列表、字典为可变类型

#### 4、字典的删操作

==del  字典名称[key]== :删除指定元素

````
person = {'name':'王大锤','age':28,'gender':'male','address':'北京海淀区'}
# 删除字典中的某个元素（gender）
del person['gender']
print(person)
````

#### 5、字典的改操作

基本语法：

```
字典名称[key] = value
注： 如果key存在则修改这个key对应的值；如果不存在则新增此键值对。
```

#### 6、字典的查操作

查询方法：使用具体的某个key查询数据，日过未找到，则直接报错

```
字典序列[key]
```

字典的相关查询方法

| 编号 | 函数     | 作用                                   |
| ---- | -------- | -------------------------------------- |
| 1    | keys()   | 一类列表返回一个字典所有的键           |
| 2    | values() | 以类列表返回字典中的所有值             |
| 3    | items()  | 以类列表返回可遍历的（键、值）元组数据 |

> 注：字典相关方法：==如keys、values、items不能直接使用，必须结合for循环遍历输出==！！！！

```
dict1 = {'name':'孙权','age':'23','address':'东吴'}
# 获取所有的keys
for k in dict1.keys():
    print(k)
# 获取所有的值
for v in dict1.values():
    print(v)
# 获取所有的键值对
for k,v in dict1.items():
    print(k,v)
```

### 四、集合类型

set集合，作用：==无序==且天生去重，特点：==去重==

#### 1、什么是集合

集合(set)是一个无序、不重复的元素序列、

* 无序
* 天生去重

#### 2、集合定义

在python中，我们可以使用一对花括号{}或者set()方法来定义集合，但是如果你定义的集合是一个空集合，则只能使用set()方法。

> 字典、集合都可以通过{}花括号，如果{}是key:value键值对，就代表是字典；如果是具体的值，就是集合。

![Snipaste_2026-04-29_10-48-54](pict\Snipaste_2026-04-29_10-48-54.png)

#### 3、集合操作的相关方法（增删查）

疑问：集合操作只有增删查而没有修改方法？

答：集合没有索引下标，也没有key，而且本身也是无序的，所以无法精准对集合中的元素进行修改。所以只有增删查方法。

##### 集合的增操作

==add()==方法：向集合中==增加==元素（单一）

```
students = set()
students.add('李哲')
students.add('刘毅')
print(students)
```

##### 集合的删操作

remove()方法：删除集合中的指定数据，如果数据不存在则报错

```
products = {'萝卜','白菜','水蜜桃','奥利奥','西红柿','凤梨'}
products.remove('白菜')
print(products)
```

##### 集合中的查操作

==in==：判断某个元素是否在集合中，如果在，则返回True，否则返回False

```
s1 = {'刘帅','英标','高原'}
if '刘帅' in s1:
	print('刘帅在s1中集合中')
else:
	print('刘帅不在s1集合中')
```

##### 集合的遍历操作

```
for i in 集合:
	print(i)
```



## 一、文件操作

### 1、文件操作内容？

在日常操作中，我们对文件的主要操作：==**创建文件、打开文件、文件读写、文件备份**==等等

### 2、文件操作的作用

文件的作用就是为了实现数据的持久化存储！

### 3、文件操作应用场景

Nginx日志文件的读取

保存分析结果到文件

## 二、文件的基本操作

### 1、文件操作三步走

* 打开文件

* 读写文件

* 关闭文件

### 2、open函数打开文件

在python中，使用open()函数，可以打开一个已存在的文件，或者创建一个新文件，

```
f = open(name, mode, 编码模式)
f = open("./data.txt", 'w', encoding='utf-8')
注：返回结果是一个file文件对象（后续方法都是f.方法()）
# 1、打开文件
f = open("./data.txt", 'w')
# 2、写入内容
f.write("人生苦短，我学python！")
# 3、关闭文件
f.close()
print("写入数据成功")
```

![Snipaste_2025-10-17_09-34-17](pict\Snipaste_2025-10-17_09-34-17.png)

### 3、文件读取操作

#### 1、**read(size)方法**：

主要用于文本类型或者二进制文件（图片、音频、视频...）数据的读取

* size表示要从文件中读取的数据长度（单位时字符/字节），如果没有传入size，那么就表示读取文件中所有的数据

* r 以文本方式读取文件：按字符长度读取

* rb以二进制方式读取文件：比如读取图片、音频、视频，按==字节大小==读取

```
f.read() # 读取文件的所有内容
f.read(1024) # 读取1024个字符长度文件内容，字母或者数字
```

```
# 1、打开文件
f = open('./data.txt', 'r', encoding='utf-8')
#2、读取文件
contents = f.read(2)
# 3、关闭文件
print(contents)
f.close()

"D:\Program Files\Python38\python.exe" H:/9-pycode/day02/01-文件操作.py
ab
```

>适合小文件的读取，也适合大文件的读取

#### 2、**readlines()方法**：

主要用于文本类型数据的读取

> readlines可以按照行的方式把整个文件中的内容进行一次性读取，并且**返回的是一个列表**，其中每一行的数据为一个元素。

```
# 1、打开文件
f = open('./data.txt', 'r', encoding='utf-8')
# 2、读写文件
lines = f.readlines()
print(lines)
# 3、关闭文件
f.close()

"D:\Program Files\Python38\python.exe" H:/9-pycode/day02/01-文件操作.py
['111\n', '222\n', '333']
注意：print(lines, end='\n') 默认print（）后面有一个换行\n
如果不想要\n可以print(lines,end='') 
```

>适合小文件的读取

#### 3、**readline()方法**：

一次读取一行内容，每运行一次readline()函数，其就会将文件的指针向下移动一行

> readline(): 没有s，代表一次读取文件一行，适用于大文件读取

```
# 1、打开文件
f = open('./data.txt', 'r', encoding='utf-8')
# 2、读写文件
while True:
    content = f.readline()
    # 判断content是否读取完成
    if not content:
        break
    print(content, end='')
# 3、关闭文件
f.close()
```

![Snipaste_2025-10-17_10-19-33](pict\Snipaste_2025-10-17_10-19-33.png)

#### 扩展：with上下文管理器与for line in f 文件对象

![Snipaste_2025-10-17_10-23-31](pict\Snipaste_2025-10-17_10-23-31.png)

## 三、文件和文件夹操作

作用：针对文件和文件夹进行相关操作，如删除文件，重命名文件，创建目录，移除目录等等

### 1、os模块

在python中我呢见和文件夹的操作要借助os模块里面的相关功能，具体步骤如下：

第一步：导入os模块

```
import os
```

第二步： 调用os模块中的相关方法

```
os.函数名()
```

### 2、与文件操作相关方法

| 编号 | 函数                          | 功能                 |
| ---- | ----------------------------- | -------------------- |
| 1    | os.rename(旧文件名，新文件名) | 对文件进行重命名操作 |
| 2    | os.remove(要删除文件的名称)   | 对文件进行删除操作   |

```
import os
import time
os.rename('python.txt', 'linux.txt')

time.sleep(20)

os.remove('linux.txt')
```

### 3、与文件夹操作相关操作

相关方法：

![Snipaste_2025-10-17_11-00-46](pict\Snipaste_2025-10-17_11-00-46.png)

````
import os
# 判断目录是否存在(不存在就创建目录)
if not os.path.exists('images'):
    os.mkdir('images')
# 判断images中是否存在avatar头像文件夹，不存在就创建
if not os.path.exists('images/avatar'):
    os.mkdir('images/avatar')
# 获取当前位置
print(os.getcwd())
# 切换目录
os.chdir('images/avatar')
# 获取当前目录
print(os.getcwd())
# 切换上一级目录
os.chdir('../')
# 获取当前目录下有哪些文件
print(os.listdir())
# 删除空目录
os.rmdir('avatar')
````

### 4、shutil模块实现递归删除

作用： 用于删除非空目录

```
import os
import shutil

if os.path.exists('images'):
    shutil.rmtree('images')
    print('文件已成功删除！')
```

## 四、nginx日志文件分析统计



![Snipaste_2025-10-17_11-15-14](pict\Snipaste_2025-10-17_11-15-14.png)

```
import os
# 定义两个变量（字典），存放ip的次数，状态码的次数
ip_status = {}
status_status = {}

# 打开文件，访问文件每一行
with open('./nginx_access.log', 'r',encoding='utf-8') as file:
    for line in file:
        # line本质时一个字符串，要进行切割获取ip和状态码
        parts = line.split(' ')
        ip = parts[0]
        status = parts[-2]
        # print(ip, status)

        # 统计ip出现的次数
        # 先判断ip有没有出现在字典key中，如果有，则执行更新操作，如果没有则执行添加操作
        if ip not in ip_status:
            ip_status[ip] = 1
        else:
            ip_status[ip] += 1

        # status 同理
        if status not in status_status:
            status_status[status] = 1
        else:
            status_status[status] += 1

# 把以上得到的结果写入到文件
with open('./nginx_summary.txt', 'w', encoding='utf-8') as summary:
     summary.write('nginx日志之IP的统计结果: \n')
     for ip, count in ip_status.items():
        summary.write(f"{ip}: {count}次\n")

     summary.write('\nnginx日志之stats的统计结果:\n')
     for stats, count in status_status.items():
         summary.write(f"{stats}: {count}次\n")
         
# 输出到当前
print('nginx日志之IP的统计结果:')
for ip, count in ip_status.items():
    print(f"{ip}: {count}次", end=' ')

print('nginx日志之status的统计结果: ')
for stats, count in status_status.items():
    print(f"{stats}: {count}次", end=' ')
```

## 五、python函数定义与使用

函数： 代码重用以及模块化编程

#### 1、为什么需要函数

![Snipaste_2025-10-17_14-50-09](pict\Snipaste_2025-10-17_14-50-09.png)

#### 2、什么是函数

![Snipaste_2025-10-17_14-50-56](pict\Snipaste_2025-10-17_14-50-56.png)

#### 3、函数的定义

```
def 函数名称([参数1， 参数2，....]):
	函数体
	...
	[retuen 返回值]

```



#### 4、函数的调用

在python中，函数和变量一样，都是先定义后使用

```
def 函数名称([参数1， 参数2，....]):
	函数体
	...
	[retuen 返回值]
# 调用函数
函数名称（参数1，参数2，...）

```

#### 5、函数的return返回值

和函数参数都是可选项。如果没有定义返回值，默认返回None

```
# 有返回值的函数
def func(num1,num2):
    result = num1 + num2
    return result

print(func(10, 30))
结果为40

# 没有返回值（没有直接定义）的函数，默认返回值为None
def func2(num1, num2):
    result = num1 + num2
    print(result) ====>返回30

print(func2(10,20))  ===>返回None
结果：
30
None
```

>小结：
>
>return作用：把函数的执行结果返回函数的调用位置
>
>return不仅可以返回结果，还具有终止函数的功能，类似循环中的break关键词
>
>python中，如果一个函数没有返回值，默认返回none；除此之外，return可以同时返回多个结果，这个结果是元组类型的数据。

#### 6、函数封装案例

封装一个验证码函数，用于返回4位长度的随机验证

```
import random
# 定义函数
def generate_code():
	# 定义一个字符串，包含0-9a-zA-Z
	str1 = '0123456789a-zA-Z'   === 通义主动触发（Alt+p）
	# 定义一个空字符串（用于保存取出来的字符串）
	code = ''
	# 编写循环，攻击4次
	for i in range(4):
		# 生成索引下标
		index = random.randint(0, len(str1) - 1)
		# 累加
		code += str1[index]
	# 循环结束，code保存4个字符
	return code

# 调用函数
print(generate_code)
```

## 六、变量的作用域

作用： 指导我们变量在哪里可以使用，在哪里不可以使用

#### 1、什么是变量的作用域

变量作用域指的是变量的作用范围（变量在哪里可用，在哪里不可以使用），只要分两类：全局作用域与局部作用域

其实作用域的划分比较简单，在函数内部定义范围就称之为局部作用域，在函数外部（全局）定义范围就是全局作用域

```
# 全局作用域
def func():
	# 局部作用域
	
```

#### 2、局部变量与全局变量

在python中，定义在函数外部的变量称之为全局变量；定义在函数内部变量就称之为局部变量

![Snipaste_2025-10-17_17-36-49](pict\Snipaste_2025-10-17_17-36-49.png)

#### 3、变量的作用范围

全局变量： 在整个程序范围内都可以直接使用

![Snipaste_2025-10-17_17-38-19](pict\Snipaste_2025-10-17_17-38-19.png)

局部变量： 在函数的调用过程中，开始定义，函数运行过程中生效，函数执行完毕后，销毁

![Snipaste_2025-10-17_17-39-43](pict\Snipaste_2025-10-17_17-39-43.png)

#### 4、global关键字的应用场景

![Snipaste_2025-10-17_17-43-53](pict\Snipaste_2025-10-17_17-43-53.png)

## 七、函数的参数进阶

def func(参数1， 参数2， 参数3)：

...

return 返回值

func(10,20,30)

#### 1、函数的参数

在函数定义与调用时，我们可以根据自己的需求来实现参数额床底。python中，函数的参数一共有两种形式：

①形参  ②实参

形参： 在函数定义时，编写的参数就称之为形式参数

实参： 在函数调用时，所传递的参数就称之为实际参数

![Snipaste_2025-10-17_17-48-34](pict\Snipaste_2025-10-17_17-48-34.png)

#### 2、函数的参数类型（传参）

##### 1、位置参数

理论上，在函数定义时，我们可以为其定义多个参数。但是在函数调用时，我们也应该传递多个参数，正常情况下其要一一对应

![Snipaste_2025-10-17_17-51-21](pict\Snipaste_2025-10-17_17-51-21.png)

##### 2、关键词参数（python特有）

函数调用，通过==**“键=值”**== 形式加以指定。可以让函数更加清晰、容易使用，同时也清楚了参数的顺序需求。

![Snipaste_2025-10-17_17-54-37](pict\Snipaste_2025-10-17_17-54-37.png)

##### 3、函数的默认值参数（缺省参数）

默认参数也叫缺省参数，用于==定义函数时，为参数提供默认值==。

优势：==调用函数可以不用为其进行传值操作，省略部分参数值的传递==（注意：所有位置参数必须出现在默认参数前，包括函数定义和调用）。

```
def user_info(name, age, sex='男'):
    print(f'姓名：{name}, 年龄：{age}, 性别：{sex}')

user_info('owen', 20)
user_info('Alice', 18, '女')

姓名：owen, 年龄：20, 性别：男
姓名：Alice, 年龄：18, 性别：女
```

>谨记：我们在定义缺省参数时，一定要把其写在参数列表的最后。

#### 3、不定长参数

不定长参数也叫可变参数。用于不确定调用的时候会传递多少个参数（不传参也可以）的场景。此时，可用包裹（packing）位置参数，或者包裹关键字参数，来进行参数传递，会显得非常方便。

##### 1、*args不定长位置（元组）参数

```
def func1(*args):
    print(args)
    # print(type(args))
# 不传参数
func1()
# 传递一个参数
func1(10)
# 传递多个参数
func1(10, 20, 30)

()
(10,)
(10, 20, 30)
```

##### 2、**kwargs不定长关键字（字典）参数

包裹关键词参数：只能接受关键词参数，参数的数量可以不固定（0、1、2甚至多个），只能使用**kwargs参数进行接受，最终返回这个变量kwargs是一个字典类型的数据

```

def func2(**kwargs):
    print(kwargs)
    print(type(kwargs))
# 不传参
func2()
# 传递一个参数
func2(a=10)
# 传递多个参数
func2(a=10, b=20)

{}
<class 'dict'>
{'a': 10}
<class 'dict'>
{'a': 10, 'b': 20}
<class 'dict'>
```

##### 3、*args与**kwargs混合使用场景

![Snipaste_2025-10-18_09-51-10](pict\Snipaste_2025-10-18_09-51-10.png)

![Snipaste_2025-10-18_09-53-59](pict\Snipaste_2025-10-18_09-53-59.png)

## 八、Python模块导入

#### 1、什么是python模块

python模块，是一个python文件，以.py结尾，包含了python对象定义和python语句。模块能定义 ==**函数，类和变量**==，模块里也能包含可执行的代码。

> 使用Ctrl+模块名 跳转到底层模块文件

![Snipaste_2025-10-18_10-00-20](pict\Snipaste_2025-10-18_10-00-20.png)

#### 2、模块分类

在python中，模块通常可以分为两大类： ==**内置模块**（目前使用的）==和==**自定义模块**==

#### 3、模块的导入方式

①import导入

import 模块名

import 模块名 as 别名

②from导入

from 模块名 import *

from 模块名 import 功能名

![Snipaste_2025-10-18_10-11-55](pict\Snipaste_2025-10-18_10-11-55.png)

![Snipaste_2025-10-18_10-13-27](pict\Snipaste_2025-10-18_10-13-27.png)

#### 4、math模块

![Snipaste_2025-10-18_10-30-59](pict\Snipaste_2025-10-18_10-30-59.png)

#### 5、内置魔术变量__name__

![Snipaste_2025-10-18_10-31-56](pict\Snipaste_2025-10-18_10-31-56.png)

![Snipaste_2025-10-18_10-37-25](pict\Snipaste_2025-10-18_10-37-25.png)

![Snipaste_2025-10-18_10-42-07](pict\Snipaste_2025-10-18_10-42-07.png)

除了可以用于自定义模块测试以外，if \__name__  ==  '\__main\__\',还可以用于项目主程序的入口。

一个python项目都有一个主程序，如app.py main.py start.py等等，几乎这些主程序都会有一个入口

```
if __name__ == '__main__'
运行程序...
```

![Snipaste_2025-10-18_10-46-39](pict\Snipaste_2025-10-18_10-46-39.png)

#### 6、引入其他模块中的方法在本函数中使用：

![Snipaste_2025-10-18_10-51-11](pict\Snipaste_2025-10-18_10-51-11.png)

#### 7、datetime时间模块

作用：就是用于实现时间筛选、比较以及分析操作

* strptime：“parse”（解析）---->从字符串解析出时间，字符串----->时间。

* strftime: "format"（格式化）----> 将时间格式化为字符串，时间----->字符串。

##### 1、时间格式与解析

![Snipaste_2025-10-18_11-09-19](pict\Snipaste_2025-10-18_11-09-19.png)

```
from datetime import datetime

# 定义时间字符串
str1 = '22/Nov/2024:10:00:00'
# 把以上的时间格式转变为时间对象（字符串格式化，必须先转化为时间对象，事件对象里面的方法才能格式化）
dt = datetime.strptime(str1, '%d/%b/%Y:%H:%M:%S')
# 有了时间格式，我们可以进一步进行格式化，时间对象 转化为  字符串
new_time = datetime.strftime(dt, '%Y-%m-%d %H:%M:%S')
# 打印dt时间对象
print(new_time)
# 打印时间对象类型
print(type(new_time)) 


```

>datetime时间对象，支持格式化为任意时间格式，支持时间范围比较:2024-11-22 10:00:00
><class 'datetime.datetime'>

##### 3、时间范围比较（重点）

在日志分析中，经常需要筛选某个时间范围内的日志记录。通过比较两个datetime 对象，可以轻松实现范围过滤。

时间范围比较：

​	使用 >= 和 <= 运算符比较时间对象。

​	帅选出指定时间范围内的日志对象

案例：

```
from datetime import datetime
# 定义一个时间范围
start_time = datetime.strptime('09/May/2025:15:00:00', '%d/%b/%Y:%H:%M:%S') # 时间对象
end_time = datetime.strptime('09/May/2025:15:30:00', '%d/%b/%Y:%H:%M:%S')  # 时间对象
# 定义一个日志时间，半段是否位于start_time 和end_time之间
log_time = '09/May/2025:15:20:40'
log_time = datetime.strptime(log_time, '%d/%b/%Y:%H:%M:%S')
# datetime时间进行比较
if start_time <= log_time <= end_time:
    print('位于时间范围区间')
else:
    print('超出范围')

位于时间范围区间
```

![Snipaste_2025-10-18_11-41-46](pict\Snipaste_2025-10-18_11-41-46.png)

##### 4、综合案例

```
from datetime import datetime
def filter_log_bytime(log_file,start_time,end_time):
    # 读取文件
    with open(log_file, 'r', encoding='utf-8') as file:
        # 定义列表用于保存读取的日志行
        result = []
        for line in file:
            # 得到第四列
            parts = line.split(' ')
            # 截取日志中的时间字段，转为时间对象
            log_time = datetime.strptime(parts[3].strip('['), '%d/%b/%Y:%H:%M:%S')
            # 判断log_time 是否位于start_time 和end_time 之间
            if start_time <= log_time <= end_time:
                # 符合条件写入列表
                result.append(line)
    return result

# 调用函数
if __name__ == '__main__':
    # 日志格式
    log_file = './nginx_access.log'
    # 定义开始时间和结束时间（为datetime时间对象）
    start_time = datetime.strptime('09/May/2025:15:04:51', '%d/%b/%Y:%H:%M:%S')
    end_time = datetime.strptime('09/May/2025:15:30:00', '%d/%b/%Y:%H:%M:%S')
    # 调用函数
    data = filter_log_bytime(log_file,start_time,end_time)
    for line in data:
        print(line)

```

## 九、数据存储与导出

作用：JSON是把一种通用的数据传输格式，我们经常需要==数据转换为JSON格式==传递给开发使用

#### 1、JSON概述

json是一种轻量级的数据交换格式，常用于存储和传输结构化数据

json文件以==键值对==的形式存储数据，支持多种数据类型： 字符串、数字、布尔值、数组和对象

结构类似于字典： json_str = {"id":"001", "name":"张三"}

>JSON数据格式，对引号特别敏感，要求内部的引号只能是双引号，如果使用单引号，会造成语法错误以及无法解析等问题。

#### 2、python处理json数据

python提供了内置的json模块，支持对json数据的解析与生成。

#### 3、核心方法

**json.dump(obj, file)**: 将python对象（列表、字典、列表+字典）存储为JSON文件。

**json.load(file)**: 从json文件中加载数据为python对象（列表、字典、列表+字典）。

口诀： ==python倾倒用dump，json加载用load；文件去掉s，字符串加上s==。

案例：将一个字典对象data转换为json格式，并写入名为data.json文件中。（ctrl+alt+l 格式化输出）

```
import json

# dict1 = {"name":"tom","age":18,"address":"New York"}
# with open('./data.txt', 'w', encoding='utf-8') as file:
#     json.dump(dict1, file)

# 定义一个列表+字典（多元素转换）
list2 = [{'name':'tom','age':18,'address':'New York'},{"name":"jack","age":24,"address":"China"}]
with open('./data.txt', 'w', encoding='utf-8') as file:
    json.dump(list2, file)

print(file)
```

案例2：把之前写入的data.json文件读取JSON格式，并将其转换为python字典对象data，然后打印出来

```
import json
# 读取文件
with open('./data.txt', 'r', encoding='utf-8') as file:
    data = json.load(file)
    print(data)
```

如果存储的字典中有中文 ，则在json.dump(data,file, ensure_ascii=Flase)

![Snipaste_2025-10-18_16-16-37](pict\Snipaste_2025-10-18_16-16-37.png)

## 十、综合案例：python实现日志分析与统计

![Snipaste_2025-10-18_16-19-47](pict\Snipaste_2025-10-18_16-19-47.png)

![Snipaste_2025-10-18_16-22-57](pict\Snipaste_2025-10-18_16-22-57.png)

#### 1、前置知识点

保存数据到json文件： 使用json.dump() 方法将筛选后的数据存储到文件中。先数据，变更为[{},{},{}]

从json文件加载数据： 使用json.load()方法从json文件中读取数据并还原为python对象。

#### 2、从nginx日志筛选并存储为json文件

①从nginx日志文件中筛选特定时间范围内的记录

②将筛选结果存储到json文件中

③从json文件加载数据并解析

```
from datetime import datetime
import json
# 定义日志筛选函数
def filter_log_by_time(log_file, start_time, end_time):
    # 定义一个变量，用于保存所有符合要求的日志信息
    result = []
    with open(log_file, 'r', encoding='utf-8') as file:
        for line in file:
            parts = line.split(' ')
            log_time = datetime.strptime(parts[3][1:], '%d/%b/%Y:%H:%M:%S')
            if start_time <= log_time <= end_time:
                result.append({
                    'ip': parts[0],
                    'time': parts[3][1:],
                    'method': parts[5][1:],
                    'path': parts[6],
                    'status': parts[8],
                })
    return result

# 定义信息存储函数，将python对象转换为json存储
def save_data_to_json(data, json_file):
    with open(json_file, 'w', encoding='utf-8') as file:
        json.dump(data, file)
    print(f'数据保存成功{json_file}文件中')

# 定义json读取函数，把json数据转换为python函数
def load_data_from_json(json_file):
    with open(json_file, 'r',encoding='utf-8') as file:
        data = json.load(file)
    return data

# 定义一个入口函数
if __name__ == '__main__':
    log_file = './nginx_access.log'
    start_time = datetime.strptime('09/May/2025:15:00:00', '%d/%b/%Y:%H:%M:%S')
    end_time = datetime.strptime('09/May/2025:15:30:00', '%d/%b/%Y:%H:%M:%S')
    filter_logs = filter_log_by_time(log_file, start_time, end_time)
    # 把筛选得到的日志数据存储到json日志中
    save_data_to_json(filter_logs, './data.json')
    # 读取json文件到python中[列表]
    log_data = load_data_from_json('./data.json')
    for log in log_data:
        print(log)

数据保存成功./data.json文件中
{'ip': '210.38.113.64', 'time': '09/May/2025:15:04:51', 'method': 'GET', 'path': '/', 'status': '200'}
{'ip': '81.201.175.55', 'time': '09/May/2025:15:02:34', 'method': 'GET', 'path': '/cart', 'status': '404'}
{'ip': '42.183.172.49', 'time': '09/May/2025:15:05:25', 'method': 'GET', 'path': '/api/data', 'status': '404'}
```

> 格式化文本：Ctrl+Alt+L

## 十一系统资源监控与数据采集

#### 一、psutil模块（重点）

psutil模块作用： 协助我们完成==cpu使用率、内存、磁盘信息、网络==等相关数据的采集！

#### 1、模块介绍

psutil是一个跨平台的python库，用于检索系统的运行信息，包括cpu使用情况、内存状态、磁盘信息、网路统计、进程信息等，非常适合运维和系统监控应用。

#### 2、安装psutil

前置操作：

第一步：启动vmware 中node1服务器（192.168.66.8）

第二步： 在pycharm中ssh连接node1服务器

第三步： 在linux中安装

>dnf install python3-pip -y
>
>pip3 install psutil -i https://pypi.tuna.tsinghua.edu.cn/simple
>
>注意：pip install 软件包名 安装软件
>
>-i 指定镜像源，从哪里下载

第四步：在linux中创建文件目录pythonProject

注意：在centos7中安装psutil时要指定pip3,因为centos7自带python2，如果遇到安装psutil时出现错误，要指定版本：pip3 install psutil==5.9.8 -i https://pypi.tuna.tsinghua.edu.cn/simple

psutil官网：https://github.com/giampaolo/psutil

#### 3、获取cpu的信息

```
# 获取cpu 1s内的使用率
import psutil
cpu_usage = psutil.cpu_percent(interval=1)
print(cpu_usage)
===> 1.0

node1上查看： top
%Cpu(s):  2.8 us,  1.2 sy,  0.0 ni, 95.7 id（比对）,  0.0 wa,  0.2 hi,  0.2 si,  0.0 st
```

获取每个核心的cpu使用率

[第一个核心使用率，第二个核心使用率..]

```
# 获取cpu每个核心数使用情况
for i in range(psutil.cpu_count()):  ===获取有几核cpu并进行循环得出第一核cpu使用率...
    print(f'第{i+1}核心cpu使用情况为：{psutil.cpu_percent(interval=1, percpu=True)[i]}')
```

#### 4、获取内存使用情况

字节Bytes => /1024kb千字节=> /1024MB兆字节 => /1024GB

```
import psutil

mem_info = psutil.virtual_memory()
print(f'总物理内存：{mem_info.total /1024/1024/1024:.2f}G')
print(f'已使用物理内存：{mem_info.used /1024/1024/1024:.2f}G')
print(f'可用的物理内存：{mem_info.available /1024/1024/1024:.2f}G')
print(f'使用率：{mem_info.percent}%')

swap_info = psutil.swap_memory()
print(f'总交换内存：{swap_info.total /1024/1024/1024:.2f}G')
print(f'已使用交换内存：{swap_info.used /1024/1024/1024:.2f}G')
print(f'可用的交换内存：{swap_info.free /1024/1024/1024:.2f}G')
print(f'使用率：{swap_info.percent}%')

总物理内存：1.77G
已使用物理内存：0.78G
可用的物理内存：0.99G
使用率：44.2%

总交换内存：2.00G
已使用交换内存：0.00G
可用的交换内存：2.00G
使用率：0.2%
```

> linux操作系统必须有两个分区：/根分区 + swap分区
>
> 安装操作系统时，默认会分配一定的磁盘空间（物理内存1-2倍，小于8G情况，物理内存2倍大于等于8G，就是1倍）作为临时内存使用；当系统内存资源不足时，系统会自动调用swap分区充当内存使用。

#### 5、获取磁盘使用情况

```
import psutil

# 获取磁盘所有分区信息
for partition in psutil.disk_partitions():
    print(f'磁盘设备名称：{partition.device}, 挂载点：{partition.mountpoint}, 文件类型：{partition.fstype}')

# 获取磁盘使用情况，disk_usage(磁盘挂载路径)方法返回磁盘使用情况
disk_usage = psutil.disk_usage('/')
print(f'根分区/总磁盘大小：{disk_usage.total/1024 ** 3:.2f}GB')
print(f'根分区/已使用磁盘大小：{disk_usage.used/1024 ** 3:.2f}GB')
print(f'根分区/磁盘使用率：{disk_usage.percent}%')

磁盘设备名称：/dev/mapper/cs-root, 挂载点：/, 文件类型：xfs
磁盘设备名称：/dev/nvme0n1p1, 挂载点：/boot, 文件类型：xfs
根分区/总磁盘大小：16.93GB
根分区/已使用磁盘大小：5.64GB
根分区/磁盘使用率：33.3%
```

磁盘IO指的是系统对磁盘设备进行数据读写的过程，主要包括：

① 读写操作：从磁盘中读取数据到内存

② 写入操作： 将数据从内存写入到磁盘

磁盘IO性能通常受磁盘硬件、文件系统类型、RAID配置和磁盘缓存的影响



常见指标：

①IOPS: 每秒处理的IO操作数量。

②吞吐量：磁盘每秒处理的数据量（如MB/s 或者GB/s）。

③比如： 一块SSD的读取额度读标为3.5GB/s 表示他每秒可以读取或者写入3.5GB的数据。

案例：

```
import psutil
import time

# 获取磁盘IO使用情况
disk_io_1 = psutil.disk_io_counters()
# 等待1秒
time.sleep(1)
# 获取磁盘IO使用情况
disk_io_2 = psutil.disk_io_counters()

# 获取磁盘吞吐量（1s内读取数据的总字节数以及1s内写入数据的字节数）
read_bytes = disk_io_2.read_bytes - disk_io_1.read_bytes
write_bytes = disk_io_2.write_bytes - disk_io_1.write_bytes
print(f'磁盘读取吞吐量：{read_bytes/1024 ** 2:.2f}MB/s')
print(f'磁盘写入吞吐量：{write_bytes/1024 ** 2:.2f}MB/s')

# 读取磁盘IOPS（每秒读取磁盘的次数）
read_count = disk_io_2.read_count - disk_io_1.read_count
write_count = disk_io_2.write_count - disk_io_1.write_count
print(f'磁盘读取IOPS：{read_count/1:.2f}IOPS')
print(f'磁盘写入IOPS：{write_count/1:.2f}IOPS')
```

![Snipaste_2025-10-18_20-04-47](pict\Snipaste_2025-10-18_20-04-47.png)

![Snipaste_2025-10-18_20-05-30](pict\Snipaste_2025-10-18_20-05-30.png)

#### 6、获取网络信息

获取网络I/O统计

网络I/O值得时系统通过网络接口进行数据收发的过程，主要包括：

① 接受操作： 从网络中接受数据包

② 发送操作： 通过网络发送数据包

③带宽： 单位时间内网络传输的数据量（如Mbps或者Gbps）

④延迟： 网络数据把从源到目标的时间

⑤吞吐量： 实际传输的数据量，通常小于带宽上限。

![Snipaste_2025-10-18_20-14-43](pict\Snipaste_2025-10-18_20-14-43.png)

```
import psutil
# 获取网络io信息
net_io = psutil.net_io_counters()
print(f'发送字节数字：{net_io.bytes_sent}')
print(f'接收字节数字：{net_io.bytes_recv}')
```

案例：获取网络io吞吐量

```
import psutil
import time
# 获取网络io使用情况
net_io_1 = psutil.net_io_counters()
time.sleep(1)
net_io_2 = psutil.net_io_counters()
# 获取网络吞吐量（1s内发送的字节数以及1s内接收的字节数）
send_bytes = net_io_2.bytes_sent - net_io_1.bytes_sent
recv_bytes = net_io_2.bytes_recv - net_io_1.bytes_recv
print(f'网络发送吞吐量：{send_bytes/1024 ** 2}MB/s')
print(f'网络接受吞吐量：{recv_bytes/1024 ** 2}MB/s')
```

##### 获取网络接口（网卡，如lo网卡、ens33、ens160、eth0）地址信息

```
import psutil

# 获取网卡信息
net_ip_addrs = psutil.net_ip_addrs()
# print(net_ip_addrs)
# 循环遍历得到kv
for interface_name, interface_addrs in net_ip_addrs.items():
    print(f'网卡名称：{interface_name}')
    print(f'网卡ip：{interface_addrs[0].address}')
    print('*'*20)
```

#### 注意：

![Snipaste_2025-10-19_09-31-12](pict\Snipaste_2025-10-19_09-31-12.png)

![Snipaste_2025-10-19_09-32-44](pict\Snipaste_2025-10-19_09-32-44.png)

#### 7、psutil运维场景

监控cpu使用率。超过80%（阈值）就发邮件给运维

文档：https://docs.python.org/release/3.8.3/library/smtplib.html

```
# 1导入模块
import psutil
import smtplib
# 导入邮件MIME模块、Header模块
from email.mime.text import MIMEText
from email.header import Header
# 定义邮件信息
from_addr = '1491506452@qq.com' # 定义邮件从哪里来
to_addr = '1491506452@qq.com' # 邮件发送到哪里去
auth_code = 'jkdxtiwapkpcfhjc'
# 2、获取cpu使用信息
cpu_usage = psutil.cpu_percent(interval=1)
# 3、判断cpu是否超过阈值80，超过就报警
if cpu_usage > 80:
    # 超过80就通过邮件报警
    # 邮件第一部分（写信：来自谁、邮寄给谁、信的主题+内容）
    subject = 'CPU使用过高，超过阈值80%'
    message = f'CPU使用过高，超过了阈值，当前CPU使用率是{cpu_usage}'
    # 写信不能太随意，心智，发送邮件必须通过MIME类型包装你的信件，plain:纯文本，html：HTML格式
    msg = MIMEText(message, 'plain', 'utf-8')
    # 标记信件
    msg['Subject'] = Header(subject, 'utf-8')
    msg['From'] = Header(from_addr)
    msg['To'] = Header(to_addr)
    # 邮件第二部分：邮局（邮件服务器，所有邮件都必须通过邮件服务器发送）
    smtp_server = smtplib.SMTP_SSL("smtp.qq.com", 465)
    smtp_server.login(from_addr, auth_code)
    # 邮件第三部分： 发送邮件
    smtp_server.sendmail(from_addr, to_addr, msg.as_string())
    smtp_server.quit()
    print('邮件发送成功！')

linux中进行压测：stress-ng --cpu 2 --timeout 60s
```

## 二、paramiko模块

目标： paramiko主要用于实现**远程登录、文件上传与下载**功能

#### 1、模块介绍

paramiko模块支持以加密和认证的方式连接远程服务器。可以实现远程文件的的上传，下载或通过ssh远程执行命令。

liunx上写shell脚本远程ssh操作处理密码有两种方法：

① ssh-keygen 实现免密操作，这样远程连接不用输入密码 ssh 192.168.66.9 touch /tmp/123

② expect 自动应答处理密码

![Snipaste_2025-10-19_10-18-43](pict\Snipaste_2025-10-19_10-18-43.png)

#### 2、安装paramiko

```
linux:
pip install paramiko -i https://pypi.tuna.tsinghua.edu.cn/simple
linux中查看python中安装的模块：python3 -m pip list|grep 模块名
```

#### 3、使用paramiko实现远程连接

```
import paramiko
# 创建sshClient对象
ssh = paramiko.SSHClient()
# 首次连接，会要求指纹确认，输入yes，表示允许连接，忽略这个信息，不输入yes
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

# 连接远程服务器hostname=192.168.66.9,port=22,username='root',password='123456'
ssh.connect(hostname='192.168.66.9',port=22,username='root',password='123456')

# 执行命令创建/tmp/python.txt文件
ssh_stdin, ssh_stdout, ssh_stderr = ssh.exec_command('touch /tmp/python.txt')

# 忽略标准输入，获取返回结果标准输出和标准错误
print(ssh_stdout.read().decode())
print(ssh_stderr.read().decode())

# 关闭ssh
ssh.close()
print('输入成功!')
```

将paramiko封装成函数（用时只需要调用函数就行）

```
import paramiko

# 封装成函数
def ssh_exec(hostname,password,cmd,port=22,username='root'):
    # 创建sshclient连接
    ssh = paramiko.SSHClient()
    # 输入yes，忽略这个确认信息
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    # 连接远程服务器
    ssh.connect(hostname=hostname,port=port,username=username,password=password)
    # 执行结果，获取返回信息标准输入+标准输出1+便准错误2
    stdin,stdout,stderr = ssh.exec_command(cmd)
    # 返回最终结果
    print(stdout.read().decode())
    print(stderr.read().decode())
    # 关闭ssh
    ssh.close()

# 调用函数
# ssh_exec('192.168.66.10','123456','ls /')
ssh_exec('192.168.66.10','123456','yum install vsftpd -y')
```

#### 4、使用paramiko免密远程登陆操作

```
linux中在192.168.66.9服务器：
ssh-keygen
ssh-copy-id 192.168.66.10
```

```
import paramiko

# 创建ssh对象
ssh = paramiko.SSHClient()
# 忽略指纹确认
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 准备私钥
private_key = paramiko.RSAKey.from_private_key_file('/root/.ssh/id_rsa')
# 连接服务器
ssh.connect(hostname='192.168.66.10',port=22,username='root',pkey=private_key)
# 执行命令
stdin,stdout,stderr = ssh.exec_command('touch /tmp/paramilo.txt')
# 获取放回结果
print(stdout.read().decode())
print(stderr.read().decode())
# 关闭连接
ssh.close()
```

#### 5、使用paramiko实现文件上传下载

##### 基于账号密码操作

```
import paramiko

# 创建传输连接（192.168.66.9为远程主机ip地址）
trans = paramiko.Transport(('192.168.66.9', 22))  # 数据为元组类型，数据不可变
# 使用账号密码连接kwin2
trans.connect(username='root', password='123456')
# 创建sftp对象，实现上传下载
sftp = paramiko.SFTPClient.from_transport(trans)
# 上传文件
sftp.put('/root/kwin1.txt', '/root/kwin1.txt')
# 下载文件
sftp.get('/root/kwin2.txt', '/root/kwin2.txt')
# 关闭连接
trans.close()
```

##### 基于免密操作

node1：

```
ssh-keygen -t rsa
ssh-copy-id 192.168.66.102
```



```
import paramiko

# 创建传输连接
trans = paramiko.Transport(('192.168.66.9', 22))
# 使用私钥进行连接
parivate_key = paramiko.RSAKey.from_private_key_file('/root/.ssh/id_rsa')
trans.connect(username='root', pkey=parivate_key)
# 创建sftp对象，实现上传下载
sftp = paramiko.SFTPClient.from_transport(trans)
# 上传文件
sftp.put('/etc/fstab', '/tmp/fstab')
# 下载文件
sftp.get('/etc/inittab', '/tmp/inittab')
# 关闭连接
trans.close()
```

## 三、subprocess模块

作用：允许我们==通过python代码执行linux命令（python调用linux系统命令），实现一些脚本自动化==操作！

#### 1、模块介绍

subprocess是python标准库中的一个模块，用于在程序中执行系统命令或者脚本。他在运维场景中非常有用，可以实现任务的自动化和系统操作的变成话

#### 2、subprocess.run()函数

subprocess.run()是subprocess这个函数返回一个CompleteProcess对象，其中包含了执行结果的各种信息，如返回码、标准输出和标准错误等。

>1> 文件，2> 文件，&> 文件
>
>1=  标准输出（stdout）
>
>2 = 标准错误 （stderr）
>
>& = 1 + 2（既包含标准输出也包含标准错误）

###### subprocess.run()的基本用法：

```
import subprocess
import os
# 判断是linux还是windows系统
if os.name == 'nt':   #windows系统
    command = ['cmd', '/c', 'dir']
else:  ==> linux/unix/macOS
    command = ['ls', '-l', '/']
# 执行命令
result = subprocess.run(command,capture_output=True,text=True)
# 打印命令输出结果
print(result.stdout)
print(result.returncode)

# capture_output=True  ====>打印结果输出到终端
# text=True   ====> 输出内容为文本格式
```

#### 3、自动化部署与配置

nginx web软件  => apache web软件（httpd）

```
import subprocess
# 安装httpd
subprocess.run(['dnf', 'install', 'httpd',  '-y'])
# 启动httpd
subprocess.run(['systemctl', 'start', 'httpd'])
# 查看httpd状态
result = subprocess.run(['systemctl', 'status', 'httpd'], capture_output= True, text=True)
print(result.returncode)
print(result.stdout)

```

#### 4、执行shell脚本

提前在/root/script.sh

```
#!/bin/bash
yum install epel-release -y
yum install sl -y
~                    
```

##### pycharm中：

```
import subprocess
# 执行脚本
result = subprocess.run(['bash', '/root/script.sh'], capture_output= True, text=True)
# 打印输出最终结果
print(result.stdout)
```

#### 5、系统资源监控

```
import subprocess
# 创建进程对象，获取磁盘信息
result = subprocess.run(['df', '-h'], capture_output= True, text=True)
print(result.stdout)

# 创建进程对象，获取内存信息
result = subprocess.run(['free', '-h'], capture_output= True, text=True)
print(result.stdout)

print('日常巡检结束')
```

>小结：
>
>subprocess模块作用： 执行系统命令
>
>subprocess在以下场景中使用（自动化部署）、（执行shell脚本）、（系统资源监控）

## 四、python定时采集

#### 1、定时采集意义

在运维开发中，==定时采集==是常见的需求，例如：

① 持续监控==系统资源使用情况==（cpu、内存、磁盘等）

②定期记录==关键日志信息==

③ 定时==备份和检测系统状态==

python提供了简单的time.sleep()方法，接货人循环实现间隔性任务。

#### 2、定时采集任务基础

① 使用while循环保证任务持续执行

②调用time.sleep(interval)设置间隔时间（单位：秒）

![Snipaste_2025-10-19_17-40-11](pict\Snipaste_2025-10-19_17-40-11.png)

#### 3、基于psutil定时采集系统资源

python提供了psutil模块，可以轻松获取系统资源信息：

①CPU使用率： psutil.cpu_percent()

②内存使用率： psutil.virtual_memory().percent

③磁盘使用率： psutil.disk_uage('/').percent

前置概念: try ....except 异常捕获，提前感知python异常，预处理，避免报错种植程序执行

datetime扩展： datetime.now()可以用于获取当前系统时间

```
import psutil
import time
from datetime import datetime
# # 获取当前时间
# current_time = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
try:
    while True:
        # 获取当前时间
        current_time = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        # 获取cpu使用率
        cpu_usage = psutil.cpu_percent(interval=1)
        # 获取内存使用率
        mem_usage = psutil.virtual_memory().percent
        # 获取硬盘使用率
        disk_usage = psutil.disk_usage('/').percent
        print(f'{current_time} | CPU使用率：{cpu_usage}% | 内存使用率：{mem_usage}% | 硬盘使用率：{disk_usage}%')
        time.sleep(5)
except KeyboardInterrupt:
    print('本次系统采集结束')

2025-10-19 17:56:32 | CPU使用率：10.0% | 内存使用率：44.2% | 硬盘使用率：33.6%
2025-10-19 17:56:38 | CPU使用率：1.0% | 内存使用率：44.2% | 硬盘使用率：33.6%
^C本次系统采集结束
```

>小结：
>
>定时采集就是使用while True + （time.sleep()）方法实现
>
>try ...except作用？答：异常处理
>
>datetime模块通过（datetime.now()）方法可以获取当前系统时间

## 五、CSV数据存储

#### 1、CSV数据存储

CSV是一种简单的表和数据存储格式，适合一下场景：

①存储结构化数据（如时间戳、CPU、内存使用率）===> 有行有列的二维表格

②易于用excel或者数据分析工具（如pandas）打开和处理

#### 2、python中的CSV模块

①import csv :导入csv模块

②csv.write :创建一个csv对象

③ csv对象.writerow: 向csv文件中写入一行

```
import csv
# 创建一个csv文件
with open('data.csv', 'w', encoding='utf-8-sig') as file:
    # 创建csv write对象
    write = csv.writer(file)
    # 写入数据（表头）
    write.writerow(['系统时间', 'CPU使用率', '内存使用率','磁盘使用率'])
    # 写入数据
    write.writerow(['2025-09-01 10:10:10', '80%', '60%', '50%'])

文件会下载在linux的pythonProject/day03/data.csv
```

#### 3、csv结合psutil获取系统信息

```
# 1、导入模块
import csv
import psutil
import time
from datetime import datetime
# 2、创建csv文件==> 包含表头信息
with open('system.csv', 'w', encoding='utf-8-sig', newline='') as file:
    write = csv.writer(file)
    write.writerow(['系统时间', 'CPU使用率(%)', '内存使用率(%)','磁盘使用率(%)'])
# 3、使用try except 捕获异常==> while True +time.sleep == > psutil 写入csv文件
try:
    while True:
        # 使用psutil获取系统信息
        cpu_uage = psutil.cpu_percent(interval=1)
        mem_usage = psutil.virtual_memory().percent
        disk_usage = psutil.disk_usage('/').percent
        current_time = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        # 将数据写入到csv文件中
        with open('system.csv', 'a', encoding='utf-8-sig', newline='') as file:
            write = csv.writer(file)
            write.writerow([current_time, cpu_uage, mem_usage, disk_usage])
        # 输出到终端
        print(f'系统时间：{current_time}，CPU使用率：{cpu_uage}%，内存使用率：{mem_usage}%，磁盘使用率：{disk_usage}%')
        # 每隔5s获取
        time.sleep(5)
except:
    print('数据采集结束，数据保存在system.csv中！')

系统时间：2025-10-19 18:46:19，CPU使用率：0.5%，内存使用率：42.5%，磁盘使用率：33.7%
系统时间：2025-10-19 18:46:25，CPU使用率：0.5%，内存使用率：42.5%，磁盘使用率：33.6%
```

## 六、requests模块

作用：requests模块主要用于==实现http请求（get请求、post请求）==，工作中主要用于==爬虫、接口调用以及企业微信告警==！

#### 1、模块介绍

requests是一个非常流行的python库，提供了简介的http请求处理方式。

①http请求:GET请求以及POST请求

②获取数据一般使用GET请求

③发送数据一般使用POST请求，比如登录、注册、支付等等。

#### 2、安装requests库

```
linux中执行： pip install requests

```

#### 3、发送get请求

GET请求用于从服务器获取数据，通常用于获取网页内容或API数据。

```
# 1、导入模块
import requests
# 2、定义url地址
url = 'http://www.baidu.com'
# 3、使用get方法发送请求，返回响应
response = requests.get(url)
# 4、打印响应
print(response.text)
```

#### 4、response响应对象的属性和方法

##### 4.1、response.text

**获取响应的==字符串==内容**

```
# 设置编码格式
response.encoding = 'utf-8'   
```

![Snipaste_2026-05-03_15-50-28](pict\Snipaste_2026-05-03_15-50-28.png)

##### 4.2、response.content

**获取响应的是==字节流（bytes类型）==的数据，如果想要拿到字符串的数据可以使用以下方式**

```
response.content.decode()
```

>问题：requests模块发起请求后，如果结果保存在response中，则reponse.text与response.content有何区别？
>
>答：response.text返回字符串，需要通过response.encoding解决中文乱码
>
>response.content返回字节流，需要通过decode进行解码才能查看文本信息

#### 5、requests携带headers

默认使用requests模块去发送请求的时候，使用的==User-Agent==不是一个正常的浏览器的UA，服务器会检测到，检测到之后，数据或者是不完整的数据

如果我们想要获取到完整的数据，就可以在发送请求的时候，携带一个正常的浏览器的User-Agent。

目的： 就是为了让我们的爬虫程序伪装的更像是一个正常的浏览器在请求对方的服务器，进而拿到正确的数据。

语法：

```
requests.get(url,headers={})

字典中的键值对。就是我们直接从浏览器中复制过来的请求头中的字段
```

案例：

```
# 导入模块
import requests

# 创建url
url = 'http://www.baidu.com'
# 创建headers
headers = {
    'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 18_5 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.5 Mobile/15E148 Safari/604.1'
}
# 发送get请求
response = requests.get(url, headers=headers)
# 获取响应
response.encoding = 'utf-8'
# 获取状态码
print(response.status_code)
print(response.text)
```

#### 6、requests携带参数

查询字符串参数：query_string

格式：以==？==开始，? 后面 ==key=value== 如果有多个参数，每个参数之间使用==&==符号进行连接。

在使用requests模块的时候想要去携带查询字符串参数如何去操作：

###### 6.1、第一种方式：直接对url地址去发送请求即可

```
import requests
# 定义url
url = 'http://www.baidu.com/s?wd=python'  ====>这里直接传参
# 定义headers
headers = {
    'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 18_5 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.5 Mobile/15E148 Safari/604.1'
}
# 发送get请求
response = requests.get(url, headers=headers)
# 获取响应
with open('heima.html', 'w', encoding='utf-8') as file:
    file.write(response.content.decode())
```

###### 6.2、第二种方式：使用get方法中提供的参数，params

```
requests.get(url,params={})

构造请求，参数的字典，只需要将等号左边的内容作为字典的key，等号右边的内容作为value
```

案例：

```
import requests
# 定义url
url = 'http://www.baidu.com/s'   ===>只写？前面的内容
# 定义headers
headers = {
    'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 18_5 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.5 Mobile/15E148 Safari/604.1'
}
# 定义参数params
params = {
    'wd': 'python'
}
# 发送get请求
response = requests.get(url, params=params,headers=headers)  ===传参
# 获取响应
with open('heima.html', 'w', encoding='utf-8') as file:
    file.write(response.content.decode())
```

#### 7、发送post请求

在http协议中，GET 和 POST 是两种最常用的请求方法，用于客户端与服务器之间的通信。他们的主要区别在于数据传递的方式和使用场景（常用于登录，注册等，将数据推送到服务器端，常用于post请求）

```
import requests

# 创建url
url = 'http://httpbin.org/post'
# 创建数据
data = {
    'username': 'admin',
    'password': '123456'
}
# 发送post请求
res = requests.post(url, json=data)
# 打印响应状态码
print(res.status_code)
# 获取响应数据
print(res.json())

```

URL地址：

**请求数据：**

>data是一个python字典，包含了需要提交的键值对数据。
>
>使用json=data参数指定将数据以json格式发送。

**发送请求：**

>requests.post(url,json=data) 将数据作为json负载发送到服务器。

**相应处理：**

>response.status_code:打印响应状态码（200表示成功）。
>
>response.json(): 将相应内容解析为python字典。

#### 8、post和get区别对比

![Snipaste_2025-10-19_20-06-25](pict\Snipaste_2025-10-19_20-06-25.png)

#### 企业微信（api）：

文档：https://developer.work.weixin.qq.com/document/path/99110

```
def send_wechat_msg(message):
    # 1、导入模块
    import requests
    # 2、定义url
    url = 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=ab84e676-913b-457a-a092-5a1246e12ae4'

    # 3、 定义headers
    headers = {'Content-Type': 'application/json'}
    # 4、定义传输格式（严格按照企业微信格式）
    data = {
        "msgtype": "text",
        "text": {
            "content": message
        }
    }
    # 5、发起post请求
    try:
        res = requests.post(url, json=data, headers=headers)
        if res.status_code == 200:
            print('企业微信调用成功！')
        else:
            print('企业微信调用失败！')
    except:
        print('企业微信调用失败！')

# 调用函数
send_wechat_msg('node1服务器cpu使用率超过80%，请及时关注')
```

## 七、阈值检测与企业微信报警

#### 1、任务背景

在生产环境中，资源使用率（如cpu、内存、磁盘）过高会导致性能问题甚至系统崩溃、为了确保系统的稳定运行，需要设置资源使用率的阈值（如cpu使用率 > 80%），并在超出阈值时触发报警，同时记录报警信息到日志文件以便后续排查。

#### 2、任务拆解

①定时监控系统资源（CPU、内存、磁盘）的使用率

②判断资源使用是否超过设定的阈值（如CPU > 80%,内存 > 90%, 磁盘 > 85%)

③ 超过阈值时，触发报警并将报警信息记录到日志文件



#### 3、任务实现

企业微信接口文档地址：https://developer.work.weixin.qq.com/document/path/92455

```
# 1、导入模块
import psutil
import time
from datetime import datetime
import requests
# 2、定义变量cpu、磁盘、内存使用率和webhook_url
LOG_FILE = 'resource_alert.log'
CPU_THRESHOLD = 80
MEMORY_THRESHOLD = 90
DISK_THRESHOLD = 80
WEBHOOK_URL = 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=ab84e676-913b-457a-a092-5a1246e12ae4'

# 3、定义log_alert函数
def log_alert(resource_type,usage,threshold):
    # 记录当前时间
    current_time = datetime.now()
    # 记录一下什么时间，cpu/内存/硬盘使用率超过阈值
    message = f'{current_time} {resource_type}当前使用率{usage}%超过阈值{threshold}%'
    # 打开文件，写入数据
    with open(LOG_FILE,'a',encoding='utf-8') as file:
        file.write(message + '\n')
    print( message)

# 4、定义send_wechat_alert函数
def send_wechat_alert(message):
    # 定义一个headers
    headers = {'Content-Type': 'application/json'}
    # 定义一个data
    data = {
    	"msgtype": "text",
    	"text": {
        	"content": message
    	}
   }
    try:
        res = requests.post(WEBHOOK_URL, json=data, headers=headers)
        if res.status_code == 200:
            print('企业微信调用成功，已将短信发送至运维人员！')
        else:
            print('企业微信调用失败！')
    except:
        print('企业微信调用失败！')

# send_wechat_alert('cpu已超过80%，请及时关注企业微信')

# 5、定义资源使用情况
def check_resource_usage():
    # 获取cpu的使用率
    cpu_usage = psutil.cpu_percent(interval=1)
    if cpu_usage > CPU_THRESHOLD:
        log_alert('cpu', cpu_usage, CPU_THRESHOLD)
        send_wechat_alert(f'cpu使用过高，已超过了阈值{CPU_THRESHOLD}%，当前使用率是{cpu_usage}%')
    # 获取内存的使用率
    memory_uasge = psutil.virtual_memory().percent
    if memory_uasge > MEMORY_THRESHOLD:
        log_alert('内存', memory_uasge, MEMORY_THRESHOLD)
        send_wechat_alert(f'内存使用过高，已超过了阈值{MEMORY_THRESHOLD}%，当前使用率是{memory_uasge}%')

    # 获取磁盘的使用率
    disk_usage = psutil.disk_usage('/').percent
    if disk_usage > DISK_THRESHOLD:
        log_alert('磁盘', disk_usage, DISK_THRESHOLD)
        send_wechat_alert(f'磁盘使用过高，已超过了阈值{DISK_THRESHOLD}%，当前使用率是{disk_usage}%')
    # 打印当前使用率
    print(f'cpu的使用率是{cpu_usage}，内存的使用率是{memory_uasge}，磁盘的使用率是{disk_usage}')
# 6、调用函数
if __name__ == '__main__':
    try:
        print('正在采集资源使用率数据...')
        while True:
            check_resource_usage()
            time.sleep(5)
    except KeyboardInterrupt:
        print('本次数据采集已结束！')
```

> 清空日志：cat /dev/null > resource_alert.log
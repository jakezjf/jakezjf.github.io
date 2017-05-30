abs()函数用来获取绝对值，例如：abs(-3) --> 3

% 运算符，用来连接字符串，例如："%s love %s" % ("I","you") --> I love you

%s 表示由一个字符串来替换，而%d 表示由一个整数来替换，另外一个很常用的就是%f， 它
表示由一个浮点数来替换。

raw_input() 函数，提示输入，像java中的Scanner

file = open("1.txt", "r")
for line in file:
    print(line)
file.close()

这样文件中的的换行符和print的换行符相累加会导致有空行，所以可以用逗号隔开，  print(line),

dir([obj]) 显示对象的属性，如果没有提供参数， 则显示全局变量的名字 
                               
help([obj]) 以一种整齐美观的形式 显示对象的文档字符串， 如果没有提供任何参
数， 则会进入交互式帮助。

int(obj) 将一个对象转换为整数

len(obj) 返回对象的长度

open(fn, mode) 以 mode('r' = 读， 'w'= 写)方式打开一个文件名为 fn 的文件

range([[start,]stop[,step]) 返回一个整数列表。起始值为 start, 结束值为 stop - 1; start

默认值为 0， step默认值为1。

raw_input(str) 等待用户输入一个字符串， 可以提供一个可选的参数 str 用作提示信息。

str(obj) 将一个对象转换为字符串

type(obj) 返回对象的类型（返回值本身是一个 type 对象！） 


- _xxx 不用'from module import *'导入
- __xxx__系统定义名字
- __xxx 类中的私有变量名

切片对象
sequence[起始索引 : 结束索引 : 步进值]。




 self.tenant_name = "aiq+PiGciUmVy4gr0CS8JA=="
 
 
 
 
     def GetDateDiff(self, date):
        if date:
            now_time = time.strftime("%Y-%m-%d %X", time.localtime())
            date2 = datetime.datetime.utcnow()
            # date1 = datetime.datetime(int(now_time[0:4]), int(now_time[5:7]), int(now_time[8:10]),
            #                           int(now_time[11:13]), int(now_time[14:16]), int(now_time[17:19]))
            # date2 = datetime.datetime(int(date[0:4]), int(date[5:7]), int(date[8:10]),
            #                           int(date[11:13])+8, int(date[14:16]), int(date[17:19]))
            # return (date1 - date2).days
            date = datetime.datetime.strptime(date, "%Y-%m-%dT%H:%M:%SZ")
            print date
            print date2
            print (date2 - date).days
            return 5
 
 
 
#### tuple 的坑
tuple 是 python 中的元组，tuple 和 list 的功能类似，但它和 list 还是有区别的，tuple 没有 append()、insert()、remove()等方法，它是不可变的，一经定义，无法修改。

定义一个元组：


	name= ("zhong", "jian", "feng")
	print(name)
	print(len(name))

打印：

	('zhong', 'jian', 'feng')
	3
	
假如我们这样定义元组：

	test = (1)
	print(type(test))
 
 打印：
 
	 <class 'int'>
	 
我们定义的 tuple 不是 tuple 类型，而是 int 型，这是为什么呢，因为 () 号既能表示 tuple 也能表示 数学公式的括号，所以这里产生了歧义。python 为了解决这个歧义，要求在只有一个元素的 tuple 中要在元素末尾加个 ‘，’号，进行区分。

定义只有一个元素的元组：


	test = (1,)
	print(type(test))
 
 打印：
 
	 <class 'tuple'>
	 
	
### dsa

	




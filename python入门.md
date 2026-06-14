# 入门
* .venv虚拟环境 放依赖
## 数据类型
* python是动态类型语言  
* python区分大小写
* 建议ni_hao(蛇形命名法)
int float str bool(True/False)  
list tuple dict set  
NoneType(None)  

常用类型判断函数: type isinstance
```py
num = 1
#还可以这样声明:
num1,num2 = 1,2
print(type(num)) # <class 'int'>
print(isinstance(num,int)) # True
```
尽管python允许混合数据类型，但是不建议
### 字符串
* 转义字符  
\n(换行) \t(制表) \r(回车) \' \"
* 基础拼接  
基础用法和java差不多 不赘述
```py
slogan = "hello" "world" #仅做了解 意义不大
```
* 类型转换  
str()
```py
num = 18
sogan = "今年刚满" + str(num) + "岁~"
print(sogan)
```
* 字符串格式化   
```py
name = "芜柒"
age = 18
# 方式1(和C比较像):
print("我是%s,今年刚满%d岁~" % (name,age))
# 方式2(推荐):  
print(f"我是{name},今年刚满{age}岁~")
```
## 输入输出
```py
str = input("请输入文本")
ptint(f"用户输入{str}")
```
## 运算符
这里只强调特性  
### 算数运算
* / 除法 输出小数(整除也是小数)  
* // 整除   
* ** 幂  
```py
a = 10
b = 3
print(a/b)  # 3.33333333333
print(a//b) # 3
print(a**b) # 1000
```
### 逻辑运算
* and (&&)  
* or  (||)  
* not (!)  
## 流程控制
* if else elif  
```py
# 缩进很重要
if a > b:
    print("a大于b")
    print("这是多行输出")
elif a == b:
    print("a等于b")
else:
    print("a小于b")
```

# 入门
流程:开始->发送http请求->解析结果提取数据->数据处理(清洗)->数据存储->结束
## 合规性
网站根目录可能有robots.txt 告诉爬虫 哪里可以爬 哪里不能爬
* User-Agent: 用户代理，通过该请求头确认爬虫的类型
* Disallow: 禁止访问的资源
* Allow: 允许访问的资源
* Sitemap: 网站地图，帮助爬虫更高效地获取网站内容
* Crawl-delay: 爬取间隔时间，避免频繁访问造成网站压力过大
## 第一个程序
1. 依赖
安装requests库 发送网络请求
```bash
pip install requests
```
安装lxml(高性能html/xml解析库,智齿基于Xpath语法来解析和获取网页数据)
```bash
pip install lxml
```
2. 编码
编程语言排行榜
```python
import requests
from lxml import html
# 定义url
target_url = "https://www.tiobe.com/tiobe-index/"
# 发送请求
response = requests.get(target_url)
document = html.fromstring(response.text) # 将html纯文本转化为对象
# 解析数据
th_list =  document.xpath("//table[@id='top20']/thead/tr/th/text()")# 需要阅读前端代码之后来写
tr_list =  document.xpath("//table[@id='top20']/tbody/tr")
print(th_list)
for tr in tr_list:
    td_list = tr.xpath("./td/text()")
    print(td_list)
```
## Xpath语法
在html/xml中定位元素的查询语言
| 表达式             | 描述                               | 样例                  |
| ------------------ | ---------------------------------- | --------------------- |
| `/`                | 从根节点的直接子元素               | `/html/body/div/h1`   |
| `//`               | 从任意位置选择节点                 | `//h1`                |
| `.`                | 当前节点下查找                     | `./a` 与 `.//a`       |
| `[n]`              | 选择第n个元素                      | `//p[2]`              |
| `[last()]`         | 选择最后一个元素                   | `//p[last()]`         |
| `[@attr]`          | 选择有该属性的元素                 | `//p[@color]`         |
| `[@attr='value']`  | 选择该属性值等于指定值的元素       | `//p[@color='red']`   |
| `*`                | 匹配任何元素节点                   | `//body/div/*`        |
| `@*`               | 匹配元素的任何属性                 | `//body/div/a/@*`     |
| `text()`           | 获取文本内容                       | `//div/p/text()`      |
## csv
Comma-separated values逗号分割值 例如:
```
姓名,性别,年龄
芜柒,女,18
```
可以直接在excel打开
* 注意编码
```bash
chcp # 查看系统默认编码(返回一个数字 数字可能代表GBK编码)
```
在保存文件时注意使用ANSI(系统默认编码)

```py
import csv
# 写
with open("resources/02.csv", "w", encoding="utf-8", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["姓名", "年龄", "性别", "爱好"])
    writer.writeheader()
    writer.writerow({"姓名": "王林", "年龄": 29, "性别": "男", "爱好": "Python,Java"})
    writer.writerow({"姓名": "红蝶", "年龄": 18, "性别": "女", "爱好": "Java"})
# 读
with open("csv_data/02.csv", "r", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row)
```
---
title: Xpath 笔记
publishDate: 2023-05-26 09:55:00
updateDate: 2024-11-28 16:14:32
description: ''
tags:
  - Xpath
language: 'Chinese'
---

## set:difference

`set:difference(Selector 1, Selector 2)`，返回`Selector 1`减去`Selector 2`后剩下的内容，两个参数需要是两个`Selector`对象。

```python
# 选择所有后代不带有class="pager"的class="row"节点
Selector1 = response.xpath('//*[@class="articles"]//*[@class="row"]')
Selector2 = response.xpath('//*[@class="articles"]//*[@class="row"][.//*[@class="pager"]]')
report_list = response.xpath('set:difference(Selector1, Selector2)')
# 也可以写成这样
report_list = response.xpath('set:difference(//*[@class="articles"]//*[@class="row"], //*[@class="articles"]//*[@class="row"][.//*[@class="pager"]])')
```

## 将p标签放置在h1至h6标签中，使用xpath或者css解析都会出现问题

`p`在`h1`标签内，解析的时候会变成`p`和`h1`并列，排在`h1`后面。

如果是`a`标签或者`span`标签在`h1`标签内，则能正确解析嵌套关系

```python
import scrapy
class AaaaSpider(scrapy.Spider):
    name = "aaaa"
    allowed_domains = ["gov.cn"]
    start_urls = [
        "https://ybj.beijing.gov.cn/zwgk/2024zcwj/202406/t20240620_3722457.html"
    ]

    def parse(self, response):
        # p在h1标签内，解析的时候会变成p和h1并列，排在h1后面
        text = "<div><h1><p>测试标题测试标题测试标题测试标题测试标题测试标题</p></h1></div>"
        article = scrapy.Selector(text=text)
        title = article.xpath("//div/node()").getall()
        print(title)
        # 打印结果为['<h1></h1>', '<p>测试标题测试标题测试标题测试标题测试标题测试标题</p>']

        # 如果将上面的测试代码换成a在h1标签内，则能正确解析嵌套关系
        text = "<div><h1><a>测试标题测试标题测试标题测试标题测试标题测试标题</a></h1></div>"
        article = scrapy.Selector(text=text)
        title = article.xpath("//div/node()").getall()
        print(title)
        # 打印结果为['<h1><a>测试标题测试标题测试标题测试标题测试标题测试标题</a></h1>']
```

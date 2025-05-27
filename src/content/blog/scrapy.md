---
title: Scrapy 笔记
publishDate: 2023-05-26 09:55:00
description: ''
tags:
  - Python
  - Scrapy
language: 'Chinese'
---

## 每个爬虫文件指定pipelines的方法

爬虫文件里写的方法会直接覆盖`settings.py`里的`pipelines`写的顺序

```python
class WinBidSpider(scrapy.spiders.Spider):
    name = "winbidVtj"
    allow_domains = ["tjconstruct.cn"]
    start_urls = ['http://www.tjconstruct.cn/Zbgs']
    custom_settings = {
        'ITEM_PIPELINES': {
            'djyanbao.a36kr_projects_pipelines.SpiderPipeline': 300
        }
    }
```

也可以在默认的`pipelines.py`里面写成这样

```python
# 通过爬虫名字有选择性的使用pipelines
def process_item(self, item, spider):
    if name == '你的爬虫名字':
        #你的处理代码
        #
        #
        return item
```

或者

```python
# 通过定义的item有选择性的使用pipelines
def process_item(self, item, spider):
    # 如果你yield的item是在items.py里面定义好的item（例如class StockReportItem(scrapy.Item):）
    # if isinstance(item, 你需要特殊处理的item类型):
    if isinstance(item, StockReportItem):
        #你的处理代码
        #
        #
        return item

    elif isinstance(item, DjyanbaoItem):
        #你的处理代码
        #
        #
        return item
```

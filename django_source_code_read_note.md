--- 
title: {{ title }} 
date: {{ date }}
top: false
cover: false
password:
toc: true 
mathjax: true 
summary: 
tags: 
categories: 
---

看一下django的一些源码，基于python2.7.16，Django 1.6.11，为什么基于这么老的版本是因为公司项目是基于这个版本的

1. 启动
启动是基于manage.py文件
```python
#!/usr/bin/env python
import os
import sys

if __name__ == "__main__":
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "settings")

    from django.core.management import execute_from_command_line

    execute_from_command_line(sys.argv)
```


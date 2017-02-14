---
title: "debug.py => IndexError: list index out of range"
date: 2015-01-22
categories:
    - django
---
若該錯誤來自Django項目的debug.py，解決方法是修改_get_lines_from_file方法，把以下這一行:
```python
source = f.readlines()
```
改為
```python
source = f.read().splitlines()
```

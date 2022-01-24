---
title: django字段总结
categories:
  - 编程
abbrlink: 65abc8f
tags: 
  - django
date: 2019-12-29 20:41:01
---
# django字段的设计
https://docs.djangoproject.com/zh-hans/2.2/ref/models/fields/#module-django.db.models.fields
orm的设计模式
在django的应用过程中，往往要设计model的设计，在model设计过程中要使用各种各样的关键词，这些关键词 关联到了数据库中每条记录的配置

```python
# 引用
from django.db import models 
```


## 通用字段

### null字段
如果设置为 True， 当该字段为空时，Django 会将数据库中该字段设置为NULL，默认为 False。
即数据库置空 如果null=true blank=true 则数据库内容为空白字符串


### blank
Field.blank
默认blank = False ,如果设置为True，则允许该字段为空。
与null不同的是,null存粹与数据库有相关，而blank与验证相关。如果blank=true,则表单验证将允许输入空值,如果blank=False 则不允许输入空值

-------------------------------

### choices

```python
class TaskSwitchModel(models.Model):
    ActionType = models.CharField(choices=[
        ('snmp','_(snmp)'),
        ("vlan","_vlan"),
        ("sysname","_sysname"),
        ("copyright-info","_copyright-info"),
        ("int-vlan","_int-vlan"),
        ("int","_int")],max_length=100,default="")
    test = models.CharField(max_length=1, choices=[('A', ('Author')), ('E', ('Editor'))]), 
```

```python

class TaskSwitchModel(model.ModelForm):
    choices=(
        ('snmp','_(snmp)'),
        ("vlan","_vlan"),
        ("sysname","_sysname"),
        ("copyright-info","_copyright-info"),
        ("int-vlan","_int-vlan"),
        ("int","_int")
    )
    ActionType = model.CharField(choices=choices,max_length=100,default="") 
```

Field.choices
每个元组中的第一个元素是要在模型上设置的实际值，第二个元素是人类可读的名称.
```python
select_choices =[
    ('A','A_'),
    ('B','B_'),
    ('C','C_'),
    ('D','D_'),
]
choices字段迭代
每个元组的第一个元素是要应用于组的名称。第二个元素是一个可迭代的2代元组
select2_select1_choices=[
    ('A',(
        ('A_/','A_movie'),
        ('A_+','A_video'),
        ('A_-','A_City'),
        )
    ),
    ('Video', (
            ('vhs', 'VHS Tape'),
            ('dvd', 'DVD'),
        )
    ),
    ('unknown', 'Unknown'),
] 
```

### db_column
设置数据库列的名称    age = models.CharFiled(db_column="年龄")

### db_index
age = models.CharFiled(db_index=true)
如果设置为True，则创建数据库索引

### default 
可以是值也可以是可调用的对象，每次实例化模型的时候 会调用这个对象或值

```python
# 1
def contact_defult():
    return {"email":"1097690268@qq.com"}

contact_info = JSONField("ContactInfo", default=contact_default)

# 2
member_name = CharFiled(default="小明")
```

### editable 
可编辑字段
如果为False，则该字段将不会显示在管理员或其他任何人中 ModelForm。在模型验证期间也将跳过它们。默认值为True。


---

### unique

name = CharFiled(unique=true)
如上则会保证该字段的值为表中唯一，也就是不会出现同名的人
如果存在了unique 则不再需要db_index 因为unique默认存在索引

## primary_key
如果设置为True，则初始设置为该模型的主键。
如果您未primary_key=True在模型中指定任何字段，则Django会自动添加一个AutoField来保留主键，因此primary_key=True暗示null=False和 unique=True。一个对象只允许使用一个主键。
主键字段是只读的。如果更改现有对象上的主键的值然后保存，则将在旧对象的旁边创建一个新对象。

## 自定义的字段
https://docs.djangoproject.com/zh-hans/2.2/howto/custom-model-fields/
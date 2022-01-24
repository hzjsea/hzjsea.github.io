---
title: drf-pagination
categories:
  - 编程
  - django
abbrlink: 10ccc76d
date: 2020-01-13 18:02:34
---

Introducing drf-pagination
how to use it 

<!--more-->

## django-rest-framework 分页

drf一共有三种分页的方式
- PageNumberPagination
此分页样式在请求查询参数中接受单个数字页码，事先规定好每一页的数量,url中定义分页的标识 ,每一页最大数量等等
形式  https://api.example.org/accounts/?page=3

全局设置
```bash
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination'
} 
```

- LimitOffsetPagination
这种分页样式反映了查找多个数据库记录时使用的语法。客户端同时包括“限制”和“偏移”查询参数。该限制指示要返回的最大项目数，并且与page_size其他样式相同。偏移量表示查询相对于未分页项的完整集合的开始位置。
形式  https://api.example.org/accounts/?limit=100&offset=300

全局设置
```bash
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 100
}
```


自定义设置
page_size  指示页面大小的数值。如果设置，则此PAGE_SIZE设置将覆盖设置。默认值为与PAGE_SIZE设置键相同的值。
page_query_param -一个字符串值，指示用于分页控件的查询参数的名称。
page_size_query_param-如果设置，则这是一个字符串值，指示查询参数的名称，该参数允许客户端根据每个请求设置页面大小。默认为None，表示客户端可能无法控制请求的页面大小。
max_page_size-如果设置，这是一个数字值，表示请求的最大允许页面大小。只有page_size_query_param同时设置了此属性，该属性才有效。
last_page_strings-字符串值的列表或元组，指示可与一起使用page_query_param以请求集合中的最后一页的值。默认为('last',)

```bash
## setting.py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
}
# 设置分页样式
class LargeResultsSetPagination(PageNumberPagination):
    page_size = 1000
    page_size_query_param = 'page_size'
    max_page_size = 10000


# 使用样式---> 局部样式
class BillingRecordsView(generics.ListAPIView):
    queryset = Billing.objects.all()
    serializer_class = BillingRecordsSerializer
    pagination_class = LargeResultsSetPagination

# 请求格式
curl http://localhost:8000/switch/?page=1
```

- CursorPagination
光标分页




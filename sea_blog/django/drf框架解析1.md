---
title: drf框架解析
categories:
  - 编程
  - django
abbrlink: 23293bb2
date: 2019-12-29 20:47:38
---
# drf框架解析1

在drf的view视图中，我们经常会用到viewset视图集的内容。
引用的方法有很多种
1. 直接引用 我们可以使用ModelViewSet
2. 根据不同的需求添加不同的mixins 
```bash
# 列出列表
class SwitchViewSet(viewsets.GenericViewSet,
                    mixins.ListModelMixin):
    serializer_class = SwitchSerializer
    queryset = SwitchModel.objects.all()
```
其中mixins有很多种
- ListModelMixin
- UpdateModelMixin
- CreateModelMixin
- RetrieveModelMixin
- DestroyModelMixin
这些mixins本身就已经提供了最基础的功能，如果我们需要添加附加的需求，可以重写这些方法
## mixins中的内容

```python
class SwitchViewSet(viewsets.GenericViewSet,
                    mixins.ListModelMixin):
    # 获取序列化工具
    serializer_class = SwitchSerializer
    # 获取集合
    queryset = SwitchModel.objects.all()

```
### ListModelMixin
```python
class ListModelMixin:
    """
    List a queryset.
    """
    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())

        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)

```
ListModelMixin 也就是get请求返回当前模型所有列表,filter_queryset达成过滤  get_queryset获取集合 
第二段判断是否有做分类功能
第三段返回serializer
```python
 def filter_queryset(self, queryset):
        """
        Given a queryset, filter it with whichever filter backend is in use.

        You are unlikely to want to override this method, although you may need
        to call it either from a list view, or from a custom `get_object`
        method if you want to apply the configured filtering backend to the
        default queryset.
        """
        for backend in list(self.filter_backends):
            queryset = backend().filter_queryset(self.request, queryset, self)
        return queryset 
```
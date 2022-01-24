---
title: drf-model
categories:
  - 编程
  - django
abbrlink: 4b10b818
date: 2020-01-21 10:17:43
---

drf-model

<!-- more -->



## drf-model
```bash
class TestViewSet(viewsets.ModelViewSet):
    serializer_class = TestSerializer
    queryset = test_Model.objects.all()

    #  create
    @action(methods=['post'],detail=False)
    def create_field(self, request, *args, **kwargs):
        request.data['name'] = "hzj默认"
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)
        headers = self.get_success_headers(serializer.data)
        return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)


    # update
    @action(methods=['patch'], detail=False)
    def update_field(self, request, *args, **kwargs):
        request.data["text"] = "更新dsdsd"
        instance = test_Model.objects.get(id=request.data['id'])
        # instance = self.get_object() 会报错
        # https://docs.djangoproject.com/en/2.2/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin.get_object
        # 默认使用id查找
        serializer = self.get_serializer(instance,data=request.data,partial=True)
        serializer.is_valid()
        serializer.save()
        return Response(serializer.data,status=status.HTTP_200_OK)

    # delete
    @action(methods=['delete'],detail=False)
    def delete_field(self,request,*args,**kwargs):
        instance = test_Model.objects.get(id=request.data['id'])
        if instance:
            self.perform_destroy(instance)
            return Response({"flag":"删除成功"},status=status.HTTP_200_OK)
        else:
            return  Response(status=status.HTTP_400_BAD_REQUEST) 
```
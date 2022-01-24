---
title: DRF框架
categories:
  - 编程
  - django
abbrlink: 7068483c
date: 2019-12-29 20:42:23
---

## rust风格
Representational State Transfer  表现层状态转化（ 资源定位及资源操作） 
表象层面说就是通过get,post,put,delete方式来实现前后台通信的一种轻量级,跨平台,跨语言架构设计风格的web服务
 
http不仅仅是传输协议，更是一种应用协议。REST，即Representational State Transfer的缩写。意为是"表现层状态转化"。RESTful表示一种风格，理解REST前需要理解资源，何谓资源，广义的资源是指可以操作的所有对象。可能是一个系统资源，如txt、jgp、xml …，亦可以是诸如自己定义的虚拟集合的抽象，如books、usrs、times。RESTfutl代表一种简洁、方便、快捷、高效、透明的架构，这取决于你怎样组合。具有如下特点：

1、规范化接口访问方式。这些http操作方法包括GET/POST/PUT/DELETE/OPTIONS等，每个操作方法都代表一个相同意义的操作，它向所有人透明地表明操作方式。比如GET只能读取/拉数据，当然你也可以是添加数据，但建议不要这么做，不然这样就失去了REST的意义。
GET 读取
POST 添加
PUT 修改
DELETE 删除
2、资源标识唯一。通过URI表示一个资源名称，形式/resource/patch。如/users，表示用户的组合，或用户群。当然还可以继续标识某个具体的一个用户，/users/11，表示id为11的用户。当然，你也可以又用一组/usrgroup/11的URI代表操作用户组，不过不建议这么做，因为这样从字面上重复了/users/11资源表示的内容。一个资源URI总是包含第一条实现的方法：
GET /users/11
POST /users/11
PUT /users/11
DELETE /users/11
当然，仅有这些还不足以包括资源操作的所有需求，所以还可以包含请求参数，如GET /users?type=list&page=1。
3、状态的转化。这就是REST的真实含义，指它允许资源URI具有不同的表现形式。同一个URI，根据不同请求方式，执行的动作不同；还可以根据请求的Header Accept的不同返回不同的结果，如text/html、text/css、text/xml等等。也可以理解为body不同。如查询快递一般，可以上次查询，物品还在仓库，而过一段时间已经在路上了。它表示的是一个互动过程。
4、所有信息都包含在当前请求中。请求的方式包含在 Request Header的Method中，还可以包含Accept、Accept-Encoding、Accept-Language，使用Authentication、Cookie等信息表明身份。同样，服务端通过发送Content-Length、Content-Type响应执行情况。最重要的是，需要返回Status Code通知执行状态，如200 201 400 404 500等http code。REST认为，所有信息都能通过请求一次性发送，而不必再采用方式保存状态，请求的信息本身已经说明了请求的意义。
5、无状态性。这是REST最重要的特性之一，这个状态指的是客户端与服务端无需为每次保存请求状态，客户端请求不必考虑当前状态，不必考虑上下文。具体上说，就是不必使用session等工具跟踪、保存请求的特殊性。比如，无论是谁，从哪里发送，几时发送，对同一个URI资源发送请求的结果都是一样的。据传，这样的设计是为当一台服务器宕机时，另一台服务器可以无差别地响应对方的请求。客户端请求只认URI，而不需理后台的设计。
实际上，在如今执行的RESTful设计当中，完全能执行这个要求的很少，最彻底的云服务，大部份为RESTful-like的风格。
6、可实现请求缓存。通过缓存减少请求次数，在HTTP响应里利用Cache-Control、Expires、Last-Modified三个头字段，可以让浏览器缓存资源一段时间。


## drf的视图集
ViewSet类只是一种类型的基于类的视图，即不提供任何方法的处理程序，例如.get()或.post()，而是提供操作，如.list()和.create()。
http://www.sinodocs.cn/api-guide/viewsets.html
### drf字段更新

默认只有list和create 对应的方法就是get 和post
partial = True 局部更新

```python
class TestViewSet(viewsets.ModelViewSet):
    serializer_class = TestSerializer
    queryset = test_Model.objects.all()

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

    @action(methods=['delete'],detail=False)
    def delete_field(self,request,*args,**kwargs):
        instance = test_Model.objects.get(id=request.data['id'])
        if instance:
            self.perform_destroy(instance)
            return Response({"flag":"删除成功"},status=status.HTTP_200_OK)
        else:
            return  Response(status=status.HTTP_400_BAD_REQUEST)
```

### 基本的视图集
使用viewsets.ModelViewSet + DefaultRouter
其中base-name表示基础url  lookup表示查询参数(只根据主键查找)
获取局部地址 http://localhost:8080/{base-name}/{lookup}

## drf路由集
为了更加便捷的开发项目,drf设计了路由集的内容，免去了路由配置中的麻烦，当然他也同时兼容django的路由设计
https://www.django-rest-framework.org/api-guide/routers/

### 使用
```python
from rest_framework import routers

router = DefaultRouter(trailing_slash=False) # 是否以斜杠/结尾
# 声明注册路由
# 同path('switch/',views.per_switch.as_view()),
router.register(prefix='users', viewset=UserViewSet,basename="another_name")
router.register('accounts', AccountViewSet)

# 添加规则，django主动扫描urlpatterns变量获取路由
urlpatterns = router.urls 
```
以上 
- prefix定义前缀
- viewset定义视图
- basename定义前缀别名

形成url格式(本地) ---> 
http://localhost:8080/user/  
http://localhost:8000/accounts/


### 兼容使用
```python
framework rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register('test',viewset=TestViewSet,base_name="test")

urlpatterns = [
    path('switch/',views.switch_list.as_view()),
]

# 变量累加
urlpatterns += router.urls 
```
### 路由集url
django的路由集是真滴变态，当你使用带值参数去查找列表中的特定内容时候，他所表现的url是
^users/{pk}/set_password/$
注意 pk在中间 set_password是你方法的名字
如果你需要修改这些内容，可以在@action中指定
url_path = set_password
url_name = change_password

```python
class SwitchViewSet(viewsets.GenericViewSet,
                    mixins.ListModelMixin):
    serializer_class = SwitchSerializer
    queryset = SwitchModel.objects.all()

    # get
    @action(detail=True,url_path="xx")
    def list_peer(self, request, pk):
        queryset = SwitchModel.objects.get(name=pk)
        print(queryset)
        # page = self.paginate_queryset(queryset)
        # if page is not None:
        #    serializer = self.get_serializer(page, many=True)
        #    return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(queryset)
        return Response(serializer.data) 
```
如上一个列表查询的地址是  http://localhost:8080/{pk}/xx


## drf过滤
### drf通用过滤
```bash
# setting.py
# 全局设置过滤器
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': ['django_filters.rest_framework.DjangoFilterBackend']
}
# views.py
# 单视图设置过滤器
  filter_backends = [django_filters.rest_framework.DjangoFilterBackend]
```
https://www.django-rest-framework.org/api-guide/filtering/
## drf分页
https://www.django-rest-framework.org/api-guide/pagination/
## drf过滤器
https://www.django-rest-framework.org/api-guide/serializers/


## drf解析器与渲染器
视图的有效解析器集始终定义为类列表。当 request.data被访问时，REST框架将检查Content-Type传入请求上的标头，并确定使用哪个解析器来解析请求内容。
如果您未设置内容类型，则大多数客户端将默认使用'application/x-www-form-urlencoded'例如，如果要json使用jQuery和.ajax（）方法发送编码数据，则应确保包括该contentType: 'application/json'设置。

### 设置解析器
仅允许json的内容
```bash
# setting.py
# 全局设置解析
REST_FRAMEWORK = {
    'DEFAULT_PARSER_CLASSES': [
        'rest_framework.parsers.JSONParser',
    ]
} 
# 单视图设置解析
class modelview():
    parser_classes = [JSONParser]
```
其他各类解析
https://www.django-rest-framework.org/api-guide/parsers/

## 渲染器
既然有收的解析器，那也有发出数据的渲染器，
REST框架包括许多内置的Renderer类，使您可以返回各种媒体类型的响应。还支持定义自己的自定义渲染器，这使您可以灵活地设计自己的媒体类型。

### 设置渲染器
```bash
# setting.py
# 全局设置解析
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
    ]
} 
# 单视图设置解析
class modelview()
    renderer_classes = [JSONRenderer]
```

其他渲染器
https://www.django-rest-framework.org/api-guide/renderers/
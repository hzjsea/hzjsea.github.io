---
title: django 用户
categories:
  - 编程
abbrlink: f736b80
tags: 
  - django
date: 2019-12-29 20:47:21
---
# django用户
django为开发者提供了三种不同的用户形式，一种是自带的用户模型(User),另外两种是可以自定义的用户模型，分别是(AbstractUser)(AbstractBaseUser),他们的继承关系分别是 User 继承子AbstractUser 继承自AbstractBaseUser
## django自带的用户模型
使用django自带的用户模型(快速实现)，该模型提供了一些开发过程中会经常使用到的参数，比如用户级别，名字，邮箱，密码等等。我们可以在$\color{red}{from django.contrib.auth.models import User}$中查看User的使用
```python
class User(AbstractUser):
    """
    Users within the Django authentication system are represented by this
    model.

    Username and password are required. Other fields are optional.
    """
    class Meta(AbstractUser.Meta):
        swappable = 'AUTH_USER_MODEL'
```
## 自定义User Model
自定义Usermodel需要注意的是，$\color{red}{需要在setting.py中添加AUTH_USER_MODEL}$ ，由于django寻找用户的方式是在setting.py中寻找这个标示，所以在我们自定义的时候需要添加这个内容
```python
# pro/setting.py
from app.model import .
AUTH_USER_MODEL="app.modelname"
```
其他需要注意的地方，请看文章底部注意栏


### 方法一：扩展AbstractUser
如果你觉得django自带的用户不能满足你所需要的字段，需要额外添加字段的话，你可以使用AbstractUser
```python
# pro/app/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class NewUser(AbstractUser):
    new_field = models.CharField(max_length=100)

# pro/setting.py
AUTH_USER_MODEL="app.modelname"
```

## 方法二：扩展AbstractBaseUser 
AbstractBaseUser中只含有3个field: password, last_login和is_active. 如果你对django user model默认的first_name, last_name不满意, 或者只想保留默认的密码储存方式, 则可以选择这一方式

自定义形式如AbstarctUser一样

使用AbstractBaseUser时，虽然他提供了User最核心的实现，比如password，但是我们依旧需要添加一些必须定义的关键字段和方法

USERNAME_FIELD，用户名称字段，必须设置，且 uniqe=True
```python
class UserProfile(AbstractBaseUser):
    author = models.CharField(max_length=40,unique=True)

    USERNAME_FIELD = 'author'
```
REQUIRED_FIELDS，当通过createsuperuser管理命令创建一个用户时，用于提示一个字段名称列表,不能包含USERNAME_FIELD以及password字段
```python
# pro/models.py

class UserProfile(AbstractBaseUser)
    date_of_birth = models.DateField()
    height = models.FloatField()

    REQUIRED_FIELDS=['date_of_birth','height']
```

is_active   必须定义。 一个布尔属性，标识用户是否是 "active" 的。AbstractBaseUser默认为 Ture

get_full_name()
必须定义。 long格式的用户标识。

get_short_name()
必须定义。 short格式的用户标识。


可用方法
get_username()
返回 USERNAME_FIELD 的值。

is_anonymous()
一直返回 False。用来区分 AnonymousUser。

is_authenticated()
一直返回 Ture。用来告诉用户已被认证。

set_password(raw_password)
设置密码。按照给定的原始字符串设置用户的密码，taking care of the password hashing。 不保存 AbstractBaseUser 对象。如果没有给定密码，密码就会被设置成不使用，同用set_unusable_password()。

check_password(raw_password)
检查密码是否正确。 给定的密码正确返回 True。

set_unusable_password()
设置user无密码。 不同于密码为空，如果使用 check_password()，则不会返回True。不保存AbstractBaseUser 对象。

has_usable_password()
如果设置了set_unusable_password()，返回False。

get_session_auth_hash()
返回密码字段的HMAC。 Used for Session invalidation on password change.

## ⚠️注意
1.在创建任何迁移或者第一次运行 manager.py migrate 前设置 AUTH_USER_MODEL。设置AUTH_USER_MODEL对你的数据库结构有很大的影响。它改变了一些会使用到的表格，并且会影响到一些外键和多对多关系的构造。在你有表格被创建后更改此设置是不被 makemigrations 支持的，并且会导致你需要手动修改数据库结构，从旧用户表中导出数据，可能重新应用一些迁移。
2.由于Django的可交换模型的动态依赖特性的局限，你必须确保 AUTH_USER_MODEL 引用的模型在所属app中第一个迁移文件中被创建（通常命名为 0001_initial），否则你会碰到错误(The easiest way to construct a compliant custom User model is to inherit fromAbstractBaseUser. AbstractBaseUser provides the core implementation of a Usermodel, including hashed passwords and tokenized password resets. You must then provide some key implementation details:)

## 引用User
如果你选择了自定义用户类型，那么当你在view.py中尝试使用外键引用他的时候，你会出现错误，你应该使用$\color{red}{django.contrib.auth.get_user_model来引用模型}
```python
def get_user_model():
    """
    Return the User model that is active in this project.
    """
    try:
        return django_apps.get_model(settings.AUTH_USER_MODEL, require_ready=False)
    except ValueError:
        raise ImproperlyConfigured("AUTH_USER_MODEL must be of the form 'app_label.model_name'")
    except LookupError:
        raise ImproperlyConfigured(
            "AUTH_USER_MODEL refers to model '%s' that has not been installed" % settings.AUTH_USER_MODEL
        
```
同样的，他会根据setting中的AUTH_USER_MODEL来识别当前用户
```python
from django.contrib.auth import get_user_model

User = get_user_model
```
**指定用户**
```python
# pro/app/models.py

class UserProfile(models.Model):
    author = models.ForeignKey(setting.AUTH_USER_MODEL)
```


https://www.cnblogs.com/huchong/p/9804635.html#_label0
https://blog.csdn.net/qq_37049050/article/details/79211059
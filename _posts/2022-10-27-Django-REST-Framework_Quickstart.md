---
layout: post
title: "[Django-REST-Framework] Quickstart"
categories: Django
tags: Django DRF
---

본 게시물은 Django REST Framework의 공식 문서 내용을 정리한 내용입니다.

추가적인 정보를 원하신다면 아래의 링크를 참고하시길 바랍니다.

[https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)

---

파이썬의 대표적인 웹 프레임워크 3개를 꼽으라면

Django, flask, fastAPI 일 것 입니다.

그 중 Django는 가장 많이 쓰이고 있고 또 가장 많이 사랑받고 있는 웹 프레임워크 입니다.

flask와 fastAPI 또한 많은 사랑을 받고 있는 프레임워크 이지만

대부분의 파이썬 백엔드 개발자들은 Django를 선호한다고 합니다.

저는 이번에 진행하는 프로젝트에서

Django를 사용해 백엔드 개발을 맡게 되었습니다.

파이썬 서버를 개발해보는 것이 처음이기 때문에

공식 문서의 도움을 아주 많이 받을 것 같습니다...ㅎㅎ

공식 문서의 내용을 학습하고 만져보면서

지금의 저와 같이 처음 Django를 배워보는 사람들에게

조금이나마 도움이 되고자 글을 작성합니다.

Django REST Framework는 Django 안에서 RESTful API 서버를 쉽게 구축할 수 있도록 도와주는 오픈소스 라이브러리입니다.

여기서 RESTful API란, REST 아키텍처의 제약 조건을 준수하는 애플리케이션 프로그래밍 인터페이스를 뜻합니다.

이것에 대해서는 다음에 따로 글을 작성하도록 하겠습니다.

저는 DRF(Django REST Framework)를 사용해 API를 만들어 볼 것입니다.

앞서 설명드렸던 것 처럼 이 게시물의 내용은 DRF 공식문서를 기반으로 하고 있습니다.

### Quick Start

DRF 공식 문서에서는 개발자들이 빠르게 API를 만들어볼 수 있도록 가이드를 제공해주고 있습니다.

관리자가 시스템의 사용자와 그룹을 보고 편집할 수 있도록 하는 간단한 API를 만들어 보겠습니다.

#### 프로젝트 설정

우선 tutorial 이라는 이름의 프로젝트를 생성한 다음 quickstart 라는 새로운 앱을 시작합니다.

```shell
# 프로젝트 디렉토리 생성
$ mkdir tutorial
$ cd tutorial

# 패키지 종속성을 로컬로 격리하는 가상 환경 생성
$ python3 -m venv env
$ source env/bin/activate  # On Windows use `env\Scripts\activate`

# 가상 환경에 Django 및 Django REST 프레임워크 설치
$ pip3 install django
$ pip3 install djangorestframework

# 단일 어플리케이션으로 새 프로젝트 설정
$ django-admin startproject tutorial .  # Note the trailing '.' character
$ cd tutorial
$ django-admin startapp quickstart
$ cd ..
```

만약 정상적으로 프로젝트가 생성되었다면 구조는 아래와 같을 것입니다.

```shell
$ pwd
<some path>/tutorial
$ find .
.
./manage.py
./tutorial
./tutorial/__init__.py
./tutorial/quickstart
./tutorial/quickstart/__init__.py
./tutorial/quickstart/admin.py
./tutorial/quickstart/apps.py
./tutorial/quickstart/migrations
./tutorial/quickstart/migrations/__init__.py
./tutorial/quickstart/models.py
./tutorial/quickstart/tests.py
./tutorial/quickstart/views.py
./tutorial/asgi.py
./tutorial/settings.py
./tutorial/urls.py
./tutorial/wsgi.py
```

프로젝트 디렉토리 내에 애플리케이션이 생성된 것이 이상하게 보일 수 있습니다.

프로젝트의 네임스페이스를 사용하면 외부 모듈과의 이름 충돌을 피할 수 있습니다.

이제 데이터베이스를 동기화합니다.

```shell
$ python manage.py migrate
```

만약 python3 버전이라면 코드를 아래와 같이 변경 후 입력하시면 됩니다.

```shell
$ python3 manage.py migrate
```

다음으로는 admin이라는 사용자를 생성할 것 입니다.

이 때 사용자의 비밀번호는 password123으로 설정되어 있습니다.

나중에 예제의 뒷 부분에서 지금 생성한 사용자로 인증합니다.

```shell
$ python manage.py createsuperuser --email admin@example.com --username admin
```

지금까지 잘 따라오셨다면 VSC와 Pycharm과 같은 프로그램을 사용해 디렉토리를 열어봅니다.

#### Serializers

먼저 Serializers를 정의해봅니다.

데이터 표현에 사용할 serializers.py 파일을 tutorial/quickstart 경로에 생성해줍니다.

그 후 아래와 같은 코드를 작성합니다.

```python
from django.contrib.auth.models import User, Group
from rest_framework import serializers


class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ['url', 'username', 'email', 'groups']


class GroupSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Group
        fields = ['url', 'name']
```

이 경우 HyperlinkedModelSerializer 와 함께 하이퍼 링크 관계를 사용하고 있습니다.

#### Views

다음은 views 생성입니다.

tutorial/quickstart 경로에 있는 views.py 파일을 열고 아래 코드를 입력합니다.

```python
from django.contrib.auth.models import User, Group
from rest_framework import viewsets
from rest_framework import permissions
from tutorial.quickstart.serializers import UserSerializer, GroupSerializer


class UserViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows users to be viewed or edited.
    """
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticated]


class GroupViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows groups to be viewed or edited.
    """
    queryset = Group.objects.all()
    serializer_class = GroupSerializer
    permission_classes = [permissions.IsAuthenticated]
```

viewsets를 사용해서 여러 개의 view를 작성하는 대신 모든 일반적인 동작들을 클래스로 그룹화합니다.

필요한 경우에는 view를 개별로 세분화 할 수 있지만

위와 같이 viewsets를 사용하면 더욱 깔끔하게 코드를 정리하고 유지할 수 있습니다.

#### URLs

이제 API URL을 연결해봅니다.

tutorial 폴더에 있는 urls.py 파일을 열어주고 아래의 코드를 입력합니다.

```python
from django.urls import include, path
from rest_framework import routers
from tutorial.quickstart import views

router = routers.DefaultRouter()
router.register(r'users', views.UserViewSet)
router.register(r'groups', views.GroupViewSet)

# 자동 URL 라우팅을 사용하여 API를 연결합니다.
# 또한 검색 가능한 API에 대한 로그인 URL도 포함되어 있습니다.
urlpatterns = [
    path('', include(router.urls)),
    path('api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```

앞서 view 대신 viewsets를 사용하였으므로

라우터 클래스에 viewset를 등록하기만 하면 API에 대한 URL conf를 자동으로 생성할 수 있습니다.

만약 API URL에 대한 더 많은 제어가 필요하다면,

단순히 일반적인 클래스 기반 view를 사용하고 URL을 명시적으로 쓰는 것으로 코드를 작성할 수 있습니다.

코드의 마지막 부분에는 탐색 가능한 API와 함께 사용할 수 있는 기본 로그인 및 로그아웃을 확인할 수 있도록 작성되어 있습니다.

이러한 기능은 선택 사항이지만 만약 API에 인증이 필요하고 검색 가능한 API를 사용하려는 경우에는 유용하게 사용할 수 있습니다.

#### Pagination

Pagination을 통해 페이지 당 반환되는 객체(object)의 수를 제어할 수 있습니다.

이를 활성화하려면 tutorial/setting.py 에 아래와 같은 코드를 입력합니다.

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}
```

#### Settings

INSTALLED_APPS 에 'rest_framework' 을 추가합니다.

설정 모듈은 tutorial/settings.py 에 있습니다.

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

#### API 테스트

이제 지금까지 빌드한 API를 테스트 할 준비가 되었습니다.

터미널에서 서버를 시작해봅시다.

```shell
$ python3 manage.py runserver
```

링크(http://127.0.0.1:8000/users/)에 접속하였을 때

화면이 위와 같다면 성공한 것입니다!

POST, GET 등을 직접 눌러보면서 개발자가 간단하게 확인 할 수 있도록

페이지를 띄워주는 것도 DRF만의 큰 장점인 것 같습니다.

---

지금까지 Django REST Framework를 사용해 관리자가 시스템의 사용자와 그룹을 보고 편집할 수 있도록 하는 간단한 API를 작성해보는 Quickstart를 함께 진행해보았습니다.

이 내용만 보더라도 DRF로 API를 작성했을 때의 장점이 무엇인지 알 수 있는 시간이었던 것 같습니다.

다음에는 본격적인 Tutorial인 Project setup을 정리해보겠습니다.

혹시라도 추가할 부분이나 수정할 부분이 있다면 언제든지 댓글 부탁드립니다.

본 게시물은 Django REST Framework의 공식 문서 내용을 정리한 내용입니다.

추가적인 정보를 원하신다면 아래의 링크를 참고하시길 바랍니다.

[https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)

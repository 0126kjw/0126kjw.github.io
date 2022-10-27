---
layout: post
categories: Django
tags: Django DRF
---

본 게시물은 Django REST Framework의 공식 문서 내용을 정리한 내용입니다.

추가적인 정보를 원하신다면 아래의 링크를 참고하시길 바랍니다.

[https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)

---

<br/>

공식 문서를 참고하여 Django REST Framework를 설치하는 과정을 알아보겠습니다.

공식 문서에서 Django REST Framework는 웹 API 구축을 위한 강력하고 유연한 툴킷이라고 소개하고 있습니다.

또한, DRF를 사용해야 하는 이유를 6가지로 설명하고 있습니다.

- [웹 탐색 ​​가능 API](https://restframework.herokuapp.com/) 는 개발자에게 큰 유용성을 제공합니다.
- [OAuth1a](https://www.django-rest-framework.org/api-guide/authentication/#django-rest-framework-oauth) 및 [OAuth2](https://www.django-rest-framework.org/api-guide/authentication/#django-oauth-toolkit) 용 패키지를 포함한 [인증 정책.](https://www.django-rest-framework.org/api-guide/authentication/)
- [ORM](https://www.django-rest-framework.org/api-guide/serializers#modelserializer) 및 [비ORM](https://www.django-rest-framework.org/api-guide/serializers#serializers) 데이터 소스 를 모두 지원하는 [직렬화.](https://www.django-rest-framework.org/api-guide/serializers/)
- 사용자 정의 가능 - [보다](https://www.django-rest-framework.org/api-guide/generic-views/) [강력한](https://www.django-rest-framework.org/api-guide/viewsets/) [기능](https://www.django-rest-framework.org/api-guide/routers/)이 필요하지 않은 경우 [일반 기능 기반 보기](https://www.django-rest-framework.org/api-guide/views#function-based-views)를 사용하십시오 .
- 광범위한 문서 및 [훌륭한 커뮤니티 지원](https://groups.google.com/forum/?fromgroups#!forum/django-rest-framework) .
- [Mozilla](https://www.mozilla.org/en-US/about/) , [Red Hat](https://www.redhat.com/) , [Heroku](https://www.heroku.com/) 및 [Eventbrite](https://www.eventbrite.co.uk/about/) 를 비롯한 국제적으로 인정받는 회사에서 사용하고 신뢰합니다 .

<br/>

### 요구사항

REST Framework를 사용하기 위해서는 다음과 같은 준비물이 필요합니다.

- Python (3.6, 3.7, 3.8, 3.9, 3.10)
- Django (2.2, 3.0, 3.1, 3.2, 4.0, 4.1)

REST Framework는 공식적으로 각 파이썬과 장고 시리즈의 최신 패치 릴리스를 지원한다고 합니다.

<br/>

### 설치

pip를 사용하여 사용자가 원하는 패키지를 설치할 수 있습니다.

아래 코드를 참고하셔서 설치하시면 됩니다.

```shell
$ pip install djangorestframework
$ pip install markdown       # 탐색 가능한 API에 대한 마크 다운 지원.
$ pip install django-filter  # 필터링 지원
```

만약 python3를 사용 중이라면 아래와 같이 코드를 수정 후 입력한다면 정상적으로 설치됩니다.

```shell
$ pip3 install djangorestframework
$ pip3 install markdown       # 탐색 가능한 API에 대한 마크 다운 지원.
$ pip3 install django-filter  # 필터링 지원
```

또한 깃헙에서 clone 하여 설치할 수 있습니다.

```shell
$ git clone https://github.com/encode/django-rest-framework
```

위의 과정을 잘 따라오셨다면 정상적으로 DRF가 설치되었을 것 입니다.

DRF를 프로젝트에 적용하기 위해서는 settings.py 내 INSTALLED_APPS 부분에 아래와 같이 추가해주시면 됩니다.

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

---

간단하게 Django REST Framework 설치하는 방법을 정리해보았습니다.

설치 난이도는 높지 않기 때문에 헷갈릴 일은 없지만

이러한 정보도 필요한 분들이 계실 것이라고 생각해 따로 작성하였습니다.

혹시라도 추가할 부분이나 수정할 부분이 있다면 언제든지 댓글 부탁드립니다.

본 게시물은 Django REST Framework의 공식 문서 내용을 정리한 내용입니다.

<br/>
<br/>
<br/>

추가적인 정보를 원하신다면 아래의 링크를 참고하시길 바랍니다.

[https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)
